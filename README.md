# Android4QEMU
To run Android on our PCs there are several options, being [Android Studio](https://developer.android.com/studio) one of them. However, Android Studio was created for Android developers and has a complex GUI with lots of features. Therefore, consumes lots of resources even without running the embedded emulator. To test apps and the files they generate, we don't need the Android Studio GUI.

This page explains how to set up and run the Android Studio Emulator **without** the Android Studio GUI.


# Table of Contents
- [Android4QEMU](#android4qemu)
- [Table of Contents](#table-of-contents)
  - [Credits](#credits)
  - [1. Preperation](#1-preperation)
  - [2. Setup Linux OS envoironment](#2-setup-linux-os-envoironment)
  - [3. Setup Windows OS envoironment](#3-setup-windows-os-envoironment)
  - [4. Commands to create an Android Virtual Device (AVD)](#4-commands-to-create-an-android-virtual-device-avd)
    - [Install Required Packages](#install-required-packages)
    - [Select the correct System Image to use](#select-the-correct-system-image-to-use)
    - [Download and install the selected system-image](#download-and-install-the-selected-system-image)
    - [Create a new AVD](#create-a-new-avd)
    - [Run the AVD](#run-the-avd)
    - [Install apps](#install-apps)
  - [5. Android apps data location](#5-android-apps-data-location)
    - [Important dirs](#important-dirs)
    - [Extract data](#extract-data)

## Credits 
This page is heavely based on:
- https://linuxhint.com/setup-android-emulator-without-installing-android-studio-in-linux/

Therefore, credits should go to its authors.



## 1. Preperation

1. Download the latest version of [Android Command Line Tools](https://developer.android.com/studio#downloads) for your Operating System (OS) (scroll down to the command line section).
   
2. Extract the downloaded archive and make a new folder named “tools” inside “cmdline-tools” directory. Copy and paste all files from “cmdline-tools” folder to “tools” folder. Your final directory layout should look like this:

```
$ANDROID_HOME/cmdline-tools/tools 
.
├── bin
├── lib
├── NOTICE.txt
└── source.properties
```

`$ANDROID_HOME` is any directory here you want to install the files.

## 2. Setup Linux OS envoironment

`$ANDROID_HOME` can be `/opt/Android`

Install `adb` tools:
```
sudo apt install adb
```


## 3. Setup Windows OS envoironment 

1. Follow the steps listed here: https://dev.to/koscheyscrag/how-to-install-android-emulator-without-installing-android-studio-3lce until step `11.`

2. After step `11.` follow the steps from [4. Commands to create an Android Virtual Device (AVD)](#commands-to-create-an-android-virtual-device-avd)
  


## 4. Commands to create an Android Virtual Device (AVD)
These commands are the same for both Linux and Windows, however:
   - remove the `./` before each command
   - change the Linux `/` to the Windows `\`, except on topic [5. Forensics analysis of Android apps](#5-forensics-analysis-of-android-apps)

### Install Required Packages

Go to the `$ANDROID_HOME/cmdline-tools/tools/bin` folder and run the following command to update repository details:
```
./sdkmanager
```

Install packages required for the Android emulator to work:
```
./sdkmanager platform-tools emulator
```

### Select the correct System Image to use
Next you need to select a system image to load in the Android emulator. 
To get a list of latest downloadable system images (API version 30), run the command:
```
./sdkmanager --list | grep "system-images;android-30"

system-images;android-30;google_apis;x86_64 | 10      | Google APIs Intel x86 Atom_64 System Image | system-images/android-30/google_apis/x86_64/
  system-images;android-30;google_apis;x86                                                 | 9            | Google APIs Intel x86 Atom System Image                             
  system-images;android-30;google_apis;x86_64                                              | 10           | Google APIs Intel x86 Atom_64 System Image                          
  system-images;android-30;google_apis_playstore;x86                                       | 9            | Google Play Intel x86 Atom System Image                             
  system-images;android-30;google_apis_playstore;x86_64                                    | 10           | Google Play Intel x86 Atom_64 System Image
```

This [page](https://source.android.com/setup/start/build-numbers#platform-code-names-versions-api-levels-and-ndk-releases) has a list of all API numbers.

For the best performance choose a system-image for the `x86_64` architecture.

**Notes:** 
- If you need root access to the folders inside emulator **don't select** a system-image with `_playstore`
- A system-image without `_playstore` won't have access to the Google Play Store, to install apps go to a website, like https://www.apkmirror.com/ and download the `APK` files you want to install


### Download and install the selected system-image 
Download the packages using the same API level number you selected in the step above. For example:
```
./sdkmanager "platforms;android-30" "system-images;android-30;google_apis;x86_64" "build-tools;30.0.3"
```

### Create a new AVD

“Android Virtual Device” (AVD) is a set of configuration parameters that defines values for a virtual device that will emulate a real Android hardware device.

o create a new AVD, you need to use the system image you downloaded in the step above. Run the following command to create a new AVD, for example:
```
$ ./avdmanager create avd -n "AFD2_API_30" -k "system-images;android-30;google_apis;x86_64"
```

Confirm that the AVD has been successfully created using the command below:
```
./avdmanager list avd
Available Android Virtual Devices:
    Name: AFD2_API_30
    Path: /home/user/.android/avd/AFD2_API_30.avd
  Target: Google APIs (Google Inc.)
          Based on: Android 11.0 (R) Tag/ABI: google_apis/x86_64
  Sdcard: 512 MB
```

Note the path of AVD in the output above. At the same path, you can find a `config.ini` file that can be used to change configuration parameters of the AVD. Edit the file `config.ini` and change the value to `yes`:

```
hw.keyboard=yes
```

**Note** 
  - If you don't do this change, the Android buttons (home, back, and overview) won't work and you won't be able to operate the Android running in the emulator.


### Run the AVD

Go to `$ANDROID_HOME/emulator` and run:
```
./emulator -avd "AFD2_API_30"
```

### Install apps
A system-image without `_playstore` won't have access to the Google Play Store. So, to install apps you need to go to a website, like https://www.apkmirror.com/ and download the `APK` file of the app you want to install.

Use the `adb` commands to connect to the emulator:
```
$ adb devices
List of devices attached
emulator-5554   device
```

Then, inside the directory where you downloaded the APK file use `adb install <file>.apk`, for example:
```
$ adb install com.google.android.apps.authenticator2_5.10.apk
Success
```

## 5. Android apps data location

### Important dirs
Public data -- data that is available even on non-rooted devices:
```
$ adb shell
generic_x86_64_arm64:/ $ cd /sdcard/Android/data/<app dir>
```

Private data -- data that is only available withh root (notice the change from `$` to `#` in the prompt):
```
$ adb shell
generic_x86_64_arm64:/ $ su
generic_x86_64_arm64:/ # cd /data/data/<app dir>
```

### Extract data

1. Connect to the Android emulator and follow the steps bellow to create a `tgz` file with the contents of the private directory af an app:
```
$ adb shell
generic_x86_64_arm64:/ $ su
generic_x86_64_arm64:/ # cd /data/data/
generic_x86_64_arm64:/ # tar -cvzf /sdcard/Download/<compressed filename>.tgz <app folder>/
generic_x86_64_arm64:/ # exit
generic_x86_64_arm64:/ $ exit
```

2. Copy the `tgz` file into your computer for analysis
```
$ adb pull /sdcard/Download/<compressed filename>.tgz
/sdcard/Download/<compressed filename>.tgz: 1 file pulled. 0.1 MB/s (180 bytes in 0.010s)
```

3. Decompress the file with `7z` and start your analysis
