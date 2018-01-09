---
layout:     post
title:      "Google Location Services"
subtitle:   "Location awareness in Android part II"
date:       2018-01-09 15:00:00
author:     "Romina Liuzzi"
header-img: "img/location-aware-app.jpg"
icon-img:   "img/icons/android.jpg"
---

<p>
	To get the user current location and request location updates from your Android app there are two (Google supported) location strategies to choose from.
	You can either use the SDK embadded <a href="/2018/01/08/android-location-package/"><b>android.location package</b></a> or the <b>Google Play Services Location APIs</b>. 
</p>
<h3>
	Google Play Services Location APIs
</h3>

<p>
	The plan is to attach a LocationListener to a LocationManager for a given LocationProvider (NETWORK_PROVIDER, GPS_PROVIDER,...) and listen for updates. 
	{% highlight yaml %}
	String locationProvider = LocationManager.NETWORK_PROVIDER;
	locationManager.requestLocationUpdates(locationProvider, 0, 0, locationListener);
	{% endhighlight %}
</p>











