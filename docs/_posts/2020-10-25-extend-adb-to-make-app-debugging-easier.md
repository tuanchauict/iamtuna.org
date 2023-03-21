---
layout: post
title:  "Extend the ADB to Make App Debugging Easier"
author: "Tuna"
comments: false
category: Android
tags: android debugging adb
excerpt_separator: <!--more-->
sticky: false
hidden: false
draft: false
---

ADB is a useful and powerful tool to interact with an Android app or the whole Android device via the command line. However, ADB is still far away from a debug tool, therefore, usually, in a big app, we have to develop a UI tool called dev menu or something like that to interact, change the configuration, etc. We may use the other library like Facebook’s [Fillper](https://fbflipper.com/) or [Stetho](http://facebook.github.io/stetho/) or [Hyperion](https://github.com/willowtreeapps/Hyperion-Android), etc<!--more-->.

In this writing, I would like to extend ADB to make development life easier.

## Why do I choose `ADB` ?
Well, we, Android devs, are already familiar with ADB, at least for some regularly used commands. Besides, since we use the console (or terminal) to interact with the ADB, we can set an alias for long commands (if you’re like me, hate typing long command lines even with git branch name) to make it easier and faster to handle.

## Choose the entrance
On the computer side, we surely use the console, terminal, or some other command-line tools which support ADB.

From the app side, how can the app receive the information from the console? There’re many options to choose

### Service dump
```bash
adb shell dumpsys ...
```
To use service dump, we just need to override `dump()` method of the Service class
```kotlin
override fun dump(
    fd: FileDescriptor?, 
    writer: PrintWriter?, 
    args: Array<out String>?
)
```
The first advantage of this way is that we can get the result directly on the console. And for security concerns, we can check the caller information `Binder.getCallingPid()` whether it is `Process.SHELL_UID` (denotes for ADB call).

The drawbacks? We have to keep the service alive. Therefore, this approach is a little bit unstable since the service can be destroyed.

### Broadcast receiver
adb shell am broadcast ...
The most intuitive pro of using a broadcast receiver is that the receiver is always ready to handle our commands, even when the app is not running. The implementation is also simple, we can pass primary types like boolean, integer, etc. or array of those types into the command handler as the receiver will receive all arguments via an `Intent` which Android already parses for us.

The cons? The most concerning issue is security, the other app can freely trigger our app via a broadcast channel. Therefore, we should enable this for debug mode only.

Another drawback of the broadcast receiver in comparison to the service dump is we cannot easily get results in the console where the command is kicked. We have to check Logcat to see the print.

A small note for using the broadcast receive command:

> Apps that target Android 8.0 or higher can no longer register broadcast receivers for implicit broadcasts in their manifest. An implicit broadcast is a broadcast that does not target that app specifically. (_[Broadcast Limitations](https://developer.android.com/about/versions/oreo/background#broadcasts)_)

That means we need to put the target app on the ADB command

```bash
# Command 1
adb shell am broadcast -p <app.package.name> -a <broadcast.action>
```

### Other approaches and final decision
There are many other possible options for communication with the app via ADB. We can create an HTTP server inside the app, open a socket then, we can use telnet, or trigger update a file (see [How to monitor folder for file changes in background?](https://stackoverflow.com/questions/48034748/how-to-monitor-folder-for-file-changes-in-background)).

For debugging purposes, I choose Broadcast Receiver because of its readiness and simple to implement, we’re sure that our commands are always handled, no need to check whether the service is alive.

## Command handler approach
Now, after being able to receive the arguments from ADB, we need to create a handler for each command. Surely enough, we will have several commands to meet our debugging purpose. To handle all of those commands, we can create a receiver for each command. However, this will abuse the receiver system of our app, and each time we create a receiver, we have to copy and paste (or using a `BaseBroadcastReceiver` class) the pre-condition check like security.

Imagine the receiver as a server, it will receive all requests from the client (terminal) and each request is identified by the path. We can add an id to each command to denote the path then the receiver will delegate the request to the correspondent handler. To do this, we will add the first mandatory extra, let’s call it cmd

```kotlin
override fun onReceive(context: Context, intent: Intent) {
    val command = intent.getStringExtra("cmd") ?: return
    // snippet
}
```
Then, at the tail of command 1 above, we will add `-e cmd <commandKey>`

After having the command key, we are free to delegate the other handler classes. In [this sample](https://github.com/tuanchauict/AdbCommandExtensionDemo), I create a simple command handler that updates background color.

![Changing color via adb](/images/2020-10-25/adb-debugging-demo.gif)

```bash
adb shell am broadcast -p com.example.adbplugindemo \
    -a debug.adb.COMMAND \
    -e cmd changeBg \
    -e bgcolor red
```
For more information about passing data into intent through ADB, please check [this link](https://developer.android.com/studio/command-line/adb?hl=fr#IntentSpec).

Getting the result at the same place
As I mentioned, one con of using Broadcast Receiver is that we cannot get the print within the console. Here I made a simple script to reduce this disadvantage.

```bash
# command.sh
runTimeFlag=$(adb shell date +"%m-%d\ %T")

adb shell am broadcast -p app.package.name \
    -a receiver.action.COMMAND -e cmd $@
adb logcat -t "$runTimeFlag.000" -v tag -s TAG | while read LOGLINE
do
  echo $LOGLINE
done
```

Then we can use as
```bash
./command.sh commandKey ...
```

and the result printed on the logcat will be shown right on the console. However, I must say it does not 100% work well. If the log doesn’t come before the pipe closing, the script shows nothing (but we are still able to check on the standard logcat). We can fix this in another way to read logcat’s logs.

## Conclusion
Using ADB for debugging is really fun and saves a lot of time from browsing the dev menu. Besides, by using the Broadcast Receiver, we can also create a separate app serving as a dev menu.

