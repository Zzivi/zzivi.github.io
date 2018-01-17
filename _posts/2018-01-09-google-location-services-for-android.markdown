---
layout:     post
title:      "Google Play Services Location API"
subtitle:   "Location awareness in Android part II"
date:       2018-01-09 15:00:00
author:     "Romina Liuzzi"
header-img: "img/location-aware-app.jpg"
icon-img:   "img/icons/android.jpg"
---

<p>
	To implement location awareness in your Android app Google supports two location strategies: <a href="/2018/01/08/android-location-package/">android.location package</a> and <a href="#">Google Play Services Location API</a>. Google strongly advices developers to use the second approach. 
</p>
<p>
	In fairness Google has simplified the process greatly and claims following this approach improves battery performance in most cases. This makes sense as the Location Service is run as system process to which you subscribe. On the less bright side this approach depends on Google Play Services being installed in the phone. 
</p>
<p>
	So in order to implement this solution and make sure your app gracefully handles all possible combination of scenarios there are a few things you need to do.
</p>
<p>
	<ol>
		<li><a href="#setup">Setup Google Play Services Location API</a></li>
		<li><a href="#verify">Verify Google Play Services is installed</a></li>
		<li><a href="#permissions">Handle App Permissions</a></li>
		<li><a href="#service">Implement Location Service</a></li>
	</ol>
</p>

<h3 id="setup">Setup Google Play Services Location API</h3>
<p>
	Google Play Services comprises several services. To get the user current location and request location updates from your Android app you will need to include the <b>play-services-location</b> package.
</p>
<p>
	In order to include this package in your app as a selective dependency of the Google Play Services you need to follow these steps:
	<ol>
		<li>
			If you haven't already done so, download and install Google Play Services from Android Studio's SDK Manager.
			<img src="/img/posts/post_20180109_3.png">
		</li>
		<li>
			Include google() or maven { url "https://maven.google.com" } repository to top level build.gradle
			<img src="/img/posts/post_20180109_2.png">
		</li>
		<li>
			Include play-services-location dependency to app level build.gradle
			{% highlight yaml %}
			compile 'com.google.android.gms:play-services-location:11.8.0'
			{% endhighlight %}
		</li>
		<li> 
			Sync Gradle dependencies
			<img src="/img/posts/post_20180109_1.png">
		</li>
	</ol>
</p>

<h3 id="verify">Verify Google Play is installed</h3>
<p>
	Your app should verify the required version of the Google Play Services APK is installed in the device. There are two strategies to perform this step:
	<ol>
		<li>Delegate the check on the service client object by passing an Activity at time on instatiation<a href="https://developers.google.com/android/guides/api-client#accessing_google_services">*</a>
			{% highlight yaml %}
			//this must be an Activity
			FusedLocationClient client = LocationServices.getFusedLocationProviderClient(this); 
			{% endhighlight %}
		</li>
		<li>
			Use the GoogleApiAvailibility singlenton service to manually check:
			{% highlight yaml %}
			public boolean checkPlayServices(final IcabbiActivity activity) {
		        GoogleApiAvailability apiAvailability = GoogleApiAvailability.getInstance();
		        int resultCode = apiAvailability.isGooglePlayServicesAvailable(context);
		        if (resultCode != ConnectionResult.SUCCESS) {
		            if (apiAvailability.isUserResolvableError(resultCode)) {
		                Dialog playServicesErrorDialog = apiAvailability.getErrorDialog(activity, resultCode, PLAY_SERVICES_RESOLUTION_REQUEST);
		                playServicesErrorDialog.setOnCancelListener(new DialogInterface.OnCancelListener() {
		                    @Override
		                    public void onCancel(DialogInterface dialogInterface) {
		                        activity.finish();
		                    }
		                });
		                playServicesErrorDialog.setOnDismissListener(new DialogInterface.OnDismissListener() {
		                    @Override
		                    public void onDismiss(DialogInterface dialog) {
		                    }
		                });
		                playServicesErrorDialog.show();
		            } else {
		                Timber.e(String.format("This device is not supported. Result code: %s", resultCode));
		                activity.finish();
		            }
		            return false;
		        }
		        return true;
		    }
			{% endhighlight %}
			Then in your activiy get the intent result
			{% highlight yaml %}
			@Override
		    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
		        super.onActivityResult(requestCode, resultCode, data);
		        if (requestCode == PermissionHelper.PLAY_SERVICES_RESOLUTION_REQUEST) {
		            if (resultCode != ConnectionResult.SUCCESS) {
		                Timber.e(String.format("This device is not supported. Result code: %s", resultCode));
		                finish();
		            }
		        }
		    }
   			{% endhighlight %}
		</li>
	</ol>
</p>

<h3 id="permissions">Handle App Permissions</h3>
<p>
	Since Android M  permission granting was moved from install time to runtime. This is good for the user and hell for the developer. There is no way around this one. Before accessing the device location you must cofirm you have permission to do so and fail gracefully if you don't.
</p>
<p>
	I am currently using the <a href="https://github.com/permissions-dispatcher/PermissionsDispatcher" target="_blank">Permision Dispatcher library</a> to handle the permissions checks.
</p>

<h3 id="service">Implement Location Service</h3>
<p>
	If you need to get location updates from you app you need to be aware that since Android O, if your app is running in the background the number of requests your app can make to the FusedLocationManager and the LocationsManager are restricted to a few every hour.<a href="https://developer.android.com/about/versions/oreo/android-8.0-changes.html#abll" target="_blank">*</a>.
</p>
<p>
	Every app is different. If you need to request frequent updates you can run your location service as a foreground service<a href="https://developer.android.com/about/versions/oreo/background-location-limits.html#tuning-behavior" target="_blank">*</a>.
</p>

<h3 id="references">References</h3>
<p>
	<ul>
		<li><a href="https://developers.google.com/android/guides/setup/" target="_blank">Setup Google Play Services</a></li>
		<li><a href="https://developers.google.com/android/guides/setup#ensure_devices_have_the_google_play_services_apk" target="_blank">Ensure Devices have the Google Play Services APK</a></li>
		<li><a href="https://developer.android.com/training/location/index.html" target="_blank">Making your app location aware</a></li>
		<li><a href="https://developers.google.com/android/guides/permissions" target="_blank">Google Play Services and Runtime Permissions</a></li>
		<li><a href="https://developer.android.com/about/versions/oreo/android-8.0-changes.html#abll" target="_blank">Android 8.0 Behavior Changes (Location Limits)</a></li>
		<li><a href="https://developer.android.com/about/versions/oreo/background-location-limits.html#tuning-behavior" target="_blank">Background Location Limits: Tunning your app's location behaviour</a></li>
		<li><a href="https://developer.android.com/guide/components/services.html#Foreground" target="_blank">Services: Running a service in the foreground</a></li>
		<li><a href="https://developer.android.com/training/location/change-location-settings.html#connect">Configure Location Services</a></li>
		<li><a href="https://developer.android.com/guide/topics/ui/notifiers/notifications.html#SimpleNotification" target="_blank">Notifications: Creating a simple notification</a></li>
		<li><a href="https://developer.android.com/training/notify-user/index.html">Notifying the User</a></li>
		<li><a href="https://developer.android.com/about/dashboards/index.html#Platform" target="_blank">Android Platforms Adoption Dashboard</a></li>
	</ul>
</p>











