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

Three years ago, I wrote [a note about how to use `adb` to backup the test data](https://iamtuna.org/2020-11-07/use-adb-backup-and-restore-local-data-for-testing). At that time, or at the time I created the scripts (before I wrote the note), we are still able to save file in the common space such as SDCard by using <!--more-->
```
adb shell run-as ...
``` 
However, as the Android 10 was launch, this is no longer possible. The command run as the app only able to save file within the app's directory.
```
Permission denied
```
I was amazed by how Android Studio able to do *Save* and *Upload* with the Device Explorer tool. I tried to search and search and search but, always, I got no clue how the IDE implemented that.

I had almost given up the search until recently, I had to figure out how to do this again since I had a bug related to Release Candidate build, which is signed with another keystore to the development build. Testing the RC build means I have to remove the app, or cleaning the backups I have before. I can use Device Explorer to backup the backup files to the PC. But again, it's a huge burden when I go back and test the development build. Switching back and forth between RC and dev builds drove me crazy.

> *I need to update my scripts to be able to save file into the common space or to the computer instead of the app's folder.*

I searched again, and I found out a gist: [`adb pull` from app data directory without root access](https://gist.github.com/jevakallio/452c54ef613792f25e45663ab2db117b) . I doubted the approach of the gist but I was too tired due to the bug, I decided to keep it opens for a while until I felt better to give it a try.
As I guessed, the solution does not work.
```
run-as com.package.name cp /data/user/0/com.package.name/file.ext /sdcard
```

It's not possible to bypass the permission just to copy file from the app's folder to the common space. However, I learnt that the app's folder is under `/data/user/0`, not under `/data/data` as the Android Studio displays.

I scrolled down to check the comments and I felt lucky because I had kept the gist opened for some days. A day after the day I first saw the gist, [listvin](https://gist.github.com/listvin) wrote a comment that had a solution that I had searched for 2 years
```
adb shell "run-as $package base64 '$1'" | base64 -d > "$save_as"
```

This beautiful command gave me an idea about how to read a file and send it to another environment via Unix's pipe:

> First, read file with `adb shell run-as` and send the content to pipe. 
> Then the subsequence command receive data from the pipe and write it to a new file.

Since my knowledge about `bash` is always bad, I applied the same `base64` command to sync my backups on the device to the computer with
```
adb shell "run-as $package tar -czf file.gz -C path ."
adb shell "run-as $package base64 file.gz" | base64 -d > file.gz
```

`base64` is slow but this snippet works. However, when syncing file back to the device
```
base64 file.gz | adb shell "run-as $package base64 -d > file.gz"
```

`>` operator does not work.

```
Permission denied
```

I need another command to write file without `>` . This demand drove me to a special command [`dd` ](https://wiki.archlinux.org/title/Dd). 

> Similarly to _cp_, by default _dd_ makes a bit-to-bit copy of the file, but with lower-level I/O flow control features.

With `dd`, I could make syncing backup file to the device works with
```
dd if=file.gz | adb shell "run-as $package dd of=file.gz"
# unzip and apply
```

This is also much faster than using `base64` when transferring data between the device and computer. Usually, it takes only around 1 second to do so.

Finally, after 3 years of search, I could understand how Android Studio implemented *Save* and *Upload* features.
