# Plan C: a bespoke iOS + watchOS app

We build this only if Plan B fails Cara in practice. The failures worth
building for: bursts of family texts produce an annoying pile of separate
emergency alerts (our app can collapse them into one), the Acknowledge
flow in Pushover costs too many taps, or sending alert text through
Pushover's servers is unwanted. The failure NOT worth building for: "the
taps still are not one long buzz." Apple's limits bind our code the same
way they bind Pushover's, as the next section spells out.

## What watchOS allows third-party code to do, with sources

- A notification delivered to a third-party Watch app plays one system
  haptic. The app does not choose its length or strength.
- An app can play haptics on demand (WKInterfaceDevice.current().play)
  only while it is the app on screen. A foreground app can therefore loop
  haptics on a timer, which gives a genuinely long pattern, but only after
  Cara has already raised her wrist and opened the app, at which point the
  alert has done its job.
- The one background path to repeated haptics is an extended runtime
  session of the "smart alarm" type (WKExtendedRuntimeSession). The app
  must schedule it in advance for a known clock time (start(at:), up to
  36 hours ahead), the session runs at most 30 minutes, and only the
  smart-alarm type can begin while the app is in the background. An
  incoming message has no known clock time, so this path cannot alert on
  messages. Sources: Apple, "Using extended runtime sessions"
  (https://developer.apple.com/documentation/watchkit/using-extended-runtime-sessions)
  and the accepted answers in developer-forum threads 682429 and 760402
  (https://developer.apple.com/forums/thread/682429,
  https://developer.apple.com/forums/thread/760402).
- Critical Alerts (notifications that force a sound through silent mode
  and Focus) require a per-app entitlement that Apple grants on written
  application, normally to medical, security, and public-safety apps. A
  personal family-texts app is unlikely to receive it. For this project
  the entitlement does not matter either way: Cara wants no sound, so we
  would not use it even if we had it.

So Plan C delivers the same form of signal as Plan B, a train of standard
taps until acknowledged, with better behavior around bursts and
acknowledgment.

## Architecture

Three pieces, with the same Shortcuts automation from Plan B as the
trigger:

1. An iOS app that defines one App Intent, "Alert Watch". App Intents
   appear as actions in Shortcuts, so the automation's single action
   becomes "Alert Watch" instead of the Pushover web request.
2. The intent schedules a burst of local notifications through
   UNUserNotificationCenter: for example 40 notifications at 15-second
   intervals (UNTimeIntervalNotificationTrigger), which covers ten
   minutes. If an unacknowledged burst is already pending, the intent
   does nothing, and that single check is the deduplication Plan B lacks.
   When the iPhone is locked, iOS forwards each notification to the
   Watch, and each forwards as one tap.
3. A watchOS companion app whose notification interface shows one button,
   Acknowledge. Tapping it sends one WatchConnectivity message to the
   iPhone app, which calls removeAllPendingNotificationRequests, and the
   burst stops. The Watch app's main screen can also loop
   WKInterfaceDevice haptics while open, for a long pattern once she has
   seen the alert.

No server, no accounts, nothing leaves the two devices.

## Developer logistics, for a first Watch project

- Hardware and tools: a Mac running current Xcode. Xcode includes the iOS
  and watchOS SDKs and simulators, and the simulators are enough for all
  of the logic except the feel of the haptics, which only the physical
  Watch shows. Language Swift, UI in SwiftUI.
- Project shape: create an iOS App project, then File > New > Target >
  watchOS > "Watch App for iOS App". That yields the two-target project
  the architecture above needs.
- Accounts: a free Apple ID lets Xcode install builds on your own devices
  over cable or Wi-Fi, but the signature expires after 7 days and the app
  stops launching until reinstalled from Xcode. For an app Cara wears
  daily, that cycle fails the first week she is away from your Mac. The
  Apple Developer Program (99 USD/year) fixes it: TestFlight installs on
  her phone and Watch over the air and builds last 90 days.
- Reading order: Apple's SwiftUI tutorials (the watchOS sections), then
  the UserNotifications framework (scheduling and canceling local
  notifications), then App Intents (exposing the Shortcut action), then
  WatchConnectivity (the acknowledge message).
- Effort estimate: the code is small, on the order of a few hundred
  lines across the two targets. For a first Watch project, expect the
  signing, entitlements, notification-permission prompts, and two-target
  setup to take longer than the code.

## Decision rule

Run Plan B for two weeks of ordinary work. Build Plan C only if one of
the named failures occurred: burst annoyance, acknowledge friction, or
the privacy objection. If Plan B simply worked, this document stays a
record of why we stopped.
