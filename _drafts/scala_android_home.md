---
layout: post
title: libgdx && android && scala && idea
---

I would like to seriously start learning Scala... So why not try it out to do some Android development? To make it even more funny (... or tragic) lets mix in the [libgdx][2] library. I installed Scala and SBT plugins for IntelliJ and started with [this][1] description for project setup. Everything works perfectly fine when running:

```
curl https://raw.github.com/n8han/conscript/master/setup.sh | sh
cs n8han/giter8
g8 ajhager/libgdx-sbt-project
```

... but the second IntelliJ tries to open the project (or you want to build it with `sbt`) - bang! - a surprise:

```scala
java.lang.RuntimeException: set ANDROID_HOME or run 'android update project -p /home/user/Projects/colors/android'
at scala.sys.package$.error(package.scala:27)
at android.Plugin$$anonfun$allPluginSettings$67$$anonfun$apply$28.apply(rules.scala:350)
at android.Plugin$$anonfun$allPluginSettings$67$$anonfun$apply$28.apply(rules.scala:350)
at scala.Option.getOrElse(Option.scala:120)
at android.Plugin$$anonfun$allPluginSettings$67.apply(rules.scala:350)
at android.Plugin$$anonfun$allPluginSettings$67.apply(rules.scala:342)
at scala.Function2$$anonfun$tupled$1.apply(Function2.scala:54)
at scala.Function2$$anonfun$tupled$1.apply(Function2.scala:53)
at scala.Function1$$anonfun$compose$1.apply(Function1.scala:47)
at sbt.EvaluateSettings$MixedNode.evaluate0(INode.scala:177)
at sbt.EvaluateSettings$INode.evaluate(INode.scala:135)
at sbt.EvaluateSettings$$anonfun$sbt$EvaluateSettings$$submitEvaluate$1.apply$mcV$sp(INode.scala:67)
at sbt.EvaluateSettings.sbt$EvaluateSettings$$run0(INode.scala:76)
at sbt.EvaluateSettings$$anon$3.run(INode.scala:72)
at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145)
at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
at java.lang.Thread.run(Thread.java:745)
[error] set ANDROID_HOME or run 'android update project -p /home/koperek/Projects/colors/android'
[error] Use 'last' for the full log.
Project loading failed: (r)etry, (q)uit, (l)ast, or (i)gnore? 
```

So how to bootstrap a working scala with libgdx project?

*1. Follow the snippet from above.*
2. Define `ANDROID_HOME` in `idea.sh` and `.bashrc`
3. Add `sbt-idea` plugin to `plugins.sbt` (found that hint [here][3]):

```scala
resolvers += "Sonatype snapshots" at "https://oss.sonatype.org/content/repositories/snapshots/"

addSbtPlugin("com.github.mpeltonen" % "sbt-idea" % "1.7.0-SNAPSHOT")
```

**Note: there needs to be an empty line before `sbt-idea` plugin definition.**
**Update:** alternatively just use my own version of template:

```
g8 pkoperek/libgdx-sbt-project
```

4. Run `sbt "project core" gen-idea`
5. Open the project in IntelliJ (not import - this doesn't work - **open**)
5. After opening in IDEA - open `android` module settings. 
5.1. Set the language level to `6.0`.
5.2. Mark `src/main/scala` as `Source Root` and `src/test/scala` as `Test Source Root`

Workflow:
* After adding a new dep remember about `sbt update` and `sbt "project core" gen-idea`
* To run app in emulator, start one with AVD and run `sbt android/android:run`


[1]: http://raintomorrow.cc/post/70000607238/develop-games-in-scala-with-libgdx-getting-started
[2]: http://libgdx.badlogicgames.com 
[3]: https://github.com/ajhager/libgdx-sbt-project.g8/wiki/IDE-Plugins
