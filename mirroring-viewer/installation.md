---
description: How to install Relay Server and Mirroring Viewer on your server.
---

# Installation

## Requirements

### ☑ OZ Server License

Make sure that your OZ license file \(`WEB-INF/license/ozlicense.xml`\) included the permission for mirroring as below.

```text
USE-SUBMONITOR="TRUE"
```

### ☑ OZ Relay Server

* Relay server requires JAVA 8 or later installed on the server.
* Inbound port \(ex: 14127\) is used by EPG to allow access from the sub viewer.
* Database port \(ex: 8441\) is used by DBS to access database.

## Installing OZ Relay Server

1. Download OZ Relay Server [here](https://drive.google.com/open?id=1q0YEKJgMleVXjD0-PodgHIJk23qUGCtS).
2. Extract it to your installation folder \(ex: OZRelayServer\). You will see two subfolders - EPG and DBS.

### ☑ Setting DBS \(DatabaseServer\)

SQLite database is included in the installation by default. You can use Oracle if you want.

#### Files under DBS

| File | Description |
| :--- | :--- |
| config/example/application.properties.SQLITE | Sample  for SQLite |
| config/application.properties | Configuration file |
| sync.db | database file |
| TP\_DBS.jar | Executable file |
| dbs\_linux.sh | Command script for Linux bash |
| dbs\_unix.sh | Command script for Unix |
| dbs\_windows.agent.cmd | Command script for Windows \(daemon mode\) |
| dbs\_windows.console.cmd | Command script for Windows \(console mode\) |

#### Setting application.properties

| Item | Value | Description |
| :--- | :--- | :--- |
| server.port | 8441 | DBS port |
| info.app.notifyServers | http://52.77.xxx.xxx | relay server url |
| info.app.notifyPort | 14127 | EPS port |
| info.app.procIndex | 1 | DBS serial number |
| spring.datasource.main.url | jdbc:sqlite:sync.db | datasource |

If no application.properties found, copy config/example/application.properties.SQLITE as config/config.properties. And then modify it as needed. 

{% hint style="danger" %}
Please do not change values of items not described here.
{% endhint %}

#### Starting DBS

Run the command script according to your OS.

{% hint style="warning" %}
* DBS started by dbs\_windows\_console.cmd will stop if the command window is closed.
* DBS started by dbs\_windows\_agent.cmd will not stop even if the command window is closed.
{% endhint %}

### ☑ Setting EPG \(EventPushGateway\)

#### Files under EPG

| File | Description |
| :--- | :--- |
| example/config.properties | Sample config.properties |
| license/key | License file |
| config.properties | Configuration file |
| TP\_EPG.jar | Executable file |
| epg\_linux.sh | Command script for Linux bash |
| epg\_unix.sh | Command script for Unix |
| epg\_windows.agent.cmd | Command script for Windows \(daemon mode\) |
| epg\_windows.console.cmd | Command script for Windows \(console mode\) |

#### Setting config.properties

| Item | Value | Description |
| :--- | :--- | :--- |
| local.host.name | blank/url | Localhost by default |
| local.port | 14127 | EPS port |
| local.epg.mode | EPS | Uppercase |
| local.eps.index | 1 | EPS serial number |
| db.server.info.1 | [http://127.0.0.1:8441](http://127.0.0.1:8441) | DBS url and port |
| epg.ssl.enable | false/true | set to true to use SSL certificate |
| epg.ssl.keyStore | pfs file path | PKCS12 SSL certificate file path |
| epg.ssl.keyStorePassword |  | Certificate password |

If no config.properties found, copy example/config.properties to EPG and modify it.

#### Starting EPG server

Run command script according to your OS.

{% hint style="warning" %}
* EPG server started by epg\_windows\_console.cmd will stop if the command window is closed.
* EPG server started by epg\_windows\_agent.cmd will not stop even if the command window is closed.
{% endhint %}

## Installing OZ Mirroring Viewer

### ☑ Download and extract files

1. Download the sample application [here](https://drive.google.com/open?id=1jvHkLztyJCUGqxaoneZi4iMcq0dxVUG_).
2. Included are two folders, **ActiveXviewer** and **mirror** \(mirroring modules and sample application\), and a file **mirror.ozr** \(sample e-Form\). 
3. Move the folder **ActiveXviewer** and **mirror** to OZ servlet folder on your server.

   `tomcat/webapps/oz-servlet/` 

4. Move **mirror.ozr** to your OZ Server repository .`WEB-INF/repository_files/samples/`

### ☑ Modify files

1. Go to `oz-servlet/mirror/`. 
2. Modify mirror-start.html, mirror-desktop-jsp and mirror-mobile.html for your own environments. You will need to change the oz server url, category name of .OZR file, EPG port, DBS port, etc.

### ☑ Test installation

1. With **IE browser**, open the mirror-start.html like: _`http://hostname:port/oz-servlet/mirror/mirror-start.html`_
2. Enter "test" for device ID.
3. OZ viewer will be installed on your desktop at the very first time. It will take a while.
4. If the sample e-Form opens up, server settings are successful.

{% hint style="warning" %}
Make sure that you open mirror-start.html with **Internet Explorer**.
{% endhint %}

## Setting OZ Mobile App

Refer to the section **Preparing OZ Mobile App in** [Demo]() and add your own Homepage URL pointing to _`http://hostname:port/oz-servlet/mirror` ._ 

Now you can start mirroring with your server. Refer to [Demo]() for details.

## Scripts for e-Form

You need to add some scripts in your .OZR file for mirroring. Open your .OZR and copy the scripts below and paste it into OnExternalEvent of the ReportTemplate. And then save and upload it to OZ Repository Server.

```javascript
if(ozarg_1 == "setvalue") {
	var old = This.GetInputValue(ozarg_2);
	if(old != ozarg_3){
		This.SetInputValue(ozarg_2, ozarg_3);
		var comp = This.GetInputComponent(ozarg_2);
		if(comp) {
			comp.TriggerEvent("OnValueChanged");
		}
	}
} else if(ozarg_1 == "getvalue") {
	return This.GetInputValue(ozarg_2);
} else if(ozarg_1 == "setcomment") {
	var page = This.GetPageByIndex(parseInt(ozarg_2));
	page.SetCommentData(ozarg_3);
} else if(ozarg_1 == "getcomment") {
	var page = This.GetPageByIndex(parseInt(ozarg_2));
	if (ozarg_3 == "encode")
		return encodeURI(page.GetCommentData());
	else
		return page.GetCommentData();
} else if(ozarg_1 == "clearcomment") {
	for(var i = 1; i <= This.GetPageCount();i++){
		var page = This.GetPageByIndex(i);
		page.SetCommentData("");
	}
} else if(ozarg_1 == "focus") {
	This.GetInputComponent(ozarg_2).SetFocus(false);
} else if(ozarg_1 == "killfocus") {
	This.GetInputComponent(ozarg_2).KillFocus(false);
}


else if(ozarg_1 == "setkeyboardtype"){
    var obj = GetInputComponent(ozarg_2);
    obj.SetKeyboardType(ozarg_3);
}else if(ozarg_1 == "enable"){
    var obj = GetInputComponent(ozarg_2);
    obj.SetEnable(true);
}else if(ozarg_1 == "disable"){
    var obj = GetInputComponent(ozarg_2);
    obj.SetEnable(false);
}else if(ozarg_1 == "click"){
    var obj = GetInputComponent(ozarg_2);
    obj.Click();
}

var formid = "";
var objComp = "";
if(ozarg_1 != "" && ozarg_1 != null){
	if(ozarg_1 == "_callExternalEvent_"){
		//_MessageBox("ozarg_1 : "+ozarg_1+", ozarg_2 : "+ozarg_2+", ozarg_3 : "+ozarg_3);
		formid = ozarg_2;
		objComp = GetInputComponent(formid);
		objComp.SetText(ozarg_3);
	}else if(ozarg_1 == "_callUserEvent_"){
		formid = ozarg_2;
		objComp = GetInputComponent(formid);
		objComp.Click();
	}
} 
```

