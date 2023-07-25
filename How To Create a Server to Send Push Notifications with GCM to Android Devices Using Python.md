# How To Create a Server to Send Push Notifications with GCM to Android Devices Using Python

```Python``` ```Ubuntu``` ```Messaging```

# Introduction


Push notifications let your Android application notify a user of an event, even when the user is not using your app. The goal of this tutorial is to send a simple push notification to your app. We’ll use Ubuntu 14.04 and Python 2.7 on the server, and Google Cloud Messaging as the push notification service.


We’ll use the term server to refer to the instance spun up with DigitalOcean. We’ll use GCM to refer to Google’s server, the one that is between the Android device and your server.


## Prerequisites


You’ll need these things before you start the tutorial:


- An Android application; see developer.android.com
- A Ubuntu 14.04 Droplet
- Your Droplet’s IP address

## About Push Notifications


Google-provided GCM Connection Servers take messages from a third-party application server, such as your Droplet, and send these messages to a GCM-enabled Android application (the client app) running on a device. Currently, Google provides connection servers for HTTP and XMPP.





In other words, you need your own server to communicate with Google’s server in order to send the notifications. Your server sends a message to a GCM (Google Cloud Messaging) Connection Server, then the connection server queues and stores the message, and then sends it to the Android device when the device is online.


# Step One — Create a Google API Project


We need to create a Google API project to enable GCM for our app.


Visit the Google Developers Console.


If you’ve never created a developer account there, you may need to fill out a few details.


Click Create Project.


Enter a project name, then click Create.





Wait a few seconds for the new project to be created. Then, view your Project ID and Project Number on the upper left of the project page.





Make a note of the Project Number. You’ll use it in your Android app client.


# Step Two - Enable GCM for Your Project


Make sure your project is still selected in the Google Developers Console.


In the sidebar on the left, select APIs & auth.


Choose APIs.


In the displayed list of APIs, turn the Google Cloud Messaging for Android toggle to ON. Accept the terms of service.


Google Cloud Messaging for Android should now be in the list of enabled APIs for this project.





In the sidebar on the left, select APIs & auth.


Choose Credentials.


Under Public API access, click Create new Key.


Choose Server key.


Enter your server’s IP address.





Click Create.


Copy the API KEY. You’ll need to enter this on your server later.





# Step Three — Link Android App


To test the notifications, we need to link our Android app to the Google API project that we made.


If you are new to Android app development, you may want to follow the official guide for Implementing GCM Client.


You can get the official source code from the gcm page.


Note that the sources are not updates, so you’ll have to modify the Gradle file:


gcm-client/GcmClient/build.gradle


Old line:


```
compile "com.google.android.gms:play-services:4.0.+"

```


Updated line:


```
compile "com.google.android.gms:play-services:5.0.89+"

```


In the main activity, locate this line:


```
String SENDER_ID = "YOUR_PROJECT_NUMBER_HERE";

```


Replace this with the Project Number from your Google API project.


Each time a device registers to GCM it receives a registration ID. We will need this registration ID in order to test the server. To get it easily, just modify these lines in the main file:


```
            if (regid.isEmpty()) {
                registerInBackground();
            }else{
                Log.e("==========================","=========================");
                Log.e("regid",regid);
                Log.e("==========================","=========================");
            }

```


After you run the app, look in the logcat and copy your regid so you have it for later. It will look like this:


```
=======================================
10-04 17:21:07.102    7550-7550/com.pushnotificationsapp.app E/==========================﹕ APA91bHDRCRNIGHpOfxivgwQt6ZFK3isuW4aTUOFwMI9qJ6MGDpC3MlOWHtEoe8k6PAKo0H_g2gXhETDO1dDKKxgP5LGulZQxTeNZSwva7tsIL3pvfNksgl0wu1xGbHyQxp2CexeZDKEzvugwyB5hywqvT1-UJY0KNqpL4EUXTWOm0RxccxpMk
10-04 17:21:07.102    7550-7550/com.pushnotificationsapp.app E/==========================﹕ =======================================

```


# Step Four — Deploy a Droplet


Deploy a fresh Ubuntu 14.04 server. We need this to be our third-party application server.


Google’s GCM Connection Servers take messages from a third-party application server (our Droplet) and send them to applications on Android devices. While Google provides Connection Servers for HTTP and CCS (XMPP), we’re focusing on HTTP for this tutorial. The HTTP server is downstream only: cloud-to-device. This means you can only send messages from the server to the devices.


Roles of our server:


- Communicates with your client
- Fires off properly formatted requests to the GCM server
- Handles requests and resends them as needed, using exponential back-off
- Stores the API key and client registration IDs. The API key is included in the header of POST requests that send messages
- Generates message IDs to uniquely identify each message it sends. Message IDs should be unique per sender ID

The client will communicate with your server by sending the registration ID of the device for you to store it and use it when you send the notification. Don’t worry now about managing it; it’s very simple and GCM provides you with help by giving you error messages in case a registration ID is invalid.


# Step Five - Set Up Python GCM Simple Server


Log in to your server with a sudo user.


Update your package lists:


```
sudo apt-get update

```


Install the Python packages:


```
sudo apt-get install python-pip python-dev build-essential

```


Install python-gcm. Find out more about python-gcm here.


```
sudo pip install python-gcm

```


Create a new Python file somewhere on the server. Let’s say:


```
sudo nano ~/test_push.py

```


Add the following information to the file. Replace the variables marked in red. The explanation is below.


```
from gcm import *

gcm = GCM("AIzaSyDejSxmynqJzzBdyrCS-IqMhp0BxiGWL1M")
data = {'the_message': 'You have x new friends', 'param2': 'value2'}

reg_id = 'APA91bHDRCRNIGHpOfxivgwQt6ZFK3isuW4aTUOFwMI9qJ6MGDpC3MlOWHtEoe8k6PAKo0H_g2gXhETDO1dDKKxgP5LGulZQxTeNZSwva7tsIL3pvfNksgl0wu1xGbHyQxp2CexeZDKEzvugwyB5hywqvT1-UxxxqpL4EUXTWOm0RXE5CrpMk'

gcm.plaintext_request(registration_id=reg_id, data=data)

```


Explanation:


- from gcm import *: this imports the Python client for Google Cloud Messaging for Android
- gcm: add your API KEY from the Google API project; make sure your server’s IP address is in the allowed IPs
- reg_id: add your regid from your Android application

# Step Six — Send a Push Notification


Run this command to send a test notification to your app:


```
sudo python ~/test_push.py

```


Wait about 10 seconds. You should get a notification on your Android device.





## Troubleshooting.


If the notification does not appear on your device after about 10 seconds, follow these steps:


- Is your smartphone/tablet connected to the internet?
- Do you have the correct project key?
- Do you have the correct regid from the app?
- Is your server’s IP address added for the Google API server key?
- Is the server connected to the internet?

If you’re still not getting the notification, it’s probably the app. Check the logcat for some errors.


# Where to Go from Here


Once you’ve done this simple test, you’ll probably want to send the notifications to all your users. Remember that you have to send them in sets of 1000. Also, if the CGM responds with “invalid ID,” you must remove it from your database.


You can adapt the examples in this tutorial to work with your own Android application.


