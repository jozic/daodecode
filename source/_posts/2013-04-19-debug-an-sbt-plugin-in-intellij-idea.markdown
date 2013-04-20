---
layout: post
title: "Debug an sbt plugin in IntelliJ IDEA"
date: 2013-04-19 12:51
comments: true
categories: [scala, sbt, idea]
---
So you have your own sbt plugin (or someone's plugin if you are not so cool and sexy) and you want to debug it.
It happens that there is nothing easier than that. If you run sbt through [sbt-extras](https://github.com/paulp/sbt-extras) script (if you don't you should)
just start your test sbt app with the plugin added to the build with the following command
```
sbt -jvm-debug 5005
```

You can use any port, I use 5005 because it's a default port for remote debug configuration in [IntelliJ IDEA](http://www.jetbrains.com/idea/) which I use for scala development.
And if you are cool and sexy, you know... :)

This is how your sbt console should look like after you start it:
```
jozic@laptop ~/projects/sbt-about-plugins $ sbt -jvm-debug 5005
Detected sbt version 0.12.3
Starting sbt: invoke with -help for other options
Listening for transport dt_socket at address: 5005
[info] Loading global plugins from /home/jozic/.sbt/plugins
[info] Loading project definition from /home/jozic/projects/sbt-about-plugins/project
[info] Set current project to sbt-about-plugins (in build file:/home/jozic/projects/sbt-about-plugins/)
>
```

Then open your project in IDEA (you can generate IDEA project files using
[sbt-idea](https://github.com/mpeltonen/sbt-idea) plugin) and create remote debug configuration.
{% img /images/sbt-debug/idea.debug.configuration.png %}
You can change port, but 5005 will do in our case, so you literaly can save default configuration.
Run it and you should get the following line in IDEA console:
```
Connected to the target VM, address: 'localhost:5005', transport: 'socket'
```

After that put a breakpoint inside your plugin command, switch to sbt console, run your command and here you go!
{% img /images/sbt-debug/idea.breakpoint.in.action.png %}


