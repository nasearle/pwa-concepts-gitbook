# Introduction to Gulp




## Contents




<a href="#introduction"><strong>Introduction</strong></a>

<a href="#what"><strong>What is gulp?</strong></a>

<a href="#how"><strong>How to set up gulp</strong></a> 

<a href="#tasks"><strong>Creating tasks</strong></a> 

<a href="#examples"><strong>Examples</strong></a>

<a href="#automation"><strong>More automation</strong></a>

<a href="#review"><strong>Review</strong></a>

<a href="#resources"><strong>Further reading</strong></a>

Codelab: <a href="https://google-developer-training.gitbooks.io/progressive-web-apps-ilt-codelabs/content/docs/lab_gulp_setup.html">Gulp Setup</a>

<a id="introduction" />


## Introduction




Modern web development has many repetitive tasks like running a local server, minifying code, optimizing images, preprocessing CSS and more. This text discusses <a href="http://gulpjs.com/">gulp</a>, a build tool for automating these tasks.

<a id="what" />


## What is gulp?




<a href="http://gulpjs.com/">Gulp</a> is a cross-platform, streaming task runner that lets developers automate many development tasks. At a high level, gulp reads files as streams and pipes the streams to different tasks. These tasks are code-based and use plugins. The tasks modify the files, building source files into production files. To get an idea of what gulp can do check the <a href="https://github.com/gulpjs/gulp/blob/master/docs/recipes/README.md">list of gulp recipes</a> on GitHub.

<a id="how" />


## How to set up gulp




Setting up gulp for the first time requires a few steps. 

### Node

Gulp requires <a href="https://nodejs.org/en/">Node</a>, and its package manager, <a href="https://www.npmjs.com/">npm</a>, which installs the gulp plugins.

If you don't already have Node and npm installed, you can install them with <a href="https://github.com/creationix/nvm">Node Version Manager</a> (nvm). This tool lets developers install multiple versions of Node, and easily switch between them.

<div class="note">
<strong>Note: </strong>If you have issues with a specific version of Node, you can <a href="https://github.com/creationix/nvm#usage">switch to another version</a> with a single command.
</div>

Nvm can then be used to install Node by running the following in the command line:

    nvm install node

This also installs Node's package manager, <a href="https://www.npmjs.com/">npm</a>. You can check that these are both installed by running the following commands from the command line:

    node -v

    npm -v

If both commands return a version number, then the installations were successful. 

### Gulp command line tool

Gulp's command line tool should also be installed globally so that gulp can be executed from the command line. Do this by running the following from the command line:

    npm install --global gulp-cli

### Creating a new project

Before installing gulp plugins, your application needs to be initialized. Do this by running the following command line command from within your project's working directory:

    npm init

This command begins the generation of a <strong>package.json</strong> file, prompting you with questions about your application. For simplicity these can all be left blank (either by skipping the prompts with the return key or by using <code>npm init -y</code> instead of the above command), but in production you could store <a href="https://nodesource.com/blog/the-basics-of-package-json-in-node-js-and-npm/">application metadata</a> here. The file looks like this (your values may be different):

#### package.json

```
{
  "name": "test",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}
```

Don't worry if you don't understand what all of these values represent, they are not critical to learning gulp. 

This file is used to track your project's packages. Tracking packages like this allows for quick reinstallation of all the packages and their dependencies in future builds (the <code>npm install</code> command will read <strong>package.json</strong> and automatically install everything listed).

<div class="note">
<strong>Note:</strong> It is a best practice not to push packages to version control systems. It's better to use <code>npm install</code> and <code>package.json</code> to install project packages locally. 
</div>

### Installing packages

Gulp and Node rely on plugins (packages) for the majority of their functionality. Node plugins can be installed with the following command line command:

    npm install pluginName --save-dev

This command uses the npm tool to install the <code>pluginName</code> plugin. Plugins and their dependencies are installed in a <strong>node_modules</strong> directory inside the project's working directory.

The <code>--save-dev</code> flag updates <strong>package.json</strong> with the new package. 

The first plugin that you want to install is gulp itself. Do this by running the following command from the command line from within your project's working directory:

    npm install gulp --save-dev

Gulp and its dependencies are then present in the the <strong>node_modules</strong> directory (inside the project directory). The <strong>package.json</strong> file is also updated to the following (your values may vary):

