---
layout: post
title: "实践Dubbo之Validation"
categories: Java
tags: Java Dubbo Validation
---

## 关键字

### ValidationException

### ConstraintViolationException



org.springframework.validation.annotation.Validated

javax.validation.Valid



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



