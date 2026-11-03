# Caregiver Call

A planned Home Assistant integration for caregiving at home: call buttons by the bed, loud alerts on caregivers'
phones until someone answers, escalation to the next caregiver, a log of response times, and a built-in dashboard.
**100% local**, with no cloud services.

> **Status: design stage. Nothing is built yet.** The design is in [DESIGN.md](DESIGN.md). Feedback is welcome
> through GitHub issues, especially on the open questions at the end of the design.

> **Not a medical device.** This will not be a certified nurse-call or medical alert system, and it comes with no
> guarantee that an alert is delivered. Keep other emergency options in place.

## What it will do

- Any button Home Assistant can see (Zigbee buttons and more) becomes a call button, with gestures chosen per button
  so an accidental press doesn't call for help.
- Caregivers' Android phones get alerts that break through silent mode, with **On my way** and **Resolved** buttons.
- Unanswered calls repeat and escalate to the next caregiver, then to everyone.
- Spoken announcements on your speakers using local text-to-speech, and optional light cues.
- Button health (battery, last seen) and phone connection status, so a dead link is visible before it matters.
- A sidebar dashboard suited to a wall tablet: live calls, one-tap responses, test buttons, call history.

Phones receive alerts over their own connection to Home Assistant (on the home network or your own VPN), never
through a cloud push service. iPhones are limited by Apple's rules for local delivery; see DESIGN.md.

## License

[Apache License 2.0](LICENSE).

Created by [@nathandlawrence](https://github.com/nathandlawrence).
