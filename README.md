# Android / iOS app with shared Rust logic

![logos](img/logos_.png)

This is an example that shows how to communicate with a shared Rust library from an Android and iOS app. The idea is to be able to share domain logic & most services (networking, database, bluetooth, etc.) using Rust and use the native SDKs for presentation and platform specific services.

# Why?

This approach gives us the best of all worlds! We prevent code duplication by using a shared library. Rust, as a highly performant and safe language is a great fit for mobile. We keep a fully native UI experience and uncomplicated access to the latest APIs of the platforms. 

# Supported

#### ✅ Functions
Calls Rust functions from Android / iOS, shows returned vale in UI.

#### ✅ Callback
Passes callback to Rust, update UI from callback with result.

#### ✅ Reactive
Subscribes in Android / iOS to events triggered by Rust and updates UI with them. 

#### ✅ Unidirectional flow
UI sends event to Rust. Rust emits to channel that is observed in Android / iOS. This allows to e.g. implement MVVM architecture writing the view models in Rust (with some glue to convert the observer-callbacks in observables/channels/SwiftUI/Jetpack compose/etc).

#### ✅ JSON
Pass serialized Objects between Android / iOS and Rust using JSON 

# Android / iOS instructions

Ensure [rustup](https://rustup.rs/) is installed. This includes [Cargo](https://doc.rust-lang.org/cargo/), so you should be able to compile & build Rust projects + install the required targets to build for Android and iOS.

List available targets: 
```
rustup target list
```

Show currently installed targets: 
```
rustup toolchain list
```

The Rust sources are [here](src)

# ![android](img/android1.png) Android instructions

These steps show how to build and run an Android app in debug mode for a 64 bits emulator. 

See [rust_swig documentation](https://dushistov.github.io/rust_swig/java-android-example.html)


### NDK

Ensure the [NDK](https://developer.android.com/ndk/guides) is installed.  

### Environment variables

```
ANDROID_TOOLCHAINS=<Directory where targets should be installed> 
ANDROID_NDK=<NDK's root directory>
```

### Add targets

```
rustup target add x86_64-linux-android
```

### Add path to linker

- Update linker path in [Cargo's config](.cargo/config).

```
[target.x86_64-linux-android]
linker = "<Directory where targets were installed (provided in environment variable)>/android-29-x86_64-4.9/bin/clang"
```

### Build

```
./gradlew assembleDebug
```

### Install

```
./gradlew installDebug
```

### Run

Ensure [adb](https://developer.android.com/studio/command-line/adb) is installed. 

then:

```
adb shell am start -n com.schuetz.rust_android_ios/com.schuetz.rust_android_ios.MainActivity
```

OR

Start the app in the emulator / device!

OR 

Run the project in Android Studio. This will build, install and run.

### Relevant configuration files

If you want to add targets or tweak the configuration, you have to edit one or more of these files:

- [App's Gradle config](app/build.gradle): This contains the apk settings (like application id, sdk version) and build steps. It builds for the provided architectures using cargo and puts the generated shared libraries (.so files) in the expected directories. If you want to build for a new target, add it [here](app/build.gradle#L45). The target's key is the [folder name where the shared library will be put](https://developer.android.com/ndk/guides/abis.html), and the value is the toolchain's name used by rustup.

- [Cargo config](.cargo/config): Contains linker paths for targets.

- [build.rs](build.rs): This is a script invoked by Cargo before everything else. For Android, it's used to tell [rust_swig](https://github.com/Dushistov/rust_swig) to generate the glue files for Java interop. If you change the app's package structure / names, you have to update this file accordingly. It also sets the import to use for the `NonNull` annotation ( `use_null_annotation_from_package`). If you're using a recent Android SDK version, you don't need to change it.

### Updating Rust

You edited something in Rust! Now you have to:

- Update [java_glue.rs.in](src/java_glue.rs.in) accordingly. This is a file [rust_swig](https://github.com/Dushistov/rust_swig) uses to generate the JNI glue. Consult [rust_swig](https://github.com/Dushistov/rust_swig) docs for syntax. 

- Build, install, run, as described above.

### Updating Kotlin/Java

The code of the Android app can be found [here](app). This is a regular Android app which you can work with normally. Just don't modify the generated JNI files and remember to update [build.rs](build.rs) as described in [Relevant configuration files](#relevant-configuration-files), if you change the package structure.

# ![iOS](img/ios1.png) iOS instructions

### App code

The iOS project is [here](ios_app).

### Add targets
```
rustup target add x86_64-apple-ios
```

### Build & run
From the project's root directory:
```
cargo build --target=x86_64-apple-ios
```
This will generate the required library files: `<project root directory>/target/mobileapp-ios.h` with the C headers and the (static) library for the target, `<project root directory>/target/<target name>/libmobcore.a`. At the moment you have to copy manually libmobcore.a into the iOS app's directory each time you update it.

With the header and library in place, you can run the iOS app. 

### Updating Rust

You edited something in Rust! Now you have to:

- Update the [C glue Rust implementation file](src/ios_c_headers.rs). Orient with the existing code. Differently to Android the glue has to be written manually, because there's no library like rust_swig.

- Build as described in "Build & run". Cargo will invoke build.rs, which uses cbindgen to generate the iOS library files.

- Copy manually the generated `<project root directory>/target/<target name>/libmobcore.a` to the iOS app project's root folder.
 
- Run!

### Updating Swift/ObjC

The code of the iOS app can be found [here](ios_app). This is a regular iOS app which you can work with normally.

# Wiki

- [Debugging guide](https://github.com/i-schuetz/rust_android_ios/wiki/Debugging)

# Contribute

1. Fork
2. Commit changes to a branch in your fork
3. Push your code and make a pull request

# Credits

Based on parts of https://github.com/Dushistov/rust_swig/tree/master/android-example and https://github.com/terhechte/rust-ios-android-example

# TODO

- Pass / return struct pointers, casting / mapping to iOS structs and Kotlin classes (partly done already for iOS).
- Avoid using global variables in iOS app.
- Automate copying of libmobcore.a or reference properly & multiple targets.
