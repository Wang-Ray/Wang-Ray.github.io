---
layout: post
title: "实践Spring Webflux"
date: 2018-04-03 11:08:00 +0800
categories: Java
tags: java Webflux Reactor RxJava Reactive
---

[Web on Reactive Stack](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html)

## HttpHandler

反应式HTTP请求处理的最底层契约，支持不同的运行时（参见[实践spring boot之webserver](/java/2020/03/04/实践Spring-Boot之WebServer/)），比如Netty、Tomcat等。

```java
package org.springframework.http.server.reactive;

import reactor.core.publisher.Mono;

/**
 * Lowest level contract for reactive HTTP request handling that serves as a
 * common denominator across different runtimes.
 *
 * <p>Higher-level, but still generic, building blocks for applications such as
 * {@code WebFilter}, {@code WebSession}, {@code ServerWebExchange}, and others
 * are available in the {@code org.springframework.web.server} package.
 *
 * <p>Application level programming models such as annotated controllers and
 * functional handlers are available in the {@code spring-webflux} module.
 *
 * <p>Typically an {@link HttpHandler} represents an entire application with
 * higher-level programming models bridged via
 * {@link org.springframework.web.server.adapter.WebHttpHandlerBuilder}.
 * Multiple applications at unique context paths can be plugged in with the
 * help of the {@link ContextPathCompositeHandler}.
 *
 * @author Arjen Poutsma
 * @author Rossen Stoyanchev
 * @since 5.0
 * @see ContextPathCompositeHandler
 */
public interface HttpHandler {

	/**
	 * Handle the given request and write to the response.
	 * @param request current request
	 * @param response current response
	 * @return indicates completion of request handling
	 */
	Mono<Void> handle(ServerHttpRequest request, ServerHttpResponse response);

}
```

handle的返回值是`Mono<Void>`

### 容器适配

#### ReactorHttpHandlerAdapter

```java
	@Override
	public Mono<Void> apply(HttpServerRequest reactorRequest, HttpServerResponse reactorResponse) {
		NettyDataBufferFactory bufferFactory = new NettyDataBufferFactory(reactorResponse.alloc());
		try {
			ReactorServerHttpRequest request = new ReactorServerHttpRequest(reactorRequest, bufferFactory);
			ServerHttpResponse response = new ReactorServerHttpResponse(reactorResponse, bufferFactory);

			if (request.getMethod() == HttpMethod.HEAD) {
				response = new HttpHeadResponseDecorator(response);
			}

			return this.httpHandler.handle(request, response)
					.doOnError(ex -> logger.trace(request.getLogPrefix() + "Failed to complete: " + ex.getMessage()))
					.doOnSuccess(aVoid -> logger.trace(request.getLogPrefix() + "Handling completed"));
		}
		catch (URISyntaxException ex) {
			if (logger.isDebugEnabled()) {
				logger.debug("Failed to get request URI: " + ex.getMessage());
			}
			reactorResponse.status(HttpResponseStatus.BAD_REQUEST);
			return Mono.empty();
		}
	}
```

#### ServletHttpHandlerAdapter

```java
	@Override
	public void service(ServletRequest request, ServletResponse response) throws ServletException, IOException {
		// Check for existing error attribute first
		if (DispatcherType.ASYNC.equals(request.getDispatcherType())) {
			Throwable ex = (Throwable) request.getAttribute(WRITE_ERROR_ATTRIBUTE_NAME);
			throw new ServletException("Failed to create response content", ex);
		}

		// Start async before Read/WriteListener registration
		AsyncContext asyncContext = request.startAsync();
		asyncContext.setTimeout(-1);

		ServletServerHttpRequest httpRequest;
		try {
			httpRequest = createRequest(((HttpServletRequest) request), asyncContext);
		}
		catch (URISyntaxException ex) {
			if (logger.isDebugEnabled()) {
				logger.debug("Failed to get request  URL: " + ex.getMessage());
			}
			((HttpServletResponse) response).setStatus(400);
			asyncContext.complete();
			return;
		}

		ServerHttpResponse httpResponse = createResponse(((HttpServletResponse) response), asyncContext, httpRequest);
		if (httpRequest.getMethod() == HttpMethod.HEAD) {
			httpResponse = new HttpHeadResponseDecorator(httpResponse);
		}

		AtomicBoolean isCompleted = new AtomicBoolean();
		HandlerResultAsyncListener listener = new HandlerResultAsyncListener(isCompleted, httpRequest);
		asyncContext.addListener(listener);

		HandlerResultSubscriber subscriber = new HandlerResultSubscriber(asyncContext, isCompleted, httpRequest);
        // 订阅
		this.httpHandler.handle(httpRequest, httpResponse).subscribe(subscriber);
	}
```

#### UndertowHttpHandlerAdapter

```java
	@Override
	public void handleRequest(HttpServerExchange exchange) {
		UndertowServerHttpRequest request = null;
		try {
			request = new UndertowServerHttpRequest(exchange, getDataBufferFactory());
		}
		catch (URISyntaxException ex) {
			if (logger.isWarnEnabled()) {
				logger.debug("Failed to get request URI: " + ex.getMessage());
			}
			exchange.setStatusCode(400);
			return;
		}
		ServerHttpResponse response = new UndertowServerHttpResponse(exchange, getDataBufferFactory(), request);

		if (request.getMethod() == HttpMethod.HEAD) {
			response = new HttpHeadResponseDecorator(response);
		}

		HandlerResultSubscriber resultSubscriber = new HandlerResultSubscriber(exchange, request);
		this.httpHandler.handle(request, response).subscribe(resultSubscriber);
	}
```



### HttpWebHandlerAdapter

将WebHandler适配为HttpHandler，以便HttpHandler将请求转给WebHandler

