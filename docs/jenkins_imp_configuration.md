# imp points :

* Most Jenkins configuration changes can be made through the Jenkins user interface or through the configuration as code plugin.

## Networking parameters
Jenkins networking configuration is generally controlled by command line arguments. The networking configuration arguments are:

Table 1. Jenkins Networking Command Line Parameters

| Command Line Parameter | Description |
|------------------------| ------------- |
| `--httpPort=$HTTP_PORT` | Runs Jenkins listener on port $HTTP_PORT using standard http protocol. The default is port 8080. To disable (because you’re using https), use port -1. This option does not impact the root URL being generated within Jenkins logic (UI, inbound agent files, etc.). It is defined by the Jenkins URL specified in the global configuration. |
| `--httpListenAddress=$HTTP_HOST` | Binds Jenkins to the IP address represented by $HTTP_HOST. The default is 0.0.0.0 — i.e. listening on all available interfaces. For example, to only listen for requests from localhost, you could use: --httpListenAddress=127.0.0.1 |
| `--httpsPort=$HTTPS_PORT` | Uses HTTPS protocol on port $HTTPS_PORT. This option does not impact the root URL being generated within Jenkins logic (UI, inbound agent files, etc.). It is defined by the Jenkins URL specified in the global configuration. |
| `--httpsListenAddress=$HTTPS_HOST` | Binds Jenkins to listen for HTTPS requests on the IP address represented by $HTTPS_HOST. |
| `--http2Port=$HTTP_PORT` | Uses HTTP/2 protocol on port $HTTP_PORT. This option does not impact the root URL being generated within Jenkins logic (UI, inbound agent files, etc.). It is defined by the Jenkins URL specified in the global configuration. |
| `--http2ListenAddress=$HTTPS_HOST` | Binds Jenkins to listen for HTTP/2 requests on the IP address represented by $HTTPS_HOST. |
| `--prefix=$PREFIX` | Runs Jenkins to include the $PREFIX at the end of the URL. For example, set --prefix=/jenkins to make Jenkins accessible at http://myServer:8080/jenkins |
| `--sessionTimeout=$TIMEOUT` | Sets the http session timeout value to $SESSION_TIMEOUT minutes. Default to what webapp specifies, and then to 60 minutes |

* Jenkins passes all command line parameters to the Winstone servlet container.

* refrence ://www.jenkins.io/doc/book/installing/initial-settings/
------------------------------------------------------------------------------------------
------------------------------------------------------------------------------------------


##Configuring HTTP :
HTTPS with an existing certificate :-
If you’re setting up Jenkins using the built-in Winstone server and want to use an existing certificate for HTTPS:
```
--httpPort=-1 \
--httpsPort=443 \
--httpsKeyStore=path/to/keystore \
--httpsKeyStorePassword=keystorePassword
```

refence :- https://www.jenkins.io/doc/book/installing/initial-settings/

------------------------------------------------------------------------------------------
------------------------------------------------------------------------------------------

## Credentials stored in Jenkins can be used:
	* anywhere applicable throughout Jenkins (i.e. global credentials),

	* by a specific Pipeline project/item (read more about this in the Handling credentials section of Using a Jenkinsfile),

	* by a specific Jenkins user (as is the case for Pipeline projects created in Blue Ocean).

## Jenkins can store the following types of credentials:

	* Secret text - a token such as an API token (e.g. a GitHub personal access token),

	* Username and password - which could be handled as separate components or as a colon separated string in the format username:password (read more about this in Handling credentials),

	* Secret file - which is essentially secret content in a file,

	* SSH Username with private key - an SSH public/private key pair,

	* Certificate - a PKCS#12 certificate file and optional password, or

	* Docker Host Certificate Authentication credentials

## Security :  
  * credentials configured in Jenkins are stored in an encrypted form on the controller Jenkins instance (encrypted by the Jenkins instance ID) and are only handled in Pipeline projects via their credential IDs.
  *  Matrix-based security).
  *  default Logged-in users can do anything setting or Anyone can do anything setting.

refrence : https://www.jenkins.io/doc/book/using/using-credentials/
-------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------------------------------



## Referencing another project by name 
+ In many places throughout Jenkins, you can refer to another project/job by name. For example, in a Pipeline Script, you might want to copy artifacts from another project:

	`copyArtifacts projectName: 'myproject'`


+ Differentiating between multiple projects with the same name
	If you’re using the Folders Plugin and you have multiple projects with the same name that are in different folders, you can differentiate between them using a path, similar to a Unix filesystem path. There are two types of paths: Absolute paths  , Relative paths


* if your Multibranch Pipeline has a branch with a slash in it (feature/myfeature), replace the slash with %2F:

	`mymultibranchproject/feature%2Fmyfeature`

refrence : https://www.jenkins.io/doc/book/using/referencing-another-project-by-name/
------------------------------------------------------------------------------------------------------------------------------------
------------------------------------------------------------------------------------------------------------------------------------



## Aborting a build 
When you click that [x] icon from the UI, the following things happen:
	1.Browser sends a request to the server.
	2.The server interrupts (via Thread.interrupt()) the thread (AKA executor thread) that is responsible for carrying out a build.
 
The server returns
	1.At this point, your browser is back, but the actual abort process happens from here asynchronously.
	2.The thread gets the interrupt signal. How quickly this happens depends on what the executor is doing at the time of the interruption. Specifically, an executor thread can only be interrupted in "interruption points" due to the Java design.
	3.Waiting for a completion of a child process (for example, maybe the build is running Ant) is an interruption point. That means if the executor was doing that, it gets interrupted instantaneously.
	4.Waiting for a computation on an agent is an interruption point.
	5.Waiting for file or network I/O is not an interruption point. This often causes the problem where a build appears to be un-abortable. For example, checking out a Subversion repository falls in this category.
	6.Normal computation is also not an interruption point.
	7.The executor performs a clean up operation. This depends on what it was doing by the time it noticed the interruption.
	8.If it was waiting for a completion of a child process, Jenkins will search for all the descendant processes and kill them all. On Unix, this is done through java.lang.UnixProcess.destroyProcess, which sends SIGTERM on Unix-based JDK implementations. On Windows, this is done through TerminateProcess API.
	9.If it was waiting for a completion of some computation in an agent, the thread that’s performing the remote computation is interrupted asynchronously. How quickly that threads gets interrupted depends on what that thread is doing. See above.
	10. Executor starts unwinding the stack, and eventually it finishes the unwinding. At this point, the build is marked as aborted and executor returns to the idle status.

* Pipeline jobs can be stopped by sending an HTTP POST request to URL endpoints of a build.
	** BUILD ID URL/stop - aborts a Pipeline.
	** BUILD ID URL/term - forcibly terminates a build (should only be used if stop does not work).
	** BUILD ID URL/kill - hard kill a pipeline. This is the most destructive way to stop a pipeline and should only be used as a last resort.

* If your build isn’t aborting Check the thread dump `http://yourserver/jenkins/threadDump` and look for the executor thread in question — they are named after the agent and executor number. That’ll normally tell you where the thread is, and often reveals why it’s not responding to an interruption.


refrence : https://www.jenkins.io/doc/book/using/aborting-a-build/
----------------------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------------------

## Tracking file usage across Jenkins jobs using fingerprints

Since recording fingerprints is a cheap operation, the simplest thing to do is just blindly record all fingerprints of the followings:
	1.jar files that your project produce
	2.jar files that your project rely on

