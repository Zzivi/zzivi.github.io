---
layout:     post
title:      "android.location package"
subtitle:   "Location Awareness in Android part I"
date:       2018-01-08 13:15:00
author:     "Romina Liuzzi"
header-img: "img/location-aware-app.jpg"
icon-img:   "img/icons/android.jpg"
---

<p>
	To get the user current location and request location updates from your Android app you could use the SDK embadded <b>android.location package</b>. 
	As these classes are part of the Android SDK there is no prior setup required and you don't need to add any dependencies to your gradle file.
</p>
<h3>
	Warning
</h3>
![image-title-here](/img/posts/android.location.deprecation.png){:class="img-responsive"}
<p>
	Google strongly advises to stop using this API and migrate your apps to use Google Location Services APIs instead
</p>
<p>
	Check out the next post to learn more about <a href="/2018/01/09/google-location-services-for-android/"><b>Google Location Services APIs</b></a>
</p>
<h3>
	android.location package
</h3>
<p> 
	You will need to get familiar with the classes: 
	<ul>
		<li>LocationProvider</li>
		<li>LocationManager</li>
		<li>LocationListener</li>
	</ul>
</p>
<p>
	The plan is to attach a LocationListener to a LocationManager for a given LocationProvider (NETWORK_PROVIDER, GPS_PROVIDER,...) and listen for updates. 
	{% highlight yaml %}
	String locationProvider = LocationManager.NETWORK_PROVIDER;
	locationManager.requestLocationUpdates(locationProvider, 0, 0, locationListener);
	{% endhighlight %}
</p>
<p>
	There are some considerations to keep in mind when using this approach like what provider to choose (i.e: GPS is more accurate but doesn't work indoors). You also need to consider every how often your updates should happen and how accurate they should be. These decisions will have an impact on the battery life of the device your app is running on so you have to be careful. 
</p>
<p>
	Checkout the full tutorial: <a href="https://developer.android.com/guide/topics/location/strategies.html" target="_blank">Android Developer Documentation</a>
</p>












