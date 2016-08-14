---
layout: post
title: Bootstrapping a new web app
comments: true
---

Several week ago one of my friends asked whether I could help in bootstrapping a new start-up. It seemed like a completely crazy idea. I rolled up my sleeves and was about to jump into coding ... and then I realised that there is no version control, infra to build on it. Just to make my life easier (any maybe someone else) this is my list of things to do when you're starting a new system completely from scratch.

1. Setup a [gitlab](https://gitlab.com): this is an absolute must-have. Unlimited private repos, unlimited code space. For free!
2. Setup a free [slack](https://slac.com) account - communicating in style is important ;)
3. Bootstrap the web-app. [yeoman](http://yeoman.io/) is a great help! To setup a complete, working web app with OAuth2 login (integrated with Facebook, Google+, Github etc) I chose the Angular full-stack generator: [generator-angular-fullstack](https://github.com/angular-fullstack/generator-angular-fullstack). In general there are many others that offer similar functionality (like [MEAN](http://meanjs.org/generator.html)) but ultimately this one turned out to be the easiest to tune and configure. It includes BrowserSync for automatic reload of code, a template of an Angular (v1.x) web app, a template of an express.js app. Everything is wrapped up into a nice npm & gulp build and gets you running really quickly.
4. Wrap the different parts of the app into docker containers. I have chosen to use a single container for front-end static files (just plain nginx image) and one for the back-end express service. There should be probably another one for a DB - but this needs only to be added when you actually need one (and not use stuff like Amazon Dynamo, etc.).
4. AWS EC2 account - an absolute must have... Or at least any kind of cloud infrastructure provider which can be reliably used to host all docker containers and nicely integrates into the docker toolkit.
