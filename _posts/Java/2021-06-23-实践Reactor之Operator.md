---
layout: post
categories: Java
tags: java Reactor Flux Mono Pubisher Operator FluxOperator MonoOperator Reactive
---

Operator在实现上是对Publisher的一种`包装`，Operator的结果依然是一个Publisher

## FluxOperator

```java
package reactor.core.publisher;

import java.util.Objects;
import java.util.function.BiFunction;
import java.util.function.Function;

import org.reactivestreams.Publisher;
import reactor.core.CoreSubscriber;
import reactor.core.Scannable;
import reactor.util.annotation.Nullable;

/**
 * A decorating {@link Flux} {@link Publisher} that exposes {@link Flux} API over an
 * arbitrary {@link Publisher}. Useful to create operators which return a {@link Flux}.
 *
 * @param <I> delegate {@link Publisher} type
 * @param <O> produced type
 */
public abstract class FluxOperator<I, O> extends Flux<O> implements Scannable {

	protected final Flux<? extends I> source;

	/**
	 * Build a {@link FluxOperator} wrapper around the passed parent {@link Publisher}
	 *
	 * @param source the {@link Publisher} to decorate
	 */
	protected FluxOperator(Flux<? extends I> source) {
		this.source = Objects.requireNonNull(source);
	}

	@Override
	@Nullable
	public Object scanUnsafe(Attr key) {
		if (key == Attr.PREFETCH) return getPrefetch();
		if (key == Attr.PARENT) return source;
		return null;
	}

}
```

### InternalFluxOperator

```java
package reactor.core.publisher;

import org.reactivestreams.Publisher;
import reactor.core.CorePublisher;
import reactor.core.CoreSubscriber;
import reactor.core.Scannable;
import reactor.util.annotation.Nullable;

abstract class InternalFluxOperator<I, O> extends FluxOperator<I, O> implements Scannable,
                                                                                OptimizableOperator<O, I> {

	@Nullable
	final OptimizableOperator<?, I> optimizableOperator;

	/**
	 * Build a {@link InternalFluxOperator} wrapper around the passed parent {@link Publisher}
	 *
	 * @param source the {@link Publisher} to decorate
	 */
	protected InternalFluxOperator(Flux<? extends I> source) {
		super(source);
		this.optimizableOperator = source instanceof OptimizableOperator ? (OptimizableOperator) source : null;
	}

	@Override
	@SuppressWarnings("unchecked")
	public final void subscribe(CoreSubscriber<? super O> subscriber) {
		OptimizableOperator operator = this;
		while (true) {
			subscriber = operator.subscribeOrReturn(subscriber);
			if (subscriber == null) {
				// null means "I will subscribe myself", returning...
				return;
			}
			OptimizableOperator newSource = operator.nextOptimizableSource();
			if (newSource == null) {
				operator.source().subscribe(subscriber);
				return;
			}
			operator = newSource;
		}
	}

	@Nullable
	public abstract CoreSubscriber<? super I> subscribeOrReturn(CoreSubscriber<? super O> actual);

	@Override
	public final CorePublisher<? extends I> source() {
		return source;
	}

	@Override
	public final OptimizableOperator<?, ? extends I> nextOptimizableSource() {
		return optimizableOperator;
	}

	@Override
	@Nullable
	public Object scanUnsafe(Attr key) {
		if (key == Attr.PREFETCH) return getPrefetch();
		if (key == Attr.PARENT) return source;
		return null;
	}

}
```

#### FluxMap

