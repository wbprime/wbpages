---
title: "Maven Note"
date: 2018-02-06T10:50:17+08:00
categories: "Notes"
tags: []
description: "Note on apache maven"
draft: false
---

# Local Repository Location

`conf/setting.xml`

```
<settings ...>
    <localRepository>/path/to/local/repo</localRepository>
</settings>
```

Use `mvn help:system` to confirm it.

# Create New Project

`mvn archetype:create -DgroupId=me.wbprime.test -DartifactId=hello -DpackageName=me.wbprime.test -Dversion=1.0`

# Maven Dependency

`mvn dependency:copy-dependencies -DoutputDirectory=lib`

# Lifecycles

Three builtin lifecycles:

- default
- clean 
- site

## "clean" Lifecycle

| phase      | description                                                   |
| ---        | ---                                                           |
| pre-clean  | execute processes needed prior to the actual project cleaning |
| clean      | remove all files generated by the previous build              |
| post-clean | execute processes needed to finalize the project cleaning     |

## "site" Lifecycle

| phase       | description                                                          |
| ---         | ---                                                                  |
| pre-site    | execute processes needed prior to the actual project site generation |
| site        | generate the project's site documentation                            |
| post-site   | execute processes needed to finalize the site generation, and to     |
| prepare     | for site deployment                                                  |
| site-deploy | deploy the generated site documentation to the specified web server  |

## "default" Lifecycle

| phase                   | description                                                                                                                                                                   |
| ---                     | ---                                                                                                                                                                           |
| validate                | validate the project is correct and all necessary information is available.                                                                                                   |
| initialize              | initialize build state, e.g. set properties or create directories.                                                                                                            |
| generate-sources        | generate any source code for inclusion in compilation.                                                                                                                        |
| process-sources         | process the source code, for example to filter any values.                                                                                                                    |
| generate-resources      | generate resources for inclusion in the package.                                                                                                                              |
| process-resources       | copy and process the resources into the destination directory, ready for packaging.                                                                                           |
| compile                 | compile the source code of the project.                                                                                                                                       |
| process-classes         | post-process the generated files from compilation, for example to do bytecode enhancement on Java classes.                                                                    |
| generate-test-sources   | generate any test source code for inclusion in compilation.                                                                                                                   |
| process-test-sources    | process the test source code, for example to filter any values.                                                                                                               |
| generate-test-resources | create resources for testing.                                                                                                                                                 |
| process-test-resources  | copy and process the resources into the test destination directory.                                                                                                           |
| test-compile            | compile the test source code into the test destination directory                                                                                                              |
| process-test-classes    | post-process the generated files from test compilation, for example to do bytecode enhancement on Java classes. For Maven 2.0.5 and above.                                    |
| test                    | run tests using a suitable unit testing framework. These tests should not require the code be packaged or deployed.                                                           |
| prepare-package         | perform any operations necessary to prepare a package before the actual packaging. This often results in an unpacked, processed version of the package. (Maven 2.1 and above) |
| package                 | take the compiled code and package it in its distributable format, such as a JAR.                                                                                             |
| pre-integration-test    | perform actions required before integration tests are executed. This may involve things such as setting up the required environment.                                          |
| integration-test        | process and deploy the package if necessary into an environment where integration tests can be run.                                                                           |
| post-integration-test   | perform actions required after integration tests have been executed. This may including cleaning up the environment.                                                          |
| verify                  | run any checks to verify the package is valid and meets quality criteria.                                                                                                     |
| install                 | install the package into the local repository, for use as a dependency in other projects locally.                                                                             |
| deploy                  | done in an integration or release environment, copies the final package to the remote repository for sharing with other developers and projects.                              |

# Lifecycle Bindings

## "clean" Lifecycle

| phase | goal        |
| ---   | ---         |
| clean | clean:clean |

## "site" Lifecycle

| phase       | goal        |
| ---         | ---         |
| site        | site:site   |
| site-deploy | site:deploy |

## "default" Lifecycle

### Packaging "pom"

| phase   | goal                   |
| ---     | ---                    |
| package | site:attach-descriptor |
| install | install:install        |
| deploy  | deploy:deploy          |

