# ProGuard Boilerplate

This repo's purpose is to centralize ProGuard configurations for Android apps. It includes a ProGuard file that has configurations for many libraries. Contributions are **strongly** suggested :-)

Everything is included in one file, `proguard.cfg`, including the Android defaults (from the Android SDK). To use it with gradle, copy `proguard.cfg` to your module's directory and add the following to your module's `build.gradle`:

```groovy
...

buildTypes {
  debug {
    minifyEnabled true
    shrinkResources true
    proguardFiles 'proguard.cfg'
  }
  release {
    minifyEnabled true
    shrinkResources true
    proguardFiles 'proguard.cfg'
  }
}

...
```

## Caveat

I am by no means an expert at ProGuard. Not even close. So if you know better than me, please feel free to create PRs that fix whatever stupid thing I did.


## Android Defaults

The first portion of the file comes from the Android SDK (`tools/proguard/proguard-android.txt`). It contains what Google believes the defaults should be. 

#### Optimizations
If you want to use optimizations (YMMV) you can include the top portion of the SDK's `tools/proguard/proguard-android-optimize.txt`.

## Project

The next portion contains the settings for you project. By default it will use the "nuclear option"; it will keep all of your public classes with their constructors, methods, and fields:

```proguard
-keep public class <YOUR PROJECT PACKAGE HERE>.** {
    <fields>;
	<init>(...);
    <methods>;
}
```

This can be customized to your desire. I suggest using the "nuclear option", or a close approximation of it, if you use dependency injection.

