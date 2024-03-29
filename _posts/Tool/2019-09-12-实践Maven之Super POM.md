---
layout: post
title: "实践Maven之Super POM"
categories: 工具
tags: Tool maven
---

%mavan_home%/lib/maven-model-builder-3.0.4.jar/org/apache/maven/model/pom-4.0.0.xml

%mavan_home%/lib/maven-core-3.0.4.jar/META-INF/plexus/components.xml

## 3.0.4

### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!--
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
-->

<!-- START SNIPPET: superpom -->
<project>
  <modelVersion>4.0.0</modelVersion>

  <repositories>
    <repository>
      <id>central</id>
      <name>Central Repository</name>
      <url>http://repo.maven.apache.org/maven2</url>
      <layout>default</layout>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
    </repository>
  </repositories>

  <pluginRepositories>
    <pluginRepository>
      <id>central</id>
      <name>Central Repository</name>
      <url>http://repo.maven.apache.org/maven2</url>
      <layout>default</layout>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
      <releases>
        <updatePolicy>never</updatePolicy>
      </releases>
    </pluginRepository>
  </pluginRepositories>

  <build>
    <directory>${project.basedir}/target</directory>
    <outputDirectory>${project.build.directory}/classes</outputDirectory>
    <finalName>${project.artifactId}-${project.version}</finalName>
    <testOutputDirectory>${project.build.directory}/test-classes</testOutputDirectory>
    <sourceDirectory>${project.basedir}/src/main/java</sourceDirectory>
    <scriptSourceDirectory>src/main/scripts</scriptSourceDirectory>
    <testSourceDirectory>${project.basedir}/src/test/java</testSourceDirectory>
    <resources>
      <resource>
        <directory>${project.basedir}/src/main/resources</directory>
      </resource>
    </resources>
    <testResources>
      <testResource>
        <directory>${project.basedir}/src/test/resources</directory>
      </testResource>
    </testResources>
    <pluginManagement>
      <!-- NOTE: These plugins will be removed from future versions of the super POM -->
      <!-- They are kept for the moment as they are very unlikely to conflict with lifecycle mappings (MNG-4453) -->
      <plugins>
        <plugin>
          <artifactId>maven-antrun-plugin</artifactId>
          <version>1.3</version>
        </plugin>
        <plugin>
          <artifactId>maven-assembly-plugin</artifactId>
          <version>2.2-beta-5</version>
        </plugin>
        <plugin>
          <artifactId>maven-dependency-plugin</artifactId>
          <version>2.1</version>
        </plugin>
        <plugin>
          <artifactId>maven-release-plugin</artifactId>
          <version>2.0</version>
        </plugin>
      </plugins>
    </pluginManagement>
  </build>

  <reporting>
    <outputDirectory>${project.build.directory}/site</outputDirectory>
  </reporting>

  <profiles>
    <!-- NOTE: The release profile will be removed from future versions of the super POM -->
    <profile>
      <id>release-profile</id>

      <activation>
        <property>
          <name>performRelease</name>
          <value>true</value>
        </property>
      </activation>

      <build>
        <plugins>
          <plugin>
            <inherited>true</inherited>
            <artifactId>maven-source-plugin</artifactId>
            <executions>
              <execution>
                <id>attach-sources</id>
                <goals>
                  <goal>jar</goal>
                </goals>
              </execution>
            </executions>
          </plugin>
          <plugin>
            <inherited>true</inherited>
            <artifactId>maven-javadoc-plugin</artifactId>
            <executions>
              <execution>
                <id>attach-javadocs</id>
                <goals>
                  <goal>jar</goal>
                </goals>
              </execution>
            </executions>
          </plugin>
          <plugin>
            <inherited>true</inherited>
            <artifactId>maven-deploy-plugin</artifactId>
            <configuration>
              <updateReleaseInfo>true</updateReleaseInfo>
            </configuration>
          </plugin>
        </plugins>
      </build>
    </profile>
  </profiles>

