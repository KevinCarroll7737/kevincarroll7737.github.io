---
layout: dev
categories: [dev]
---

> I've been ask by a sponsor to develop a slot machine mobile application. I wasn't sure if I should accept it since I never develop a whole project from scratch. So I made some researches on the upstream and found many solutions which I tought could be realist for me and accepted the deal.

This article will walk you trough this journey. Hopefuly, it will guide you where I found my self a lost. Let's be honest, I am NOT a senior full stack developer and the main reason why I accepted to make this project, is to learn. I encourage everyone to do so :-)

![](/assets/images/gohead.jpeg)

### It's when I started searching for a cross platform solution that I met [cordova](https://cordova.apache.org/). 

> Mobile apps with HTML, CSS & JS
>
> Target multiple platforms with one code base
>
> Free and open source

![logo](/assets/images/logo_cordova_phonegap.jpg)

This framework do NOT transpile your HTML base project into a native languague. It simply creates an application that is aloud to run in a `WebView`. Thus, these applications are called `hybrid` not `native`.

![chart](/assets/images/cordova_chart.jpg)

Essentially, Cordova Appache only runs a server localy which contains all the pages of your application and uses the platform's libraries to render it as HTML or to run native functionality.

![server](/assets/images/cordova_server.jpg)

> Everything has its limit. Apache Cordova applications are slower than native applications of course. Still, here is a list of known companies who are using it:

+ Facebook. Facebook uses a forked version of Apache Cordova in their mobile SDK. ...
+ Salesforce. Salesforce uses a fork of Apache Cordova for their mobile development SDK. ...
+ IBM. ...
+ Microsoft. ...
+ RIM. ...
+ Zynga. ...
+ Logitech.

### How to create an application that looks like an application?

It exists a bunch of frameworks that provide mobile application component like buttons, chips, notifications, etc. Here's a list of the most popular frameworks that offer that!


+ Ionic
+ React Native
+ Framework7
+ Quasar

> Since I wanted to optimize the approach, I made my choice in function of the HTML framework that was supported at this time. I heard good comments about VueJS performances. Since `Ionic 4` and `Quasar` offer it only in a beta version, I had to go with `Framework 7 v.3.2`

[VueJS](https://vuejs.org) seems to offer a better and cleaner templating, state management and documentation than React, but it also offers a better learning curve.

![vuejs](/assets/images/vuejs.jpg)

> Here's my first advice, when you start a project make sure to fork the right repository hehe! I would say, look for the one that is provided by the actual framework. That way, it will be references a lot more on the support pages. In other words, do what I say not what I do. 

<img src="/assets/images/framework7.jpg" style="max-width: 30%">

Here's the [framework7 repository](https://github.com/framework7io/framework7) that offers the VueJS [kitchen sink](https://github.com/framework7io/framework7/tree/master/kitchen-sink) boiler plate. 

> I don't have kids, but this will certainly by the sentence that I'll repeat the most: Do what I say and NOT what I do! ;-)
>
> My project is build on this [repository](https://github.com/caiobiodere/cordova-template-framework7-vue-webpack). ATM,  I am still strugglin with certain cordova-plugins.


First thing first, make sure you start building on a solid foundation ;-). [Webpack](https://webpack.js.org) is a [npm](https://www.npmjs.com/) library that let's you bundle up all the dependencies of the project.


__The following commands are for the repository that I used, which includes webpack.__

![webpack](/assets/images/webpack.jpg)

#### Pro tip: Do NOT use `npm` with sudo privilege. It's not necessary and might create dependency conflicts.

```bash
npm i -g cordova # install cordova localy
git clone https://github.com/caiobiodere/cordova-template-framework7-vue-webpack && cordova-template-framework7-vue-webpack/template_src
npm i # install the dependencies from package.json
cordova platform add browser ios android
cordova run browser --verbose -- --lr # open the app in a browser with Live Reload
```
> So far, nothing is incredible except Webpack that is really useful on the long run. Let's get into more technical details!

<br>

