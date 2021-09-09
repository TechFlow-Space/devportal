# Creating a workspace

We use gradle to create the workspace. If you are unfamiliar with gradle please check their [documentation](
https://docs.gradle.org/current/userguide/what_is_gradle.html)

## Using CLI
### Create a project folder

Gradle comes with a built-in task, called init, that initializes a new Gradle project in an empty folder. The init task 
uses the (also built-in) wrapper task to create a Gradle wrapper script, gradlew.

The first step is to create a folder for the new project and change directory into it.
```shell
$ mkdir hello-world
$ cd hello-world
```

### Run the init task

```shell
$ gradle init

Select type of project to generate:
  1: basic
  2: application
  3: library
  4: Gradle plugin
Enter selection (default: basic) [1..4] 2

Select implementation language:
  1: C++
  2: Groovy
  3: Java
  4: Kotlin
  5: Scala
  6: Swift
Enter selection (default: Java) [1..6] 3

Split functionality across multiple subprojects?:
  1: no - only one application project
  2: yes - application and library projects
Enter selection (default: no - only one application project) [1..2] 1

Select build script DSL:
  1: Groovy
  2: Kotlin
Enter selection (default: Groovy) [1..2] 1

Select test framework:
  1: JUnit 4
  2: TestNG
  3: Spock
  4: JUnit Jupiter
Enter selection (default: JUnit Jupiter) [1..4] 4

Project name (default: hello-world): 
Source package (default: hello.world): 

> Task :init
Get more help with your project: https://docs.gradle.org/7.2/samples/sample_building_java_applications.html

BUILD SUCCESSFUL in 26s
2 actionable tasks: 2 executed

```

