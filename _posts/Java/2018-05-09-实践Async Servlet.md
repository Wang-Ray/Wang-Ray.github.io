---
layout: post
title: "实践Async Servlet"
date: 2018-05-09 11:08:00 +0800
categories: Java
tags: java Async-Servlet Servlet
---

Servlet3.0新增异步Servlet，将Servlet线程和业务线程分离。

## 样例

worker线程执行业务逻辑：

```java
@WebServlet(name = "asyncServlet", urlPatterns = "/asyncServlet", asyncSupported = true)
public class AsyncServlet extends HttpServlet {

	Logger logger = LoggerFactory.getLogger(AsyncServlet.class);

	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		logger.info("doGet begin");

		try {
			TimeUnit.SECONDS.sleep(3);
		} catch (Exception e) {
			e.printStackTrace();
		}

		final AsyncContext asyncContext = req.startAsync();
		asyncContext.setTimeout(30 * 1000);
      	// 另开worker线程进行回调处理
		asyncContext.addListener(new AsyncListener() {
			public void onComplete(AsyncEvent event) throws IOException {
				logger.info("onComplete begin");
				try {
					TimeUnit.SECONDS.sleep(3);
				} catch (Exception e) {
					e.printStackTrace();
				}
				logger.info("onComplete end");
			}

			public void onTimeout(AsyncEvent event) throws IOException {
				logger.info("onTimeout");
			}

			public void onError(AsyncEvent event) throws IOException {
				logger.info("onError");
			}

			public void onStartAsync(AsyncEvent event) throws IOException {
				logger.info("onStartAsync");
			}
		});
      	// 另开worker线程进行业务处理
		asyncContext.start(new Runnable() {
			public void run() {
				logger.info("biz begin");
				PrintWriter printWriter = null;
				try {
					TimeUnit.SECONDS.sleep(20);
					printWriter = asyncContext.getResponse().getWriter();
				} catch (Exception e) {
					e.printStackTrace();
				}
				printWriter.println("Hello, AsyncServlet.");
				printWriter.flush();
				printWriter.close();
				asyncContext.complete();
				logger.info("biz end");
			}
		});
		logger.info("doGet end");
	}
}
```

收到一笔请求时日志输出：

```
2018-05-09 09:12:06.867 INFO [http-bio-8080-exec-1] servlet.AsyncServlet:31 >> doGet begin
2018-05-09 09:12:09.877 INFO [http-bio-8080-exec-1] servlet.AsyncServlet:81 >> doGet end
2018-05-09 09:12:09.877 INFO [http-bio-8080-exec-2] servlet.AsyncServlet$2:66 >> biz begin
2018-05-09 09:12:29.887 INFO [http-bio-8080-exec-2] servlet.AsyncServlet$2:78 >> biz end
2018-05-09 09:12:29.888 INFO [http-bio-8080-exec-3] servlet.AsyncServlet$1:43 >> onComplete begin
2018-05-09 09:12:32.889 INFO [http-bio-8080-exec-3] servlet.AsyncServlet$1:49 >> onComplete end
```

（设定worker线程池大小为3的情况下）连续收到两笔请求时线程栈如下：

![AsyncServlet](/images/AsyncServlet.png)

可见worker线程在AysncServlet下不是一直占用至Servlet结束，这样一个worker线程可以服务于多个请求。

另起业务线程执行业务逻辑：

