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

Command-line Pipeline Linter
Jenkins can validate, or "lint", a Declarative Pipeline from the command line before actually running it. This can be done using a Jenkins CLI command or by making an HTTP POST request with appropriate parameters. We recommended using the SSH interface to run the linter. See the Jenkins CLI documentation for details on how to properly configure Jenkins for secure command-line access.

Linting via the CLI with SSH

'''
# ssh (Jenkins CLI)
# JENKINS_PORT=[sshd port on controller]
# JENKINS_HOST=[Jenkins controller hostname]
ssh -p $JENKINS_PORT $JENKINS_HOST declarative-linter < Jenkinsfile
'''

or

Linting via HTTP POST using curl

'''
# curl (REST API)
# Assuming "anonymous read access" has been enabled on your Jenkins instance.
# JENKINS_URL=[root URL of Jenkins controller]
# JENKINS_CRUMB is needed if your Jenkins controller has CRSF protection enabled as it should
JENKINS_CRUMB=`curl "$JENKINS_URL/crumbIssuer/api/xml?xpath=concat(//crumbRequestField,\":\",//crumb)"`
curl -X POST -H $JENKINS_CRUMB -F "jenkinsfile=<Jenkinsfile" $JENKINS_URL/pipeline-model-converter/validate
'''

Blue Ocean Editor
The Blue Ocean Pipeline Editor provides a WYSIWYG way to create Declarative Pipelines. The editor offers a structural view of all the stages, parallel branches, and steps in a Pipeline. The editor validates Pipeline changes as they are made, eliminating many errors before they are even committed. Behind the scenes it still generates Declarative Pipeline code.

IDE Integrations
Eclipse / atom /sublime /vscode etc 

