---
layout:     post
title:      "Generate Android APKs with Travis"
subtitle:   "Automatically upload Android APKs to Github Releases with Travis"
date:       2015-08-25 00:00:00
author:     "Daniel Viorreta"
header-img: "img/travis-bg.png"
icon-img:   "img/icons/android.jpg"
---

<h2>Motivation</h2>
I wanted that every time that I commit a change in my Android repo a new APK is automatically generated and stored. Although I use Jenkins as my Continuous Integration server at work, I use Travis CI for this project. Travis CI launched Android Builds Support in Beta and I wanted to give them a chance :P 

<h2>Building the project</h2>
Travis CI automatically detects when a commit has been made and pushed to a GitHub repository that is using Travis CI, and each time this happens, it will try to build the project and run tests.
Following the instructions that Travis CI provides is very easy to setup your project and compile your code:
[Building an Android Project with Travis](http://docs.travis-ci.com/user/languages/android/)



<h2>Uploading the APK in GitHub Releases</h2>
<p>
Travis can upload the artifacts of your build in AWS S3. But I want to have the APK in the same place where a I have my code (I save a little of money too :P)
</p>
<p>
So I setup my yaml file to upload the APK to GitHub Releases every time that a new tag in Github is created. If you use Git Flow, it will create a tag automatically for each release. So you will have your APK ready to be published in Google Play.
</p>
{% highlight yaml %}
language: android
script: ./gradlew build assembleRelease
android:
  components:
  - build-tools-20.0.0
  - android-19

deploy:
  - provider: releases
    api_key:
      <your key>
    file: app/build/outputs/apk/app-release.apk
    on:
      repo: Zzivi/Sodexo
      tags: true

notifications:
  email:
    recipients:
    - daniel.viorreta@zzivi.com
    - romina.liuzzi@zzivi.com
    on_success: always
    on_failure: always
{% endhighlight %}

Notice that I use <i>./gradlew build assembleRelease</i> to avoid launching the Android emulator. The <i>deploy</i> section has the configuration to upload the apk only when a tag is created.

[Android App using Travis](https://github.com/zzivi/Sodexo)
<br>
[GitHub Releases Uploading with Travis](http://docs.travis-ci.com/user/deployment/releases/)
