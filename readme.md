# Grunt

_Note: This is my personal reference about grunt. Most of codes are similar to official documentation._

[Grunt](https://gruntjs.com/) is a automation tool which has a simple workflow and structure. It help as make tasks which executes automatically (e.g. on file change), as a part of ci/cd pipeline or manually trough terminal command.

* [Installation](#installation)
* [Plugins usage](#plugins-usage)
* [Execute task on file change](#execute-task-on-file-change)
* [Asynchronous and Custom task](#asynchronous-and-custom-task)
* [Files allocation (src-dest)](#files-allocation)
* [Globing patterns](#globing-patterns)
* [Options](#options)
* [Arguments](#arguments)
* [External task load](#external-task-load)
* [Create plugin](#create-plugin)
* [Plugins](#plugins)

## Installation

Both Grunt CLI and Grunt module are required.

```
npm install grunt-cli -g
npm install grunt --save-dev
```

_Note: installation commands for plugins used in the examples are listed at the bottom of page._

## Plugins usage

Grunt provides simple and readable structure for plugin use.
In the following example command:

* `grunt` executes `default` task (`jshing` and `uglify`).
* `grunt jshint` executes only `jshint` task.

```js
module.exports = function(grunt) {
    // configure tasks
    grunt.initConfig({
        pkg: grunt.file.readJSON("package.json"), // JSON file 2 Object :)
        // task one
        jshint: {
            build: {
                src: "Gruntfile.js"
            }
        },
        //task two (uglifying Gruntfile.js just for presentation purpose)
        uglify: {
            build: {
                src: "Gruntfile.js",
                dest: "Gruntfile.min.js"
            }
        }
    });

    // load the plugin jshint, uglify
    grunt.loadNpmTasks("grunt-contrib-jshint");
    grunt.loadNpmTasks("grunt-contrib-uglify");

    // default task
    grunt.registerTask("default", ["jshint", "uglify"]);
};
```

## Execute task on file change

Remember. You must use watcher!
What happens here:

* Command `grunt` executes `buldTask` once,
* and starts `watch` task.
* on any further file change buldTask will be exacuted.

```js
/**
 * Grunt example
 * This example shows:
 * - how to use and configure watch plugin
 * - how to setup task for both initial and 'on file change' execution
 */
module.exports = function(grunt) {
    // configure tasks
    grunt.initConfig({
        watch: {
            files: ["src/**/*.js"],
            tasks: ["buldTask"]
        },
        jshint: {
            build: {
                src: "src/**/*.js"
            }
        },
        uglify: {
            build: {
                src: "src/**/*.js",
                dest: "dist/build.min.js"
            }
        }
    });

    // load the plugins
    grunt.loadNpmTasks("grunt-contrib-watch");
    grunt.loadNpmTasks("grunt-contrib-jshint");
    grunt.loadNpmTasks("grunt-contrib-uglify");

    // buildTask - executes on file change
    grunt.registerTask("buldTask", ["jshint", "uglify"]);

    // default - executes buldTask once, and starts watch task
    grunt.registerTask("default", ["buldTask", "watch"]);
};
```

## Asynchronous and Custom task

In this example I've created two custom and one asynchronous task.
They need to executes in this order:

* taskFirst
* taskAsync - asynchronous task
* taskLast

Command `this.async();` tells grunt I want to use async exacution.
Grunt will continue when `done()` function is called.
If task executes successfully pass true (otherwise false).

```js
module.exports = function(grunt) {
    grunt.registerTask("taskFirst", "First task", function() {
        grunt.log.writeln("Do somthing in first task");
    });

    grunt.registerTask("taskAsync", "First task", function() {
        var done = this.async(); // IMPORTANT for async

        setTimeout(() => {
            grunt.log.writeln(`Do somthing in ${grunt.task.current.name} task`); //ex. how to access current task obj
            grunt.log.writeln(`I'll randomy pass/fail`);

            done(Math.random() > 0.5); // true/false randomy fail for demonstration
        }, 1000);
    });

    grunt.registerTask("taskLast", "First task", function() {
        grunt.log.writeln("Do somthing in last task");
    });

    grunt.registerTask("default", ["taskFirst", "taskAsync", "taskLast"]);
};
```

## Files allocation

There are several ways to set src-dest files allocation.
Plugins like jshint may accept only src property.

```js
    // compact
    src: "src.js",
    dest: "dest.js"

    // compact (array of sources)
    src: ["srcOne.js", "srcTwo.js"],
    dest: "dest.js"

    //object - use pattern: dest:[src, src2, ...]
    files: {
        "destOne.js": ["srcOneA.js", "srcOneB.js"],
        "destTwo.js": ["srcTwoA.js", "srcTwoB.js"],
    }

    // array
    files: [
        {src: "src.js", dest: "dest.js"},
        {src: ["src/**/*"], dest: "destinationDir", filter: isFile},
    ]

    // except
    {src: ["*.js", "!skipThisFile.js"], dest: "dest.js"}

    // All files in alpha order, but with bar.js at the end.
    {src: ["foo/*.js", "!foo/bar.js", "foo/bar.js"], dest: ...}

    // templates
    {src: ["src/<%= filename %>.js"], dest: "dest/<%= filename %>.min.js"}

    //extend=true
     files: [
        {
          expand: true,     // Allow as to add extra config (cwd, ext, extDot, flaten, rename)
          cwd: "lib/",      // Basebath for src
          ext: ".min.js",   // Overwrite extension
          extDot: "first"   // Use "first" or "last" for to determine extension part
          src: ["**/*.js"],  
          dest: "build/",
        }, ...
      ]
```

## Globing patterns

From grunt documentation (extended with examples)

* `*` matches any number of characters, but not /  
  `src/*.js` matches all js files in src directory
* `?` matches a single character, but not /  
  `src/file-?.js` matches `file-C.js, file-5.js ...` in src directory
* `**` matches any number of characters, including /,  
  `src/**` match all inside src
  `src/**/*` match all inside src
  `src/**/*.js` match all js files inside src
* `{}` allows for a comma-separated list of "or" expressions  
  `{one,two}.js` matches `one.js, two.js`
* `!` removes files if pattern starts with !
  `["**/*","!skipThisFile.js"]` matches all files except `skipThisFile.js`

## Options

Options are part of each plugin. Each prugin expose set of options. Mostly, we use option to tune/change/adjust plugin or task behavior.

* Each task may have an `options`.
* Each target may have an `options`.
* Target-level `options` will override task-level `options`.

From grunt docs.

```js
grunt.initConfig({
    jshint: {
        options: {
            // task-level options
        },
        build: {
            options: {
                // this options overrids task-level options.
            }
        },
        dev: {
            // this target will use task-level options.
        }
    }
});
```

## Arguments

Arguments are probably the most useful for automated deploy (development, testing, staging, production)  
Execute different task target depen on argument

* `grunt deploy` and `grunt deploy:development` executes customDeploy:development
* `grunt deploy --target=testing` executes customDeploy:testing
* `grunt deploy --target=staging` executes customDeploy:staging
* `grunt deploy --target=production` executes customDeploy:production

```js
grunt.initConfig({
  customDeploy: {
    development: {...},
    testing: { ...},
    staging: {...},
    production: {...},
  }
});

var target = grunt.option("target") || "development"; // default is developmnet
grunt.registerTask("deploy", ["customDeploy:" + target]);
```

## External task load

Make Gruntfile.js lighweight. Put tasks in a extenals files.

Gruntfile.js

```js
module.exports = function(grunt) {
    //load tasks from grunt directory
    grunt.loadTasks("grunt");

    // default task
    grunt.registerTask("default", ["uglify", "jshint"]);
};
```

grunt/jshint.js

```js
module.exports = function(grunt) {
    // configure and compose tasks
    grunt.config("jshint", {
        build: {
            src: "Gruntfile.js"
        }
    });

    // load the plugin jshint, uglify
    grunt.loadNpmTasks("grunt-contrib-jshint");
};
```

grunt/uglify.js

```js
module.exports = function(grunt) {
    // configure and compose tasks
    grunt.config("uglify", {
        build: {
            src: "Gruntfile.js",
            dest: "Gruntfile.min.js"
        }
    });

    // load the plugin jshint, uglify
    grunt.loadNpmTasks("grunt-contrib-uglify");
};
```

## Create plugin

Create plugin boilerplate

```
npm install -g grunt-init

git clone git://github.com/gruntjs/grunt-init-gruntplugin.git ~/.grunt-init/gruntplugin  

# on windows:
git clone git://github.com/gruntjs/grunt-init-gruntplugin.git "%USERPROFILE%\.grunt-init\gruntplugin"

grunt-init gruntplugin

npm install

# to publish:
npm publish
```

## Plugins

<https://gruntjs.com/plugins>

Common pattern for grunt modules installation

```
npm install <module> --save-dev
```

Examples

```
npm install grunt-contrib-watch --save-dev
npm install grunt-contrib-jshint --save-dev
npm install grunt-jslint --save-dev
npm install grunt-eslint --save-dev
npm install grunt-contrib-uglify --save-dev
npm install grunt-contrib-sass --save-dev
npm install grunt-contrib-cssmin --save-dev
npm install grunt-concurrent --save-dev
npm install grunt-newer --save-dev
```
