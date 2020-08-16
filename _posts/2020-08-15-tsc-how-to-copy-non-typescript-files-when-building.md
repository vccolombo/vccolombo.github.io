---
title: "tsc: How to copy non-TypeScript files when building"
excerpt: "tsc does not provide a way to copy non-TypeScript files during build. Here is how to set up a script to do this in a Node.js project."
date: 2020-08-16T10:35:30-03:00
categories:
  - Blog
tags:
  - TypeScript
  - tsc
---

When you run `tsc`, it transpile all your TypeScript code to JavaScript and put them inside a target folder (normally called `lib` or `dist`). But what about the other files? All the JS, HTML and CSS files (that were written for the client browser, for example) are ignored by tsc. Now you have a lot of transpiled JS, but no static files that are essential for the application to work.

## Does tsc have an option to copy non-TypeScript files during build?

**The short answer is no**. `tsc` only job is to convert TS code to JS, and there is no functionality to copy other files to the build directory. According to [this post](https://github.com/Microsoft/TypeScript/issues/30835), this is intended and is planned to continue this way.

## How can I copy other files during the build?

The current situation is not good. If you run `tsc` you might miss non-TS files and end up with a broken application. So what is the solution?

### `"allowJs": true` in tsconfig.json:

To "copy" JS files during the build, you can just add the `"allowJs": true` in tsconfig.json. It would actually "transpile" your JS files into JS that follows the rules you set in the configurations, which is nice. However, that does not fix the issue with other files like HTML and CSS.

### copyfiles

The best option right now is to set up a build pipeline in package.json. The idea is to set a script that runs `tsc` and copies all missing files to the target folder.

An example would be:

```json
"build": "yarn clean && tsc && yarn copy-files",
"clean": "rm -rf ./dist",
"copy-files": "cp ./src/public/ ./dist/"
```

This would copy the public folder into the build directory.

This solution is ok but is not OS-independent. So a better option would be to use some libraries that are specific for removing and copying files/directories.

The first one is [rimraf](https://www.npmjs.com/package/rimraf), which is described as _"The UNIX command `rm -rf` for node"_.

The second is [copyfiles](https://www.npmjs.com/package/copyfiles), with a very descriptive name.

First, install the packages as dev dependencies:

```terminal
$ npm install --save-dev rimraf copyfiles
# or yarn
$ yarn add -D rimraf copyfiles
```

And finally, put the script in package.json:

```json
// ...

"scripts": {
    "clean": "rimraf dist/",
    "copy-files": "copyfiles -u 1 src/**/*.html src/**/*.css dist/",
    "build": "yarn clean && tsc && yarn copy-files"
},

// ...
```

Easy as that. Now there is no need to keep track of where each of the non-TS files are, as every HTML and CSS files in `src` will be copied to the target directory. If you need any other file type, just add it to the `copy-files` script (e.g. `src/**/*.txt`).

### Migrate to Babel

Another option would be migrating to Babel. I have never used Babel, so I will abstain from talking about it in this post. However, it is very popular and may even reduce build times.

## Conclusion

`tsc` has no built-in function to copy non-TypeScript files to the build directory. So in this post, we learned about simple ways to copy the files by ourselves in an OS-independent way.
