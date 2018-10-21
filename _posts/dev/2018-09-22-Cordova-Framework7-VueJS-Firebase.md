---
layout: dev
categories: [dev]
---

> I've been ask by a sponsor to develop a slot machine mobile application. I wasn't sure if I should accept it since I never develop a whole project from scratch. So I made some researches on the upstream and found many solutions which I tought could be realist for me and accepted the deal.

This article will walk you trough this journey. Hopefuly, it will guide you where I found my self a lost. Let's be honest, I am NOT a senior full stack developer and the main reason why I accepted to make this project, is to learn. I encourage everyone to do so :-)

![](/assets/images/gohead.jpeg)

> It's when I started searching for a cross platform solution that I met [cordova](https://cordova.apache.org/). 

+ Mobile apps with HTML, CSS & JS
+ Target multiple platforms with one code base
+ Free and open source

![logo](/assets/images/logo_cordova_phonegap.jpg)

This framework do NOT transpile your HTML base project into a native languague. It simply creates an application that is aloud to run in a `WebView`. Thus, these applications are called `hybrid` not `native`.

![chart](/assets/images/cordova_chart.jpg)

Essentially, Cordova Appache only runs a server localy which contains all the pages of your application and uses the platform's libraries to render it as HTML or to run native functionality.

![server](/assets/images/cordova_server.jpg)

> Everything has its limit. Apache Cordova applications are slower than native applications of course. Still, here is a list of known companies that are using it:

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

![legit](/assets/images/legit.jpg)

> Since I wanted to optimize the approach, I made my choice in function of the HTML framework that was supported at this time. I heard good comments about VueJS performances. Since `Ionic 4` and `Quasar` offer it only in a beta version, I had to go with `Framework 7 v.3.2`

[VueJS](https://vuejs.org) seems to offer a better and cleaner templating, state management and documentation than React, but it also offers a better learning curve.

![vuejs](/assets/images/vuejs.jpg)

> Here's my first advice, when you start a project make sure to fork the right repository hehe! I would say, look for the one that is provided by the actual framework. That way, it will be references a lot more on the support pages.

<img src="/assets/images/framework7.jpg" style="max-width: 30%">

Here's the [framework7 repository](https://github.com/framework7io/framework7) that offers the VueJS [kitchen sink](https://github.com/framework7io/framework7/tree/master/kitchen-sink) boiler plate. 

> I don't have kids, but that will certainly by the sentence that I'll repeat the most: Do what I say and NOT what I do! ;-)

> My project is build on this [repository](https://github.com/caiobiodere/cordova-template-framework7-vue-webpack). ATM,  I am still strugglin with certain cordova plugins. There's a lot of work around though. Still, I is necessary to be able to use it. I'll update this post as soon as this `issues` will be closed.


First thing first, make sure you start building on a solid foundation ;-). [Webpack](https://webpack.js.org) is a [npm](https://www.npmjs.com/) library that let you bundle up all the dependencies of a JS project. Since this project will be handling a lot of depency and framework, I strongly recommand using it.

![webpack](/assets/images/webpack.jpg)

__The following commands are for the repository that I used, which includes webpack.__

#### Pro tip: Do NOT use `npm` with sudo privilege. It's not necessary and might creates dependency conflicts.

```bash
npm i -g cordova # install cordova localy
git clone https://github.com/caiobiodere/cordova-template-framework7-vue-webpack && cordova-template-framework7-vue-webpack/template_src
npm i # install the dependencies from package.json
cordova platform add browser ios android
cordova prepare # or cordova build
```
> FMPOV, the best to develop is to run the envionement in a browser. That way, you can easily console.log and see the error in the console (F12).

```
cordova run browser --verbose -- --lr # open the app in a browser with Live Reload
```

If you want to develop and test in an emulator, you need to download and install SDK tools. Cordova covers those [steps](https://cordova.apache.org/docs/fr/latest/guide/platforms/android/index.html#installer-le-sdk-android) for both platforms. The installations are a bit a pain in the ass though!

#### Pro tip: if you are not running on a Mac, you can spin [Mac OS X Lion](https://www.hackintosh.computer/199/direct-download-macos/) in a VM:

> At this point you should be able to run your boiler plate in a developement environement (browser, Android SDK, )
<br>

- CORS Request
- PopUps (interactive)
- Promises




































To make it sign by Google Play:

```
cordova build android --release
cp /home/srbz/freeTattoo/app_freeTattoo/platforms/android/app/build/outputs/apk/release/app-release-unsigned.apk app.apk
jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore mySuperKey.keystore app.apk alias_name
cp app.apk ~/
ls -ltr ~/
```