```java
package org.springframework.web.server.adapter;

import java.util.Arrays;
import java.util.HashSet;
import java.util.Set;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import reactor.core.publisher.Mono;

import org.springframework.context.ApplicationContext;
import org.springframework.core.NestedExceptionUtils;
import org.springframework.core.log.LogFormatUtils;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.codec.LoggingCodecSupport;
import org.springframework.http.codec.ServerCodecConfigurer;
import org.springframework.http.server.reactive.HttpHandler;
import org.springframework.http.server.reactive.ServerHttpRequest;
import org.springframework.http.server.reactive.ServerHttpResponse;
import org.springframework.lang.Nullable;
import org.springframework.util.Assert;
import org.springframework.util.StringUtils;
import org.springframework.web.server.ServerWebExchange;
import org.springframework.web.server.WebHandler;
import org.springframework.web.server.handler.WebHandlerDecorator;
import org.springframework.web.server.i18n.AcceptHeaderLocaleContextResolver;
import org.springframework.web.server.i18n.LocaleContextResolver;
import org.springframework.web.server.session.DefaultWebSessionManager;
import org.springframework.web.server.session.WebSessionManager;

/**
 * Default adapter of {@link WebHandler} to the {@link HttpHandler} contract.
 *
 * <p>By default creates and configures a {@link DefaultServerWebExchange} and
 * then invokes the target {@code WebHandler}.
 *
 * @author Rossen Stoyanchev
 * @author Sebastien Deleuze
 * @since 5.0
 */
public class HttpWebHandlerAdapter extends WebHandlerDecorator implements HttpHandler {

    /**
     * Dedicated log category for disconnected client exceptions.
     * <p>Servlet containers dn't expose a a client disconnected callback, see
     * <a href="https://github.com/eclipse-ee4j/servlet-api/issues/44">eclipse-ee4j/servlet-api#44</a>.
     * <p>To avoid filling logs with unnecessary stack traces, we make an
     * effort to identify such network failures on a per-server basis, and then
     * log under a separate log category a simple one-line message at DEBUG level
     * or a full stack trace only at TRACE level.
     */
    private static final String DISCONNECTED_CLIENT_LOG_CATEGORY =
            "org.springframework.web.server.DisconnectedClient";

    /**
     * Tomcat: ClientAbortException or EOFException
     * Jetty: EofException
     * WildFly, GlassFish: java.io.IOException "Broken pipe" (already covered)
     * <p>TODO:
     * This definition is currently duplicated between HttpWebHandlerAdapter
     * and AbstractSockJsSession. It is a candidate for a common utility class.
     * @see #isDisconnectedClientError(Throwable)
     */
    private static final Set<String> DISCONNECTED_CLIENT_EXCEPTIONS =
            new HashSet<>(Arrays.asList("ClientAbortException", "EOFException", "EofException"));


    private static final Log logger = LogFactory.getLog(HttpWebHandlerAdapter.class);

    private static final Log lostClientLogger = LogFactory.getLog(DISCONNECTED_CLIENT_LOG_CATEGORY);


    private WebSessionManager sessionManager = new DefaultWebSessionManager();

    private ServerCodecConfigurer codecConfigurer = ServerCodecConfigurer.create();

    private LocaleContextResolver localeContextResolver = new AcceptHeaderLocaleContextResolver();

    @Nullable
    private ForwardedHeaderTransformer forwardedHeaderTransformer;

    @Nullable
    private ApplicationContext applicationContext;

    /** Whether to log potentially sensitive info (form data at DEBUG, headers at TRACE). */
    private boolean enableLoggingRequestDetails = false;


    public HttpWebHandlerAdapter(WebHandler delegate) {
        super(delegate);
    }


    /**
     * Configure a custom {@link WebSessionManager} to use for managing web
     * sessions. The provided instance is set on each created
     * {@link DefaultServerWebExchange}.
     * <p>By default this is set to {@link DefaultWebSessionManager}.
     * @param sessionManager the session manager to use
     */
    public void setSessionManager(WebSessionManager sessionManager) {
        Assert.notNull(sessionManager, "WebSessionManager must not be null");
        this.sessionManager = sessionManager;
    }

    /**
     * Return the configured {@link WebSessionManager}.
     */
    public WebSessionManager getSessionManager() {
        return this.sessionManager;
    }

    /**
     * Configure a custom {@link ServerCodecConfigurer}. The provided instance is set on
     * each created {@link DefaultServerWebExchange}.
     * <p>By default this is set to {@link ServerCodecConfigurer#create()}.
     * @param codecConfigurer the codec configurer to use
     */
    public void setCodecConfigurer(ServerCodecConfigurer codecConfigurer) {
        Assert.notNull(codecConfigurer, "ServerCodecConfigurer is required");
        this.codecConfigurer = codecConfigurer;

        this.enableLoggingRequestDetails = false;
        this.codecConfigurer.getReaders().stream()
                .filter(LoggingCodecSupport.class::isInstance)
                .forEach(reader -> {
                    if (((LoggingCodecSupport) reader).isEnableLoggingRequestDetails()) {
                        this.enableLoggingRequestDetails = true;
                    }
                });
    }

    /**
     * Return the configured {@link ServerCodecConfigurer}.
     */
    public ServerCodecConfigurer getCodecConfigurer() {
        return this.codecConfigurer;
    }

    /**
     * Configure a custom {@link LocaleContextResolver}. The provided instance is set on
     * each created {@link DefaultServerWebExchange}.
     * <p>By default this is set to
     * {@link org.springframework.web.server.i18n.AcceptHeaderLocaleContextResolver}.
     * @param resolver the locale context resolver to use
     */
    public void setLocaleContextResolver(LocaleContextResolver resolver) {
        Assert.notNull(resolver, "LocaleContextResolver is required");
        this.localeContextResolver = resolver;
    }

    /**
     * Return the configured {@link LocaleContextResolver}.
     */
    public LocaleContextResolver getLocaleContextResolver() {
        return this.localeContextResolver;
    }

    /**
     * Enable processing of forwarded headers, either extracting and removing,
     * or remove only.
     * <p>By default this is not set.
     * @param transformer the transformer to use
     * @since 5.1
     */
    public void setForwardedHeaderTransformer(ForwardedHeaderTransformer transformer) {
        Assert.notNull(transformer, "ForwardedHeaderTransformer is required");
        this.forwardedHeaderTransformer = transformer;
    }

    /**
     * Return the configured {@link ForwardedHeaderTransformer}.
     * @since 5.1
     */
    @Nullable
    public ForwardedHeaderTransformer getForwardedHeaderTransformer() {
        return this.forwardedHeaderTransformer;
    }

    /**
     * Configure the {@code ApplicationContext} associated with the web application,
     * if it was initialized with one via
     * {@link org.springframework.web.server.adapter.WebHttpHandlerBuilder#applicationContext
     * WebHttpHandlerBuilder#applicationContext}.
     * @param applicationContext the context
     * @since 5.0.3
     */
    public void setApplicationContext(ApplicationContext applicationContext) {
        this.applicationContext = applicationContext;
    }

    /**
     * Return the configured {@code ApplicationContext}, if any.
     * @since 5.0.3
     */
    @Nullable
    public ApplicationContext getApplicationContext() {
        return this.applicationContext;
    }

    /**
     * This method must be invoked after all properties have been set to
     * complete initialization.
     */
    public void afterPropertiesSet() {
        if (logger.isDebugEnabled()) {
            String value = this.enableLoggingRequestDetails ?
                    "shown which may lead to unsafe logging of potentially sensitive data" :
                    "masked to prevent unsafe logging of potentially sensitive data";
            logger.debug("enableLoggingRequestDetails='" + this.enableLoggingRequestDetails +
                    "': form data and headers will be " + value);
        }
    }


    @Override
    public Mono<Void> handle(ServerHttpRequest request, ServerHttpResponse response) {

        if (this.forwardedHeaderTransformer != null) {
            request = this.forwardedHeaderTransformer.apply(request);
        }

        ServerWebExchange exchange = createExchange(request, response);

        LogFormatUtils.traceDebug(logger, traceOn ->
                exchange.getLogPrefix() + formatRequest(exchange.getRequest()) +
                        (traceOn ? ", headers=" + formatHeaders(exchange.getRequest().getHeaders()) : ""));

        return getDelegate().handle(exchange)
                .doOnSuccess(aVoid -> logResponse(exchange))
                .onErrorResume(ex -> handleUnresolvedError(exchange, ex))
                .then(Mono.defer(response::setComplete));
    }

    protected ServerWebExchange createExchange(ServerHttpRequest request, ServerHttpResponse response) {
        return new DefaultServerWebExchange(request, response, this.sessionManager,
                getCodecConfigurer(), getLocaleContextResolver(), this.applicationContext);
    }

    private String formatRequest(ServerHttpRequest request) {
        String rawQuery = request.getURI().getRawQuery();
        String query = StringUtils.hasText(rawQuery) ? "?" + rawQuery : "";
        return "HTTP " + request.getMethod() + " \"" + request.getPath() + query + "\"";
    }

    private void logResponse(ServerWebExchange exchange) {
        LogFormatUtils.traceDebug(logger, traceOn -> {
            HttpStatus status = exchange.getResponse().getStatusCode();
            return exchange.getLogPrefix() + "Completed " + (status != null ? status : "200 OK") +
                    (traceOn ? ", headers=" + formatHeaders(exchange.getResponse().getHeaders()) : "");
        });
    }

    private String formatHeaders(HttpHeaders responseHeaders) {
        return this.enableLoggingRequestDetails ?
                responseHeaders.toString() : responseHeaders.isEmpty() ? "{}" : "{masked}";
    }

    private Mono<Void> handleUnresolvedError(ServerWebExchange exchange, Throwable ex) {

        ServerHttpRequest request = exchange.getRequest();
        ServerHttpResponse response = exchange.getResponse();
        String logPrefix = exchange.getLogPrefix();

        if (isDisconnectedClientError(ex)) {
            if (lostClientLogger.isTraceEnabled()) {
                lostClientLogger.trace(logPrefix + "Client went away", ex);
            }
            else if (lostClientLogger.isDebugEnabled()) {
                lostClientLogger.debug(logPrefix + "Client went away: " + ex +
                        " (stacktrace at TRACE level for '" + DISCONNECTED_CLIENT_LOG_CATEGORY + "')");
            }
            return Mono.empty();
        }
        else if (response.setStatusCode(HttpStatus.INTERNAL_SERVER_ERROR)) {
            logger.error(logPrefix + "500 Server Error for " + formatRequest(request), ex);
            return Mono.empty();
        }
        else {
            // After the response is committed, propagate errors to the server..
            logger.error(logPrefix + "Error [" + ex + "] for " + formatRequest(request) +
                    ", but ServerHttpResponse already committed (" + response.getStatusCode() + ")");
            return Mono.error(ex);
        }
    }

    private boolean isDisconnectedClientError(Throwable ex)  {
        String message = NestedExceptionUtils.getMostSpecificCause(ex).getMessage();
        message = (message != null ? message.toLowerCase() : "");
        String className = ex.getClass().getSimpleName();
        return (message.contains("broken pipe") || DISCONNECTED_CLIENT_EXCEPTIONS.contains(className));
    }

}

```

