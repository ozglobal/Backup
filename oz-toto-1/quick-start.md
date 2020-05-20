# Quick Start

## Download Sample App

## ‌Create Android App

### ☑ Create Android project

‌Create a new project with Empty Activity.‌ In this example, we're using Android Studio 3.4.2.‌

### ☑ Install ozlicense, Toto framework and Android viewer

Create a folder app/src/main/assets/license and add an android viewer license file ozlicense.xml \(ask your OZ e-Form sales representative\).‌

Unzip oztoto80\_android\_20190724201.tar.gz and copy oztotoframework\_android.jar to app/libs.‌

Unzip ozrv80\_android\_20190821100.tar.gz and copy android-support-v4.jar, ozprint\_android.jar, ozrv\_android.jar, ozrv\_android.jar.properties to app/libs.‌

Add project module dependencies for 3 jar files as pictured below.‌

Copy armeabi folder to app/src/main/jniLibs.‌

Copy armeabi/libozrvpack\_pdfwriter.so to app/libs for saving eform/report as local pdf. Project file tree will be like as captured below.‌

### ☑ AndroidManifest.xml

Copy below into the AndroidManifest.xml

```markup
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    package="com.forcs.toto">
    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@android:style/Theme.Holo.Light"
        android:largeHeap="true"
        android:hardwareAccelerated="true"
        android:usesCleartextTraffic="true"
        tools:replace="android:appComponentFactory"
        android:appComponentFactory="whateverString">
        <activity
            android:name="com.forcs.toto.MainActivity"
            android:windowSoftInputMode="adjustResize"
            android:configChanges="orientation|screenSize" >
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>
    </application>
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.RECORD_AUDIO" />
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />
    <uses-permission android:name="android.permission.WAKE_LOCK" />
    <uses-permission android:name="android.permission.CALL_PHONE" />
    <uses-permission android:name="android.permission.VIBRATE"/>
    <uses-permission android:name="android.permission.NFC"/>
    <uses-feature android:name="android.hardware.camera" android:required="false"/>
    <uses-feature android:name="android.hardware.camera.autofocus" android:required="false"/>
</manifest>
```

### ☑ MainActivity.java

Copy below into the MainActivity.java.

