---
layout:     post
title:      "Building an AAR Library in Android Studio"
subtitle:   "Part 1: Intro and Setup"
date:       2015-05-31 20:00:00
author:     "Romina"
header-img: "img/post-bg-06.jpg"
---

<h3>What is an ".aar" library?</h3>
<p>
The .aar bundle is the binary distribution of the Android Library Project.
It is simply a zip file with at least the following entries at the root level: 
<ul>
	<li>/AndroidManifest.xml</li>
	<li>/classes.jar</li>
	<li>/res/</li>
	<li>/R.txt</li>
</ul>
It might also include /assets/, /libs/, /jni/, /profguard.txt and /lint.jar.
</p>

<h3>Why use an .aar?</h3>
<p>
Often after building one or two applications you find common code you could like to pull out of your project and centralize in one location. You can then add the library to your project as a gradle dependency and make all changes only in one place rather than copy pasting bug fixes to several repos.
</p>
<p>
Once your library is all neat and ready you can share it with the community and spare some fellow developers the work you have just gone through.You can also benefit from community solutions by integrating third party libraries into your project.
</p>
<p>
In summary you use .aar libraries to:
<ul>
	<li>Organize your code</li>
	<li>Share your solutions with the community</li>
	<li>Use other developers solutions</li>
</ul>
</p>

<h3>How to build an .aar?</h3>
<p>
It is not necessary, but I recommend having all your source code into a git repo.
Library projects cannot be compiled to Android applications and started without another project using them, so for testing purposes I first created a project and then created a module.
<ul>
	<li><b>Project:&nbsp;</b>AppCore</li>
	<li><b>Module Application:&nbsp;</b>app</li>
	<li><b>Module Library:&nbsp;</b>apilibrary</li>
</ul>
Within Android Studio (v1 and above) right click on your Project structure and choose the option "new module".
</p>
[SOURCE CODE](https://github.com/rliuzzi/apilibrary)
