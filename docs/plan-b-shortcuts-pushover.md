# Plan B: Shortcuts + Pushover setup

The design: a Shortcuts automation on Cara's iPhone fires when a family
member's message arrives and sends one HTTPS request to Pushover's API at
emergency priority. Pushover then delivers a notification to her iPhone
and Watch and re-delivers it every 30 seconds until she taps Acknowledge.
Each re-delivery is a fresh notification, so each one taps the Watch
again. Cara wants persistent haptics and no alert sound at all, so the
setup below is silent end to end: the API request names Pushover's silent
sound (step 6), and we leave Critical Alerts off (step 4), because a
critical alert is Apple's name for a notification that plays a sound
through silent mode. Everything below happens on Cara's devices.

## One-time setup

1. Create a Pushover account at https://pushover.net. The dashboard shows
   a "user key" (a 30-character string). Copy it somewhere; step 6 needs
   it.
2. Install the Pushover app from the App Store on her iPhone. Then open
   the Watch app on the iPhone, scroll to Available Apps, and install
   Pushover on the Watch.
3. On https://pushover.net, choose "Create an Application/API Token".
   Name it anything (for example, family-alerts). Copy the API token it
   shows; step 6 needs it too.
4. Leave Pushover's Critical Alerts setting off (it is off by default).
   Its one function is to force a sound through silent mode and Focus,
   and Cara wants no sound. The repeats do not depend on it: Pushover's
   server re-sends the notification every retry seconds whether or not
   critical alerts are on. Two settings replace what it would have done:
   - So the alerts survive her Work Focus: iPhone Settings > Focus >
     Work > Apps > add Pushover to the allowed apps. An allowed app's
     notifications deliver during the Focus, silently if they carry no
     sound.
   - So the Watch never sounds: on the Watch, Settings > Sounds &
     Haptics > turn on Silent Mode. A Watch in Silent Mode alerts by
     haptic only.
5. Open Shortcuts on the iPhone > Automation tab > "+" > Message.
   - Sender: choose the family members. If her iOS version lets the
     Sender field hold only one contact, create one automation per
     person; the actions below are identical in each.
   - Choose "Run Immediately" (on older iOS versions, turn off "Ask
     Before Running"). Without this, the phone asks for confirmation and
     the whole design dies.
6. Add one action to the automation: Get Contents of URL.
   - URL: https://api.pushover.net/1/messages.json
   - Method: POST
   - Request Body: Form, with these fields:
     - token: the API token from step 3
     - user: the user key from step 1
     - title: Family text
     - message: Check Messages
     - priority: 2
     - retry: 30
     - expire: 600
     - sound: none
   - sound "none" is Pushover's silent option: the notification arrives
     with no tone on the phone or the Watch. The Watch's tap is governed
     by its own alert settings, not by the notification's sound, so a
     silent notification should still tap; the first test below checks
     exactly this before anything else is built on it.
   - priority 2 is Pushover's emergency priority. retry is the number of
     seconds between re-deliveries; 30 is the minimum Pushover accepts.
     expire is how many seconds Pushover keeps retrying before it gives
     up; 600 stops after ten minutes, and the maximum is 10800 (three
     hours). Pick expire by answering: after how long is a reminder no
     longer useful?
   - The Message trigger hands the automation the message text as
     Shortcut Input. Putting that variable in the message field makes the
     Watch show the actual text, at the price that the text transits
     Pushover's servers. The fixed "Check Messages" keeps the content on
     Apple's servers only. Cara chooses.

## The test

Lock the iPhone. Jake sends a text. Within a few seconds the Watch
should tap with a Pushover notification, and it should tap again every 30
seconds. Cara acknowledges by tapping the notification and pressing
Acknowledge (on the Watch or the phone), or by opening the Pushover app.
After acknowledging, the taps stop. The first run also answers the
open question from step 6: a tap with no sound anywhere means sound
"none" behaves as intended. Then run the test twice more, once with the
iPhone in silent mode and once inside her Work Focus, to confirm the
allowed-app setting from step 4 carries the silent alert through both.

## What to expect in use

- Each incoming message fires the automation once, so three texts in a
  row start three emergency alerts and Cara acknowledges each. If the
  family often sends bursts, this is the first annoyance to watch for,
  and deduplication is the main thing Plan C would add.
- The repeats stop at expire seconds even if unacknowledged. A missed
  alert after that window is simply missed; raise expire toward 10800 if
  ten minutes proves too short.
- The Watch-only-when-phone-locked condition from the README applies: an
  unlocked phone takes the alert itself, and since the alert is silent,
  an unlocked phone face-down on the desk gives Cara nothing at all. A
  short Auto-Lock time (iPhone Settings > Display & Brightness) keeps
  the phone locked and the alerts on her wrist.
