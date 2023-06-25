# Gradle

How to Compile Gradle project?
gradlew compile 
It will compile the project java files


How to run spring boot project with Gradle**
In the spring boot project, once the Gradle plugin is configured, Application is built using the below command

gradle build



How do you make a jar file for the spring boot application?

gradle bootJar
Here is the command for creating a war file for the same

gradle bootWar
Spring boot application can be started using the bootRun command.




gradle bootRun
This will compile, copy and run the spring boot server, It is not required to build and compile the application.

How to list tasks in a project
all tasks can be listed for the build file using the below command It will display all tasks related to the build file in a project Gradle tasks command lists out all the tasks configured in a project

gradle tasks
gradle tasks --all




Gradle tasks are run with the below command.

gradle taskname



for example, to build the project

gradle build


How to Convert maven project to Gradle project
gradle setupBuild command converts the maven project to gradle build

gradle setupBuild
This read pom.xml configuration and outputs build.gradle file

How to run test case execution in Gradle
test tasks used to compile and run test cases in Gradle projects

gradle test
How to list out dependencies of the Gradle project
the below command list out all dependencies of a project

gradle dependencies
How to run clean gradle build
gradle clean command deletes previous build output. This will help to run a freshly build project.
gradle build


gradle wrapper command adds the wrapper and required jars to the project

gradle wrapper --gradle-version 7.1.1
This adds the following files to the project.

└── <project folder>
    └── gradlew
    └── gradlew.bat
    └── build.gradle
    └── 
    └── 
    └── gradle
        └── wrapper
            └── gradle-wrapper.jar
            └── gradle-wrapper.properties
------------------

Common commands¶
./gradlew tasks in your project directory lists which tasks you can run in your project, such as building or running your code.
./gradlew projects
./gradlew properties
Most commonly used Java tasks:

./gradlew build will compile your project's code into a /build folder.
./gradlew run will run the compiled code in your build folder.
./gradlew clean will purge that build folder.
./gradlew test will execute unit tests without building or running your code again.
Build process¶
Gradle launches as a new JVM process
It parses the gradle.properties file and configures Gradle accordingly
Next, it creates a Settings instance for the build
Then, it evaluates the settings.gradle file against the Settings object
It creates a hierarchy of Projects, based on the configured Settings object
Finally, it executes each build.gradle file against its project
In case of a multi-project build, we'd probably have multiple different build.gradle files, one for each project. The build.gradle file is executed against a Project instance, with one Project instance created per subproject.

Groovy DSL¶
Every Gradle build is made up of one or more projects. What a project represents depends on what it is that you are doing with Gradle. For example, a project might represent a library JAR or a web application.

Each project is made up of one or more tasks. A task represents some atomic piece of work which a build performs. This might be compiling some classes, creating a JAR, generating Javadoc, or publishing some archives to a repository.

Tasks¶
Tasks are snippets that we can run directly from the command line in our project directory via ./gradlew [TASK_NAME]

------------------------

Deatiled : https://touchlab.co/gradle-cheat-sheet/



Gradle.build file : https://github.com/jitpack/gradle-simple
