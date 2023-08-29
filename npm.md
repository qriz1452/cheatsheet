  # NPM Cheat Sheet

- npm or the Node Package Manager, is one of the most used tools for any Node.js developer. Here's a list of the most common commands you'll use when working with npm. 

- Install package.json dependencies ` npm install `

- Shorthand
```
# install
npm i <package>

# uninstall
npm un <package>

# update
npm up <package>
```

- Flags
`-S` is the same as `--save`,
 and `-D` is the same as `--save-dev`.

- List globally installed packages ` npm list -g --depth=0 `
- Uninstall global package `npm -g uninstall <name> `
- Upgrade npm on Windows `npm-windows-upgrade`
- Update global packages
- To see which packages need updating, use: `npm outdated -g --depth=0`
- To update global packages individually you can use: `npm update -g <package> <package> <package>`
- list available scripts to ru `npm run`
- Update npm `npm install -g npm@latest`
-  using windows? Then use `npm-windows-upgrade`
- Installed version `npm list # for local packages`

-Node Version Manager nvm
nvm makes it easy to switch between different versions of Node.js. Read more about it on the project's GitHub page.

- Once you have nvm installed, if you want to install the latest version of Node v12 just run: `nvm install 12`
- If you have multiple versions of Node.js installed on your workspace, you can switch to a specific version by writing: `nvm use 10.19.0`

- Make a Node version default :  `nvm alias default 12`
Where the latest version of 12 is the version you want to be used by default.

- Update npm : `nvm install-latest-npm`

---------------------------------------------------------------------

-  Package management
`npm i`	Alias for npm install
`npm install`	Install everything in package.json
`npm install --production`	Install everything in package.json, except devDependecies
`npm install lodash`	Install a package
`npm install --save-dev lodash`	Install as devDependency
`npm install --save-exact lodash`	Install with exact
`npm version 1.2.3`	Bump the package version to 1.2.3
`npm version major`	Bump the major package version by 1 (1.2.3 → 2.0.0)
`npm version minor`	Bump the minor package version by 1 (1.2.3 → 1.3.0)
`npm version patch`	Bump the patch package version by 1 (1.2.3 → 1.2.4)



-  Install names
`npm i sax`	NPM package
`npm i sax@latest`	Specify tag latest
`npm i sax@3.0.0`	Specify version 3.0.0
`npm i sax@">=1 <2.0"`	Specify version range
`npm i @org/sax`	Scoped NPM package
`npm i user/repo`	GitHub
`npm i user/repo#master`	GitHub
`npm i github:user/repo`	GitHub
`npm i gitlab:user/repo`	GitLab
`npm i /path/to/repo`	Absolute path
`npm i ./archive.tgz`	Tarball
`npm i https://site.com/archive.tgz`	Tarball via HTTP

- Updating
`npm update`	Update production packages
`npm update --dev`	Update dev packages
`npm update -g`	Update global packages
`npm update lodash`	Update a package

- Listing
`npm list`	Lists the installed versions of all dependencies in this software
`npm list -g --depth 0`	Lists the installed versions of all globally installed packages
`npm view`	Lists the latest versions of all dependencies in this software
`npm outdated`	Lists only the dependencies in this software which are outdated

- Misc features
`npm owner add USERNAME PACKAGENAME`  Add someone as an owner
`npm ls`  list packages
`npm deprecate PACKAGE@"< 0.2.0" "critical bug fixed in v0.2.0"`  Adds warning to those that install a package of old versions
`npm update [-g] PACKAGE`   update all packages, or selected packages
`npm outdated [PACKAGE]`   Check for outdated packages

- Removing
`npm rm lodash`	Remove package production packages


- Get Version
```
npm -v
npm --version
```

- Get Help
```
npm

npm help
```
- Create package.json
```
npm init

# Below commands will create package.json file with default values
npm init -y

npm init --yes
```

-  Set Defaults
```
npm config set init-author-name 'YOUR NAME'

npm config set init-license 'MIT'
Get Defaults
npm config get init-author-name

npm config get init-license
```
-  Remove Defaults
```
npm config delete init-author-name

npm config delete init-license
```
-  Install Packages Globally
```npm install -g package-name```
-  Production dependency
```npm install --save package-name```
-  Development dependency
```npm install --save-dev package-name```
-  Install Certain Version
```
#Globally
npm install -g package-name@package-version
#Production dependency
npm install --save package-name@package-version
#Development dependency
npm install --save-dev package-name@package-version
```
-  Update Packages
```
#Globally
npm update -g package-name
#Production dependency
npm update --save package-name
#Development dependency
npm update --save-dev package-name
```
-  Remove Packages
```
#Globally
npm uninstall -g package-name
#Production dependency
npm uninstall --save package-name
#Development dependency
npm uninstall --save-dev package-name
```
-  Move to Another Folder
NPM stores installed packages inside node_modules folder.
Most people usually doesn't share node_modules folder along with their code, because you can easily install all the NPM packages they have used using below commands.

