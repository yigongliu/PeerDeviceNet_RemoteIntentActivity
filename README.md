PeerDeviceNet_RemoteIntentActivity
==================================

This sample demonstrates how to send remote intents thru PeerDeviceNet - "intenting" API.

The sample can be divided into three parts performing the following functions:

	* allow user interactively define the intents to be sent (actions, URIs)
	* send the defined intents to remote peers
	* receive remote intents sent by peers and do its work.
	
1. Define intents interactively.

	The sample allows choosing intent action from a drop down spinner:
 
		* ACTION_VIEW
		* ACTION_EDIT
		* ACTION_CALL
		* ACTION_DIAL

	It allows defining intent data uris in two ways:

		* enter uri text directly into "data uri" text box
		* pick a photo or video from gallery and use its content uri 
			
2. Send intents to remote peers.

	PeerDeviceNet provides both "intenting" and idl APIs for sending remote intents. 
	Here we'll use intenting api.
	Internally PeerDeviceNet will stream the data content pointed to by intent uri to remote
	peers, so that it can be viewed or edited over there.
	
	Code changes:
	
	2.1. add the following permission in AndroidManifest to allow sending remote intents:

		<uses-permission android:name="com.xconns.peerdevicenet.permission.REMOTE_INTENT" />
	
	2.2. "Send" button onClick() callback function collect remote intent information and 
		send to peers

	2.2.1. collect intent information from GUI widgets. 

	   Please note the special translation of intent actions such as ACTION_CALL and ACTION_DIAL which requires system permissions. The detailed explanation are in next section.

	2.2.2. create an "carrier" intent.

	   The carrier intent is defined with 

		action = "com.xconns.peerdevicenet.START_REMOTE_ACTIVITY"

	   All other data related to remote intents are packed as extended data items:

		* "REMOTE_INTENT": the remote intent to send are packed as a Parcelable object.
		* "PEER_NAME"/"PEER_ADDR"/"PEER_PORT": optional information about destination peer device, all are packed as strings.
		* "PACKAGE_NAME": optional package name; at destination device, if no apps
			can handle the delivered intent, PeerDeviceNet will prompt user to install app with this name.
	   			
	Please note that if no destination device information is defined, PeerDeviceNet
	   		GUI will show the list of connected peer devices to allow user choose which
	   		devices to send to.
	   		
3. Receive remote intents.

	Some intent actions (such as ACTION_VIEW) does not require any special permissions,
		PeerDeviceNet will dispatch them directly to installed apps.

	Some intent actions (such as ACTION_DIAL) require system permissions. PeerDeviceNet 
		only pass messages between peer devices on behalf of apps, it will not have all
		these permissions. So this kind of intents are handled as following:

	3.1. create a custom intent action for it
			in our sample, we defined:

		"com.xconns.samples.ACTION_REMOTE_CALL" for ACTION_CALL
		"com.xconns.samples.ACTION_REMOTE_DIAL" for ACTION_DIAL

	3.2. at sending app, we translate this kind of intents into custom intent before
			packing it into "carrier" intents.

	3.3. at destination device, we must have a "receiving" app with proper system 
			permissons. This receiving app will register for this custom intent;
			receive this custom intents and translate them back into proper intents.
			Since the receiving app has the proper permissions, it can dispatch the
			translated intents without problem.
			
	In our sample, SendIntentsActivity is both the sending app and receiving app. 
	For receiving custom intents, it will add the following in AndroidManifest:

		<action android:name="com.xconns.samples.ACTION_REMOTE_CALL" />
		<action android:name="com.xconns.samples.ACTION_REMOTE_DIAL" />
		
	In onCreate()/onNewIntent() methods, it will call processRemoteCallDial() to handle
		these custom intents.
		
		
	
		