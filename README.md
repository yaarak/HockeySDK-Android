This README describes on how to integrate HockeyApp into you Android apps. The client allows testers to update your app to another beta version right from within the application. It will notify the tester if a new update is available. It also allows to send crash reports right from within the application. If a crash has append, it will ask the tester on the next resume of the main activity whether he wants to send information about the crash to the server. The latter feature works for both beta apps and apps for the Android Market.

## Requirements

* ADT Plugin 0.9.7 (or higher)
* Android 2.2 (or higher)
* Eclipse 3.5 (or higher)

It might work with older SDKs and/or without Eclipse, but we haven't tested it.

## Integration Into Your Own App

### Import Library Project

* Checkout the latest version of [HockeySDK-Android](https://github.com/codenauts/HockeySDK-Android from GitHub:)<pre>git co git://github.com/codenauts/HockeySDK-Android</pre>
* Open Eclipse.
* Go to File > Import.
* Select General > Existing Projects into Workspace, then Next.
* Click Browse to choose the checked-out directory.
* Click Finish.

### Add Library Project

* Right-click on your project in the Package Explorer and select Properties.
* Select Android in the sidebar.
* Click on Add in the Library section.
* Select Hockey and then twice OK. You should be back in the Eclipse main window.

### Change Code

* Open your AndroidManifest.xml.
* Add the following line as a child element of application: <pre>&lt;activity android:name="net.hockeyapp.android.UpdateActivity" /></pre>
* If you want to do beta distribution and crash reporting, add the following lines as child elements of your manifest: <pre>&lt;uses-permission android:name="android.permission.INTERNET" />
&lt;uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" /></pre>
* If you want to do only crash reporting (e.g. for apps in the Android Market), add the following line as child elements of your manifest: <pre>&lt;uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" /></pre>
* Save the AndroidManifest.xml.
* Open your main activity or the activity in which you want to integrate the update process.
* Add the following lines:

<pre>import net.hockeyapp.android.CrashManager;
import net.hockeyapp.android.UpdateManager;
               
public class YourActivity extends Activity {
  @Override
  public void onCreate(Bundle savedInstanceState) {
    // Your own code to create the view
    // ...
    
    // Check for updates
    checkForUpdates();
  }

  @Override
  public void onResume() {
    super.onResume();
    checkForCrashes();
  }

  @Override
  public Object onRetainNonConfigurationInstance() {
    checkUpdateTask.detach();
    return checkUpdateTask;
  }
    
  private void checkForCrashes() {
    CrashManager.register(this, "https://rink.hockeyapp.net/", "APP_ID");
  }

  private void checkForUpdates() {
    UpdateManager.register(this, "https://rink.hockeyapp.net/", "APP_ID", R.drawable.icon);
  }
  
  // Probably more methods
}</pre>

* The param APP_ID has to be replaced by your app's identifier. There are two options two create this: Either upload an existing .apk file of your app to HockeyApp or create a new app manually. The App ID can then be found an the app's page in the section App Info.
* Build your project, create an .apk file, upload it to HockeyApp and you are ready to go.

The above code does two things: 

1. When the activity is created, the update managed launched an async tasks which checks for new updates. If it finds a new update, an alert dialog is shown and if the user presses Show, another activity is launched. Therefore the async task stores your activity as its context. 
2. When the activity is resumed, the crash manager is triggered and checks if a new crash was created before. If yes, it brings up a dialog to ask the user whether he wants to send the crash log to HockeyApp. The crash manage also registers a new exception handler, but only on the first launch.

The reason for the two different entry points is that the update check causes network traffic and therefore potential costs for your users. In contrast, the crash manager only searches for new files in the file system, i.e. the call is pretty cheap. 

## Checklist if Crashes Do Not Appear in HockeyApp

1. Check if the APP_ID matches the App ID in HockeyApp.

2. Check if the package name in your AndroidManifest.xml matches the Bundle Identifier of the app in HockeyApp. HockeyApp accepts crashes only if both the App ID and the package name equal their corresponding values in your app.

3. If your app crashes and you start it again, is the dialog shown which asks the user to send the crash report? If not, please crash your app again, then connect the debugger and set a break point in CrashManager.java, method [register](https://github.com/TheRealKerni/HockeyKit/blob/develop/client/Android/src/net/hockeyapp/android/CrashManager.java#L27) to see why the dialog is not shown.

5. If it still does not work, please [contact us](http://support.hockeyapp.net/discussion/new).