### Packaging "ejb" / "ejb3" / "jar" / "par" / "rar" / "war"

| phase                  | goal                                                             |
| ---                    | ---                                                              |
| process-resources      | resources:resources                                              |
| compile                | compiler:compile                                                 |
| process-test-resources | resources:testResources                                          |
| test-compile           | compiler:testCompile                                             |
| test                   | surefire:test                                                    |
| package                | ejb:ejb or ejb3:ejb3 or jar:jar or par:par or rar:rar or war:war |
| install                | install:install                                                  |
| deploy                 | deploy:deploy                                                    |

### Packaging "ear"

| phase              | goal                         |
| ---                | ---                          |
| generate-resources | ear:generate-application-xml |
| process-resources  | resources:resources          |
| package            | ear:ear                      |
| install            | install:install              |
| deploy             | deploy:deploy                |

### Packaging "maven-plugin"

| phase                  | goal                                         |
| ---                    | ---                                          |
| generate-resources     | plugin:descriptor                            |
| process-resources      | resources:resources                          |
| compile                | compiler:compile                             |
| process-test-resources | resources:testResources                      |
| test-compile           | compiler:testCompile                         |
| test                   | surefire:test                                |
| package                | jar:jar and plugin:addPluginArtifactMetadata |
| install                | install:install                              |
| deploy                 | deploy:deploy                                |

# Classloader 

In some operating systems (Windows), there's a limit on how long you can make
your command line, and therefore a limit on how long you can make your
classpath. 

## Generic Solutions 

There are two "tricks" you can use to workaround this problem; both of them can cause other problems in some cases.

1. Isolated Class Loader: One workaround is to use an isolated class loader. Instead of launching MyApp directly, we can launch some other application (a "booter") with a much shorter classpath. We can then create a new java.lang.ClassLoader (usually a java.net.URLClassLoader) with the desired classpath configured. The booter can then load up MyApp from the class loader; when MyApp refers to other classes, they will be automatically loaded from our isolated class loader.

The problem with using an isolated class loader is that your classpath isn't really correct, and some applications can detect this and object. For example, the system property java.class.path won't include your jars; if your application notices this, it could cause a problem.

There's another similar problem with using an isolated class loader: any class may call the static method ClassLoader.getSystemClassLoader() and attempt to load classes out of that class loader, instead of using the default class loader. Classes often do this if they need to create class loaders of their own. Unfortunately, Java-based web application servers like Jetty, Tomcat, BEA WebLogic and IBM WebSphere are very likely to try to escape the confines of an isolated class loader.

2. Manifest-Only JAR: Another workaround is to use a "manifest-only JAR." In this case, you create a temporary JAR that's almost completely empty, except for a META-INF/MANIFEST.MF file. Java manifests can contain attributes that the Java virtual machine will honor as directives. For example, you can have a Class-Path attribute, which specifies a list of other JARs to add to the classpath.

## Advantages/Disadvantages of each Solution

If your application tries to interrogate its own class loader for a list of JARs, it may work better under an isolated class loader than it would with a manifest-only JAR. However, if your application tries to escape its default class loader, it may not work under an isolated class loader at all.

One advantage of using an isolated class loader is that it's the only way to use an isolated class loader without forking a separate process, running all of the tests in the same process as Maven itself. But that itself can be pretty risky, especially if Maven is running embedded in your IDE!

Finally, of course, you could just try to wire up a plain old Java classpath and hope it's short enough. In the worst case your classpath might work on some machines and not others. Windows boxes would behave differently from Linux boxes; users with short user names might have more success than users with long user names, etc. For this reason, we chose not to make the basic classpath the default, though we do provide it as an option (mostly as a last resort).

## Configuration

```
<project>
  [...]
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>2.19.1</version>
        <configuration>
          <useSystemClassLoader>false</useSystemClassLoader>
	      <useManifestOnlyJar>false</useManifestOnlyJar>
        </configuration>
      </plugin>
    </plugins>
  </build>
  [...]
</project>
```

## Update for Maven Surefire 2.8.2

It turns out setting the CLASSPATH as an environment variable may remove most of the practical length limitations, as documented in SUREFIRE-727. This means most of the length-related problems in this article may be outdated.