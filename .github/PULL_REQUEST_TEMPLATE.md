## Controller

- Maker and model:
- How it shows up (MIDI port name, or the Device Inquiry reply):
- Firmware version, if known:

## What changed

- [ ] A new map (`version = 1`, `status = "community"`)
- [ ] A change to an existing map (its `version` is higher than before)
- [ ] An update of a map Sublayer ships (version newer than the built-in; same ports and inquiry)

## Tested

- [ ] On the hardware with Sublayer (build number: ): every mapped control does what the map says
- [ ] Not tested on hardware (the map stays `community`)

Sources for the MIDI messages (the maker's MIDI list, another app's mapping, a capture):

## Checklist

- [ ] No `output_approved` in the file
- [ ] Start-up (`init`) and shutdown messages are only what the controller needs, and come from
      a cited source
- [ ] `sublayer-map-check check --deny-warnings` passes locally (the check job runs it too)
