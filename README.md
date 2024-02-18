# Hotfix
The Hotfix can dynamically fix online bugs for Android without republishing an app.
## Getting started
Add hotfix-gradle-plugin as a dependency in your main `build.gradle` in the root of your project:

```gradle
buildscript {
    dependencies {
        classpath ('com.lyr.hotfix:hotfix-patch-gradle-plugin:1.9.1')
    }
}
```

Then you need to "apply" the plugin and add dependencies by adding the following lines to your `app/build.gradle`.

```gradle
dependencies {
    //optional, help to generate the final application
    provided('com.lyr.hotfix:hotfix-android-anno:1.9.1')
    //hotfix's main Android lib
    compile('com.lyr.hotfix:hotfix-android-lib:1.9.1')
}
...
...
apply plugin: 'com.lyr.hotfix.patch'
```

If your app has a class that subclasses android.app.Application, then you need to modify that class, and move all its implements to [SampleApplicationLike] rather than Application:

```java
-public class YourApplication extends Application {
+public class SampleApplicationLike extends DefaultApplicationLike {
```

Now you should change your `Application` class, make it a subclass of [hotfixApplication]. As you can see from its API, it is an abstract class that does not have a default constructor, so you must define a no-arg constructor:

```java
public class SampleApplication extends hotfixApplication {
    public SampleApplication() {
      super(
        //hotfixFlags, which types is supported
        //dex only, library only, all support
        ShareConstants.hotfix_ENABLE_ALL,
        // This is passed as a string so the shell application does not
        // have a binary dependency on your ApplicationLifeCycle class.
        "hotfix.sample.android.app.SampleApplicationLike");
    }
}
```

Use `hotfix-android-anno` to generate your `Application` is recommended, you just need to add an annotation for your [SampleApplicationLike] class

```java
@DefaultLifeCycle(
application = "hotfix.sample.android.app.SampleApplication",             //application name to generate
flags = ShareConstants.hotfix_ENABLE_ALL)                                //hotfixFlags above
public class SampleApplicationLike extends DefaultApplicationLike
```

How to install hotfix? learn more at the sample [SampleApplicationLike].

For proguard, we have already made the proguard config automatic, and hotfix will also generate the multiDex keep proguard file for you.

For more hotfix configurations, learn more at the sample [app/build.gradle].

## Ark Support
How to run hotfix on the Ark?
### building patch
Just use the following command:
```buildconfig
bash build_patch_dexdiff.sh old=xxx new=xxx
```
* `old` indicates the absolute path of android apk(not compiled by Ark) with bugs
* `new` indicates the absolute path of android apk(not compiled by Ark) with fixing

The patch file is packaged in APK.
### compiling in Ark
TODO

At present it's compiled by Ark compiler team. The output patch is still packaged in APK format without signature.
### packaging the patch
For hotfix-cli, add the following lines to your `hotfix_config.xml`. Otherwise, the default configure will be used.
```xml
<issue id="arkHot">
   <path value="arkHot"/>         // path of patch
   <name value="patch.apk"/>      // name of patch
</issue>
```
For gradle, add the following lines to your `app/build.gradle`. Otherwise, the default configure will be used.
```gradle
ark {
   path = "arkHot"         // path of patch
   name = "patch.apk"      // name of patch
}
```
The patch is compiled by Ark and placed on the above path. all subsequent operations are same as hotfix-cli or gradle.

The ultimated patch APK consists of two patch files:

* `classes.dex` for android
* `patch.apk` with so for Ark.

## hotfix Known Issues
There are some issues which hotfix can't dynamic update.

1. Can't update AndroidManifest.xml, such as add Android Component.
2. Do not support some Samsung models with os version android-23.
3. Due to Google Play Developer Distribution Agreement, we can't dynamic update our apk.

## hotfix Support
Any problem?

1. Learn more from [hotfix-sample-android].
2. Read the [source code].
3. Contact us for help.