### ServerWebExchange



## WebHandler

Web请求处理。spring boot默认基于`webHandler`这个bean id来初始化`WebHandler`，默认的`WebHandler`实现是`DispatcherHandler`，可以使用自定义的覆盖。

```
# 定义bean id webHandler
org.springframework.web.server.adapter.WebHttpHandlerBuilder#WEB_HANDLER_BEAN_NAME
# 声明bean webHandler（DispatcherHandler）
org.springframework.web.reactive.config.WebFluxConfigurationSupport#webHandler
# 装载webHandler
org.springframework.web.server.adapter.WebHttpHandlerBuilder#applicationContext
```

WebHandler：

```java
package org.springframework.web.server;

import reactor.core.publisher.Mono;

import org.springframework.web.server.adapter.HttpWebHandlerAdapter;
import org.springframework.web.server.adapter.WebHttpHandlerBuilder;

/**
 * Contract to handle a web request.
 *
 * <p>Use {@link HttpWebHandlerAdapter} to adapt a {@code WebHandler} to an
 * {@link org.springframework.http.server.reactive.HttpHandler HttpHandler}.
 * The {@link WebHttpHandlerBuilder} provides a convenient way to do that while
 * also optionally configuring one or more filters and/or exception handlers.
 *
 * @author Rossen Stoyanchev
 * @since 5.0
 */
public interface WebHandler {

	/**
	 * Handle the web server exchange.
	 * @param exchange the current server exchange
	 * @return {@code Mono<Void>} to indicate when request handling is complete
	 */
	Mono<Void> handle(ServerWebExchange exchange);

}

```

### DispatcherHandler