</project>
<!-- END SNIPPET: superpom -->
```

### components.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<component-set>
  <components>
    <component>
      <role>org.apache.maven.lifecycle.DefaultSchedules</role>
      <implementation>org.apache.maven.lifecycle.DefaultSchedules</implementation>
      <configuration>
        <schedules>
          <scheduling>
            <lifecycle>default</lifecycle>
            <schedules>
              <schedule>
                <phase>test</phase>
                <mojoSynchronized>false</mojoSynchronized>
                <parallel>true</parallel>
              </schedule>
              <schedule>
                <pluginKey>org.apache.maven.plugins:maven-assembly-plugin</pluginKey>
                <mojoSynchronized>true</mojoSynchronized>
              </schedule>
              <schedule>
                <pluginKey>org.apache.maven.plugins:maven-ear-plugin</pluginKey>
                <mojoGoal>generate-application-xml</mojoGoal>
                <upstreamPhase>package</upstreamPhase>
              </schedule>
            </schedules>
          </scheduling>
        </schedules>
      </configuration>
    </component>

    
    <component>
      <role>org.apache.maven.lifecycle.Lifecycle</role>
      <implementation>org.apache.maven.lifecycle.Lifecycle</implementation>
      <role-hint>default</role-hint>
      <configuration>
        <id>default</id>
        
        <phases>
          <phase>validate</phase>
          <phase>initialize</phase>
          <phase>generate-sources</phase>
          <phase>process-sources</phase>
          <phase>generate-resources</phase>
          <phase>process-resources</phase>
          <phase>compile</phase>
          <phase>process-classes</phase>
          <phase>generate-test-sources</phase>
          <phase>process-test-sources</phase>
          <phase>generate-test-resources</phase>
          <phase>process-test-resources</phase>
          <phase>test-compile</phase>
          <phase>process-test-classes</phase>
          <phase>test</phase>
          <phase>prepare-package</phase>
          <phase>package</phase>
          <phase>pre-integration-test</phase>
          <phase>integration-test</phase>
          <phase>post-integration-test</phase>
          <phase>verify</phase>
          <phase>install</phase>
          <phase>deploy</phase>
        </phases>
        
      </configuration>
    </component>

    
    <component>
      <role>org.apache.maven.lifecycle.Lifecycle</role>
      <implementation>org.apache.maven.lifecycle.Lifecycle</implementation>
      <role-hint>clean</role-hint>
      <configuration>
        <id>clean</id>
        
        <phases>
          <phase>pre-clean</phase>
          <phase>clean</phase>
          <phase>post-clean</phase>
        </phases>
        <default-phases>
          <clean>
            org.apache.maven.plugins:maven-clean-plugin:2.4.1:clean
          </clean>
        </default-phases>
        
      </configuration>
    </component>

    
    <component>
      <role>org.apache.maven.lifecycle.Lifecycle</role>
      <implementation>org.apache.maven.lifecycle.Lifecycle</implementation>
      <role-hint>site</role-hint>
      <configuration>
        <id>site</id>
        
        <phases>
          <phase>pre-site</phase>
          <phase>site</phase>
          <phase>post-site</phase>
          <phase>site-deploy</phase>
        </phases>
        <default-phases>
          <site>
            org.apache.maven.plugins:maven-site-plugin:3.0:site
          </site>
          <site-deploy>
            org.apache.maven.plugins:maven-site-plugin:3.0:deploy
          </site-deploy>
        </default-phases>
        
      </configuration>
    </component>

    <component>
      <role>org.sonatype.plexus.components.sec.dispatcher.SecDispatcher</role>
      <role-hint>maven</role-hint>
      <implementation>org.sonatype.plexus.components.sec.dispatcher.DefaultSecDispatcher</implementation>
      <description>Maven Security dispatcher</description>
      <requirements>
        <requirement>
          <role>org.sonatype.plexus.components.cipher.PlexusCipher</role>
          <field-name>_cipher</field-name>
        </requirement>
      </requirements>
      <configuration>
        <_configuration-file>~/.m2/settings-security.xml</_configuration-file>
      </configuration>
    </component>
  <component>
      <role>org.apache.maven.artifact.handler.ArtifactHandler</role>
      <role-hint>pom</role-hint>
      <implementation>org.apache.maven.artifact.handler.DefaultArtifactHandler</implementation>
      <configuration>
        <type>pom</type>
      </configuration>
    </component><component>
      <role>org.apache.maven.lifecycle.mapping.LifecycleMapping</role>
      <role-hint>pom</role-hint>
      <implementation>org.apache.maven.lifecycle.mapping.DefaultLifecycleMapping</implementation>
      <configuration>
        <lifecycles>
          <lifecycle>
            <id>default</id>
            
            <phases>
              <install>
                org.apache.maven.plugins:maven-install-plugin:2.3.1:install
              </install>
              <deploy>
                org.apache.maven.plugins:maven-deploy-plugin:2.7:deploy
              </deploy>
            </phases>
            
          </lifecycle>
        </lifecycles>
      </configuration>
    </component><component>
      <role>org.apache.maven.artifact.handler.ArtifactHandler</role>
      <role-hint>jar</role-hint>
      <implementation>org.apache.maven.artifact.handler.DefaultArtifactHandler</implementation>
      <configuration>
        <type>jar</type>
        <language>java</language>
        <addedToClasspath>true</addedToClasspath>
      </configuration>
    </component><component>
      <role>org.apache.maven.lifecycle.mapping.LifecycleMapping</role>
      <role-hint>jar</role-hint>
      <implementation>org.apache.maven.lifecycle.mapping.DefaultLifecycleMapping</implementation>
      <configuration>
        <lifecycles>
          <lifecycle>
            <id>default</id>
            
            <phases>
              <process-resources>
                org.apache.maven.plugins:maven-resources-plugin:2.5:resources
              </process-resources>
              <compile>
                org.apache.maven.plugins:maven-compiler-plugin:2.3.2:compile
              </compile>
              <process-test-resources>
                org.apache.maven.plugins:maven-resources-plugin:2.5:testResources
              </process-test-resources>
              <test-compile>
                org.apache.maven.plugins:maven-compiler-plugin:2.3.2:testCompile
              </test-compile>
              <test>
                org.apache.maven.plugins:maven-surefire-plugin:2.10:test
              </test>
              <package>
                org.apache.maven.plugins:maven-jar-plugin:2.3.2:jar
              </package>
              <install>
                org.apache.maven.plugins:maven-install-plugin:2.3.1:install
              </install>
              <deploy>
                org.apache.maven.plugins:maven-deploy-plugin:2.7:deploy
              </deploy>
            </phases>
            
          </lifecycle>
        </lifecycles>
      </configuration>
    </component><component>
      <role>org.apache.maven.artifact.handler.ArtifactHandler</role>
      <role-hint>ejb</role-hint>
      <implementation>org.apache.maven.artifact.handler.DefaultArtifactHandler</implementation>
      <configuration>
        <type>ejb</type>
        <extension>jar</extension>
        <language>java</language>
        <addedToClasspath>true</addedToClasspath>
      </configuration>
    </component><component>
      <role>org.apache.maven.lifecycle.mapping.LifecycleMapping</role>
      <role-hint>ejb</role-hint>
      <implementation>org.apache.maven.lifecycle.mapping.DefaultLifecycleMapping</implementation>
      <configuration>
        <lifecycles>
          <lifecycle>
            <id>default</id>
            
            <phases>
              <process-resources>
                org.apache.maven.plugins:maven-resources-plugin:2.5:resources
              </process-resources>
              <compile>
                org.apache.maven.plugins:maven-compiler-plugin:2.3.2:compile
              </compile>
              <process-test-resources>
                org.apache.maven.plugins:maven-resources-plugin:2.5:testResources
              </process-test-resources>
              <test-compile>
                org.apache.maven.plugins:maven-compiler-plugin:2.3.2:testCompile
              </test-compile>
              <test>
                org.apache.maven.plugins:maven-surefire-plugin:2.10:test
              </test>
              <package>
                org.apache.maven.plugins:maven-ejb-plugin:2.3:ejb
              </package>
              <install>
                org.apache.maven.plugins:maven-install-plugin:2.3.1:install
              </install>
              <deploy>
                org.apache.maven.plugins:maven-deploy-plugin:2.7:deploy
              </deploy>
            </phases>
            
          </lifecycle>
        </lifecycles>
      </configuration>
    </component><component>
      <role>org.apache.maven.artifact.handler.ArtifactHandler</role>
      <role-hint>ejb-client</role-hint>
      <implementation>org.apache.maven.artifact.handler.DefaultArtifactHandler</implementation>
      <configuration>
        <type>ejb-client</type>
        <extension>jar</extension>
        <packaging>ejb</packaging>
        <classifier>client</classifier>
        <language>java</language>
        <addedToClasspath>true</addedToClasspath>
      </configuration>
    </component><component>
      <role>org.apache.maven.artifact.handler.ArtifactHandler</role>
      <role-hint>ejb3</role-hint>
      <implementation>org.apache.maven.artifact.handler.DefaultArtifactHandler</implementation>
      <configuration>
        <type>ejb3</type>
        <includesDependencies>true</includesDependencies>
        <language>java</language>
        <addedToClasspath>false</addedToClasspath>
      </configuration>
    </component><component>
      <role>org.apache.maven.lifecycle.mapping.LifecycleMapping</role>
      <role-hint>ejb3</role-hint>
      <implementation>org.apache.maven.lifecycle.mapping.DefaultLifecycleMapping</implementation>
      <configuration>
        <lifecycles>
          <lifecycle>
            <id>default</id>
            
            <phases>
              <process-resources>
                org.apache.maven.plugins:maven-resources-plugin:2.5:resources
              </process-resources>
              <compile>
                org.apache.maven.plugins:maven-compiler-plugin:2.3.2:compile
              </compile>
              <process-test-resources>
                org.apache.maven.plugins:maven-resources-plugin:2.5:testResources
              </process-test-resources>
              <test-compile>
                org.apache.maven.plugins:maven-compiler-plugin:2.3.2:testCompile
              </test-compile>
              <test>
                org.apache.maven.plugins:maven-surefire-plugin:2.10:test
              </test>
              <package>
                org.apache.maven.plugins:maven-ejb3-plugin:ejb3
              </package>
              <install>
                org.apache.maven.plugins:maven-install-plugin:2.3.1:install
              </install>
              <deploy>
                org.apache.maven.plugins:maven-deploy-plugin:2.7:deploy
              </deploy>
            </phases>
            
          </lifecycle>
        </lifecycles>
      </configuration>
    </component><component>
      <role>org.apache.maven.artifact.handler.ArtifactHandler</role>
      <role-hint>test-jar</role-hint>
      <implementation>org.apache.maven.artifact.handler.DefaultArtifactHandler</implementation>
      <configuration>
        <classifier>tests</classifier>
        <extension>jar</extension>
        <type>test-jar</type>
        <packaging>jar</packaging>
        <language>java</language>
        <addedToClasspath>true</addedToClasspath>
      </configuration>
    </component><component>
      <role>org.apache.maven.artifact.handler.ArtifactHandler</role>
      <role-hint>maven-plugin</role-hint>
      <implementation>org.apache.maven.artifact.handler.DefaultArtifactHandler</implementation>
      <configuration>
        <type>maven-plugin</type>
        <extension>jar</extension>
        <language>java</language>
        <addedToClasspath>true</addedToClasspath>
      </configuration>
    </component><component>
      <role>org.apache.maven.lifecycle.mapping.LifecycleMapping</role>
      <role-hint>maven-plugin</role-hint>
      <implementation>org.apache.maven.lifecycle.mapping.DefaultLifecycleMapping</implementation>
      <configuration>
        <lifecycles>
          <lifecycle>
            <id>default</id>
            
            <phases>
              <generate-resources>
                org.apache.maven.plugins:maven-plugin-plugin:2.9:descriptor
              </generate-resources>
              <process-resources>
                org.apache.maven.plugins:maven-resources-plugin:2.5:resources
              </process-resources>
              <compile>
                org.apache.maven.plugins:maven-compiler-plugin:2.3.2:compile
              </compile>
              <process-test-resources>
                org.apache.maven.plugins:maven-resources-plugin:2.5:testResources
              </process-test-resources>
              <test-compile>
                org.apache.maven.plugins:maven-compiler-plugin:2.3.2:testCompile
              </test-compile>
              <test>
                org.apache.maven.plugins:maven-surefire-plugin:2.10:test
              </test>
              <package>
                org.apache.maven.plugins:maven-jar-plugin:2.3.1:jar,
                org.apache.maven.plugins:maven-plugin-plugin:2.9:addPluginArtifactMetadata
              </package>
              <install>
                org.apache.maven.plugins:maven-install-plugin:2.3.1:install
              </install>
              <deploy>
                org.apache.maven.plugins:maven-deploy-plugin:2.7:deploy
              </deploy>
            </phases>
            
          </lifecycle>
        </lifecycles>
      </configuration>
    </component><component>
      <role>org.apache.maven.artifact.handler.ArtifactHandler</role>
      <role-hint>java-source</role-hint>
      <implementation>org.apache.maven.artifact.handler.DefaultArtifactHandler</implementation>
      <configuration>
        <classifier>sources</classifier>
        <type>java-source</type>
        <extension>jar</extension>
        <language>java</language>
        <addedToClasspath>false</addedToClasspath>
      </configuration>
    </component><component>
      <role>org.apache.maven.artifact.handler.ArtifactHandler</role>
      <role-hint>javadoc</role-hint>
      <implementation>org.apache.maven.artifact.handler.DefaultArtifactHandler</implementation>
      <configuration>
        <classifier>javadoc</classifier>
        <type>javadoc</type>
        <extension>jar</extension>
        <language>java</language>
        <addedToClasspath>true</addedToClasspath>
      </configuration>
    </component><component>
      <role>org.apache.maven.artifact.handler.ArtifactHandler</role>
      <role-hint>war</role-hint>
      <implementation>org.apache.maven.artifact.handler.DefaultArtifactHandler</implementation>
      <configuration>
        <type>war</type>
        <includesDependencies>true</includesDependencies>
        <language>java</language>
        <addedToClasspath>false</addedToClasspath>
      </configuration>
    </component><component>
      <role>org.apache.maven.lifecycle.mapping.LifecycleMapping</role>
      <role-hint>war</role-hint>
      <implementation>org.apache.maven.lifecycle.mapping.DefaultLifecycleMapping</implementation>
      <configuration>
        <lifecycles>
          <lifecycle>
            <id>default</id>
            
            <phases>
              <process-resources>
                org.apache.maven.plugins:maven-resources-plugin:2.5:resources
              </process-resources>
              <compile>
                org.apache.maven.plugins:maven-compiler-plugin:2.3.2:compile
              </compile>
              <process-test-resources>
                org.apache.maven.plugins:maven-resources-plugin:2.5:testResources
              </process-test-resources>
              <test-compile>
                org.apache.maven.plugins:maven-compiler-plugin:2.3.2:testCompile
              </test-compile>
              <test>
                org.apache.maven.plugins:maven-surefire-plugin:2.10:test
              </test>
              <package>
                org.apache.maven.plugins:maven-war-plugin:2.1.1:war
              </package>
              <install>
                org.apache.maven.plugins:maven-install-plugin:2.3.1:install
              </install>
              <deploy>
                org.apache.maven.plugins:maven-deploy-plugin:2.7:deploy
              </deploy>
            </phases>
            
          </lifecycle>
        </lifecycles>
      </configuration>
    </component><component>
      <role>org.apache.maven.artifact.handler.ArtifactHandler</role>
      <role-hint>ear</role-hint>
      <implementation>org.apache.maven.artifact.handler.DefaultArtifactHandler</implementation>
      <configuration>
        <type>ear</type>
        <includesDependencies>true</includesDependencies>
        <language>java</language>
        <addedToClasspath>false</addedToClasspath>
      </configuration>
    </component><component>
      <role>org.apache.maven.lifecycle.mapping.LifecycleMapping</role>
      <role-hint>ear</role-hint>
      <implementation>org.apache.maven.lifecycle.mapping.DefaultLifecycleMapping</implementation>
      <configuration>
        <lifecycles>
          <lifecycle>
            <id>default</id>
            
            <phases>
              <generate-resources>
                org.apache.maven.plugins:maven-ear-plugin:2.5:generate-application-xml
              </generate-resources>
              <process-resources>
                org.apache.maven.plugins:maven-resources-plugin:2.4.3:resources
              </process-resources>
              <package>
                org.apache.maven.plugins:maven-ear-plugin:2.6:ear
              </package>
              <install>
                org.apache.maven.plugins:maven-install-plugin:2.3.1:install
              </install>
              <deploy>
                org.apache.maven.plugins:maven-deploy-plugin:2.7:deploy
              </deploy>
            </phases>
            
          </lifecycle>
        </lifecycles>
      </configuration>
    </component><component>
      <role>org.apache.maven.artifact.handler.ArtifactHandler</role>
      <role-hint>rar</role-hint>
      <implementation>org.apache.maven.artifact.handler.DefaultArtifactHandler</implementation>
      <configuration>
        <type>rar</type>
        <includesDependencies>true</includesDependencies>
        <language>java</language>
        <addedToClasspath>false</addedToClasspath>
      </configuration>
    </component><component>
      <role>org.apache.maven.lifecycle.mapping.LifecycleMapping</role>
      <role-hint>rar</role-hint>
      <implementation>org.apache.maven.lifecycle.mapping.DefaultLifecycleMapping</implementation>
      <configuration>
        <lifecycles>
          <lifecycle>
            <id>default</id>
            
            <phases>
              <process-resources>
                org.apache.maven.plugins:maven-resources-plugin:2.5:resources
              </process-resources>
              <compile>
                org.apache.maven.plugins:maven-compiler-plugin:2.3.2:compile
              </compile>
              <process-test-resources>
                org.apache.maven.plugins:maven-resources-plugin:2.5:testResources
              </process-test-resources>
              <test-compile>
                org.apache.maven.plugins:maven-compiler-plugin:2.3.2:testCompile
              </test-compile>
              <test>
                org.apache.maven.plugins:maven-surefire-plugin:2.10:test
              </test>
              <package>
                org.apache.maven.plugins:maven-rar-plugin:2.2:rar
              </package>
              <install>
                org.apache.maven.plugins:maven-install-plugin:2.3.1:install
              </install>
              <deploy>
                org.apache.maven.plugins:maven-deploy-plugin:2.7:deploy
              </deploy>
            </phases>
            
          </lifecycle>
        </lifecycles>
      </configuration>
    </component><component>
      <role>org.apache.maven.artifact.handler.ArtifactHandler</role>
      <role-hint>par</role-hint>
      <implementation>org.apache.maven.artifact.handler.DefaultArtifactHandler</implementation>
      <configuration>
        <type>par</type>
        <includesDependencies>true</includesDependencies>
        <language>java</language>
        <addedToClasspath>false</addedToClasspath>
      </configuration>
    </component><component>
      <role>org.apache.maven.lifecycle.mapping.LifecycleMapping</role>
      <role-hint>par</role-hint>
      <implementation>org.apache.maven.lifecycle.mapping.DefaultLifecycleMapping</implementation>
      <configuration>
        <lifecycles>
          <lifecycle>
            <id>default</id>
            
            <phases>
              <process-resources>
                org.apache.maven.plugins:maven-resources-plugin:2.5:resources
              </process-resources>
              <compile>
                org.apache.maven.plugins:maven-compiler-plugin:2.3.2:compile
              </compile>
              <process-test-resources>
                org.apache.maven.plugins:maven-resources-plugin:2.5:testResources
              </process-test-resources>
              <test-compile>
                org.apache.maven.plugins:maven-compiler-plugin:2.3.2:testCompile
              </test-compile>
              <test>
                org.apache.maven.plugins:maven-surefire-plugin:2.10:test
              </test>
              <package>
                org.apache.maven.plugins:maven-par-plugin:par
              </package>
              <install>
                org.apache.maven.plugins:maven-install-plugin:2.3.1:install
              </install>
              <deploy>
                org.apache.maven.plugins:maven-deploy-plugin:2.7:deploy
              </deploy>
            </phases>
            
          </lifecycle>
        </lifecycles>
      </configuration>
    </component><component>
      <role>org.apache.maven.artifact.factory.ArtifactFactory</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.artifact.factory.DefaultArtifactFactory</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.artifact.handler.manager.ArtifactHandlerManager</role>
          <role-hint />
          <field-name>artifactHandlerManager</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.artifact.handler.ArtifactHandler</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.artifact.handler.DefaultArtifactHandler</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
    </component><component>
      <role>org.apache.maven.artifact.handler.manager.ArtifactHandlerManager</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.artifact.handler.manager.DefaultArtifactHandlerManager</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.artifact.handler.ArtifactHandler</role>
          <field-name>artifactHandlers</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.artifact.repository.layout.ArtifactRepositoryLayout</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.artifact.repository.layout.DefaultRepositoryLayout</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
    </component><component>
      <role>org.apache.maven.artifact.repository.metadata.io.MetadataReader</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.artifact.repository.metadata.io.DefaultMetadataReader</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
    </component><component>
      <role>org.apache.maven.artifact.resolver.ResolutionErrorHandler</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.artifact.resolver.DefaultResolutionErrorHandler</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
    </component><component>
      <role>org.apache.maven.classrealm.ClassRealmManager</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.classrealm.DefaultClassRealmManager</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <role-hint />
          <field-name>logger</field-name>
        </requirement>
        <requirement>
          <role>org.codehaus.plexus.PlexusContainer</role>
          <role-hint />
          <field-name>container</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.configuration.BeanConfigurator</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.configuration.internal.DefaultBeanConfigurator</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
    </component><component>
      <role>org.apache.maven.ArtifactFilterManager</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.DefaultArtifactFilterManager</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.PlexusContainer</role>
          <role-hint />
          <field-name>plexus</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.Maven</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.DefaultMaven</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <role-hint />
          <field-name>logger</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.project.ProjectBuilder</role>
          <role-hint />
          <field-name>projectBuilder</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.LifecycleStarter</role>
          <role-hint />
          <field-name>lifecycleStarter</field-name>
        </requirement>
        <requirement>
          <role>org.codehaus.plexus.PlexusContainer</role>
          <role-hint />
          <field-name>container</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.execution.MavenExecutionRequestPopulator</role>
          <role-hint />
          <field-name>populator</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.ExecutionEventCatapult</role>
          <role-hint />
          <field-name>eventCatapult</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.artifact.handler.manager.ArtifactHandlerManager</role>
          <role-hint />
          <field-name>artifactHandlerManager</field-name>
        </requirement>
        <requirement>
          <role>org.sonatype.aether.repository.WorkspaceReader</role>
          <role-hint>ide</role-hint>
          <field-name>workspaceRepository</field-name>
          <optional>true</optional>
        </requirement>
        <requirement>
          <role>org.sonatype.aether.RepositorySystem</role>
          <role-hint />
          <field-name>repoSystem</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.settings.crypto.SettingsDecrypter</role>
          <role-hint />
          <field-name>settingsDecrypter</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.LegacySupport</role>
          <role-hint />
          <field-name>legacySupport</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.eventspy.internal.EventSpyDispatcher</role>
          <role-hint />
          <field-name>eventSpyDispatcher</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.ProjectDependenciesResolver</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.DefaultProjectDependenciesResolver</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.repository.RepositorySystem</role>
          <role-hint />
          <field-name>repositorySystem</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.artifact.resolver.ResolutionErrorHandler</role>
          <role-hint />
          <field-name>resolutionErrorHandler</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.eventspy.internal.EventSpyDispatcher</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.eventspy.internal.EventSpyDispatcher</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <role-hint />
          <field-name>logger</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.eventspy.EventSpy</role>
          <field-name>eventSpies</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.exception.ExceptionHandler</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.exception.DefaultExceptionHandler</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
    </component><component>
      <role>org.apache.maven.execution.MavenExecutionRequestPopulator</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.execution.DefaultMavenExecutionRequestPopulator</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.repository.RepositorySystem</role>
          <role-hint />
          <field-name>repositorySystem</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.lifecycle.LifecycleExecutor</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.lifecycle.DefaultLifecycleExecutor</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.lifecycle.LifeCyclePluginAnalyzer</role>
          <role-hint />
          <field-name>lifeCyclePluginAnalyzer</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.DefaultLifecycles</role>
          <role-hint />
          <field-name>defaultLifeCycles</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.LifecycleTaskSegmentCalculator</role>
          <role-hint />
          <field-name>lifecycleTaskSegmentCalculator</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.LifecycleExecutionPlanCalculator</role>
          <role-hint />
          <field-name>lifecycleExecutionPlanCalculator</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.MojoExecutor</role>
          <role-hint />
          <field-name>mojoExecutor</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.LifecycleStarter</role>
          <role-hint />
          <field-name>lifecycleStarter</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.MojoDescriptorCreator</role>
          <role-hint />
          <field-name>mojoDescriptorCreator</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.lifecycle.DefaultLifecycles</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.lifecycle.DefaultLifecycles</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.lifecycle.Lifecycle</role>
          <field-name>lifecycles</field-name>
        </requirement>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <role-hint />
          <field-name>logger</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.lifecycle.internal.BuilderCommon</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.lifecycle.internal.BuilderCommon</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.LifecycleDebugLogger</role>
          <role-hint />
          <field-name>lifecycleDebugLogger</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.LifecycleExecutionPlanCalculator</role>
          <role-hint />
          <field-name>lifeCycleExecutionPlanCalculator</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.ExecutionEventCatapult</role>
          <role-hint />
          <field-name>eventCatapult</field-name>
        </requirement>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <role-hint />
          <field-name>logger</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.lifecycle.internal.BuildListCalculator</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.lifecycle.internal.BuildListCalculator</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
    </component><component>
      <role>org.apache.maven.lifecycle.internal.ExecutionEventCatapult</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.lifecycle.internal.DefaultExecutionEventCatapult</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
    </component><component>
      <role>org.apache.maven.lifecycle.internal.LifecycleExecutionPlanCalculator</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.lifecycle.internal.DefaultLifecycleExecutionPlanCalculator</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.plugin.version.PluginVersionResolver</role>
          <role-hint />
          <field-name>pluginVersionResolver</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.BuildPluginManager</role>
          <role-hint />
          <field-name>pluginManager</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.DefaultLifecycles</role>
          <role-hint />
          <field-name>defaultLifeCycles</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.DefaultSchedules</role>
          <role-hint />
          <field-name>defaultSchedules</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.MojoDescriptorCreator</role>
          <role-hint />
          <field-name>mojoDescriptorCreator</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.LifecyclePluginResolver</role>
          <role-hint />
          <field-name>lifecyclePluginResolver</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.lifecycle.LifeCyclePluginAnalyzer</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.lifecycle.internal.DefaultLifecyclePluginAnalyzer</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.lifecycle.mapping.LifecycleMapping</role>
          <field-name>lifecycleMappings</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.DefaultLifecycles</role>
          <role-hint />
          <field-name>defaultLifeCycles</field-name>
        </requirement>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <role-hint />
          <field-name>logger</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.lifecycle.internal.LifecycleTaskSegmentCalculator</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.lifecycle.internal.DefaultLifecycleTaskSegmentCalculator</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.MojoDescriptorCreator</role>
          <role-hint />
          <field-name>mojoDescriptorCreator</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.LifecyclePluginResolver</role>
          <role-hint />
          <field-name>lifecyclePluginResolver</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.lifecycle.internal.LifecycleDebugLogger</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.lifecycle.internal.LifecycleDebugLogger</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <role-hint />
          <field-name>logger</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.lifecycle.internal.LifecycleDependencyResolver</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.lifecycle.internal.LifecycleDependencyResolver</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.project.ProjectDependenciesResolver</role>
          <role-hint />
          <field-name>dependenciesResolver</field-name>
        </requirement>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <role-hint />
          <field-name>logger</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.artifact.factory.ArtifactFactory</role>
          <role-hint />
          <field-name>artifactFactory</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.eventspy.internal.EventSpyDispatcher</role>
          <role-hint />
          <field-name>eventSpyDispatcher</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.lifecycle.internal.LifecycleModuleBuilder</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.lifecycle.internal.LifecycleModuleBuilder</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.MojoExecutor</role>
          <role-hint />
          <field-name>mojoExecutor</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.BuilderCommon</role>
          <role-hint />
          <field-name>builderCommon</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.ExecutionEventCatapult</role>
          <role-hint />
          <field-name>eventCatapult</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.lifecycle.internal.LifecyclePluginResolver</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.lifecycle.internal.LifecyclePluginResolver</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.plugin.version.PluginVersionResolver</role>
          <role-hint />
          <field-name>pluginVersionResolver</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.lifecycle.internal.LifecycleStarter</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.lifecycle.internal.LifecycleStarter</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.ExecutionEventCatapult</role>
          <role-hint />
          <field-name>eventCatapult</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.DefaultLifecycles</role>
          <role-hint />
          <field-name>defaultLifeCycles</field-name>
        </requirement>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <role-hint />
          <field-name>logger</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.LifecycleModuleBuilder</role>
          <role-hint />
          <field-name>lifecycleModuleBuilder</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.LifecycleWeaveBuilder</role>
          <role-hint />
          <field-name>lifeCycleWeaveBuilder</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.LifecycleThreadedBuilder</role>
          <role-hint />
          <field-name>lifecycleThreadedBuilder</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.BuildListCalculator</role>
          <role-hint />
          <field-name>buildListCalculator</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.LifecycleDebugLogger</role>
          <role-hint />
          <field-name>lifecycleDebugLogger</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.LifecycleTaskSegmentCalculator</role>
          <role-hint />
          <field-name>lifecycleTaskSegmentCalculator</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.ThreadConfigurationService</role>
          <role-hint />
          <field-name>threadConfigService</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.lifecycle.internal.LifecycleThreadedBuilder</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.lifecycle.internal.LifecycleThreadedBuilder</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <role-hint />
          <field-name>logger</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.LifecycleModuleBuilder</role>
          <role-hint />
          <field-name>lifecycleModuleBuilder</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.lifecycle.internal.LifecycleWeaveBuilder</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.lifecycle.internal.LifecycleWeaveBuilder</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.MojoExecutor</role>
          <role-hint />
          <field-name>mojoExecutor</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.BuilderCommon</role>
          <role-hint />
          <field-name>builderCommon</field-name>
        </requirement>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <role-hint />
          <field-name>logger</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.ExecutionEventCatapult</role>
          <role-hint />
          <field-name>eventCatapult</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.lifecycle.internal.MojoDescriptorCreator</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.lifecycle.internal.MojoDescriptorCreator</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <role-hint />
          <field-name>logger</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.version.PluginVersionResolver</role>
          <role-hint />
          <field-name>pluginVersionResolver</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.BuildPluginManager</role>
          <role-hint />
          <field-name>pluginManager</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.prefix.PluginPrefixResolver</role>
          <role-hint />
          <field-name>pluginPrefixResolver</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.LifecyclePluginResolver</role>
          <role-hint />
          <field-name>lifecyclePluginResolver</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.lifecycle.internal.MojoExecutor</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.lifecycle.internal.MojoExecutor</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.plugin.BuildPluginManager</role>
          <role-hint />
          <field-name>pluginManager</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.MavenPluginManager</role>
          <role-hint />
          <field-name>mavenPluginManager</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.LifecycleDependencyResolver</role>
          <role-hint />
          <field-name>lifeCycleDependencyResolver</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.ExecutionEventCatapult</role>
          <role-hint />
          <field-name>eventCatapult</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.lifecycle.internal.ThreadConfigurationService</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.lifecycle.internal.ThreadConfigurationService</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <role-hint />
          <field-name>logger</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.model.plugin.LifecycleBindingsInjector</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.model.plugin.DefaultLifecycleBindingsInjector</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.lifecycle.LifeCyclePluginAnalyzer</role>
          <role-hint />
          <field-name>lifecycle</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.plugin.BuildPluginManager</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.plugin.DefaultBuildPluginManager</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.plugin.MavenPluginManager</role>
          <role-hint />
          <field-name>mavenPluginManager</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.LegacySupport</role>
          <role-hint />
          <field-name>legacySupport</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.plugin.ExtensionRealmCache</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.plugin.DefaultExtensionRealmCache</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
    </component><component>
      <role>org.apache.maven.plugin.PluginArtifactsCache</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.plugin.DefaultPluginArtifactsCache</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
    </component><component>
      <role>org.apache.maven.plugin.PluginDescriptorCache</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.plugin.DefaultPluginDescriptorCache</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
    </component><component>
      <role>org.apache.maven.plugin.PluginRealmCache</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.plugin.DefaultPluginRealmCache</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
    </component><component>
      <role>org.apache.maven.plugin.LegacySupport</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.plugin.internal.DefaultLegacySupport</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
    </component><component>
      <role>org.apache.maven.plugin.MavenPluginManager</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.plugin.internal.DefaultMavenPluginManager</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <role-hint />
          <field-name>logger</field-name>
        </requirement>
        <requirement>
          <role>org.codehaus.plexus.PlexusContainer</role>
          <role-hint />
          <field-name>container</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.classrealm.ClassRealmManager</role>
          <role-hint />
          <field-name>classRealmManager</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.PluginDescriptorCache</role>
          <role-hint />
          <field-name>pluginDescriptorCache</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.PluginRealmCache</role>
          <role-hint />
          <field-name>pluginRealmCache</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.internal.PluginDependenciesResolver</role>
          <role-hint />
          <field-name>pluginDependenciesResolver</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.rtinfo.RuntimeInformation</role>
          <role-hint />
          <field-name>runtimeInformation</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.plugin.internal.PluginDependenciesResolver</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.plugin.internal.DefaultPluginDependenciesResolver</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <role-hint />
          <field-name>logger</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.ArtifactFilterManager</role>
          <role-hint />
          <field-name>artifactFilterManager</field-name>
        </requirement>
        <requirement>
          <role>org.sonatype.aether.RepositorySystem</role>
          <role-hint />
          <field-name>repoSystem</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.plugin.PluginManager</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.plugin.internal.DefaultPluginManager</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.PlexusContainer</role>
          <role-hint />
          <field-name>container</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.MavenPluginManager</role>
          <role-hint />
          <field-name>pluginManager</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.version.PluginVersionResolver</role>
          <role-hint />
          <field-name>pluginVersionResolver</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.prefix.PluginPrefixResolver</role>
          <role-hint />
          <field-name>pluginPrefixResolver</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.LegacySupport</role>
          <role-hint />
          <field-name>legacySupport</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.plugin.prefix.PluginPrefixResolver</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.plugin.prefix.internal.DefaultPluginPrefixResolver</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <role-hint />
          <field-name>logger</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.BuildPluginManager</role>
          <role-hint />
          <field-name>pluginManager</field-name>
        </requirement>
        <requirement>
          <role>org.sonatype.aether.RepositorySystem</role>
          <role-hint />
          <field-name>repositorySystem</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.artifact.repository.metadata.io.MetadataReader</role>
          <role-hint />
          <field-name>metadataReader</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.plugin.version.PluginVersionResolver</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.plugin.version.internal.DefaultPluginVersionResolver</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <role-hint />
          <field-name>logger</field-name>
        </requirement>
        <requirement>
          <role>org.sonatype.aether.RepositorySystem</role>
          <role-hint />
          <field-name>repositorySystem</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.artifact.repository.metadata.io.MetadataReader</role>
          <role-hint />
          <field-name>metadataReader</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.MavenPluginManager</role>
          <role-hint />
          <field-name>pluginManager</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.project.artifact.MavenMetadataCache</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.project.artifact.DefaultMavenMetadataCache</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
    </component><component>
      <role>org.apache.maven.artifact.metadata.ArtifactMetadataSource</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.project.artifact.DefaultMetadataSource</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.artifact.repository.metadata.RepositoryMetadataManager</role>
          <role-hint />
          <field-name>repositoryMetadataManager</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.artifact.factory.ArtifactFactory</role>
          <role-hint />
          <field-name>repositorySystem</field-name>
        </requirement>
        <requirement>
          <role>org.codehaus.plexus.PlexusContainer</role>
          <role-hint />
          <field-name>container</field-name>
        </requirement>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <role-hint />
          <field-name>logger</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.project.artifact.MavenMetadataCache</role>
          <role-hint />
          <field-name>cache</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.LegacySupport</role>
          <role-hint />
          <field-name>legacySupport</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.artifact.metadata.ArtifactMetadataSource</role>
      <role-hint>maven</role-hint>
      <implementation>org.apache.maven.project.artifact.MavenMetadataSource</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.artifact.repository.metadata.RepositoryMetadataManager</role>
          <role-hint />
          <field-name>repositoryMetadataManager</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.artifact.factory.ArtifactFactory</role>
          <role-hint />
          <field-name>repositorySystem</field-name>
        </requirement>
        <requirement>
          <role>org.codehaus.plexus.PlexusContainer</role>
          <role-hint />
          <field-name>container</field-name>
        </requirement>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <role-hint />
          <field-name>logger</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.project.artifact.MavenMetadataCache</role>
          <role-hint />
          <field-name>cache</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.LegacySupport</role>
          <role-hint />
          <field-name>legacySupport</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.project.MavenProjectHelper</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.project.DefaultMavenProjectHelper</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.artifact.handler.manager.ArtifactHandlerManager</role>
          <role-hint />
          <field-name>artifactHandlerManager</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.project.ProjectBuilder</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.project.DefaultProjectBuilder</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <role-hint />
          <field-name>logger</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.model.building.ModelBuilder</role>
          <role-hint />
          <field-name>modelBuilder</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.model.building.ModelProcessor</role>
          <role-hint />
          <field-name>modelProcessor</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.project.ProjectBuildingHelper</role>
          <role-hint />
          <field-name>projectBuildingHelper</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.repository.RepositorySystem</role>
          <role-hint />
          <field-name>repositorySystem</field-name>
        </requirement>
        <requirement>
          <role>org.sonatype.aether.RepositorySystem</role>
          <role-hint />
          <field-name>repoSystem</field-name>
        </requirement>
        <requirement>
          <role>org.sonatype.aether.impl.RemoteRepositoryManager</role>
          <role-hint />
          <field-name>repositoryManager</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.project.ProjectDependenciesResolver</role>
          <role-hint />
          <field-name>dependencyResolver</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.project.ProjectBuildingHelper</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.project.DefaultProjectBuildingHelper</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <role-hint />
          <field-name>logger</field-name>
        </requirement>
        <requirement>
          <role>org.codehaus.plexus.PlexusContainer</role>
          <role-hint />
          <field-name>container</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.classrealm.ClassRealmManager</role>
          <role-hint />
          <field-name>classRealmManager</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.PluginArtifactsCache</role>
          <role-hint />
          <field-name>pluginArtifactsCache</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.ExtensionRealmCache</role>
          <role-hint />
          <field-name>extensionRealmCache</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.project.ProjectRealmCache</role>
          <role-hint />
          <field-name>projectRealmCache</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.repository.RepositorySystem</role>
          <role-hint />
          <field-name>repositorySystem</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.version.PluginVersionResolver</role>
          <role-hint />
          <field-name>pluginVersionResolver</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.internal.PluginDependenciesResolver</role>
          <role-hint />
          <field-name>pluginDependenciesResolver</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.project.ProjectDependenciesResolver</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.project.DefaultProjectDependenciesResolver</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <role-hint />
          <field-name>logger</field-name>
        </requirement>
        <requirement>
          <role>org.sonatype.aether.RepositorySystem</role>
          <role-hint />
          <field-name>repoSystem</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.project.ProjectRealmCache</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.project.DefaultProjectRealmCache</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
    </component><component>
      <role>org.apache.maven.rtinfo.RuntimeInformation</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.rtinfo.internal.DefaultRuntimeInformation</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <role-hint />
          <field-name>logger</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.settings.MavenSettingsBuilder</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.settings.DefaultMavenSettingsBuilder</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.settings.building.SettingsBuilder</role>
          <role-hint />
          <field-name>settingsBuilder</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.toolchain.ToolchainManager</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.toolchain.DefaultToolchainManager</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <role-hint />
          <field-name>logger</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.toolchain.ToolchainFactory</role>
          <field-name>factories</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.toolchain.ToolchainManagerPrivate</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.toolchain.DefaultToolchainManagerPrivate</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.toolchain.ToolchainsBuilder</role>
          <role-hint />
          <field-name>toolchainsBuilder</field-name>
        </requirement>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <role-hint />
          <field-name>logger</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.toolchain.ToolchainFactory</role>
          <field-name>factories</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.toolchain.ToolchainsBuilder</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.toolchain.DefaultToolchainsBuilder</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <role-hint />
          <field-name>logger</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.toolchain.java.JavaToolChain</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.toolchain.java.DefaultJavaToolChain</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
    </component><component>
      <role>org.apache.maven.toolchain.ToolchainFactory</role>
      <role-hint>jdk</role-hint>
      <implementation>org.apache.maven.toolchain.java.DefaultJavaToolchainFactory</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <role-hint />
          <field-name>logger</field-name>
        </requirement>
      </requirements>
    </component></components>
</component-set>
```

