# heroes2

The official Angular2 Tour of Heroes for rc-6 using TypeScript.

The site will be available [on Heroku](https://myra-the-ferryboat.herokuapp.com/) as soon as the deployment errors are solved.

Table of Contents

1. [Development](#development)
2. [Current Work](#current-work)
2. [Tour of Heroes](#tour-of-heroes)
3. [The favicon.ico](#the-favicon.ico)
4. [Webpack](#webpack)
5. [Deploying to Heroku](#deploying-to-heroku)
5. [Setup](#setup)
5. [Original Angular 2 QuickStart Source](#original-angular-2-quickstart-source)
5. [Prerequisites](#prerequisites)
5. [Install npm packages](#Iinstall-npm-packages)
5. [npm scripts](#npm-scripts)
5. [Testing](#testing)
5. [Unit Tests](#unit-tests)
5. [End-to-end (E2E) Tests](#end-to-end)

## <a name="development">Development</a>
This is a quick reference of command used to develop the app.

* `$ npm start` - Start the lite server for development.
* `$ node server.js` - Start the NodeJS server for production environment and serving the API.
* `$ git push -u remote master` - Push to GitHub.
* `$ git push heroku master` - Push to Heroku.
* `$ npm test` - Unit tests. (test output will is saved in `./_test-output/tests.html`)
* `$ npm run e2e` - End to end tests. (generates a file at `./_test-output/protractor-results.txt`)

See the [npm scripts](#npm-scripts) for other commands.

## <a name="current-work">Current work</a>

Completed [part two](https://angular.io/docs/ts/latest/tutorial/toh-pt2.html) of the Angular2 Tour of Heros, titled Master/Detail.

Added NodeJS server to use for deployment on Heroku.  The app is now live!

Looking at fixing the tests before the refactoring during the [next part: component templates](https://angular.io/docs/ts/latest/tutorial/toh-pt3.html).
Currently, two out of three unit tests are failing.
```
Failed	should instantiate component	AppComponent with TCB 
Error: Template parse errors: Can't bind to 'ngModel' since it isn't a known property of 'input'. 
(" <div> <label>name: </label> <input [ERROR ->][(ngModel)]="selectedHero.name" placeholder="name"/> </div> </div> "): 
AppComponent@7:11
```
Some advice [from StackOverflow](http://stackoverflow.com/questions/39040011/cant-bind-to-ngmodel-since-it-isnt-a-known-property-of-input-despite-impor) on this:
`You need to create a root NgModule in which you import the FormsModule`
And example shows using this combo:
```
import { FORM_DIRECTIVES } from '@angular/forms';
...
, directives: [FORM_DIRECTIVES]
```
However, for us, the first one yeilds the following error:
```
[ts] Module '"/Users/tim/angular/ng2/heroes2/node_modules/@angular/forms/index"' has no exported member 'FORM_DIRECTIVES'.
import FORM_DIRECTIVES
```
The answer did mention that this is RC5, and we are using RC6 here.
The official docs point to using forms with two way binding with this import:
```
import { FormsModule }   from '@angular/forms';
```
That is what we already have in our app.modules.ts.
In RC6, provideForms, disableDeprecatedForms are removed as deprecated.
These were fixes for the problem mentioned.  So now is there no fix for testing forms with two way binding (pretty much the only place you would want it) in RC6?
Just about to get frustred when a search prefixed with RC6 brought up [this answer](http://stackoverflow.com/questions/35229960/cant-bind-to-for-since-it-isnt-a-known-native-property-angular2):
`Angular by default uses property binding but label doesn't have a property for. To tell Angular explicitly to use attribute binding, use instead.`

So using this works:
```html
<input [(attr.ngModel)]="selectedHero.name" placeholder="name"/>
```
Günter Zöchbauer had this comment explaining why:
'With attr.for you have to explicitly opt in to attribute binding because attribute binding is expensive. Attributes are reflected in the DOM and changes require for example to check if CSS selectors are registered that match with this attribute set. Property binding is JS only and cheap, therefore the default.'
I have some notes somewhere on property vs. attribute binding.  Not a simple subject.


The second error (yes, I said there were two!:
```
Failed	should have expected <h1> text	AppComponent with TCB 
Error: Template parse errors: Can't bind to 'ngModel' since it isn't a known property of 'input'. (" <div> <label>name: </label> <input [ERROR ->][(ngModel)]="selectedHero.name" placeholder="name"/> </div> </div> "): AppComponent@7:11
```
That is easy to fix.  Since we changed the title, expecting the new title passes the test.

Now for the end to end tests.
Running them yeilds this error:
```
npm ERR! Failed at the angular2-quickstart@1.0.0 e2e script 
'tsc && concurrently "http-server -s" "protractor protractor.config.js" 
--kill-others --success first'.
```
(kill-others --success first?  Really?!)
Anyhow, that is a similar error we when trying the Webpack introduction right at the end of the Developers Guide section of the Angular docs, and the Heroku deployment debacle.
Actually, that error comes AFTER this message:
```
[1]     failed - should display: My First Angular 2 App
[1]   Suite failed: QuickStart E2E Tests
```
So the test is failing for the same reason the last unit test was failing!
That error may be because of the clean up config we removed to make the app deploy to Heroku.
Change that and the test passes.


## <a name="tour-of-heroes-multiple-components">Tour of Heroes: Multiple Components</a>

After getting through part 3: Multiple Components, I got this error:
```
zone.js:484 Unhandled Promise rejection: Template parse errors:
Can't bind to 'hero' since it isn't a known property of 'my-hero-detail'.
1. If 'my-hero-detail' is an Angular component and it has 'hero' input, 
then verify that it is part of this module.
2. If 'my-hero-detail' is a Web Component then add "CUSTOM_ELEMENTS_SCHEMA" to the '@NgModule.schema' of this component to suppress this message.
 ("
    </li>
</ul>
<my-hero-detail [ERROR ->][hero]="selectedHero"></my-hero-detail>
"): AppComponent@19:16
```
This is definately case 1.
We had forgotten to inport the new detail view in the app.module.ts file:
```
import { HeroDetailComponent } from './hero-detail.component';
```
But this did not solve the error.  We had to also declare it in the declarations section:
```
declarations: [
    AppComponent,
    HeroDetailComponent
],
```



## <a name="tour-of-heroes-master-detail">Tour of Heroes: Master/Detail</a>

For part two, the master detail pattern, creating an array of heroes causes this error:
```
    const HEROES: Hero[] = [
        { id: 11, name: 'Mr. Nice' },
        ...
        { id: 19, name: 'Magma' },
        { id: 20, name: 'Tornado' }
    ];
```    
[ts]
A class member cannot have the 'const' keyword.

If 'const' is changed to 'var', then most of the red squiggly lines are gone, but there is a red squiggly line under the var keyword, with the mouse-over message:
[ts] 
Unexpected token. A constructor, method, accessor, or property was expected.

If the const keyword is removed altogether, there is no error.
The problem was the location where this was.
Originally, I put it in the AppComponent class.  But it should actually reside outside that class.
It could be in it's own file, but putting it after the Hero class after the import statements works for now.

On [StackOverflow, this answer said](http://stackoverflow.com/questions/36142879/const-keyword-in-typescript)
Why a class member cannot have the 'const' keyword in TypeScript?
const does not imply deep immutability so the following is valid:
```
const foo:any = {};
foo.bar = 123;  // Okay
```
In that sense readonly makes better sense for class members and that is supported :


Completed [toh-pt1](https://angular.io/docs/ts/latest/tutorial/toh-pt1.html) step.
Will jump ahead next and [add Webpack](https://angular.io/docs/ts/latest/guide/webpack.html) to replace SystemJS.


## <a name="the-favicon.ico">The favicon.ico</a>

The StackOverflow [#1 answer](http://stackoverflow.com/questions/2208933/how-do-i-force-a-favicon-refresh): 
force browsers to download a new version using the link tag and a querystring on your 
filename. This is especially helpful in production environments to make sure your users 
get the update.
```
<link rel="icon" href="http://www.yoursite.com/favicon.ico?v=2" />
```
We're using this variation:
```
<link rel="shortcut icon" href="favicon.ico?v=2" />
```

Answer #2:
he easy way to fix it is close to that of lineofbirds

1. type in www.yoursite.com/favicon.ico
2. push enter
3. ctrl+f5
4. Restart Browser (for IE and Firefox)

However, this is not working.  There is either no icon, or the old Angular icon.
When using Node, there is a Spring leaf icon.


## <a name="webpack">Webpack</a>
This is an alternative to the SystemJS approach used throughout the tutorial.
It has been chosen to replace SystemJS in the Angular CLI project, so getting used to it with Angular2 is the idea here.
We will add a new config file for it.
webpack.config.js
If you look at systemjs.config.js, you can see that everything needed is added there manually.
Webpack will inspects webpack.config.js and traverses files listed there for their import dependencies recursively.
It sees that we're importing @angular/core so it adds that to its dependency list for (potential) inclusion in the bundle. It opens @angular/core and follows its network of import statements until it has build the complete dependency graph from app.ts down.
Then it outputs these files to the app.js bundle file designated in configuration:
separate our volatile application app code from comparatively stable vendor code modules.

webpack.config.js (two entries)
```
entry: {
  app: 'src/app.ts',
  vendor: 'src/vendor.ts'
},
output: {
  filename: '[name].js'
}
```

We have an app.component.ts file, but no vendor.ts file.  That file would looks something like this:
```
// Angular 2
import '@angular/platform-browser';
import '@angular/platform-browser-dynamic';
import '@angular/core';
import '@angular/common';
import '@angular/http';
import '@angular/router';
// RxJS
import 'rxjs';
// Other vendors for example jQuery, Lodash or Bootstrap
// You can import js, ts, css, sass, ...
```

We have to tell Webpack to configure loaders for TypeScript and CSS and other non Javascript files.

Official Angular Webpack [introduction](https://angular.io/docs/ts/latest/guide/webpack.html)

These files  are the basics of the setup.
```
package.json 
typings.json 
tsconfig.json 
webpack.config.js 
karma.conf.js 
config/helpers.js
```
In the config directory, define separate configurations for development, production, and test environments in a separate file called webpack.common.js.
This is a JavaScript commonjs module file that begins with require statements
The standard polyfills are required to run Angular 2 applications in most modern browsers.
Load Zone.js early, immediately after the other ES6 and metadata shims.

Using the files from the end of the starter tutorial, this is some of what is output while running this command:
```
$ npm start
> angular2-webpack@1.0.0 start /Users/tim/angular/ng2/webpack-ng2
> webpack-dev-server --inline --progress --port 8080
 70% 3/3 build modules http://localhost:8080/
webpack result is served from http://localhost:8080/
content is served from /Users/tim/angular/ng2/webpack-ng2
404s will fallback to /index.html
 54% 6/8 build modulests-loader: Using typescript@1.8.10 and /Users/tim/angular/ng2/webpack-ng2/tsconfig.json
chunk    {0} app.js (app) 40 bytes {2} [rendered]
chunk    {1} polyfills.js (polyfills) 218 kB [rendered]
chunk    {2} vendor.js (vendor) 40 bytes {1} [rendered]
ERROR in /Users/tim/angular/ng2/webpack-ng2/node_modules/rxjs/operator/toPromise.d.ts
(7,59): error TS2304: Cannot find name 'Promise'.
...
ERROR in /Users/tim/angular/ng2/webpack-ng2/node_modules/@angular/platform-browser/src/dom/dom_adapter.d.ts
(97,42): error TS2304: Cannot find name 'Map'.
...
ERROR in /Users/tim/angular/ng2/webpack-ng2/node_modules/@angular/core/src/facade/collection.d.ts
(2,25): error TS2304: Cannot find name 'MapConstructor'.
...
ERROR in /Users/tim/angular/ng2/webpack-ng2/node_modules/@angular/core/src/facade/collection.d.ts
(103,25): error TS2304: Cannot find name 'Set'.
ERROR in /Users/tim/angular/ng2/webpack-ng2/node_modules/@angular/core/src/linker/system_js_ng_module_factory_loader.d.ts
(28,25): error TS2304: Cannot find name 'Promise'.
...
ERROR in multi polyfills
Module not found: Error: Cannot resolve 'file' or 'directory' ./src/polyfills.ts in /Users/tim/angular/ng2/webpack-ng2
 @ multi polyfills
ERROR in multi vendor
Module not found: Error: Cannot resolve 'file' or 'directory' ./src/vendor.ts in /Users/tim/angular/ng2/webpack-ng2
 @ multi vendor
Child html-webpack-plugin for "index.html":
    chunk    {0} index.html 20 bytes [rendered]
webpack: bundle is now VALID.
```

Going to localhost:8080 reveals a number of errors in the console:
```
polyfills.js:102 Uncaught Error: Cannot find module "./src/polyfills.ts"
vendor.js:6 Uncaught Error: Cannot find module "./src/vendor.ts"
client?cd17:46 [WDS] Errors while compiling.
client?cd17:48 /Users/tim/angular/ng2/webpack-ng2/node_modules/rxjs/operator/toPromise.d.ts
(7,59): error TS2304: Cannot find name 'Promise'.
...
client?cd17:48/Users/tim/angular/ng2/webpack-ng2/src/app/polyfills.ts
(9,3): error TS2304: Cannot find name 'require'.errors @ client?cd17:48
client?cd17:48multi polyfills
Module not found: Error: Cannot resolve 'file' or 'directory' ./src/polyfills.ts in /Users/tim/angular/ng2/webpack-ng2
resolve file
  /Users/tim/angular/ng2/webpack-ng2/src/polyfills.ts doesn't exist
  /Users/tim/angular/ng2/webpack-ng2/src/polyfills.ts.ts doesn't exist
  /Users/tim/angular/ng2/webpack-ng2/src/polyfills.ts.js doesn't exist
  /Users/tim/angular/ng2/webpack-ng2/src/polyfills.ts.css doesn't exist
  /Users/tim/angular/ng2/webpack-ng2/src/polyfills.ts.html doesn't exist
resolve directory
  /Users/tim/angular/ng2/webpack-ng2/src/polyfills.ts doesn't exist (directory default file)
  /Users/tim/angular/ng2/webpack-ng2/src/polyfills.ts/package.json doesn't exist (directory description file)
[/Users/tim/angular/ng2/webpack-ng2/src/polyfills.ts]
[/Users/tim/angular/ng2/webpack-ng2/src/polyfills.ts.ts]
[/Users/tim/angular/ng2/webpack-ng2/src/polyfills.ts.js]
[/Users/tim/angular/ng2/webpack-ng2/src/polyfills.ts.css]
[/Users/tim/angular/ng2/webpack-ng2/src/polyfills.ts.html]
 @ multi polyfillserrors @ client?cd17:48
client?cd17:48multi vendor
Module not found: Error: Cannot resolve 'file' or 'directory' ./src/vendor.ts in /Users/tim/angular/ng2/webpack-ng2
resolve file
  /Users/tim/angular/ng2/webpack-ng2/src/vendor.ts doesn't exist
  /Users/tim/angular/ng2/webpack-ng2/src/vendor.ts.js doesn't exist
  /Users/tim/angular/ng2/webpack-ng2/src/vendor.ts.ts doesn't exist
  /Users/t
```

Runnging git add . takes forever and this kind of message fills the console:
```
warning: CRLF will be replaced by LF in node_modules/karma/node_modules/expand-braces/node_modules/braces/node_modules/expand-range/node_modules/repeat-string/.npmignore.
The file will have its original line endings in your working directory.
```

Regarding this error:
```
error TS2304: Cannot find name 'Promise'.
```
Someone had a similar problem from beta 7.  

Make sure you have a reference to angular typings in your app's entry point.

See: https://github.com/ericmdantas/angular2-typescript-todo/blob/master/index.ts#L1

StackOverflow [says this](http://stackoverflow.com/questions/33332394/angular-2-typescript-cant-find-names):
    Core reason: the .d.ts file implicitly included by TypeScript varies with the compile target, so one needs to have more ambient declarations when targeting es5 even if things are actually present in the runtimes (e.g. chrome). More on lib.d.ts
There is no solution to the answer, only what is causing the problem.

This is an old issue that went away and came back again.
Here is the [Angular Github issue](https://github.com/angular/angular/issues/4902)
It involes this script, which is already in our package.json.
"scripts": {
    "postinstall": "typings install"
}
So lookging at another [SO Q/A(http://stackoverflow.com/questions/35660498/angular-2-cant-find-promise-map-set-and-iterator)]:
UPDATE: USING ANGULAR RC4 or RC5 WITH TYPESCRIPT 2.0.0
To get this to work with typescript 2.0.0, I did the following.
```
npm install --save-dev @types/core-js
```

This didn't help.  Giving up on Webpack for now until Angular provides an official demo that works.


## <a name="deploying-to-heroku">Deploying to Heroku</a>

For Heroku, we serve the app with NodeJS.  This will also provide an API to get and save data for the app later.
Heroku is easy to set up.  You create an account, download the toolbelt, add the remote and push to the server.
However, with Angular2, it was not so easy.  Had the following error when trying to deploy.
```
remote:        sh: 1: typings: not found
remote:        npm ERR! Linux 3.13.0-93-generic
remote:        npm ERR! argv "/tmp/build_248acdd4c019d67d7106e34460406ddd/.heroku/node/bin/node" "/tmp/build_248acdd4c019d67d7106e34460406ddd/.heroku/node/bin/npm" "install" "--unsafe-perm" "--userconfig" "/tmp/build_248acdd4c019d67d7106e34460406ddd/.npmrc"
remote:        npm ERR! node v5.11.1
remote:        npm ERR! npm  v3.8.6
...
remote: -----> Build failed
remote:        We're sorry this build is failing! You can troubleshoot common issues here:
remote:        https://devcenter.heroku.com/articles/troubleshooting-node-deploys
remote:        Some possible problems:
remote:        - Node version not specified in package.json
remote:        https://devcenter.heroku.com/articles/nodejs-support#specifying-a-node-js-version
remote:        Love,
remote:        Heroku
remote:  !     Push rejected, failed to compile Node.js app.
remote:  !     Push failed
remote: Verifying deploy...
remote: 
remote: !	Push rejected to myra-the-ferryboat.
remote: 
To https://git.heroku.com/myra-the-ferryboat.git
 ! [remote rejected] master -> master (pre-receive hook declined)
error: failed to push some refs to 'https://git.heroku.com/myra-the-ferryboat.git'
```

This is the same error the the angular2-webpack-starter had.

Some fixes call for removing the postinstall in the package.json file:
```
    "postinstall": "typings install",
```

This deploys without an error, but the page is broken with similar errors in the heroku logs:
```
2016-09-11T01:49:01.928261+00:00 app[web.1]: sh: 1: tsc: not found
2016-09-11T01:49:01.939402+00:00 app[web.1]: npm ERR! Linux 3.13.0-93-generic
2016-09-11T01:49:01.939703+00:00 app[web.1]: npm ERR! argv "/app/.heroku/node/bin/node" "/app/.heroku/node/bin/npm" "start"
2016-09-11T01:49:01.939883+00:00 app[web.1]: npm ERR! node v5.11.1
2016-09-11T01:49:01.940262+00:00 app[web.1]: npm ERR! npm  v3.8.6
2016-09-11T01:49:01.940488+00:00 app[web.1]: npm ERR! file sh
2016-09-11T01:49:01.940634+00:00 app[web.1]: npm ERR! code ELIFECYCLE
2016-09-11T01:49:01.940774+00:00 app[web.1]: npm ERR! errno ENOENT
2016-09-11T01:49:01.940917+00:00 app[web.1]: npm ERR! syscall spawn
2016-09-11T01:49:01.941187+00:00 app[web.1]: npm ERR! spawn ENOENT
2016-09-11T01:49:01.941040+00:00 app[web.1]: npm ERR! angular2-quickstart@1.0.0 start: `tsc && concurrently "tsc -w" "lite-server" `
2016-09-11T01:49:01.941425+00:00 app[web.1]: npm ERR! Failed at the angular2-quickstart@1.0.0 start script 'tsc && concurrently "tsc -w" "lite-server" '.
2016-09-11T01:49:01.941558+00:00 app[web.1]: npm ERR! Make sure you have the latest version of node.js and npm installed.
2016-09-11T01:49:01.941802+00:00 app[web.1]: npm ERR! If you do, this is most likely a problem with the angular2-quickstart package,
2016-09-11T01:49:01.941942+00:00 app[web.1]: npm ERR! not with npm itself.
2016-09-11T01:49:01.942376+00:00 app[web.1]: npm ERR! Tell the author that this fails on your system:
2016-09-11T01:49:01.942474+00:00 app[web.1]: npm ERR!     tsc && concurrently "tsc -w" "lite-server" 
2016-09-11T01:49:01.942687+00:00 app[web.1]: npm ERR!     npm bugs angular2-quickstart
2016-09-11T01:49:01.942584+00:00 app[web.1]: npm ERR! You can get information on how to open an issue for this project with:
2016-09-11T01:49:01.942790+00:00 app[web.1]: npm ERR! Or if that isn't available, you can get their info via:
2016-09-11T01:49:01.943010+00:00 app[web.1]: npm ERR! There is likely additional logging output above.
2016-09-11T01:49:01.942895+00:00 app[web.1]: npm ERR!     npm owner ls angular2-quickstart
2016-09-11T01:49:01.946377+00:00 app[web.1]: npm ERR! Please include the following file with any support request:
2016-09-11T01:49:01.946485+00:00 app[web.1]: npm ERR!     /app/npm-debug.log
2016-09-11T01:49:02.033017+00:00 heroku[web.1]: State changed from starting to crashed
2016-09-11T01:49:47.651741+00:00 heroku[router]: at=error code=H10 desc="App crashed" method=GET path="/" host=myra-the-ferryboat.herokuapp.com request_id=74c04f02-88b5-464b-a053-da73a395c221 fwd="115.69.35.53" dyno= connect= service= status=503 bytes=
```

Some advice from S.O.:Change the start field in package.json from
```
"start": "tsc && concurrently \"npm run tsc:w\" \"npm run lite\" "
```
to
```
"start": "concurrently \"npm run tsc:w\" \"npm run lite\" "
```

Worth a try, although our current start script is this:
```
"start": "tsc && concurrently \"tsc -w\" \"lite-server\" ",
```

However, the error in the logs is still the same:
```
npm ERR! Failed at the angular2-quickstart@1.0.0 start script 'concurrently "npm run tsc:w" "npm run lite" '.
```

Tried a basic version:
```
"start": "node server.js"
```
This is how we do it locally, however, the logs on Heroku say this:
```
2016-09-11T06:09:59.426976+00:00 app[web.1]: Error: Cannot find module '/app/server.js'
```

Why is it looking in the app directory?  This is how we have done it before.

And this:
```
node ../server.js
```
With the result:
```
Error: Cannot find module '/server.js'
```

And this: Run both commands in 2 separate cmds:
-in the first one run npm run tsc:w
-in the second one npm run lite
```
"start": "tsc && npm run tsc:w | npm run lite",
```

Gives the same error:
```
sh: 1: tsc: not found
```

Trying a Procfile:
```
web: node server.js
```
However, this, along with the original script causes the deploy error.
So removed the post install again:
```
"postinstall": "typings install",
```
Then the deploy works but the server crashes and we get the now familiar message in the logs:
```
Error: Cannot find module '/app/server.js'
```

Try renaming the file index.js.  This is the default apparently on Heroku.
Then, realizing that git status was ignoring the server.js file becuase there was actually this like in the .gitignore file:
```
*.js
```
Why was that there?  Anyhow, now there is a new error (thank godess for small miracles):
```
Error: Cannot find module 'express'
```
So move that from dev dependencies to just dependencies, and now the app appears to hang and then eventually crashes on Heroku.
The logs you might ask?
```
2016-09-11T06:50:21.177537+00:00 app[web.1]: Node app is running on port 8080
2016-09-11T06:50:36.125422+00:00 heroku[router]: at=error code=H20 desc="App boot timeout" method=GET path="/" host=myra-the-ferryboat.herokuapp.com request_id=d8eafe9a-f183-49da-a299-d6703f6edbb0 fwd="115.69.35.53" dyno= connect= service= status=503 bytes=
2016-09-11T06:51:19.752934+00:00 heroku[web.1]: Error R10 (Boot timeout) -> Web process failed to bind to $PORT within 60 seconds of launch
```
Apparently we should'nt be using port 8080.  So try 3000...
But same error:
```
2016-09-11T06:50:21.177537+00:00 app[web.1]: Node app is running on port 8080
2016-09-11T06:50:36.125422+00:00 heroku[router]: at=error code=H20 desc="App boot timeout" method=GET path="/" host=myra-the-ferryboat.herokuapp.com request_id=d8eafe9a-f183-49da-a299-d6703f6edbb0 fwd="115.69.35.53" dyno= connect= service= status=503 bytes=
2016-09-11T06:51:19.752934+00:00 heroku[web.1]: Error R10 (Boot timeout) -> Web process failed to bind to $PORT within 60 seconds of launch
2016-09-11T06:51:19.752993+00:00 heroku[web.1]: Stopping process with SIGKILL
```
Why is it trying port 8080 when we changed it to 3000?
Did the changes not get pushed?  Yer they did.
```
app.set('port', (process.env.PORT || 3000));
```
So the process environment port must be 8080.  Oh.  Doh!

```
app.set('port', (process.env.PORT || 3000));
app.listen(app.get('port'),function () {
	console.log('Node app is running on port 8080');
});
```
Wasn't using the port in the listen function!
So along with removing the postinstall in the package.json, creating the server.js file and configuring it correctly is the answer.


## <a name="setup">Setup</a>
Following the section beolow in the original Angular 2 Quickstart, with the following exception:
```
QuinquenniumF:heroes2 tim$ git remote add origin https://github.com/timofeysie/heroes2.git
fatal: remote origin already exists.
QuinquenniumF:heroes2 tim$ git remote add remote https://github.com/timofeysie/heroes2.git
```

Next, the typings folder didn't show up after npm install, so installed them manually with:
```
$ npm run typings -- install
```

Then, to push changes to GitHub:
```
$ git push -u remote master
```

To Deploy to Heroku
```
$ git push heroku master
```


## <a name="original-angular-2-quickstart-source">Original Angular 2 QuickStart Source</a>
[![Build Status][travis-badge]][travis-badge-url]

This repository holds the TypeScript source code of the [angular.io quickstart](https://angular.io/docs/ts/latest/quickstart.html),
the foundation for most of the documentation samples and potentially a good starting point for your application.

It's been extended with testing support so you can start writing tests immediately.

**This is not the perfect arrangement for your application. It is not designed for production.
It exists primarily to get you started quickly with learning and prototyping in Angular 2**

We are unlikely to accept suggestions about how to grow this QuickStart into something it is not.
Please keep that in mind before posting issues and PRs.

## <a name="prerequisites">Prerequisites</a>

Node.js and npm are essential to Angular 2 development. 
    
<a href="https://docs.npmjs.com/getting-started/installing-node" target="_blank" title="Installing Node.js and updating npm">
Get it now</a> if it's not already installed on your machine.
 
**Verify that you are running at least node `v4.x.x` and npm `3.x.x`**
by running `node -v` and `npm -v` in a terminal/console window.
Older versions produce errors.

We recommend [nvm](https://github.com/creationix/nvm) for managing multiple versions of node and npm.

## Create a new project based on the QuickStart

Clone this repo into new project folder (e.g., `my-proj`).
```bash
git clone  https://github.com/angular/quickstart  my-proj
cd my-proj
```

We have no intention of updating the source on `angular/quickstart`.
Discard everything "git-like" by deleting the `.git` folder.
```bash
rm -rf .git  # non-Windows
rd .git /S/Q # windows
```

## Install npm packages

> See npm and nvm version notes above

Install the npm packages described in the `package.json` and verify that it works:

**Attention Windows Developers:  You must run all of these commands in administrator mode**.

```bash
npm install
npm start
```

> If the `typings` folder doesn't show up after `npm install` please install them manually with:

> `npm run typings -- install`

The `npm start` command first compiles the application, 
then simultaneously re-compiles and runs the `lite-server`.
Both the compiler and the server watch for file changes.

Shut it down manually with Ctrl-C.

You're ready to write your application.

### <a name="npm-scripts">npm scripts</a>

We've captured many of the most useful commands in npm scripts defined in the `package.json`:

* `npm start` - runs the compiler and a server at the same time, both in "watch mode".
* `npm run tsc` - runs the TypeScript compiler once.
* `npm run tsc:w` - runs the TypeScript compiler in watch mode; the process keeps running, awaiting changes to TypeScript files and re-compiling when it sees them.
* `npm run lite` - runs the [lite-server](https://www.npmjs.com/package/lite-server), a light-weight, static file server, written and maintained by
[John Papa](https://github.com/johnpapa) and
[Christopher Martin](https://github.com/cgmartin)
with excellent support for Angular apps that use routing.
* `npm run typings` - runs the typings tool.
* `npm run postinstall` - called by *npm* automatically *after* it successfully completes package installation. This script installs the TypeScript definition files this app requires.
Here are the test related scripts:
* `npm test` - compiles, runs and watches the karma unit tests
* `npm run e2e` - run protractor e2e tests, written in JavaScript (*e2e-spec.js)

## Testing

The QuickStart documentation doesn't discuss testing.
This repo adds both karma/jasmine unit test and protractor end-to-end testing support.

These tools are configured for specific conventions described below.

*It is unwise and rarely possible to run the application, the unit tests, and the e2e tests at the same time.
We recommend that you shut down one before starting another.*

### Unit Tests
TypeScript unit-tests are usually in the `app` folder. Their filenames must end in `.spec`.

Look for the example `app/app.component.spec.ts`.
Add more `.spec.ts` files as you wish; we configured karma to find them.

Run it with `npm test`

That command first compiles the application, then simultaneously re-compiles and runs the karma test-runner.
Both the compiler and the karma watch for (different) file changes.

Shut it down manually with Ctrl-C.

Test-runner output appears in the terminal window.
We can update our app and our tests in real-time, keeping a weather eye on the console for broken tests.
Karma is occasionally confused and it is often necessary to shut down its browser or even shut the command down (Ctrl-C) and
restart it. No worries; it's pretty quick.

The `HTML-Reporter` is also wired in. That produces a prettier output; look for it in `~_test-output/tests.html`.

### End-to-end (E2E) Tests

E2E tests are in the `e2e` directory, side by side with the `app` folder.
Their filenames must end in `.e2e-spec.ts`.

Look for the example `e2e/app.e2e-spec.ts`.
Add more `.e2e-spec.js` files as you wish (although one usually suffices for small projects);
we configured protractor to find them.

Thereafter, run them with `npm run e2e`.

That command first compiles, then simultaneously starts the Http-Server at `localhost:8080`
and launches protractor.  

The pass/fail test results appear at the bottom of the terminal window.
A custom reporter (see `protractor.config.js`) generates a  `./_test-output/protractor-results.txt` file
which is easier to read; this file is excluded from source control.

Shut it down manually with Ctrl-C.

[travis-badge]: https://travis-ci.org/angular/quickstart.svg?branch=master
[travis-badge-url]: https://travis-ci.org/angular/quickstart
