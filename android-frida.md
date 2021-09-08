+++
author = "Cale"
title = "Settting up Frida on Android"
date = "2021-01-10"
description = "Settting up Frida on Android"
tags = [
    "android", 
    "frida"
]
+++

Hello All!

In this post, we will explain how to setup Frida with Android through USB on MacOS. Let's first prepare your environment.

#### Environment
You need:

Rooted Android device or emulator, we like the Pixel 4a.
* [Android SDK Platform-Tools](https://developer.android.com/studio/releases/platform-tools)
* [brew](https://brew.sh)
* Python (latest 3.x recommended)
* [Frida](https://github.com/frida/frida/releases)

Note: This guide targets Android 9.0 and later.

#### Developer mode
Enable developer mode by locating the `Build Number` item in our system settings and tapping it 7 times.

This is found at:  `Settings > About Phone > Build Number`.

Once completed, you will see a `You are now a developer!` message.

#### USB debugging
Enable USB debugging at: `Settings > System > Advanced > Developer Options > USB debugging`

#### Setup Tools
Install Android SDK Platform-Tools This is easy, thanks to `brew`:
{{< highlight html >}}
$ brew install android-platform-tools
{{< /highlight >}}
After installation, we will test `adb` along with USB debugging on our Android device. If using a physical device, be sure that it's connected to your computer over USB. Run the following command (we will use this output to select the right Frida server binary below):

{{< highlight html >}}
$ adb shell uname -m
aarch64
{{< /highlight >}}
Success, we have `adb` and USB debugging functioning. Also, we confirmed this device is running AArch64/ARM64.

Using `adb shell <cmd>`, enables us to run a single command on our Android device and get output back. If we wanted to drop into a shell on our device, we would just run `adb shell`.

Common issues with `adb` can be resolved by stopping and starting the server:

{{< highlight html >}}
$ adb kill-server
$ adb start-server
{{< /highlight >}}

#### Setup Frida Server
Download the latest [Frida server](https://github.com/frida/frida/releases) binary matching your devices architecture. In this example for the Pixel 4a, we use Arm64.
{{< highlight html >}}
$ wget https://github.com/frida/frida/releases/download/15.1.1/frida-server-15.1.1-android-arm64.xz
{{< /highlight >}}
Extract and rename the binary to `frida-server`:

{{< highlight html >}}
$ unxz frida-server-14.2.3-android-arm64.xz$ mv frida-server-14.2.3-android-arm64 frida-server
{{< /highlight >}}

Upload `frida-server` to your Android device and mark it as executable:
{{< highlight html >}}
$ adb push frida-server /data/local/tmp/
$ adb shell "chmod 755 /data/local/tmp/frida-server"
{{< /highlight >}}

Get a shell, elevate to `root`, execute and background `frida-server`:
{{< highlight html >}}
$ adb shell
sunfish:/ $ su
sunfish:/ # id
uid=0(root) gid=0(root) groups=0(root) context=u:r:magisk:s0
sunfish:/ # /data/local/tmp/frida-server &
[1] 23244
{{< /highlight >}}

Verify `frida-server` is running as `root`:
{{< highlight html >}}
$ adb shell
2|sunfish:/ $ ps -e | grep  frida-server
root          23244      1 10852136 56048 0                   0 S frida-server
{{< /highlight >}}

Confirmed to be running as the `root` user, let's move on.

#### Setup Frida Tools
This can be accomplished with `pip`, the Python package installer. Frida prefers Python3 and I have `pip3` configured as the package installer for my Python3 environment.

{{< highlight html >}}
$ pip3 install frida-tools
{{< /highlight >}}
#### Frida Basics
Get list of running applications:
{{< highlight html >}}
$ frida-ps -Ua
{{< /highlight >}}

Should return something like:
{{< highlight html >}}
  PID  Name                      Identifier
-----  ------------------------  ------------------------------------------
22319   Firefox               org.mozilla.firefox
26648   Google                com.google.android.googlequicksearchbox
15871   Google Play Store     com.android.vending
{{< /highlight >}}

Attach to a process:
{{< highlight html >}}
$ frida -U org.mozilla.firefox
{{< /highlight >}}

Attach to a process and load Frida script:
{{< highlight html >}}
$ frida -U org.mozilla.firefox -l script.js
{{< /highlight >}}

#### That is all for now.
Thank you for reading and hope you enjoyed as an initial walkthrough or reference next time you need Frida up in a hurry. Have an issue or want to comment, tweet me.

All the best,

[--Cale](https://twitter.com/0xc413)