```java
@WebServlet(name = "asyncServletExecutorService", urlPatterns = "/asyncServletExecutorService", asyncSupported = true)
public class AsyncServletExecutorService extends HttpServlet {

    Logger logger = LoggerFactory.getLogger(AsyncServletExecutorService.class);

    private final ExecutorService executorService = Executors.newCachedThreadPool();

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        logger.info("doGet begin");

        try {
            TimeUnit.SECONDS.sleep(3);
        } catch (Exception e) {
            e.printStackTrace();
        }

        final AsyncContext asyncContext = req.startAsync();
        asyncContext.setTimeout(30 * 1000);
        // 另开worker线程进行回调处理
        asyncContext.addListener(new AsyncListener() {
            public void onComplete(AsyncEvent event) throws IOException {
                logger.info("onComplete begin");
                try {
                    TimeUnit.SECONDS.sleep(3);
                } catch (Exception e) {
                    e.printStackTrace();
                }
                logger.info("onComplete end");
            }

            public void onTimeout(AsyncEvent event) throws IOException {
                logger.info("onTimeout");
            }

            public void onError(AsyncEvent event) throws IOException {
                logger.info("onError");
            }

            public void onStartAsync(AsyncEvent event) throws IOException {
                logger.info("onStartAsync");
            }
        });
        // 另开业务线程进行业务处理
        executorService.execute(new Runnable() {
            public void run() {
                logger.info("biz begin");
                PrintWriter printWriter = null;
                try {
                    TimeUnit.SECONDS.sleep(20);
                    printWriter = asyncContext.getResponse().getWriter();
                } catch (Exception e) {
                    e.printStackTrace();
                }
                printWriter.println("Hello, AsyncServlet.");
                printWriter.flush();
                printWriter.close();
                asyncContext.complete();
                logger.info("biz end");
            }
        });
        logger.info("doGet end");
    }
}
```



## AsyncContext

