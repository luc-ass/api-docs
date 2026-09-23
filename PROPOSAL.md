# Proposal: make `k-40-rf.yaml` machine-readable

**This branch is a discussion aid, not a merge candidate.** It sketches what
the K 40 RF description would look like if a client could be generated from it
instead of written against its examples. Every commit is one idea and can be
read, taken or dropped on its own.

It builds on
[`k-40-rf-examples-from-a-live-device`](https://github.com/bosch-home-comfort/api-docs/compare/main...luc-ass:api-docs:k-40-rf-examples-from-a-live-device),
which corrects the places where the current examples disagree with a real
gateway. [Compare against that branch](https://github.com/luc-ass/api-docs/compare/k-40-rf-examples-from-a-live-device...k-40-rf-proposal-machine-readable)
to see only the proposals, or [against
`main`](https://github.com/bosch-home-comfort/api-docs/compare/main...luc-ass:api-docs:k-40-rf-proposal-machine-readable)
to see both.

## Where this comes from

An independent, read-only Home Assistant integration for the K 40 RF, built
entirely from this repository plus what one gateway answers. Writing it
surfaced the points below. None of them is a complaint about the device — the
local API is a good one, and the fact that a third party could build on it at
all is the reason this exists.

## The six points

| # | Commit | Problem |
|---|---|---|
| 1 | Collection endpoints | A client cannot ask which circuits exist. It has to call every declared path and count the 404s — over 200 requests at every startup. `/signals` already solves this with a `refEnum`; the same shape for `/heatSources`, `/heatingCircuits` and the rest would replace all of it. |
| 2 | Enumerations in the schema | This file already does this on **17 paths** — `/system/type` declares its vocabulary as `value.enum` in the schema. **54 paths** carry the same information only as `allowedValues` inside the example, where nothing validates it, and every enum defect found against a live gateway sits in that second group. Demonstrated on three of the 54; the rest is mechanical. One caution from three appliances: `allowedValues` is per installation (`/devices/uhc/assignedTo` offers `hc1` on two, not on the third), so the schema enum is the union and the live list wins. |
| 3 | `403` means "not in the spec" | The gateway answers `403` for paths it does not implement, not `404`. A client that reads `403` the natural way — "credentials rejected" — logs the user out over one unsupported resource. It cost us a broken installation. |
| 4 | `/signals` has a grammar | The branch is documented as "a list of signals". Its ids are structured; its `state` is an object where everywhere else it is a sentinel list, and that object means an enumeration on a bare signal but a list of codes on one that carries a unit; flags arrive as the words `"true"` and `"false"` in a `stringValue` — every string signal on three appliances but two identity fields, which `allowedValues: ["false", "true"]` could say without changing the wire format; a module can head its own ids (`HC2MOD.FlowTemp`); and the appliance decides how many signals exist — 87, 99 and 120 on three machines, 68 common to all. Any of that written down saves every client from guessing. We guessed the `state` distinction wrong ourselves and shipped it. |
| 5 | One unit per quantity | `1/min` and `rpm` for fan speed, `s` and `mins` for time, `db` for `dB`, `wh` for `Wh`, `C` for `°C` — and `day`, `month` and `year` on three signals that are not quantities but one date split in three. This one is not a documentation defect — the firmware sends it that way, and the examples reproduce it faithfully. It belongs at the source. |
| 6 | One signal is not JSON | With EEBUS commissioned, `/signals/GWEEBUS.CEM.SKI` puts the raw bytes of the key identifier inside a JSON string. No encoding makes that body valid. A client that reads `/signals` in one pass loses the whole branch over this one field — the installation that reported it saw 120 signal ids and not one value. Like point 5 this is the firmware, not the file: sent as hex, the fingerprint would be ordinary text. |

## What we are offering

The comparison behind all of this is a script that walks this file against a
live gateway's answers and prints every disagreement. It runs against any
installation. If it would help to have it pointed at other appliances — a
cascade, a boiler, a solar system — we are collecting those diagnostics anyway
and will gladly report what they say.

It has now been run against two more appliances, both owned by someone else:
a Buderus-branded Logatherm WLW186i-12 on newer appliance firmware, and a Bosch
Compress CS5800iAW with two heating circuits, the second mixed. Neither
produced a defect in the static endpoints the first device had not already
shown — every remaining disagreement is the file declaring hardware a machine
does not have — so the enum and unit corrections reproduce on three
independently owned appliances and are this file's rather than one machine's.
`/heatingCircuits/{id}/mixerPosition` is now confirmed as described.

What they changed is the `/signals` side. The Buderus turned up the second
meaning of a `state` map (point 4). The CS5800iAW turned up the flags written
as words, a module heading its own ids, the date split over three units
(point 5), the per-installation `allowedValues` (point 2) and the one signal
that is not JSON (point 6).
