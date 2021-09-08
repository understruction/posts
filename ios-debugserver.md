+++
author = "EvilPenguin"
title = "Settting up iOS Debugging"
date = "2021-01-03"
description = "Settting up iOS Debugging"
tags = [
    "debug", 
    "debugserver", 
    "lldb"
]
+++

Hi Nerds!

In this post, we will explain how to setup debugging on iOS through USB. Let's kick this off with the environment.

#### Environment

You need:
* macOS with the latest [`Xcode`](https://developer.apple.com/xcode)
* [`usbmuxd`](https://github.com/libimobiledevice/usbmuxd)
* [`brew`](https://brew.sh)
* [Jailbroken](https://canijailbreak.com/) Device with [`OpenSSH`](https://cydia.saurik.com/openssh.html) installed
* Connected Jailbroken Device via USB
* Terminal (We enjoy [iTerm2](https://iterm2.com/))

#### usbmuxd

We need to install `usbmuxd`. Run the following command in Terminal:

{{< highlight html >}}
$ brew install libusbmuxd
{{< /highlight >}}

After installation, we will run this command (used for the SSH session below):

{{< highlight html >}}
$ iproxy 2222 22 &
{{< /highlight >}}

Finally, run this command (used for the `debugserver` session below):

{{< highlight html >}}
$ iproxy 6666 6666 &
{{< /highlight >}}

#### Debugserver

We need to pull `debugserver` from Xcode. Run this command:

{{< highlight html >}}
$ ls /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/DeviceSupport/
{{< /highlight >}}

The output should be the available iOS versions:

{{< highlight html >}}
10.0 10.1 10.2 10.3 11.0 11.1 11.2 11.3 11.4 12.0 12.1 12.2 12.3 12.4 13.0 13.1 13.2 13.3 13.4 13.5 13.6 13.7 14.0 14.1 14.2 14.3 9.0 9.1 9.2 9.3
{{< /highlight >}}

Choose the iOS version for your device. For this example, we are going to work with 13.7 to extract `debugserver`.

{{< highlight html >}}
$ hdiutil attach /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/DeviceSupport/13.7/DeveloperDiskImage.dmg$ cp /Volumes/DeveloperDiskImage/usr/bin/debugserver .$ hdiutil detach /Volumes/DeveloperDiskImage
{{< /highlight >}}

Now you should have `debugserver` in your current directory.

#### Sign Debugserver

Sign `debugserver` with the following command:

{{< highlight html >}}
$ codesign -s - --entitlements entitlements.plist -f debugserver
{{< /highlight >}}

where `entitlements.plist` is the following plist:

{{< highlight html >}}
<?xml version=\"1.0\" encoding=\"UTF-8\"?><!DOCTYPE plist PUBLIC \"-//Apple//DTD PLIST 1.0//EN\" \"http://www.apple.com/DTDs/PropertyList-1.0.dtd\">
<plist version=\"1.0\">
    <dict>
        <key>com.apple.backboardd.debugapplications</key>
        <true/>
        <key>com.apple.backboardd.launchapplications</key>
        <true/>
        <key>com.apple.diagnosticd.diagnostic</key>
        <true/>
        <key>com.apple.frontboard.debugapplications</key><true/>
        <key>com.apple.frontboard.launchapplications</key>
        <true/>
        <key>com.apple.security.network.client</key>
        <true/>
        <key>com.apple.security.network.server</key>
        <true/>
        <key>com.apple.springboard.debugapplications</key>
        <true/>
        <key>com.apple.system-task-ports</key>
        <true/>
        <key>get-task-allow</key>
        <true/>
        <key>platform-application</key>
        <true/>
        <key>run-unsigned-code</key>
        <true/>
        <key>task_for_pid-allow</key>
        <true/>
    </dict>
</plist>
{{< /highlight >}}

You can download the entitlements.plist [here](/uploads/entitlements.plist).

#### Copy Debugserver to the device

Since we started `usbmuxd` with `iproxy` above, we are simply going to `scp debugserver` to the device. Run this command:

{{< highlight html >}}
$ scp -P 2222 ./debugserver root@localhost:/usr/bin/
{{< /highlight >}}

#### Checking Debugserver on the device

First, we need to SSH into the iOS device. Run this command:

{{< highlight html >}}
$ ssh root@localhost -p 2222
{{< /highlight >}}

Next, let's verify that `debugserver` is installed by running this command:

{{< highlight html >}}
ls -la /usr/bin/ | grep -i debugserver
{{< /highlight >}}

The output should look something like this:

{{< highlight html >}}
-rwxr-xr-x   1 root wheel   9872352 Jan  2 18:32 debugserver*
{{< /highlight >}}

#### Start Debugging

Launch the "Settings" app on your iOS device. We should also validate the process has started by running this command:

{{< highlight html >}}
ps aux | grep -i Preferences
{{< /highlight >}}

The output should be similar to this:

{{< highlight html >}}
mobile         10115   0.0  3.1  4903072  63392   ??  Ss    5:47PM   0:01.11 /Applications/Preferences.app/Preferences
{{< /highlight >}}

At last! Lets attach `debugserver` to the `Preferences` process:

{{< highlight html >}}
debugserver localhost:6666 -a Preferences
{{< /highlight >}}

The output should be similar to the following:

{{< highlight html >}}
debugserver-@(#)PROGRAM:LLDB  PROJECT:lldb-10.0.0 for arm64.
Attaching to process Preferences...
Listening to port 6666 for a connection from localhost...
{{< /highlight >}}

Now, open a new Terminal window on macOS and run the following:

{{< highlight html >}}
$ lldb
(lldb) platform select remote-ios
(lldb) process connect connect://localhost:6666
{{< /highlight >}}

After some time, finally, it will connect and display something like this:

{{< highlight html >}}
Process 10115 stopped* thread #1, queue = 'com.apple.main-thread', stop reason = signal SIGSTOP    
    frame #0: 0x00000001ab104634 libsystem_kernel.dylib`mach_msg_trap + 8libsystem_kernel.dylib`mach_msg_trap:->  
        0x1ab104634 <+8>: retlibsystem_kernel.dylib`mach_msg_overwrite_trap:    
        0x1ab104638 <+0>: mov    x16, #-0x20    
        0x1ab10463c <+4>: svc    #0x80    
        0x1ab104640 <+8>: retTarget 0: 
(Preferences) stopped.
{{< /highlight >}}

#### The End

Well that wraps it up. Thanks for reading and we hope you enjoyed it. 
