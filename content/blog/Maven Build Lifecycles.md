---
title: "Maven Build Lifecycles"
date: 2019-12-31T12:14:34+06:00
description: "Maven's Build Lifecycles in a nutshell"
author: "Sajad Hayatlou"
type: "post"
---

A Build Lifecycle is Made Up of Phases

{{< figure src="https://www.wikitechy.com/tutorials/maven/img/maven-images/maven-default-lifecycle-phases.png" 
  width=300 height=300 >}}

Each of these build lifecycles is defined by a different list of build phases, wherein a build phase represents a stage in the lifecycle.

For example, the default lifecycle comprises of the following phases (for a complete list of the lifecycle phases, refer to the Lifecycle Reference):

   * `validate` – validate the project is correct and all necessary information is available
   * `compile` – compile the source code of the project
   * `test` – test the compiled source code using a suitable unit testing framework. These tests should not require the code be packaged or deployed
   * `package` – take the compiled code and package it in its distributable format, such as a JAR.
   * `verify` – run any checks on results of integration tests to ensure quality criteria are met
   * `install` – install the package into the local repository, for use as a dependency in other projects locally
   * `deploy` – done in the build environment, copies the final package to the remote repository for sharing with other developers and projects.