The disk usage is affected more by the number of files fingerprinted, as opposed to the size of files or the number of builds they are used. So unless you have a plenty of disk space, you don’t want to fingerprint **/*.


* Configure a job to Record Fingerprint(s) of a file or set of files
Go to your project, click Configure in the left navigation bar, then scroll down to the Post-build Actions section of the job

Click on the button to add a Post-build action.

Select Record fingerprints of files to track usage.

The post-build action configuration fields provide you with a pattern option to match the files you want to fingerprint as well as a couple check-box selections to do your file fingerprinting.

Maven job type does this automatically for its dependencies and artifacts.

* How does it work?
The fingerprint of a file is simply an MD5 checksum. Jenkins maintains a database of MD5 checksums, and for each MD5 checksum, Jenkins records which builds of which projects used it. This database is updated every time a build runs and files are fingerprinted.

To avoid excessive disk usage, Jenkins does not store the actual file. Instead, it just stores MD5 checksums and their usages. These files can be seen in

`$JENKINS_HOME/fingerprints`

refrence : https://www.jenkins.io/doc/book/using/fingerprints/

-----------------------------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------------------------


## Remote Access API 
Jenkins provides machine-consumable remote access API to its functionalities. Currently it comes in three flavors:

1.XML
2.JSON with JSONP support
3.Python

Remote access API is offered in a REST-like style. That is, there is no single entry point for all features, and instead they are available under the `".../api/"` URL where "..." portion is the data that it acts on.

For example, if your Jenkins installation sits at https://ci.jenkins.io, visiting `https://ci.jenkins.io/api/` will show just the top-level API features available – primarily a listing of the configured jobs for this Jenkins instance.
Or if you want to access information about a particular build, e.g.` https://ci.jenkins.io/job/Websites/job/jenkins.io/job/master/lastSuccessfulBuild/` , then go to `https://ci.jenkins.io/job/Websites/job/jenkins.io/job/master/lastSuccessfulBuild/api/` and you’ll see the list of functionalities for that build.

*What can you do with it?
	*Remote API can be used to do things like these:

	*retrieve information from Jenkins for programmatic consumption.

	*trigger a new build

	*create/copy jobs

	*Submitting jobs
	*Jobs without parameters

	*You merely need to perform an HTTP POST on `JENKINS_URL/job/JOBNAME/build`.

	*This also works for Multibranch Pipelines and Organization Folders. It would trigger a scan.

##Jobs with parameters

+ Simple example - sending "String Parameters":

```
curl JENKINS_URL/job/JOB_NAME/buildWithParameters \
  --user USER:TOKEN \
  --data id=123 --data verbosity=high
```
+ Another example - sending a "File Parameter":
```
curl JENKINS_URL/job/JOB_NAME/buildWithParameters \
  --user USER:PASSWORD \
  --form FILE_LOCATION_AS_SET_IN_JENKINS=@PATH_TO_FILE
```
The symbol '@' is important in this example. Also, the path to the file is absolute path. In order to make this command work, you need to configure your Jenkins job to take a file parameter and match the File location field in the Jenkins job configuration with the key in the --form option.


##Detecting Jenkins version
To check the version of Jenkins, load the top page or any `.../api/*` page and check for the X-Jenkins response header. This contains the version number of Jenkins, like "1.404" This is also a good way to check if an URL is a Jenkins URL.

refrence : https://www.jenkins.io/doc/book/using/remote-access-api/
----------------------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------------------

Executor Starvation 
If you see a little black clock icon in the build queue as shown below, it is an indication that your job is sitting in the queue unnecessarily.

Reasons :- 
Agents are offline: 
Waiting for an available executor on an agent: 

https://www.jenkins.io/doc/book/using/executor-starvation/
-------------------------------------------------------------------------------------------------------------------


Using Jenkins agents 

The agents require a Java installation and a network connection to the Jenkins controller.
Environment
To run this guide you will need a machine with:

Java installation

Jenkins installation

Docker installation

SSH key pair

https://www.jenkins.io/doc/book/using/using-agents/
---------------------------------------------------------------------------------------------------------------------

Install the Performance plugin
To integrate JMeter with Jenkins, we will use the Performance plugin.

Apache JMeter
Apache JMeter may be used to test the performance of static sites, dynamic sites, and complete web applications. It can also be used to simulate a heavy load on a server, group of servers, network, or object, allowing for strength testing or overall performance analyzation under different load types.
From the Console Output view, you can access the Performance Report, and can view the JMeter report data.


https://www.jenkins.io/doc/book/using/using-jmeter-with-jenkins/
---------------------------------------------------------------------------------------------------------------------------

jenkins doesnot support cloudnative however we can configure few parts using 3rd party :
https://www.jenkins.io/doc/book/using/pluggable-storage/

---------------------------------------------------------------------------------------------------------------------


Why Pipeline?
Jenkins is, fundamentally, an automation engine which supports a number of automation patterns. Pipeline adds a powerful set of automation tools onto Jenkins, supporting use cases that span from simple continuous integration to comprehensive CD pipelines. By modeling a series of related tasks, users can take advantage of the many features of Pipeline:

Code: Pipelines are implemented in code and typically checked into source control, giving teams the ability to edit, review, and iterate upon their delivery pipeline.

Durable: Pipelines can survive both planned and unplanned restarts of the Jenkins controller.

Pausable: Pipelines can optionally stop and wait for human input or approval before continuing the Pipeline run.

Versatile: Pipelines support complex real-world CD requirements, including the ability to fork/join, loop, and perform work in parallel.

Extensible: The Pipeline plugin supports custom extensions to its DSL [1] and multiple options for integration with other plugins.

Pipeline
A Pipeline is a user-defined model of a CD pipeline. A Pipeline’s code defines your entire build process, which typically includes stages for building an application, testing it and then delivering it.

Also, a pipeline block is a key part of Declarative Pipeline syntax.

Node
A node is a machine which is part of the Jenkins environment and is capable of executing a Pipeline.

Also, a node block is a key part of Scripted Pipeline syntax.

Stage
A stage block defines a conceptually distinct subset of tasks performed through the entire Pipeline (e.g. "Build", "Test" and "Deploy" stages), which is used by many plugins to visualize or present Jenkins Pipeline status/progress. [6]

Step
A single task. Fundamentally, a step tells Jenkins what to do at a particular point in time (or "step" in the process). For example, to execute the shell command make use the sh step: sh 'make'. When a plugin extends the Pipeline DSL, [1] that typically means the plugin has implemented a new step.

Pipeline example - Declarative
'''
pipeline {
    agent any
    options {
        skipStagesAfterUnstable()
    }
    stages {
        stage('Build') {
            steps {
                sh 'make'
            }
        }
        stage('Test'){
            steps {
                sh 'make check'
                junit 'reports/**/*.xml'
            }
        }
        stage('Deploy') {
            steps {
                sh 'make publish'
            }
        }
    }
}
'''

A Pipeline can be created in one of the following ways:

Through Blue Ocean - after setting up a Pipeline project in Blue Ocean, the Blue Ocean UI helps you write your Pipeline’s Jenkinsfile and commit it to source control.

Through the classic UI - you can enter a basic Pipeline directly in Jenkins through the classic UI.

In SCM - you can write a Jenkinsfile manually, which you can commit to your project’s source control repository. [3]

Through Blue Ocean
If you are new to Jenkins Pipeline, the Blue Ocean UI helps you set up your Pipeline project, and automatically creates and writes your Pipeline (i.e. the Jenkinsfile) for you through the graphical Pipeline editor.

As part of setting up your Pipeline project in Blue Ocean, Jenkins configures a secure and appropriately authenticated connection to your project’s source control repository. Therefore, any changes you make to the Jenkinsfile via Blue Ocean’s Pipeline editor are automatically saved and committed to source control.

The Pipeline syntax snippet generator assists users as they define Pipeline steps with their arguments. It is the preferred tool for Jenkins Pipeline creation, as it provides online help for the Pipeline steps available in your Jenkins controller. It uses the plugins installed on your Jenkins controller to generate the Pipeline syntax. Refer to the Pipeline steps reference page for information on all available Pipeline steps.

The built-in documentation can be found globally at ${YOUR_JENKINS_URL}/pipeline-syntax. The same documentation is also linked as Pipeline Syntax in the side-bar for any configured Pipeline project.

The variables provided by default in Pipeline are:

env
Exposes environment variables, for example: env.PATH or env.BUILD_ID. Consult the built-in global variable reference at ${YOUR_JENKINS_URL}/pipeline-syntax/globals#env for a complete, and up to date, list of environment variables available in Pipeline.

params
Exposes all parameters defined for the Pipeline as a read-only Map, for example: params.MY_PARAM_NAME.

currentBuild
May be used to discover information about the currently executing Pipeline, with properties such as currentBuild.result, currentBuild.displayName, etc. Consult the built-in global variable reference at ${YOUR_JENKINS_URL}/pipeline-syntax/globals for a complete, and up to date, list of properties available on currentBuild.

Declarative Directive Generator
While the Snippet Generator helps with generating steps for a Scripted Pipeline or for the steps block in a stage in a Declarative Pipeline, it does not cover the sections and directives used to define a Declarative Pipeline. The "Declarative Directive Generator" utility helps with that. Similar to the Snippet Generator, the Directive Generator allows you to choose a Declarative directive, configure it in a form, and generate the configuration for that directive, which you can then use in your Declarative Pipeline.

To generate a Declarative directive using the Declarative Directive Generator:

Navigate to the Pipeline Syntax link (referenced above) from a configured Pipeline, and then click on the Declarative Directive Generator link in the sidepanel, or go directly to ${YOUR_JENKINS_URL}/directive-generator.

Select the desired directive in the dropdown menu

Use the dynamically populated area below the dropdown to configure the selected directive.

Click Generate Directive to create the directive’s configuration to copy into your Pipeline.

The Directive Generator can generate configuration for nested directives, such as conditions inside a when directive, but it cannot generate Pipeline steps. For the contents of directives which contain steps, such as steps inside a stage or conditions like always or failure inside post, the Directive Generator adds a placeholder comment instead. You will still need to add steps to your Pipeline by hand.

Creating a Jenkinsfile, which is checked into source control [1], provides a number of immediate benefits:

Code review/iteration on the Pipeline

Audit trail for the Pipeline

Single source of truth [2] for the Pipeline, which can be viewed and edited by multiple members of the project.

The Declarative Pipeline example above contains the minimum necessary structure to implement a continuous delivery pipeline. The agent directive, which is required, instructs Jenkins to allocate an executor and workspace for the Pipeline. Without an agent directive, not only is the Declarative Pipeline not valid, it would not be capable of doing any work! By default the agent directive ensures that the source repository is checked out and made available for steps in the subsequent stages.

The stages directive, and steps directives are also required for a valid Declarative Pipeline as they instruct Jenkins what to execute and in which stage it should be executed.

Build
For many projects the beginning of "work" in the Pipeline would be the "build" stage. Typically this stage of the Pipeline will be where source code is assembled, compiled, or packaged. The Jenkinsfile is not a replacement for an existing build tool such as GNU/Make, Maven, Gradle, etc, but rather can be viewed as a glue layer to bind the multiple phases of a project’s development lifecycle (build, test, deploy, etc) together.

Jenkins has a number of plugins for invoking practically any build tool in general use, but this example will simply invoke make from a shell step (sh). The sh step assumes the system is Unix/Linux-based, for Windows-based systems the bat could be used instead.

Archiving artifacts is not a substitute for using external artifact repositories such as Artifactory or Nexus and should be considered only for basic reporting and file archival.



Test
Running automated tests is a crucial component of any successful continuous delivery process. As such, Jenkins has a number of test recording, reporting, and visualization facilities provided by a number of plugins. At a fundamental level, when there are test failures, it is useful to have Jenkins record the failures for reporting and visualization in the web UI. The example below uses the junit step, provided by the JUnit plugin.

In the example below, if tests fail, the Pipeline is marked "unstable", as denoted by a yellow ball in the web UI. Based on the recorded test reports, Jenkins can also provide historical trend analysis and visualization.

'''
pipeline {
    agent any

    stages {
        stage('Test') {
            steps {
                /* `make check` returns non-zero on test failures,
                * using `true` to allow the Pipeline to continue nonetheless
                */
                sh 'make check || true'
                junit '**/target/*.xml'
            }
        }
    }
}
'''


Using environment variables
Jenkins Pipeline exposes environment variables via the global variable env, which is available from anywhere within a Jenkinsfile. The full list of environment variables accessible from within Jenkins Pipeline is documented at ${YOUR_JENKINS_URL}/pipeline-syntax/globals#env and includes:

BUILD_ID
The current build ID, identical to BUILD_NUMBER for builds created in Jenkins versions 1.597+

BUILD_NUMBER
The current build number, such as "153"

BUILD_TAG
String of jenkins-${JOB_NAME}-${BUILD_NUMBER}. Convenient to put into a resource file, a jar file, etc for easier identification

BUILD_URL
The URL where the results of this build can be found (for example http://buildserver/jenkins/job/MyJobName/17/ )

EXECUTOR_NUMBER
The unique number that identifies the current executor (among executors of the same machine) performing this build. This is the number you see in the "build executor status", except that the number starts from 0, not 1

JAVA_HOME
If your job is configured to use a specific JDK, this variable is set to the JAVA_HOME of the specified JDK. When this variable is set, PATH is also updated to include the bin subdirectory of JAVA_HOME

JENKINS_URL
Full URL of Jenkins, such as https://example.com:port/jenkins/ (NOTE: only available if Jenkins URL set in "System Configuration")

JOB_NAME
Name of the project of this build, such as "foo" or "foo/bar".

NODE_NAME
The name of the node the current build is running on. Set to 'master' for the Jenkins controller.

WORKSPACE
The absolute path of the workspace

Referencing or using these environment variables can be accomplished like accessing any key in a Groovy Map, for example:

'''
pipeline {
    agent any
    stages {
        stage('Example') {
            steps {
                echo "Running ${env.BUILD_ID} on ${env.JENKINS_URL}"
            }
        }
    }
}
'''

Setting environment variables
Setting an environment variable within a Jenkins Pipeline is accomplished differently depending on whether Declarative or Scripted Pipeline is used.

Declarative Pipeline supports an environment directive, whereas users of Scripted Pipeline must use the withEnv step.
'''
pipeline {
    agent any
    environment {
        CC = 'clang'
    }
    stages {
        stage('Example') {
            environment {
                DEBUG_FLAGS = '-g'
            }
            steps {
                sh 'printenv'
            }
        }
    }
}
'''

	An environment directive used in the top-level pipeline block will apply to all steps within the Pipeline.
An environment directive defined within a stage will only apply the given environment variables to steps within the stage.


Secret files
A secret file is a credential which is stored in a file and uploaded to Jenkins. Secret files are used for credentials that are:

too unwieldy to enter directly into Jenkins, and/or

in binary format, such as a GPG file.

String interpolation
Jenkins Pipeline uses rules identical to Groovy for string interpolation. Groovy’s String interpolation support can be confusing to many newcomers to the language. While Groovy supports declaring a string with either single quotes, or double quotes, for example:

def singlyQuoted = 'Hello'
def doublyQuoted = "World"
Only the latter string will support the dollar-sign ($) based string interpolation, for example:

def username = 'Jenkins'
echo 'Hello Mr. ${username}'
echo "I said, Hello Mr. ${username}"
Would result in:

Hello Mr. ${username}
I said, Hello Mr. Jenkins


-Interpolation of sensitive environment variables : https://www.jenkins.io/doc/book/pipeline/jenkinsfile/#interpolation-of-sensitive-environment-variables

Handling parameters
Declarative Pipeline supports parameters out-of-the-box, allowing the Pipeline to accept user-specified parameters at runtime via the parameters directive. Configuring parameters with Scripted Pipeline is done with the properties step, which can be found in the Snippet Generator.

If you configured your pipeline to accept parameters using the Build with Parameters option, those parameters are accessible as members of the params variable.

Assuming that a String parameter named "Greeting" has been configuring in the Jenkinsfile, it can access that parameter via ${params.Greeting}:
'''
pipeline {
    agent any
    parameters {
        string(name: 'Greeting', defaultValue: 'Hello', description: 'How should I greet the world?')
    }
    stages {
        stage('Example') {
            steps {
                echo "${params.Greeting} World!"
            }
        }
    }
}
'''

Using multiple agents
In all the previous examples, only a single agent has been used. This means Jenkins will allocate an executor wherever one is available, regardless of how it is labeled or configured. Not only can this behavior be overridden, but Pipeline allows utilizing multiple agents in the Jenkins environment from within the same Jenkinsfile, which can helpful for more advanced use-cases such as executing builds/tests across multiple platforms.

In the example below, the "Build" stage will be performed on one agent and the built results will be reused on two subsequent agents, labelled "linux" and "windows" respectively, during the "Test" stage.

'''
pipeline {
    agent none
    stages {
        stage('Build') {
            agent any
            steps {
                checkout scm
                sh 'make'
                stash includes: '**/target/*.jar', name: 'app'
            }
        }
        stage('Test on Linux') {
            agent {
                label 'linux'
            }
            steps {
                unstash 'app'
                sh 'make check'
            }
            post {
                always {
                    junit '**/target/*.xml'
                }
            }
        }
        stage('Test on Windows') {
            agent {
                label 'windows'
            }
            steps {
                unstash 'app'
                bat 'make check'
            }
            post {
                always {
                    junit '**/target/*.xml'
                }
            }
        }
    }
}
'''

Parallel stages with Declarative Pipeline : https://www.jenkins.io/blog/2017/09/25/declarative-1/
You can now specify either steps or parallel for a stage, and within parallel, you can specify a list of stage directives to run in parallel, with all the configuration you’re used to for a stage in Declarative Pipeline. We think this will be really useful for cross-platform builds and testing, as an example. Support for parallel stages will be in the soon-to-be-released Blue Ocean Pipeline Editor 1.3 as well.


'''
pipeline {
    agent none
    stages {
        stage('Run Tests') {
            parallel {
                stage('Test On Windows') {
                    agent {
                        label "windows"
                    }
                    steps {
                        bat "run-tests.bat"
                    }
                    post {
                        always {
                            junit "**/TEST-*.xml"
                        }
                    }
                }
                stage('Test On Linux') {
                    agent {
                        label "linux"
                    }
                    steps {
                        sh "run-tests.sh"
                    }
                    post {
                        always {
                            junit "**/TEST-*.xml"
                        }
                    }
                }
            }
        }
    }
}
'''



https://www.jenkins.io/doc/book/pipeline/jenkinsfile/
---------------------------------------------------------------------------------------
Preserving `stash`es for Use with Restarted Stages
Normally, when you run the stash step in your Pipeline, the resulting stash of artifacts is cleared when the Pipeline completes, regardless of the result of the Pipeline. Since stash artifacts aren’t accessible outside of the Pipeline run that created them, this has not created any limitations on usage. But with Declarative stage restarting, you may want to be able to unstash artifacts from a stage which ran before the stage you’re restarting from.

To enable this, there is a job property that allows you to configure a maximum number of completed runs whose stash artifacts should be preserved for reuse in a restarted run. You can specify anywhere from 1 to 50 as the number of runs to preserve.

This job property can be configured in your Declarative Pipeline’s options section, as below

'''
options {
    preserveStashes()
    // or
    preserveStashes(buildCount: 5)
}
'''

Scheduling jobs in Jenkins
The scheduling function lets you schedule jobs to run automatically during off-hours or down times. Scheduling jobs can help you to scale your environment as Jenkins usage increases.

Creating a Multibranch Pipeline
The Multibranch Pipeline project type enables you to implement different Jenkinsfiles for different branches of the same project. In a Multibranch Pipeline project, Jenkins automatically discovers, manages and executes Pipelines for branches which contain a Jenkinsfile in source control.

This eliminates the need for manual Pipeline creation and management.

Supporting Pull Requests
Multibranch Pipelines can be used for validating pull/change requests with the appropriate plugin. This functionality is provided by the following plugins:


Using Organization Folders
Organization Folders enable Jenkins to monitor an entire GitHub Organization, Bitbucket Team/Project, GitLab organization, or Gitea organization and automatically create new Multibranch Pipelines for repositories which contain branches and pull requests containing a Jenkinsfile.

https://www.jenkins.io/doc/book/pipeline/multibranch/
---------------------------------------------------------------------------------------------------------------

Using Docker with Pipeline 
Many organizations use Docker to unify their build and test environments across machines, and to provide an efficient mechanism for deploying applications. Starting with Pipeline versions 2.5 and higher, Pipeline has built-in support for interacting with Docker from within a Jenkinsfile.

Customizing the execution environment
Pipeline is designed to easily use Docker images as the execution environment for a single Stage or the entire Pipeline. Meaning that a user can define the tools required for their Pipeline, without having to manually configure agents. Practically any tool which can be packaged in a Docker container. can be used with ease by making only minor edits to a Jenkinsfile.
'''
pipeline {
    agent {
        docker { image 'node:18.16.0-alpine' }
    }
    stages {
        stage('Test') {
            steps {
                sh 'node --version'
            }
        }
    }
}
'''
When the Pipeline executes, Jenkins will automatically start the specified container and execute the defined steps within it:

Workspace synchronization
Short: if it is important to keep workspace synchronized with other stages, use reuseNode true. Otherwise, dockerized stage can be run on any other agent or on the same agent, but in temporary workspace.

By default, for containerized stage, Jenkins does:

pick any agent,

create new empty workspace,

clone pipeline code into it,

mount this new workspace into container.

If you have multiple Jenkins agents, your containerized stage can be started on any of them.

When reuseNode set to true: no new workspace will be created, and current workspace from current agent will be mounted into container, and container will be started at the same node, so whole data will be synchronized.

'''
pipeline {
    agent any
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'gradle:6.7-jdk11'
                    // Run the container on the node specified at the
                    // top-level of the Pipeline, in the same workspace,
                    // rather than on a new node entirely:
                    reuseNode true
                }
            }
            steps {
                sh 'gradle --version'
            }
        }
    }
}
'''

Caching data for containers
Many build tools will download external dependencies and cache them locally for future re-use. Since containers are initially created with "clean" file systems, this can result in slower Pipelines, as they may not take advantage of on-disk caches between subsequent Pipeline runs.

Pipeline supports adding custom arguments which are passed to Docker, allowing users to specify custom Docker Volumes to mount, which can be used for caching data on the agent between Pipeline runs. The following example will cache ~/.m2 between Pipeline runs utilizing the maven container, thereby avoiding the need to re-download dependencies for subsequent runs of the Pipeline.

'''
pipeline {
    agent {
        docker {
            image 'maven:3.9.0-eclipse-temurin-11'
            args '-v $HOME/.m2:/root/.m2'
        }
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn -B'
            }
        }
    }
}
'''

Using a Dockerfile
For projects which require a more customized execution environment, Pipeline also supports building and running a container from a Dockerfile in the source repository. In contrast to the previous approach of using an "off-the-shelf" container, using the agent { dockerfile true } syntax will build a new image from a Dockerfile rather than pulling one from Docker Hub.

Reusing an example from above, with a more custom Dockerfile:

'''
FROM node:18.16.0-alpine

