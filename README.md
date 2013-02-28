# grunt-cachebuster

This grunt task iterates over its source files, calculating the MD5 hash of each, then creates a file containing the
list of filenames and hashes. This file can then be used in your project to generate filenames that contain the MD5
hash of the file's contents, e.g. main-ae65552d65cd19ab4f1996c77915ed42.js, so that even if a sticky cache is used,
clients will always load the latest version of files whenever they change.

## Getting Started
This plugin requires Grunt `~0.4.0`

If you haven't used [Grunt](http://gruntjs.com/) before, be sure to check out the [Getting Started](http://gruntjs.com/getting-started) guide, as it explains how to create a [Gruntfile](http://gruntjs.com/sample-gruntfile) as well as install and use Grunt plugins. Once you're familiar with that process, you may install this plugin with this command:

```shell
npm install grunt-cachebuster --save-dev
```

One the plugin has been installed, it may be enabled inside your Gruntfile with this line of JavaScript:

```js
grunt.loadNpmTasks('grunt-cachebuster');
```

## The "cachebuster" task

### Overview
In your project's Gruntfile, add a section named `cachebuster` to the data object passed into `grunt.initConfig()`.

```js
grunt.initConfig({
  cachebuster: {
    options: {
      // Task-specific options go here.
    },
    your_target: {
      // Target-specific file lists and/or options go here.
    },
  },
})
```

### Options

#### options.format
Type: `String`
Default value: `'json'`
Supported values: `'json'`, `'php'`

Specifies in which format the destination file will be generated.

#### options.formatter
Type: `Function`
Default value: none

If `options.formatter` is specified, then `options.format` will be ignored and the specified function will be called
instead to generate the contents of the destination file.

The function will be passed two arguments, `hashes` and `banner`, and is expected to return a string. The returned
string will be written unmodified to the destination file.

Function arguments:

  * `hashes`: an object containing the MD5 hashes of all specified source files keyed by filename.
  * `banner`: the banner string to be prepended to the output, or an empty string if no banner was configured.

Example `hashes` parameter:

```js
    {
      "path/to/filename1" : "fa6a5a3224d7da66d9e0bdec25f62cf0",
      "path/to/filename2" : "5ba48b6e5a7c4d4930fda256f411e55b"}
    }
```

#### options.basedir
Type: `String`
Default value: none

If specified, source filenames will be converted to be relative to this path when they are written to the destination
file. For example, given the following configuration:

```js
grunt.initConfig({
    cachebuster: {
        build: {
            options: {
                banner: '<%= meta.custom_banner %>',
                format: 'json',
                basedir: 'src/assets/'
            },
            src: [ 'src/assets/filename1', 'src/assets/folder1/filename2' ],
            dest: 'target/cachebusters.json'
        }
    },
)
```

the resulting `target/cachebusters.json` would be:

```js
{"filename1":"fa6a5a3224d7da66d9e0bdec25f62cf0","folder1/filename2":"5ba48b6e5a7c4d4930fda256f411e55b"}
```

#### options.banner
Type: `String`
Default value: `''`

If specified, this text is inserted at the top of the generated file. Take care that the banner will be valid in the
chosen file format - e.g. if options.format is `'json'` then banner should be a javascript comment, and in that case
only if your json parser supports comments; if options.format is `'php'`, then the banner will be inserted *after* the
`<?php` line and the banner should be specified as a valid php code or comment(s).

You may use Grunt templates in the banner, for example:

```js
    banner: '/*! <%= pkg.name %> - v<%= pkg.version %> - ' +
            '<%= grunt.template.today("yyyy-mm-dd") %> */'
```

#### options.complete
Type: `Function`
Default value: none

If specified, this function will be called, passing the finished hashes object as its sole parameter, and its return
value will be processed through the configured formatter for writing to the destination file. This can be used to
augment the hashes in some way before the file is generated.

This may be useful if you're using the `'php'` output format, but you need your php array structure to be different
to the default. For example, this function can be used to nest the array inside a parent array for use as a Laravel
configuration file:

```js
grunt.initConfig({
    laravel-cachebuster-configuration: {
        options: {
            basedir: 'public/',
            format: 'php',
            banner:
                '/**\n' +
                ' * GENERATED FILE, DO NOT EDIT. This file is simply a collection of generated hashes for static assets in \n' +
                ' * the project. It is generated by grunt, see Gruntfile.js for details.\n' +
                ' */'
            complete: function(hashes) {
                return {
                    md5: hashes
                };
            }
        },
        src: ['public/**/*'],
        dest: 'application/config/cachebuster.php'
    },
})
```

The resulting `'application/config/cachebuster.php'` file will contain something like:

```php
<?php
/**
 * GENERATED FILE, DO NOT EDIT. This file is simply a collection of generated hashes for static assets in
 * the project. It is generated by grunt, see Gruntfile.js for details.
 */
return array(
	'md5' => array(
		'js/main.js' => 'ae65552d65cd19ab4f1996c77915ed42',
		'js/vendor/modernizr-2.6.2.min.js' => 'b8009fa783ea3de3802efcd29d7473d5',
		'img/bg/about.jpg' => '7e402c1d64f0b00b4ade850f9017556a',
		'crossdomain.xml' => '625e6c239ea0b5504ce0641b74ec2a3b',
	)
);
```


### Usage Examples

#### Default Options
In this example, the default options are used; so the files `src/file1` and `src/file2` will each be read, and their
MD5 hashes will be written into the file `dest/cachebusters.json` in the default format, which is JSON.

```js
grunt.initConfig({
    cachebuster: {
        options: {},
        files: {
            'dest/cachebusters.json': ['src/file1', 'src/file2'],
        },
    },
})
```


#### Custom Options
In this example, the use of a custom formatter allows writing the MD5 hashes of the source files to the destination
file in CSV format.

```js
grunt.initConfig({
    cachebuster: {
        options: {
            basedir: 'src/assets/',
            formatter: function(hashes) {
                var output = '"Filename","Hash"\n';
                for (var filename in hashes) {
                    output += '"' + filename + '","' + hashes[filename] + '"\n';
                }
                return output;
            }
        },
        src: 'src/assets/**/*',
        dest: 'dest/cachebusters.csv'
    }
})
```

## Contributing
In lieu of a formal styleguide, take care to maintain the existing coding style. Add unit tests for any new or changed functionality. Lint and test your code using [Grunt](http://gruntjs.com/).

## Release History
_(Nothing yet)_
