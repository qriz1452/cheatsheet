# Maven

Creating a Project in MVN
```
mvn archetype:generate -DgroupId=org.yourcompany.project -DartifactId=application
```

MVN Command Execution Phases:


MVN command execution is separated into phases. They form the lifecycle of the build.
,
Maven Command Execution Phases
,
```
clean  :   Delete target directory.
validate   :    Validate if the project is correct.
compile   :   Compile source code, classes stored in target/classes.
test   :   Run tests.
package   :   Take the compiled code and package it in its distributable format, e.g. JAR, WAR.
verify   :    Run any checks to verify the MVN package is valid and meets quality criteria.
install   :    Install the package into the local repository.
deploy   :   Copies the final MVN package to the remote repository.
```


Maven Command Line Options : 


```

-DskipTests=true   :    Compiles the tests, but skips running them.
-Dmaven.test.skip=true   :    Skips compiling the tests and does not run them.
-T    :   Specify number of parallel threads involved in the build. If your project can be built in parallel, for example the modules that do not depend on each other, this option, say -T 4 (to use 4 threads) can significantly speed up your build time. Also if you're interested, we've looked into speeding up the Maven build before.
-rf    OR    --resume-from makes   :   Maven resume the build from the specified project, if one invocation of the Maven option fails, you pretty much don't want to repeat the work that succeeded, so you can start from where the build get interrupted.
-pl OR --projects    :   Makes Maven build only specified modules and not the whole project. When you're working in one module of the multi-module project, you know that your changes are localized there, so you'd likely want to avoid doing empty work by building everything else.
-am  OR --also-make   :   Use this syntax to build a Maven project offline. It uses the local repository for everything, and doesn't check the internet.
-o OR  --offline     :   Build a Maven project offline. It uses the local repository for everything, and doesn't check the internet.
-X OR --debug   :   Enables the debug output. When things go wrong, Maven doesn't always print the most useful error messages, enabling the debug can get you the needed hint to what went wrong.
-U  OR --update-snapshots   :   Forces a check for updated dependencies from the remote repositories. When you depend on a project that is still in development, you'll need to depend on the SNAPSHOT versions. And Maven will cache them in the local repository as usual. However, the problem with the snapshots is that one version identifier can mean different artifacts, when a new version of the snapshot is pushed to the repository. To make Maven ignore the local cache, use this option.
```


Run a single test:

Sometimes you would like to execute a single test instead of all your tests.   :  `mvn test -Dtest="NameOfYourTest"`


Detailed cheat sheet : https://medium.com/@TimvanBaarsen/maven-cheat-sheet-45942d8c0b86
Best : https://www.javaguides.net/2018/06/maven-cheat-sheet.html