好比Spring MVC里面的DispatcherServlet，负责请求分发到具体的业务处理handler。

```java
package org.springframework.web.reactive;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.Map;

import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

import org.springframework.beans.factory.BeanFactoryUtils;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.core.annotation.AnnotationAwareOrderComparator;
import org.springframework.http.HttpStatus;
import org.springframework.lang.Nullable;
import org.springframework.web.server.ResponseStatusException;
import org.springframework.web.server.ServerWebExchange;
import org.springframework.web.server.WebHandler;
import org.springframework.web.server.adapter.WebHttpHandlerBuilder;

/**
 * Central dispatcher for HTTP request handlers/controllers. Dispatches to
 * registered handlers for processing a request, providing convenient mapping
 * facilities.
 *
 * <p>{@code DispatcherHandler} discovers the delegate components it needs from
 * Spring configuration. It detects the following in the application context:
 * <ul>
 * <li>{@link HandlerMapping} -- map requests to handler objects
 * <li>{@link HandlerAdapter} -- for using any handler interface
 * <li>{@link HandlerResultHandler} -- process handler return values
 * </ul>
 *
 * <p>{@code DispatcherHandler} is also designed to be a Spring bean itself and
 * implements {@link ApplicationContextAware} for access to the context it runs
 * in. If {@code DispatcherHandler} is declared with the bean name "webHandler"
 * it is discovered by {@link WebHttpHandlerBuilder#applicationContext} which
 * creates a processing chain together with {@code WebFilter},
 * {@code WebExceptionHandler} and others.
 *
 * <p>A {@code DispatcherHandler} bean declaration is included in
 * {@link org.springframework.web.reactive.config.EnableWebFlux @EnableWebFlux}
 * configuration.
 *
 * @author Rossen Stoyanchev
 * @author Sebastien Deleuze
 * @author Juergen Hoeller
 * @since 5.0
 * @see WebHttpHandlerBuilder#applicationContext(ApplicationContext)
 */
public class DispatcherHandler implements WebHandler, ApplicationContextAware {

	@SuppressWarnings("ThrowableInstanceNeverThrown")
	private static final Exception HANDLER_NOT_FOUND_EXCEPTION =
			new ResponseStatusException(HttpStatus.NOT_FOUND, "No matching handler");


	@Nullable
	private List<HandlerMapping> handlerMappings;

	@Nullable
	private List<HandlerAdapter> handlerAdapters;

	@Nullable
	private List<HandlerResultHandler> resultHandlers;


	/**
	 * Create a new {@code DispatcherHandler} which needs to be configured with
	 * an {@link ApplicationContext} through {@link #setApplicationContext}.
	 */
	public DispatcherHandler() {
	}

	/**
	 * Create a new {@code DispatcherHandler} for the given {@link ApplicationContext}.
	 * @param applicationContext the application context to find the handler beans in
	 */
	public DispatcherHandler(ApplicationContext applicationContext) {
		initStrategies(applicationContext);
	}


	/**
	 * Return all {@link HandlerMapping} beans detected by type in the
	 * {@link #setApplicationContext injected context} and also
	 * {@link AnnotationAwareOrderComparator#sort(List) sorted}.
	 * <p><strong>Note:</strong> This method may return {@code null} if invoked
	 * prior to {@link #setApplicationContext(ApplicationContext)}.
	 * @return immutable list with the configured mappings or {@code null}
	 */
	@Nullable
	public final List<HandlerMapping> getHandlerMappings() {
		return this.handlerMappings;
	}

	@Override
	public void setApplicationContext(ApplicationContext applicationContext) {
		initStrategies(applicationContext);
	}


	protected void initStrategies(ApplicationContext context) {
		Map<String, HandlerMapping> mappingBeans = BeanFactoryUtils.beansOfTypeIncludingAncestors(
				context, HandlerMapping.class, true, false);

		ArrayList<HandlerMapping> mappings = new ArrayList<>(mappingBeans.values());
		AnnotationAwareOrderComparator.sort(mappings);
		this.handlerMappings = Collections.unmodifiableList(mappings);

		Map<String, HandlerAdapter> adapterBeans = BeanFactoryUtils.beansOfTypeIncludingAncestors(
				context, HandlerAdapter.class, true, false);

		this.handlerAdapters = new ArrayList<>(adapterBeans.values());
		AnnotationAwareOrderComparator.sort(this.handlerAdapters);

		Map<String, HandlerResultHandler> beans = BeanFactoryUtils.beansOfTypeIncludingAncestors(
				context, HandlerResultHandler.class, true, false);

		this.resultHandlers = new ArrayList<>(beans.values());
		AnnotationAwareOrderComparator.sort(this.resultHandlers);
	}


	@Override
	public Mono<Void> handle(ServerWebExchange exchange) {
		if (this.handlerMappings == null) {
			return createNotFoundError();
		}
		return Flux.fromIterable(this.handlerMappings)
				.concatMap(mapping -> mapping.getHandler(exchange))
				.next()
				.switchIfEmpty(createNotFoundError())
				.flatMap(handler -> invokeHandler(exchange, handler))
				.flatMap(result -> handleResult(exchange, result));
	}

	private <R> Mono<R> createNotFoundError() {
		return Mono.defer(() -> {
			Exception ex = new ResponseStatusException(HttpStatus.NOT_FOUND, "No matching handler");
			return Mono.error(ex);
		});
	}

	private Mono<HandlerResult> invokeHandler(ServerWebExchange exchange, Object handler) {
		if (this.handlerAdapters != null) {
			for (HandlerAdapter handlerAdapter : this.handlerAdapters) {
				if (handlerAdapter.supports(handler)) {
					return handlerAdapter.handle(exchange, handler);
				}
			}
		}
		return Mono.error(new IllegalStateException("No HandlerAdapter: " + handler));
	}

	private Mono<Void> handleResult(ServerWebExchange exchange, HandlerResult result) {
		return getResultHandler(result).handleResult(exchange, result)
				.onErrorResume(ex -> result.applyExceptionHandler(ex).flatMap(exceptionResult ->
						getResultHandler(exceptionResult).handleResult(exchange, exceptionResult)));
	}

	private HandlerResultHandler getResultHandler(HandlerResult handlerResult) {
		if (this.resultHandlers != null) {
			for (HandlerResultHandler resultHandler : this.resultHandlers) {
				if (resultHandler.supports(handlerResult)) {
					return resultHandler;
				}
			}
		}
		throw new IllegalStateException("No HandlerResultHandler for " + handlerResult.getReturnValue());
	}

}

```

