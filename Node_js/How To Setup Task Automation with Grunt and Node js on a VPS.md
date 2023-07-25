# How To Setup Task Automation with Grunt and Node js on a VPS

```Ubuntu``` ```Node.js```

## Introduction


So you want to develop your own website which is modern, easy to use, attractive to users, and works well across all browsers and platforms? Well, this usually requires a lot of work based on the complexity of your website. But there is an amazing tool which helps you immensely with the development and work-flow of your application: a task-based build tool called Grunt.


With Grunt, you can do almost anything. You can configure it to do specific tasks like concatenating and minifying while watching for changes in your source code.


Confused? Let me explain to you how the above mentioned tasks impact your site performance and why you should use them.


## Concatenating


Concatenating source files reduces the number of requests a browser has to make by combining several JavaScript (or CSS) files into one single file or to group them based on your needs. This is effective because no matter how small your files are, every single request your browser makes, takes at least a few milliseconds. This might not sound like very much, but most websites are made up of several scripts, stylesheets, images and other assets - which adds up to a substantial impact on the performance of your site. If we imagine that a browser takes 50 ms for one single request, then 20 source files would slow down the page load time by one whole second. Trust me, nobody likes a slow website.


## Minifying


Don't think that concatenating gives you the most effective results. Developers tend to waste a lot of space by commenting, organizing, and structuring source code to make it easier to read and debuggable for others or themselves. This is not a bad thing per se, in fact, developers are encouraged to do so, but your browser doesn't care how beautifully structured your code is, it will execute it no matter how it looks like. Minifying is the process of removing all unneeded characters from your source files without breaking or changing its functionality. Comments, white-space, and new-line characters take up unnecessary space and are not required for it to execute. Various other small optimizations often take place, like shortening variable names in JavaScript files and compressing certain statements. This often vastly reduces the amount of data that needs to be transmitted to a browser, leading to a further reduction of page load time.


## Run Tasks Automatically with Grunt


Grunt is simple and it makes almost everything automated. You surely don't want to run your concatenating and minifying processes manually by hand every time you do a tiny change in your source code. This is where Grunt comes in, it not only makes your job faster and easier, but more enjoyable too. It takes once to set up and zero effort after that.


Another benefit of Grunt is its ecosystem, which leads to its growth and the development of many new useful plugins. So why not create your own Grunt-plugin with a custom task you use often? Some of the most common tasks have already been turned into Grunt-plugins, therefore taking a look at the plugin list, choosing the one you need and configuring it is going to be enough to set up your task automation.


## Requirements


This tutorial assumes that you're already familiar with node.js, npm, basic Linux administrative tasks like connecting to your VPS using SSH but most importantly, that you already have a basic website project using a web application framework like Express.


If you're unfamiliar with node.js or if you haven't installed it yet, please refer to the node.js section on the Articles & Tutorials page above to find installation instructions for your operating system.


We also have many great articles on our Help & Community page about Linux Basics.


# Installation and Configuration


Let's get started by installing Grunt's command line interface (CLI)
globally by issuing the following command:


npm install -g grunt-cli


You might have to use sudo npm install -g grunt-cli (if
you are on Linux, OSX, BSD, etc.) or run your command as Administrator
on a Windows-based system.


Now we have to change into the directory of your web-project using the cd /path/to/your/project/ command.


Install Grunt and our required Grunt-plugin dependencies in your project directory using this command:


```
npm install grunt grunt-contrib-concat grunt-contrib-uglify grunt-contrib-cssmin grunt-contrib-watch --save-dev
```


Create a new Grunt configuration file called Gruntfile.js at the
root level of your project and add the following sample configuration:


```
module.exports = function(grunt) {

  grunt.initConfig({
    jsDir: 'public/javascripts/',
    jsDistDir: 'dist/javascripts/',    
    cssDir: 'public/stylesheets/',
    cssDistDir: 'dist/stylesheets/',
    pkg: grunt.file.readJSON('package.json'),
    concat: {
      js: {
        options: {
          separator: ';'
        },
        src: ['<%=jsDir%>*.js'],
        dest: '<%=jsDistDir%><%= pkg.name %>.js'
      },
      css: {
        src: ['<%=cssDir%>*.css'],
        dest: '<%=cssDistDir%><%= pkg.name %>.css'
      }
    },
    uglify: {
      options: {
        banner: '/*! <%= pkg.name %> <%=grunt.template.today("dd-mm-yyyy") %> */\n'
      },
      dist: {
        files: {
          '<%=jsDistDir%><%= pkg.name %>.min.js': ['<%= concat.js.dest %>']
        }
      }
    },
    cssmin: {
      add_banner: {
        options: {
          banner: '/*! <%= pkg.name %> <%=grunt.template.today("dd-mm-yyyy") %> */\n'
        },
        files: {
          '<%=cssDistDir%><%= pkg.name %>.min.css': ['<%= concat.css.dest %>']
        }
      }
    },
    watch: {
    files: ['<%=jsDir%>*.js', '<%=cssDir%>*.css'],
    tasks: ['concat', 'uglify', 'cssmin']
    }
  });

  grunt.loadNpmTasks('grunt-contrib-concat');
  grunt.loadNpmTasks('grunt-contrib-uglify');
  grunt.loadNpmTasks('grunt-contrib-cssmin');
  grunt.loadNpmTasks('grunt-contrib-watch');

  grunt.registerTask('default', [
    'concat',
    'uglify',
    'cssmin',
    'watch'
  ]);
  
};

```


Edit the four variables "jsDir", "jsDistDir", "cssDir", and "cssDistDir" according to your specific needs. These variables define the source directories of your Javascript/CSS files and the ready-to-distribute ("dist") directories. The "dist" directories will contain your concatenated and minified source code.


Execute our Grunt task by simply issuing grunt in your terminal and it should display an output similar to the following:


```
Running "concat:js" (concat) task
File "dist/js/application-name.js" created.

Running "concat:css" (concat) task
File "dist/css/application-name.css" created.

Running "uglify:dist" (uglify) task
File "dist/js/application-name.min.js" created.

Running "cssmin:add_banner" (cssmin) task
File dist/css/application-name.min.css created.

Running "watch" task
Waiting...

```


Grunt will now watch for any changes in your source code and concatenate/minify your source code whenever you modify it.


For more information about Grunt, please visit their official site and documentation at http://gruntjs.com.


