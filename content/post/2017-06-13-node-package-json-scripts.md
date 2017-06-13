---
title: "Node.js - Package.json scripts"
description: "Mock @angular/cli commands on a simple node project"
tags:
  - node.js
date: "2017-06-13"
featured: false
slug: node-package-json-scripts
image: /images/nodecli.png
---

If you are developing an angular application, chances are you are using @angular/cli. It is a fantastic tools which comes packed with awesome time saving features. Now recently I had to work on a node.js backend project and I dearly missed @angular/cli, so this post is about a couple tricks I found to mock @angular/cli's features.

# ng test

Usually angular's unit test are along side the file they refer to, in node sadly most of the time it is not the case. You have a bit of set up to do configure jasmine to work in this way. Also with `ng test` karma implements a watchers that triggers the test every time a file changes, this is extremely useful in a TDD approach.

Goals:
- Have the test files side by side with the source files.
- Have a watcher to reload the test every time a file changes.

1- Install jasmine and nodemon

`npm install jasmine nodemon --save-dev`

We will use nodemon to watch for file changes and jasmine as development framework.

2- configure jasmine

spec/support/jasmine.json
``` json
{
  "spec_dir": "",
  "spec_files": [
    "services/**/*.spec.js",
    "controllers/**/*.spec.js"
  ],
  "stopSpecOnExpectationFailure": false,
  "random": false
}
```

3- Configure scripts in package.json

``` json
{
  "scripts": {
    "test": "./node_modules/nodemon/bin/nodemon.js --exec ./node_modules/jasmine/bin/jasmine.js",
  }
}
```

# ng test --code-coverage

Running the test and generating the coverage report with istanbul is another awesome feature of @angular/cli. To implement this we just need to install istanbul have set it up!

1- Install istanbul
`npm install istanbul --save-dev`

2- configure istanbul

``` yaml
# .istanbul.yml
instrumentation:
  include-all-sources: true
  verbose: true
  excludes: ["**/*.spec.js"]
reporting:
  dir: "coverage"
```

Two very important settings here that I struggled to find:
- `include-all-sources`: this makes sure all the files are included in the report, not just only the one loaded by the spec files.
- `excludes`: this removes the spec files from the report.

3- Configure scripts in package.json

``` json
{
  "scripts": {
    "coverage": "node_modules/istanbul/lib/cli.js cover node_modules/jasmine/bin/jasmine.js"
  }
}
```

Note: the only difference with @angular/cli is there will be a couple more files in the coverage folder, I didn't find out how to crack this, but I guess it works alright like this.

# ng lint
Linters are fabulous to keep high coding standard in your project. Lets see how to make them available through npm.

1- Install eslint

`npm install eslint --save-dev`

2- Tweak the configuration file

I advise you to take a look online at the documentation to find all the rules and configuration you can use with linters. There is no point me posting my linters in here as this settings are specific to each developer/teams.
Anyhow make sure you have an eslint configuration at the root folder

3- Configure scripts in package.json

``` json
{
  "scripts": {
    "lint": "node_modules/.bin/eslint **/*.js"
  }
}
```