#### Obfuscation
`-dontobfuscate` is also used. If you want obfuscation, just remove it (I've always run into trouble with obfuscation on and dependency injection)

## 3rd Party Libraries

The following lists all the 3rd party libraries that we provide ProGuard configurations for, and the versions they were tested against.

 - [RoboGuice](https://github.com/roboguice/roboguice) [3.0.1] - (RoboBlender enabled) ([explanation](#roboguice))
 - [GSON](https://github.com/google/gson) - ([explanation](#gson)) - [2.3.1, 2.4.0]
 - [Retrofit](https://github.com/square/retrofit) [1.9.0] - ([explanation](#retrofit))
 - [Firebase](https://www.firebase.com/docs/android/quickstart.html) [2.4.1] - ([explanation](#firebase))
 - [Mixpanel](https://github.com/mixpanel/mixpanel-android) [4.7.0] - ([explanation](#mixpanel))
 - [Parceler](https://github.com/johncarl81/parceler) [1.0.4] - ([explanation](#parceler))
 - [Metadata-Extractor](https://github.com/drewnoakes/metadata-extractor) [2.8.1] - ([explanation](#metadata-extractor))
 - [Widgets](https://github.com/eygraber/widgets) [0.1.15] - ([explanation](#widgets))
 - [Google Play Services](https://developers.google.com/android/guides/setup) [8.3.0] - ([explanation](#google-play-services))
 

## 3rd Party Library Notes

This is hopefully where things get a little less opinionated. Below is a list of ProGuard settings that I've compiled for libraries that I use in my apps. Again, feel free to add to, or modify, this list. Where possible, I try to include URLs to official documentation of PRoGuard settings.

Any `-keepattributes` specified by documentation is included at the end of `proguard.cfg` with a comment noting what libraries required it.

The following is not a full list of the libraries we provide configurations for. It only includes the ones that need some explanation.

### RoboGuice

[Documentation](https://github.com/roboguice/roboguice/wiki/ProGuard) - As of 11/24/2015, the last time the documentation was updated was on 10/12/2012. I haven't followed the directions exactly, but it works for me on some large projects.

See [obfuscation](#obfuscation) regarding `-dontobfuscate`
See [optimizations](#optimizations) regarding `-dontoptimize`

What wasn't included from their suggestions:

```proguard
-target 1.6 # didn't need this; also most folks target 1.7 now
-dontobfuscate # didn't need this; we defined it already

# didn't need the following; we defined them in the Android defaults
-dontoptimize
-dontusemixedcaseclassnames
-dontskipnonpubliclibraryclasses
-dontpreverify
-verbose

# didn't need the following; also not sure exactly what they do
-dump ../bin/class_files.txt
-printseeds ../bin/seeds.txt
-printusage ../bin/unused.txt
-printmapping ../bin/mapping.txt

# The -optimizations option disables some arithmetic simplifications that Dalvik 1.0 and 1.5 can't handle. 
-optimizations !code/simplification/arithmetic  # I believe we don't need this since -dontoptimize is set

# didn't need the following; we defined them already
-keep public class * extends android.app.Activity
-keep public class * extends android.app.Application
-keep public class * extends android.app.Service
-keep public class * extends android.content.BroadcastReceiver
-keep public class * extends android.content.ContentProvider

# we needed more than this, explained below
-keepclassmembers class * {
    @com.google.inject.Inject <init>(...);
}

# This was done on a large project that didn't use the On*Event convention so it was handled by adding @android.support.annotation.Keep to methods that used @Observes
# Because of that, this is commented out in our settings. Feel free to enable it if you want it, but notice that it will apply to any method with a signature that matches that regex
# There's no way to keep all @Observes methods, so use the On*Event convention to identify event handlers
-keepclassmembers class * { 
    void *(**On*Event); 
}

# didn't need the following; we defined them already
-keep public class * extends android.view.View {
      public <init>(android.content.Context);
      public <init>(android.content.Context, android.util.AttributeSet);
      public <init>(android.content.Context, android.util.AttributeSet, int);
      public void set*(...);
  } 
```

What was included / modified:

```proguard
-keep class com.google.inject.Binder

# it is possible to use @com.google.inject.Inject or @javax.Inject annotations with RoboGuice
# We want to keep classes that have fields or constructors annotated with them
-keepclassmembers class * {
    @com.google.inject.Inject <fields>;
    @com.google.inject.Inject <init>(...);
}
-keepclassmembers class * {
    @javax.Inject <fields>;
    @javax.Inject <init>(...);
}

-keep public class roboguice.**

# got errors without it
-keep public class roboguice.event.EventManager {
    <fields>;
}

# Using RoboGuice with Proguard requires you to add a package to your AnnotationDatabaseImpl (note that this is not the same as roboguice.AnnotationDatabaseImpl)
# https://github.com/roboguice/roboguice/wiki/RoboBlender-wiki#configuring-roboblender-for-a-large-application-using-libraries
-keep public class <THE PACKAGE YOU PROVIDED FOR AnnotationDatabaseImpl>.AnnotationDatabaseImpl {
    *;
}

# RoboGuice didn't seem to be able to bootstrap without this
-keepclasseswithmembers class * {
  @com.google.inject.Provides <methods>;
}

-dontwarn roboguice.**
-dontwarn org.roboguice.**
```

### GSON

[Documentation](https://github.com/google/gson/blob/master/examples/android-proguard-example/proguard.cfg) - As of 11/24/2015

Note that any class that will be serialized/deserialized with GSON will need to be explicitly kept (if it is part of your package then it will be by default using the "nuclear option").

### Retrofit

Note that any service interface will need to be explicitly kept (if it is part of your package then it will be by default using the "nuclear option").

### Firebase

[Documentation](http://stackoverflow.com/a/26274623/691639) - As of 11/24/2015, didn't follow exactly.

```proguard
# DIDN'T USE
-keep class org.apache.** { *; }
-keepnames class com.fasterxml.jackson.** { *; }
-keepnames class javax.servlet.** { *; }
-keepnames class org.ietf.jgss.** { *; }
-dontwarn org.w3c.dom.**
-dontwarn org.joda.time.**
-dontwarn org.shaded.apache.**
-dontwarn org.ietf.jgss.**

# USED
-keep class com.firebase.** { *; }
-dontwarn com.fasterxml.**
```

### Mixpanel

[Documentation](http://stackoverflow.com/a/28371616/691639) - As of 11/24/2015, didn't follow exactly.

```proguard
# USED
-keep class com.mixpanel.android.abtesting.** { *; }
-keep class com.mixpanel.android.mpmetrics.** { *; }
-keep class com.mixpanel.android.surveys.** { *; }
-keep class com.mixpanel.android.util.** { *; }
-keep class com.mixpanel.android.java_websocket.** { *; }

-keepattributes InnerClasses

# DIDN'T USE
-keep class **.R
# DIDN'T USE - already in Android defaults
-keep class **.R$* {
    <fields>;
}
```

### Parceler

[Documentation](https://github.com/johncarl81/parceler/blob/master/examples/app/proguard.cfg#L74) - As of 11/24/2015

### Metadata-Extractor

[Documentation](https://github.com/drewnoakes/metadata-extractor/wiki/ProGuard) - As of 11/24/2015

### Widgets

[Documentation](https://github.com/eygraber/widgets#proguard) - As of 11/24/2015

### Google Play Services

[Documentation](https://developers.google.com/android/guides/setup) - Even though Google says they include the correct ProGuard configuration during the build process, I've found that I've had trouble with it in the past, which is why I included the settings for it that have worked for me.