RUN apk add -U subversion
'''

Building containers
In order to create a Docker image, the Docker Pipeline plugin also provides a build() method for creating a new image, from a Dockerfile in the repository, during a Pipeline run.

ref : https://stackoverflow.com/questions/50042753/how-to-build-docker-images-using-a-declarative-jenkinsfile


Using a remote Docker server
By default, the Docker Pipeline plugin will communicate with a local Docker daemon, typically accessed through /var/run/docker.sock.

To select a non-default Docker server, such as with Docker Swarm, the withServer() method should be used.

Using a custom registry
By default the Docker Pipeline integrates assumes the default Docker Registry of Docker Hub.


https://www.jenkins.io/doc/book/pipeline/docker/
------------------------------------------------------------------------------------------------------

Extending with Shared Libraries 

As Pipeline is adopted for more and more projects in an organization, common patterns are likely to emerge. Oftentimes it is useful to share parts of Pipelines between various projects to reduce redundancies and keep code "DRY" [1].

Pipeline has support for creating "Shared Libraries" which can be defined in external source control repositories and loaded into existing Pipelines.

https://www.jenkins.io/doc/book/pipeline/shared-libraries/
-----------------------------------------------------------------------------------------------------------------------------------

## Command-line Pipeline Linter
Jenkins can validate, or "lint", a Declarative Pipeline from the command line before actually running it. This can be done using a Jenkins CLI command or by making an HTTP POST request with appropriate parameters. We recommended using the SSH interface to run the linter. See the Jenkins CLI documentation for details on how to properly configure Jenkins for secure command-line access.

* Linting via the CLI with SSH

```
# ssh (Jenkins CLI)
# JENKINS_PORT=[sshd port on controller]
# JENKINS_HOST=[Jenkins controller hostname]
ssh -p $JENKINS_PORT $JENKINS_HOST declarative-linter < Jenkinsfile
```

or

* Linting via HTTP POST using curl

```
# curl (REST API)
# Assuming "anonymous read access" has been enabled on your Jenkins instance.
# JENKINS_URL=[root URL of Jenkins controller]
# JENKINS_CRUMB is needed if your Jenkins controller has CRSF protection enabled as it should
JENKINS_CRUMB=`curl "$JENKINS_URL/crumbIssuer/api/xml?xpath=concat(//crumbRequestField,\":\",//crumb)"`
curl -X POST -H $JENKINS_CRUMB -F "jenkinsfile=<Jenkinsfile" $JENKINS_URL/pipeline-model-converter/validate
```
** Blue Ocean Editor
The Blue Ocean Pipeline Editor provides a WYSIWYG way to create Declarative Pipelines. The editor offers a structural view of all the stages, parallel branches, and steps in a Pipeline. The editor validates Pipeline changes as they are made, eliminating many errors before they are even committed. Behind the scenes it still generates Declarative Pipeline code.

* IDE Integrations
Eclipse / atom /sublime /vscode etc 

refrence : https://www.jenkins.io/doc/book/pipeline/development/

-----------------------------------------------------------------------------
------------------------------------------------------------------------------------------------------

## Differences between top level agents and stage level agents
	* read articke for more details, included top level and stage level agent timeout

## Parameters
In order to support the wide variety of use-cases Pipeline authors may have, the agent section supports a few different types of parameters. These parameters can be applied at the top-level of the pipeline block, or within each stage directive.

### any
Execute the Pipeline, or stage, on any available agent. For example: agent any

### none
When applied at the top-level of the pipeline block no global agent will be allocated for the entire Pipeline run and each stage section will need to contain its own agent section. For example: agent none

### label
Execute the Pipeline, or stage, on an agent available in the Jenkins environment with the provided label. For example: agent { label 'my-defined-label' }

Label conditions can also be used. For example: agent { label 'my-label1 && my-label2' } or agent { label 'my-label1 || my-label2' }

### node
agent { node { label 'labelName' } } behaves the same as agent { label 'labelName' }, but node allows for additional options (such as customWorkspace).

### docker
Execute the Pipeline, or stage, with the given container which will be dynamically provisioned on a node pre-configured to accept Docker-based Pipelines, or on a node matching the optionally defined label parameter. docker also optionally accepts an args parameter which may contain arguments to pass directly to a docker run invocation, and an alwaysPull option, which will force a docker pull even if the image name is already present. For example: agent { docker 'maven:3.9.0-eclipse-temurin-11' } or

```
agent {
    docker {
        image 'maven:3.9.0-eclipse-temurin-11'
        label 'my-defined-label'
        args  '-v /tmp:/tmp'
    }
}
```

docker also optionally accepts a registryUrl and registryCredentialsId parameters which will help to specify the Docker Registry to use and its credentials. The parameter registryCredentialsId could be used alone for private repositories within the docker hub. For example:


```
agent {
    docker {
        image 'myregistry.com/node'
        label 'my-defined-label'
        registryUrl 'https://myregistry.com/'
        registryCredentialsId 'myPredefinedCredentialsInJenkins'
    }
}
```

Execute the Pipeline, or stage, with a container built from a Dockerfile contained in the source repository. In order to use this option, the Jenkinsfile must be loaded from either a Multibranch Pipeline or a Pipeline from SCM. Conventionally this is the Dockerfile in the root of the source repository: agent { dockerfile true }. If building a Dockerfile in another directory, use the dir option: agent { dockerfile { dir 'someSubDir' } }. If your Dockerfile has another name, you can specify the file name with the filename option. You can pass additional arguments to the docker build …​command with the additionalBuildArgs option, like agent { dockerfile { additionalBuildArgs '--build-arg foo=bar' } }. For example, a repository with the file build/Dockerfile.build, expecting a build argument version:

```
agent {
    // Equivalent to "docker build -f Dockerfile.build --build-arg version=1.0.2 ./build/
    dockerfile {
        filename 'Dockerfile.build'
        dir 'build'
        label 'my-defined-label'
        additionalBuildArgs  '--build-arg version=1.0.2'
        args '-v /tmp:/tmp'
    }
}
```
dockerfile also optionally accepts a registryUrl and registryCredentialsId parameters which will help to specify the Docker Registry to use and its credentials. For example:
```
agent {
    dockerfile {
        filename 'Dockerfile.build'
        dir 'build'
        label 'my-defined-label'
        registryUrl 'https://myregistry.com/'
        registryCredentialsId 'myPredefinedCredentialsInJenkins'
    }
}
```

### kubernetes
Execute the Pipeline, or stage, inside a pod deployed on a Kubernetes cluster. In order to use this option, the Jenkinsfile must be loaded from either a Multibranch Pipeline or a Pipeline from SCM. The Pod template is defined inside the kubernetes { } block. For example, if you want a pod with a Kaniko container inside it, you would define it as follows:
```
agent {
    kubernetes {
        defaultContainer 'kaniko'
        yaml '''
