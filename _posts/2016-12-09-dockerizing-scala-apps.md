---
layout: post
title: Dockerizing Scala apps. Notes.
comments: true
---

At some point I realized that building and shipping my software with deb packages sucks a little bit (especially managing a deb repo on a small scale). I have been looking at docker and containers for a some time already and it seemed to be a great idea. When I started working on my new home backup system I decided to give it a try. Since the app I was building was a Scala app, the learning curve was a little bit steeper than for other people. Along the way I noticed following things:

* docker file can be automatically created in couple ways: by using the `sbt-docker` plugin, the `sbt-native-packager` plugin or created manually
* gitlab hosts its own docker image registry and you can use it for free (yay!)
* gitlab offers also CI pipelines what makes it super easy to build everything after a push and then automatically deploy to infra

The `sbt-native-packager` is probably the right way to go in my case as I just want something working and don't want to configure a lot of things (and this plugin's philosophy is exactly that). For more complex setups probably `sbt-docker` is better.

The changes I had to make in build configuration to get the Dockerfile generated correctly:

* get correct imports: add `import com.typesafe.sbt.packager.docker.DockerPlugin.autoImport._` 
* enable the plugin: add `enablePlugins(DockerPlugin)`
* customize the dockerfile (add the keys from autoImport to settings sequence):

```scala
    ...
    dockerExposedPorts := Seq(8080),
    dockerUpdateLatest := true,
    ...
```

There are some hacks which are not obvious (and helped in my scenario):

* adding commands in the middle of the dockerfile:

    ```scala
    dockerCommands := {
      val (left, right) = dockerCommands.value.splitAt(3)

      left ++ Seq(
        ExecCmd("RUN", "mkdir", "-p", "/directory/path"),
      ) ++ right
    },
    ```

* pushing to gitlab registry requires some hacks:

    ```scala
    dockerRepository := Some("registry.gitlab.com/..."), // set the repository url
    packageName in Docker := "project-name", // UUUGLY hack - do this if your repo name is different than project name in sbt
    ```

* if you're using a private repository (as I do) remember to include a `docker login ...` somewhere before `./sbt docker:publish` in the build pipeline
* when you want to access the image from outside, remember to prefix it with the private repo name!

    ```bash
    $ docker pull registry.gitlab.com/username/image:tag
    ```

Links:

* [sbt-native-packager - docker plugin](http://www.scala-sbt.org/sbt-native-packager/formats/docker.html)
* [sbt-docker](https://github.com/marcuslonnberg/sbt-docker)
* [Blog post with a lot of context on sbt-docker](http://velvia.github.io/Docker-Scala-Sbt/)
