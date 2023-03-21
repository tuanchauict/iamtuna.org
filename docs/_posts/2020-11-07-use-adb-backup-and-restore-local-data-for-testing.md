---
layout: post
title:  "Use ADB to backup and restore local data for testing"
author: "Tuna"
comments: false
category: Android
tags: android adb debugging
excerpt_separator: <!--more-->
sticky: false
hidden: false
draft: false
---

In addition to [making ADB more useful in testing and debugging Android apps](/2020-10-25/extend-adb-to-make-app-debugging-easier), today I would like to introduce a short trick to backup and restore the local data for testing.

Test data on debugging built app is as important as the real data on users’ devices. Have you ever had to clean the app data to be able to start the app for debugging? Or have you ever worked in parallel for debug and release versions and they have different versions of Database? If the answer is YES (I’m sure most of us do), it hurts, especially when the data is related to date or time (UI is different for items created last month and today); or it’s hard and takes time to create test data (like creating new accounts)<!--more-->.

## Let’s backup
```bash
package=<app.package>
device_path="zzzBackup" # use any name you like
backup_time=$(date +%F_%H-%M-%S)
backup_version= \
    $(adb shell dumpsys package $package \
   | grep versionName | sed -E "s/    versionName=//")
echo "Backup version:===$backup_version==="
separator="__version_"
backup_path="$device_path/$backup_time$separator$backup_version"
echo Backup to "$backup_path"
adb shell "run-as $package mkdir -p $backup_path"
adb shell "run-as $package cp -a shared_prefs $backup_path"
adb shell "run-as $package cp -a databases $backup_path"
```

With this script, a new folder under the app data folder will be created and it will be the root directory for all of the data (`/data/data/app.package`) we backup (`zzzBackup`). Actually, you can use an external path, like `/sdcard/Download/Backup` for **`device_path`** to be able to check it with a File browser app. Unfortunately, this doesn’t work on Samsung devices or from Android 11.

The script uses date and time and the app version to create the backup version folder. The folder will be created like this:
```
2020-09-28_15-30-27__version_1.1.0
```

One more thing, the script only backup two main directories of the app which almost most of the app will have (shared_prefs and databases). You can add more folders you want or just need to backup all with `.` . However, backup all may get permission error or slow due to cached or leak canary folder (if you use [leak canary](https://square.github.io/leakcanary/)).

## Restore
```bash
package=<app.package>

device_path="zzzBackup"
restore_path="$device_path/$1"
echo "Restore from $restore_path"

adb shell am force-stop $package

adb shell "run-as $package rm -rf shared_prefs" # Clean app's data
adb shell "run-as $package rm -rf databases" # Clean app's data

adb shell "run-as $package cp -a $restore_path/shared_prefs ."
adb shell "run-as $package cp -a $restore_path/databases ."
```

There are two things we must guarantee in the restore script. The first is **`device_path`**, it must have the same value as the path in the backup script. The latter is the backed-up directories.

## Browse
It’s hard to remember all of the back-ups, so, we need the third script to check which backup do we have.
```bash
package=jp.naver.line.android
device_path="zzzBackup"
adb shell "run-as $package ls $device_path"
```
Again, we must guarantee the device_path value.

That’s all for backup and restore. Let’s run these scripts together

```bash
sh backup.sh
sh browse.sh
sh restore.sh 2020-09-28_15-30-27__version_1.1.0
```

Happy debugging and MAGA — Make ADB Great Again!