#### package.json

```
{
  "name": "test",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "gulp": "^3.9.1"
  }
}
```

Note that there is now a <code>devDependencies</code> field with gulp and its current version listed.

#### Gulpfile

Once packages are installed (in <strong>node_modules</strong>), you are ready to use them. All gulp code is written in a <strong>gulpfile.js</strong> file. To use a package, start by including it in <strong>gulpfile.js</strong>. The following code in your gulpfile includes the gulp package that was installed in the previous section:

#### gulpfile.js

```
var gulp = require('gulp');
```

<a id="tasks" />


## Creating tasks




Gulp tasks are defined in the <strong>gulpfile.js</strong> file using <a href="https://github.com/gulpjs/gulp/blob/master/docs/API.md#gulptaskname--deps--fn"><code>gulp.task</code></a>. A simple task looks like this: 

#### gulpfile.js

```
gulp.task('hello', function() {
  console.log('Hello, World!');
});
```

This code defines a <code>hello</code> task that can be executed by running the following from the command line:

    gulp hello

A common pattern for gulp tasks is the following:

1. Read some source files using <a href="https://github.com/gulpjs/gulp/blob/master/docs/API.md#gulpsrcglobs-options"><code>gulp.src</code></a>
2. Process these files with one or more functions using Node's <code>pipe</code> functionality
3. Write the modified files to a destination directory (creating the directory if doesn't exist) with <a href="https://github.com/gulpjs/gulp/blob/master/docs/API.md#gulpdestpath-options"><code>gulp.dest</code></a>

        gulp.task('task-name', function() {
          gulp.src('source-files') // 1
          .pipe(gulpPluginFunction()) // 2
          .pipe(gulp.dest('destination')); // 3
        });
        


A complete gulpfile might look like this:

#### gulpfile.js

```
// Include plugins
var gulp = require('gulp'); // Required
var pluginA = require('pluginA');
var pluginB = require('pluginB');
var pluginC = require('pluginC');

// Define tasks
gulp.task('task-A', function() {
  gulp.src('some-source-files')
  .pipe(pluginA())
  .pipe(gulp.dest('some-destination'));
});

gulp.task('task-BC', function() {
  gulp.src('other-source-files')
  .pipe(pluginB())
  .pipe(pluginC())
  .pipe(gulp.dest('some-other-destination'));
});
```

Where each installed plugin is included with <code>require()</code> and tasks are then defined using functions from the installed plugins. Note that functionality from multiple plugins can exist in a single task.

<a id="examples" />


## Examples




Let's look at some examples. 

#### Uglify JavaScript

Uglifying (or minifying) JavaScript is a common developer chore. The following steps set up a gulp task to do this for you (assuming Node, npm, and the gulp command line tool are installed):

1. Create a new project & <strong>package.json</strong> by running the following in the command line (from the project's working directory):

        npm init
        


2. Install the gulp package by running the following in the command line:

        npm install gulp --save-dev
        


3. Install the <a href="https://www.npmjs.com/package/gulp-uglify">gulp-uglify</a> package by running the following in the command line:

        npm install gulp-uglify --save-dev
        


4. Create a <strong>gulpfile.js</strong> and add the following code to include gulp and gulp-uglify:

        var gulp = require('gulp');
        var uglify = require('gulp-uglify');
        


5. Define the <code>uglify</code> task by adding the following code to <strong>gulpfile.js</strong>:

        gulp.task('uglify', function() {
          gulp.src('js/**/*.js')
          .pipe(uglify())
          .pipe(gulp.dest('build'));
        });
        


6. Run the task from the command line with the following:

        gulp uglify
        


The task reads all JavaScript files in the <strong>js</strong> directory (relative to the <strong>gulpfile.js</strong> file), executes the <code>uglify</code> function on them (uglifying/minifying the code), and then puts them in a <strong>build</strong> directory (creating it if it doesn't exist).

#### Prefix CSS & build sourcemaps

Multiple plugins can be used in a single task. The following steps set up a gulp task to prefix CSS files and create <a href="http://www.html5rocks.com/en/tutorials/developertools/sourcemaps/">sourcemaps</a> for them (assuming Node, npm, and the gulp command line tool are installed):

1. As in the previous example, create a new project and install gulp, <a href="https://www.npmjs.com/package/gulp-autoprefixer">gulp-autoprefixer</a>, and <a href="https://www.npmjs.com/package/gulp-sourcemaps">gulp-sourcemaps</a> by running the following in the command line (from the project's working directory):

        npm init
        npm install gulp --save-dev
        npm install gulp-autoprefixer --save-dev
        npm install gulp-sourcemaps --save-dev
        


2. Include the installed plugins by adding the following code to a <strong>gulpfile.js</strong> file:

        var gulp = require('gulp');
        var autoprefixer = require('gulp-autoprefixer');
        var sourcemaps = require('gulp-sourcemaps');
        


3. Create a task that prefixes CSS files, creates sourcemaps on the files, and writes the new files to the <strong>build</strong> directory by adding the following code to <strong>gulpfile.js</strong>:

        gulp.task('processCSS', function() {
          gulp.src('styles/**/*.css')
          .pipe(sourcemaps.init())
          .pipe(autoprefixer())
          .pipe(sourcemaps.write())
          .pipe(gulp.dest('build'));
        });
        


This task uses two plugins in the same task.

<a id="automation" />


## More automation




### Default tasks

Usually, developers want to run multiple tasks each time an application is updated rather than running each task individually. Default tasks are helpful for this, executing anytime the <code>grunt</code> command is run from the command line. 

Let's add the following code to <strong>gulpfile.js</strong> to set <code>task1</code> and <code>task2</code> as default tasks:

#### gulpfile.js

```
gulp.task('default', ['task1', 'task2']);
```

Running <code>gulp</code> in the command line executes both <code>task1</code> and <code>task2</code>.

### Gulp.watch

Even with default tasks, running tasks each time a file is updated during development can become tedious. <a href="https://github.com/gulpjs/gulp/blob/master/docs/API.md#gulpwatchglob--opts-tasks-or-gulpwatchglob--opts-cb"><code>gulp.watch</code></a> watches files and automatically runs tasks when the corresponding files change. For example, the following code in <strong>gulpfile.js</strong> watches CSS files and executes the <code>processCSS</code> task any time the files are updated:

#### gulpfile.js

```
gulp.task('watch', function() {
  gulp.watch('styles/**/*.css', ['processCSS']);
});
```

Running the following in the command line starts the watch:

    gulp watch

<div class="note">
<strong>Note:</strong> The watch task continues to execute once initiated. To stop the task, use <strong>Ctrl + C</strong> in the command line or close the command line window.
</div>

<a id="review" />


## Review




Using a build tool for the first time can be daunting with multiple tools to install and new files to create. Let's review what we've covered and how it all fits together.

Because gulp and its plugins are node packages, gulp requires <a href="https://nodejs.org/en/">Node</a> and its package manager, <a href="https://www.npmjs.com/">npm</a>. They are global tools so you only need to install them once on your machine, not each time you create a project. In this text, we used <a href="https://github.com/creationix/nvm">Node Version Manager</a> (nvm) to install Node and npm, but they could also have been installed directly. 

Gulp runs from the command line, so it requires a command line tool to be installed. Like Node, it's a global tool, and only needs to be installed on your machine (not per project). 

When you want to use gulp in a project, you start by initializing the project with <code>npm init</code>. This creates a file called <strong>package.json</strong>. The <strong>package.json</strong> file tracks the Node packages that are installed for that project. Each time a new package is installed, such as the gulp-uglify plugin, <strong>package.json</strong> is updated with the <code>--save-dev</code> flag. If the project is stored in version control or transferred without including all of the packages (a best practice), the packages can be quickly re-installed with <code>npm install</code>. This reads <strong>package.json</strong> and installs all required packages. 

Once plugins are installed, they need to be included in the <strong>gulpfile.js</strong> file. This file is where all gulp code belongs. This file is also where gulp tasks are defined. Gulp tasks use JavaScript code and the imported functions from plugins to perform various tasks on files. 

With everything installed and tasks defined, gulp tasks can be run by executing command line commands (such as <code>gulp uglify</code>).

<a id="resources" />


## Further reading




* <a href="https://github.com/gulpjs/gulp/blob/master/docs/getting-started.md">Gulp's Getting Started guide</a>
* <a href="https://github.com/gulpjs/gulp/blob/master/docs/recipes/README.md">List of gulp Recipes</a>
* <a href="http://gulpjs.com/plugins/">Gulp Plugin Registry</a>


