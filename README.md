# basic-flutter-test
This project demonstrates, how we can use git actions to make automated builds for iOS, Android & Web using flutter.

In the home directory, we need to create `.github/workflows/build.yaml` file.

```
on:
  push:
    branches:
      - main

name: Build and Release Apps
jobs:
  build:
    name: Build Apps
    runs-on: macos-latest
    steps:
      - name: Export Release Timestamp
        run: echo "APP_VERSION=v$(date +'%Y.%m.%d.%H.%M.%S')" >> $GITHUB_ENV
      - name: Checkout repository
        uses: actions/checkout@v1
      - name: Set up Java
        uses: actions/setup-java@v2
        with:
          java-version: '12.x'
          distribution: 'zulu'
      - name: Setup Android SDK
        uses: android-actions/setup-android@v2

      - name: Set up Flutter
        uses: subosito/flutter-action@v1
        with:
          flutter-version : '2.5.3'
          channel: 'stable'
      - run: flutter  config --enable-web
      - name: Install pub Dependencies
        run: flutter pub get
      - name: Run Tests
        run: flutter test
      - name: Build Android App
        run:  flutter build apk --no-sound-null-safety -v
      - name: Build iOS App
        run: |
          flutter build ios --no-codesign
          cd build/ios/iphoneos
          mkdir Payload
          cd Payload
          ln -s ../Runner.app
          cd ..
          zip -r app.ipa Payload
      - name: Build Web App
        run: |
          flutter build web
          cd build/web
          zip -r web-app.zip .
      - name: Release Apps
        uses: ncipollo/release-action@v1
        with:
          tag: ${{ github.run_number }}
          name: ${{ env.APP_VERSION }}
          token: ${{ secrets.TOKEN }}
          artifacts: 'build/app/outputs/apk/release/*.apk,build/ios/iphoneos/app.ipa,build/web/web-app.zip'
```

> This will determine the type of your remote machine.
``` runs-on: macos-latest```
> clone the code using the below action.

```
- name: Checkout repository
        uses: actions/checkout@v1
```

>setup java on your machine with the action below

```
- name: Set up Java
        uses: actions/setup-java@v2
        with:
          java-version: '12.x'
          distribution: 'zulu'
```
>This action sets up the flutter on your remote machine

```
- name: Set up Flutter
        uses: subosito/flutter-action@v1
        with:
          flutter-version : '2.5.3'
          channel: 'stable'
```

and after that we've standard flutter commands to generate builds.

This action will create release build of apk, ipa and web archive in the release section of the github repository.
```
- name: Release Apps
        uses: ncipollo/release-action@v1
        with:
          tag: ${{ github.run_number }}
          name: ${{ env.APP_VERSION }}
          token: ${{ secrets.TOKEN }}
          artifacts: 'build/app/outputs/apk/release/*.apk,build/ios/iphoneos/app.ipa,build/web/web-app.zip'
```