-  Install production and development dependencies.
```
npm install
# Install production dependencies only.
npm install --production
```
-  Find Root Folder
```
# Globally
npm root -g
# Locally
npm root
```

-  List Packages
```
# Globally
npm list -g

npm list -g --depth 0

npm list -g --depth 1
# Locally
npm list

npm list --depth 0

npm list --depth 1
```
-  NPM Scripts
Define scripts in package.json file.
```
"scripts": {
  "start": "node index.js",
  "script-name":"command-to-run"
}

# Run scripts
# Only "start" script will execute without run command
npm start

npm run script-name
```


-  Publish a package not under the default "latest" tag

`npm publish --tag beta`

-  Package Version
Find below, What package version with different symbols (*, ~, ^) represents in package.json file.

| Version	| Result |
|---------| -------|
|"*"	| Install package with latest version |
|"4.17.3"	| Install package with exact version |
|"~4.17.3"	| Install package with latest patch update (Highlighted part gets updated)|
|"^4.17.3"	| Install package with latest minor update (Highlighted part gets updated)|
Most Preferred
"package-name": "^4.17.3"

### Security best practices : https://cheatsheetseries.owasp.org/cheatsheets/NPM_Security_Cheat_Sheet.html


===========================================

```diff
+ DEVOPS
```

1. **Package Management**:
   - npm is the default package manager for Node.js and is used to install, manage, and update libraries and packages that your project depends on.
   - It provides a command-line interface (CLI) that allows you to interact with your project's dependencies easily.

2. **Package.json**:
   - The `package.json` file is at the heart of any Node.js project. It contains metadata about the project as well as a list of dependencies and other configuration settings.
   - DevOps engineers often manage `package.json` files to ensure consistent and reproducible builds across different environments.

3. **Dependencies**:
   - npm distinguishes between two types of dependencies: `dependencies` and `devDependencies`.
   - `dependencies` are packages required for the production runtime of your application.
   - `devDependencies` are packages required only during development and testing.

4. **Semantic Versioning**:
   - npm uses Semantic Versioning (SemVer) to manage package versions. Versions are specified in the `package.json` file using the format `MAJOR.MINOR.PATCH`.

5. **Installing Packages**:
   - To install dependencies listed in the `package.json` file, use the command `npm install`.
   - Use `npm install --save-dev` to install packages as dev dependencies.

6. **Global vs. Local Packages**:
   - Packages can be installed globally (system-wide) using the `-g` flag with `npm install`. However, for project-specific dependencies, it's recommended to install them locally.

7. **Security and Auditing**:
   - npm provides tools for auditing the security of your project's dependencies to identify vulnerabilities. Use `npm audit` and `npm audit fix` to manage security concerns.

8. **Updating Packages**:
   - Keep dependencies up to date by running `npm update` or specifying the desired version in the `package.json` file and running `npm install`.

9. **Scripts**:
   - npm allows you to define custom scripts in the `package.json` file. These scripts can be executed using `npm run <script-name>`.
   - Common scripts include `start`, `test`, and custom deployment or build scripts.

10. **Version Control**:
    - The `node_modules` directory, where npm stores package dependencies, should not be included in version control. Instead, commit the `package.json` and `package-lock.json` (or `yarn.lock`) files.

11. **Lock Files**:
    - npm generates a `package-lock.json` file that locks down the exact versions of all dependencies to ensure consistent installations across different environments.

12. **Yarn vs. npm**:
    - While npm is the default package manager, Yarn is an alternative that offers faster and more reliable dependency management. DevOps engineers might evaluate which one best fits their organization's needs.

13. **Caching**:
    - npm caches packages locally to speed up installations. However, occasionally clearing the cache might be necessary to resolve issues.

14. **Publishing Packages**:
    - If your organization develops reusable packages, you can publish them to the npm registry. Use `npm publish` after logging into your npm account.

15. **Continuous Integration/Continuous Deployment (CI/CD)**:
    - Incorporate npm commands into your CI/CD pipeline for automated testing, building, and deployment.

16. **Proxy and Registry Configuration**:
    - For security or organization-specific reasons, you might need to configure npm to work through proxies or private registries.

----------------------------------


