# EAS Update (OTA) Integration and Usage Guide

This document summarizes the process for setting up and using Expo Application Services (EAS) Update to push JavaScript changes instantly without rebuilding native binaries.

---

## 1. Installation & Configuration (One-time Setup)

### Install the library
$ npx expo install expo-updates $

### Configure app.json
Ensure these fields exist to identify your project on the Expo servers:
{
  "expo": {
    "updates": { "url": "https://u.expo.dev/YOUR_PROJECT_ID" },
    "runtimeVersion": { "policy": "appVersion" },
    "extra": { "eas": { "projectId": "YOUR_PROJECT_ID" } }
  }
}

### Configure eas.json
Add the "channel" field so the Native App knows where to look for updates:
{
  "build": {
    "production": {
      "channel": "production", 
      "autoIncrement": true,
      "android": { "buildType": "app-bundle" },
      "env": { "EXPO_PUBLIC_API_URL": "..." }
    }
  }
}

---

## 2. Cloud Pipeline Setup (One-time Setup)

This process establishes the connection between your Git branches and your production environment.

1.  **Create Channel**: eas channel:create production (Select the master branch when prompted).
2.  **Create Branch**: eas branch:create master (If it doesn't exist yet).
3.  **Link them**: 
    eas channel:edit production --branch master
    
    *Explanation: This command ensures the production channel always pulls the latest update from the master branch.*

---

## 3. Shipping the Native Build

Run a full build when you: Install new Native modules, change Icons/Splash screens, or update the appVersion.
$ eas build --platform all --profile production --auto-submit $

*Note: This build carries the "address" of the production channel hardcoded within the binary.*

---

## 4. Rapid JS Updates (OTA Update)

Use this for: UI tweaks, text changes, React logic, or JS bug fixes.
$ eas update --branch master --message "Description of changes" $

---

## 💡 How Updates Are Applied on Devices

1.  **First Launch**: The app silently downloads the update in the background (takes approx. 10–30 seconds).
2.  **Force Close (Kill app)**: Swipe the app up to close it completely.
3.  **Second Launch**: The app swaps the old code for the new update.

> [!IMPORTANT]
> Runtime Version Compatibility: A JS Update will only be delivered to the specific "runtime version" it was built for. If you increment the version in app.json to 1.0.3, you must submit a new Native Build to receive updates targeted at that version.
