---
layout: post
title: "实践Spring之Validation"
categories: Java
tags: Java Dubbo Validation
---

## 关键字

### ValidationException

### ConstraintViolationException



org.springframework.validation.annotation.Validated

javax.validation.Valid



```java
import java.util.concurrent.TimeUnit;

import javax.validation.Valid;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Service;
import org.springframework.validation.annotation.Validated;

import wang.ray.dubbo.sample.BookRequest;
import wang.ray.dubbo.sample.BookResponse;
import wang.ray.dubbo.spring.boot.starter.sample.provider.service.SampleService;

@Service
@Validated
public class SampleServiceImpl implements SampleService {

    Logger logger = LoggerFactory.getLogger(SampleServiceImpl.class);

    @Override
    public BookResponse getBook(@Valid BookRequest bookRequest) {
        logger.info("BookRequest: {}", bookRequest);
        BookResponse bookResponse = new BookResponse();
        bookResponse.setId(bookRequest.getId());
        bookResponse.setName(bookRequest.getName());
        bookResponse.setResult("answer");
        bookResponse.setPageList(bookRequest.getPageList());
        return bookResponse;
    }

}
```







```java
try {
// 需要参数校验的方法
} catch (ConstraintViolationException cve) {
	String message = "";
	Set<ConstraintViolation<?>> constraintViolations = cve.getConstraintViolations();
	for (ConstraintViolation<?> constraint : constraintViolations) {
		message = constraint.getMessage();
		break;
    }
	logger.error(message);
}
```



