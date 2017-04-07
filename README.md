# api documentation for  [jenkins-api (v0.2.8)](https://github.com/jansepar/node-jenkins-api)  [![npm package](https://img.shields.io/npm/v/npmdoc-jenkins-api.svg?style=flat-square)](https://www.npmjs.org/package/npmdoc-jenkins-api) [![travis-ci.org build-status](https://api.travis-ci.org/npmdoc/node-npmdoc-jenkins-api.svg)](https://travis-ci.org/npmdoc/node-npmdoc-jenkins-api)
#### Jenkins API written in Node.js

[![NPM](https://nodei.co/npm/jenkins-api.png?downloads=true)](https://www.npmjs.com/package/jenkins-api)

[![apidoc](https://npmdoc.github.io/node-npmdoc-jenkins-api/build/screenCapture.buildNpmdoc.browser._2Fhome_2Ftravis_2Fbuild_2Fnpmdoc_2Fnode-npmdoc-jenkins-api_2Ftmp_2Fbuild_2Fapidoc.html.png)](https://npmdoc.github.io/node-npmdoc-jenkins-api/build/apidoc.html)

![npmPackageListing](https://npmdoc.github.io/node-npmdoc-jenkins-api/build/screenCapture.npmPackageListing.svg)

![npmPackageDependencyTree](https://npmdoc.github.io/node-npmdoc-jenkins-api/build/screenCapture.npmPackageDependencyTree.svg)



# package.json

```json

{
    "author": {
        "name": "Shawn Jansepar",
        "email": "shawnjan@gmail.com"
    },
    "bugs": {
        "url": "https://github.com/jansepar/node-jenkins-api/issues"
    },
    "dependencies": {
        "request": ">= 2.2.9"
    },
    "description": "Jenkins API written in Node.js",
    "devDependencies": {},
    "directories": {},
    "dist": {
        "shasum": "174e154151d16a9a5fda2f4fa479f2b4bb2b020b",
        "tarball": "https://registry.npmjs.org/jenkins-api/-/jenkins-api-0.2.8.tgz"
    },
    "engine": {
        "node": ">=0.4"
    },
    "gitHead": "e6f5cf5069c6f2dbc131048cc0250324ac998188",
    "homepage": "https://github.com/jansepar/node-jenkins-api",
    "keywords": [
        "github",
        "jenkins"
    ],
    "main": "./lib/main.js",
    "maintainers": [
        {
            "name": "shawn",
            "email": "shawn@mobify.com"
        }
    ],
    "name": "jenkins-api",
    "optionalDependencies": {},
    "readme": "ERROR: No README data found!",
    "repository": {
        "type": "git",
        "url": "git+ssh://git@github.com/jansepar/node-jenkins-api.git"
    },
    "scripts": {},
    "version": "0.2.8"
}
```



# <a name="apidoc.tableOfContents"></a>[table of contents](#apidoc.tableOfContents)

#### [module jenkins-api](#apidoc.module.jenkins-api)
1.  [function <span class="apidocSignatureSpan">jenkins-api.</span>init (host, options)](#apidoc.element.jenkins-api.init)



# <a name="apidoc.module.jenkins-api"></a>[module jenkins-api](#apidoc.module.jenkins-api)

#### <a name="apidoc.element.jenkins-api.init"></a>[function <span class="apidocSignatureSpan">jenkins-api.</span>init (host, options)](#apidoc.element.jenkins-api.init)
- description and source-code
```javascript
init = function (host, options) {

    if (options) {
        request = request.defaults(options);
    }

    //Helper Functions
    var build_url = function(command) {
<span class="apidocCodeCommentSpan">        /*
        Builds REST url to Jenkins
        */
</span>
        // Create the url using the format function
        var url = util.format.call(this, command, host);

        // if command is the only arg, we are done
        if (arguments.length == 1) return url;

        // Grab the arguments except for the first (command)
        var args = Array.prototype.slice.call(arguments).slice(1);

        // Append url to front of args array
        args.unshift(url);

        // Create full url with all the arguments
        url = util.format.apply(this, args);

        return url;


    };
    return {
        build: function(jobname, params, callback) {
            var buildurl;
            /*
            Trigger Jenkins to build.
            */
            if (typeof params === 'function') {
                buildurl = build_url(BUILD, jobname);
                callback = params;
            } else {
                buildurl = build_url(BUILDWITHPARAMS+"?"+qs.stringify(params), jobname)
            }

            request({method: 'POST', url: buildurl }, function(error, response) {
                if ( error || (response.statusCode !== 201 && response.statusCode !== 302) ) {
                    callback(error, response);
                    return;
                }
                /*
                Return queue location of newly-created job as per
                  https://issues.jenkins-ci.org/browse/JENKINS-12827?focusedCommentId=201381#comment-201381
                */
                var data = {
                    message: "job is executed",
                    location: response.headers["Location"] || response.headers["location"]
                };
                callback(null, data);
            });
        },
        stop_build: function(jobname, buildNumber, callback) {
            var buildurl;
            buildurl = build_url(STOP_BUILD, jobname, buildNumber);

            request({method: 'POST', url: buildurl }, function(error, response) {
                if ( error || (response.statusCode !== 201 && response.statusCode !== 302) ) {
                    callback(error, response);
                    return;
                }
                var data = "job is stopped";
                callback(null, data);
            });
        },
        all_jobs: function(callback) {
            /*
            Return a list of object literals containing the name and color of all jobs on the Jenkins server
            */
            request({method: 'GET', url: build_url(LIST)}, function(error, response, body) {
                if ( error || response.statusCode !== 200 ) {
                    callback(error || true, response);
                    return;
                }
                var data = JSON.parse(body.toString()).jobs;
                callback(null, data);
            });
        },
        all_jobs_in_view: function(view, callback) {
            /*
            Return a list of objet literals containing the name and color of all the jobs for a view on the Jenkins server
             */
            request({method: 'GET', url: build_url(LIST_VIEW, view)}, function (error, response, body) {
                if ( error || response.statusCode !== 200 ) {
                    callback(error || true, response);
                    return;
                }
                var data = JSON.parse(body.toString()).jobs;
                callback(null, data);
            })
        },
        job_info: function(jobname, callback) {
            /*
            Get all information for a job
            */
            request({method: 'GET', url: build_url(JOBINFO, jobname)}, function(error, response, body) {
                if ( error || response.statusCode !== 200 ) {
                    callback(error || true, response);
                    return;
                }
                var data = JSON.parse(body.toString());
                callba ...
```
- example usage
```shell
...

### setup

'''javascript
var jenkinsapi = require('jenkins-api');

// no auth
var jenkins = jenkinsapi.init("http://jenkins.yoursite.com");

// username/password
var jenkins = jenkinsapi.init("http://username:password@jenkins.yoursite.com");

// API Token
var jenkins = jenkinsapi.init('https://username:token@jenkins.company.com');
...
```



# misc
- this document was created with [utility2](https://github.com/kaizhu256/node-utility2)
