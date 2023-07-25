# Anatomy of a package json File

```Node.js```

If you’ve been working on JavaScript and/or Node.js projects at any capacity these past few years, surely you’ve come across some package.json files, npm’s configuration file for projects and modules. In this post we’ll explore some of the most important keys and values found in a typical package.json file.


# Initiating a Project


The easiest and fastest way to initiate a project with the npm package manager is using the init command and the -y flag, which answers yes to all the questions:


```
$ npm init -y

```



The name of the project will be the same as the name of the current folder you’re in.

The init command will write a package.json file to the current directory with JSON content that looks like this:


```
{
  "name": "hello-alligator",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}

```


Let’s go over each of the keys in an initial package.json file:


- name: The name for the project, which should be all lower case and URL-safe. The name can be prefixed with a scope (i.e.: @angular/angular-cli) and is optional if the project is private, but if the project is to be published publicly the name is required and must be unique on the npm repository.
- version: A version number that should be understandable by node-semver. This is also optional for private and required and very important for public modules.
- description: A description for the project. This is optional and is useful if you can the project to be found easily on the repository.
- main: The entry file for the project.
- scripts: The scripts key expects an object with script names as keys and commands as values. This is useful to specify scripts that can be run directly from the command line and that can do all sorts of things like starting your project on a local server, building for production or running your tests. Chances are that scripts is where you’ll make the most manual changes in a typical package.json file.
- keywords: An array of keywords to helps with finding the module on the npm repository.
- author: The author field expects an object with keys for name, email and url. It makes it easy for people to then get in touch with the project’s owner.
- license: Expects a license name using its SPDX identifier. It defaults to the ISC license, and MIT would be another popular license choice. You can also use UNLICENSED for projects that are private and closed-source.

# Managing Dependencies


npm’s main strength is its ability to easily manage a project’s dependencies. It’s therefore only natural that the package.json file for a project centers mostly around specifying the dependencies for a project. There’s the regular dependencies, but there can also be devDependencies, peerDependencies, optionalDependencies and bundledDependencies. Let’s go over them:


- dependencies: Normal project dependencies. This is where the bulk of your dependencies will most likely be. Add such dependencies to your project using $ npm install my-dependency.
- devDependencies: Dependencies that are useful only when working on the project. For example, testing libraries and transpilers should most often be added as devDependencies. Add a devDependency using npm’s --save-dev flag with the install command.
- optionalDependencies: Dependencies that should be treated as optional by npm. That means that npm won’t complain or fail to install when optional dependencies can’t be met. Install optional dependencies using the --save-optional with the install command.
- bundledDependencies: Expects an array of package names, which will be bundled with the project. Use the --save-bundle flag with the install command for a dependency to also be added to the list of bundled dependencies.
- peerDependencies: Peer dependencies are useful for specifying modules that your project depends on. This way, npm will issue a warning when some peer dependencies are missing.

As you’ve probably seen before, dependencies can accept different formats to specify which versions or range of versions can be used for a dependency. For example:


- 2.4.2: Exactly version 2.4.2
- ^2.4.2: The latest version compatible with version 2.4.2
- ~2.4.2: Works for versions such as 2.4.2, 2.4.3, 2.4.4,…
- ~2.4: This will work for versions such as 2.4, 2.5, 2.6,…
- 2.4.x: Works with any patch version of the package’s 2.4 version
- 2.x: Works for any minor version of the package’s 2 version
- >=2.4: A version greater or equal to 2.4. You can also use <, <= or >
- 2.4.2 3.1.1: Any version between and including version 2.4.2 and version 3.1.1

You can even specify multiple possible version ranges by separating each range with ||.


# More Useful Configuration Keys


There’s more configuration that can optionally go into your project’s package.json file, so let’s briefly touch on some of the most useful configurations:


- browser: Use the browser key instead of main for projects that are meant to be used in a browser instead of a server.
- private: If this key is set to true, the project won’t be able to be published publicly to the npm repository. This is useful if you want to prevent accidentally publishing a project to the world.
- engines: Use engines to specify specific versions of Node.js and/or npm that the project works with. It takes an object with a key for node and/or npm and a value that looks like what you’d have as values for dependencies that specifies a range of versions.
- homepage: The URL for the home page of the project.
- bugs: A URL where issues and bugs can be reported. This will often be an URL to the Github issues page for a project.
- files: This optional key expects an array of files that are included when the project is added a dependency to another project. File names/paths can use a glob pattern and when the files key is not provided a default value of ["*"] is used, which means that all the files will be included. Not to worry though, certain files/folders such as .git and nome_modules are always ignored.

With this, you should have a pretty good general idea of what can go into your package.json configuration. You can also reference the official documentation for fine grained details on all the possible configuration keys.


