# Lab 7: Firebase registration and push notifications

This Flutter app creates an account with Firebase Authentication, saves a profile in Realtime Database, and sends a registration notification through Firebase Cloud Messaging (FCM).

## Run on Android

1. Double-click `start-push-server.cmd` in the project root and leave the console open.
2. In Android Studio, select an emulator with Google Play Services and run `lib/main.dart`. In Post Explorer, open the registration form using the profile icon.
3. Allow notifications. Enter a name, email address, phone number, and password of at least eight characters.
4. The form confirms registration once the database accepts the profile. The worker then sends a notification to the device's FCM token.
5. Use the new registration button to create another account. If saving a profile fails, retry to reuse the account that was just created.

The Spark configuration uses a worker on your computer. Both the computer and the app need internet access for push delivery. With the worker stopped, registration and profile storage still work. The worker uses your existing Firebase CLI session. If the session expires, run `npx firebase-tools@latest login` and restart the worker. Port 17877 prevents a second local worker from starting.

## Data and delivery

The Firebase project is `lab7-uib`, the Android package is `com.example.untitled`, and the database URL is `https://lab7-uib-default-rtdb.firebaseio.com`. Email/password authentication handles the password. Each user can read and write their own profile at `users/{uid}`. The profile contains these fields:

```text
name, email, phone, createdAt, fcmToken, pushRequestedAt
```

The worker watches for new profiles and repeat push requests. It sends messages through the Firebase Admin SDK and writes the delivery request status to `registrationPush/{uid}`. Only the server can write that status. A status of `sent` means FCM accepted the message. Receipt is confirmed separately on the device.

On Android 13 and later, the app requests notification permission. Incoming messages use the `registration` channel. The app uses `flutter_local_notifications` to display a foreground message. Android displays background notifications, and tapping one opens the app. The app saves refreshed FCM tokens. Use the retry push button to request permission and delivery again. If Android has blocked the permission, enable it in system settings first.

## Project files

| File | Purpose |
| --- | --- |
| `lib/main.dart` | App entry point and background handler registration |
| `lib/registration.page.dart` | Form, validation, errors, and success screen |
| `lib/services/registration_service.dart` | Authentication, profiles, token updates, and send status |
| `lib/services/push_service.dart` | Permissions, FCM events, and notification display |
| `functions/local-worker.js` | Worker that runs on the computer |
| `functions/send-registration-push.js` | Firebase Admin SDK sender |
| `functions/index.js` | Alternative Cloud Functions handler |
| `database.rules.json` | Database access rules |
| `scripts/verify-registration.cjs` | Checks test records without printing tokens |
| `docs/evidence/` | Android verification screenshots |

The Cloud Functions handler is an alternative deployment option that requires a Blaze plan. This project was tested with the local worker. Stop that worker before deploying the cloud handler to avoid duplicate notifications.

## Verification on November 5, 2026

Static analysis passed. All three Flutter form tests and all three worker tests passed. Run `flutter analyze`, `flutter test`, and `npm test` in the `functions` directory for the worker tests.

A debug APK was installed on a Medium Phone API 36 x64 emulator with Google Play Services. Registration created test accounts matching `lab7-test-...@example.com`. Cloud profiles and FCM tokens were verified. Push arrived in both foreground and background, tapping the notification reopened the app, and unauthenticated database access returned HTTP 401. A delivery status race was fixed so a late server acknowledgement preserves the device's receipt confirmation.

[Registration form](docs/evidence/registration-form.png) | [Foreground notification](docs/evidence/registration-push-foreground.png) | [Background notification](docs/evidence/push-background.png)

The emulator build outputs `build/app/outputs/flutter-apk/app-debug.apk`. For a physical Android phone, run `flutter build apk --debug`.
