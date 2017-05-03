# Make a beautiful class-based gulpfiles with Typescript and Gulpclass

Allows to create a gulp files in classes, each method of which can be a gulp task.

## Installation

1. Install module:

    `npm install gulpclass --save-dev`

2. Use [typings](https://github.com/typings/typings) to install all required definition dependencies.

    `typings install`

## Usage

1. Create a `gulpfile.ts` and describe your tasks
    
    ```typescript
    import {Gulpclass, Task, SequenceTask} from "gulpclass";
    import * as gulp from "gulp";
    import * as del from "del";

    @Gulpclass()
    export class Gulpfile {
    
        @Task() // return promise to indicate your task completion
        clean() {
            // del module returns promise for you automatically
            return del("./dist/**");
        }
        
        @Task() // or use provided callback instead
        clean(cb: Function) {
            return del("./dist/**", cb);
        }
    
        @Task()
        copyFiles() { // don't forget to return stream from your task function
            return gulp.src(["./README.md"])
                .pipe(gulp.dest("./dist"));
        }
    
        @Task("copy-source-files") // you can specify custom task name if you need
        copySourceFiles() {
            return gulp.src(["./src/**.js"])
                .pipe(gulp.dest("./dist/src"));
        }
    
        @SequenceTask() // this special annotation using "run-sequence" module to run returned tasks in sequence
        build() {
            return ["copyFiles", "copy-source-files"];
        }
    
        // list of dependencies could be specified as a second argument to trigger other tasks
        // the example below is equivalent to `gulp.task("default", ["build"]);`
        @Task("default", ["build"])
        defaultTask() {
            // using "defaultTask", because "default" is a reserved keyword in ES2015
        }
    
    }
    ```
    
2. How to run

    ## First option. Use ts-node
    
    Install [ts-node](https://github.com/TypeStrong/ts-node), it supports running gulp tasks directly using gulp command
    
    
    ## Second option. If you don't want to use ts-node

    There is a caveat of using gulp and typescript together. The problem is that when you run your `gulp` commands 
    in console, gulp cannot read your tasks from your typescript code - it can read only from `gulpfile.js`. 
    But there is a simple workaround - you can create a gulpfile.js, compile and execute typescript on-the-fly.
    
    Create a **gulpfile.js** and put there this piece of code:
    ```javascript
    eval(require("typescript").transpile(require("fs").readFileSync("./gulpfile.ts").toString()));
    ```
    This piece of code reads your gulpfile.ts contents, and asks typescript to transpile it on-the-fly and executes transpiled result as javascript.
    
    (you need to run `npm install typescript --save-dev` if you dont have typescript package installed)
    
    Please note that if you are NOT using `outDir` in typescript compile options, you may have problems if your 
    gulpclass file is named `gulpfile.ts` typescript compiler will try to compile it to `gulpfile.js`, and will override
    code you added to gulpfile.js. Solution is simple - rename your `gulpfile.ts`. You can call it as you wish, 
    for example you can call it `gulpclass.ts`.
    
    ## Alternative approaches

    Alternative approaches depend on which tsconfig configuration you use. These examples assume that you are using 
    `"outDir": "build"` as a directory to where files are compiled:

    * create `gulpfile.js` and put there ```require("build/gulpfile")```
    * or run gulp in cmd with extra parameters: `gulp --gulpfile build/gulpfile.js --cwd .`
    
    Benefits of this apporach is that you can use debug your gulp classes in IDEs.

## FAQ

* How to load tasks from multiple files?

Edit your `gulpfile.js` and change it following way:

```javascript
const files = [
    "./othertask.ts",
    "./gulpfile.ts"
];
files.forEach(function(file) { eval(require("typescript").transpile(require("fs").readFileSync(file).toString())) });
```

* How to use task dependencies?

Simple use second argument of the `@Task` decorator:

```typescript
@Task("someTask", ["clean", "compile"])
someTask() {
    return doSomething();
}
```

* Why my task `gulp.task("default", ["clean", "compile", "build"])` is not working?

I have such a task and its not working:

```typescript
@Task("default")
default() {
    return ["clean", "compile", "build"];
}
```

Why? Because the array you are returning is what task is doing, not a task
dependencies as you wish:

```typescript
@Task("default", ["clean", "compile", "build"])
default() {
}
```


## Samples

This project itself using [gulpfile.ts](https://github.com/pleerock/gulpclass/blob/master/gulpfile.ts).
Take a look on it as an example.
