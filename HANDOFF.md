# Handoff

2026-10-26. Repository: jwbowers/family-message-alerts. For the session
that continues this project with no memory of the conversation that
started it.

## What we were doing and why

Cara misses family iMessages while she is focused on work; the Watch tap
for a message is a single short pulse and she misses it more often than
she notices it. Jake wants her Watch to give a persistent, haptic-only
signal when a family message arrives, repeating until she acknowledges
it, with no alert sound anywhere. Jake does the device setup and, if a
bespoke app becomes necessary, the programming; it would be his first
watchOS project. The README and the two files in docs/ hold the full
plan; this file records what the plan rests on.

## Decisions, each with its reason

1. Every design starts from a Shortcuts personal automation on Cara's
   iPhone (the "Message" trigger, filtered to family senders, set to run
   immediately), because Apple exposes no other hook: no third-party app
   can read incoming iMessages.
2. The target signal is a train of taps repeating until acknowledged, not
   one long buzz, because watchOS gives third-party code no background
   path to a continuous haptic. A notification to a Watch app plays one
   system tap of fixed length; on-demand haptics (WKInterfaceDevice.play)
   work only with the app on screen; and the one background path, a
   smart-alarm extended runtime session, must be scheduled for a known
   clock time in advance, which an incoming message never has. Sources
   are linked in docs/plan-c-watch-app.md.
3. Three plans in order of effort: A (settings), B (Shortcuts automation
   POSTs to Pushover's API at emergency priority), C (bespoke iOS +
   watchOS app). Later plans cost more hours, so each is tried only
   after the one before it fails.
4. Plan A was tested 2026-10-26 and failed: Cara felt none of the taps.
5. Plan B is the recommendation. Pushover's emergency priority
   (priority=2 with retry=30, expire=600 in the API call) re-delivers
   the notification every 30 seconds until she taps Acknowledge, and
   each re-delivery taps the Watch again.
6. Everything is set up silent (decision 2026-10-26, when Jake said no
   alert sound at all): the API call carries sound=none, Pushover's
   Critical Alerts setting stays off because its one function is forcing
   a sound through silent mode, the Work Focus passes the alerts because
   Pushover joins its allowed apps, the Watch sits in Silent Mode (which
   makes Watch alerts haptic-only), and Plan A's Emergency Bypass step
   was dropped because it too exists to force a sound.
7. Plan C gets built only after two weeks of Plan B use ending in one of
   the named failures: bursts of texts pile up separate alerts
   (deduplication is the main thing Plan C adds), acknowledging costs
   too many taps, or routing alert text through Pushover's servers is
   unwanted.

## What was checked, and how

- The watchOS background-haptic limits in decision 2 were read this
  session in Apple's "Using extended runtime sessions" page and in Apple
  developer forum threads 682429 and 760402 (links in
  docs/plan-c-watch-app.md): only smart-alarm sessions start from the
  background, they are scheduled with start(at:) up to 36 hours ahead,
  and they run at most 30 minutes.
- That Pushover holds Apple's Critical Alerts entitlement and that
  emergency priority repeats until acknowledged was read this session on
  Pushover's blog and support pages.
- The repository state: two commits on main, pushed; git ls-remote
  showed the remote head equal to the local head (1cfb78e) when this
  file was written.

## What we only said, never verified

Each item below was asserted in conversation from memory or from
secondary sources. Treat none of them as established.

- That a Pushover notification sent with sound=none still taps the
  Watch. The Watch's tap is governed by its own alert settings rather
  than the notification's sound, which is the argument for expecting a
  tap, but the pairing was never run. The first test in
  docs/plan-b-shortcuts-pushover.md exists to check exactly this.
- That an app on a Focus's allowed list delivers silently during that
  Focus on the Watch as well as the phone.
- The delivery condition itself: that the Watch receives alerts only
  when the iPhone is locked and the Watch is unlocked and on the wrist.
  This came from Apple community forum posts, several of them years old.
- That the Shortcuts Message trigger hands the message text to the
  automation as Shortcut Input, and that its Sender field accepts
  several contacts (the docs hedge: one automation per person if not).
- Pushover specifics from memory of its API documentation: retry minimum
  30 seconds, expire maximum 10800 seconds, license about five dollars
  one-time per platform after a 30-day trial.
- Apple developer logistics from training knowledge: free-account builds
  stop launching 7 days after install, the Developer Program costs 99
  USD/year and TestFlight builds last 90 days.
- Settings menu paths (Prominent Haptics, Wrist Detection under Watch >
  Settings > Passcode, Repeat Alerts, Auto-Lock): menu names move
  between OS versions.

## Open questions, each with what it blocks

1. RESOLVED 2026-10-26: yes. With the phone locked and the Watch on her
   wrist, the tap arrives and Cara feels it when she pays attention; she
   ignores it while working at her computer. The failure is attention,
   not delivery or strength. The original question read: Does a
   notification reach Cara's Watch at all when the phone is
   locked and the Watch is on her wrist? Plan A's failure ("felt
   nothing") is consistent with taps arriving too weak OR with no
   delivery, and the one-text test at the end of the README's Plan A
   section distinguishes them. This blocks everything: if delivery is
   broken, Plan B inherits the failure, and the fixes are Wrist
   Detection, the Watch's passcode lock, and whether the phone was
   actually locked.
2. Does sound=none still tap the Watch? Blocks Plan B as a silent
   design; if it does not, the fallback is choosing Pushover's quietest
   tone, which violates the no-sound requirement and needs Jake's call.
3. Does the allowed-app route deliver on the Watch during her Work
   Focus? Blocks whether Plan B works during exactly the hours the
   project exists for.
4. After two weeks of use, did a named Plan C failure occur? Blocks the
   decision to build the app.

5. Is a repeated tap enough when a single tap is ignored, or does the
   signal also need to appear on her computer screen? Blocks whether
   Plan B alone is the design or whether a computer-side alert is added.
   Needs from Jake: which computer Cara uses (Mac or Windows) and whether
   she accepts a silent on-screen alert there.

## What to do first

The delivery test is done (open question 1). Ask Jake the two things
open question 5 needs, then walk him through the Plan B setup in
docs/plan-b-shortcuts-pushover.md, and record each test outcome as a
dated line in the README's Status section.

## Terms this file uses

- Emergency priority: Pushover's priority=2, where the server re-sends
  the notification every "retry" seconds until the recipient taps
  Acknowledge or "expire" seconds pass.
- Critical Alert: Apple's name for a notification that plays a sound
  through silent mode and Focus; a per-app entitlement Apple grants on
  written application. Off in this project by decision 6.
- Focus: the iPhone's do-not-disturb mode (Cara's is a Work Focus);
  apps and people on its allowed lists still deliver during it.
- Smart-alarm session: the WKExtendedRuntimeSession type that can wake a
  Watch app in the background at a pre-scheduled clock time; the reason
  it cannot serve here is in decision 2.
- Plans A, B, C: the three tiers in README.md (settings; Shortcuts +
  Pushover; bespoke app).