#### HandlerMapping

基于匹配寻找Handler

#### HandlerAdapter

适配多种不同类型的Handler，解耦DispatcherHandler和Handler

| implement                    | support          | comment |
| ---------------------------- | ---------------- | ------- |
| HandlerFunctionAdapter       | HandlerFunction  |         |
| RequestMappingHandlerAdapter | HandlerMethod    |         |
| SimpleHandlerAdapter         | WebHandler       |         |
| WebSocketHandlerAdapter      | WebSocketHandler |         |



#### HandlerResultHandler

基于handler处理结果HandlerResult，执行后续处理，比如直接应答或渲染视图页面

| implement                   |                                                              |      |
| --------------------------- | ------------------------------------------------------------ | ---- |
| ResponseEntityResultHandler | `ResponseEntity`, typically from `@Controller` instances.    |      |
| ServerResponseResultHandler | `ServerResponse`, typically from functional endpoints.       |      |
| ResponseBodyResultHandler   | Handle return values from `@ResponseBody` methods or `@RestController` classes. |      |
| ViewResolutionResultHandler | `CharSequence`, [`View`](https://docs.spring.io/spring-framework/docs/5.2.4.RELEASE/javadoc-api/org/springframework/web/reactive/result/view/View.html), [Model](https://docs.spring.io/spring-framework/docs/5.2.4.RELEASE/javadoc-api/org/springframework/ui/Model.html), `Map`, [Rendering](https://docs.spring.io/spring-framework/docs/5.2.4.RELEASE/javadoc-api/org/springframework/web/reactive/result/view/Rendering.html), or any other `Object` is treated as a model attribute. |      |

### Handler

handler指业务处理方法，比如Controller里的业务处理方法。

#### ExceptionHandler

针对指定handler相关的异常处理，包括Handler调用和HandlerResultHandler调用

ExceptionHandlerMethodResolver

## WebFilter



```java
package org.springframework.web.server;

import reactor.core.publisher.Mono;

/**
 * Contract for interception-style, chained processing of Web requests that may
 * be used to implement cross-cutting, application-agnostic requirements such
 * as security, timeouts, and others.
 *
 * @author Rossen Stoyanchev
 * @since 5.0
 */
public interface WebFilter {

	/**
	 * Process the Web request and (optionally) delegate to the next
	 * {@code WebFilter} through the given {@link WebFilterChain}.
	 * @param exchange the current server exchange
	 * @param chain provides a way to delegate to the next filter
	 * @return {@code Mono<Void>} to indicate when request processing is complete
	 */
	Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain);

}

```

### DefaultWebFilterChain

### CORS

CorsWebFilter

CrossOrigin

## Exception

装饰WebHandler，负责WebHandler的异常处理

```java
package org.springframework.web.server;

import reactor.core.publisher.Mono;

/**
 * Contract for handling exceptions during web server exchange processing.
 *
 * @author Rossen Stoyanchev
 * @since 5.0
 */
public interface WebExceptionHandler {

	/**
	 * Handle the given exception. A completion signal through the return value
	 * indicates error handling is complete while an error signal indicates the
	 * exception is still not handled.
	 * @param exchange the current exchange
	 * @param ex the exception to handle
	 * @return {@code Mono<Void>} to indicate when exception handling is complete
	 */
	Mono<Void> handle(ServerWebExchange exchange, Throwable ex);

}

```

## WebSessionManager

## ServerCodecConfigurer

## LocaleContextResolver

## ForwardedHeaderTransformer

当服务被反向代理时，中间经过了代理服务器，scheme、host和port等信息可能改变了，会导致客户端指向错误的地址信息，比如重定向时，[RFC7239](https://tools.ietf.org/html/rfc7239)引入了Forwarded相关header（`X-Forwarded-Host`, `X-Forwarded-Port`, `X-Forwarded-Proto`, `X-Forwarded-Ssl`, and `X-Forwarded-Prefix`），记录经过代理转发的相关信息，ForwardedHeaderTransformer会基于这些信息找到正确的地址信息。

## WebHttpHandlerBuilder

流程编排

```java
package org.springframework.web.server.adapter;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.function.Consumer;
import java.util.stream.Collectors;

import org.springframework.beans.factory.NoSuchBeanDefinitionException;
import org.springframework.context.ApplicationContext;
import org.springframework.core.annotation.AnnotationAwareOrderComparator;
import org.springframework.http.codec.ServerCodecConfigurer;
import org.springframework.http.server.reactive.HttpHandler;
import org.springframework.lang.Nullable;
import org.springframework.util.Assert;
import org.springframework.util.ObjectUtils;
import org.springframework.web.server.ServerWebExchange;
import org.springframework.web.server.WebExceptionHandler;
import org.springframework.web.server.WebFilter;
import org.springframework.web.server.WebHandler;
import org.springframework.web.server.handler.ExceptionHandlingWebHandler;
import org.springframework.web.server.handler.FilteringWebHandler;
import org.springframework.web.server.i18n.LocaleContextResolver;
import org.springframework.web.server.session.DefaultWebSessionManager;
import org.springframework.web.server.session.WebSessionManager;

/**
 * This builder has two purposes:
 *
 * <p>One is to assemble a processing chain that consists of a target {@link WebHandler},
 * then decorated with a set of {@link WebFilter WebFilters}, then further decorated with
 * a set of {@link WebExceptionHandler WebExceptionHandlers}.
 *
 * <p>The second purpose is to adapt the resulting processing chain to an {@link HttpHandler}:
 * the lowest-level reactive HTTP handling abstraction which can then be used with any of the
 * supported runtimes. The adaptation is done with the help of {@link HttpWebHandlerAdapter}.
 *
 * <p>The processing chain can be assembled manually via builder methods, or detected from
 * a Spring {@link ApplicationContext} via {@link #applicationContext}, or a mix of both.
 *
 * @author Rossen Stoyanchev
 * @author Sebastien Deleuze
 * @since 5.0
 * @see HttpWebHandlerAdapter
 */
public final class WebHttpHandlerBuilder {

	/** Well-known name for the target WebHandler in the bean factory. */
	public static final String WEB_HANDLER_BEAN_NAME = "webHandler";

	/** Well-known name for the WebSessionManager in the bean factory. */
	public static final String WEB_SESSION_MANAGER_BEAN_NAME = "webSessionManager";

	/** Well-known name for the ServerCodecConfigurer in the bean factory. */
	public static final String SERVER_CODEC_CONFIGURER_BEAN_NAME = "serverCodecConfigurer";

	/** Well-known name for the LocaleContextResolver in the bean factory. */
	public static final String LOCALE_CONTEXT_RESOLVER_BEAN_NAME = "localeContextResolver";

	/** Well-known name for the ForwardedHeaderTransformer in the bean factory. */
	public static final String FORWARDED_HEADER_TRANSFORMER_BEAN_NAME = "forwardedHeaderTransformer";


	private final WebHandler webHandler;

	@Nullable
	private final ApplicationContext applicationContext;

	private final List<WebFilter> filters = new ArrayList<>();

	private final List<WebExceptionHandler> exceptionHandlers = new ArrayList<>();

	@Nullable
	private WebSessionManager sessionManager;

	@Nullable
	private ServerCodecConfigurer codecConfigurer;

	@Nullable
	private LocaleContextResolver localeContextResolver;

	@Nullable
	private ForwardedHeaderTransformer forwardedHeaderTransformer;


	/**
	 * Private constructor to use when initialized from an ApplicationContext.
	 */
	private WebHttpHandlerBuilder(WebHandler webHandler, @Nullable ApplicationContext applicationContext) {
		Assert.notNull(webHandler, "WebHandler must not be null");
		this.webHandler = webHandler;
		this.applicationContext = applicationContext;
	}

	/**
	 * Copy constructor.
	 */
	private WebHttpHandlerBuilder(WebHttpHandlerBuilder other) {
		this.webHandler = other.webHandler;
		this.applicationContext = other.applicationContext;
		this.filters.addAll(other.filters);
		this.exceptionHandlers.addAll(other.exceptionHandlers);
		this.sessionManager = other.sessionManager;
		this.codecConfigurer = other.codecConfigurer;
		this.localeContextResolver = other.localeContextResolver;
		this.forwardedHeaderTransformer = other.forwardedHeaderTransformer;
	}


	/**
	 * Static factory method to create a new builder instance.
	 * @param webHandler the target handler for the request
	 * @return the prepared builder
	 */
	public static WebHttpHandlerBuilder webHandler(WebHandler webHandler) {
		return new WebHttpHandlerBuilder(webHandler, null);
	}

	/**
	 * Static factory method to create a new builder instance by detecting beans
	 * in an {@link ApplicationContext}. The following are detected:
	 * <ul>
	 * <li>{@link WebHandler} [1] -- looked up by the name
	 * {@link #WEB_HANDLER_BEAN_NAME}.
	 * <li>{@link WebFilter} [0..N] -- detected by type and ordered,
	 * see {@link AnnotationAwareOrderComparator}.
	 * <li>{@link WebExceptionHandler} [0..N] -- detected by type and
	 * ordered.
	 * <li>{@link WebSessionManager} [0..1] -- looked up by the name
	 * {@link #WEB_SESSION_MANAGER_BEAN_NAME}.
	 * <li>{@link ServerCodecConfigurer} [0..1] -- looked up by the name
	 * {@link #SERVER_CODEC_CONFIGURER_BEAN_NAME}.
	 * <li>{@link LocaleContextResolver} [0..1] -- looked up by the name
	 * {@link #LOCALE_CONTEXT_RESOLVER_BEAN_NAME}.
	 * </ul>
	 * @param context the application context to use for the lookup
	 * @return the prepared builder
	 */
	public static WebHttpHandlerBuilder applicationContext(ApplicationContext context) {
		WebHttpHandlerBuilder builder = new WebHttpHandlerBuilder(
				context.getBean(WEB_HANDLER_BEAN_NAME, WebHandler.class), context);

		List<WebFilter> webFilters = context
				.getBeanProvider(WebFilter.class)
				.orderedStream()
				.collect(Collectors.toList());
		builder.filters(filters -> filters.addAll(webFilters));
		List<WebExceptionHandler> exceptionHandlers = context
				.getBeanProvider(WebExceptionHandler.class)
				.orderedStream()
				.collect(Collectors.toList());
		builder.exceptionHandlers(handlers -> handlers.addAll(exceptionHandlers));

		try {
			builder.sessionManager(
					context.getBean(WEB_SESSION_MANAGER_BEAN_NAME, WebSessionManager.class));
		}
		catch (NoSuchBeanDefinitionException ex) {
			// Fall back on default
		}

		try {
			builder.codecConfigurer(
					context.getBean(SERVER_CODEC_CONFIGURER_BEAN_NAME, ServerCodecConfigurer.class));
		}
		catch (NoSuchBeanDefinitionException ex) {
			// Fall back on default
		}

		try {
			builder.localeContextResolver(
					context.getBean(LOCALE_CONTEXT_RESOLVER_BEAN_NAME, LocaleContextResolver.class));
		}
		catch (NoSuchBeanDefinitionException ex) {
			// Fall back on default
		}

		try {
			builder.localeContextResolver(
					context.getBean(LOCALE_CONTEXT_RESOLVER_BEAN_NAME, LocaleContextResolver.class));
		}
		catch (NoSuchBeanDefinitionException ex) {
			// Fall back on default
		}

		try {
			builder.forwardedHeaderTransformer(
					context.getBean(FORWARDED_HEADER_TRANSFORMER_BEAN_NAME, ForwardedHeaderTransformer.class));
		}
		catch (NoSuchBeanDefinitionException ex) {
			// Fall back on default
		}

		return builder;
	}


	/**
	 * Add the given filter(s).
	 * @param filters the filter(s) to add that's
	 */
	public WebHttpHandlerBuilder filter(WebFilter... filters) {
		if (!ObjectUtils.isEmpty(filters)) {
			this.filters.addAll(Arrays.asList(filters));
			updateFilters();
		}
		return this;
	}

	/**
	 * Manipulate the "live" list of currently configured filters.
	 * @param consumer the consumer to use
	 */
	public WebHttpHandlerBuilder filters(Consumer<List<WebFilter>> consumer) {
		consumer.accept(this.filters);
		updateFilters();
		return this;
	}

	private void updateFilters() {

		if (this.filters.isEmpty()) {
			return;
		}

		List<WebFilter> filtersToUse = this.filters.stream()
				.peek(filter -> {
					if (filter instanceof ForwardedHeaderTransformer && this.forwardedHeaderTransformer == null) {
						this.forwardedHeaderTransformer = (ForwardedHeaderTransformer) filter;
					}
				})
				.filter(filter -> !(filter instanceof ForwardedHeaderTransformer))
				.collect(Collectors.toList());

		this.filters.clear();
		this.filters.addAll(filtersToUse);
	}

	/**
	 * Add the given exception handler(s).
	 * @param handlers the exception handler(s)
	 */
	public WebHttpHandlerBuilder exceptionHandler(WebExceptionHandler... handlers) {
		if (!ObjectUtils.isEmpty(handlers)) {
			this.exceptionHandlers.addAll(Arrays.asList(handlers));
		}
		return this;
	}

	/**
	 * Manipulate the "live" list of currently configured exception handlers.
	 * @param consumer the consumer to use
	 */
	public WebHttpHandlerBuilder exceptionHandlers(Consumer<List<WebExceptionHandler>> consumer) {
		consumer.accept(this.exceptionHandlers);
		return this;
	}

	/**
	 * Configure the {@link WebSessionManager} to set on the
	 * {@link ServerWebExchange WebServerExchange}.
	 * <p>By default {@link DefaultWebSessionManager} is used.
	 * @param manager the session manager
	 * @see HttpWebHandlerAdapter#setSessionManager(WebSessionManager)
	 */
	public WebHttpHandlerBuilder sessionManager(WebSessionManager manager) {
		this.sessionManager = manager;
		return this;
	}

	/**
	 * Whether a {@code WebSessionManager} is configured or not, either detected from an
	 * {@code ApplicationContext} or explicitly configured via {@link #sessionManager}.
	 * @since 5.0.9
	 */
	public boolean hasSessionManager() {
		return (this.sessionManager != null);
	}

	/**
	 * Configure the {@link ServerCodecConfigurer} to set on the {@code WebServerExchange}.
	 * @param codecConfigurer the codec configurer
	 */
	public WebHttpHandlerBuilder codecConfigurer(ServerCodecConfigurer codecConfigurer) {
		this.codecConfigurer = codecConfigurer;
		return this;
	}


	/**
	 * Whether a {@code ServerCodecConfigurer} is configured or not, either detected from an
	 * {@code ApplicationContext} or explicitly configured via {@link #codecConfigurer}.
	 * @since 5.0.9
	 */
	public boolean hasCodecConfigurer() {
		return (this.codecConfigurer != null);
	}

	/**
	 * Configure the {@link LocaleContextResolver} to set on the
	 * {@link ServerWebExchange WebServerExchange}.
	 * @param localeContextResolver the locale context resolver
	 */
	public WebHttpHandlerBuilder localeContextResolver(LocaleContextResolver localeContextResolver) {
		this.localeContextResolver = localeContextResolver;
		return this;
	}

	/**
	 * Whether a {@code LocaleContextResolver} is configured or not, either detected from an
	 * {@code ApplicationContext} or explicitly configured via {@link #localeContextResolver}.
	 * @since 5.0.9
	 */
	public boolean hasLocaleContextResolver() {
		return (this.localeContextResolver != null);
	}

	/**
	 * Configure the {@link ForwardedHeaderTransformer} for extracting and/or
	 * removing forwarded headers.
	 * @param transformer the transformer
	 * @since 5.1
	 */
	public WebHttpHandlerBuilder forwardedHeaderTransformer(ForwardedHeaderTransformer transformer) {
		this.forwardedHeaderTransformer = transformer;
		return this;
	}

	/**
	 * Whether a {@code ForwardedHeaderTransformer} is configured or not, either
	 * detected from an {@code ApplicationContext} or explicitly configured via
	 * {@link #forwardedHeaderTransformer(ForwardedHeaderTransformer)}.
	 * @since 5.1
	 */
	public boolean hasForwardedHeaderTransformer() {
		return (this.forwardedHeaderTransformer != null);
	}


	/**
	 * Build the {@link HttpHandler}.
	 */
	public HttpHandler build() {

		WebHandler decorated = new FilteringWebHandler(this.webHandler, this.filters);
		decorated = new ExceptionHandlingWebHandler(decorated,  this.exceptionHandlers);

		HttpWebHandlerAdapter adapted = new HttpWebHandlerAdapter(decorated);
		if (this.sessionManager != null) {
			adapted.setSessionManager(this.sessionManager);
		}
		if (this.codecConfigurer != null) {
			adapted.setCodecConfigurer(this.codecConfigurer);
		}
		if (this.localeContextResolver != null) {
			adapted.setLocaleContextResolver(this.localeContextResolver);
		}
		if (this.forwardedHeaderTransformer != null) {
			adapted.setForwardedHeaderTransformer(this.forwardedHeaderTransformer);
		}
		if (this.applicationContext != null) {
			adapted.setApplicationContext(this.applicationContext);
		}
		adapted.afterPropertiesSet();

		return adapted;
	}

	/**
	 * Clone this {@link WebHttpHandlerBuilder}.
	 * @return the cloned builder instance
	 */
	@Override
	public WebHttpHandlerBuilder clone() {
		return new WebHttpHandlerBuilder(this);
	}

}

```



## View

ViewResolutionResultHandler

ViewResolver

UrlBasedViewResolver：redirect

## Codes

## WebClient

### ClientHttpConnector

## Logging

## WebFlux Config

### EnableWebFlux

引入DelegatingWebFluxConfiguration

### WebFluxConfigurationSupport

WebFlux的主要配置类

#### DelegatingWebFluxConfiguration

继承WebFluxConfigurationSupport，扩展以便通过WebFluxConfigurer来定制配置

```java
/**
 * A subclass of {@code WebFluxConfigurationSupport} that detects and delegates
 * to all beans of type {@link WebFluxConfigurer} allowing them to customize the
 * configuration provided by {@code WebFluxConfigurationSupport}. This is the
 * class actually imported by {@link EnableWebFlux @EnableWebFlux}.
 *
 * @author Brian Clozel
 * @since 5.0
 */
@Configuration
public class DelegatingWebFluxConfiguration extends WebFluxConfigurationSupport {

	private final WebFluxConfigurerComposite configurers = new WebFluxConfigurerComposite();

	@Autowired(required = false)
	public void setConfigurers(List<WebFluxConfigurer> configurers) {
		if (!CollectionUtils.isEmpty(configurers)) {
			this.configurers.addWebFluxConfigurers(configurers);
		}
	}
	// 省略其他内容
}
```

### WebFluxConfigurer

WebFlux定制配置，支持N个实例

```java
package org.springframework.web.reactive.config;

import org.springframework.core.convert.converter.Converter;
import org.springframework.format.Formatter;
import org.springframework.format.FormatterRegistry;
import org.springframework.http.codec.ServerCodecConfigurer;
import org.springframework.lang.Nullable;
import org.springframework.validation.MessageCodesResolver;
import org.springframework.validation.Validator;
import org.springframework.web.reactive.accept.RequestedContentTypeResolverBuilder;
import org.springframework.web.reactive.result.method.annotation.ArgumentResolverConfigurer;

/**
 * Defines callback methods to customize the configuration for WebFlux
 * applications enabled via {@link EnableWebFlux @EnableWebFlux}.
 *
 * <p>{@code @EnableWebFlux}-annotated configuration classes may implement
 * this interface to be called back and given a chance to customize the
 * default configuration. Consider implementing this interface and
 * overriding the relevant methods for your needs.
 *
 * @author Brian Clozel
 * @author Rossen Stoyanchev
 * @since 5.0
 * @see WebFluxConfigurationSupport
 * @see DelegatingWebFluxConfiguration
 */
public interface WebFluxConfigurer {

	/**
	 * Configure how the content type requested for the response is resolved
	 * when handling requests with annotated controllers.
	 * @param builder for configuring the resolvers to use
	 */
	default void configureContentTypeResolver(RequestedContentTypeResolverBuilder builder) {
	}

	/**
	 * Configure "global" cross origin request processing.
	 * <p>The configured readers and writers will apply to all requests including
	 * annotated controllers and functional endpoints. Annotated controllers can
	 * further declare more fine-grained configuration via
	 * {@link org.springframework.web.bind.annotation.CrossOrigin @CrossOrigin}.
	 * @see CorsRegistry
	 */
	default void addCorsMappings(CorsRegistry registry) {
	}

	/**
	 * Configure path matching options.
	 * <p>The configured path matching options will be used for mapping to
	 * annotated controllers and also
	 * {@link #addResourceHandlers(ResourceHandlerRegistry) static resources}.
	 * @param configurer the {@link PathMatchConfigurer} instance
	 */
	default void configurePathMatching(PathMatchConfigurer configurer) {
	}

	/**
	 * Add resource handlers for serving static resources.
	 * @see ResourceHandlerRegistry
	 */
	default void addResourceHandlers(ResourceHandlerRegistry registry) {
	}

	/**
	 * Configure resolvers for custom {@code @RequestMapping} method arguments.
	 * @param configurer to configurer to use
	 */
	default void configureArgumentResolvers(ArgumentResolverConfigurer configurer) {
	}

	/**
	 * Configure custom HTTP message readers and writers or override built-in ones.
	 * <p>The configured readers and writers will be used for both annotated
	 * controllers and functional endpoints.
	 * @param configurer the configurer to use
	 */
	default void configureHttpMessageCodecs(ServerCodecConfigurer configurer) {
	}

	/**
	 * Add custom {@link Converter Converters} and {@link Formatter Formatters} for
	 * performing type conversion and formatting of annotated controller method arguments.
	 */
	default void addFormatters(FormatterRegistry registry) {
	}

	/**
	 * Provide a custom {@link Validator}.
	 * <p>By default a validator for standard bean validation is created if
	 * bean validation API is present on the classpath.
	 * <p>The configured validator is used for validating annotated controller
	 * method arguments.
	 */
	@Nullable
	default Validator getValidator() {
		return null;
	}

	/**
	 * Provide a custom {@link MessageCodesResolver} to use for data binding in
	 * annotated controller method arguments instead of the one created by
	 * default in {@link org.springframework.validation.DataBinder}.
	 */
	@Nullable
	default MessageCodesResolver getMessageCodesResolver() {
		return null;
	}

	/**
	 * Configure view resolution for rendering responses with a view and a model,
	 * where the view is typically an HTML template but could also be based on
	 * an HTTP message writer (e.g. JSON, XML).
	 * <p>The configured view resolvers will be used for both annotated
	 * controllers and functional endpoints.
	 */
	default void configureViewResolvers(ViewResolverRegistry registry) {
	}

}
```



## 业务层

### Annotated Controller

### Functional Handler

#### RouterFunction

#### HandlerFunction



## ResourceHandler

ResourceHandlerRegistrationCustomizer

## SSE

```java
public static Mono<ServerResponse> sendTimePerSec(ServerRequest serverRequest) {
    return ok().contentType(MediaType.TEXT_EVENT_STREAM).body(
            Flux.interval(Duration.ofSeconds(1)).
                    map(l -> new SimpleDateFormat("HH:mm:ss").format(new Date())), String.class);
}
```



## 不断上送

```java
    @Test
    public void webClientTest4() {
        Flux<MyEvent> eventFlux = Flux.interval(Duration.ofSeconds(1))
                .map(l -> new MyEvent(System.currentTimeMillis(), "message-" + l)).take(5);
        WebClient webClient = WebClient.create("http://localhost:8080");
        webClient
                .post().uri("/events")
                .contentType(MediaType.APPLICATION_STREAM_JSON)
                .body(eventFlux, MyEvent.class)
                .retrieve()
                .bodyToMono(Void.class)
                .block();
    }
```



## 参考

[Web关于反应堆栈 5.0.0.RELEASE 中文版](https://www.springcloud.cc/web-reactive.html)

[web reactive 5.1.3.RELEASE 中文版](https://www.docs4dev.com/docs/zh/spring-framework/5.1.3.RELEASE/reference/web-reactive.html)