```java
package javax.servlet;

/**
 * Class representing the execution context for an asynchronous operation
 * that was initiated on a ServletRequest.
 *
 * <p>An AsyncContext is created and initialized by a call to
 * {@link ServletRequest#startAsync()} or
 * {@link ServletRequest#startAsync(ServletRequest, ServletResponse)}.
 * Repeated invocations of these methods will return the same AsyncContext
 * instance, reinitialized as appropriate.
 *
 * <p>In the event that an asynchronous operation has timed out, the
 * container must run through these steps:
 * <ol>
 * <li>Invoke, at their {@link AsyncListener#onTimeout onTimeout} method, all
 * {@link AsyncListener} instances registered with the ServletRequest
 * on which the asynchronous operation was initiated.</li>
 * <li>If none of the listeners called {@link #complete} or any of the
 * {@link #dispatch} methods, perform an error dispatch with a status code
 * equal to <tt>HttpServletResponse.SC_INTERNAL_SERVER_ERROR</tt>.</li>
 * <li>If no matching error page was found, or the error page did not call
 * {@link #complete} or any of the {@link #dispatch} methods, call
 * {@link #complete}.</li>
 * </ol>
 *
 * @since Servlet 3.0
 */
public interface AsyncContext {

    /**
     * The name of the request attribute under which the original
     * request URI is made available to the target of a
     * {@link #dispatch(String)} or {@link #dispatch(ServletContext,String)} 
     */
    static final String ASYNC_REQUEST_URI = "javax.servlet.async.request_uri";

    /**
     * The name of the request attribute under which the original
     * context path is made available to the target of a
     * {@link #dispatch(String)} or {@link #dispatch(ServletContext,String)} 
     */
    static final String ASYNC_CONTEXT_PATH = "javax.servlet.async.context_path";

    /**
     * The name of the request attribute under which the original
     * path info is made available to the target of a
     * {@link #dispatch(String)} or {@link #dispatch(ServletContext,String)} 
     */
    static final String ASYNC_PATH_INFO = "javax.servlet.async.path_info";

    /**
     * The name of the request attribute under which the original
     * servlet path is made available to the target of a
     * {@link #dispatch(String)} or {@link #dispatch(ServletContext,String)}  
     */
    static final String ASYNC_SERVLET_PATH = "javax.servlet.async.servlet_path";

    /**
     * The name of the request attribute under which the original
     * query string is made available to the target of a
     * {@link #dispatch(String)} or {@link #dispatch(ServletContext,String)} 
     */
    static final String ASYNC_QUERY_STRING = "javax.servlet.async.query_string";


    /**
     * Gets the request that was used to initialize this AsyncContext
     * by calling {@link ServletRequest#startAsync()} or
     * {@link ServletRequest#startAsync(ServletRequest, ServletResponse)}.
     *
     * @return the request that was used to initialize this AsyncContext
     *
     * @exception IllegalStateException  if {@link #complete} or any of the
     *                                  {@link #dispatch} methods has been
     *                                  called in the asynchronous cycle
     */
    public ServletRequest getRequest();


    /**
     * Gets the response that was used to initialize this AsyncContext
     * by calling {@link ServletRequest#startAsync()} or
     * {@link ServletRequest#startAsync(ServletRequest, ServletResponse)}.
     *
     * @return the response that was used to initialize this AsyncContext
     *
     * @exception IllegalStateException  if {@link #complete} or any of the
     *                                  {@link #dispatch} methods has been
     *                                  called in the asynchronous cycle
     */
    public ServletResponse getResponse();


    /**
     * Checks if this AsyncContext was initialized with the original or
     * application-wrapped request and response objects.
     * 
     * <p>This information may be used by filters invoked in the
     * <i>outbound</i> direction, after a request was put into
     * asynchronous mode, to determine whether any request and/or response
     * wrappers that they added during their <i>inbound</i> invocation need
     * to be preserved for the duration of the asynchronous operation, or may
     * be released.
     *
     * @return true if this AsyncContext was initialized with the original
     * request and response objects by calling
     * {@link ServletRequest#startAsync()}, or if it was initialized by
     * calling
     * {@link ServletRequest#startAsync(ServletRequest, ServletResponse)},
     * and neither the ServletRequest nor ServletResponse arguments 
     * carried any application-provided wrappers; false otherwise
     */
    public boolean hasOriginalRequestAndResponse();


    /**
     * Dispatches the request and response objects of this AsyncContext
     * to the servlet container.
     * 
     * <p>If the asynchronous cycle was started with
     * {@link ServletRequest#startAsync(ServletRequest, ServletResponse)},
     * and the request passed is an instance of HttpServletRequest,
     * then the dispatch is to the URI returned by
     * {@link javax.servlet.http.HttpServletRequest#getRequestURI}.
     * Otherwise, the dispatch is to the URI of the request when it was
     * last dispatched by the container.
     *
     * <p>The following sequence illustrates how this will work:
     * <code><pre>
     * // REQUEST dispatch to /url/A
     * AsyncContext ac = request.startAsync();
     * ...
     * ac.dispatch(); // ASYNC dispatch to /url/A
     * 
     * // REQUEST to /url/A
     * // FORWARD dispatch to /url/B
     * request.getRequestDispatcher("/url/B").forward(request,response);
     * // Start async operation from within the target of the FORWARD
     * // dispatch
     * ac = request.startAsync();
     * ...
     * ac.dispatch(); // ASYNC dispatch to /url/A
     * 
     * // REQUEST to /url/A
     * // FORWARD dispatch to /url/B
     * request.getRequestDispatcher("/url/B").forward(request,response);
     * // Start async operation from within the target of the FORWARD
     * // dispatch
     * ac = request.startAsync(request,response);
     * ...
     * ac.dispatch(); // ASYNC dispatch to /url/B
     * </pre></code>
     *
     * <p>This method returns immediately after passing the request
     * and response objects to a container managed thread, on which the
     * dispatch operation will be performed.
     * If this method is called before the container-initiated dispatch
     * that called <tt>startAsync</tt> has returned to the container, the
     * dispatch operation will be delayed until after the container-initiated
     * dispatch has returned to the container.
     *
     * <p>The dispatcher type of the request is set to
     * <tt>DispatcherType.ASYNC</tt>. Unlike
     * {@link RequestDispatcher#forward(ServletRequest, ServletResponse)
     * forward dispatches}, the response buffer and
     * headers will not be reset, and it is legal to dispatch even if the
     * response has already been committed.
     *
     * <p>Control over the request and response is delegated
     * to the dispatch target, and the response will be closed when the
     * dispatch target has completed execution, unless
     * {@link ServletRequest#startAsync()} or
     * {@link ServletRequest#startAsync(ServletRequest, ServletResponse)}
     * are called.
     * 
     * <p>Any errors or exceptions that may occur during the execution
     * of this method must be caught and handled by the container, as
     * follows:
     * <ol>
     * <li>Invoke, at their {@link AsyncListener#onError onError} method, all
     * {@link AsyncListener} instances registered with the ServletRequest
     * for which this AsyncContext was created, and make the caught 
     * <tt>Throwable</tt> available via {@link AsyncEvent#getThrowable}.</li>
     * <li>If none of the listeners called {@link #complete} or any of the
     * {@link #dispatch} methods, perform an error dispatch with a status code
     * equal to <tt>HttpServletResponse.SC_INTERNAL_SERVER_ERROR</tt>, and
     * make the above <tt>Throwable</tt> available as the value of the
     * <tt>RequestDispatcher.ERROR_EXCEPTION</tt> request attribute.</li>
     * <li>If no matching error page was found, or the error page did not call
     * {@link #complete} or any of the {@link #dispatch} methods, call
     * {@link #complete}.</li>
     * </ol>
     *
     * <p>There can be at most one asynchronous dispatch operation per
     * asynchronous cycle, which is started by a call to one of the
     * {@link ServletRequest#startAsync} methods. Any attempt to perform an
     * additional asynchronous dispatch operation within the same
     * asynchronous cycle will result in an IllegalStateException.
     * If startAsync is subsequently called on the dispatched request,
     * then any of the dispatch or {@link #complete} methods may be called.
     *
     * @throws IllegalStateException if one of the dispatch methods
     * has been called and the startAsync method has not been
     * called during the resulting dispatch, or if {@link #complete}
     * was called
     *
     * @see ServletRequest#getDispatcherType
     */
    public void dispatch();


    /**
     * Dispatches the request and response objects of this AsyncContext
     * to the given <tt>path</tt>.
     *
     * <p>The <tt>path</tt> parameter is interpreted in the same way 
     * as in {@link ServletRequest#getRequestDispatcher(String)}, within
     * the scope of the {@link ServletContext} from which this
     * AsyncContext was initialized.
     *
     * <p>All path related query methods of the request must reflect the
     * dispatch target, while the original request URI, context path,
     * path info, servlet path, and query string may be recovered from
     * the {@link #ASYNC_REQUEST_URI}, {@link #ASYNC_CONTEXT_PATH},
     * {@link #ASYNC_PATH_INFO}, {@link #ASYNC_SERVLET_PATH}, and
     * {@link #ASYNC_QUERY_STRING} attributes of the request. These
     * attributes will always reflect the original path elements, even under
     * repeated dispatches.
     *
     * <p>There can be at most one asynchronous dispatch operation per
     * asynchronous cycle, which is started by a call to one of the
     * {@link ServletRequest#startAsync} methods. Any attempt to perform an
     * additional asynchronous dispatch operation within the same
     * asynchronous cycle will result in an IllegalStateException.
     * If startAsync is subsequently called on the dispatched request,
     * then any of the dispatch or {@link #complete} methods may be called.
     *
     * <p>See {@link #dispatch()} for additional details, including error
     * handling.
     *
     * @param path the path of the dispatch target, scoped to the
     * ServletContext from which this AsyncContext was initialized
     *
     * @throws IllegalStateException if one of the dispatch methods
     * has been called and the startAsync method has not been
     * called during the resulting dispatch, or if {@link #complete}
     * was called
     *
     * @see ServletRequest#getDispatcherType
     */
    public void dispatch(String path);


    /**
     * Dispatches the request and response objects of this AsyncContext
     * to the given <tt>path</tt> scoped to the given <tt>context</tt>.
     *
     * <p>The <tt>path</tt> parameter is interpreted in the same way 
     * as in {@link ServletRequest#getRequestDispatcher(String)}, except that
     * it is scoped to the given <tt>context</tt>.
     *
     * <p>All path related query methods of the request must reflect the
     * dispatch target, while the original request URI, context path,
     * path info, servlet path, and query string may be recovered from
     * the {@link #ASYNC_REQUEST_URI}, {@link #ASYNC_CONTEXT_PATH},
     * {@link #ASYNC_PATH_INFO}, {@link #ASYNC_SERVLET_PATH}, and
     * {@link #ASYNC_QUERY_STRING} attributes of the request. These
     * attributes will always reflect the original path elements, even under
     * repeated dispatches.
     *
     * <p>There can be at most one asynchronous dispatch operation per
     * asynchronous cycle, which is started by a call to one of the
     * {@link ServletRequest#startAsync} methods. Any attempt to perform an
     * additional asynchronous dispatch operation within the same
     * asynchronous cycle will result in an IllegalStateException.
     * If startAsync is subsequently called on the dispatched request,
     * then any of the dispatch or {@link #complete} methods may be called.
     *
     * <p>See {@link #dispatch()} for additional details, including error
     * handling.
     *
     * @param context the ServletContext of the dispatch target
     * @param path the path of the dispatch target, scoped to the given
     * ServletContext
     *
     * @throws IllegalStateException if one of the dispatch methods
     * has been called and the startAsync method has not been
     * called during the resulting dispatch, or if {@link #complete}
     * was called
     *
     * @see ServletRequest#getDispatcherType
     */
    public void dispatch(ServletContext context, String path);


    /**
     * Completes the asynchronous operation that was started on the request
     * that was used to initialze this AsyncContext, closing the response
     * that was used to initialize this AsyncContext.
     *
     * <p>Any listeners of type {@link AsyncListener} that were registered
     * with the ServletRequest for which this AsyncContext was created will
     * be invoked at their {@link AsyncListener#onComplete(AsyncEvent)
     * onComplete} method.
     *
     * <p>It is legal to call this method any time after a call to
     * {@link ServletRequest#startAsync()} or
     * {@link ServletRequest#startAsync(ServletRequest, ServletResponse)},
     * and before a call to one of the <tt>dispatch</tt> methods
     * of this class. 
     * If this method is called before the container-initiated dispatch
     * that called <tt>startAsync</tt> has returned to the container, then
     * the call will not take effect (and any invocations of
     * {@link AsyncListener#onComplete(AsyncEvent)} will be delayed) until
     * after the container-initiated dispatch has returned to the container.
     */
    public void complete();


    /**
     * Causes the container to dispatch a thread, possibly from a managed
     * thread pool, to run the specified <tt>Runnable</tt>. The container may
     * propagate appropriate contextual information to the <tt>Runnable</tt>. 
     *
     * @param run the asynchronous handler
     */
    public void start(Runnable run);


    /**
     * Registers the given {@link AsyncListener} with the most recent
     * asynchronous cycle that was started by a call to one of the
     * {@link ServletRequest#startAsync} methods.
     *
     * <p>The given AsyncListener will receive an {@link AsyncEvent} when
     * the asynchronous cycle completes successfully, times out, results
     * in an error, or a new asynchronous cycle is being initiated via
     * one of the {@link ServletRequest#startAsync} methods.
     *
     * <p>AsyncListener instances will be notified in the order in which
     * they were added.
     *
     * <p>If {@link ServletRequest#startAsync(ServletRequest, ServletResponse)}
     * or {@link ServletRequest#startAsync} is called,
     * the exact same request and response objects are available from the
     * {@link AsyncEvent} when the {@link AsyncListener} is notified.
     *
     * @param listener the AsyncListener to be registered
     * 
     * @throws IllegalStateException if this method is called after
     * the container-initiated dispatch, during which one of the
     * {@link ServletRequest#startAsync} methods was called, has
     * returned to the container
     */
    public void addListener(AsyncListener listener);


    /**
     * Registers the given {@link AsyncListener} with the most recent
     * asynchronous cycle that was started by a call to one of the
     * {@link ServletRequest#startAsync} methods.
     *
     * <p>The given AsyncListener will receive an {@link AsyncEvent} when
     * the asynchronous cycle completes successfully, times out, results
     * in an error, or a new asynchronous cycle is being initiated via
     * one of the {@link ServletRequest#startAsync} methods.
     *
     * <p>AsyncListener instances will be notified in the order in which
     * they were added.
     *
     * <p>The given ServletRequest and ServletResponse objects will
     * be made available to the given AsyncListener via the
     * {@link AsyncEvent#getSuppliedRequest getSuppliedRequest} and
     * {@link AsyncEvent#getSuppliedResponse getSuppliedResponse} methods,
     * respectively, of the {@link AsyncEvent} delivered to it. These objects
     * should not be read from or written to, respectively, at the time the
     * AsyncEvent is delivered, because additional wrapping may have
     * occurred since the given AsyncListener was registered, but may be used
     * in order to release any resources associated with them.
     *
     * @param listener the AsyncListener to be registered
     * @param servletRequest the ServletRequest that will be included
     * in the AsyncEvent
     * @param servletResponse the ServletResponse that will be included
     * in the AsyncEvent
     *
     * @throws IllegalStateException if this method is called after
     * the container-initiated dispatch, during which one of the
     * {@link ServletRequest#startAsync} methods was called, has
     * returned to the container
     */
    public void addListener(AsyncListener listener,
                            ServletRequest servletRequest,
                            ServletResponse servletResponse);


    /**
     * Instantiates the given {@link AsyncListener} class.
     *
     * <p>The returned AsyncListener instance may be further customized
     * before it is registered with this AsyncContext via a call to one of 
     * the <code>addListener</code> methods.
     *
     * <p>The given AsyncListener class must define a zero argument
     * constructor, which is used to instantiate it.
     *
     * <p>This method supports resource injection if the given
     * <tt>clazz</tt> represents a Managed Bean.
     * See the Java EE platform and JSR 299 specifications for additional
     * details about Managed Beans and resource injection.

     * <p>This method supports any annotations applicable to AsyncListener.
     *
     * @param clazz the AsyncListener class to instantiate
     *
     * @return the new AsyncListener instance
     *
     * @throws ServletException if the given <tt>clazz</tt> fails to be
     * instantiated
     */
    public <T extends AsyncListener> T createListener(Class<T> clazz)
        throws ServletException; 


    /**
     * Sets the timeout (in milliseconds) for this AsyncContext.
     *
     * <p>The timeout applies to this AsyncContext once the
     * container-initiated dispatch during which one of the
     * {@link ServletRequest#startAsync} methods was called has
     * returned to the container. 
     *
     * <p>The timeout will expire if neither the {@link #complete} method
     * nor any of the dispatch methods are called. A timeout value of
     * zero or less indicates no timeout. 
     * 
     * <p>If {@link #setTimeout} is not called, then the container's
     * default timeout, which is available via a call to
     * {@link #getTimeout}, will apply.
     *
     * <p>The default value is <code>30000</code> ms.
     *
     * @param timeout the timeout in milliseconds
     *
     * @throws IllegalStateException if this method is called after
     * the container-initiated dispatch, during which one of the
     * {@link ServletRequest#startAsync} methods was called, has
     * returned to the container
     */
    public void setTimeout(long timeout);


    /**
     * Gets the timeout (in milliseconds) for this AsyncContext.
     *
     * <p>This method returns the container's default timeout for
     * asynchronous operations, or the timeout value passed to the most
     * recent invocation of {@link #setTimeout}.
     *
     * <p>A timeout value of zero or less indicates no timeout.
     *
     * @return the timeout in milliseconds
     */
    public long getTimeout();

}
```

