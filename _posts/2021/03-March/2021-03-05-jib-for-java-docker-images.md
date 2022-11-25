---
layout: post
title: Man the Jib, ya scurvy lot!
subtitle: Using Jib to package Java apps (with Maven)
tags: [devops, java, docker]
published: true
---

Jib is a tool for Docker Java image creation, maintained by Google to package your Java applications from source to image.  Ideally you're using Spring Boot, but at least using Maven (or Gradle I guess) and building fat-jars that contain your source code and dependencies in one package.  Traditionally to make that step into Docker images required a Dockerfile:

```
FROM openjdk:8-jdk-alpine
COPY target/java-application-1.0.0.jar java-application-1.0.0.jar
ENTRYPOINT ["java","-jar","/java-application-1.0.0.jar"]
```

It can be as easy as that, but doesn't really address a litany list of better practices that should be followed.  Just off the top:

- unless specified, the image will run as root user which is a security nightmare in most cases
- if your application has a large number of dependencies, but relatively little in terms of source code, then second layer (the COPY one above) is going to include some files that change a lot (source) and don't change often (dependencies)
- assuming your application needs properties that might change between environments (like dev, test, prod) then you need different packages for each env, assuming that you don't externalize your configuration to things like Consul or Spring Configuration servers.

Right there we're starting to violate the 12-factor application standards for application development, an in ways that aren't easy to address without rebuilding parts of our DevOps pipeline.

That's where [Jib](https://github.com/GoogleContainerTools/jib) comes in.

    Jib builds optimized Docker and OCI images for your Java applications without a Docker daemon - and without deep mastery of Docker best-practices. It is available as plugins for Maven and Gradle and as a Java library.
    - from (https://github.com/GoogleContainerTools/jib)

With the provided [Maven](https://maven.apache.org) or [Gradle](https://gradle.org) plugins, you can easily package your application.  And since it's not using a local Docker daemon, adding this to CI servers become very easy and efficient for building those images.

## Maven Configuration
Adding this to the `pom.xml` will enable the plugin.  Further configuration details can be found [here](https://github.com/GoogleContainerTools/jib/tree/master/jib-maven-plugin#quickstart).  You can adjust things like the `FROM` image, and the only required configuration item is the `TO` image.
```
<project>
  ...
  <build>
    <plugins>
      ...
      <plugin>
        <groupId>com.google.cloud.tools</groupId>
        <artifactId>jib-maven-plugin</artifactId>
        <version>2.8.0</version> # don't forget to check for newest version
        <configuration>
          <to>
            <image>myimage</image>
          </to>
        </configuration>
      </plugin>
      ...
    </plugins>
  </build>
  ...
</project>
```

Once configured you can now run different targets like `jib:build` to push the image to the target repo in the `to` configuration above, `jib:dockerBuild` that will build the image to the local Docker registry.

I would also suggest that you bind the builder to a specific stage of the lifecycle like `package`:

```
<plugin>
  <groupId>com.google.cloud.tools</groupId>
  <artifactId>jib-maven-plugin</artifactId>
  ...
  <executions>
    <execution>
      <phase>package</phase>
      <goals>
        <goal>build</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```

That way ever time `Maven` runs through it's lifecycle, the image build will be triggered.