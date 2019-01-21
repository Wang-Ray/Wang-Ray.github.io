---
layout: post
title: "实践maven-antrun-extended-plugin"
date: 2018-11-13 16:03:13 +0800
categories: 工具
tags: maven tool maven-antrun-extended-plugin ant test report
---



```xml
<!-- 用mvn ant生成格式更友好的report -->  
            <plugin>  
                <groupId>org.jvnet.maven-antrun-extended-plugin</groupId>  
                <artifactId>maven-antrun-extended-plugin</artifactId>
                <version>1.43</version>
                <executions>  
                    <execution>  
                        <id>test-reports</id>  
                        <phase>test</phase>
                        <configuration>  
                            <tasks>  
                                <junitreport todir="${basedir}/target/surefire-reports">  
                                    <fileset dir="${basedir}/target/surefire-reports">  
                                        <include name="**/*.xml" />  
                                    </fileset>  
                                    <report format="frames" todir="${basedir}/target/surefire-reports" />
                                </junitreport>  
                            </tasks>  
                        </configuration>  
                        <goals>  
                            <goal>run</goal>  
                        </goals>  
                    </execution>  
                </executions>  
                <dependencies>  
                    <dependency>  
                        <groupId>org.apache.ant</groupId>  
                        <artifactId>ant-junit</artifactId>  
                        <version>1.8.0</version>  
                    </dependency>  
                    <dependency>  
                        <groupId>org.apache.ant</groupId>  
                        <artifactId>ant-trax</artifactId>  
                        <version>1.8.0</version>  
                    </dependency>  
                </dependencies>  
            </plugin>
```

`mvn test`





