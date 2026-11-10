

==================== FILE: PRIVACY_POLICY.md ====================

# ROSIE AI Privacy Policy

*Last updated: [DATE]. Replace the bracketed items with your company details before publishing, and have a lawyer review it for your regions.*

ROSIE AI ("Rosie", "the app") is a voice assistant for Android. This policy explains what we collect, why, and how you stay in control.

## What the app accesses, and only with your permission
| Permission | Why | When |
|---|---|---|
| Microphone | Hear your voice commands | When you tap the mic, or while you have turned on "Hey Rosie" mode (a notification is always visible) |
| Contacts | Find a name when you say "Call Mom" | Only when you ask to call or message someone |
| Phone / call management | Answer, reject or end a call by voice | Only when you use those commands |
| Calendar | Read today's schedule | Only when you ask for your schedule |
| Notifications access | Read your notifications aloud | Only if you enable it in Android settings and ask |
| Notifications | Reminders and optional friendly messages | You can turn these off |
| Modify system settings, Do Not Disturb access | Brightness and Do Not Disturb | Only when you use those commands |

You can withdraw any permission at any time in Android settings.

## What we do NOT do
- We never secretly listen or record. Audio is processed by your phone's speech recognition service and is not saved by Rosie.
- We never record calls or read private conversations.
- We never ask for or store passwords or banking PINs.
- We do not sell your data.

## What is stored, and where
- **On your phone (encrypted settings or app storage):** your settings, notes, tasks, chat history and smart-home details you enter.
- **In your private cloud account (Firebase):** an account ID, your voice/language/personality settings, and facts you explicitly ask Rosie to remember ("remember that...").
- **Sent to our server to answer questions:** the text of your recent messages, your remembered facts, and any image or PDF you choose to attach. This is forwarded to an AI provider to produce a reply, and is not used by us for advertising. [Describe your retention period, e.g. "We do not keep message content after the reply is generated."]
- **Speech recognition and text-to-speech** are provided by your phone's installed services (for example Google). Their own privacy policies apply.
- **Payments** are handled entirely by Google Play. We never see your card details.

## Your choices and deletion
- Settings > **Delete all my data** removes your notes, tasks, chat history, cloud memories and your account.
- You can also email [CONTACT EMAIL] to request deletion.
- Turn off the microphone or "Hey Rosie" mode at any time from Settings or the notification.

## Security
Traffic between the app and our server uses HTTPS. On-device settings are encrypted. AI provider keys are stored only on our server.

## Children
Rosie is not directed to children under 13 [or the minimum age in your region].

## Changes and contact
We will post updates here. Contact: [COMPANY NAME], [ADDRESS], [CONTACT EMAIL].


==================== FILE: README.md ====================

# ROSIE AI: Android voice assistant

Native Kotlin + Jetpack Compose app, with a small Node backend that keeps the AI key off the phone.

> **Status:** complete source project. It has **not been compiled or run on a device yet** (no Android SDK in the environment it was written in). Open it in Android Studio, sync, and fix any small compile issues first. Then test on a real phone, because voice, flashlight, calls and sensors can't be verified on an emulator.

## Setup (about 30 minutes)

1. **Android Studio** (Koala or newer): open the `rosie-ai` folder, let Gradle sync.
2. **Firebase:** create a project, enable *Authentication > Anonymous*, create a *Firestore* database, paste `backend/firestore.rules`, add an Android app with package `com.rosie.ai`, download `google-services.json` into `app/`, then uncomment the `google-services` plugin line in `app/build.gradle.kts`.
3. **Backend:** `cd backend && npm install`, then run with
   `ANTHROPIC_API_KEY=... GOOGLE_APPLICATION_CREDENTIALS=serviceAccount.json node server.js`
   (set `ENABLE_WEB_SEARCH=1` for live web answers). Deploy anywhere that serves HTTPS (Cloud Run, Render, Fly...). Put the URL in `BACKEND_URL` in `app/build.gradle.kts`.
4. **Billing:** in Play Console create a subscription with product ID `rosie_premium`. Billing only works from a Play-distributed build (internal testing track is fine).
5. Replace the placeholder launcher icon and fill in `PRIVACY_POLICY.md`.

## What every voice command does

