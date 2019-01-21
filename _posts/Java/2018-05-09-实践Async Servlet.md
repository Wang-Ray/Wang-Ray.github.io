---
layout: post
title: "实践Async Servlet"
date: 2018-05-09 11:08:00 +0800
categories: Java
tags: java Async-Servlet Servlet
---

Servlet3.0新增异步Servlet，将Servlet线程和业务线程分离。

## 样例

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

## AsyncContext

`javax.servlet.ServletRequest#startAsync()`

### setTimeout

### start

### complete

## AsyncListner

```java
public interface AsyncListener extends EventListener {
    void onComplete(AsyncEvent event) throws IOException;
    void onTimeout(AsyncEvent event) throws IOException;
    void onError(AsyncEvent event) throws IOException;
    void onStartAsync(AsyncEvent event) throws IOException;
}
```

