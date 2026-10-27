# Plan B: Shortcuts + Pushover setup

The design: a Shortcuts automation on Cara's iPhone fires when a family
member's message arrives and sends one HTTPS request to Pushover's API at
emergency priority. Pushover then delivers a notification to her iPhone
and Watch and re-delivers it every 30 seconds until she taps Acknowledge.
Each re-delivery is a fresh notification, so each one taps the Watch
again. Cara wants persistent haptics and no alert sound at all, so the
setup below is silent end to end. The API request names Pushover's
vibrate-only sound (step 8), and we leave Critical Alerts off (step 5),
because a critical alert is Apple's name for a notification that plays a
sound through silent mode.

Only two things come from the Pushover website: the user key and the API
token (Part 1). Everything else happens on Cara's iPhone and Watch. The
Shortcuts automation in Part 4 is built in the Shortcuts app on her
iPhone, not on pushover.net.

## Before you start

Have Cara's iPhone and Watch, with the Watch on her wrist, and about an
hour. The Pushover account already exists, and on 2026-10-26 its
dashboard listed her iPhone as "caraiphone" with "Trial Period" beside
it. Pushover's website says the iPhone app is free for 30 days and then
needs a one-time $4.99 payment.

Settle three choices with Cara before the setup:

- Which family members trigger the alert.
- What the alert says. A fixed "Check Messages" keeps the message text
  on Apple's servers only. Showing the actual message text on the Watch
  means the text also passes through Pushover's servers. Step 8 shows
  how to set either.
- How long the repeats go on if she does not acknowledge. The setup uses
  ten minutes (expire 600). The longest Pushover allows is three hours
  (expire 10800). The question to answer is: after how long is a
  reminder no longer useful?

## Part 1: on the Pushover website, collect two strings

Do this part in Safari on Cara's iPhone, logged in to
https://pushover.net. Each string is 30 characters, so you copy them on
the iPhone and paste them into Shortcuts in Part 4 instead of typing
them.
Paste both into a note on her iPhone for now. Anyone holding both
strings can send alerts to Cara's devices, so delete the note after
Part 4.

1. User key. The dashboard shows it under the heading "Your User Key".
   Copy it into the note.
2. API token. Go to https://pushover.net/apps/build (the dashboard links
   to it as "Create an Application/API Token"). Name the application
   anything, for example family-alerts, and create it. Copy the API token
   it shows into the note.

## Part 2: set up the Pushover app on the iPhone and Watch

3. The dashboard already lists caraiphone, so the Pushover app is
   probably installed and logged in on her iPhone. Check that it is (if
   not, install it from the App Store and log in), and check that iPhone
   Settings > Notifications > Pushover has Allow Notifications on.
4. Install Pushover on the Watch: on the iPhone, open the Watch app,
   scroll to Available Apps, and install Pushover. Then, still in the
   Watch app, go to My Watch > Notifications and make sure Pushover's
   alerts are on. If the README's first free step was done, it turned
   off Watch alerts for most apps, and Pushover may be among them. If
   Pushover is listed under "Mirror iPhone Alerts From", turn it on
   there.
5. Silence settings. Leave Pushover's Critical Alerts setting off (it is
   off by default). Its one function is to force a sound through silent
   mode and Focus, and Cara wants no sound. The repeats do not depend on
   it, because Pushover's server re-sends the notification whether or not
   critical alerts are on. Two other settings let the alerts through
   without a sound:
   - So the alerts get through her Work Focus: iPhone Settings > Focus >
     Work > Apps > add Pushover to the allowed apps. An allowed app's
     notifications deliver during the Focus, silently if they carry no
     sound.
   - So the Watch never sounds: on the Watch, Settings > Sounds &
     Haptics > turn on Silent Mode. A Watch in Silent Mode alerts by
     haptic only.

## Part 3: test delivery from the website, before building anything

This test checks whether a vibrate-only emergency alert taps Cara's
Watch. No one has checked that yet. If it does not tap, alerts sent by
the Shortcut in Part 4 would not tap either, so we run this test before
building Part 4.