### startAsync

`javax.servlet.ServletRequest#startAsync()`

### setTimeout

### start

### complete

## AsyncListner

```java
package javax.servlet;

import java.io.IOException;
import java.util.EventListener;

/**
 * Listener that will be notified in the event that an asynchronous
 * operation initiated on a ServletRequest to which the listener had been 
 * added has completed, timed out, or resulted in an error.
 *
 * @since Servlet 3.0
 */
public interface AsyncListener extends EventListener {
    
    /**
     * Notifies this AsyncListener that an asynchronous operation
     * has been completed.
     * 
     * <p>The {@link AsyncContext} corresponding to the asynchronous
     * operation that has been completed may be obtained by calling
     * {@link AsyncEvent#getAsyncContext getAsyncContext} on the given
     * <tt>event</tt>.
     *
     * <p>In addition, if this AsyncListener had been registered via a call
     * to {@link AsyncContext#addListener(AsyncListener,
     * ServletRequest, ServletResponse)}, the supplied ServletRequest and
     * ServletResponse objects may be retrieved by calling
     * {@link AsyncEvent#getSuppliedRequest getSuppliedRequest} and
     * {@link AsyncEvent#getSuppliedResponse getSuppliedResponse},
     * respectively, on the given <tt>event</tt>.
     *
     * @param event the AsyncEvent indicating that an asynchronous
     * operation has been completed
     *
     * @throws IOException if an I/O related error has occurred during the
     * processing of the given AsyncEvent
     */
    public void onComplete(AsyncEvent event) throws IOException;


    /**
     * Notifies this AsyncListener that an asynchronous operation
     * has timed out.
     * 
     * <p>The {@link AsyncContext} corresponding to the asynchronous
     * operation that has timed out may be obtained by calling
     * {@link AsyncEvent#getAsyncContext getAsyncContext} on the given
     * <tt>event</tt>.
     *
     * <p>In addition, if this AsyncListener had been registered via a call
     * to {@link AsyncContext#addListener(AsyncListener,
     * ServletRequest, ServletResponse)}, the supplied ServletRequest and
     * ServletResponse objects may be retrieved by calling
     * {@link AsyncEvent#getSuppliedRequest getSuppliedRequest} and
     * {@link AsyncEvent#getSuppliedResponse getSuppliedResponse},
     * respectively, on the given <tt>event</tt>.
     *
     * @param event the AsyncEvent indicating that an asynchronous
     * operation has timed out
     *
     * @throws IOException if an I/O related error has occurred during the
     * processing of the given AsyncEvent
     */
    public void onTimeout(AsyncEvent event) throws IOException;


    /**
     * Notifies this AsyncListener that an asynchronous operation 
     * has failed to complete.
     * 
     * <p>The {@link AsyncContext} corresponding to the asynchronous
     * operation that failed to complete may be obtained by calling
     * {@link AsyncEvent#getAsyncContext getAsyncContext} on the given
     * <tt>event</tt>.
     * 
     * <p>In addition, if this AsyncListener had been registered via a call
     * to {@link AsyncContext#addListener(AsyncListener,
     * ServletRequest, ServletResponse)}, the supplied ServletRequest and
     * ServletResponse objects may be retrieved by calling
     * {@link AsyncEvent#getSuppliedRequest getSuppliedRequest} and
     * {@link AsyncEvent#getSuppliedResponse getSuppliedResponse},
     * respectively, on the given <tt>event</tt>.
     *
     * @param event the AsyncEvent indicating that an asynchronous
     * operation has failed to complete
     *
     * @throws IOException if an I/O related error has occurred during the
     * processing of the given AsyncEvent
     */
    public void onError(AsyncEvent event) throws IOException;


    /**
     * Notifies this AsyncListener that a new asynchronous cycle is being
     * initiated via a call to one of the {@link ServletRequest#startAsync}
     * methods.
     *
     * <p>The {@link AsyncContext} corresponding to the asynchronous
     * operation that is being reinitialized may be obtained by calling
     * {@link AsyncEvent#getAsyncContext getAsyncContext} on the given
     * <tt>event</tt>.
     * 
     * <p>In addition, if this AsyncListener had been registered via a call
     * to {@link AsyncContext#addListener(AsyncListener,
     * ServletRequest, ServletResponse)}, the supplied ServletRequest and
     * ServletResponse objects may be retrieved by calling
     * {@link AsyncEvent#getSuppliedRequest getSuppliedRequest} and
     * {@link AsyncEvent#getSuppliedResponse getSuppliedResponse},
     * respectively, on the given <tt>event</tt>.
     *
     * <p>This AsyncListener will not receive any events related to the
     * new asynchronous cycle unless it registers itself (via a call
     * to {@link AsyncContext#addListener}) with the AsyncContext that
     * is delivered as part of the given AsyncEvent.
     *
     * @param event the AsyncEvent indicating that a new asynchronous
     * cycle is being initiated
     *
     * @throws IOException if an I/O related error has occurred during the
     * processing of the given AsyncEvent
     */
    public void onStartAsync(AsyncEvent event) throws IOException;     

}
```



