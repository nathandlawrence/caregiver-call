# Caregiver Call - design sketch

A Home Assistant integration for caregiving at home. The person being cared for presses a call button; the
caregivers' phones (and speakers, and optional lights) alert loudly until someone answers; if nobody answers, it
escalates to the next person; every call is logged with its response time; a built-in panel shows it all.

> **Not a medical device.** Not a certified nurse-call or medical alert system, with no guarantee of delivery. Keep
> other emergency options in place.

Status: design only. Nothing is built yet. Feedback on the design is welcome.

---

## 1. Design rules

1. **A missed call is the failure that matters.** Loud by default. Phone alerts break through silent mode and Do Not
   Disturb (Android alarm stream).
2. **Never drop a press.** Presses queue; a press during an open call is recorded, never swallowed.
3. **No step can block another.** Phones go first; every delivery step runs independently; a failed step is logged
   and moves the call to the next escalation step at once.
4. **Calls stack, never replace.** Each call is its own phone notification (no shared notification tag).
5. **100% local.** No cloud service anywhere: not for phone alerts, not for speech, not as a backup. The only way in
   from outside is your own VPN back into the house. Every call works with the internet down. (Section 5 explains how
   phones get alerts without Google's push service.)
6. **Accidental presses can't summon help by mistake.** Which gesture means "call" is set per button. For example:
   hold = call, single/double = bedside light, so mashing a button in distress gives light, not a false alarm.
7. **Fixed labels.** Each person has a permanent label (for example Bed 1, Bed 2) used identically in alerts, the
   panel and the log, so a caregiver moving fast at 3 AM never has to work out who is who.
8. **Lights only where chosen.** Light cues are opt-in per light; lights in a resident's room are never flashed
   unless set.

## 2. The pieces

| Piece | What it is |
|---|---|
| **Person** | Who is cared for: name, permanent label ("Bed 1"), room, the words used in announcements, their own lights. |
| **Call button** | Any HA trigger: a Zigbee (ZHA / Zigbee2MQTT) button action, an event or button entity, a binary sensor, or a manual/voice call. Each gesture maps to: Call, Lights bright, Lights restore, or nothing. |
| **Caregiver** | An HA person plus their phone (HA companion app), whether they are on duty, and their place in the order. |
| **Escalation plan** | Steps with timings. Default: step 1 on-duty caregiver (phone + speaker), repeating every 60 s; step 2 after 2 min unanswered adds the next caregiver; step 3 after 5 min alerts everyone and every speaker, plus an HA notification and an optional self-hosted backup channel. |

## 3. A call, start to finish

`Calling` -> `On the way` (someone tapped **On my way**: who and when are recorded; repeats stop) -> `Resolved`
(tapped **Resolved** on the phone or panel, or a bedside gesture).

- Pressed again while open: re-alerts as "still needs help".
- On the way but not resolved after N minutes: an optional reminder to whoever acknowledged.
- Acknowledging is not silencing: it stops the repeats because a person is coming, and it is recorded.

## 4. How it alerts

- **Phones** (HA companion app, local delivery only - see section 5): Android alarm stream, high priority, actions
  **On my way** / **Resolved**; optional spoken message at maximum alarm volume.
- **Speakers:** a spoken announcement through local text-to-speech (for example Piper) on chosen speakers ("Bed One
  needs assistance."). The volume is set again after the stream starts, because some players reset their volume on a
  source switch, and restored afterwards.
- **Lights (optional):** a cue in the caregiver's room or hallway, such as a color or slow pulse.
- **Fallbacks:** an HA notification always; optionally a self-hosted channel (for example an ntfy server on your own
  network), never a public service.

## 5. Phone alerts without the cloud

- The HA phone app normally receives notifications through Google's push service (Firebase), which is cloud. Instead,
  the app keeps its own **direct connection to Home Assistant** ("persistent connection" / local push) and receives
  alerts over it: on the home network at home, over your VPN when away.
- Each caregiver phone: HA app with **Persistent connection = Always**, battery optimization **off** for the app, and
  a VPN that comes on automatically away from home. The app build without Google services (F-Droid "minimal") cannot
  fall back to the cloud at all, so it is the strictest choice.
- Caregiver Call **only sends to phones that are connected locally right now.** A phone that is not connected (VPN down,
  out of range, app killed) counts as a failed delivery: the call escalates at once and the speakers carry it.
- The panel shows each caregiver's phone as **connected / not connected**, so a dead link is visible before it matters.
- iPhones: Apple only allows local delivery on chosen home Wi-Fi networks, not over a VPN on cellular, so an iPhone
  caregiver would get no alerts away from home. Documented as a limitation; Android is the supported phone.

## 6. What it adds to Home Assistant

- **Per person:** call open (on/off), call state (idle / calling / on the way), last call time, last response time.
- **Per button:** health (ok / low battery / not seen lately) and a **Test** button that sends a clearly marked test call.
- **Overall:** open calls (count + list), who is on duty first, today's calls with average and longest response, an
  on/off switch for escalation, and an event for each call so other automations can react.
- **Actions** (for voice assistants and automations): `caregiver_call.call`, `caregiver_call.acknowledge`,
  `caregiver_call.resolve`, `caregiver_call.test`.

## 7. Built-in dashboard (sidebar panel)

- **Live calls:** a large card per open call: who, how long they have waited, who is coming. Turns red when unanswered.
- **One tap** On my way / Resolved on the panel, not only on the phone.
- **Button health:** battery and last-seen for every call button, with a warning when one is low or silent.
- **Phones:** each caregiver's phone shown as connected / not connected.
- **Test** each button end to end without a real emergency.
- **On duty:** who gets alerts first and the escalation order, changeable on the spot.
- **Call log:** today and recent days with response times.
- **Wall-tablet mode:** big text, high contrast, optional chime on the tablet itself.

## 8. Setup (all in the HA UI)

1. Add people (name, label, room).
2. Add call buttons: pick the device, press each gesture once so it is learned, choose what each gesture does.
3. Add caregivers (person + phone) and their order.
4. Escalation timings (defaults above).
5. Speakers, TTS voice and announcement volume.
6. Lights (opt-in).

Everything is editable later from the integration's Configure screen.

**Moving over from existing automations:** a shadow mode logs calls without sending anything, so it can run beside
an existing setup until it has been tested.

## 9. Reliability

- Open calls survive a Home Assistant restart: they reload and alert again.
- Button watch: low battery and "not seen" warnings, from each device's own battery and last-seen data.
- A warning the moment a caregiver's phone loses its local connection (VPN down, app killed, reinstalled or logged out).
- Optional weekly "test your buttons" reminder.
- Call history stored locally, kept for a set number of days.
- Testing: the call and escalation logic is tested with a simulated clock; Home Assistant's test harness covers setup
  and entities; live tests only at agreed times, since a real press wakes a real caregiver.

## 10. Not in version 1

Fall detection, vital signs, medication reminders, staff scheduling, multi-home or cloud dashboards.

## 11. Open design questions (feedback welcome)

1. Default escalation timings: is 60 s between repeats and 2 minutes before the second caregiver right for most homes?
2. Should **On my way** close a call, or is a separate **Resolved** step worth the extra tap?
3. Should a repeat press during an open call re-alert everyone, or only the caregiver who is on the way?
4. Which call buttons do you use? Support for more button types is easier to add with real devices to test against.

---

Apache-2.0 (planned). Not affiliated with any nurse-call or medical alert vendor.
