#What is code (or test) coverage?

Code coverage (or test coverage) shows which lines of the code were (or were not) being executed by the tests. It is also a metric which helps you to find out the percentage of your covered (executed) code by the tests. E.g.: It tells you that your codebase consists of 10 lines, 8 lines were being executed by your tests, so your coverage is 80%.  
(It tells you nothing about the quality of your software or how good your tests are.)

If you want to read more about code coverage, check these:

- [Wiki - Code Coverage](https://en.wikipedia.org/wiki/Code_coverage)
- [Martin Fowler - Test Coverage](http://martinfowler.com/bliki/TestCoverage.html)

##What are the subtypes of test coverage?

Basically, test coverage can be measured for all levels of tests, like unit-, integration-, acceptance tests, etc.
For example, unit test coverage is a subtype of test coverage, it shows which lines of the source code were (or were not) being executed by the **unit tests**.

#How to measure code coverage?

The usual way (at least in the Java world) to measure code coverage is instrumenting the code. Instrumentation is a technique to track the execution of the code at runtime. This can be done in different ways:

- Offline instrumentation
  - Source code modification
  - Byte code manipulation
- On-the-fly instrumentation
  - Using an instrumenting ClassLoader
  - Using a JVM agent

##Offline instrumentation

Offline instrumentation is a technique to modify the source code or the byte code at compile time in order to track the execution of the code at runtime. In practice, this means that the code coverage tool injects data collector calls into your source or byte code to record if a line was executed or not.

###Offline instrumentation example (Clover)

Clover uses source code instrumentation but I only show you the decompiled code because it is easier to get the byte code.
The original method is the following:
```java
public static void main(String[] args) {
    SpringApplication.run(CoverageDemoApplication.class, args);
}
```

After instrumenting-compiling-decompiling the code, we get something like this:
```java
public static void main(String[] var0) {
    CoverageDemoApplication.__CLR4_1_144in3j646t.R.inc(4);
    CoverageDemoApplication.__CLR4_1_144in3j646t.R.inc(5);
    SpringApplication.run(CoverageDemoApplication.class, var0);
}
```

The instrumented code is a bit more sophisticated, here you can check the whole file: [instrumented and decompiled code](https://github.com/jonatan-ivanov/coverage-demo/tree/master/clover-instrumentation)

###Offline instrumentation demo project

I created a [GitHub repo](https://github.com/jonatan-ivanov/coverage-demo) where I hack the Gradle Clover plugin in order to see the instrumented files. So if you want the red pill, I can show you how deep the rabbit hole goes: what the Clover plugin does under the hood and how to get the instrumented class files: [Offline instrumentation demo](https://github.com/jonatan-ivanov/coverage-demo/blob/master/Offline_instrumentation_demo.md)

###On-the-fly instrumentation

This instrumentation process happens on-the-fly during class loading by using a Java Agent or a special Class Loader so the source/byte code remains untouched.

#Picking the *"right"* coverage tool

This question becomes more interesting above unit testing level. For example, in order to run integration tests, you need to:
  1. Start the application
  2. Run the tests
  3. Stop the application

This process is a bit more complicated than running unit tests.  
Suppose that we have a classic, servlet-based web application which runs in Tomcat (or in any other Servlet Container) and we want to measure the integration test coverage.
In this case, the integration testing consists of the following steps:
  1. Compile the code and create the application package (a war file in our case)
  2. Start the app in Tomcat
  3. Run the integration tests and track execution data
  4. Stop the app
  5. Generate report from the collected data

If our coverage tool instruments the code offline, we will end up with two packages: one for the test environments (in order to be able to measure test coverage) and one for non-testing environments (e.g.: production). Having different packages for the same application is something that we really want to avoid.
If our coverage tool instruments the code on-the-fly, we can deploy the same package to each of our environments. The only difference will be the configuration for the different environments, e.g.: an additional JVM option where we configure the coverage tool which seems much more convenient.

##Demo project
I created a [GitHub repo](https://github.com/jonatan-ivanov/coverage-demo) to show this scenario in action. So here is what you need:
- Separate source sets for [unit-](https://github.com/jonatan-ivanov/coverage-demo/tree/master/src/test/java/com/example/controller) and [functional](https://github.com/jonatan-ivanov/coverage-demo/tree/master/src/functionalTest/java/com/example/controller) tests: [`functionalTest.gradle`](https://github.com/jonatan-ivanov/coverage-demo/blob/master/gradle/functionalTest.gradle)
- To use the [JaCoCo plugin](https://docs.gradle.org/current/userguide/jacoco_plugin.html) and get the JaCoCo Agent: [`jacoco.gradle`](https://github.com/jonatan-ivanov/coverage-demo/blob/master/gradle/jacoco.gradle)
- To deploy and undeploy the application using the agent: [`deploy.gradle`](https://github.com/jonatan-ivanov/coverage-demo/blob/master/gradle/deploy.gradle)
- Configure the JaCoCo plugin to create coverage report for unit- and functional tests: [`jacoco.gradle`](https://github.com/jonatan-ivanov/coverage-demo/blob/master/gradle/jacoco.gradle),  [`functionalTest.gradle`](https://github.com/jonatan-ivanov/coverage-demo/blob/master/gradle/functionalTest.gradle)
- Configure the JaCoCo plugin to create a merged report which aggregates the coverage data of the unit- and functional tests: [`functionalTest.gradle`](https://github.com/jonatan-ivanov/coverage-demo/blob/master/gradle/functionalTest.gradle)
- Wire up the tasks (set task dependencies)

So if you clone this repo and run `./gradlew build`, you should see something like this in the console:
```txt
$ ./gradlew build
[...]
:jar
:bootRepackage
:assemble

:compileTestJava
[...]
:test

:unZipJacocoAgent
:deploy
:compileFunctionalTestJava
[...]
:functionalTest
:undeploy

:jacocoFunctionalTestReport
:jacocoTestReport
:jacocoMerge
:mergedReport
:check
:build
```

And you should see three reports in the `build/reports/jacoco` directory. Or in the [resources](https://github.com/jonatan-ivanov/articles/tree/master/Code_coverage_not_only_for_Unit_tests/resources) directory of this repo.

###Covered lines (unit tests, functional tests, merged)
![Covered lines for unit tests](https://raw.githubusercontent.com/jonatan-ivanov/articles/master/Code_coverage_not_only_for_Unit_tests/resources/screenshots/unit-tests-code.png)
![Covered lines for functional tests](https://raw.githubusercontent.com/jonatan-ivanov/articles/master/Code_coverage_not_only_for_Unit_tests/resources/screenshots/functional-tests-code.png)
![Covered lines](https://raw.githubusercontent.com/jonatan-ivanov/articles/master/Code_coverage_not_only_for_Unit_tests/resources/screenshots/merged-code.png)

###Stats (unit tests, functional tests, merged)
![Stats for unit tests](https://raw.githubusercontent.com/jonatan-ivanov/articles/master/Code_coverage_not_only_for_Unit_tests/resources/screenshots/unit-tests-stats.png)
![Stats for functional tests](https://raw.githubusercontent.com/jonatan-ivanov/articles/master/Code_coverage_not_only_for_Unit_tests/resources/screenshots/functional-tests-stats.png)
![Merged stats](https://raw.githubusercontent.com/jonatan-ivanov/articles/master/Code_coverage_not_only_for_Unit_tests/resources/screenshots/merged-stats.png)