kind: Pod
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    imagePullPolicy: Always
    command:
    - sleep
    args:
    - 99d
    volumeMounts:
      - name: aws-secret
        mountPath: /root/.aws/
      - name: docker-registry-config
        mountPath: /kaniko/.docker
  volumes:
    - name: aws-secret
      secret:
        secretName: aws-secret
    - name: docker-registry-config
      configMap:
        name: docker-registry-config
'''
   }
```
You will need to create a secret aws-secret for Kaniko to be able to authenticate with ECR. This secret should contain the contents of ~/.aws/credentials. The other volume is a ConfigMap which should contain the endpoint of your ECR registry. For example:
```
{
      "credHelpers": {
        "<your-aws-account-id>.dkr.ecr.eu-central-1.amazonaws.com": "ecr-login"
      }
}
```


### customWorkspace
A string. Run the Pipeline or individual stage this agent is applied to within this custom workspace, rather than the default. It can be either a relative path, in which case the custom workspace will be under the workspace root on the node, or an absolute path. For example:
```
agent {
    node {
        label 'my-defined-label'
        customWorkspace '/some/other/path'
    }
}
```
This option is valid for node, docker, and dockerfile.

### reuseNode
A boolean, false by default. If true, run the container on the node specified at the top-level of the Pipeline, in the same workspace, rather than on a new node entirely.

This option is valid for docker and dockerfile, and only has an effect when used on an agent for an individual stage.

## post
The post section defines one or more additional steps that are run upon the completion of a Pipeline’s or stage’s run (depending on the location of the post section within the Pipeline). post can support any of the following post-condition blocks: always, changed, fixed, regression, aborted, failure, success, unstable, unsuccessful, and cleanup. These condition blocks allow the execution of steps inside each condition depending on the completion status of the Pipeline or stage. 

### Conditions
* always
Run the steps in the post section regardless of the completion status of the Pipeline’s or stage’s run.

* changed
Only run the steps in post if the current Pipeline’s run has a different completion status from its previous run.

* fixed
Only run the steps in post if the current Pipeline’s run is successful and the previous run failed or was unstable.

* regression
Only run the steps in post if the current Pipeline’s or status is failure, unstable, or aborted and the previous run was successful.

* aborted
Only run the steps in post if the current Pipeline’s run has an "aborted" status, usually due to the Pipeline being manually aborted. This is typically denoted by gray in the web UI.

* failure
Only run the steps in post if the current Pipeline’s or stage’s run has a "failed" status, typically denoted by red in the web UI.

* success
Only run the steps in post if the current Pipeline’s or stage’s run has a "success" status, typically denoted by blue or green in the web UI.

* unstable
Only run the steps in post if the current Pipeline’s run has an "unstable" status, usually caused by test failures, code violations, etc. This is typically denoted by yellow in the web UI.

* unsuccessful
Only run the steps in post if the current Pipeline’s or stage’s run has not a "success" status. This is typically denoted in the web UI depending on the status previously mentioned (for stages this may fire if the build itself is unstable).

* cleanup
Run the steps in this post condition after every other post condition has been evaluated, regardless of the Pipeline or stage’s status.

```
pipeline {
    agent any
    stages {
        stage('Example') {
            steps {
                echo 'Hello World'
            }
        }
    }
    post {
        always {
            echo 'I will always say Hello again!'
        }
    }
}
```

## Directives :

### environment
The environment directive specifies a sequence of key-value pairs which will be defined as environment variables for all steps, or stage-specific steps, depending on where the environment directive is located within the Pipeline.

This directive supports a special helper method credentials() which can be used to access pre-defined Credentials by their identifier in the Jenkins environment

### options
The options directive allows configuring Pipeline-specific options from within the Pipeline itself. Pipeline provides a number of these options, such as buildDiscarder, but they may also be provided by plugins, such as timestamps.

## Available Options
* buildDiscarder
Persist artifacts and console output for the specific number of recent Pipeline runs. For example: options { buildDiscarder(logRotator(numToKeepStr: '1')) }

* checkoutToSubdirectory
Perform the automatic source control checkout in a subdirectory of the workspace. For example: options { checkoutToSubdirectory('foo') }

* disableConcurrentBuilds
Disallow concurrent executions of the Pipeline. Can be useful for preventing simultaneous accesses to shared resources, etc. For example: options { disableConcurrentBuilds() } to queue a build when there’s already an executing build of the Pipeline, or options { disableConcurrentBuilds(abortPrevious: true) } to abort the running one and start the new build.

* disableResume
Do not allow the pipeline to resume if the controller restarts. For example: options { disableResume() }

* newContainerPerStage
Used with docker or dockerfile top-level agent. When specified, each stage will run in a new container instance on the same node, rather than all stages running in the same container instance.

* overrideIndexTriggers
Allows overriding default treatment of branch indexing triggers. If branch indexing triggers are disabled at the multibranch or organization label, options { overrideIndexTriggers(true) } will enable them for this job only. Otherwise, options { overrideIndexTriggers(false) } will disable branch indexing triggers for this job only.

* preserveStashes
Preserve stashes from completed builds, for use with stage restarting. For example: options { preserveStashes() } to preserve the stashes from the most recent completed build, or options { preserveStashes(buildCount: 5) } to preserve the stashes from the five most recent completed builds.

* quietPeriod
Set the quiet period, in seconds, for the Pipeline, overriding the global default. For example: options { quietPeriod(30) }

* retry
On failure, retry the entire Pipeline the specified number of times. For example: options { retry(3) }

* skipDefaultCheckout
Skip checking out code from source control by default in the agent directive. For example: options { skipDefaultCheckout() }

* skipStagesAfterUnstable
Skip stages once the build status has gone to UNSTABLE. For example: options { skipStagesAfterUnstable() }

* timeout
Set a timeout period for the Pipeline run, after which Jenkins should abort the Pipeline. For example: options { timeout(time: 1, unit: 'HOURS') }

* timestamps
Prepend all console output generated by the Pipeline run with the time at which the line was emitted. For example: options { timestamps() }

* parallelsAlwaysFailFast
Set failfast true for all subsequent parallel stages in the pipeline. For example: options { parallelsAlwaysFailFast() }

* disableRestartFromStage
Completely disable option "Restart From Stage" visible in classic Jenkins UI and Blue Ocean as well. For example: options { disableRestartFromStage() }. This option can not be used inside of the stage.


## parameters
The parameters directive provides a list of parameters that a user should provide when triggering the Pipeline. The values for these user-specified parameters are made available to Pipeline steps via the params object, see the Parameters, Declarative Pipeline for its specific usage.

Each parameter has a Name and Value, depending on the parameter type. This information is exported as environment variables when the build starts, allowing subsequent parts of the build configuration to access those values. For example, use the ${PARAMETER_NAME} syntax with POSIX shells like bash and ksh, the ${Env:PARAMETER_NAME} syntax with PowerShell, or the %PARAMETER_NAME% syntax with Windows cmd.exe.

### Available Parameters
* string
A parameter of a string type, for example: parameters { string(name: 'DEPLOY_ENV', defaultValue: 'staging', description: '') }

* text
A text parameter, which can contain multiple lines, for example: parameters { text(name: 'DEPLOY_TEXT', defaultValue: 'One\nTwo\nThree\n', description: '') }

* booleanParam
A boolean parameter, for example: parameters { booleanParam(name: 'DEBUG_BUILD', defaultValue: true, description: '') }

* choice
A choice parameter, for example: parameters { choice(name: 'CHOICES', choices: ['one', 'two', 'three'], description: '') }. The first value is the default.

* password
A password parameter, for example: parameters { password(name: 'PASSWORD', defaultValue: 'SECRET', description: 'A secret password') }

```
pipeline {
    agent any
    parameters {
        string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')

        text(name: 'BIOGRAPHY', defaultValue: '', description: 'Enter some information about the person')

        booleanParam(name: 'TOGGLE', defaultValue: true, description: 'Toggle this value')

        choice(name: 'CHOICE', choices: ['One', 'Two', 'Three'], description: 'Pick something')

        password(name: 'PASSWORD', defaultValue: 'SECRET', description: 'Enter a password')
    }
    stages {
        stage('Example') {
            steps {
                echo "Hello ${params.PERSON}"

                echo "Biography: ${params.BIOGRAPHY}"

                echo "Toggle: ${params.TOGGLE}"

                echo "Choice: ${params.CHOICE}"

                echo "Password: ${params.PASSWORD}"
            }
        }
    }
}
```

### triggers
The triggers directive defines the automated ways in which the Pipeline should be re-triggered. For Pipelines which are integrated with a source such as GitHub or BitBucket, triggers may not be necessary as webhooks-based integration will likely already be present. The triggers currently available are cron, pollSCM and upstream.

cron
Accepts a cron-style string to define a regular interval at which the Pipeline should be re-triggered, for example: triggers { cron('H */4 * * 1-5') }

pollSCM
Accepts a cron-style string to define a regular interval at which Jenkins should check for new source changes. If new changes exist, the Pipeline will be re-triggered. For example: triggers { pollSCM('H */4 * * 1-5') }

upstream
Accepts a comma-separated string of jobs and a threshold. When any job in the string finishes with the minimum threshold, the Pipeline will be re-triggered. For example: triggers { upstream(upstreamProjects: 'job1,job2', threshold: hudson.model.Result.SUCCESS) }

## tools
A section defining tools to auto-install and put on the PATH. This is ignored if agent none is specified.

*Supported Tools
	* maven
	* jdk
	* gradle
## input
The input directive on a stage allows you to prompt for input, using the input step. The stage will pause after any options have been applied, and before entering the agent block for that stage or evaluating the when condition of the stage. If the input is approved, the stage will then continue. Any parameters provided as part of the input submission will be available in the environment for the rest of the stage.

### Configuration options
* message
Required. This will be presented to the user when they go to submit the input.

* id
An optional identifier for this input. The default value is based on the stage name.

* ok
Optional text for the "ok" button on the input form.

* submitter
An optional comma-separated list of users or external group names who are allowed to submit this input. Defaults to allowing any user.

* submitterParameter
An optional name of an environment variable to set with the submitter name, if present.

* parameters
An optional list of parameters to prompt the submitter to provide. See parameters for more information.

Example 14. Input Step, Declarative Pipeline
```
pipeline {
    agent any
    stages {
        stage('Example') {
            input {
                message "Should we continue?"
                ok "Yes, we should."
                submitter "alice,bob"
                parameters {
                    string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
                }
            }
            steps {
                echo "Hello, ${PERSON}, nice to meet you."
            }
        }
    }
}
```
### when
The when directive allows the Pipeline to determine whether the stage should be executed depending on the given condition. The when directive must contain at least one condition. If the when directive contains more than one condition, all the child conditions must return true for the stage to execute. This is the same as if the child conditions were nested in an allOf condition (see the examples below). If an anyOf condition is used, note that the condition skips remaining tests as soon as the first "true" condition is found.

More complex conditional structures can be built using the nesting conditions: not, allOf, or anyOf. Nesting conditions may be nested to any arbitrary depth.

#THIS and other topics need to complete pipeline topic:
Sequential Stages
Parallel
Matrix
axes
stages
excludes (optional)
Matrix cell-level directives (optional)
Steps
script
Scripted Pipeline
Flow Control
Steps etc 

refrence : https://www.jenkins.io/doc/book/pipeline/syntax/


-----------------------------------------------------------
----------------------------------------------------------


## Blue ocean - only plugin ( just another UI of jenkins ) so no need to learn more

-----------------------------------------------------------
----------------------------------------------------------


## Managing Jenkins :
### Configuration as Code 
The Jenkins Configuration as Code (JCasC) feature defines Jenkins configuration parameters in a human-readable YAML file that can be stored as source code. This essentially captures the configuration parameters and values that are used when configuring Jenkins from the web UI. The configuration can then be modified by editing this file and then applying it.

JCasC provides the convenience and flexibility of configuring controllers without using the UI. It does not require more understanding of the configuration parameters than is required to configure Jenkins through the UI and it provides some checks on the values that are provided.

The JCasC configuration file can be checked into an SCM, which enables you to determine who made what modifications to the configuration and to roll back to a previous configuration if necessary.

The Configuration as Code plugin must be installed on the Jenkins controller that you will use to build out your JCasC configuration. If you do not see the Configuration as Code tile in the System Configuration section of the Manage Jenkins page on your dashboard, you need to install the plugin.

file example : https://medium.com/globant/jenkins-jcasc-for-beginners-819dff6f8bc
It only stores jenkins conf. not build details and all.

### Managing Plugins 

* Advanced installation
The Update Center only allows the installation of the most recently released version of a plugin. In cases where an older release of the plugin is desired, a Jenkins administrator can download an older .hpi archive and manually install that on the Jenkins controller.

Jenkins stores plugins it has downloaded in the plugins directory with a .jpi suffix, whether the plugins originally had a .jpi or an .hpi suffix.

If an administrator manually copies a plugin archive into the plugins directory, it should be named with a .jpi suffix to match the file names used by plugins installed from the update center.

* From the web UI
Assuming a .hpi file has been downloaded, a logged-in Jenkins administrator may upload the file from within the web UI:

Navigate to the Manage Jenkins > Plugins page in the web UI.

Click on the Advanced tab.

Choose the .hpi file from your system or enter a URL to the archive file under the Deploy Plugin section.

Deploy the plugin file.
* On the controller
Assuming a .hpi file has been explicitly downloaded by a system administrator, the administrator can manually place the file in a specific location on the file system.

Copy the downloaded .hpi` file into the JENKINS_HOME/plugins directory on the Jenkins controller (for example, on Debian systems JENKINS_HOME is generally /var/lib/jenkins). If an administrator manually copies a plugin archive into the plugins directory, it should be named with a .jpi suffix to match the file names used by plugins installed from the update center.

The controller must be restarted before the plugin is loaded and made available in the Jenkins environment.


A plugin may also be uninstalled by removing the corresponding .jpi file from the JENKINS_HOME/plugins directory on the controller. The plugin will continue to function until the controller has been restarted.

If a plugin file is removed but required by other plugins, the Jenkins controller may fail to boot correctly.

* Uninstalling a plugin does not remove the configuration that the plugin may have created. If there are existing jobs/nodes/views/builds/etc configurations that reference data created by the plugin, during boot Jenkins will warn that some configurations could not be fully loaded and ignore the unrecognized data. *

Since the configuration(s) will be preserved until they are overwritten, re-installing the plugin will result in those configuration values reappearing.

* Removing old data
Jenkins provides a facility for purging configuration left behind by uninstalled plugins. Navigate to Manage Jenkins and then click on Manage Old Data to review and remove old data.

* A systems administrator may also disable a plugin by creating a file on the Jenkins controller, such as: JENKINS_HOME/plugins/PLUGIN_NAME.jpi.disabled.


### plugins using the cli : Using the Jenkins CLI
https://www.jenkins.io/doc/book/managing/plugins/#using-the-jenkins-cli


The Plugin Manager allows plugins to be explicitly unpinned. The JENKINS_HOME/plugins/PLUGIN_NAME.hpi.pinned file can also be manually created/deleted to control the pinning behavior. If the pinned file is present, Jenkins will use whatever plugin version the user has specified. If the file is absent, Jenkins will restore the plugin to the default version on startup.


refrence : https://www.jenkins.io/doc/book/managing/plugins/

-----------------------------------------------------------
----------------------------------------------------------

## About Jenkins 
The Manage Jenkins >> About Jenkins page shows the current release of Jenkins on your system plus information about licenses for all components. The top of the display shows the release and version of Jenkins that is running.

The following information about your Jenkins cluster is also provided:

	* List of all third-party libraries used for this release of Jenkins, with links to licensing details about each library.

	* List of static resources that are installed.

	* List of installed plugins, each of which includes a link to the page that shows all third-party dependencies for each plugin with a link to licensing details about each library.


refrence : https://www.jenkins.io/doc/book/managing/about-jenkins/

-----------------------------------------------------------
----------------------------------------------------------

## System Information 
The Manage Jenkins >> System Information page provides detailed information about what is available on this Jenkins instance:

System Properties that can be used as arguments to the command line used to start Jenkins.

	* Environment Variables recognized on this system, with current values. This includes the environment variables defined by Jenkins and available on all systems as well as environment variables associated with plugins installed on this instance.

	* List of Plugins installed on the system.

	* Memory Usage gives a graph that shows the current memory usage for this instance.

refrence : https://www.jenkins.io/doc/book/managing/system-info/

-----------------------------------------------------------
----------------------------------------------------------


Jenkins Features Controlled with System Properties 
hudson and other system plugins and we can change the properties from CLI. 
refrence : https://www.jenkins.io/doc/book/managing/system-properties/

-----------------------------------------------------------
----------------------------------------------------------

Change System Time Zone 
refrence : https://www.jenkins.io/doc/book/managing/change-system-timezone/

-----------------------------------------------------------
----------------------------------------------------------

jenkins cli
refrence : https://www.jenkins.io/doc/book/managing/cli/

script console 
refrence : https://www.jenkins.io/doc/book/managing/script-console/

Groovy Hook Scripts 
refrence : https://www.jenkins.io/doc/book/managing/groovy-hook-scripts/



-----------------------------------------------------------
----------------------------------------------------------

## Managing Nodes 

* Jenkins controller
	* The Jenkins controller is the Jenkins service itself and where Jenkins is installed. It is also a web server that also acts as a "brain" for deciding how, when, and where to run tasks. Management tasks such as configuration, authorization, and authentication are executed on the controller, which serves HTTP requests. Files written when a Pipeline executes are written to the filesystem on the controller, unless they are off-loaded to an artifact repository such as Nexus or Artifactory.

* Nodes
	* Nodes are the "machines" on which build agents run. Jenkins monitors each attached node for disk space, free temp space, free swap, clock time/sync, and response time. A node is taken offline if any of these values go outside the configured threshold. Jenkins supports two types of nodes:

* built-in node
	* The built-in node is a node that exists within the controller process. It is possible to use agents and the build-in node to run tasks. However, running tasks on the built-in node is discouraged for security, performance, and scalability reasons. The number of executors configured for the node determines the node’s ability to run tasks. Set the number of executors to 0 to disable running tasks on the built-in node.



* Executors
	* An executor is a slot for the execution of tasks. Effectively, it is a thread in the agent. The number of executors on a node defines the number of concurrent tasks that can run. In other words, this determines the number of concurrent Pipeline stages that can execute at the same time. Determine the correct number of executors per build node must be determined based on the resources available on the node and the resources required for the workload. When determining how many executors to run on a node, consider CPU and memory requirements, as well as the amount of I/O and network activity:

		* One executor per node is the safest configuration.

		* One executor per CPU core can work well, if the tasks running are small.

		* Monitor I/O performance, CPU load, memory usage, and I/O throughput carefully when running multiple executors on a node.


refrence : https://www.jenkins.io/doc/book/managing/nodes/

-----------------------------------------------------------
----------------------------------------------------------
## In-process Script Approval 
To protect Jenkins from execution of malicious scripts, these plugins execute user-provided scripts in a Groovy Sandbox that limits the internal APIs that are accessible. This protection is provided by the Script Security plugin. As soon as an unsafe method is used in any of the scripts, the administrator can use the "In-process Script Approval" action appears in Manage Jenkins to allow the unsafe method. Unsafe methods should not be enabled without careful consideration of the impact.


refrence : https://www.jenkins.io/doc/book/managing/script-approval/




-----------------------------------------------------------
----------------------------------------------------------

## User Content 
Jenkins has a mechanism known as "User Content", where administrators can place files inside $JENKINS_HOME/userContent, and these files are served from http://yourhost/jenkins/userContent. This can be thought of as a mini HTTP server to serve images, stylesheets, and other static resources that you can use from various description fields inside Jenkins.

* Note that these files are not subject to any access controls beyond requiring Overall/Read access.
* See Git userContent plugin for how to manage these files through a Git repository.

  refrence : https://www.jenkins.io/doc/book/managing/user-content/

  

-----------------------------------------------------------
----------------------------------------------------------

## Spawning Processes From Build 

refrence : https://www.jenkins.io/doc/book/managing/spawning-processes/

-----------------------------------------------------------
----------------------------------------------------------
## Architecting for Scale 
* Vertical growth is when the load on a Jenkins controller load is increased by having more configured jobs or orchestrating more frequent builds. This may also mean that more teams are depending on that one controller.

* Horizontal growth is the creation of additional Jenkins controllers to accommodate new teams or projects, rather than adding new teams or projects to an existing controller.

There are potential pitfalls associated with each approach to scaling Jenkins, but with careful planning, many of them can be avoided or managed. Here are some things to consider when choosing a strategy for scaling your organization’s Jenkins instances:  REFER TO DOCS


## Calculating how many jobs, controllers, and executors are needed
Having the best possible estimate of necessary configurations for a Jenkins installation allows an organization to get started on the right foot with Jenkins and reduces the number of configuration iterations needed to achieve an optimal installation. The challenge for Jenkins architects is that true limit of vertical scaling on a Jenkins controller is constrained by whatever hardware is in place for the controller, as well as harder to quantify pieces like the types of builds and tests that will be run on the build nodes.

There is a way to estimate roughly how many controllers, jobs and executors will be needed based on build needs and number of developers served. These equations assume that the Jenkins controller will have 5 cores with one core per 100 jobs (500 total jobs/controller) and that teams will be divided into groups of 40.

If you have information on the actual number of available cores on your planned controller, you can make adjustments to the "number of controllers" equations accordingly.
The equation for estimating the number of controllers and executors needed when the number of configured jobs is known is as follows:

controllers = number of jobs/500
executors = number of jobs * 0.03
The equation for estimating the maximum number of jobs, controllers, and executors needed for an organization based on the number of developers is as follows:

number of jobs = number of developers * 3.333
number of controllers = number of jobs/500
number of executors = number of jobs * 0.03
These numbers will provide a good starting point for a Jenkins installation, but adjustments to actual installation size may be needed based on the types of builds and tests that an installation runs.




## Finding your $JENKINS_HOME
* Windows :  $JENKINS_HOME will be "C:\Program Files (x86)\jenkins". You can edit the location of your $JENKINS_HOME by opening the jenkins.xml file and editing the $JENKINS_HOME variable, or going to the "Manage Jenkins" screen, clicking on the "Install as Windows Service" option in the menu, and then editing the "Installation Directory" field to point to another existing directory.
* Linux : By default, $JENKINS_HOME is set to /var/lib/jenkins and $JENKINS_WAR is set to /usr/share/java/jenkins.war.
You can edit the location of $JENKINS_HOME by running systemctl edit jenkins and adding the following:

## Anatomy of a $JENKINS_HOME
The folder structure for a $JENKINS_HOME directory is as follows:

```
JENKINS_HOME
 +- config.xml     (Jenkins root configuration file)
 +- *.xml          (other site-wide configuration files)
 +- identity.key   (RSA key pair that identifies an instance)
 +- secret.key     (deprecated key used for some plugins' secure operations)
 +- secret.key.not-so-secret  (used for validating _$JENKINS_HOME_ creation date)
 +- userContent    (files served under your https://server/userContent/)
 +- secrets        (root directory for the secret+key for credential decryption)
     +- hudson.util.Secret   (used for encrypting some Jenkins data)
     +- master.key           (used for encrypting the hudson.util.Secret key)
     +- InstanceIdentity.KEY (used to identity this instance)
 +- fingerprints   (stores fingerprint records, if any)
 +- plugins        (root directory for all Jenkins plugins)
     +- [PLUGINNAME]   (sub directory for each plugin)
         +- META-INF       (subdirectory for plugin manifest + pom.xml)
         +- WEB-INF        (subdirectory for plugin jar(s) and licenses.xml)
     +- [PLUGINNAME].jpi   (.jpi or .hpi file for the plugin)
 +- jobs           (root directory for all Jenkins jobs)
     +- [JOBNAME]      (sub directory for each job)
         +- config.xml     (job configuration file)
         +- workspace      (working directory for the version control system)
         +- latest         (symbolic link to the last successful build)
         +- builds         (stores past build records)
             +- [BUILD_ID]     (subdirectory for each build)
                 +- build.xml      (build result summary)
                 +- log            (log file)
                 +- changelog.xml  (change log)
     +- [FOLDERNAME]   (sub directory for each folder)
         +- config.xml     (folder configuration file)
         +- jobs           (sub directory for all nested jobs)
```

## Choosing a backup strategy

When it comes to creating a backup, it is recommended to exclude archiving the following folders to reduce the size of your backup:

/war      (the exploded Jenkins war directory)
/cache    (downloaded tools)
/tools    (extracted tools)
System configurations
Your instance’s system configurations exist in the root level of the $JENKINS_HOME folder:

 +- config.xml     (Jenkins root configuration file)
 +- *.xml          (other site-wide configuration files)

 
## Resilient Jenkins Architecture
Step 1: Make each controller highly available
Step 2: Enable security
Step 3: Add build nodes (agents) to controller
Step 4: Setup a test instance

refrence : https://www.jenkins.io/doc/book/scaling/architecting-for-scale/

-----------------------------------------------------------
----------------------------------------------------------



## This is the simplest way of obtaining thread dumps.

If Jenkins or its build agents are operating normally, you can obtain a thread dump remotely by going to http://your.jenkins.server/threadDump. For an agent named 'xyz', go to http://your.jenkins.server/computer/xyz/systemInfo. You need to have the administrator permission on the system.