| You say | What happens | Notes |
|---|---|---|
| Open WhatsApp / Instagram / Snapchat / YouTube / any app | Launches the app (finds any installed app by name) | Says so if it isn't installed |
| Open Settings / Camera / Gallery | Opens them | |
| Flashlight on/off | Camera2 torch | "No flashlight" message if the hardware is missing |
| Volume up/down, "set volume to 40" | AudioManager | |
| Set brightness to 50% | Writes system brightness | Asks you to grant "Modify system settings" once |
| Bluetooth / Wi-Fi on/off | Android 12 and below: toggled directly (Wi-Fi up to Android 9). Newer: opens the system panel | Android blocks apps from doing this on newer versions |
| Do Not Disturb on/off | NotificationManager | Asks for DND access once |
| Set an alarm for 7:30 am | Creates it in your clock app | |
| Remind me to X in 10 minutes | WorkManager notification | |
| Call [name] / [number] | Looks up contacts, **asks you to confirm**, then dials | Falls back to the dialer if CALL_PHONE is declined |
| Send a WhatsApp message to [name] saying ... | **Confirms**, opens the chat with the text filled in | You tap send; Android doesn't allow silent sending |
| Answer my call / Reject this call / End my call | TelecomManager (Android 9+) | Needs the Phone permission |
| Read my notifications | Reads the latest few | Needs Notification Access (you enable it) |
| Turn on/off the light, fan, plug | Home Assistant REST API | Needs your Home Assistant hub (Settings) |
| Search X on YouTube / Navigate to X / Open example.com / Play music | Intents | |
| Take a note..., Add task..., Show my tasks | Stored on the phone | |
| What's my schedule | Reads today's calendar | Asks for Calendar permission |
| Battery / storage | System info | |
| Share ... | Android share sheet | |
| Remember that ... | Saves to your cloud memory (your explicit approval) | |
| Anything else | Sent to the AI brain (questions, maths, coding, translation, study, writing, etc.) | Can attach an image or PDF with the paperclip |

"Hey Rosie" always-listening mode: Settings > Microphone > turn on. It runs as a foreground service with a visible notification and a Stop button.

## Honest limitations
- **Wake word** uses Android's speech recognizer in a loop. It works, but uses more battery than a dedicated wake-word engine and some phones play a beep. For production, consider Picovoice Porcupine with a custom "Hey Rosie" model. Android 10+ also limits starting apps from the background, so some commands in always-listening mode may need Rosie to be open or have the "Display over other apps" permission.
- **Incoming calls:** Rosie announces a ringing call and answers/rejects through TelecomManager. A full call-screening role (`CallScreeningService`) and a custom dialer are not implemented.
- **Screen understanding and accessibility actions** (control other apps by tapping) are **not included**. They need an AccessibilityService, with explicit user setup and Play Store policy review. The camera assistant is available via the image attach button (pick a photo), not a live camera view.
- **Authentication:** anonymous Firebase sign-in is used so everything works immediately. Add email or Google sign-in before launch so users keep their data across devices.
- **Premium:** the app grants premium locally after a Play purchase. For real enforcement, verify purchase tokens on your server with the Google Play Developer API and set `users/{uid}.premium` (the backend already reads it).
- **Voice quality:** the voice is Android's text-to-speech (pitch/speed adjustable). For a more natural, expressive voice, swap `TtsManager` for a neural TTS service.
- **Weather, birthday reminders, study timetable:** ask Rosie (web search gives weather), or say "remind me...". There is no dedicated weather or birthday screen yet.
- Wi-Fi/Bluetooth/Do Not Disturb/brightness behaviour differs by manufacturer. Test on several devices (Pixel, Samsung, Xiaomi, OnePlus) and Android versions (8 to 15) before launch.

## Project layout
```
app/src/main/java/com/rosie/ai/
  MainActivity.kt          permissions, navigation
  RosieApp.kt              notification channels
  data/                    Prefs (encrypted), Persona (cute phrases), AiClient, CloudRepo (Firebase), TextStore
  device/                  CommandRouter (all commands), SmartHome, Workers, RosieNotificationListener
  voice/                   SpeechManager, TtsManager, WakeWordService
  billing/                 BillingManager (Google Play)
  ui/                      Theme, RosieScreen (orb, waveform, chat), SettingsScreen, RosieViewModel
backend/                   server.js (AI proxy), firestore.rules
PRIVACY_POLICY.md
```