## 3.6.0

### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!--
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
-->

<!-- START SNIPPET: superpom -->
<project>
  <modelVersion>4.0.0</modelVersion>

  <repositories>
    <repository>
      <id>central</id>
      <name>Central Repository</name>
      <url>https://repo.maven.apache.org/maven2</url>
      <layout>default</layout>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
    </repository>
  </repositories>

  <pluginRepositories>
    <pluginRepository>
      <id>central</id>
      <name>Central Repository</name>
      <url>https://repo.maven.apache.org/maven2</url>
      <layout>default</layout>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
      <releases>
        <updatePolicy>never</updatePolicy>
      </releases>
    </pluginRepository>
  </pluginRepositories>

  <build>
    <directory>${project.basedir}/target</directory>
    <outputDirectory>${project.build.directory}/classes</outputDirectory>
    <finalName>${project.artifactId}-${project.version}</finalName>
    <testOutputDirectory>${project.build.directory}/test-classes</testOutputDirectory>
    <sourceDirectory>${project.basedir}/src/main/java</sourceDirectory>
    <scriptSourceDirectory>${project.basedir}/src/main/scripts</scriptSourceDirectory>
    <testSourceDirectory>${project.basedir}/src/test/java</testSourceDirectory>
    <resources>
      <resource>
        <directory>${project.basedir}/src/main/resources</directory>
      </resource>
    </resources>
    <testResources>
      <testResource>
        <directory>${project.basedir}/src/test/resources</directory>
      </testResource>
    </testResources>
    <pluginManagement>
      <!-- NOTE: These plugins will be removed from future versions of the super POM -->
      <!-- They are kept for the moment as they are very unlikely to conflict with lifecycle mappings (MNG-4453) -->
      <plugins>
        <plugin>
          <artifactId>maven-antrun-plugin</artifactId>
          <version>1.3</version>
        </plugin>
        <plugin>
          <artifactId>maven-assembly-plugin</artifactId>
          <version>2.2-beta-5</version>
        </plugin>
        <plugin>
          <artifactId>maven-dependency-plugin</artifactId>
          <version>2.8</version>
        </plugin>
        <plugin>
          <artifactId>maven-release-plugin</artifactId>
          <version>2.5.3</version>
        </plugin>
      </plugins>
    </pluginManagement>
  </build>

  <reporting>
    <outputDirectory>${project.build.directory}/site</outputDirectory>
  </reporting>

  <profiles>
    <!-- NOTE: The release profile will be removed from future versions of the super POM -->
    <profile>
      <id>release-profile</id>

      <activation>
        <property>
          <name>performRelease</name>
          <value>true</value>
        </property>
      </activation>

      <build>
        <plugins>
          <plugin>
            <inherited>true</inherited>
            <artifactId>maven-source-plugin</artifactId>
            <executions>
              <execution>
                <id>attach-sources</id>
                <goals>
                  <goal>jar-no-fork</goal>
                </goals>
              </execution>
            </executions>
          </plugin>
          <plugin>
            <inherited>true</inherited>
            <artifactId>maven-javadoc-plugin</artifactId>
            <executions>
              <execution>
                <id>attach-javadocs</id>
                <goals>
                  <goal>jar</goal>
                </goals>
              </execution>
            </executions>
          </plugin>
          <plugin>
            <inherited>true</inherited>
            <artifactId>maven-deploy-plugin</artifactId>
            <configuration>
              <updateReleaseInfo>true</updateReleaseInfo>
            </configuration>
          </plugin>
        </plugins>
      </build>
    </profile>
  </profiles>

