---
title: Managed app versions
---
import { Collapsible } from '~/ui/components/Collapsible';

Both Android and iOS expose 2 values that identify the version of an application, one that is visible in stores and one visible only to developers. In managed projects we use fields `version`/`android.versionCode`/`ios.buildNumber` in **app.json** to define a version, where `android.versionCode`/`ios.buildNumber` represents developer facing build version and `version` is the user facing value visible in stores. For bare projects each of those values map to specific parts of the native configuration:

- `version` field in **app.json** on iOS represents `CFBundleShortVersionString` in **Info.plist**.
- `version` field in **app.json** on Android represents `versionName` in **android/app/build.gradle**.
- `ios.buildNumber` field in **app.json** represents `CFBundleVersion` in **Info.plist**.
- `android.versionCode` field in **app.json** represents `versionCode` in **android/app/build.gradle**.

EAS Build provides multiple ways of managing your application versions. By default your local configuration takes precedence, but you can also opt-into EAS server managing your application versions.

To simplify this description, we will use **app.json** terminology (`version`/`versionCode`/`buildNumber`) in the rest of this page, but unless stated otherwise the same applies to bare projects.

## Examples

<Collapsible summary="Remote version source">

With this **eas.json**, all builds will have version based on the value stored on EAS servers, but the version will be incremented only when building with `production` profile.

```json
{
  "cli": {
    "appVersionSource": "remote"
  },
  "build": {
    "staging": {
      "distribution": "internal",
      "android": {
        "buildType": "apk"
      }
    },
    "production": {
      "autoIncrement": true
    }
  }
}
```

</Collapsible>

<Collapsible summary="Local version source">

With this **eas.json**, all builds will have version based on the value from **app.json** or native code. When you build using `production` profile, a version will be incremented in the local code before the build.

```json
{
  "cli": {
    "appVersionSource": "local"
  },
  "build": {
    "staging": {
      "distribution": "internal",
      "android": {
        "buildType": "apk"
      }
    },
    "production": {
      "autoIncrement": true
    }
  }
}
```

</Collapsible>

## Remote version source

You can configure your project to rely on EAS servers to store and manage version of your app, to do that add `{ "cli": { "appVersionSource": "remote" } }` in your **eas.json**. Remote version will be initialized with the value from a local project, but if you want to set it to something else, or EAS CLI is not able to detect what version an app is on, you can configure it with `eas build:version:set` command. EAS is storing version information scoped by account, slug, platform and application ID/bundle identifier, so e.g. if you are building variants with multiple application ID or bundle identifiers, versioning will be independent for each of them.

If you want to build your project in Android Studio or Xcode, you can update your local project with the remote versions using `eas build:version:sync`.

Enabling `autoIncrement` option in this mode is only possible for `versionCode`/`buildNumber`, make sure to remove those values from **app.json** otherwise those versions will still show up in the manifest exposed by `expo-constants` even though app itself will have a correct version.

### Limitations

- `eas build:version:sync` command on Android does not support bare projects with multiple flavors, but rest of the remote versioning functionality should work with all projects.
- `autoIncrement` does not support `version` option if `appVersionSource` is set to remote.
- It's not supported if you are using EAS Update and runtime policy set to `"runtimeVersion": { "policy": "nativeVersion" }`.

### Recommended workflow

Main goal of this feature is to avoid manual changes to the project every time you are uploading new archive to run it on a TestFlight or Play Store testing channels, but when you are doing production release the change should be explicit. We recommend to always update `version` field after

## Local version source

By default all projects keep versions locally, EAS itself does not interact with and just builds project as it is, unless `autoIncrement` option is enabled. In case of bare React Native projects values in native code take priority, but `expo-constants` and `expo-updates` read values from **app.json**, so if you rely on those values for anything you should keep them in sync with native code. It's especially important if you are using EAS Update with runtime policy set to `"runtimeVersion": { "policy": "nativeVersion" }`, because it might result in delivery of updates to the wrong version of an application.

### Limitations

- With `autoIncrement` option, you need to commit your changes on every build, if you want the version change to persist. It's especially problematic when building from CI.
- `autoIncrement` is not supported if you are using a dynamic config (**app.config.js**).
- For a bare React Native projects with gradle configuration that supports multiple flavors, EAS CLI is not able to read or modify the version, so `autoIncrement` options is not supported and versions will not be listed on the build page.


