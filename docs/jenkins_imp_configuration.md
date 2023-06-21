# imp points :

* Most Jenkins configuration changes can be made through the Jenkins user interface or through the configuration as code plugin.

## Networking parameters
Jenkins networking configuration is generally controlled by command line arguments. The networking configuration arguments are:

Table 1. Jenkins Networking Command Line Parameters
Command Line Parameter	Description
--httpPort=$HTTP_PORT

Runs Jenkins listener on port $HTTP_PORT using standard http protocol. The default is port 8080. To disable (because you’re using https), use port -1. This option does not impact the root URL being generated within Jenkins logic (UI, inbound agent files, etc.). It is defined by the Jenkins URL specified in the global configuration.

--httpListenAddress=$HTTP_HOST

Binds Jenkins to the IP address represented by $HTTP_HOST. The default is 0.0.0.0 — i.e. listening on all available interfaces. For example, to only listen for requests from localhost, you could use: --httpListenAddress=127.0.0.1

--httpsPort=$HTTPS_PORT

Uses HTTPS protocol on port $HTTPS_PORT. This option does not impact the root URL being generated within Jenkins logic (UI, inbound agent files, etc.). It is defined by the Jenkins URL specified in the global configuration.

--httpsListenAddress=$HTTPS_HOST

Binds Jenkins to listen for HTTPS requests on the IP address represented by $HTTPS_HOST.

--http2Port=$HTTP_PORT

Uses HTTP/2 protocol on port $HTTP_PORT. This option does not impact the root URL being generated within Jenkins logic (UI, inbound agent files, etc.). It is defined by the Jenkins URL specified in the global configuration.

--http2ListenAddress=$HTTPS_HOST

Binds Jenkins to listen for HTTP/2 requests on the IP address represented by $HTTPS_HOST.

--prefix=$PREFIX

Runs Jenkins to include the $PREFIX at the end of the URL. For example, set --prefix=/jenkins to make Jenkins accessible at http://myServer:8080/jenkins

--sessionTimeout=$TIMEOUT

Sets the http session timeout value to $SESSION_TIMEOUT minutes. Default to what webapp specifies, and then to 60 minutes

* Jenkins passes all command line parameters to the Winstone servlet container.

https://www.jenkins.io/doc/book/installing/initial-settings/
------------------------------------------------------------------------------------------

Configuring HTTP
HTTPS with an existing certificate
If you’re setting up Jenkins using the built-in Winstone server and want to use an existing certificate for HTTPS:

--httpPort=-1 \
--httpsPort=443 \
--httpsKeyStore=path/to/keystore \
--httpsKeyStorePassword=keystorePassword

https://www.jenkins.io/doc/book/installing/initial-settings/

------------------------------------------------------------------------------------------
------------------------------------------------------------------------------------------

Credentials stored in Jenkins can be used:

anywhere applicable throughout Jenkins (i.e. global credentials),

by a specific Pipeline project/item (read more about this in the Handling credentials section of Using a Jenkinsfile),

by a specific Jenkins user (as is the case for Pipeline projects created in Blue Ocean).

Jenkins can store the following types of credentials:

Secret text - a token such as an API token (e.g. a GitHub personal access token),

Username and password - which could be handled as separate components or as a colon separated string in the format username:password (read more about this in Handling credentials),

Secret file - which is essentially secret content in a file,

SSH Username with private key - an SSH public/private key pair,

Certificate - a PKCS#12 certificate file and optional password, or

Docker Host Certificate Authentication credentials

Security :  
  * credentials configured in Jenkins are stored in an encrypted form on the controller Jenkins instance (encrypted by the Jenkins instance ID) and are only handled in Pipeline projects via their credential IDs.
  *  Matrix-based security).
  *  default Logged-in users can do anything setting or Anyone can do anything setting.

https://www.jenkins.io/doc/book/using/using-credentials/
-------------------------------------------------------------------------------------------------------------------------



Referencing another project by name 
In many places throughout Jenkins, you can refer to another project/job by name. For example, in a Pipeline Script, you might want to copy artifacts from another project:

copyArtifacts projectName: 'myproject'


Differentiating between multiple projects with the same name
If you’re using the Folders Plugin and you have multiple projects with the same name that are in different folders, you can differentiate between them using a path, similar to a Unix filesystem path. There are two types of paths: Absolute paths  , Relative paths


* if your Multibranch Pipeline has a branch with a slash in it (feature/myfeature), replace the slash with %2F:

mymultibranchproject/feature%2Fmyfeature

https://www.jenkins.io/doc/book/using/referencing-another-project-by-name/
------------------------------------------------------------------------------------------------------------------------------------



Aborting a build 
When you click that [x] icon from the UI, the following things happen:

Browser sends a request to the server.

The server interrupts (via Thread.interrupt()) the thread (AKA executor thread) that is responsible for carrying out a build.

The server returns

At this point, your browser is back, but the actual abort process happens from here asynchronously.

The thread gets the interrupt signal. How quickly this happens depends on what the executor is doing at the time of the interruption. Specifically, an executor thread can only be interrupted in "interruption points" due to the Java design.

