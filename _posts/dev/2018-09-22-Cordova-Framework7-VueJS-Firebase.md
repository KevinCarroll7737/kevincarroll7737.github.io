---
layout: dev
categories: [dev]
---

> I've been ask by a sponsor to develop a slot machine mobile application. I wasn't sure if I should accept since I never develop a whole project from scratch. So I made some researches on the upstream and found bunch of frameworks and Git repos which made me I think that it coul be realist for me. So I closed the deal.

This article will walk you trough this journey. Hopefuly, it will guide you where I found my self lost... Let's be honest, I am NOT a senior full stack developer and the main reason why I accepted to make this project, was to grow and learn. I encourage everyone to do so :-)

![](/assets/images/gohead.jpeg)

> It's when I started searching for a cross platform solution that I met [Cordova](https://cordova.apache.org/). 

+ Mobile apps with HTML, CSS & JS
+ Target multiple platforms with one code base
+ Free and open source

<img style="width: 100%; height: auto" src="/assets/images/logo_cordova_phonegap.jpg" >

This framework do NOT transpile your HTML base project into a native languague. It simply creates an application that is aloud to run in a `WebView`. Thus, these applications are called `hybrid` not `native`.


<img style="width: 100%; height: auto" src="/assets/images/cordova_chart.jpg" >

Essentially, Cordova Appache only runs a server localy which contains all the pages of your application and uses the platform's libraries to render it as HTML or to run native functionality.

<img style="width: 70%; height: auto" src="/assets/images/cordova_server.jpg" >

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

<img style="width: 70%; height: auto" src="/assets/images/legit.jpg" >

> Since I wanted to optimize the approach, I made my choice in function of the HTML framework that was supported at this time. I heard good comments about VueJS performances. Since `Ionic 4` and `Quasar` offer it only in a beta version, I had to go with `Framework 7 v.3.2`

[VueJS](https://vuejs.org) seems to offer a better and cleaner templating, state management and documentation than React, but it also offers a better learning curve.


<img style="width: 70%; height: auto" src="/assets/images/vuejs.jpg" >

> Here's my first advice, when you start a project make sure to fork the right repository hehe! I would say, look for the one that is provided by the actual framework. That way, it will be references a lot more on the support pages.

<img src="/assets/images/framework7.jpg" style="max-width: 30%">

Here's the [framework7 repository](https://github.com/framework7io/framework7) that offers the VueJS [kitchen sink](https://github.com/framework7io/framework7/tree/master/kitchen-sink) boiler plate. 

> I don't have kids, but that will certainly by the sentence that I'll repeat the most: Do what I say and NOT what I do! ;-)

