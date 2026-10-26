# Plan B: Shortcuts + Pushover setup

The design: a Shortcuts automation on Cara's iPhone fires when a family
member's message arrives and sends one HTTPS request to Pushover's API at
emergency priority. Pushover then delivers a notification to her iPhone
and Watch and re-delivers it every 30 seconds until she taps Acknowledge.
Each re-delivery is a fresh notification, so each one taps the Watch
again. Everything below happens on Cara's devices.

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
4. In the Pushover iPhone app: Settings > enable Critical Alerts for
   emergency-priority messages. iOS asks once for permission. This is
   what lets the alert sound through silent mode and Focus; it is off by
   default.
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
After acknowledging, the taps stop. Run the test once more with the
iPhone in silent mode and once inside her Work Focus, to confirm the
Critical Alerts setting from step 4 carries the alert through both.

## What to expect in use

- Each incoming message fires the automation once, so three texts in a
  row start three emergency alerts and Cara acknowledges each. If the
  family often sends bursts, this is the first annoyance to watch for,
  and deduplication is the main thing Plan C would add.
- The repeats stop at expire seconds even if unacknowledged. A missed
  alert after that window is simply missed; raise expire toward 10800 if
  ten minutes proves too short.
- The Watch-only-when-phone-locked condition from the README applies: an
  unlocked phone takes the alert itself, though with Critical Alerts on,
  the phone sounds even in silent mode, which may be signal enough.