</project>
<!-- END SNIPPET: superpom -->
```

### components.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<component-set>
  <components>
    
    <component>
      <role>org.apache.maven.lifecycle.mapping.LifecycleMapping</role>
      <role-hint>pom</role-hint>
      <implementation>org.apache.maven.lifecycle.mapping.DefaultLifecycleMapping</implementation>
      <configuration>
        <lifecycles>
          <lifecycle>
            <id>default</id>
            
            <phases>
              <install>
                org.apache.maven.plugins:maven-install-plugin:2.4:install
              </install>
              <deploy>
                org.apache.maven.plugins:maven-deploy-plugin:2.7:deploy
              </deploy>
            </phases>
            
          </lifecycle>
        </lifecycles>
      </configuration>
    </component>

    
    <component>
      <role>org.apache.maven.lifecycle.mapping.LifecycleMapping</role>
      <role-hint>jar</role-hint>
      <implementation>org.apache.maven.lifecycle.mapping.DefaultLifecycleMapping</implementation>
      <configuration>
        <lifecycles>
          <lifecycle>
            <id>default</id>
            
            <phases>
              <process-resources>
                org.apache.maven.plugins:maven-resources-plugin:2.6:resources
              </process-resources>
              <compile>
                org.apache.maven.plugins:maven-compiler-plugin:3.1:compile
              </compile>
              <process-test-resources>
                org.apache.maven.plugins:maven-resources-plugin:2.6:testResources
              </process-test-resources>
              <test-compile>
                org.apache.maven.plugins:maven-compiler-plugin:3.1:testCompile
              </test-compile>
              <test>
                org.apache.maven.plugins:maven-surefire-plugin:2.12.4:test
              </test>
              <package>
                org.apache.maven.plugins:maven-jar-plugin:2.4:jar
              </package>
              <install>
                org.apache.maven.plugins:maven-install-plugin:2.4:install
              </install>
              <deploy>
                org.apache.maven.plugins:maven-deploy-plugin:2.7:deploy
              </deploy>
            </phases>
            
          </lifecycle>
        </lifecycles>
      </configuration>
    </component>

    
    <component>
      <role>org.apache.maven.lifecycle.mapping.LifecycleMapping</role>
      <role-hint>ejb</role-hint>
      <implementation>org.apache.maven.lifecycle.mapping.DefaultLifecycleMapping</implementation>
      <configuration>
        <lifecycles>
          <lifecycle>
            <id>default</id>
            
            <phases>
              <process-resources>
                org.apache.maven.plugins:maven-resources-plugin:2.6:resources
              </process-resources>
              <compile>
                org.apache.maven.plugins:maven-compiler-plugin:3.1:compile
              </compile>
              <process-test-resources>
                org.apache.maven.plugins:maven-resources-plugin:2.6:testResources
              </process-test-resources>
              <test-compile>
                org.apache.maven.plugins:maven-compiler-plugin:3.1:testCompile
              </test-compile>
              <test>
                org.apache.maven.plugins:maven-surefire-plugin:2.12.4:test
              </test>
              <package>
                org.apache.maven.plugins:maven-ejb-plugin:2.3:ejb
              </package>
              <install>
                org.apache.maven.plugins:maven-install-plugin:2.4:install
              </install>
              <deploy>
                org.apache.maven.plugins:maven-deploy-plugin:2.7:deploy
              </deploy>
            </phases>
            
          </lifecycle>
        </lifecycles>
      </configuration>
    </component>

    
    <component>
      <role>org.apache.maven.lifecycle.mapping.LifecycleMapping</role>
      <role-hint>maven-plugin</role-hint>
      <implementation>org.apache.maven.lifecycle.mapping.DefaultLifecycleMapping</implementation>
      <configuration>
        <lifecycles>
          <lifecycle>
            <id>default</id>
            
            <phases>
              <process-resources>
                org.apache.maven.plugins:maven-resources-plugin:2.6:resources
              </process-resources>
              <compile>
                org.apache.maven.plugins:maven-compiler-plugin:3.1:compile
              </compile>
              <process-classes>
                org.apache.maven.plugins:maven-plugin-plugin:3.2:descriptor
              </process-classes>
              <process-test-resources>
                org.apache.maven.plugins:maven-resources-plugin:2.6:testResources
              </process-test-resources>
              <test-compile>
                org.apache.maven.plugins:maven-compiler-plugin:3.1:testCompile
              </test-compile>
              <test>
                org.apache.maven.plugins:maven-surefire-plugin:2.12.4:test
              </test>
              <package>
                org.apache.maven.plugins:maven-jar-plugin:2.4:jar,
                org.apache.maven.plugins:maven-plugin-plugin:3.2:addPluginArtifactMetadata
              </package>
              <install>
                org.apache.maven.plugins:maven-install-plugin:2.4:install
              </install>
              <deploy>
                org.apache.maven.plugins:maven-deploy-plugin:2.7:deploy
              </deploy>
            </phases>
            
          </lifecycle>
        </lifecycles>
      </configuration>
    </component>

    
    <component>
      <role>org.apache.maven.lifecycle.mapping.LifecycleMapping</role>
      <role-hint>war</role-hint>
      <implementation>org.apache.maven.lifecycle.mapping.DefaultLifecycleMapping</implementation>
      <configuration>
        <lifecycles>
          <lifecycle>
            <id>default</id>
            
            <phases>
              <process-resources>
                org.apache.maven.plugins:maven-resources-plugin:2.6:resources
              </process-resources>
              <compile>
                org.apache.maven.plugins:maven-compiler-plugin:3.1:compile
              </compile>
              <process-test-resources>
                org.apache.maven.plugins:maven-resources-plugin:2.6:testResources
              </process-test-resources>
              <test-compile>
                org.apache.maven.plugins:maven-compiler-plugin:3.1:testCompile
              </test-compile>
              <test>
                org.apache.maven.plugins:maven-surefire-plugin:2.12.4:test
              </test>
              <package>
                org.apache.maven.plugins:maven-war-plugin:2.2:war
              </package>
              <install>
                org.apache.maven.plugins:maven-install-plugin:2.4:install
              </install>
              <deploy>
                org.apache.maven.plugins:maven-deploy-plugin:2.7:deploy
              </deploy>
            </phases>
            
          </lifecycle>
        </lifecycles>
      </configuration>
    </component>

    
    <component>
      <role>org.apache.maven.lifecycle.mapping.LifecycleMapping</role>
      <role-hint>ear</role-hint>
      <implementation>org.apache.maven.lifecycle.mapping.DefaultLifecycleMapping</implementation>
      <configuration>
        <lifecycles>
          <lifecycle>
            <id>default</id>
            
            <phases>
              <generate-resources>
                org.apache.maven.plugins:maven-ear-plugin:2.8:generate-application-xml
              </generate-resources>
              <process-resources>
                org.apache.maven.plugins:maven-resources-plugin:2.6:resources
              </process-resources>
              <package>
                org.apache.maven.plugins:maven-ear-plugin:2.8:ear
              </package>
              <install>
                org.apache.maven.plugins:maven-install-plugin:2.4:install
              </install>
              <deploy>
                org.apache.maven.plugins:maven-deploy-plugin:2.7:deploy
              </deploy>
            </phases>
            
          </lifecycle>
        </lifecycles>
      </configuration>
    </component>

    
    <component>
      <role>org.apache.maven.lifecycle.mapping.LifecycleMapping</role>
      <role-hint>rar</role-hint>
      <implementation>org.apache.maven.lifecycle.mapping.DefaultLifecycleMapping</implementation>
      <configuration>
        <lifecycles>
          <lifecycle>
            <id>default</id>
            
            <phases>
              <process-resources>
                org.apache.maven.plugins:maven-resources-plugin:2.6:resources
              </process-resources>
              <compile>
                org.apache.maven.plugins:maven-compiler-plugin:3.1:compile
              </compile>
              <process-test-resources>
                org.apache.maven.plugins:maven-resources-plugin:2.6:testResources
              </process-test-resources>
              <test-compile>
                org.apache.maven.plugins:maven-compiler-plugin:3.1:testCompile
              </test-compile>
              <test>
                org.apache.maven.plugins:maven-surefire-plugin:2.12.4:test
              </test>
              <package>
                org.apache.maven.plugins:maven-rar-plugin:2.2:rar
              </package>
              <install>
                org.apache.maven.plugins:maven-install-plugin:2.4:install
              </install>
              <deploy>
                org.apache.maven.plugins:maven-deploy-plugin:2.7:deploy
              </deploy>
            </phases>
            
          </lifecycle>
        </lifecycles>
      </configuration>
    </component>

  <component>
      <role>org.apache.maven.lifecycle.Lifecycle</role>
      <implementation>org.apache.maven.lifecycle.Lifecycle</implementation>
      <role-hint>default</role-hint>
      <configuration>
        <id>default</id>
        
        <phases>
          <phase>validate</phase>
          <phase>initialize</phase>
          <phase>generate-sources</phase>
          <phase>process-sources</phase>
          <phase>generate-resources</phase>
          <phase>process-resources</phase>
          <phase>compile</phase>
          <phase>process-classes</phase>
          <phase>generate-test-sources</phase>
          <phase>process-test-sources</phase>
          <phase>generate-test-resources</phase>
          <phase>process-test-resources</phase>
          <phase>test-compile</phase>
          <phase>process-test-classes</phase>
          <phase>test</phase>
          <phase>prepare-package</phase>
          <phase>package</phase>
          <phase>pre-integration-test</phase>
          <phase>integration-test</phase>
          <phase>post-integration-test</phase>
          <phase>verify</phase>
          <phase>install</phase>
          <phase>deploy</phase>
        </phases>
        
      </configuration>
    </component><component>
      <role>org.apache.maven.lifecycle.Lifecycle</role>
      <implementation>org.apache.maven.lifecycle.Lifecycle</implementation>
      <role-hint>clean</role-hint>
      <configuration>
        <id>clean</id>
        
        <phases>
          <phase>pre-clean</phase>
          <phase>clean</phase>
          <phase>post-clean</phase>
        </phases>
        <default-phases>
          <clean>
            org.apache.maven.plugins:maven-clean-plugin:2.5:clean
          </clean>
        </default-phases>
        
      </configuration>
    </component><component>
      <role>org.apache.maven.lifecycle.Lifecycle</role>
      <implementation>org.apache.maven.lifecycle.Lifecycle</implementation>
      <role-hint>site</role-hint>
      <configuration>
        <id>site</id>
        
        <phases>
          <phase>pre-site</phase>
          <phase>site</phase>
          <phase>post-site</phase>
          <phase>site-deploy</phase>
        </phases>
        <default-phases>
          <site>
            org.apache.maven.plugins:maven-site-plugin:3.3:site
          </site>
          <site-deploy>
            org.apache.maven.plugins:maven-site-plugin:3.3:deploy
          </site-deploy>
        </default-phases>
        
      </configuration>
    </component><component>
      <role>org.sonatype.plexus.components.sec.dispatcher.SecDispatcher</role>
      <role-hint>maven</role-hint>
      <implementation>org.sonatype.plexus.components.sec.dispatcher.DefaultSecDispatcher</implementation>
      <description>Maven Security dispatcher</description>
      <requirements>
        <requirement>
          <role>org.sonatype.plexus.components.cipher.PlexusCipher</role>
          <field-name>_cipher</field-name>
        </requirement>
        <requirement>
          <role>org.sonatype.plexus.components.sec.dispatcher.PasswordDecryptor</role>
          <field-name>_decryptors</field-name>
        </requirement>
      </requirements>
      <configuration>
        <_configuration-file>~/.m2/settings-security.xml</_configuration-file>
      </configuration>
    </component><component>
      <role>org.apache.maven.artifact.handler.ArtifactHandler</role>
      <role-hint>pom</role-hint>
      <implementation>org.apache.maven.artifact.handler.DefaultArtifactHandler</implementation>
      <configuration>
        <type>pom</type>
      </configuration>
    </component><component>
      <role>org.apache.maven.artifact.handler.ArtifactHandler</role>
      <role-hint>jar</role-hint>
      <implementation>org.apache.maven.artifact.handler.DefaultArtifactHandler</implementation>
      <configuration>
        <type>jar</type>
        <language>java</language>
        <addedToClasspath>true</addedToClasspath>
      </configuration>
    </component><component>
      <role>org.apache.maven.artifact.handler.ArtifactHandler</role>
      <role-hint>ejb</role-hint>
      <implementation>org.apache.maven.artifact.handler.DefaultArtifactHandler</implementation>
      <configuration>
        <type>ejb</type>
        <extension>jar</extension>
        <language>java</language>
        <addedToClasspath>true</addedToClasspath>
      </configuration>
    </component><component>
      <role>org.apache.maven.artifact.handler.ArtifactHandler</role>
      <role-hint>ejb-client</role-hint>
      <implementation>org.apache.maven.artifact.handler.DefaultArtifactHandler</implementation>
      <configuration>
        <type>ejb-client</type>
        <extension>jar</extension>
        <packaging>ejb</packaging>
        <classifier>client</classifier>
        <language>java</language>
        <addedToClasspath>true</addedToClasspath>
      </configuration>
    </component><component>
      <role>org.apache.maven.artifact.handler.ArtifactHandler</role>
      <role-hint>test-jar</role-hint>
      <implementation>org.apache.maven.artifact.handler.DefaultArtifactHandler</implementation>
      <configuration>
        <classifier>tests</classifier>
        <extension>jar</extension>
        <type>test-jar</type>
        <packaging>jar</packaging>
        <language>java</language>
        <addedToClasspath>true</addedToClasspath>
      </configuration>
    </component><component>
      <role>org.apache.maven.artifact.handler.ArtifactHandler</role>
      <role-hint>maven-plugin</role-hint>
      <implementation>org.apache.maven.artifact.handler.DefaultArtifactHandler</implementation>
      <configuration>
        <type>maven-plugin</type>
        <extension>jar</extension>
        <language>java</language>
        <addedToClasspath>true</addedToClasspath>
      </configuration>
    </component><component>
      <role>org.apache.maven.artifact.handler.ArtifactHandler</role>
      <role-hint>java-source</role-hint>
      <implementation>org.apache.maven.artifact.handler.DefaultArtifactHandler</implementation>
      <configuration>
        <classifier>sources</classifier>
        <type>java-source</type>
        <extension>jar</extension>
        <language>java</language>
        <addedToClasspath>false</addedToClasspath>
      </configuration>
    </component><component>
      <role>org.apache.maven.artifact.handler.ArtifactHandler</role>
      <role-hint>javadoc</role-hint>
      <implementation>org.apache.maven.artifact.handler.DefaultArtifactHandler</implementation>
      <configuration>
        <classifier>javadoc</classifier>
        <type>javadoc</type>
        <extension>jar</extension>
        <language>java</language>
        <addedToClasspath>true</addedToClasspath>
      </configuration>
    </component><component>
      <role>org.apache.maven.artifact.handler.ArtifactHandler</role>
      <role-hint>war</role-hint>
      <implementation>org.apache.maven.artifact.handler.DefaultArtifactHandler</implementation>
      <configuration>
        <type>war</type>
        <includesDependencies>true</includesDependencies>
        <language>java</language>
        <addedToClasspath>false</addedToClasspath>
      </configuration>
    </component><component>
      <role>org.apache.maven.artifact.handler.ArtifactHandler</role>
      <role-hint>ear</role-hint>
      <implementation>org.apache.maven.artifact.handler.DefaultArtifactHandler</implementation>
      <configuration>
        <type>ear</type>
        <includesDependencies>true</includesDependencies>
        <language>java</language>
        <addedToClasspath>false</addedToClasspath>
      </configuration>
    </component><component>
      <role>org.apache.maven.artifact.handler.ArtifactHandler</role>
      <role-hint>rar</role-hint>
      <implementation>org.apache.maven.artifact.handler.DefaultArtifactHandler</implementation>
      <configuration>
        <type>rar</type>
        <includesDependencies>true</includesDependencies>
        <language>java</language>
        <addedToClasspath>false</addedToClasspath>
      </configuration>
    </component><component>
      <role>org.apache.maven.artifact.factory.ArtifactFactory</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.artifact.factory.DefaultArtifactFactory</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.artifact.handler.manager.ArtifactHandlerManager</role>
          <field-name>artifactHandlerManager</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.artifact.handler.ArtifactHandler</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.artifact.handler.DefaultArtifactHandler</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
    </component><component>
      <role>org.apache.maven.artifact.handler.manager.ArtifactHandlerManager</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.artifact.handler.manager.DefaultArtifactHandlerManager</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.artifact.handler.ArtifactHandler</role>
          <field-name>artifactHandlers</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.artifact.repository.layout.ArtifactRepositoryLayout</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.artifact.repository.layout.DefaultRepositoryLayout</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
    </component><component>
      <role>org.apache.maven.artifact.repository.metadata.io.MetadataReader</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.artifact.repository.metadata.io.DefaultMetadataReader</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
    </component><component>
      <role>org.apache.maven.artifact.resolver.ResolutionErrorHandler</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.artifact.resolver.DefaultResolutionErrorHandler</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
    </component><component>
      <role>org.apache.maven.bridge.MavenRepositorySystem</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.bridge.MavenRepositorySystem</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.artifact.handler.manager.ArtifactHandlerManager</role>
          <field-name>artifactHandlerManager</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.artifact.repository.layout.ArtifactRepositoryLayout</role>
          <field-name>layouts</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.configuration.BeanConfigurator</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.configuration.internal.DefaultBeanConfigurator</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
    </component><component>
      <role>org.apache.maven.Maven</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.DefaultMaven</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <field-name>logger</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.project.ProjectBuilder</role>
          <field-name>projectBuilder</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.LifecycleStarter</role>
          <field-name>lifecycleStarter</field-name>
        </requirement>
        <requirement>
          <role>org.codehaus.plexus.PlexusContainer</role>
          <field-name>container</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.ExecutionEventCatapult</role>
          <field-name>eventCatapult</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.LegacySupport</role>
          <field-name>legacySupport</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.session.scope.internal.SessionScope</role>
          <field-name>sessionScope</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.internal.aether.DefaultRepositorySystemSessionFactory</role>
          <field-name>repositorySessionFactory</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.graph.GraphBuilder</role>
          <role-hint>graphBuilder</role-hint>
          <field-name>graphBuilder</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.ProjectDependenciesResolver</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.DefaultProjectDependenciesResolver</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.repository.RepositorySystem</role>
          <field-name>repositorySystem</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.artifact.resolver.ResolutionErrorHandler</role>
          <field-name>resolutionErrorHandler</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.eventspy.internal.EventSpyDispatcher</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.eventspy.internal.EventSpyDispatcher</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <field-name>logger</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.eventspy.EventSpy</role>
          <field-name>eventSpies</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.exception.ExceptionHandler</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.exception.DefaultExceptionHandler</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
    </component><component>
      <role>org.apache.maven.graph.GraphBuilder</role>
      <role-hint>graphBuilder</role-hint>
      <implementation>org.apache.maven.graph.DefaultGraphBuilder</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <field-name>logger</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.project.ProjectBuilder</role>
          <field-name>projectBuilder</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.lifecycle.LifecycleExecutor</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.lifecycle.DefaultLifecycleExecutor</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.lifecycle.LifeCyclePluginAnalyzer</role>
          <field-name>lifeCyclePluginAnalyzer</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.DefaultLifecycles</role>
          <field-name>defaultLifeCycles</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.LifecycleTaskSegmentCalculator</role>
          <field-name>lifecycleTaskSegmentCalculator</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.LifecycleExecutionPlanCalculator</role>
          <field-name>lifecycleExecutionPlanCalculator</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.MojoExecutor</role>
          <field-name>mojoExecutor</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.LifecycleStarter</role>
          <field-name>lifecycleStarter</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.MojoDescriptorCreator</role>
          <field-name>mojoDescriptorCreator</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.lifecycle.DefaultLifecycles</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.lifecycle.DefaultLifecycles</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.lifecycle.Lifecycle</role>
          <field-name>lifecycles</field-name>
        </requirement>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <field-name>logger</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.lifecycle.internal.builder.BuilderCommon</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.lifecycle.internal.builder.BuilderCommon</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.LifecycleDebugLogger</role>
          <field-name>lifecycleDebugLogger</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.LifecycleExecutionPlanCalculator</role>
          <field-name>lifeCycleExecutionPlanCalculator</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.ExecutionEventCatapult</role>
          <field-name>eventCatapult</field-name>
        </requirement>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <field-name>logger</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.lifecycle.internal.builder.Builder</role>
      <role-hint>multithreaded</role-hint>
      <implementation>org.apache.maven.lifecycle.internal.builder.multithreaded.MultiThreadedBuilder</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <field-name>logger</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.LifecycleModuleBuilder</role>
          <field-name>lifecycleModuleBuilder</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.lifecycle.internal.builder.Builder</role>
      <role-hint>singlethreaded</role-hint>
      <implementation>org.apache.maven.lifecycle.internal.builder.singlethreaded.SingleThreadedBuilder</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.LifecycleModuleBuilder</role>
          <field-name>lifecycleModuleBuilder</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.lifecycle.internal.BuildListCalculator</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.lifecycle.internal.BuildListCalculator</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
    </component><component>
      <role>org.apache.maven.lifecycle.internal.ExecutionEventCatapult</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.lifecycle.internal.DefaultExecutionEventCatapult</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
    </component><component>
      <role>org.apache.maven.lifecycle.internal.LifecycleExecutionPlanCalculator</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.lifecycle.internal.DefaultLifecycleExecutionPlanCalculator</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.plugin.version.PluginVersionResolver</role>
          <field-name>pluginVersionResolver</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.BuildPluginManager</role>
          <field-name>pluginManager</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.DefaultLifecycles</role>
          <field-name>defaultLifeCycles</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.MojoDescriptorCreator</role>
          <field-name>mojoDescriptorCreator</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.LifecyclePluginResolver</role>
          <field-name>lifecyclePluginResolver</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.LifecycleMappingDelegate</role>
          <role-hint>default</role-hint>
          <field-name>standardDelegate</field-name>
        </requirement>
        <requirement>
          <role>java.util.Map</role>
          <field-name>delegates</field-name>
        </requirement>
        <requirement>
          <role>java.util.Map</role>
          <field-name>mojoExecutionConfigurators</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.lifecycle.LifecycleMappingDelegate</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.lifecycle.internal.DefaultLifecycleMappingDelegate</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.plugin.BuildPluginManager</role>
          <field-name>pluginManager</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.lifecycle.LifeCyclePluginAnalyzer</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.lifecycle.internal.DefaultLifecyclePluginAnalyzer</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.lifecycle.mapping.LifecycleMapping</role>
          <field-name>lifecycleMappings</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.DefaultLifecycles</role>
          <field-name>defaultLifeCycles</field-name>
        </requirement>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <field-name>logger</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.lifecycle.internal.LifecycleTaskSegmentCalculator</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.lifecycle.internal.DefaultLifecycleTaskSegmentCalculator</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.MojoDescriptorCreator</role>
          <field-name>mojoDescriptorCreator</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.LifecyclePluginResolver</role>
          <field-name>lifecyclePluginResolver</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.lifecycle.MojoExecutionConfigurator</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.lifecycle.internal.DefaultMojoExecutionConfigurator</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
    </component><component>
      <role>org.apache.maven.lifecycle.internal.LifecycleDebugLogger</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.lifecycle.internal.LifecycleDebugLogger</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <field-name>logger</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.lifecycle.internal.LifecycleModuleBuilder</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.lifecycle.internal.LifecycleModuleBuilder</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.MojoExecutor</role>
          <field-name>mojoExecutor</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.builder.BuilderCommon</role>
          <field-name>builderCommon</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.ExecutionEventCatapult</role>
          <field-name>eventCatapult</field-name>
        </requirement>
        <requirement>
          <role>java.util.List</role>
          <field-name>projectExecutionListeners</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.session.scope.internal.SessionScope</role>
          <field-name>sessionScope</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.lifecycle.internal.LifecyclePluginResolver</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.lifecycle.internal.LifecyclePluginResolver</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.plugin.version.PluginVersionResolver</role>
          <field-name>pluginVersionResolver</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.lifecycle.internal.LifecycleStarter</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.lifecycle.internal.LifecycleStarter</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.ExecutionEventCatapult</role>
          <field-name>eventCatapult</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.DefaultLifecycles</role>
          <field-name>defaultLifeCycles</field-name>
        </requirement>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <field-name>logger</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.BuildListCalculator</role>
          <field-name>buildListCalculator</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.LifecycleDebugLogger</role>
          <field-name>lifecycleDebugLogger</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.LifecycleTaskSegmentCalculator</role>
          <field-name>lifecycleTaskSegmentCalculator</field-name>
        </requirement>
        <requirement>
          <role>java.util.Map</role>
          <field-name>builders</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.session.scope.internal.SessionScope</role>
          <field-name>sessionScope</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.lifecycle.internal.MojoDescriptorCreator</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.lifecycle.internal.MojoDescriptorCreator</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <field-name>logger</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.version.PluginVersionResolver</role>
          <field-name>pluginVersionResolver</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.BuildPluginManager</role>
          <field-name>pluginManager</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.prefix.PluginPrefixResolver</role>
          <field-name>pluginPrefixResolver</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.LifecyclePluginResolver</role>
          <field-name>lifecyclePluginResolver</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.lifecycle.internal.MojoExecutor</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.lifecycle.internal.MojoExecutor</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.plugin.BuildPluginManager</role>
          <field-name>pluginManager</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.MavenPluginManager</role>
          <field-name>mavenPluginManager</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.LifecycleDependencyResolver</role>
          <field-name>lifeCycleDependencyResolver</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.lifecycle.internal.ExecutionEventCatapult</role>
          <field-name>eventCatapult</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.model.plugin.LifecycleBindingsInjector</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.model.plugin.DefaultLifecycleBindingsInjector</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.lifecycle.LifeCyclePluginAnalyzer</role>
          <field-name>lifecycle</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.plugin.BuildPluginManager</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.plugin.DefaultBuildPluginManager</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.plugin.MavenPluginManager</role>
          <field-name>mavenPluginManager</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.LegacySupport</role>
          <field-name>legacySupport</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.execution.scope.internal.MojoExecutionScope</role>
          <field-name>scope</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.execution.MojoExecutionListener</role>
          <field-name>mojoExecutionListeners</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.plugin.ExtensionRealmCache</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.plugin.DefaultExtensionRealmCache</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
    </component><component>
      <role>org.apache.maven.plugin.PluginArtifactsCache</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.plugin.DefaultPluginArtifactsCache</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
    </component><component>
      <role>org.apache.maven.plugin.PluginDescriptorCache</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.plugin.DefaultPluginDescriptorCache</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
    </component><component>
      <role>org.apache.maven.plugin.PluginRealmCache</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.plugin.DefaultPluginRealmCache</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
    </component><component>
      <role>org.apache.maven.plugin.LegacySupport</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.plugin.internal.DefaultLegacySupport</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
    </component><component>
      <role>org.apache.maven.plugin.MavenPluginManager</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.plugin.internal.DefaultMavenPluginManager</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <field-name>logger</field-name>
        </requirement>
        <requirement>
          <role>org.codehaus.plexus.logging.LoggerManager</role>
          <field-name>loggerManager</field-name>
        </requirement>
        <requirement>
          <role>org.codehaus.plexus.PlexusContainer</role>
          <field-name>container</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.classrealm.ClassRealmManager</role>
          <field-name>classRealmManager</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.PluginDescriptorCache</role>
          <field-name>pluginDescriptorCache</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.PluginRealmCache</role>
          <field-name>pluginRealmCache</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.internal.PluginDependenciesResolver</role>
          <field-name>pluginDependenciesResolver</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.rtinfo.RuntimeInformation</role>
          <field-name>runtimeInformation</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.ExtensionRealmCache</role>
          <field-name>extensionRealmCache</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.version.PluginVersionResolver</role>
          <field-name>pluginVersionResolver</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.PluginArtifactsCache</role>
          <field-name>pluginArtifactsCache</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.plugin.internal.PluginDependenciesResolver</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.plugin.internal.DefaultPluginDependenciesResolver</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <field-name>logger</field-name>
        </requirement>
        <requirement>
          <role>org.eclipse.aether.RepositorySystem</role>
          <field-name>repoSystem</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.plugin.PluginManager</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.plugin.internal.DefaultPluginManager</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.PlexusContainer</role>
          <field-name>container</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.MavenPluginManager</role>
          <field-name>pluginManager</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.version.PluginVersionResolver</role>
          <field-name>pluginVersionResolver</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.prefix.PluginPrefixResolver</role>
          <field-name>pluginPrefixResolver</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.LegacySupport</role>
          <field-name>legacySupport</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.plugin.prefix.PluginPrefixResolver</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.plugin.prefix.internal.DefaultPluginPrefixResolver</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <field-name>logger</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.BuildPluginManager</role>
          <field-name>pluginManager</field-name>
        </requirement>
        <requirement>
          <role>org.eclipse.aether.RepositorySystem</role>
          <field-name>repositorySystem</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.artifact.repository.metadata.io.MetadataReader</role>
          <field-name>metadataReader</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.plugin.version.PluginVersionResolver</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.plugin.version.internal.DefaultPluginVersionResolver</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <field-name>logger</field-name>
        </requirement>
        <requirement>
          <role>org.eclipse.aether.RepositorySystem</role>
          <field-name>repositorySystem</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.artifact.repository.metadata.io.MetadataReader</role>
          <field-name>metadataReader</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.MavenPluginManager</role>
          <field-name>pluginManager</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.project.artifact.MavenMetadataCache</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.project.artifact.DefaultMavenMetadataCache</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
    </component><component>
      <role>org.apache.maven.artifact.metadata.ArtifactMetadataSource</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.project.artifact.DefaultMetadataSource</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.artifact.repository.metadata.RepositoryMetadataManager</role>
          <field-name>repositoryMetadataManager</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.artifact.factory.ArtifactFactory</role>
          <field-name>repositorySystem</field-name>
        </requirement>
        <requirement>
          <role>org.codehaus.plexus.PlexusContainer</role>
          <field-name>container</field-name>
        </requirement>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <field-name>logger</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.project.artifact.MavenMetadataCache</role>
          <field-name>cache</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.LegacySupport</role>
          <field-name>legacySupport</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.project.artifact.ProjectArtifactsCache</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.project.artifact.DefaultProjectArtifactsCache</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
    </component><component>
      <role>org.apache.maven.artifact.metadata.ArtifactMetadataSource</role>
      <role-hint>maven</role-hint>
      <implementation>org.apache.maven.project.artifact.MavenMetadataSource</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.artifact.repository.metadata.RepositoryMetadataManager</role>
          <field-name>repositoryMetadataManager</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.artifact.factory.ArtifactFactory</role>
          <field-name>repositorySystem</field-name>
        </requirement>
        <requirement>
          <role>org.codehaus.plexus.PlexusContainer</role>
          <field-name>container</field-name>
        </requirement>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <field-name>logger</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.project.artifact.MavenMetadataCache</role>
          <field-name>cache</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.LegacySupport</role>
          <field-name>legacySupport</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.project.MavenProjectHelper</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.project.DefaultMavenProjectHelper</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.artifact.handler.manager.ArtifactHandlerManager</role>
          <field-name>artifactHandlerManager</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.project.ProjectBuilder</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.project.DefaultProjectBuilder</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <field-name>logger</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.model.building.ModelBuilder</role>
          <field-name>modelBuilder</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.model.building.ModelProcessor</role>
          <field-name>modelProcessor</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.project.ProjectBuildingHelper</role>
          <field-name>projectBuildingHelper</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.bridge.MavenRepositorySystem</role>
          <field-name>repositorySystem</field-name>
        </requirement>
        <requirement>
          <role>org.eclipse.aether.RepositorySystem</role>
          <field-name>repoSystem</field-name>
        </requirement>
        <requirement>
          <role>org.eclipse.aether.impl.RemoteRepositoryManager</role>
          <field-name>repositoryManager</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.project.ProjectDependenciesResolver</role>
          <field-name>dependencyResolver</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.project.ProjectBuildingHelper</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.project.DefaultProjectBuildingHelper</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <field-name>logger</field-name>
        </requirement>
        <requirement>
          <role>org.codehaus.plexus.PlexusContainer</role>
          <field-name>container</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.classrealm.ClassRealmManager</role>
          <field-name>classRealmManager</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.project.ProjectRealmCache</role>
          <field-name>projectRealmCache</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.repository.RepositorySystem</role>
          <field-name>repositorySystem</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.plugin.MavenPluginManager</role>
          <field-name>pluginManager</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.project.ProjectDependenciesResolver</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.project.DefaultProjectDependenciesResolver</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <field-name>logger</field-name>
        </requirement>
        <requirement>
          <role>org.eclipse.aether.RepositorySystem</role>
          <field-name>repoSystem</field-name>
        </requirement>
        <requirement>
          <role>java.util.List</role>
          <field-name>decorators</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.project.ProjectRealmCache</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.project.DefaultProjectRealmCache</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
    </component><component>
      <role>org.apache.maven.rtinfo.RuntimeInformation</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.rtinfo.internal.DefaultRuntimeInformation</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <field-name>logger</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.settings.MavenSettingsBuilder</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.settings.DefaultMavenSettingsBuilder</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.apache.maven.settings.building.SettingsBuilder</role>
          <field-name>settingsBuilder</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.toolchain.ToolchainManager</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.toolchain.DefaultToolchainManager</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <field-name>logger</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.toolchain.ToolchainFactory</role>
          <field-name>factories</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.toolchain.ToolchainManagerPrivate</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.toolchain.DefaultToolchainManagerPrivate</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <field-name>logger</field-name>
        </requirement>
        <requirement>
          <role>org.apache.maven.toolchain.ToolchainFactory</role>
          <field-name>factories</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.toolchain.ToolchainsBuilder</role>
      <role-hint>default</role-hint>
      <implementation>org.apache.maven.toolchain.DefaultToolchainsBuilder</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <field-name>logger</field-name>
        </requirement>
      </requirements>
    </component><component>
      <role>org.apache.maven.toolchain.ToolchainFactory</role>
      <role-hint>jdk</role-hint>
      <implementation>org.apache.maven.toolchain.java.JavaToolchainFactory</implementation>
      <description />
      <isolated-realm>false</isolated-realm>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.logging.Logger</role>
          <field-name>logger</field-name>
        </requirement>
      </requirements>
    </component></components>
</component-set>
```

