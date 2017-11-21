---
title: "Angular2 - pre bootstrap loading animation"
description: "Create a pre bootstrap loading animation for angular projects in pure css"
tags:
  - angular2
date: "2017-03-07"
slug: "create-pre-bootstrap-animation"
image: "/images/spinner.png"
---

After a while it gets annoying to see angular2's native `loading..` everytime to start your app. Today's post will show you how to change this.
It is fairly easy to do, first let's take a look at our index.html file. I am going to use a `pre-bootstrap` wrapper to position our animation `pre-bootstrap-spinner` and a small label `pre-bootstrap-text`.

```html
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <title>XapiDemo</title>
  <base href="/">

  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="icon" type="image/x-icon" href="favicon.ico">
</head>
<body>
  <app-root>
    <div class="pre-bootstrap">
      <div class="pre-bootstrap-text">Loading..</div>
      <div class="pre-bootstrap-spinner"></div>
    </div>
  </app-root>
</body>
</html>
```

The styling is done with pure css animation.

```css
.pre-bootstrap{
  position: absolute;
  left: calc(50% - 30px);
  top:  calc(50% - 50px);
}

.pre-bootstrap-text{
  line-height: 20px;
  margin-bottom: 20px;

  font-family: Roboto,"Helvetica Neue",sans-serif;
}

.pre-bootstrap-spinner {
  width: 60px;
  height: 60px;
  background-color: #ff5252;

  /*margin: 0 auto;
  margin-top: calc(25% - 40px);*/
  -webkit-animation: sk-rotateplane 1.2s infinite ease-in-out;
  animation: sk-rotateplane 1.2s infinite ease-in-out;
}

@-webkit-keyframes sk-rotateplane {
  0% { -webkit-transform: perspective(120px) }
  50% { -webkit-transform: perspective(120px) rotateY(180deg) }
  100% { -webkit-transform: perspective(120px) rotateY(180deg)  rotateX(180deg) }
}

@keyframes sk-rotateplane {
  0% {
    transform: perspective(120px) rotateX(0deg) rotateY(0deg);
    -webkit-transform: perspective(120px) rotateX(0deg) rotateY(0deg)
  } 50% {
    transform: perspective(120px) rotateX(-180.1deg) rotateY(0deg);
    -webkit-transform: perspective(120px) rotateX(-180.1deg) rotateY(0deg)
  } 100% {
    transform: perspective(120px) rotateX(-180deg) rotateY(-179.9deg);
    -webkit-transform: perspective(120px) rotateX(-180deg) rotateY(-179.9deg);
  }
}
```
