---
layout: post
title: "实践Sentinel之sentinel-extension"
date: 2019-11-22 11:08:00 +0800
categories: Java
tags: java Sentinel flow-control circuit-breaking
---

## ReadableDataSource

```java
/**
 * The readable data source is responsible for retrieving configs (read-only).
 *
 * @param <S> source data type
 * @param <T> target data type
 * @author leyou
 * @author Eric Zhao
 */
public interface ReadableDataSource<S, T> {

    /**
     * Load data data source as the target type.
     *
     * @return the target data.
     * @throws Exception IO or other error occurs
     */
    T loadConfig() throws Exception;

    /**
     * Read original data from the data source.
     *
     * @return the original data.
     * @throws Exception IO or other error occurs
     */
    S readSource() throws Exception;

    /**
     * Get {@link SentinelProperty} of the data source.
     *
     * @return the property.
     */
    SentinelProperty<T> getProperty();

    /**
     * Close the data source.
     *
     * @throws Exception IO or other error occurs
     */
    void close() throws Exception;
}
```

![ReadableDataSource](/images/ReadableDataSource.png)

## WritableDataSource

```java
/**
 * Interface of writable data source support.
 *
 * @author Eric Zhao
 * @since 0.2.0
 */
public interface WritableDataSource<T> {

    /**
     * Write the {@code value} to the data source.
     *
     * @param value value to write
     * @throws Exception IO or other error occurs
     */
    void write(T value) throws Exception;

    /**
     * Close the data source.
     *
     * @throws Exception IO or other error occurs
     */
    void close() throws Exception;
}
```

![WritableDataSource](/images/WritableDataSource.png)