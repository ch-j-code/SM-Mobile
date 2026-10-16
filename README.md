# SM Mobile — CH-J

SM Mobile is a standalone Android and iOS edition of CH-J Server Manager, built with Flutter and Dart. It connects directly to servers over SSH and SFTP, without an account, cloud service, or telemetry. The project is currently a development alpha (0.0.1). The desktop application is not a dependency.

## Official links and source code

- [SM Mobile — official website](https://www.sm.ch-j.cz/cs/?edition=mobile)
- [SM Mobile — releases and downloads](https://www.sm.ch-j.cz/cs/releases/?edition=mobile)
- [SM Mobile — source code browser and source archives](https://www.sm.ch-j.cz/cs/source/?edition=mobile)
- [SM Mobile — GitHub](https://github.com/ch-j-code/SM-Mobile)

This GitHub repository contains only this README. Application source code, source archives, documentation, and builds are published on the official website. To follow the development and build instructions below, download and extract the mobile source archive from the source browser, then open the extracted project directory.

## Desktop application

CH-J Server Manager is the separate desktop application for Windows, macOS, and Linux. The mobile and desktop applications can each be used independently.

- [CH-J Server Manager — official website](https://www.sm.ch-j.cz/cs/)
- [CH-J Server Manager — GitHub](https://github.com/ch-j-code/CH-J-Servermanager)
- [CH-J Server Manager — desktop source code](https://www.sm.ch-j.cz/cs/source/)

## Requirements

- Flutter with Dart compatible with the SDK constraint in `pubspec.yaml` (`>=3.10.0 <4.0.0`).
- For Android development: OpenJDK 17, Android SDK and platform tools, and an Android device or emulator. Android Studio is optional; the command-line tools can be used instead.
- For iOS development: macOS, Xcode, CocoaPods, and an iOS device or simulator.
- Python 3 for the build and integration test scripts.

The application supports Android 11 and later (API 30+), and iOS 15 and later on iPhone and iPad. Run `flutter doctor -v` to check your development environment before building.

## Getting started

Run the following commands from the project directory:

```sh
flutter pub get
flutter devices
flutter run -d <device-id>
```

Replace `<device-id>` with a device listed by `flutter devices`. To use an Android emulator, list available emulators with `flutter emulators` and launch one with `flutter emulators --launch <emulator-id>`. On macOS, you can open the iOS Simulator with `open -a Simulator`.

For VS Code, install the Dart and Flutter extensions, open `SM Mobile.code-workspace`, select a device in the status bar, and press F5.

## Using the application

1. Choose a language, read the security recommendations and the full CH-J license version 1.14, and explicitly accept the license.
2. Create a Vault with a master password of 12–128 characters. This password cannot be recovered. You can enable biometric authentication or the device passcode later in Settings.
3. Add a server. Hostnames are stored as entered; DNS results are never written back to profiles. IPv6 addresses can also be entered in square brackets.
4. Select a server and connect using a password or a securely stored private key. Private key passphrases are requested when connecting and are not stored.
5. Verify the SSH host key fingerprint through an independent channel before accepting it for the first time. A change to a known host key blocks the connection.
6. Use the terminal with a real PTY shell. CTRL and ALT apply to the next input only; ESC, TAB, and arrow keys are available on the scrollable toolbar.
7. Use Files to browse over SFTP, upload files, download through the system share/save interface, rename items, create directories, and delete items after confirmation. Directory deletion is non-recursive. Existing destination files are not overwritten; choose a different name.
8. Open Monitor to collect Linux metrics while that screen is visible and the application is in the foreground. Diagnostics provides IPv4/IPv6 DNS and TCP checks, plus details of the active SSH connection. TCP checks are not ICMP ping.

On phones, Servers, Terminal, and Files appear in the bottom navigation bar; the last item opens Monitor, Diagnostics, and Settings. Wider displays use a scrollable navigation sidebar. Settings offers Czech, German, and English, along with light, dark, and system themes.

## Importing and exporting profiles

Settings → Import / Export profiles uses the desktop-compatible `chj-server-profiles` JSON format, version 1. Each profile contains only `name`, `host`, `username`, and `port`. Passwords, private keys, passphrases, and host key fingerprints are never transferred.

Imports are limited to 5 MiB and 10,000 records. A preview lets you skip duplicates, update existing profiles, or create copies. Updating a profile clears its credentials. Equivalent IPv6 representations are normalized only for duplicate comparison; hostnames are not resolved. Configure authentication after importing or duplicating profiles. CSV is not supported in the mobile edition.

## Building and validation

Use the build script when preparing builds to share. Each run creates a new build ID, refreshes third-party license notices, and runs static analysis and tests:

```sh
python3 tool/build.py all
```

The `all` target requires macOS and both Android and iOS toolchains. It produces the following development artifacts:

- APK: `build/app/outputs/flutter-apk/app-debug.apk`
- App bundle: `build/app/outputs/bundle/debug/app-debug.aab`
- iOS simulator application: `build/ios/iphonesimulator/Runner.app`

The APK and app bundle produced by this command are debug builds, not release packages. To build a single target, use `apk`, `aab`, or `ios-simulator` instead of `all`.

You can also run the checks and builds individually:

```sh
flutter pub get
flutter analyze
flutter test
flutter build apk --debug
flutter build appbundle --debug
flutter build ios --simulator
```

### Website distribution

```sh
python3 tool/build.py distribution
# Optional simulator build with the same build ID:
flutter build ios --simulator --debug
python3 tool/prepare_release.py
```

This target requires macOS, both toolchains, and `android/key.properties`. It creates a signed release APK and AAB plus an **unsigned** iOS device application, all with the same new build ID. An unsigned IPA made from that application requires your own Apple signing and provisioning before installation; it is not an installable Ad Hoc release. The AAB is for distribution tooling, not direct installation. Use the APK on Android.

The website offers separate Desktop and Mobile editions and a source browser with the exact mobile build snapshot. Source archives exclude signing keys, password configuration, provisioning profiles, local SDK paths, caches and build outputs. Keep the Android release keystore and its password configuration backed up privately; future Android updates must use the same signing identity.

### Android release

Create your own `android/key.properties` with `storeFile`, `storePassword`, `keyAlias`, and `keyPassword`, then run:

```sh
python3 tool/build.py aab --release
```

Signing keys and the password configuration file are excluded by `.gitignore`. Release builds do not fall back to a debug signing identity.

### iOS release

Prepare an unsigned device build:

```sh
python3 tool/build.py ios
```

Open `ios/Runner.xcworkspace`, configure your own development team, provisioning, and distribution certificate, then use Xcode's Archive and App Store Connect workflow. The script does not publish anything automatically.

Direct `flutter build` commands retain the existing build ID. Do not reuse that ID for a different distributed build; use `tool/build.py` to generate a new one.

## Testing

`flutter test` covers models, profile import/export, Vault cryptography, host key verification, license acceptance, application lifecycle, and widgets. Six network scenarios are skipped unless explicitly enabled. A successful standard test run does not replace these integration tests.

### Local OpenSSH integration tests

On macOS or Linux, with OpenSSH available and port 22229 free, run:

```sh
python3 tool/test_openssh.py
python3 tool/test_openssh.py --device <ios-simulator-id>
python3 tool/test_openssh.py --device <android-device-id> --android
```

These are alternative invocations for the default test runner, an iOS simulator, or an Android emulator. Replace device placeholders with the appropriate IDs.

The script creates temporary keys in a private directory, starts a server listening only on `127.0.0.1`, and stops it after testing. It does not configure the system SSH server or modify `~/.ssh`. The test account is the current local user. The fixture is not chrooted, so test operations stay within a temporary directory. Run it on a development machine. Docker is not required.

For physical devices, follow the scenarios in [Testing](https://www.sm.ch-j.cz/cs/source/?edition=mobile&path=docs%2FTESTING.md) against your own isolated test server.

### Optional Linux container tests

An optional container test covers a real Linux environment, SSH password authentication, and monitoring. Docker and Colima are test tools, not dependencies of the application or its builds.

With a Docker-compatible runtime running, use:

```sh
python3 tool/test_linux.py --context <docker-context>
```

For example, on macOS with Colima installed:

```sh
colima start --profile sm-mobile-test --vm-type vz --cpu 2 --memory 2 --disk 10
python3 tool/test_linux.py --context colima-sm-mobile-test
colima stop --profile sm-mobile-test
```

The fixture listens only on `127.0.0.1:22239`, uses a randomly generated password for each run, removes the container automatically, and pins the image by digest. See the [OpenSSH image documentation](https://docs.linuxserver.io/images/docker-openssh-server/) and [Colima project](https://github.com/abiosoft/colima) for the underlying tools.

## Documentation

The following documents are available in the official mobile source browser and in the downloaded source archive:

- [Source audit](https://www.sm.ch-j.cz/cs/source/?edition=mobile&path=docs%2FAUDIT.md)
- [Security](https://www.sm.ch-j.cz/cs/source/?edition=mobile&path=docs%2FSECURITY.md)
- [Dependencies](https://www.sm.ch-j.cz/cs/source/?edition=mobile&path=docs%2FDEPENDENCIES.md)
- [Test scenarios and results](https://www.sm.ch-j.cz/cs/source/?edition=mobile&path=docs%2FTESTING.md)
- [Resolved components](https://www.sm.ch-j.cz/cs/source/?edition=mobile&path=docs%2FRESOLVED_COMPONENTS.md)

## License

This edition is licensed under the **CH-J Proprietary Software License 1.14**. See [LICENSE](https://www.sm.ch-j.cz/cs/source/?edition=mobile&path=LICENSE) for the full terms and [NOTICE](https://www.sm.ch-j.cz/cs/source/?edition=mobile&path=NOTICE) for distribution notices.

Third-party components retain their own licenses. Full license texts are available in [Third-party notices](https://www.sm.ch-j.cz/cs/source/?edition=mobile&path=assets%2Flegal%2Fthird_party_notices.txt), with a component list in [Resolved components](https://www.sm.ch-j.cz/cs/source/?edition=mobile&path=docs%2FRESOLVED_COMPONENTS.md). They are also available in the application under Settings → Component licenses. The security recommendations are adapted from the CH-J desktop project in Czech, German, and English.

## Topics

`#flutter` `#dart` `#android` `#ios` `#ipados` `#ssh` `#ssh-client` `#sftp` `#terminal` `#server-management` `#server-monitoring` `#network-diagnostics`
