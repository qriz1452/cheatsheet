# YARN :
- package manager
- javascript app
- ensure correct veriosn of dependencues are reolved
- 
- faster package installation than npm
- uses caching mechanism
- bcz of cache it speeds up update and subsequesnt installation
- 
- yarn.lock locks down  version of all dependncies
- auto generate
- should not edit manually , else we may  face conf drift
- lockfile ensure all envt should have same version of dependencies

- yarn uses its own algorithm to install dependency
- helps to avoid dependency hell

```diff
- commands :
```

```
- yarn install --> install dependences based on yarn lock file
- yarn add package
- yarn remove package
- yarn upgrade package
- yarn outdated
- yarn audit
- yarn add -D packgae
- yarn add -E package
- yarn global add package
- yarn remove package
- yarn list
- yarn list --depth=0
- yarn global list --depth=0
- yarn list --pattern "gulp|grunt" --depth=0
- yarn autoclean
- yarn outdated
- yarn why <query>
- yarn cache clean
-
```
```
yarn install
--no-lockfile    // Do not generate or update the yarn.lock file
--pure-lockfile  // Use only the existing yarn.lock and do not generate a new one
--frozen-lockfile    // Install using only the dependencies specified in yarn.lock
--silent     // Suppress detailed logging for a cleaner console output
--offline    // Use only locally cached packages and do not fetch new dependencies
--update-checksums    // Update checksums in the yarn.lock file
--check-files   // Verify that all installed package files are present and valid
--flat     // Install dependencies in a flat directory structure
--force    // Reinstall all packages, even if they are already present
--ignore-scripts   // Skip execution of package installation scripts
--modules-folder <path>   // Specify a custom location for node_modules
--production[=true|false]   // Install only production dependencies (default: false)
```
-
 

```
yarn add
--dev      // Add the package as a development dependency
--peer     // Add the package as a peer dependency
--optional // Add the package as an optional dependency
--exact    // Install the exact version of the package
--tilde    // Install the most recent release with the same minor version
```
- can override a package version by adding version in package json filr




- can create workspace within single project like envt we use in python application

- can use private repo hosted at npm , jfrog , nexus etc
- for this we need to update yarnrc config file
- this file has URL ,network settings etc

- yarn.lock should be version controlled
- yarn install will auto generate lock file based on package.json


```diff
+  DEVOPS: 

```

- yarn package manager for js
- npm takes long time to install dependencies ,
- by using yarn build time has been reduced
- which helped other jobs to execute earlier --> directly saved the cost ( executer , thread , cpu , cost , employee resource )
- yarn.lock file to install dependencies, consistet build across various envt
- sometimes conflict may arose but we should not edit file manually to avoid conf drift
- use yarn add , yarn remove etc command to add dependencies
- sometimes yarn registry may down , in this case we can use our self hosted registry



- uses jenkins for ci , used yarn.lock which guaranteed every build and deployment has same dependencies
- when building docker container   Yarn's caching mechanism conflicted with Docker's layer caching,
- which loaded outdated dependencies in container and testings were gettng failed
- and later we may get error in k8 when deploying the container to clusters eveb if we use yarnlock file
- and if we are using ansible and terraform to manage iac we should pass cli argument in script and we should check again whether the correct verion and dependcies are installed or not.
- reg docker, docker uses layered fs , where each layer represents a change or addition to the filesystem. These layers are cached, which speeds up subsequent builds by reusing unchanged layers. However, this caching behavior can sometimes clash with how Yarn handles its own caching.
-  Let's say we have a Dockerfile for an application that uses Yarn to install its JavaScript dependencies. During the build process, Yarn fetches and installs the required packages, and the cached dependencies are saved in its cache. Now, if we rebuild the container with no changes to the application code but with changes to, let's say, the environment setup, Docker's caching would kick in. It would reuse the layers from the previous build, including the cached layers that Yarn had used.
-  This clash between Yarn's cached dependencies and Docker's cached layers could potentially result in outdated packages being included in the new container. And since containers are meant to be isolated and reproducible, this is a serious concern. Our applications might behave unexpectedly due to these inconsistencies.
- To address this, we had to carefully manage both Yarn's caching and Docker's caching. We explored a couple of strategies. First, we adjusted our Dockerfiles to ensure that the Yarn install step happened after any other environment setup. This way, we minimized the chances of Docker's cache affecting the Yarn dependencies. Additionally, we made use of Docker's build-time arguments to invalidate cache when certain parts of the Dockerfile changed, ensuring that Yarn's dependencies were fetched fresh when needed.
- github issue 5573

-
```
# Use a base image suitable for your project
FROM node:14

# Set the working directory
WORKDIR /app

# Copy package.json and yarn.lock to the container
COPY package.json yarn.lock /app/

# Install global Yarn (if needed)
RUN npm install -g yarn

# Run yarn install separately to utilize Docker caching
RUN yarn install --frozen-lockfile

# Copy the rest of the application code to the container
COPY . /app/

# Your further Dockerfile steps
```




