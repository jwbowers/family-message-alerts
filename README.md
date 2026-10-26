# family-message-alerts

Cara misses family iMessages while she is focused on work. The Apple Watch
tap for an incoming message is a single short pulse, and she misses that
pulse more often than she notices it. She wants to stop missing the
messages. This project sets up her iPhone and Watch so that a message from
a family member produces a signal that repeats until she acknowledges it.
The signal is haptic only: Cara wants no alert sound anywhere, so every
design below is set up silent, and anything that exists to force a sound
through silent mode (Emergency Bypass, Critical Alerts) stays off.
Jake does the setup and, if it comes to that, the programming.

## Two platform facts that shape every design

Apple does not let third-party code read incoming iMessages. No app we
install or write can see that a message arrived, who sent it, or what it
said. The one hook Apple exposes is a personal automation in the Shortcuts
app on the receiving iPhone: its "Message" trigger fires when a message
arrives from a sender the automation names, and the automation can run its
actions immediately, without asking. So every design below starts from a
Shortcuts automation on Cara's iPhone, and the designs differ only in what
that automation does next.

Apple also does not let third-party code play a long continuous haptic on a
Watch whose screen the app does not currently occupy. A notification
arriving at a third-party Watch app produces one system tap, the same
length as the Messages tap. An app can play repeated haptics only while it
is on screen, or inside a "smart alarm" session that the app must schedule
in advance for a known clock time, and an unpredictable incoming message
never has a known clock time. (Apple's extended-runtime-session
documentation and the developer forums confirm both limits; links are in
docs/plan-c-watch-app.md.) So "a long and strong haptic" cannot take the
form of one sustained buzz. The achievable form is a train of taps that
repeats until Cara acknowledges it. Every plan below aims at that form.

One delivery condition applies to all three plans: the Watch receives a
notification only when the iPhone is locked and the Watch is unlocked and
on her wrist. If her phone lies unlocked on the desk, iOS sends every
alert to the phone and the Watch stays silent. If Cara works with her
phone unlocked beside her, no Watch-side design can fire, and the fix is a
shorter auto-lock time (iPhone Settings > Display & Brightness >
Auto-Lock).

## The plans, in order of effort

We try the plans in order, because each later plan costs more hours and
adds less signal than the one before it.

### Plan A: settings only (about ten minutes)

These settings make the existing tap stronger and repeat it.

1. On the Watch: Settings > Sounds & Haptics > set Haptic Strength to
   maximum and turn on Prominent Haptics. Prominent adds an extra,
   stronger pre-tap before each alert.
2. On the iPhone: Settings > Notifications > Messages > Repeat Alerts >
   set to 10. The iPhone then re-announces an unread message every two
   minutes, up to ten times. Apple documents the repeats for the iPhone;
   we have not yet tested whether each repeat also re-taps the Watch. Test:
   lock the phone, send a message, do not touch anything, and wait two
   minutes for the second tap.
3. (Dropped 2026-10-26: Emergency Bypass exists to force a sound through
   silent mode, and Cara wants no sound. If it was turned on during the
   Plan A test, turn it back off: Contacts > person > Text Tone >
   Emergency Bypass.)
4. If Cara uses a Work Focus: Settings > Focus > Work > add the family
   contacts to the allowed people.

Plan A keeps every alert a single short tap and spaces the repeats two
minutes apart, with at most ten of them. If that is still too easy to
miss, move to Plan B.

Tested 2026-10-26: Cara felt none of the taps. That outcome is consistent
with two different failures, and they call for different fixes, so before
Plan B we distinguish them with one test. Jake sends a text while Cara's
phone is locked and the Watch is on her wrist, and Cara watches the Watch
face. If the notification appears on the screen and she felt nothing, the
taps arrive but are too weak to feel; a snugger band helps (a Watch worn
loose transmits the tap to air, not skin), and Plan B helps, because a tap
every 30 seconds until acknowledged gives many chances for one to land.
If no notification appears on the Watch at all, the taps never arrive, and
Plan B inherits the same failure; the settings to check are Wrist
Detection (Watch > Settings > Passcode, must be on, or iOS sends nothing
to the Watch), whether the Watch is passcode-locked off the wrist, and
whether the phone was in fact locked.

### Plan B: Shortcuts + Pushover (recommended; no code; about an hour)

Pushover is a notification relay: an iPhone app with a Watch app, plus an
HTTPS API that anything can call. Its "emergency priority" re-sends a
notification every 30 seconds (or any longer interval) until the recipient
taps Acknowledge. The Shortcuts automation on Cara's phone calls the API
when a family message arrives; her Watch then taps every 30 seconds until
she acknowledges, and with the API's silent sound option plus the Watch's
own Silent Mode, every one of those alerts is a tap with no tone. Her
Work Focus lets them through because the Pushover app joins the Focus's
allowed apps, which delivers silently; Pushover also offers Critical
Alerts, but their one function is forcing a sound through silent mode, so
we leave that off. Cost: a 30-day free trial, then a one-time license of
about five dollars per platform.

Setup steps are in docs/plan-b-shortcuts-pushover.md.

### Plan C: a bespoke iOS + watchOS app

Building our own app buys deduplication (several rapid messages collapse
into one alert), a one-tap acknowledge screen on the Watch, and no third
party seeing the alerts. It does not buy a continuous buzz, because the
platform limit above binds our code too. The architecture,
the platform limits restated with sources, and the developer-account
logistics are in docs/plan-c-watch-app.md. We build it only if Plan B
fails Cara in practice.

## Status

- 2026-10-26: plans drafted.
- 2026-10-26: Plan A tested and failed. Cara felt none of the taps. Next
  step: the one-text delivery test described at the end of Plan A, to
  learn whether taps arrive too weak (band fit, then Plan B) or never
  arrive (fix Wrist Detection and locking first, since Plan B needs
  delivery working too). Then the Plan B setup, two weeks of ordinary
  use, and only then a decision about Plan C.
- 2026-10-26: requirement sharpened: no alert sound at all, haptics only.
  Plan B adjusted (sound "none" in the API call, Critical Alerts left
  off, Focus passage via allowed app, Watch in Silent Mode) and Plan A's
  Emergency Bypass step dropped.
