# EAS Update Integration and Usage Guide (OTA)

This document summarizes the process of setting up and using Expo Application Services (EAS) Update to quickly update JavaScript source code without rebuilding the Native binary.

---

## 1. Installation and Configuration (One-time setup)

### Install the library
```bash
npx expo install expo-updates
```

### Configure `app.json`
Ensure the following fields exist to identify your project on the Expo server:
```json
{
  "expo": {
    "updates": { "url": "https://u.expo.dev/YOUR_PROJECT_ID" },
    "runtimeVersion": { "policy": "appVersion" },
    "extra": { "eas": { "projectId": "YOUR_PROJECT_ID" } }
  }
}
```

### Configure `eas.json`
Add the `"channel": "production"` line so the Native App knows where to look for updates:
```json
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
```

---

## 2. Cloud Pipeline Setup (One-time setup)

This process builds the "pipeline" that delivers code from your Git branch to the user's app.

1. **Create Channel**: `eas channel:create production` (Select the `master` branch when prompted).
2. **Create Branch**: `eas branch:create master` (If it doesn't already exist).
3. **Link them**: 
   ```bash
   eas channel:edit production --branch master
   ```
   *Explanation: This command ensures that the `production` channel always pulls the latest code from the `master` branch.*

---

## 3. Exporting the Native Build (Binary)

You need to run this command when: You install new Native libraries, change the Icon/Splash screen, or change the `appVersion`.
```bash
eas build --platform all --profile production --auto-submit
```
*Note: This build will have the "address" of the `production` channel embedded within it.*

---

## 4. Fast JS Code Update (OTA Update)

Use this when: You fix CSS, text, React logic, or JS bugs.
```bash
eas update --branch master --message "Description of changes"
```

---

## 💡 Rules for Receiving Updates on Devices

1. **Open app for the 1st time**: The app silently downloads the update in the background (takes about 10-30 seconds).
2. **Force close (Kill app)**: Swipe the app up to close it completely.
3. **Open app for the 2nd time**: The app will replace the old code with the new updated code.

> [!IMPORTANT]
> **Runtime Version**: JS Updates are only sent to the matching "runtime version" (e.g., 1.0.2). If you increase the version in `app.json` to 1.0.3, you must rebuild and submit a new Native App to receive updates intended for version 1.0.3.
