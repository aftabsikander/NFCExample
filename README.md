# NFC
NFC typically requiring a distance of 4cm or less to initiate a connection. NFC allows you to share small payloads of data between an NFC tag and an Android-powered device, or between two Android-powered devices.

The data stored in the tag can also be written in a variety of formats, but many of the Android framework APIs are based around a [NFC Forum](http://nfc-forum.org/) standard called NDEF (NFC Data Exchange Format)

There are some basic steps we have to follow before using the NFC.

####  NFC NDEF Structure
Here are several types of NFC (NDEF) tag:

 1. NFC Forum well-known type [NFC RTD] 
 2. Media-type as defined in RFC 2046 
 3. Absolute URI as defined in RFC 3986
 4. NFC Forum external type [NFC RTD]

Reading NDEF data from an NFC tag is handled with the [tag dispatch system](https://developer.android.com/guide/topics/connectivity/nfc/nfc.html#tag-dispatch), which analyzes discovered NFC tags, appropriately categorizes the data, and starts an application that is interested in the categorized data. An application that wants to handle the scanned NFC tag can declare an [intent filter](https://developer.android.com/guide/topics/connectivity/nfc/nfc.html#filtering-intents) and request to handle the data.

Android provides a special tag dispatch system that analyzes scanned NFC tags, parses them, and tries to locate applications that are interested in the scanned data. It does this by:

 1. Parsing the NFC tag and figuring out the MIME type or a URI that
    identifies the data payload in the tag.
    
 2. Encapsulating the MIME type or URI and the payload into an intent. These first two steps are described in [How NFC tags are mapped to MIME types and URIs](https://developer.android.com/guide/topics/connectivity/nfc/nfc.html#ndef)
 3. Starts an activity based on the intent. This is described in [How NFC Tags are Dispatched to Applications.](https://developer.android.com/guide/topics/connectivity/nfc/nfc.html#dispatching)

#### NDEF Details

 1. NDEF data is encapsulated inside a message [(NdefMessage)](https://developer.android.com/reference/android/nfc/NdefMessage.html)
    that contains one or more records [(NdefRecord)](https://developer.android.com/reference/android/nfc/NdefRecord.html)
    
 2. When device scans an NFC tag containing NDEF formatted data, it parses the message and tries to figure out the data's MIME type or identifying URI.
 3. system reads the first (NdefRecord) inside the (NdefMessage) to determine how to interpret the entire NDEF message (an NDEF message can have multiple NDEF records).

#### Android NFC Filter
The first thing we want is our app is notified when we get near a NFC tag. To this purpose we use a intent filter. Android SDK provides three different filter that we can use with different level of priority:

 1. ACTION_NDEF_DISCOVERED
 2. ACTION_TECH_DISCOVERED
 3. ACTION_TAG_DISCOVERED

##### What is ACTION_NDEF_DISCOVERED Filter
This intent is used to start an Activity when a tag that contains an NDEF payload is scanned and is of a recognized type. This is the highest priority intent, and the tag dispatch system tries to start an Activity with this intent before any other intent, whenever possible.

##### What is ACTION_TECH_DISCOVERED Filter
If no activities register to handle the ACTION_NDEF_DISCOVERED intent, the tag dispatch system tries to start an application with this intent. This intent is also directly started (without starting ACTION_NDEF_DISCOVERED first) if the tag that is scanned contains NDEF data that cannot be mapped to a MIME type or URI.

##### What is ACTION_TAG_DISCOVERED Filter
This intent is started if no activities handle the ACTION_NDEF_DISCOVERED or ACTION_TECH_DISCOVERED intents.

![Android NFC Filter Figure](https://developer.android.com/images/nfc_tag_dispatch.png)

##### Sample Code

**Example 1**

    <manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.aftabsikander.example.nfc" >
    <intent-filter>
	    <action android:name="android.nfc.action.NDEF_DISCOVERED"/>
	    <category android:name="android.intent.category.DEFAULT" />
	    <data android:mimeType="text/plain"/>
    </intent-filter>
    <manifest>

**Example 2**

    <intent-filter>
        <action android:name="android.nfc.action.NDEF_DISCOVERED"/>
        <category android:name="android.intent.category.DEFAULT"/>
       <data android:scheme="http"
                  android:host="developer.android.com"
                  android:pathPrefix="/index.html" />
    </intent-filter>


## Host Card Emulation  (HCE)
HCE allows an Android app to act like a Smart card. Before Android 4.4, doing this was generally not possible for developers, because it required installing a JavaCard applet in the Secure Element (SE) of the phone, which is something that normally only the device vendor can do. [more information on HCE](https://developer.android.com/guide/topics/connectivity/nfc/hce.html)

With HCE, an app can register an Android Service that can communicate with an NFC terminal through exchange of APDUs (Application Protocol Data Units). The service is identified by an AID (Application Id), a unique byte string that identifies an NFC application, similar to an IntentFilter. The NFC terminal initially queries the Android NFC stack if it supports the AID it expects. If Android finds a service supporting the AID, it will establish a link between the Android Service and the NFC terminal for the duration of the session.
After the link is set up, the Service and the terminal can exchange arbitrary data through APDUs (Application Protocol Data Units).

### Protocol Definition
The diagram below shows the flow of data between the phone, the NFC terminal and the HTTP server.
![diagram for NFC Terminal and HTTP Server](https://cdn-images-1.medium.com/max/800/1*6aJAqU-zqJ8Ww_jx6hWJrQ.png)