```java
package reactor.core.publisher;

import java.util.Objects;
import java.util.function.Function;

import org.reactivestreams.Subscription;
import reactor.core.CorePublisher;
import reactor.core.CoreSubscriber;
import reactor.core.Fuseable;
import reactor.util.annotation.Nullable;

/**
 * Maps the values of the source publisher one-on-one via a mapper function.
 *
 * @param <T> the source value type
 * @param <R> the result value type
 *
 * @see <a href="https://github.com/reactor/reactive-streams-commons">Reactive-Streams-Commons</a>
 */
final class FluxMap<T, R> extends InternalFluxOperator<T, R> {

	final Function<? super T, ? extends R> mapper;

	/**
	 * Constructs a FluxMap instance with the given source and mapper.
	 *
	 * @param source the source Publisher instance
	 * @param mapper the mapper function
	 *
	 * @throws NullPointerException if either {@code source} or {@code mapper} is null.
	 */
	FluxMap(Flux<? extends T> source,
			Function<? super T, ? extends R> mapper) {
		super(source);
		this.mapper = Objects.requireNonNull(mapper, "mapper");
	}

	@Override
	@SuppressWarnings("unchecked")
	public CoreSubscriber<? super T> subscribeOrReturn(CoreSubscriber<? super R> actual) {
		if (actual instanceof Fuseable.ConditionalSubscriber) {
			Fuseable.ConditionalSubscriber<? super R> cs =
					(Fuseable.ConditionalSubscriber<? super R>) actual;
			return new MapConditionalSubscriber<>(cs, mapper);
		}
		return new MapSubscriber<>(actual, mapper);
	}

	static final class MapSubscriber<T, R>
			implements InnerOperator<T, R> {

		final CoreSubscriber<? super R>        actual;
		final Function<? super T, ? extends R> mapper;

		boolean done;

		Subscription s;

		MapSubscriber(CoreSubscriber<? super R> actual,
				Function<? super T, ? extends R> mapper) {
			this.actual = actual;
			this.mapper = mapper;
		}

		@Override
		public void onSubscribe(Subscription s) {
			if (Operators.validate(this.s, s)) {
				this.s = s;

				actual.onSubscribe(this);
			}
		}

		@Override
		public void onNext(T t) {
			if (done) {
				Operators.onNextDropped(t, actual.currentContext());
				return;
			}

			R v;

			try {
				v = Objects.requireNonNull(mapper.apply(t),
						"The mapper returned a null value.");
			}
			catch (Throwable e) {
				Throwable e_ = Operators.onNextError(t, e, actual.currentContext(), s);
				if (e_ != null) {
					onError(e_);
				}
				else {
					s.request(1);
				}
				return;
			}

			actual.onNext(v);
		}

		@Override
		public void onError(Throwable t) {
			if (done) {
				Operators.onErrorDropped(t, actual.currentContext());
				return;
			}

			done = true;

			actual.onError(t);
		}

		@Override
		public void onComplete() {
			if (done) {
				return;
			}
			done = true;

			actual.onComplete();
		}

		@Override
		@Nullable
		public Object scanUnsafe(Attr key) {
			if (key == Attr.PARENT) return s;
			if (key == Attr.TERMINATED) return done;

			return InnerOperator.super.scanUnsafe(key);
		}

		@Override
		public CoreSubscriber<? super R> actual() {
			return actual;
		}

		@Override
		public void request(long n) {
			s.request(n);
		}

		@Override
		public void cancel() {
			s.cancel();
		}
	}

	static final class MapConditionalSubscriber<T, R>
			implements Fuseable.ConditionalSubscriber<T>, InnerOperator<T, R> {

		final Fuseable.ConditionalSubscriber<? super R> actual;
		final Function<? super T, ? extends R>          mapper;

		boolean done;

		Subscription s;

		MapConditionalSubscriber(Fuseable.ConditionalSubscriber<? super R> actual,
				Function<? super T, ? extends R> mapper) {
			this.actual = actual;
			this.mapper = mapper;
		}

		@Override
		public void onSubscribe(Subscription s) {
			if (Operators.validate(this.s, s)) {
				this.s = s;

				actual.onSubscribe(this);
			}
		}

		@Override
		public void onNext(T t) {
			if (done) {
				Operators.onNextDropped(t, actual.currentContext());
				return;
			}

			R v;

			try {
				v = Objects.requireNonNull(mapper.apply(t),
						"The mapper returned a null value.");
			}
			catch (Throwable e) {
				Throwable e_ = Operators.onNextError(t, e, actual.currentContext(), s);
				if (e_ != null) {
					onError(e_);
				}
				else {
					s.request(1);
				}
				return;
			}

			actual.onNext(v);
		}

		@Override
		public boolean tryOnNext(T t) {
			if (done) {
				Operators.onNextDropped(t, actual.currentContext());
				return true;
			}

			R v;

			try {
				v = Objects.requireNonNull(mapper.apply(t),
						"The mapper returned a null value.");
				return actual.tryOnNext(v);
			}
			catch (Throwable e) {
				Throwable e_ = Operators.onNextError(t, e, actual.currentContext(), s);
				if (e_ != null) {
					done = true;
					actual.onError(e_);
					return true;
				}
				else {
					return false;
				}
			}
		}

		@Override
		public void onError(Throwable t) {
			if (done) {
				Operators.onErrorDropped(t, actual.currentContext());
				return;
			}

			done = true;

			actual.onError(t);
		}

		@Override
		public void onComplete() {
			if (done) {
				return;
			}
			done = true;

			actual.onComplete();
		}

		@Override
		@Nullable
		public Object scanUnsafe(Attr key) {
			if (key == Attr.PARENT) return s;
			if (key == Attr.TERMINATED) return done;

			return InnerOperator.super.scanUnsafe(key);
		}

		@Override
		public CoreSubscriber<? super R> actual() {
			return actual;
		}

		@Override
		public void request(long n) {
			s.request(n);
		}

		@Override
		public void cancel() {
			s.cancel();
		}
	}

}
```