```java
package com.forcs.toto;

import androidx.annotation.RequiresApi;
import android.os.Bundle;
import android.os.Build;
import android.app.Activity;
import android.app.AlertDialog;
import android.content.Intent;
import android.view.Window;
import android.widget.FrameLayout;
import android.Manifest;
import android.content.Context;
import android.content.pm.PackageManager;
import android.content.DialogInterface;

import oz.toto.framework.OZTotoEvent;
import oz.toto.framework.OZTotoEventHandler;
import oz.toto.framework.OZTotoError;
import oz.toto.framework.OZTotoRuntime;
import oz.toto.framework.OZTotoWebView;
import oz.toto.framework.OZTotoWebViewListener;

public class MainActivity extends Activity {

    FrameLayout parentView = null;
    OZTotoWebView toto = null;
    OZTotoRuntime toto_runtime = null;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
    
        super.onCreate(savedInstanceState);
        requestWindowFeature(Window.FEATURE_NO_TITLE);
        requestAppPermission(this);

        parentView = new FrameLayout(this);
        toto = new OZTotoWebView(this);
        parentView.addView(toto, new FrameLayout.LayoutParams(FrameLayout.LayoutParams.MATCH_PARENT, FrameLayout.LayoutParams.MATCH_PARENT));
        setContentView(parentView);
        OZTotoWebView.setDebugMode(true);
        
        // prepare oz server connection. replace ip with your oz server ip.
        private String ozserverip = "http://127.0.0.1:8080/oz"; 
        private String page = "";
        private String param = "";

        toto.setTotoWebViewListener(new OZTotoWebViewListener() {
            @Override
            public void onPageLoad(final OZTotoRuntime runtime) {
                // add toto event to get from the server
                runtime.getFramework().addEventListener("_exitApp_", OZTotoFrameworkEvent);
                toto_runtime = runtime;
            }
            @Override
            public void onPageUnload(OZTotoRuntime runtime) {
            }
            @Override
            public void onSelectedMenuButton() {
            }
            @Override
            public void onPageLoadFailure(OZTotoError ozTotoError) {
                System.out.println(ozTotoError.errMsg);
            }
            // add toto event handler
            OZTotoEventHandler OZTotoFrameworkEvent = new OZTotoEventHandler() {
                @RequiresApi(api = Build.VERSION_CODES.M)
                @Override
                public void onEvent(OZTotoEvent e) {
                    if (e.eventName.equals("_exitApp_")) {
                        exit();
                    }
                }
            };
        });
        toto.run(ozserverip, page, param); // connect to oz server
    }

    private void exit() {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                new AlertDialog.Builder(MainActivity.this)
                        .setIcon(android.R.drawable.ic_dialog_alert)
                        .setTitle("Exit")
                        .setMessage("Are you sure you want to exit?")
                        .setPositiveButton("YES", new DialogInterface.OnClickListener() {
                            @Override
                            public void onClick(DialogInterface dialog, int which) {
                                finish();
                            }
                        })
                        .setNegativeButton("NO", null)
                        .show();
            }
        });
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        if (toto != null) {
            toto.onActivityResult( this, requestCode, resultCode, data);
        }
        super.onActivityResult(requestCode, resultCode, data);
    }
    
    static boolean requestAppPermission(Context context) {
        if (Build.VERSION.SDK_INT >= 23) {
            String[] need = {Manifest.permission.WRITE_EXTERNAL_STORAGE, Manifest.permission.READ_EXTERNAL_STORAGE, Manifest.permission.CAMERA, Manifest.permission.RECORD_AUDIO, Manifest.permission.ACCESS_FINE_LOCATION, Manifest.permission.ACCESS_COARSE_LOCATION, Manifest.permission.READ_PHONE_STATE};
            for (int i = 0; i < need.length; i++) {
                if (context.checkSelfPermission(need[i]) != PackageManager.PERMISSION_GRANTED) {
                    if (context instanceof Activity) {
                        ((Activity) context).requestPermissions(need, 1);
                    }
                    return false;
                }
            }
        }
        return true;
    }
}
```

## Create Web App <a id="2-create-web-application"></a>

### ☑ ‌toto.manifest.js <a id="3-1-toto-manifest-js"></a>

‌‌Create toto.manifest.js as below under your web application \(in this example, webapps/oz/toto\).toto.manifest.js

```javascript
{
    "home" : { 
        "online" : "sample.html",
		"offline" : null
    },
    "navigator" : {
        "visible" : true,
		"setting" : true
    }
}
```

‌‌Once the App we created above connects to the oz server, it will first see this toto.manifest.js and open sample.html page.‌‌

At the bottom of mobile app screen will show navigation bar with home, back, forward, refresh and settings icon. If you want your own navigation control, just set **visible** and **setting** to false.

### ‌☑ Sample.html

Let's create a simple html application as below to open an e-Form from the server.

```markup
<html>
<head>
<title>OZ Toto Framework</title>

<script src="oztotoframework.js"></script>
<script language="JavaScript">

var context = "http://" + location.host + "/oz";
var ozviewer = new OZTotoFramework.OZViewer();

function createViewer() {
	var	param = "";
	param += "\n" + "connection.servlet=" + context + "/server";
	param += "\n" + "connection.reportname=//toto//sample.ozr"; // your ozr file
	param += "\n" + "connection.pcount=1";
	param += "\n" + "connection.args1=videourl=" + context + "/toto/video.mp4"; // OZ Form Parameter

	ozviewer.createViewer('bottom', param, "\n");
	ozviewer.setVisible(true);
}

function disposeViewer() {
	ozviewer.dispose();
}
</script>
</head>
<body>
<div>
<table>
	<tr>
		<td width="150"><a href="javascript:createViewer();">Open</a></td>
		<td width="150"><a href="javascript:disposeViewer();">Close</a></td>
	</tr>
</table>
</div>
</body>
</html>
```

## Run Sample App

Finally, build and run the app and then you will see the sample eform on mobile device like this.

