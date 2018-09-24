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

> Everything has its limit. Apache Cordova applications are slower than native applications of course. Still here is a list of known companies who are using it:

+ Facebook. Facebook uses a forked version of Apache Cordova in their mobile SDK. ...
+ Salesforce. Salesforce uses a fork of Apache Cordova for their mobile development SDK. ...
+ IBM. ...
+ Microsoft. ...
+ RIM. ...
+ Zynga. ...
+ Logitech.