## Servlet 3.1提供的非阻塞IO能力

### ReadListener

```java
package javax.servlet;

import java.io.IOException;

/**
 * Receives notification of read events when using non-blocking IO.
 *
 * @since Servlet 3.1
 */
public interface ReadListener extends java.util.EventListener{

    /**
     * Invoked when data is available to read. The container will invoke this
     * method the first time for a request as soon as there is data to read.
     * Subsequent invocations will only occur if a call to
     * {@link ServletInputStream#isReady()} has returned false and data has
     * subsequently become available to read.
     *
     * @throws IOException id an I/O error occurs while processing the event
     */
    public abstract void onDataAvailable() throws IOException;

    /**
     * Invoked when the request body has been fully read.
     *
     * @throws IOException id an I/O error occurs while processing the event
     */
    public abstract void onAllDataRead() throws IOException;

    /**
     * Invoked if an error occurs while reading the request body.
     *
     * @param throwable The exception that occurred
     */
    public abstract void onError(java.lang.Throwable throwable);
}
```

`javax.servlet.ServletInputStream#isReady`

`curl --limit-rate 5k -F 'file=@bigfile' http://localhost:8080/async-read`

```java
"http-nio-8080-exec-1@5471" daemon prio=5 tid=0x1a nid=NA runnable
  java.lang.Thread.State: RUNNABLE
	  at wang.ray.sample.asyncservlet2.web.nio3.AsyncReadServlet$1.onDataAvailable(AsyncReadServlet.java:36)
	  at org.apache.catalina.connector.CoyoteAdapter.asyncDispatch(CoyoteAdapter.java:210)
	  at org.apache.coyote.AbstractProcessor.dispatch(AbstractProcessor.java:241)
	  at org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:49)
	  at org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:791)
	  at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1417)
	  at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49)
	  - locked <0x165b> (a org.apache.tomcat.util.net.NioEndpoint$NioSocketWrapper)
	  at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	  at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	  at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
	  at java.lang.Thread.run(Thread.java:748)

```



