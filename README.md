# Android4QEMU
How to setup and run Android Emulator without Android Studio

## Credits 
The credits should go to the authors of these publications that I used to create this file:
- https://linuxhint.com/setup-android-emulator-without-installing-android-studio-in-linux/
- https://dev.to/koscheyscrag/how-to-install-android-emulator-without-installing-android-studio-3lce


# Table of Contents
- [Android4QEMU](#android4qemu)
  - [Credits](#credits)
- [Table of Contents](#table-of-contents)
  - [Preperation (Linux and Windows)](#preperation-linux-and-windows)
  - [Setup Linux OS envoironment](#setup-linux-os-envoironment)
  - [Setup Windows OS envoironment](#setup-windows-os-envoironment)
  - [Commands to create an Android Virtual Device (AVD)](#commands-to-create-an-android-virtual-device-avd)
    - [Install Required Packages](#install-required-packages)
    - [Select the correct System Image to use](#select-the-correct-system-image-to-use)
    - [Download and install the selected system-image](#download-and-install-the-selected-system-image)
    - [Create a new AVD](#create-a-new-avd)
    - [Run the AVD](#run-the-avd)
    - [Install apps](#install-apps)
  - [Forensics analysis of Android apps](#forensics-analysis-of-android-apps)
    - [Important dirs](#important-dirs)
    - [Extract data](#extract-data)


## Preperation (Linux and Windows)

1. Download the latest version of [Android Command Line Tools](https://developer.android.com/studio#downloads) for your Operating System (OS) (scroll down to the command line section).
2. Extract the downloaded archive and make a new folder named “tools” inside “cmdline-tools” directory. Copy and paste all files from “cmdline-tools” folder to “tools” folder. Your final directory layout should look like this:

```
$ANDROID_SDK_ROOT/cmdline-tools/tools 
.
├── bin
├── lib
├── NOTICE.txt
└── source.properties
```

`$ANDROID_SDK_ROOT` is any directory here you want to install the files.

## Setup Linux OS envoironment

Install `adb` tools:
```
sudo apt install adb
```


## Setup Windows OS envoironment 
Under construction ...

## Commands to create an Android Virtual Device (AVD)
These commands are the same for both Linux and Windows

### Install Required Packages

Go to the `$ANDROID_SDK_ROOT/cmdline-tools/tools/bin` folder and run the following command to update repository details:
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
For the best performance choose a system-image for the `x86_64` architecture.

**Notes:** 
- If you need root access to the folders inside emulator **don't select** a system-image with `_playstore`
- A system-image without `_playstore` won't have access to the Google Play Store, to install apps go to a website, like https://www.apkmirror.com/ and download the `APK` files you want to install


### Download and install the selected system-image 
Download the packages using the same API level number you selected in the step above. Foe example:
```
./sdkmanager "platforms;android-30" "system-images;android-30;google_apis;x86_64" "build-tools;30.0.3"
```

### Create a new AVD

“Android Virtual Device” (AVD) is a set of configuration parameters that defines values for a virtual device that will emulate a real Android hardware device.

o create a new AVD, you need to use the system image you downloaded in the step above. Run the following command to create a new AVD, for example:
```
./avdmanager create avd -n "AFD2_API_30" -k "system-images;android-30;google_apis;x86_64"
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

Note the path of AVD in the output above. At the same path, you can find a `config.ini` file that can be used to change configuration parameters of the AVD.

Edit the file `config.ini` and change this value to `yes`:

```
hw.keyboard=yes

```

You can create as many as AVDs as you want and each AVD / System Image will be treated separately.

### Run the AVD

Go to `$ANDROID_SDK_ROOT/emulator` and run:
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

## Forensics analysis of Android apps

### Important dirs
Public data -- data that is available even on non-rooted devices:
```
adb shell
generic_x86_64_arm64:/ $ cd /sdcard/Android/data/<app dir>
```

Private data -- data that is only available withh root (notice the change from `$` to `#` in the prompt):
```
adb shell
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