6. Lock Cara's iPhone, with the Watch on her wrist. On
   https://pushover.net (Jake's Mac works for this), fill in the "Push a
   Notification" form at the top of the dashboard:
   - Device: caraiphone
   - Sound: Vibrate Only
   - Priority: +2 (Emergency, repeat until acknowledged)
   - Title: Family text
   - Message: Check Messages

   Press Send Notification. This form is Pushover's manual sender: it
   sends one alert when someone presses the button and never fires on
   its own when a text arrives. Four of its boxes correspond to fields
   in step 8 (Title to title, Message to message, Priority to priority,
   Sound to sound). It has no boxes for retry and expire, so its repeats
   may come at a different interval from the 30 seconds that step 8
   sets.

   If all goes as planned, the Watch taps with no tone and taps again
   at each repeat until Cara opens the notification and presses Acknowledge
   (on the Watch or the phone). Write down what happened: did the Watch
   tap, did any sound play, and did the taps repeat?

   If the Watch does not tap, stop here and record it. A vibrate-only
   alert that does not tap the Watch means Plan B cannot give Cara a
   silent tap as written. The fallback is Pushover's quietest tone,
   which breaks the no-sound requirement, so Jake and Cara decide
   whether to accept it.

## Part 4: build the Shortcuts automation on the iPhone

7. Create the trigger. On Cara's iPhone, open the Shortcuts app, go to
   the Automation tab, tap "+", and choose Message.
   - Sender: choose the family members. If her iOS version lets the
     Sender field hold only one contact, create one automation per
     person, repeating steps 7 and 8 for each.
   - Choose "Run Immediately" (on older iOS versions, turn off "Ask
     Before Running"). Without it, the phone asks Cara to confirm each
     run, so no alert goes out until she has already noticed the
     message.
   - Tap Next.
8. Add the action. Choose New Blank Automation, tap Add Action, and
   search for "Get Contents of URL". Then:
   - Tap the URL placeholder in the action and enter
     https://api.pushover.net/1/messages.json
   - Tap the small arrow at the right edge of the action to show Method
     and Request Body. Set Method to POST and Request Body to Form.
   - Tap "Add new field" > Text eight times, and enter these keys and
     values, pasting the token and user key from the note:

     ```
     token     <the API token from step 2>
     user      <the user key from step 1>
     title     Family text
     message   Check Messages
     priority  2
     retry     30
     expire    600
     sound     vibrate
     ```

   - Tap Done.

   The labels may differ slightly between iOS versions. What the fields
   mean:
   - sound "vibrate" is the value Pushover's API documentation lists as
     "Vibrate Only", the same choice the website form offers. It plays
     no tone and asks for a vibration. Pushover also offers "none",
     listed as "None (silent)", which does not ask for a vibration, so
     we do not use it.
   - priority 2 is Pushover's emergency priority. retry is the number of
     seconds between re-deliveries, and 30 is the minimum Pushover
     accepts. expire is how many seconds Pushover keeps retrying before
     it gives up. 600 stops after ten minutes, and the maximum is 10800
     (three hours).
   - To show the actual message text instead of "Check Messages" (the
     choice in "Before you start" about what the alert says), tap the
     value of the message field and choose the Shortcut Input variable.
     While a field is being edited, Shortcuts offers its variables in a
     bar above the keyboard. The Message trigger hands the automation
     the message text as Shortcut Input.

Delete the note holding the user key and API token once every
automation is built.

## Part 5: the end-to-end test

Lock the iPhone. Jake sends Cara a text. Within a few seconds the Watch
should tap with a Pushover notification, and it should tap again every 30
seconds. Cara acknowledges by tapping the notification and pressing
Acknowledge (on the Watch or the phone), or by opening the Pushover app.
After acknowledging, the taps stop. Then run the test twice more, once
with the iPhone in silent mode and once inside her Work Focus, to confirm
that the allowed-app setting from step 5 carries the silent alert through
both.

## Recording the results

Add a dated line to the Status list in the README for each test: the
website test in Part 3 and the three runs in Part 5. For each one, write
whether the Watch tapped, whether any sound played, and whether the taps
repeated until Cara acknowledged.

## What to expect in use

- Each incoming message fires the automation once, so three texts in a
  row start three emergency alerts and Cara acknowledges each. If the
  family often sends bursts, this is the first annoyance to watch for,
  and deduplication is the main thing Plan C would add.
- The repeats stop at expire seconds even if unacknowledged. An alert
  that goes unacknowledged past that window is missed. Raise expire
  toward 10800 if ten minutes proves too short.
- The Watch-only-when-phone-locked condition from the README applies: an
  unlocked phone takes the alert itself, and since the alert is silent,
  an unlocked phone face-down on the desk gives Cara nothing at all. A
  short Auto-Lock time (iPhone Settings > Display & Brightness) keeps
  the phone locked and the alerts on her wrist.
