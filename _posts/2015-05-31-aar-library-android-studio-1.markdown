---
layout:     post
title:      "Building an AAR Library in Android Studio"
subtitle:   "Part 1: Intro and Setup"
date:       2015-05-31 20:00:00
author:     "Romina"
header-img: "img/post-bg-06.jpg"
---
<p>
What is an ".aar" library?
The .aar bundle is the binary distribution of the Android Library Project.
It is simply a zip file with at least the following entries at the root level: 
/AndroidManifest.xml
/classes.jar
/res/
/R.txt
It might also include /assets/, /libs/, /jni/, /profguard.txt and /lint.jar.
</p>
<p>
Why use an .aar?
Often after building one or two applications you find common code you could like to pull out of your project and centralize in one location. You can then add the library to your project as a gradle dependency and make all changes only in one place rather than copy pasting bug fixes to several repos.  
Once your library is all neat and ready you can share it with the community and spare some fellow developers the work you have just gone through.
You can also benefit from community solutions by integrating third party libraries into your project.
In summary you use .aar libraries to:
- Organize your code 
- Share your solutions with the community
- Use other developers solutions
</p>
<p>
How to build an .aar?
It is not necessary, but I recommend having all your source code into a git repo.
Library projects cannot be compiled to Android applications and started without another project using them, so for testing purposes I first created a project and then created a module.

Project: AppCore
Module Application: app
Module Library: apilibrary

Within Android Studio (v1 and above) right click on your Project structure and choose the option "new module".
all the sources are available at:
https://github.com/rliuzzi/apilibrary
</p>