> My project is build on this [repository](https://github.com/caiobiodere/cordova-template-framework7-vue-webpack). ATM,  I am still strugglin with certain cordova plugins. There's a lot of work around though. Still, I is necessary to be able to use it. I'll update this post as soon as this `issues` will be closed.


First thing first, make sure you start building on a solid foundation ;-). [Webpack](https://webpack.js.org) is a [npm](https://www.npmjs.com/) library that let you bundle up all the dependencies of a JS project. Since this project will be handling a lot of depency and framework, I strongly recommand using it.

<img src="/assets/images/webpack.jpg" style="width: 100%; height: auto">

__The following commands are for the repository that I used, which includes webpack.__

#### Pro tip: Do NOT use `npm` with sudo privilege. It's not necessary and might creates dependency conflicts.

    npm i -g cordova # install cordova localy
    git clone https://github.com/caiobiodere/cordova-template-framework7-vue-webpack && cordova-template-framework7-vue-webpack/template_src
    npm i # install the dependencies from package.json
    cordova platform add browser ios android
    cordova prepare # or cordova build

> FMPOV, the best to develop is to run the envionement in a browser. That way, you can easily console.log and see the error in the console (F12).

    cordova run browser --verbose -- --lr # open the app in a browser with Live Reload

If you want to develop and test in an emulator, you need to download and install SDK tools. Cordova covers those [steps](https://cordova.apache.org/docs/fr/latest/guide/platforms/android/index.html#installer-le-sdk-android) for both platforms. The installations are a bit a pain in the ass though!

#### Pro tip: if you are not a Mac user, you can spin [Mac OS X Lion](https://www.hackintosh.computer/199/direct-download-macos/) in a VM:

> At this point you should be able to run your boiler plate in a developement environement (browser, Android SDK, )

<br>

#### Firebase Functions: HTTP Request

It might be a bit abstract for you, but in fact this project has a client side REST application server, its DB is in the cloud and you can call simples HTTPs GET request to process data... lol! 

Here's how to manage these HTTPs requests sent to the firebase backend.

<img style="width: 70%; height: auto" src="/assets/images/goat.jpeg" >

> Let's make this simple:

+ Every `functions.https.onRequest()` has to send a response (i.e.: `res.send('Your Response')`).
+ The exports method is what Firebase use as referer.
+ Simply go to Firebase > Functions > Dashboard, and copy the associated URL to call it..
+ Simply pass the parameter as regular backend PHP POST request (i.e.: `https://ASSOCIATED_URL?REQ_PARAM_1&REQ_PARAM_2&REQ_PARAM_n`).

```js
// Constants init
const functions = require('firebase-functions');
const admin = require('firebase-admin');
admin.initializeApp();

const cors = require('cors'); // Allows cross origin requests
const stripe = require('stripe')(functions.config().stripe.token);  // Securely use Stripe token (optional)

/**
 * Stripe Connect (Standard) API
 *
 * Listen for new POST of Stripe. 
 * Add the "acct" to the appropriate shop in the DB.
 *
 */
var confirmMail = function confirmMail(req, res){

    const q = req.query

    // Error
    if (q.error){
        res.send(q.error_description)    
    // No Error
    } else {
                                                                                                       
        res.send("Succes: 200")
    }   


}    

exports.stripeConnect= functions.https.onRequest((req, res) => {
    var corsFn = cors();
    corsFn(req, res, function() {
        stripeConnect(req, res);
    }); 
})

```

"The Same-Origin Policy (SOP) might be too restrictive for large applications that use, for example, multiple subdomains.  There are a number of techniques for relaxing the SOP in a controlled manner. One of these techniques is the Cross-Origin Resource Sharing (CORS)."

> In other words, client side needs to process information from another server. Most of the time this formation is a JSON response. For more infos on that subject go read [this](https://www.bedefended.com/papers/cors-security-guide).


<img style="width: 100%; height: auto" src="/assets/images/firebase.jpg" >

Finaly, in `app/config.xml` modify `<access origin=...>` to fits your need! ;)

#### JavaScript Promises:

> This is the easiest thing in the world, but there's so many ECMAScript versions that exist. So I decided to add this topic ;)

Promises are essential when communicating with the backend. This is what tells the frontend to wait for the ressources to be received before processing and rendering.

+ 4th Edition (abandoned) ...
+ 5th Edition. ...
+ 6th Edition - ECMAScript 2015. ...
+ 7th Edition - ECMAScript 2016. ...
+ 8th Edition - ECMAScript 2017. ...
+ 9th Edition - ECMAScript 2018.

For the stake of this post, I will be covering ES8 (ECMAscript 2017) which is a lot intuitive than promises in ES7.

```js
function fetchJson(url) {
    return fetch(url)
    .then(request => request.text())
    .then(text => {
        return JSON.parse(text);
    })
    .catch(error => {
        console.log(`ERROR: ${error.stack}`);
    });
}
fetchJson('http://example.com/some_file.json')
.then(obj => console.log(obj));
```

The following variants of async functions exist:

    Async function declarations: async function foo() {}
    Async function expressions: const foo = async function () {};
    Async method definitions: let obj = { async foo() {} }
    Async arrow functions: const foo = async () => {};


#### Before having it sign by Google Play:

For security, it is essential the having you application sign. An unsign application might be blocked by other applications. 

    cordova build android --release
    cp /home/srbz/freeTattoo/app_freeTattoo/platforms/android/app/build/outputs/apk/release/app-release-unsigned.apk app.apk
    jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore mySuperKey.keystore app.apk alias_name
    cp app.apk ~/
    ls -ltr ~/