Waiting for a completion of a child process (for example, maybe the build is running Ant) is an interruption point. That means if the executor was doing that, it gets interrupted instantaneously.

Waiting for a computation on an agent is an interruption point.

Waiting for file or network I/O is not an interruption point. This often causes the problem where a build appears to be un-abortable. For example, checking out a Subversion repository falls in this category.

Normal computation is also not an interruption point.

The executor performs a clean up operation. This depends on what it was doing by the time it noticed the interruption.

If it was waiting for a completion of a child process, Jenkins will search for all the descendant processes and kill them all. On Unix, this is done through java.lang.UnixProcess.destroyProcess, which sends SIGTERM on Unix-based JDK implementations. On Windows, this is done through TerminateProcess API.

If it was waiting for a completion of some computation in an agent, the thread that’s performing the remote computation is interrupted asynchronously. How quickly that threads gets interrupted depends on what that thread is doing. See above.

Executor starts unwinding the stack, and eventually it finishes the unwinding. At this point, the build is marked as aborted and executor returns to the idle status.

Pipeline jobs can be stopped by sending an HTTP POST request to URL endpoints of a build.

BUILD ID URL/stop - aborts a Pipeline.

BUILD ID URL/term - forcibly terminates a build (should only be used if stop does not work).

BUILD ID URL/kill - hard kill a pipeline. This is the most destructive way to stop a pipeline and should only be used as a last resort.

If your build isn’t aborting
Check the thread dump http://yourserver/jenkins/threadDump and look for the executor thread in question — they are named after the agent and executor number. That’ll normally tell you where the thread is, and often reveals why it’s not responding to an interruption.


https://www.jenkins.io/doc/book/using/aborting-a-build/
----------------------------------------------------------------------------------------------------------------

Tracking file usage across Jenkins jobs using fingerprints

Since recording fingerprints is a cheap operation, the simplest thing to do is just blindly record all fingerprints of the followings:

jar files that your project produce

jar files that your project rely on

The disk usage is affected more by the number of files fingerprinted, as opposed to the size of files or the number of builds they are used. So unless you have a plenty of disk space, you don’t want to fingerprint **/*.


Configure a job to Record Fingerprint(s) of a file or set of files
Go to your project, click Configure in the left navigation bar, then scroll down to the Post-build Actions section of the job

Click on the button to add a Post-build action.

Select Record fingerprints of files to track usage.

The post-build action configuration fields provide you with a pattern option to match the files you want to fingerprint as well as a couple check-box selections to do your file fingerprinting.

Maven job type does this automatically for its dependencies and artifacts.

How does it work?
The fingerprint of a file is simply an MD5 checksum. Jenkins maintains a database of MD5 checksums, and for each MD5 checksum, Jenkins records which builds of which projects used it. This database is updated every time a build runs and files are fingerprinted.

To avoid excessive disk usage, Jenkins does not store the actual file. Instead, it just stores MD5 checksums and their usages. These files can be seen in

$JENKINS_HOME/fingerprints

https://www.jenkins.io/doc/book/using/fingerprints/
-----------------------------------------------------------------------------------------------------------------

Remote Access API 
Jenkins provides machine-consumable remote access API to its functionalities. Currently it comes in three flavors:

XML

JSON with JSONP support

Python

Remote access API is offered in a REST-like style. That is, there is no single entry point for all features, and instead they are available under the ".../api/" URL where "..." portion is the data that it acts on.

For example, if your Jenkins installation sits at https://ci.jenkins.io, visiting https://ci.jenkins.io/api/ will show just the top-level API features available – primarily a listing of the configured jobs for this Jenkins instance.
Or if you want to access information about a particular build, e.g. https://ci.jenkins.io/job/Websites/job/jenkins.io/job/master/lastSuccessfulBuild/ , then go to https://ci.jenkins.io/job/Websites/job/jenkins.io/job/master/lastSuccessfulBuild/api/ and you’ll see the list of functionalities for that build.

What can you do with it?
Remote API can be used to do things like these:

retrieve information from Jenkins for programmatic consumption.

trigger a new build

create/copy jobs

Submitting jobs
Jobs without parameters

You merely need to perform an HTTP POST on JENKINS_URL/job/JOBNAME/build.

This also works for Multibranch Pipelines and Organization Folders. It would trigger a scan.

Jobs with parameters

Simple example - sending "String Parameters":

curl JENKINS_URL/job/JOB_NAME/buildWithParameters \
  --user USER:TOKEN \
  --data id=123 --data verbosity=high
Another example - sending a "File Parameter":

curl JENKINS_URL/job/JOB_NAME/buildWithParameters \
  --user USER:PASSWORD \
  --form FILE_LOCATION_AS_SET_IN_JENKINS=@PATH_TO_FILE
The symbol '@' is important in this example. Also, the path to the file is absolute path. In order to make this command work, you need to configure your Jenkins job to take a file parameter and match the File location field in the Jenkins job configuration with the key in the --form option.


Detecting Jenkins version
To check the version of Jenkins, load the top page or any .../api/* page and check for the X-Jenkins response header. This contains the version number of Jenkins, like "1.404" This is also a good way to check if an URL is a Jenkins URL.

https://www.jenkins.io/doc/book/using/remote-access-api/
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

