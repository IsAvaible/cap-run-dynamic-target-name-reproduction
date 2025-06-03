# Capacitor iOS Dynamic App Target Issue Reproduction

This repository demonstrates an issue with the Capacitor CLI (`npx cap run ios`) when using Xcode projects with multiple iOS schemes whose names do not match the target name.

It is a reproduction for the following issue [[Bug]: Dynamically resolve target name in cap run ios #8024](https://github.com/ionic-team/capacitor/issues/8024)

## Problem Overview

When running an iOS app using Capacitor with a custom scheme via:

```bash
npx cap run ios --scheme Dev --configuration Dev
```

The CLI assumes that the resulting `.app` bundle will be named after the scheme (e.g., `Dev.app`). However, Xcode names the `.app` file based on the **target name**, not the **scheme name**. If these differ, the deployment fails because the `.app` bundle is not found at the expected location.

### Example Error

```bash
Deploying Dev.app to 00008101-001525992141001E - failed!
[error] ERR_UNKNOWN: Path
        '<project_dir>/ios/DerivedData/<id>/Build/Products/Dev-iphoneos/Dev.app'
        not found
```

In reality, the `.app` bundle exists at:

```
<project_dir>/ios/DerivedData/<id>/Build/Products/Dev-iphoneos/App.app
```

(`App` is the name of the target in this example.)

## Expected Behavior

The Capacitor CLI should:

* Determine the **actual target name** for the selected scheme using:

  ```bash
  xcodebuild -scheme <scheme> -showBuildSettings
  ```
* Extract the `TARGET_NAME` from the output.
* Use this `TARGET_NAME` to correctly locate the resulting `.app` bundle.

## Reproduction Details

This repo contains a simple Capacitor project with two iOS schemes:

* `Dev`
* `Release`

Both schemes build the same target: `App`.

To reproduce the issue:

1. Clone the repo and install dependencies:

   ```bash
   npm install
   ```

2. Add the iOS platform:

   ```bash
   npx cap add ios
   ```

3. Try to run with a custom scheme:

   ```bash
   npx cap run ios --scheme Dev --configuration Dev
   ```

You’ll see a deployment failure due to the mismatched `.app` path.