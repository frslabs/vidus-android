# VIDUS ANDROID SDK
![version](https://img.shields.io/badge/version-v3.1.2-blue)

The Vidus SDK comes with a set of screens and configurations to record live video of customers. Each of the recording options in the SDK are called nodes which can be configured by developers.

You can find the release history at [Changelog](CHANGELOG.md)

‼ ATTENTION ‼ → BREAKING CHANGE introduced at Vidus SDK v3.1.0. We have introduced a new license format. If you are using versions prior to v3.1.0 and intend to update to v3.1.0 or above, contact support@frslabs.com for an updated license.

# Table Of Content

- [Prerequisite](#prerequisite)
- [Android SDK Requirements](#android-sdk-requirements)
- [Download](#download)
  - [Using maven repository](#using-maven-repository)
- [Setup](#setup)
  - [Permissions](#permissions)
- [Quick Start](#quick-start)
- [Vidus Result](#vidus-result)
- [Vidus Error Codes](#vidus-error-codes)
- [Help](#help)

## Prerequisite

***NOTE : Encryption of VIDUS SDK Result is under development***

You will need a valid license to use the Vidus SDK, which can be obtained by contacting `support@frslabs.com` . 

Depending on the license - offline or online - you have opted for, the ping functionality to billing servers will be disabled or enabled. For instance, if you have opted for the offline SDK model, then there will be no server ping needed to our billing server to bill you. However, if you have chosen a transaction based pricing, then after each transaction, a ping request will be made to our billing server. This cannot be overrided by the App. A point to note is that if the ping transaction fails for any reason, the whole transaction will be void without any results from the SDK.

Once you have the license , follow the below instructions for a successful integration of Vidus SDK onto your Android Application.

## Android SDK Requirements

**Minimum SDK Version** -  **21** or higher

## Download

#### Using maven repository

Add the following code to your `project` level `build.gradle` file

```groovy
allprojects { 
    repositories { 

        maven { 
            // URL for Vidus SDK. 
            url "https://vidus-android.repo.frslabs.space/"                  
            credentials { 
                username '<YOUR_USERNAME>' 
                password '<YOUR_PASSOWRD>' 
            }
        }
       
       /*
        *Include below code only for transaction based billing
        */
        //Maven credentials for the Torus SDK
        maven {
            url "https://torus-android.repo.frslabs.space/"
            credentials {
                username '<YOUR_USERNAME>'
                password '<YOUR_PASSOWRD>'
            }
        }
        
    }
}
```

After that, add the following code to your `app` level `build.gradle` file

```groovy
// ...

defaultConfig { 

    // ...

    vectorDrawables.useSupportLibrary true 
    
}

// ...
```

And then, add the dependencies
```groovy

// ...

dependencies {

    /* Dependencies for Vidus SDK */ 
    implementation 'androidx.appcompat:appcompat:<lastest verison>'
    
    implementation 'androidx.constraintlayout:constraintlayout:<lastest verison>'
    
    implementation 'androidx.legacy:legacy-support-v4:<lastest verison>'
    
    implementation 'com.google.android.material:material:<lastest verison>'
       
    // Vidus Core Dependency
    implementation 'com.frslabs.android.sdk:vidus:3.1.0'
    
    // OPTIONAL - Required if transaction based billing is enabled
    // Vidus billing dependencies
    implementation 'com.frslabs.android.sdk:torus:1.1.0'
    
    implementation 'androidx.multidex:multidex:2.0.1'
    
    implementation 'com.google.code.gson:gson:2.8.5'
   
}
```

## Setup

#### Permissions

Vidus requires the following permission to operate properly

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android" package="your.package.name" >

    <!-- Required by Vidus -->
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.RECORD_AUDIO" />
  
    <!-- Optional - Required if transaction based billing is enabled -->
    <uses-permission android:name="android.permission.INTERNET" />  
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
  
    <uses-feature android:name="android.hardware.camera" android:required="true" />
    <uses-feature android:name="android.hardware.camera.front" android:required="false" />
    <uses-feature android:name="android.hardware.camera.autofocus" />
    <uses-feature android:name="android.hardware.microphone" />

    <application>
      ...
    </application>

</manifest>
```
## Quick Start

#### Initiating the Vidus Sdk

Initialize the `Vidus` instance with the appropriate configurations to invoke the Vidus Sdk

Here, only the `SimpleRecorderNode` is being invoked 

```java
public class MainActivity extends AppCompatActivity implements VidusResultCallback {

    // ...

    /* Enter the Vidus license key here */
    private String VIDUS_LICENSE_KEY = "ENTER_YOUR_LICENSE_KEY_HERE";

    /* Node ID - required to identify the node in the result */
    int SIMPLE_NODE_1 = 100; 
    
    /* (OPTIONAL)  Enter the Vidus api credentials here */
    private String VIDUS_API_BASE_URL = "ENTER_BASE_URL_HERE"
            , VIDUS_API_REFERENCE_ID = "ENTER_REF_ID_HERE"
            , VIDUS_API_CRED1 = "ENTER_API_CRED1_HERE"
            , VIDUS_API_CRED2 = "ENTER_API_CRED2_HERE";
    
    /* (OPTIONAL)  Enter the Vidus encryption key here */
    private String VIDUS_ENCRYPTION_KEY = "ENTER_ENCRYPTION_KEY_HERE";
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button callSdk = findViewById(R.id.call_sdk);
        callSdk.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                /* Invoke the Vidus Sdk */
                callVidusSdk();
            }
        });
    }

    public void callVidusSdk() {

        //Initialize the Vidus Node object with the appropriate nodes and parameters
        VidusNode vidusNode = new VidusNodeBuilder()
                .addNode(SIMPLE_NODE_1, new SimpleRecorderNode().setVideoRecordTime(5))
                .build();

        //Build the Vidus Sdk Config object with the appropriate configurations
        VidusConfig vidusConfig = new VidusConfig.Builder()
                .setLicenseKey(VIDUS_LICENSE_KEY)
                .setShowInstruction(false) 
                //If the user wants preivew enabled to true
                .showPreview(false)
                .setEncryptionKey(VIDUS_ENCRYPTION_KEY)
                .setVidusNode(vidusNode)
                .setApiCredentials(new VidusApiCredentials(VIDUS_API_BASE_URL
                        , VIDUS_API_REFERENCE_ID
                        , VIDUS_API_CRED1
                        , VIDUS_API_CRED2))
                .build();

        //Call the Vidus Sdk 
        Vidus vidus = new Vidus(vidusConfig);
        vidus.start(this, this);
        
    }

    // ...

}
```

#### Handling the result

Your activity must implement `VidusResultCallback` to receive the result.

```java
    // ...

    @Override
    public void onVidusSuccess(VidusResult vidusResult) {

        String videoPath = vidusResult.getVideoPath();
        String referenceId = vidusResult.getReferenceId(); //Optional
        
        SimpleRecorderNodeResult simpleRecorderNode = vidusResult.getSimpleRecorderNodeById(SIMPLE_NODE_1);

        /* Handle the Vidus Sdk result here */
        Log.d(TAG, "onVidusSuccess: VideoPath:"+videoPath+" NodeResult:"+simpleRecorderNode.toString());

    }

    @Override
    public void onVidusFailure(int errorCode) {
    
        /* Handle the Vidus Sdk failure result here */
        Log.d(TAG, "onVidusFailure: "+errorCode);
    }

    // ...
```

#### Invoking all the nodes

For details of the available nodes refer [Vidus Parameters](#vidus-parameters)

The nodes will be processed in the order in which they are added to `VidusNode`
Initialise `VidusNode` to include all the nodes as given below

```java

        //...
        
        VidusNode vidusNode = new VidusNodeBuilder()
                .addNode(SIMPLE_NODE, new SimpleRecorderNode()
                        .setVideoRecordTime(5))

                .addNode(CHALLENGE_CODE_NODE, new ChallengeCodeNode()
                        .setVideoChallengeCodeText("Please tell secret number")
                        .setVideoChallengeCodeVoiceType(VidusUtility.VOICE_TYPE_FEMALE))

                .addNode(CHALLENGE_TEXT_NODE, new ChallengeTextNode()
                        .setVideoChallengeText("Please tell your date of birth")
                        .setVideoChallengeTextTime(5)
                        .setVideoChallengeTextVoiceType(VidusUtility.VOICE_TYPE_FEMALE))

                .addNode(DECLARATION_NODE, new DeclarationNode()
                        .setVideoDeclarationText("PLACE_TEXT_HERE")
                        .setVideoDeclarationSpokenMethod(VidusUtility.SPOKEN_BY_MACHINE)
                        //Lesser the value greater the text scroll speed (user can specify from 10.0 to 50.0)
                        .setShowFaceOverlay(true) //OPTIONAL: default value true
                        .setVideoTextSpeed(35.0)//35.0f
                        .setVideoDeclarationVoiceType(VidusUtility.VOICE_TYPE_FEMALE))

                .addNode(OSV_RECORDER_NODE, new OSVRecorderNode()
                        .setVideoRecordTime(5))

                .addNode(OSV_TEXT_RECORDER_NODE, new OSVChallengeTextNode()
                        .setVideoChallengeText("Please show you PAN card")
                        .setVideoChallengeTextTime(8)
                        .setVideoChallengeTextVoiceType(VidusUtility.VOICE_TYPE_FEMALE))
                        
                .addNode(PIV_NODE, new PIVNode()
                        .setVideoChallengeText("Enter your question here. Prompt to confirm.")
                        .setVideoChallengeTextVoiceType(VidusUtility.VOICE_TYPE_FEMALE)
                        .setVideoChallengeTextToSpeak("पाठ को अंग्रेजी में फॉलबैक के रूप में बोला जाए।")
                        .setFallbackVideoChallengeTextToSpeak("Enter text to be spoken as fallback in english. Prompt to confirm. ")
                        .setLanguage("hi_IN")
                        .setPositiveButtonText("Yes")
                        .setNegativeButtonText("No"))                                                             .addNode(PIVC_NODE,com.frslabs.android.sdk.vidus.utils.FileUtils.getPIVCNode(com.frslabs.android.sdk.vidus.utils.FileUtils.readFromfileInputStream(this.getResources().getAssets().open("en_US")), templateJsonString, "en_US")) # Template JsonString 
                .build();

        // ...

```

For handling the result of all the nodes, refer the code below 

```java

    // ...
    
    @Override
    public void onVidusSuccess(VidusResult vidusResult) {

        String videoPath = vidusResult.getVideoPath();
        String referenceId = vidusResult.getReferenceId(); //Optional
        String imagePath = vidusResult.getImagePath();// Optional


        SimpleRecorderNodeResult simpleRecorderNodeResult = vidusResult.getSimpleRecorderNodeById(SIMPLE_NODE_1);
        ChallengeCodeNodeResult challengeCodeNodeResult = vidusResult.getChallengeCodeNodeById(CHALLENGE_CODE_NODE);
        ChallengeTextNodeResult challengeTextNodeResult = vidusResult.getChallengeTextNodeById(CHALLENGE_TEXT_NODE);
        DeclarationNodeResult declarationNodeResult = vidusResult.getDeclarationNodeById(DECLARATION_NODE);
        OSVRecorderNodeResult osvRecorderNodeResult = vidusResult.getOSVRecorderNodeById(OSV_RECORDER_NODE);
        OSVChallengeTextNodeResult osvChallengeTextNodeResult =               vidusResult.getOSVChallengeTextNodeById(OSV_TEXT_RECORDER_NODE);
        PIVNodeResult pivNodeResult = vidusResult.getPIVNodeById(PIV_NODE);
        PIVCNodeResult pivcNodeResult = vidusResult.getPIVCNodeById(PIVC_NODE);

        /* Handle the Vidus Sdk result here */
        Log.i(TAG, "onActivityResult: VideoPath:" + videoPath);
        Log.i(TAG, "onActivityResult: " + simpleRecorderNodeResult.toString());
        Log.i(TAG, "onActivityResult: " + challengeCodeNodeResult.toString());
        Log.i(TAG, "onActivityResult: " + challengeTextNodeResult.toString());
        Log.i(TAG, "onActivityResult: " + declarationNodeResult.toString());
        Log.i(TAG, "onActivityResult: " + osvRecorderNodeResult.toString());
        Log.i(TAG, "onActivityResult: " + osvChallengeTextNodeResult.toString());
        Log.i(TAG, "onActivityResult: " + pivNodeResult.toString());
        Log.i(TAG, "onActivityResult: " + pivcNodeResult.toString());


    }

    @Override
    public void onVidusFailure(int errorCode) {

        /* Handle the Vidus Sdk failure result here */
        Log.d(TAG, "onVidusFailure: "+errorCode);

    }
    
    // ...
    
```

## Vidus Result
The result is obtained through the `VidusResult` object , which has the results of each nodes in their respective result classes .

Given below are the Result classes in brief.

##### VidusResult

<div>
<table style="width:100%">
 <tr>
 <th bgcolor="#F1F1F1" colspan="3">Public Methods</th>
 </tr>
 <tr>
 <td>String</td>
 <td>getVideoPath()</td>
 <td>Gets the path of video which was recorded</td>
 </tr>
 <tr>
 <td>BaseNodeResult</td>
 <td>getNodeById()</td>
 <td>Gets the exact node result by Id. (Once retrieved should be casted to appropriate node class)</td>
 </tr>
 <tr>
 <td>String</td>
 <td>getReferenceId()</td>
 <td>Gets the Reference Id</td>
 </tr>
 <tr>
 <td>SimpleRecorderNodeResult</td>
 <td>getSimpleRecorderNodeById()</td>
 <td>Gets the exact SimpleRecorderNode result by Id</td>
 </tr>
 <tr>
 <td>ChallengeCodeNodeResult</td>
 <td>getChallengeCodeNodeById()</td>
 <td>Gets the exact ChallengeCodeNode result by Id</td>
 </tr>
 <tr>
 <td>ChallengeTextNodeResult</td>
 <td>getChallengeTextNodeById()</td>
 <td>Gets the exact ChallengeTextNode result by Id</td>
 </tr>
 <tr>
 <td>DeclarationNodeResult</td>
 <td>getDeclarationNodeById()</td>
 <td>Gets the exact DeclarationNode result by Id</td>
 </tr>
 <tr>
 <td>OSVRecorderNodeResult</td>
 <td>getOSVRecorderNodeById()</td>
 <td>Gets the exact OSVRecorderNode result by Id</td>
 </tr>
 <tr>
 <td>OSVChallengeTextNodeResult</td>
 <td>getOSVChallengeTextNodeById()</td>
 <td>Gets the exact OSVChallengeTextNode result by Id</td>
 </tr>
 <tr>
 <td>PIVNodeResult</td>
 <td>getPIVNodeById()</td>
 <td>Gets the exact PIVNode result by Id</td>
 </tr>
   <tr>
 <td>PIVCNodeResult</td>
 <td>getPIVCNodeById()</td>
 <td>Gets the exact PIVCNode result by Id</td>
 </tr>
</table>
</div>


Each of the individual Node result classes are briefed below 

##### SimpleRecorderNodeResult

<div>
<table style="width:100%">
 <tr>
 <th bgcolor="#F1F1F1" colspan="3">Public Methods</th>
 </tr>
 <tr>
 <td>int</td>
 <td>getStartTime()</td>
 <td>Gets the time(<em>seconds</em>) when the node started</td>
 </tr>
 <tr>
 <td>int</td>
 <td>getStopTime()</td>
 <td>Gets the time(<em>seconds</em>) when the node stopped</td>
 </tr>
</table>
</div>

##### ChallengeCodeNodeResult

<div>
<table style="width:100%">
 <tr>
 <th bgcolor="#F1F1F1" colspan="3">Public Methods</th>
 </tr>
 <tr>
 <td>int</td>
 <td>getStartTime()</td>
 <td>Gets the time(<em>seconds</em>) when the node started</td>
 </tr>
 <tr>
 <td>int</td>
 <td>getStopTime()</td>
 <td>Gets the time(<em>seconds</em>) when the node stopped</td>
 </tr>
 <tr>
 <td>int</td>
 <td>getCodeRecordStartTime()</td>
 <td>Gets the time(<em>seconds</em>) when the user-spoken code starts to get recorded</td>
 </tr>
 <tr>
 <td>String</td>
 <td>getVideoChallengeCodeText()</td>
 <td>Gets the prompt text which was displayed</td>
 </tr>
 <tr>
 <td>String</td>
 <td>getVideoChallengeCode()</td>
 <td>Gets the code which was prompted to be spoken</td>
 </tr>
  
 <tr>
 <td>boolean</td>
 <td>isCodeVerified()</td>
 <td>Returns whether the code spoken matches with the challenge code displayed</td>
 </tr>
 
</table>
</div>

##### ChallengeTextNodeResult

<div>
<table style="width:100%">
 <tr>
 <th bgcolor="#F1F1F1" colspan="3">Public Methods</th>
 </tr>
 <tr>
 <td>int</td>
 <td>getStartTime()</td>
 <td>Gets the time(<em>seconds</em>) when the node started</td>
 </tr>
 <tr>
 <td>int</td>
 <td>getStopTime()</td>
 <td>Gets the time(<em>seconds</em>) when the node stopped</td>
 </tr>
 <tr>
 <td>int</td>
 <td>getTextRecordStartTime()</td>
 <td>Gets the time(<em>seconds</em>) when the user-spoken code starts to get recorded</td>
 </tr>
 <tr>
 <td>String</td>
 <td>getVideoChallengeText()</td>
 <td>Gets the prompt text which was displayed</td>
 </tr>
</table>
</div>

##### DeclarationNodeResult

<div>
<table style="width:100%">
 <tr>
 <th bgcolor="#F1F1F1" colspan="3">Public Methods</th>
 </tr>
 <tr>
 <td>int</td>
 <td>getStartTime()</td>
 <td>Gets the time(<em>seconds</em>) when the node started</td>
 </tr>
 <tr>
 <td>int</td>
 <td>getStopTime()</td>
 <td>Gets the time(<em>seconds</em>) when the node stopped</td>
 </tr>
 <tr>
 <td>String</td>
 <td>getVideoDeclarationText()</td>
 <td>Gets the text which was displayed</td>
 </tr>
 <tr>
 <td>int</td>
 <td>getVideoDeclarationSpokenMethod()</td>
 <td>Gets the spoken method type</td>
 </tr>
</table>
</div>

##### OSVRecorderNodeResult

<div>
<table style="width:100%">
<tr>
<th bgcolor="#F1F1F1" colspan="3">Public Methods</th>
</tr>
<tr>
<td>int</td>
<td>getStartTime()</td>
<td>Gets the time(<em>seconds</em>) when the node started</td>
</tr>
<tr>
<td>int</td>
<td>getStopTime()</td>
<td>Gets the time(<em>seconds</em>) when the node stopped</td>
</tr>
</table>
</div>

##### OSVChallengeTextNodeResult

<div>
<table style="width:100%">
<tr>
<th bgcolor="#F1F1F1" colspan="3">Public Methods</th>
</tr>
<tr>
<td>int</td>
<td>getStartTime()</td>
<td>Gets the time(<em>seconds</em>) when the node started</td>
</tr>
<tr>
<td>int</td>
<td>getStopTime()</td>
<td>Gets the time(<em>seconds</em>) when the node stopped</td>
</tr>
<tr>
<td>int</td>
<td>getTextRecordStartTime()</td>
<td> Gets the time(seconds) when the user-spoken code starts to get recorded</td>
</tr>
<tr>
<td>String</td>
<td>getVideoChallengeText()</td>
<td>Gets the prompt text which was displayed</td>
</tr>
</table>
</div>


##### PIVNodeResult

<div>
<table style="width:100%">
<tr>
<th bgcolor="#F1F1F1" colspan="3">Public Methods</th>
</tr>
  
<tr>
<td>int</td>
<td>getStartTime()</td>
<td>Gets the time(<em>seconds</em>) when the node started</td>
</tr>

<tr>
<td>int</td>
<td>getStopTime()</td>
<td>Gets the time(<em>seconds</em>) when the node stopped</td>
</tr>
  
<tr>
<td>String</td>
<td>getVideoChallengeQuestion()</td>
<td>Gets the question which was displayed</td>
</tr>

<tr>
<td>String</td>
<td>getVideoChallengeAnswer()</td>
<td>Gets the value of button clicked</td>
</tr>

<tr>
<td>String</td>
<td>getVideoChallengeTextToSpeak()</td>
<td>Gets the text which was spoken</td>
</tr>

<tr>
<td>String</td>
<td>getFallbackVideoChallengeTextToSpeak()</td>
<td>Gets the fallback text (English) which was spoken. (Fallback text is spoken only if speaking <b><i>VideoChallengeText</i></b> in a different language failed)</td>
</tr>

</table>
</div>

##### PIVCNodeResult

<div>
<table style="width:100%">
<tr>
<th bgcolor="#F1F1F1" colspan="3">Public Methods</th>
</tr>
  
<tr>
<td>int</td>
<td>getStartTime()</td>
<td>Gets the time(<em>seconds</em>) when the node started</td>
</tr>

<tr>
<td>int</td>
<td>getStopTime()</td>
<td>Gets the time(<em>seconds</em>) when the node stopped</td>
</tr>
  
<tr>
<td>String</td>
<td>getVideoChallengeQuestion()</td>
<td>Gets the question which was displayed</td>
</tr>

<tr>
<td>String</td>
<td>getVideoChallengeAnswer()</td>
<td>Gets the value of button clicked</td>
</tr>

<tr>
<td>String</td>
<td>getVideoChallengeTextToSpeak()</td>
<td>Gets the text which was spoken</td>
</tr>

<tr>
<td>String</td>
<td>getFallbackVideoChallengeTextToSpeak()</td>
<td>Gets the fallback text (English) which was spoken. (Fallback text is spoken only if speaking <b><i>VideoChallengeText</i></b> in a different language failed)</td>
</tr>

<tr>
<td>String</td>
<td>getDisplayText()</td>
<td>Gets display question text</td>
</tr>

</table>
</div>


## Vidus Error Codes

Following error codes will be returned on the `onVidusFailure` method of the callback

| CODE | DESCRIPTION                  |
| ---- | ---------------------------- |
| 803  | Permission denied             |
| 804  | SDK was interrupted       |
| 805  | Vidus SDK License expired            |
| 806  | Vidus SDK License was invalid |
| 807  | Invalid Config         |
| 808  | Transaction Failed       |
| 809  | No Internet Available             |
| 810  | Timeout Error             |
| 811  | Network Error |
| 812  | Camera Error  |

## Vidus Parameters

The Vidus SDK has APIs to capture interactive realtime selfie video with customizable actions. Each functionality is separated into unique `nodes`. The APIs are contained in the following  input nodes

1. **[Simple Recorder Node](#simple-recorder-node)**

2. **[Challenge Code Node](#challenge-code-node)**

3. **[Challenge Text Node](#challenge-text-node)**

4. **[Declaration Node](#declaration-node)** 

5. **[OSV Recorder Node](#osv-recorder-node)** 

6. **[OSV Challenge Text Node](#osv-challenge-text-node)**

7. **[PIV Node](#piv-node)** (Pre-Issuance Verification Node)

7. **[PIVC Node](#pivc-node)**


The Input Nodes are explained below,

#### Simple Recorder Node

Captures a video recording with a user defined time

<div>
<table style="width:100%">
  <tr>
    <th bgcolor="#F1F1F1" colspan="2">Public Methods</th>
  </tr>
  <tr>
    <td><b>setVideoRecordTime</b>(<em>int videoRecordTime</em>)</td>
    <td>Sets the time(<em>seconds</em>) taken for the recording of the node.
    </br></br> Values should be between <b> 1 and 100 </b> </td>
  </tr>
</table>
</div>

#### Challenge Code Node

Captures a video recording which will include a user-read 4 digit code prompted by a customizable machine-read question

<div>
<table style="width:100%">
  <tr>
    <th bgcolor="#F1F1F1" colspan="2">Public Methods</th>
  </tr>
  <tr>
    <td><b>setVideoChallengeCodeText</b>(<em>String videoChallengeCodeText</em>)</td>
    <td>Sets the text that will be spoken prior to displaying the code.</td>
  </tr>
  <tr>
 <td><b>setVideoChallengeCodeVoiceType</b>(<em>int videoChallengeCodeVoiceType</em>)</td>
 <td>(OPTIONAL) </br>Sets the voice type for the machine-read text. </br> 
  </br> Values are </br> <b>VideoSdkUtility.VOICE_TYPE_MALE </br> VideoSdkUtility.VOICE_TYPE_FEMALE </b>(Default Value) </td>
 </tr>
</table>
</div>

#### Challenge Text Node

Captures a video recording which will include a user-prompted answer with a customizable machine-read question

<div>
<table style="width:100%">
 <tr>
 <th bgcolor="#F1F1F1" colspan="2">Public Methods</th>
 </tr>
 <tr>
 <td><b>setVideoChallengeText</b>(<em>String videoChallengeText</em>)</td>
 <td>Sets the text that will be spoken.</td>
 </tr>
 <tr>
 <td><b>setVideoChallengeTextTime</b>(<em>int videoChallengeTextTime</em>)</td>
 <td>Sets the recording time(<em>seconds</em>) for the node after the text is spoken.
 </br></br> Values should be between <b> 1 and 100 </b> </td>
 </tr>
 <tr>
 <td><b>setVideoChallengeTextVoiceType</b>(<em>int videoChallengeTextVoiceType</em>)</td>
 <td>(OPTIONAL) </br>Sets the voice type for the machine-read text. </br>
 </br> Values are </br> <b>VideoSdkUtility.VOICE_TYPE_MALE </br> VideoSdkUtility.VOICE_TYPE_FEMALE </b>(Default Value) </td>
 </tr>
</table>
</div>

#### Declaration Node

Captures a video recording which will include a user-read or machine-read text/declaration

<div>
<table style="width:100%">
 <tr>
 <th bgcolor="#F1F1F1" colspan="2">Public Methods</th>
 </tr>
 <tr>
 <td><b>setVideoDeclarationText</b>(<em>String videoDeclarationText</em>)</td>
 <td>Sets the text that will be displayed.</td>
 </tr>
 <tr>
 <td><b>setVideoDeclarationVoiceType</b>(<em>int videoDeclarationVoiceType</em>)</td>
 <td>(OPTIONAL) </br> Sets the voice type for the machine-read text.
 </br></br> Values are </br> <b>VideoSdkUtility.VOICE_TYPE_MALE </br> VideoSdkUtility.VOICE_TYPE_FEMALE</b>(Default Value) </td>
 </tr>
 <tr>
 <td><b>setVideoTextSpeed</b>(<em>int VideoTextSpeed</em>)</td>
 <td>(OPTIONAL) </br> Lesser the value greater the text scroll speed.
 </br></br> Values are </br> <b>10.0 </br> 60.0</b>(Default Value is 35.0) </td>
 </tr>
 <tr>
 <td><b>setShowFaceOverlay</b>(<em>boolean showFaceOverlay</em>)</td>
 <td>(OPTIONAL) </br> Show or hide face overlay.
</br></br> Values are </br>  <b>true or false</b>(Default Value is true) </td>
 </tr>
 <tr>
 <td><b>setVideoDeclarationSpokenMethod</b>(<em>int videoDeclarationSpokenMethod</em>)</td>
 <td>(OPTIONAL) </br> Sets the way the text is to be spoken, 
 </br> Either by the user or by the machine.</br>
 </br> Values are </br> <b>VideoSdkUtility.SPOKEN_BY_MACHINE </br> VideoSdkUtility.SPOKEN_BY_USER </b>(Default Value) </td>
 </tr>
</table>
</div>

#### OSV Recorder Node

Captures a video recording with a user defined time

<div>
<table style="width:100%">
<tr>
<th bgcolor="#F1F1F1" colspan="2">Public Methods</th>
</tr>
<tr>
<td><b>setVideoRecordTime</b>(<em>int videoRecordTime</em>)</td>
<td>Sets the time(<em>seconds</em>) taken for the recording of the node.
</br></br> Values should be between <b> 1 and 100 </b> </td>
</tr>
</table>
</div>

#### OSV Challenge Text Node

<div>
<table style="width:100%">
<tr>
<th bgcolor="#F1F1F1" colspan="2">Public Methods</th>
</tr>
  
<tr>
<td><b>setVideoRecordTime</b>(<em>int videoRecordTime</em>)</td>
<td>Sets the text that will be spoken.
</tr>

<tr>
<td><b>setVideoRecordTime</b>(<em>int videoRecordTime</em>)</td>
<td>Sets the time(<em>seconds</em>) taken for the recording of the node.
</br></br> Values should be between <b> 1 and 100 </b> </td>
</tr>

<tr>
<td><b>setVideoChallengeTextVoiceType</b>(<em>int videoChallengeTextVoiceType</em>)</td>
<td>(OPTIONAL) </br> Sets the voice type for the machine-read text.
 </br></br> Values are </br> <b>VideoSdkUtility.VOICE_TYPE_MALE </br> VideoSdkUtility.VOICE_TYPE_FEMALE</b>(Default Value) </td>
</tr>

</table>
</div>

#### PIV Node

<div>
<table style="width:100%">
<tr>
<th bgcolor="#F1F1F1" colspan="2">Public Methods</th>
</tr>
<tr>
<td><b>setVideoChallengeText</b>(<em>String videoChallengeText</em>)</td>
<td>Sets the text that will be displayed.
</tr>

<tr>
<td><b>setVideoChallengeTextTime</b>(<em>int videoChallengeTextTime</em>)</td>
<td>Sets the time(<em>seconds</em>) taken for the recording of the node.
</br></br> Values should be between <b> 1 and 100 </b> </td>
</tr>

<tr>
<td><b>setVideoChallengeTextToSpeak</b>(<em>String videoChallengeTextToSpeak</em>)</td>
<td>Sets the text(In English) that will be spoken. In PIV node, clients should ensure that currencies are written in full for the speech to text function to work correctly. For example 2Cr should be written as 2 Crores. And currency symbols should be written in full. For example, Rs should be written as Rupees. Any number over 6 digits will be treated as a monetary number when spoken by the machine. </td>
</tr>

<tr>
<td><b>setFallbackVideoChallengeTextToSpeak</b>(<em>String fallbackVideoChallengeTextToSpeak</em>)</td>
<td>Sets the text (Vernacular) that will be spoken, with a necessary fallback text (In English) that will be spoken if speaking the former text failed. In PIV node, clients should ensure that currencies are written in full for the speech to text function to work correctly. For example 2Cr should be written as 2 Crores. And currency symbols should be written in full. For example, Rs should be written as Rupees. Any number over 6 digits will be treated as a monetary number when spoken by the machine.</td>
</tr>

<tr>
<td><b>setLanguage</b>(<em>String language</em>)</td>
<td>Sets the language in which text set in <b><i>setVideoChallengeTextToSpeak</i></b> has to be spoken</td>
</tr>

<tr>
<td><b>setPositiveButtonText</b>(<em>String positiveButtonText</em>)</td>
<td>Sets the text to be shown in the Positive action button.
</tr>

<tr>
<td><b>setNegativeButtonText</b>(<em>String negativeButtonText</em>)</td>
<td>Sets the text to be shown in the Negative action button.
</tr>

<tr>
<td><b>setVideoChallengeTextVoiceType</b>(<em>int videoChallengeTextVoiceType</em>)</td>
<td>(OPTIONAL) </br> Sets the voice type for the machine-read text.
</br></br> Values are </br> <b>VideoSdkUtility.VOICE_TYPE_MALE </br> VideoSdkUtility.VOICE_TYPE_FEMALE</b>(Default Value) </td>
</tr>

</table>
</div>

#### PIVC Node

<div>
<table style="width:100%">
<tr>
<th bgcolor="#F1F1F1" colspan="2">Public Methods</th>
</tr>
<tr>
<td><b>setVideoChallengeText</b>(<em>String videoChallengeText</em>)</td>
<td>Sets the text that will be displayed.
</tr>

<tr>
<td><b>setVideoChallengeTextTime</b>(<em>int videoChallengeTextTime</em>)</td>
<td>Sets the time(<em>seconds</em>) taken for the recording of the node.
</br></br> Values should be between <b> 1 and 100 </b> </td>
</tr>

<tr>
<td><b>setVideoChallengeTextToSpeak</b>(<em>String videoChallengeTextToSpeak</em>)</td>
<td>Sets the text(In English) that will be spoken. In PIV node, clients should ensure that currencies are written in full for the speech to text function to work correctly. For example 2Cr should be written as 2 Crores. And currency symbols should be written in full. For example, Rs should be written as Rupees. Any number over 6 digits will be treated as a monetary number when spoken by the machine. </td>
</tr>

<tr>
<td><b>setFallbackVideoChallengeTextToSpeak</b>(<em>String fallbackVideoChallengeTextToSpeak</em>)</td>
<td>Sets the text (Vernacular) that will be spoken, with a necessary fallback text (In English) that will be spoken if speaking the former text failed. In PIV node, clients should ensure that currencies are written in full for the speech to text function to work correctly. For example 2Cr should be written as 2 Crores. And currency symbols should be written in full. For example, Rs should be written as Rupees. Any number over 6 digits will be treated as a monetary number when spoken by the machine.</td>
</tr>

<tr>
<td><b>setLanguage</b>(<em>String language</em>)</td>
<td>Sets the language in which text set in <b><i>setVideoChallengeTextToSpeak</i></b> has to be spoken</td>
</tr>

<tr>
<td><b>setPositiveButtonText</b>(<em>String positiveButtonText</em>)</td>
<td>Sets the text to be shown in the Positive action button.
</tr>

<tr>
<td><b>setNegativeButtonText</b>(<em>String negativeButtonText</em>)</td>
<td>Sets the text to be shown in the Negative action button.
</tr>

<tr>
<td><b>setVideoChallengeTextVoiceType</b>(<em>int videoChallengeTextVoiceType</em>)</td>
<td>(OPTIONAL) </br> Sets the voice type for the machine-read text.
</br></br> Values are </br> <b>VideoSdkUtility.VOICE_TYPE_MALE </br> VideoSdkUtility.VOICE_TYPE_FEMALE</b>(Default Value) </td>
</tr>

<tr>
<td><b>setDisplayText</b>(<em>String displayText</em>)</td>
<td>Sets the text that will be displayed. </td>
</tr>


</table>
</div>

## Help
For any queries/feedback , contact us at `support@frslabs.com` 