### WriteListener

```java
package javax.servlet;

import java.io.IOException;

/**
 * Receives notification of write events when using non-blocking IO.
 *
 * @since Servlet 3.1
 */
public interface WriteListener extends java.util.EventListener{

    /**
     * Invoked when it it possible to write data without blocking. The container
     * will invoke this method the first time for a request as soon as data can
     * be written. Subsequent invocations will only occur if a call to
     * {@link ServletOutputStream#isReady()} has returned false and it has since
     * become possible to write data.
     *
     * @throws IOException if an I/O error occurs while processing this event
     */
    public void onWritePossible() throws IOException;

    /**
     * Invoked if an error occurs while writing the response.
     *
     * @param throwable The throwable that represents the error that occurred
     */
    public void onError(java.lang.Throwable throwable);
}
```

`javax.servlet.ServletOutputStream#isReady`



`curl --limit-rate 200k -o bigfile http://localhost:8080/async-write`



```java
"http-nio-8080-exec-1@5470" daemon prio=5 tid=0x19 nid=NA runnable
  java.lang.Thread.State: RUNNABLE
	  at wang.ray.sample.asyncservlet2.web.nio3.AsyncWriteServlet$1.onWritePossible(AsyncWriteServlet.java:36)
	  at org.apache.coyote.Response.onWritePossible(Response.java:760)
	  at org.apache.catalina.connector.CoyoteAdapter.asyncDispatch(CoyoteAdapter.java:188)
	  at org.apache.coyote.AbstractProcessor.dispatch(AbstractProcessor.java:241)
	  at org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:49)
	  at org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:791)
	  at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1417)
	  at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49)
	  - locked <0x172f> (a org.apache.tomcat.util.net.NioEndpoint$NioSocketWrapper)
	  at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	  at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	  at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
	  at java.lang.Thread.run(Thread.java:748)
```

