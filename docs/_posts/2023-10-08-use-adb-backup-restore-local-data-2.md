---
layout: post
title:  "Use ADB to backup and restore local data for testing 2"
author: "Tuna"
comments: false
category: Android
tags: android adb debugging bash
excerpt_separator: <!--more-->
sticky: false
hidden: false
draft: false
---

Three years ago, I wrote [a note about how to use `adb` to backup the test data](https://iamtuna.org/2020-11-07/use-adb-backup-and-restore-local-data-for-testing). At that time, or at the time when I created the scripts (before writing the note), we were still able to save files in the common spaces such as SDCard by using 
```bash
adb shell "run-as ..."
``` 
However, with the launch of Android 10, this is no longer possible. The command under `run-as` only able to save file within the app's directory.
```
Permission denied
```
I was amazed by how Android Studio was able to implement the *Save* and *Upload* functions within the Device Explorer tool. Despite my extensive searching, I couldn't find any clues on how the IDE implemented those features.

I had almost given up on my search until recently when I had to figure out how to do this again due to a bug related to the Release Candidate build. The RC build is signed with a different keystore from the development build, so testing the RC build required removing the app or cleaning up the previous backups. I used the Device Explorer to backup the backup files to my PC, but it became a burden when I needed to switch back and forth between the RC and development builds. This constant switching drove me crazy.

> *I needed to update my scripts to be able to save files in common spaces or on the computer instead of the app's folder.*

I searched again and came across a helpful gist: [`adb pull` from app data directory without root access](https://gist.github.com/jevakallio/452c54ef613792f25e45663ab2db117b) . Although I had doubts about the approach in the gist, due to the bug, I decided to keep it open until I felt better and could give it a try. As I suspected, the solution didn't work as expected when I tried running the command:
```bash
run-as com.package.name cp /data/user/0/com.package.name/file.ext /sdcard
```

It's not possible to bypass the permission to copy file from the app's folder to the common space. However, I learned that the app's folder is actually located under `/data/user/0`, not under `/data/data` as displayed in Android Studio.

I scrolled down to check the comments and felt lucky because I had kept the gist opened for a few days. A day after the day I first saw the gist, [@listvin](https://gist.github.com/listvin) wrote a comment that provided a solution I had been searching for over 3 years:
```
adb shell "run-as $package base64 '$1'" | base64 -d > "$save_as"
```

This beautiful command gave me an idea of how to read a file and send it to another environment using Unix's pipe:

> First, read the file with adb shell run-as and send the content to the pipe. Then, the subsequent command receives data from the pipe and writes it to a new file.

Since my knowledge about `bash` is always poor, I applied the same `base64` command to sync my backups on the device to the computer:
```bash
adb shell "run-as $package tar -czf file.gz -C path ."
adb shell "run-as $package base64 file.gz" | base64 -d > file.gz
```

Although `base64` is slow, this snippet works. However, when syncing the file back to the device:
```
base64 file.gz | adb shell "run-as $package base64 -d > file.gz"
```

I encountered an issue with the `>` operator

```
Permission denied
```

I need another command to write the file without using `>` . This requirement led me to discover a special command called [`dd`](https://wiki.archlinux.org/title/Dd). 

> Similarly to _cp_, by default _dd_ makes a bit-to-bit copy of the file, but with lower-level I/O flow control features.

Using `dd`, I was able to successfully sync the backup file to the device with the following command:
```bash
dd if=file.gz | adb shell "run-as $package dd of=file.gz"
# unzip and apply
```

This method is much faster than using `base64` when transferring data between the device and computer. Typically, it takes only around 1 second to complete.

After 3 years of searching, I finally gained an understanding of how Android Studio implemented the *Save* and *Upload* features.

-----

**Bonus: Push and Pull files between the app's folder and the computer**

```bash
# Push
dd if="$1" | adb shell "run-as $package dd of='$2'"

# Pull
adb shell "run-as $package dd if='$1'" | dd of="$2"
```
