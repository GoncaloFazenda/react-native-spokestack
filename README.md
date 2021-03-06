# react-native-spokestack

React Native wrapper for the [Spokestack](https://github.com/pylon/spokestack-android) speech activity detection/automated speech recognition project.

## Getting started

[![](https://img.shields.io/npm/v/react-native-spokestack.svg)](https://www.npmjs.com/package/react-native-spokestack)

`$ npm install react-native-spokestack --save`

- _Android_: Android SDK 26+
- _iOS_: iOS 11+

### (Mostly) automatic installation

#### Android

`$ react-native link react-native-spokestack`

#### iOS

[![](https://img.shields.io/cocoapods/v/RNSpokestack.svg)](https://cocoapods.org/pods/RNSpokestack)

#### Prerequistes

1. iOS 11+, Swift 4.2

#### Installation

1. install [CocoaPods](https://guides.cocoapods.org/using/using-cocoapods.html#adding-pods-to-an-xcode-project), v1.6.0+
2. `cd ios && pod init`
3. edit the resulting `Podfile` and add the following contents:

```
platform :ios, '11.0'

target 'YOUR_PROJECT' do
  use_frameworks!

  pod 'RNSpokestack', :path => '../node_modules/react-native-spokestack'
  pod 'yoga', path: '../node_modules/react-native/ReactCommon/yoga'
  pod 'DoubleConversion', :podspec => '../node_modules/react-native/third-party-podspecs/DoubleConversion.podspec'
  pod 'Folly', :podspec => '../node_modules/react-native/third-party-podspecs/Folly.podspec'
  pod 'glog', :podspec => '../node_modules/react-native/third-party-podspecs/glog.podspec'
  pod 'React', path: '../node_modules/react-native', subspecs: [
  'Core',
  'jschelpers',
  'cxxreact',
  'CxxBridge',
  'DevSupport',
  'RCTText',
  'RCTImage',
  'RCTLinkingIOS',
  'RCTNetwork',
  'RCTActionSheet',
  'RCTAnimation',
  'RCTWebSocket',
  ]
end

```

4. `pod install`
5. Remove all `lib*` files from **Link Binary with Libraries** under your project target in xCode. This prevents [dueling installations of React](https://sandstorm.de/de/blog/post/react-native-managing-native-dependencies-using-xcode-and-cocoapods.html).

> ...thus we need to ensure that they reference the same React Native library which you link to from the outer project.

#### RN 0.58+ notes

- In the `Podfile` remove `jschelpers` from the React subspec. ([reference](https://github.com/facebook/react-native/commit/f85692cf8fb19d1334998ea647a25953dc849eee#diff-66230b3e029caa37b0fbdc8cbd47f4ab))
- If using Rn 0.58.0 - 0.58.4, an additional header [path](https://github.com/amccarri/react-native/commit/4e18338365175c1e7cceb784e98bf540b991c190#diff-66230b3e029caa37b0fbdc8cbd47f4ab) needs to be added to the `jsiexecutor` subspec in `node_modules/react-native/React.podspec`. You may use [patch-package](https://www.npmjs.com/package/patch-package) as a solution. This issue is was addressed and fixed in RN 0.58.5

### Manual installation

#### Android

1. Open up `android/app/src/main/java/[...]/MainApplication.java` (could also be called `MainActivity.java` or similar)

- Add `import com.pylon.RNSpokestack.RNSpokestackPackage;` to the imports at the top of the file
- Add `new RNSpokestackPackage()` to the list returned by the `getPackages()` method

2. Append the following lines to `android/settings.gradle`:
   ```
   include ':react-native-spokestack'
   project(':react-native-spokestack').projectDir = new File(rootProject.projectDir, 	'../node_modules/react-native-spokestack/android')
   ```
3. Insert the following lines inside the dependencies block in `android/app/build.gradle`:
   ```
     implementation project(':react-native-spokestack')
   ```

#### iOS (not using CocoaPods)

- Currently only buildable on an `amd_64` target

- Drag the RNSpokestack.xcodeproj from the react-native-spokestack/ios folder to the Libraries group on Xcode in your poject.

- Click on your main project file (the one that represents the .xcodeproj) select Build Phases and drag the static library, libRNSpokestack.a, from the Libraries/RNSpokestack.xcodeproj/Products folder to Link Binary With Libraries

##### Link the necessary libraries:

- Project Build Phases

  - Link Binary with Libraries:

    - /node_modules/react-native-spokestack/Frameworks
    - AVFoundation
    - SpokeStack.framework

  - Copy Bundle Resources:
    - gRCPCertificate.bundle

- General
  - Always Embed Swift Standard Libraries: Yes
  - Embedded Binaries:
    - /node_modules/react-native-spokestack/Frameworks
    - SpokeStack.framework
    - Linked Frameworks and Binaries:
    - /node_modules/react-native-spokestack/Frameworks
    - SpokeStack.framework

### Android Support

#### `android/build.gradle`

Make sure the Google repo is listed first

```
buildcripts {
   ...
    repositories {
      google()
      ...
    }
}

allprojects{
      repositories {
        google()
       ...
    }
}
```

#### `android/app/build.gradle`

```

...
android {
...
packagingOptions {
exclude 'project.properties'
exclude 'META-INF/INDEX.LIST'
exclude 'META-INF/DEPENDENCIES'
}
}
...

```

#### `android/app/src/main/AndroidManifest.xml`

```

<uses-permission android:name="android.permission.RECORD_AUDIO" />
```

## Usage

### Javascript

```javascript
import Spokestack from "react-native-spokestack";

// initialize the Spokestack pipeline.
// The pipeline has three required top-level keys: 'input', 'stages', and 'properties'.
// For further examples, see https://github.com/pylon/spokestack-android#configuration
Spokestack.initialize({
  input: "com.pylon.spokestack.android.MicrophoneInput", // required, provides audio input into the stages
  stages: [
    "com.pylon.spokestack.webrtc.VoiceActivityDetector", // voice activity detection. necessary to trigger speech recognition.
    "com.pylon.spokestack.google.GoogleSpeechRecognizer" // one of the two supplied speech recognition services
    // 'com.pylon.spokestack.microsoft.BingSpeechRecognizer'
  ],
  properties: {
    locale: "en-US",
    "google-credentials": YOUR_GOOGLE_VOICE_CREDENTIALS, // Android-supported api
    "google-api-key": YOUR_GOOGLE_API_KEY, // iOS supported google api
    // 'bing-speech-api-key': YOUR_BING_VOICE_CREDENTIALS,
    "trace-level": Spokestack.TraceLevel.DEBUG
  }
});

// Start and stop the speech pipeline. All methods can be called repeatedly.
Spokestack.start(); // start speech pipeline. can only start after initialize is called.
Spokestack.stop(); // stop speech pipeline
Spokestack.activate(); // manually activate the speech pipeline. The speech pipeline is now actively listening for speech to recognize.
Spokestack.deactivate(); // manually deactivate the speech pipeline. The speech pipeline is now passively waiting for an activation trigger.

// Binding events
const logEvent = e => console.log(e);
Spokestack.onActivate = logEvent;
Spokestack.onDeactivate = logEvent;
Spokestack.onError = e => {
  Spokestack.stop();
  logEvent(e);
};
Spokestack.onTrace = e => { // subscribe to tracing events according to the trace-level property
  logEvent(e);
  console.log(e.message);
};
Spokestack.onRecognize = e => {
  logEvent(e);
  console.log(e.transcript); // "Hello Spokestack"
};

```

## API

### Methods

| Method Name                | Description                                                                     | OS |
| -------------------------- | ------------------------------------------------------------------------------- | -- |
| Spokestack.initialize()    | Initialize the speech pipeline; required for all other methods                        | Android, iOS |
| Spokestack.start()           | Starts the speech pipeline. The speech pipeline starts in the `deactivate` state. | Android, iOS |
| Spokestack.stop()          | Stops the speech pipeline                                                       | Android, iOS |
| Spokestack.activate()      | Manually activate the speech pipeline                                           | Android, iOS |
| Spokestack.deactivate()    | Manually deactivate the speech pipeline                                         | Android, iOS |

### Events

| Event Name                           | Property              | Description                                                                                                          | OS           |
| ------------------------------------ | --------              | ---------------------------------------                                                                              | --           |
| Spokestack.onActivate(event)         | `null`                | Invoked when the speech pipeline is activated, which enables the speech recognizer and begins a new dialogue session | Android, iOS |
| Spokestack.onDeactivate(event)       | `null`                | Invoked when the speech pipeline has been deactivated                                                                | Android, iOS |
| Spokestack.onStart(event)            | `null`                | Invoked when the speech pipeline is started                                                                          | Android, iOS |
| Spokestack.onStop(event)             | `null`                | Invoked when the speech pipeline has been stopped                                                                    | Android, iOS |
| Spokestack.onRecognize(event)        | `transcript`:`string` | Invoked when speech has been recognized                                                                              | Android, iOS |
| Spokestack.onTimeout(event)          | `null`                | Invoked when no speech has been detected for `wake-active-max` after activation                                      | Android, iOS |
| Spokestack.onTrace(event)            | `message`:`string`    | Invoked when a trace message become available                                                                        | Android      |
| Spokestack.onError(event)            | `error`:`string`      | Invoked upon an error in the speech pipeline execution                                                               | Android, iOS |

### Enums

| TraceLevel | Value |
| ---------- | ----- |
| DEBUG      | 10    |
| PERF       | 20    |
| INFO       | 30    |
| NONE       | 100   |

## Gotchas

### Android

- Requires Android SDK 26 level support
- Requires Gradle 3.0.1+ (`classpath 'com.android.tools.build:gradle:3.0.1'` in root `build.gradle` `dependencies`)
- Add app setting for microphone permission

### iOS

- Add app setting for microphone permission (`NSMicrophoneUsageDescription`) and speech recognition (`NSSpeechRecognitionUsageDescription`)
- Spokestack on iOS does not manage `AudioSession` settings. The client app is required to implement whatever `AudioSession` category and options are necessary. At minimum, the session category should allow for recording, eg `AVAudioSessionCategoryRecord` or `AVAudioSessionCategoryPlayAndRecord`. A simple `AudioSession` setting, suitable for insertion in `AppDelegate.m`, could be:
```
  [[AVAudioSession sharedInstance] setCategory:AVAudioSessionCategoryPlayAndRecord mode:AVAudioSessionModeDefault options:AVAudioSessionCategoryOptionDefaultToSpeaker  error:nil];
  [[AVAudioSession sharedInstance] setActive:YES error:nil];
```
## License

Copyright 2018 Pylon, Inc.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