==================== FILE: app/build.gradle.kts ====================

plugins {
    id("com.android.application")
    id("org.jetbrains.kotlin.android")
    id("org.jetbrains.kotlin.plugin.compose")
    // Enable after adding app/google-services.json from your Firebase project:
    // id("com.google.gms.google-services")
}

android {
    namespace = "com.rosie.ai"
    compileSdk = 34

    defaultConfig {
        applicationId = "com.rosie.ai"
        minSdk = 26
        targetSdk = 34
        versionCode = 1
        versionName = "1.0.0"
        // Backend proxy URL. The AI provider key lives ONLY on the server.
        buildConfigField("String", "BACKEND_URL", "\"https://your-backend.example.com\"")
    }

    buildTypes {
        release {
            isMinifyEnabled = true
            proguardFiles(getDefaultProguardFile("proguard-android-optimize.txt"))
        }
    }
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }
    kotlinOptions { jvmTarget = "17" }
    buildFeatures {
        compose = true
        buildConfig = true
    }
}

dependencies {
    val composeBom = platform("androidx.compose:compose-bom:2024.09.00")
    implementation(composeBom)
    implementation("androidx.compose.ui:ui")
    implementation("androidx.compose.material3:material3")
    implementation("androidx.compose.material:material-icons-extended")
    implementation("androidx.compose.ui:ui-tooling-preview")
    implementation("androidx.activity:activity-compose:1.9.2")
    implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.8.6")
    implementation("androidx.lifecycle:lifecycle-runtime-compose:2.8.6")
    implementation("androidx.core:core-ktx:1.13.1")
    implementation("androidx.work:work-runtime-ktx:2.9.1")
    implementation("androidx.datastore:datastore-preferences:1.1.1")
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.8.1")
    implementation("com.squareup.okhttp3:okhttp:4.12.0")
    implementation("androidx.security:security-crypto:1.1.0-alpha06")
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-play-services:1.8.1")

    // Firebase (Auth + Firestore for user-approved preferences)
    implementation(platform("com.google.firebase:firebase-bom:33.3.0"))
    implementation("com.google.firebase:firebase-auth-ktx")
    implementation("com.google.firebase:firebase-firestore-ktx")

    // Google Play Billing
    implementation("com.android.billingclient:billing-ktx:7.0.0")

    debugImplementation("androidx.compose.ui:ui-tooling")
}


==================== FILE: app/src/main/AndroidManifest.xml ====================

<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <!-- Every permission below is requested at runtime, only when the matching feature is used. -->
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.RECORD_AUDIO" />
    <uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE_MICROPHONE" />
    <uses-permission android:name="android.permission.CALL_PHONE" />
    <uses-permission android:name="android.permission.ANSWER_PHONE_CALLS" />
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />
    <uses-permission android:name="android.permission.READ_CONTACTS" />
    <uses-permission android:name="android.permission.READ_CALENDAR" />
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.SET_ALARM" />
    <uses-permission android:name="android.permission.WRITE_SETTINGS" tools:ignore="ProtectedPermissions" />
    <uses-permission android:name="android.permission.ACCESS_NOTIFICATION_POLICY" />
    <uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />
    <uses-permission android:name="android.permission.BLUETOOTH" android:maxSdkVersion="30" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />

    <uses-feature android:name="android.hardware.microphone" android:required="false" />
    <uses-feature android:name="android.hardware.camera.flash" android:required="false" />
    <uses-feature android:name="android.hardware.telephony" android:required="false" />

    <queries>
        <intent>
            <action android:name="android.intent.action.MAIN" />
            <category android:name="android.intent.category.LAUNCHER" />
        </intent>
        <intent><action android:name="android.speech.RecognitionService" /></intent>
        <intent><action android:name="android.intent.action.TTS_SERVICE" /></intent>
    </queries>

    <application
        android:name=".RosieApp"
        android:allowBackup="false"
        android:icon="@android:drawable/ic_btn_speak_now"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/Theme.Rosie">

        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:windowSoftInputMode="adjustResize">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <service
            android:name=".voice.WakeWordService"
            android:exported="false"
            android:foregroundServiceType="microphone" />

        <service
            android:name=".device.RosieNotificationListener"
            android:exported="true"
            android:label="@string/app_name"
            android:permission="android.permission.BIND_NOTIFICATION_LISTENER_SERVICE">
            <intent-filter>
                <action android:name="android.service.notification.NotificationListenerService" />
            </intent-filter>
        </service>
    </application>
