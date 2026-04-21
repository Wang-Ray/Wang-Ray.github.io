---
layout: post
title: "实践CORS（Cross Origin Resource Share）"
categories: secure
tags: SOP "Cross Origin Resource Share" CORS CSRF
---

[**Cross-Origin Resource Sharing** ](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS) is an [HTTP](https://developer.mozilla.org/en-US/docs/Glossary/HTTP)-header based mechanism that allows a server to indicate any [origins](https://developer.mozilla.org/en-US/docs/Glossary/Origin) (domain, scheme, or port) other than its own from which a browser should permit loading resources. CORS also relies on a mechanism by which browsers make a "preflight" request to the server hosting the cross-origin resource, in order to check that the server will permit the actual request. In that preflight, the browser sends headers that indicate the HTTP method and headers that will be used in the actual request.