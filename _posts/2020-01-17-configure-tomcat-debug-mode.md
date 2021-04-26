---
layout: post
title: Configuring Apache Tomcat in debug mode
description: Let's go through different techniques to configure the tomcat server to work in debug mode
date:  2020-01-17
time: 3 mins
tags: tomcat java
language: english
---

Having a Tomcat server running in debug mode is utterly priceless for a myriad of situations when a deep analysing of the deployed application gets necessary.

There are different ways to enable the debug mode, so let's go through the most common options.

<!-- more -->

## Adding the JPDA Debugger to the startup script

This straightforward configuration will enable the debug mode in your Tomcat instance permanently.

1 . Edit the file `$tomcat/bin/startup.sh` and find the following instruction at the end of the document:

```sh
exec "$PRGDIR"/"$EXECUTABLE" start "$@"
```

2 . Include the parameter `jpda` to the above command ([more info about the JPDA Debugger](https://cwiki.apache.org/confluence/display/TOMCAT/Developing)):

```sh
exec "$PRGDIR"/"$EXECUTABLE" jpda start "$@"
```

3 . Restart the server.

After these steps, you can confirm the tomcat server is running in debug mode by checking the java process includes the debug agent (e.g. `ps aux | grep java`):

```
-agentlib:jdwp=transport=dt_socket,address=8000,server=y,suspend=n
```

Meaning the server is accepting connections to the local port 8000.

### Configuring the JPDA agent

In case the jpda default configuration doesn't sound like your thing, some agent's parameters can be set up separately by using local variables.

```sh
# Unix lovers
export JPDA_ADDRESS=8000
export JPDA_TRANSPORT=dt_socket

# Windows lovers
set JPDA_ADDRESS=8000
set JPDA_TRANSPORT=dt_socket
```

## Adding the JPDA Debugger to the java command

On the other hand, if you don't want to active the jpda debugger through the `startup.sh` file at all, you can include the agent configuration in the `JAVA_OPTS` variable directly:

```sh
JAVA_OPTS="$JAVA_OPTS -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=8000"
```

The result would be exactly the same, plus the flexibility of configuring the agent's parameters freely.