</manifest>


==================== FILE: app/src/main/java/com/rosie/ai/MainActivity.kt ====================

package com.rosie.ai

import android.Manifest
import android.content.pm.PackageManager
import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.rememberLauncherForActivityResult
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.activity.result.contract.ActivityResultContracts
import androidx.activity.viewModels
import androidx.compose.foundation.layout.Column
import androidx.compose.material3.AlertDialog
import androidx.compose.material3.Text
import androidx.compose.material3.TextButton
import androidx.compose.runtime.*
import androidx.core.content.ContextCompat
import com.rosie.ai.billing.BillingManager
import com.rosie.ai.device.Scheduler
import com.rosie.ai.ui.RosieScreen
import com.rosie.ai.ui.RosieTheme
import com.rosie.ai.ui.RosieViewModel
import com.rosie.ai.ui.SettingsScreen

class MainActivity : ComponentActivity() {
    private val vm: RosieViewModel by viewModels()
    private var onPerm: ((Boolean) -> Unit)? = null
    private val permLauncher = registerForActivityResult(ActivityResultContracts.RequestPermission()) { ok ->
        onPerm?.invoke(ok); onPerm = null
    }
    private lateinit var billing: BillingManager
    private var billingTick by mutableIntStateOf(0)

    private fun ensure(perm: String, result: (Boolean) -> Unit) {
        if (ContextCompat.checkSelfPermission(this, perm) == PackageManager.PERMISSION_GRANTED) result(true)
        else { onPerm = result; permLauncher.launch(perm) }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        billing = BillingManager(this, vm.prefs) { billingTick++ }
        billing.connect()
        Scheduler.applyGreetingSchedule(this, vm.prefs)

        setContent {
            RosieTheme(vm.themeMode) { dark ->
                var screen by remember { mutableStateOf("home") }
                var showMicRationale by remember { mutableStateOf(false) }
                val picker = rememberLauncherForActivityResult(ActivityResultContracts.GetContent()) { uri -> uri?.let(vm::attach) }

                // When a command needs a runtime permission, ask for it, then Rosie retries the command.
                LaunchedEffect(vm.permissionRequest) {
                    vm.permissionRequest?.let { p -> ensure(p) { vm.onPermissionResult(it) } }
                }

                fun micTapped() {
                    if (vm.status == com.rosie.ai.ui.Status.LISTENING) { vm.stopListening(); return }
                    if (ContextCompat.checkSelfPermission(this, Manifest.permission.RECORD_AUDIO) == PackageManager.PERMISSION_GRANTED) vm.startListening()
                    else showMicRationale = true
                }

                if (screen == "settings") {
                    key(billingTick) { SettingsScreen(vm, dark, billing, ::ensure) { screen = "home" } }
                } else {
                    RosieScreen(vm, dark, onMic = ::micTapped, onAttach = { picker.launch("*/*") }, onSettings = { screen = "settings" })
                }

                if (showMicRationale) AlertDialog(
                    onDismissRequest = { showMicRationale = false },
                    title = { Text("Allow microphone?") },
                    text = { Text("Rosie needs the microphone to hear your voice commands. She only listens while you tap the mic (or if you turn on Hey Rosie mode), and never records or stores audio.") },
                    confirmButton = {
                        TextButton(onClick = { showMicRationale = false; ensure(Manifest.permission.RECORD_AUDIO) { if (it) vm.startListening() } }) { Text("Continue") }
                    },
                    dismissButton = { TextButton(onClick = { showMicRationale = false }) { Text("Not now") } }
                )
            }
        }
    }
}


==================== FILE: app/src/main/java/com/rosie/ai/RosieApp.kt ====================

package com.rosie.ai

import android.app.Application
import android.app.NotificationChannel
import android.app.NotificationManager
import android.os.Build

class RosieApp : Application() {
    override fun onCreate() {
        super.onCreate()
        if (Build.VERSION.SDK_INT >= 26) {
            val nm = getSystemService(Notifi
