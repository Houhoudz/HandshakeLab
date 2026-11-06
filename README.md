# Handshake Lab (Android)

A minimal, offline-only Android Studio project with two functions:

1. Convert supported handshake files (PCAP, PCAPNG, HC22000/22000, HCCAPX, HCCAP) to Hashcat 22000 text (`.hc22000`).
2. Check a user-selected local dictionary against the selected handshake file, using the bundled native C verifier through JNI.

The app requests no location, Wi-Fi, storage-wide, network, or root permissions. File access is performed through Android's Storage Access Framework (`ACTION_OPEN_DOCUMENT` / `ACTION_CREATE_DOCUMENT`). The app does not capture traffic or scan for networks.

## Build

Requirements:
- Android Studio with Android SDK Platform 35
- Android SDK Build-Tools compatible with AGP 8.7.3
- Android NDK installed through SDK Manager
- CMake 3.22.1
- JDK 17

Open this folder in Android Studio, allow Gradle sync, then choose **Build > Build Bundle(s) / APK(s) > Build APK(s)**.

Or from a machine with Android SDK configured:

```bash
./gradlew assembleDebug
```

APK output (after a successful local build):

```text
app/build/outputs/apk/debug/app-debug.apk
```

This environment did not have Android SDK/NDK installed, so an APK has not been built or device-tested here. The ZIP contains the Android Studio source project.

## Use

- Choose a PCAP/PCAPNG or supported handshake file.
- Tap **Convert to HC22000** and choose an output location.
- For offline dictionary testing, choose a handshake file and a UTF-8 wordlist, then start the local test.
- Candidate passphrases are limited to 8–63 UTF-8 bytes.
- The included native backend supports PMKID and EAPOL Key Version 1/2. EAPOL Key Version 3 is not supported by this native backend.

Use only handshake files and networks that you own or are explicitly authorized to assess. Dictionary testing can be computationally expensive and a negative result only means that the tested dictionary did not contain the passphrase.

## Architecture

```text
PCAP / PCAPNG / HC22000
       -> HandshakeParser.kt + PcapParser.kt
       -> Hashcat 22000 text output

Wordlist -> Kotlin batch loop -> NativeCracker.kt
                              -> libcracker.so (JNI / C)
```

The JNI package and class name are intentionally retained as `com.lsd.wififrankenstein.util.NativeCracker` because the native C exports use those exact JNI symbol names.

## Source and license

The parser and native verifier files in this project are derived from **WIFI-Frankenstein** by LowSkillDeveloper:
https://github.com/LowSkillDeveloper/WIFI-Frankenstein

The upstream repository is licensed under GNU GPL v3.0. This derivative project is distributed under GPL-3.0 as well; see `LICENSE`. The small Android UI and project wiring are added for this minimal application. Preserve the license and source attribution when redistributing.

## Build APK with GitHub Actions

This project includes `.github/workflows/android.yml`. To build the APK in GitHub:

1. Create a new repository on GitHub.
2. Upload the **contents** of this project folder (including the hidden `.github` folder) to the repository's `main` branch.
3. Open the repository's **Actions** tab and select **Android Debug APK**.
4. Select **Run workflow** and confirm.
5. When the run finishes successfully, open that run and download the `HandshakeLab-debug-apk` artifact. Extract it to get `app-debug.apk`.

The workflow builds a debug APK with Android SDK 35, NDK 27.2.12479018, CMake 3.22.1, and JDK 17. The APK is not included in this source ZIP; GitHub creates it only after a successful workflow run.
