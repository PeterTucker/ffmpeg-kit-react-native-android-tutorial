
# Integrating `ffmpeg-kit-react-native` Locally in a React Native Project for Android

This guide explains how to build and use a local version of `ffmpeg-kit-react-native` in a React Native project (`MyApp`), including `.aar` generation, dependency setup, and usage instructions.

---
## Initial Setup
```
mkdir 'my-project'
cd my-project
npx @react-native-community/cli init MyApp --pm yarn
git clone https://github.com/arthenica/ffmpeg-kit
```

## 📁 1. Folder Structure

Your directory layout should look like this:

```
~/my-project/
├── MyApp/                      # Your React Native app
└── ffmpeg-kit/                 # Cloned ffmpeg-kit repo
    └── react-native/
```
---

## 🛠️ 2. Build the `.aar` File

Follow pre-requisites:
[https://github.com/arthenica/ffmpeg-kit/blob/main/android/README.md](https://github.com/arthenica/ffmpeg-kit/blob/main/android/README.md)

Navigate to the `ffmpeg-kit` repo and run:

```bash
./android.sh
```

To build only for modern devices (`arm64-v8a`):

```bash
./android.sh \
  --disable-arm-v7a \
  --disable-arm-v7a-neon \
  --disable-x86 \
  --disable-x86-64
```

Then copy the output:

```bash
mkdir ../MyApp/android/libs/
cp prebuilt/bundle-android-aar/ffmpeg-kit/ffmpeg-kit.aar ../MyApp/android/libs/ffmpeg-kit.aar
```

---

## 📦 3. Download Required `.jar` Files

Download from:

> https://github.com/tanersener/smart-exception/releases

Copy all `.jar` files into:

```bash
MyApp/android/libs/
```

Make sure to include files like:

- `smart-exception-java-0.2.1.jar`
- `smart-exception-common-0.2.1.jar`
- `smart-exception-java9-0.2.1.jar`
- `smart-exception-logback-0.2.1.jar`

---

## ⚙️ 4. Update `android/build.gradle`

```groovy
allprojects {
    repositories {
        google()
        mavenCentral()
        flatDir {
            dirs "$rootDir/libs"
        }
    }
}
```

---

## ⚙️ 5. Update `android/app/build.gradle`

```groovy
repositories {
    flatDir {
        dirs "${rootProject.projectDir}/libs"
    }
}

dependencies {
    implementation("com.facebook.react:react-android")
    implementation fileTree(dir: "$rootDir/libs", include: ["*.jar"])
    implementation(name: 'ffmpeg-kit', ext: 'aar')

    if (hermesEnabled.toBoolean()) {
        implementation("com.facebook.react:hermes-android")
    } else {
        implementation jscFlavor
    }
}
```
---
## ⚙️ 6. Update `ffmpeg-kit/react-native/android/bundle.gradle`
remove `repositories` and `dependencies` section and replace with:
```gradle
repositories {
  google()
  mavenCentral()
}

dependencies {
  api 'com.facebook.react:react-native:+'
  implementation(name: 'ffmpeg-kit', ext: 'aar')
}
```

---

## 🔗 7. Link `ffmpeg-kit-react-native` Locally

In `MyApp/package.json`:

```json
"dependencies": {
  "ffmpeg-kit-react-native": "../ffmpeg-kit/react-native"
}
```
or...
```
yarn add '../ffmpeg-kit/react-native/'
```

Then run:

```bash
yarn install
```
---

## ⚙️ 8. Compile:
Add the following line to App.tsx in `MyApp`
```
import { FFmpegKit } from 'ffmpeg-kit-react-native';
```

**Start Metro Server:**

In one terminal window:
```bash
npx react-native start --reset-cache
```
**Clean and Launch App:**

In 2nd terminal window:
```bash
cd android
./gradlew clean
cd ..
npx react-native run-android
```

If you get no errors, then you've likely compiled the app with ffmpeg correctly.

---

## 🚀 Extra commands:

**Build `.aar`:**

```bash
./android.sh
```

**Build for `arm64-v8a` only:**

```bash
./android.sh \
  --disable-arm-v7a \
  --disable-arm-v7a-neon \
  --disable-x86 \
  --disable-x86-64
```

**Kill Metro server (if stuck):**

```bash
lsof -i :8081
kill -9 <PID>
```

**Start Metro Server:**

```bash
npx react-native start --reset-cache
```

**Clean and Launch App:**

```bash
cd android
./gradlew clean
cd ..
npx react-native run-android
```
