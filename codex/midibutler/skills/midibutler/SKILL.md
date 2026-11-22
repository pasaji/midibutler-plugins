---
name: midibutler
description: Use when the user wants to create, adjust or send settings for a hardware MIDI instrument (synth, drum machine, sampler, effects unit) through the MIDI Butler tools. Covers researching the device manual, saving settings as versions, and asking the user's browser to send them (CC, Program Change, SysEx).
---

# MIDI Butler

MIDI Butler is a relay between you and the user's browser. You research the instrument, build the messages and save them as versions. The user's browser sends them to the instrument over Web MIDI. MIDI Butler has **no device knowledge of its own**: it does not know which controller number does what. That is your job, from the manufacturer's documentation.

What it can do: save settings (CC, Program Change, SysEx) and ask the browser to send them.
What it cannot do: listen to audio, read settings back from the device, handle NRPN/RPN, or run sequences. Do not offer these.

## 1. Check the connection

If the MIDI Butler tools are not available, tell the user to connect MIDI Butler in the app (Connect agent) and stop. Do not work around it. Call `list_sounds` first; it is a safe read and shows the library.

## 2. Understand the request

Ask only what you need:

- Which instrument (make, model, firmware or OS version if relevant)?
- Which MIDI channel does it listen on? Ask before saving; do not assume channel 1. For a multi-part device (several voices or parts, each with its own MIDI channel) ask which part the sound is for.
  A rough answer is fine. The user can change a version's channel later in the app, but that moves **every** message in the version to one channel. So keep one version to one channel or part, and save separate versions for other parts.
  If the user does not know, use the default channel the manual documents, say so clearly before saving, and note it in the version `description`.
- What do they want: a sound, a tweak to an existing one, a connection test?

## 3. Research the manual yourself

- Find the **manufacturer's official** MIDI implementation chart or manual. Prefer it over forum posts.
- Confirm every controller number, value range, Program Change/bank behavior and SysEx format in that document. Never guess or rely on memory for numbers.
- Watch value scaling: panels often show 1–128, 0–100 or percentages, while MIDI is 0–127. Program Change numbers in MIDI Butler are 0–127 even if the device shows 1–128.
- Say which document you used and any assumption you made (channel, mode, firmware). Put a short note in the version `description`.
- If you cannot verify something, say so and ask. Saving fewer, verified messages is better than a full guess.
- **Do not save a value whose meaning you could not verify in the documentation.** Examples: which value selects which option, how a 14-bit value scales, or what a displayed range maps to. Leave it out, or ask the user first. If you must include an approximation, mark it APPROXIMATE in the version `description` and tell the user before they send.

## 4. Pick the sound

- Use the existing sound when continuing work: `list_sounds`, then `list_versions` with `sound_id`, and save the next version with `sound_id` and `parent_id` (the version you build on).
- Create a new sound with `create_sound` only for a new idea or an explicit variation. Set `device_label` to the instrument name; it only organizes the library and is not a device profile.
- Names are not identities. Use ids.

## 5. Build and save a version

`save_version` stores a full set of settings and **never sends MIDI**. The payload is:

```json
{
  "schemaVersion": 1,
  "name": "Short descriptive name",
  "description": "What it is, which manual it comes from, assumptions.",
  "messages": [
    { "type": "cc", "channel": 1, "controller": 17, "value": 40, "delayMs": 0 },
    { "type": "program_change", "channel": 1, "program": 5, "delayMs": 0 }
  ],
  "controls": [
    { "id": "brightness", "label": "Brightness", "channel": 1, "controller": 17, "value": 40, "groupId": "tone" }
  ],
  "controlGroups": [{ "id": "tone", "label": "Tone", "color": "cyan" }]
}
```

The numbers above only show the format. They are not a recipe for any device.

- CC: channel 1–16, controller 0–127, value 0–127. Program Change: program 0–127. `delayMs` (0–2000) is a pause **before** that message.
- Order matters; messages are sent in the order you give. Put bank select before Program Change if the manual requires it.
- `controls` are knobs the user can turn afterwards. Add them for the parameters worth tweaking, with clear labels, and make each match a CC message in `messages`. Maximum 32 controls and 16 groups. Colors: neutral, cyan, mint, amber, rose. Groups and colors are presentation only.
- Limits: 256 messages, 64 KiB per SysEx message, 256 KiB of MIDI data in total.
- Save the settings you know are correct. They are not a guaranteed snapshot of the device.
- After saving, tell the user what you saved and that nothing has been sent yet.

## 6. Send only when asked

1. `list_sessions` shows the user's browser windows, newest first. If there is exactly one, use it. If there are several, use the newest and tell the user which one you used. Closed windows stay listed for up to 12 hours, so the newest is usually the open one. Do not make the user identify a window.
2. Before you request, remind the user to connect a MIDI output in the app, keep that window open and in the foreground, and turn **Live MIDI** on. Without Live MIDI the request is not delivered, and they can send the saved version from the app themselves instead.
3. `request_send` with `version_id`, `session_id` and a **new UUID** as `request_id`. Reuse the same `request_id` for retries so the request is not duplicated. A pending request expires after 60 seconds and is not replayed.
4. Right after `request_send`, call `get_delivery` with `wait_seconds` 10. The browser picks a request up within a few seconds, so "pending" a moment after queueing is normal: wait for it instead of reporting it. Only if it is still pending after the wait, tell the user and ask them to check the window.

Be exact about the result:

- `sent` means the browser wrote the messages to its MIDI output. It does **not** mean the device received, accepted or applied them. Never say the instrument "changed", "accepted" or "sounds right".
- If it stays pending or expires, ask the user to check that the app window is open, a MIDI output is selected and Live MIDI is on, then offer to retry with the same `request_id`. Do not keep queuing new requests on your own.

## 7. SysEx needs extra care

- Use SysEx only from the manufacturer's documentation. Never invent bytes or guess a checksum. Compute checksums exactly as the manual specifies.
- The data includes the `F0` start and `F7` end, and every byte in between is 7-bit (0–127).
- **Every** SysEx send needs the user's explicit approval in the browser each time. It is never sent automatically. Say so before you request it.
- Explain in plain words what the message does before saving it.
- Do not put destructive messages (factory reset, bulk memory or pattern writes, firmware-related data) into a version unless the user explicitly asked for that and understands what it overwrites.

## 8. Be honest and careful

- Never claim you heard or verified a sound. You cannot listen.
- Do not ask for or handle passwords or access keys. Sign-in is done by the user in the browser.
- Do not write MIDI Butler data into long-term memory or other files unless the user asks.
- If a tool returns an error, report it as it is and suggest the next step. Do not retry in a loop.