## FluxOperator

### InternalMonoOperator

```java
package reactor.core.publisher;

import org.reactivestreams.Publisher;
import reactor.core.CorePublisher;
import reactor.core.CoreSubscriber;
import reactor.core.Scannable;
import reactor.util.annotation.Nullable;

/**
 * A decorating {@link Mono} {@link Publisher} that exposes {@link Mono} API over an
 * arbitrary {@link Publisher} Useful to create operators which return a {@link Mono}.
 *
 * @param <I> delegate {@link Publisher} type
 * @param <O> produced type
 */
abstract class InternalMonoOperator<I, O> extends MonoOperator<I, O> implements Scannable,
                                                                                OptimizableOperator<O, I> {

	@Nullable
	final OptimizableOperator<?, I> optimizableOperator;

	protected InternalMonoOperator(Mono<? extends I> source) {
		super(source);
		this.optimizableOperator = source instanceof OptimizableOperator ? (OptimizableOperator) source : null;
	}

	@Override
	@SuppressWarnings("unchecked")
	public final void subscribe(CoreSubscriber<? super O> subscriber) {
		OptimizableOperator operator = this;
		while (true) {
			subscriber = operator.subscribeOrReturn(subscriber);
			if (subscriber == null) {
				// null means "I will subscribe myself", returning...
				return;
			}
			OptimizableOperator newSource = operator.nextOptimizableSource();
			if (newSource == null) {
				operator.source().subscribe(subscriber);
				return;
			}
			operator = newSource;
		}
	}

	@Nullable
	public abstract CoreSubscriber<? super I> subscribeOrReturn(CoreSubscriber<? super O> actual);

	@Override
	public final CorePublisher<? extends I> source() {
		return source;
	}

	@Override
	public final OptimizableOperator<?, ? extends I> nextOptimizableSource() {
		return optimizableOperator;
	}
}
```

#### MonoMap

```java
package reactor.core.publisher;

import java.util.Objects;
import java.util.function.Function;

import reactor.core.CoreSubscriber;
import reactor.core.Fuseable;

/**
 * Maps the values of the source publisher one-on-one via a mapper function.
 *
 * @param <T> the source value type
 * @param <R> the result value type
 * @see <a href="https://github.com/reactor/reactive-streams-commons">Reactive-Streams-Commons</a>
 */
final class MonoMap<T, R> extends InternalMonoOperator<T, R> {

	final Function<? super T, ? extends R> mapper;

	/**
	 * Constructs a StreamMap instance with the given source and mapper.
	 *
	 * @param source the source Publisher instance
	 * @param mapper the mapper function
	 * @throws NullPointerException if either {@code source} or {@code mapper} is null.
	 */
	MonoMap(Mono<? extends T> source, Function<? super T, ? extends R> mapper) {
		super(source);
		this.mapper = Objects.requireNonNull(mapper, "mapper");
	}

	@Override
	@SuppressWarnings("unchecked")
	public CoreSubscriber<? super T> subscribeOrReturn(CoreSubscriber<? super R> actual) {
		if (actual instanceof Fuseable.ConditionalSubscriber) {
			Fuseable.ConditionalSubscriber<? super R> cs = (Fuseable.ConditionalSubscriber<? super R>) actual;
			return new FluxMap.MapConditionalSubscriber<>(cs, mapper);
		}
		return new FluxMap.MapSubscriber<>(actual, mapper);
	}
}
```



## InnerOperator

```java
package reactor.core.publisher;

import reactor.util.context.Context;

/**
 *
 * @param <I> input operator consumed type
 * @param <O> output operator produced type
 *
 * @author Stephane Maldini
 */
interface InnerOperator<I, O>
		extends InnerConsumer<I>, InnerProducer<O> {

	@Override
	default Context currentContext() {
		return actual().currentContext();
	}
}
```

