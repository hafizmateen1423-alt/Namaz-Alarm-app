name: Build APK

on:
  push:
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 17

      - uses: gradle/actions/setup-gradle@v4
        with:
          gradle-version: 8.9

      - name: Create project
        run: |
          mkdir -p app/src/main/java/com/example/namazalarm
          cat > settings.gradle.kts <<'END'
          pluginManagement {
              repositories {
                  google()
                  mavenCentral()
                  gradlePluginPortal()
              }
          }
          dependencyResolutionManagement {
              repositories {
                  google()
                  mavenCentral()
              }
          }
          rootProject.name = "NamazAlarm"
          include(":app")
          END
          cat > build.gradle.kts <<'END'
          plugins {
              id("com.android.application") version "8.5.2" apply false
              id("org.jetbrains.kotlin.android") version "1.9.24" apply false
          }
          END
          cat > app/build.gradle.kts <<'END'
          plugins {
              id("com.android.application")
              id("org.jetbrains.kotlin.android")
          }
          android {
              namespace = "com.example.namazalarm"
              compileSdk = 34
              defaultConfig {
                  applicationId = "com.example.namazalarm"
                  minSdk = 24
                  targetSdk = 34
                  versionCode = 1
                  versionName = "1.0"
              }
              compileOptions {
                  sourceCompatibility = JavaVersion.VERSION_17
                  targetCompatibility = JavaVersion.VERSION_17
              }
              kotlinOptions {
                  jvmTarget = "17"
              }
          }
          END
          cat > app/src/main/AndroidManifest.xml <<'END'
          <manifest xmlns:android="http://schemas.android.com/apk/res/android">
              <application
                  android:label="Namaz Alarm"
                  android:theme="@android:style/Theme.Material.Light.NoActionBar">
                  <activity
                      android:name=".MainActivity"
                      android:exported="true">
                      <intent-filter>
                          <action android:name="android.intent.action.MAIN" />
                          <category android:name="android.intent.category.LAUNCHER" />
                      </intent-filter>
                  </activity>
              </application>
          </manifest>
          END
          cat > app/src/main/java/com/example/namazalarm/MainActivity.kt <<'END'
          package com.example.namazalarm

          import android.app.Activity
          import android.os.Bundle
          import android.view.Gravity
          import android.widget.TextView

          class MainActivity : Activity() {
              override fun onCreate(savedInstanceState: Bundle?) {
                  super.onCreate(savedInstanceState)
                  val t = TextView(this)
                  t.text = "Namaz Alarm"
                  t.textSize = 30f
                  t.gravity = Gravity.CENTER
                  setContentView(t)
              }
          }
          END

      - name: Build APK
        run: gradle assembleDebug --no-daemon

      - uses: actions/upload-artifact@v4
        with:
          name: NamazAlarm-APK
          path: app/build/outputs/apk/debug/*.apk
