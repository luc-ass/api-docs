# Proposal: make `k-40-rf.yaml` machine-readable

**This branch is a discussion aid, not a merge candidate.** It sketches what
the K 40 RF description would look like if a client could be generated from it
instead of written against its examples. Every commit is one idea and can be
read, taken or dropped on its own.

It builds on
[`k-40-rf-examples-from-a-live-device`](https://github.com/bosch-home-comfort/api-docs/compare/main...luc-ass:api-docs:k-40-rf-examples-from-a-live-device),
which corrects the places where the current examples disagree with a real
gateway. Compare against that branch to see only the proposals; compare against
`main` to see both.

## Where this comes from

An independent, read-only Home Assistant integration for the K 40 RF, built
entirely from this repository plus what one gateway answers. Writing it
surfaced the points below. None of them is a complaint about the device — the
local API is a good one, and the fact that a third party could build on it at
all is the reason this exists.

## The five points

| # | Commit | Problem |
|---|---|---|
| 1 | Collection endpoints | A client cannot ask which circuits exist. It has to call every declared path and count the 404s — over 200 requests at every startup. `/signals` already solves this with a `refEnum`; the same shape for `/heatSources`, `/heatingCircuits` and the rest would replace all of it. |
| 2 | Enumerations in the schema | This file already does this on **17 paths** — `/system/type` declares its vocabulary as `value.enum` in the schema. **54 paths** carry the same information only as `allowedValues` inside the example, where nothing validates it, and every enum defect found against a live gateway sits in that second group. Demonstrated on two of the 54; the rest is mechanical. |
| 3 | `403` means "not in the spec" | The gateway answers `403` for paths it does not implement, not `404`. A client that reads `403` the natural way — "credentials rejected" — logs the user out over one unsupported resource. It cost us a broken installation. |
| 4 | `/signals` has a grammar | The branch is documented as "a list of signals". Its ids are structured, its `state` is an object and means an enumeration rather than the sentinel list `state` means everywhere else, and one appliance serves 87 of them. Any of that written down saves every client from guessing. |
| 5 | One unit per quantity | `1/min` and `rpm` for fan speed, `s` and `mins` for time, `db` for `dB`, `wh` for `Wh`, `C` for `°C`. This one is not a documentation defect — the firmware sends it that way, and the examples reproduce it faithfully. It belongs at the source. |

## What we are offering

The comparison behind all of this is a script that walks this file against a
live gateway's answers and prints every disagreement. It runs against any
installation. If it would help to have it pointed at other appliances — a
cascade, a boiler, a solar system — we are collecting those diagnostics anyway
and will gladly report what they say.
