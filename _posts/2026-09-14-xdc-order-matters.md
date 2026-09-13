---
layout: default
title: "XDC Files Are Order-Dependent, and Vivado Won't Tell You When That Bites"
---

Xilinx/AMD constraint files (XDC) read like a declarative spec — a list of facts about your design's clocks and timing exceptions. They aren't evaluated that way. Vivado processes an XDC file top to bottom, and a constraint that references something not yet defined doesn't wait for it or error out loudly. It just does the wrong thing quietly.

The specific trap: `set_clock_groups` referencing a clock that a later `create_clock` defines. If the group command runs first, it resolves against a clock that, as far as Vivado is concerned at that point, doesn't exist yet. You get an empty clock group — and Vivado doesn't stop the build. It emits two critical warnings (in recent Vivado versions, `12-4739` and `12-5201`) and then discards the *entire* constraint, not just the broken reference. Implementation proceeds.

Here's the part that makes this dangerous rather than just annoying: the clock-domain-crossing paths that constraint was supposed to declare as false paths, or asynchronous, or exempt from single-clock timing analysis — those paths don't disappear. They get analyzed as if no exception exists, and depending on the rest of your constraint set, that can mean they're timed at effectively 0 ns, or excluded from the report entirely, or worse, silently reported as passing when the actual crossing has no real timing relationship at all. Timing closure can look clean. It isn't measuring what you think it's measuring.

None of this shows up as a build failure. `write_bitstream` succeeds. The design might even work — right up until it doesn't, under a corner case in silicon that your static timing analysis never actually checked.

What actually catches it:

- **Order your XDC deliberately.** Define every clock with `create_clock` before anything downstream (`set_clock_groups`, `set_false_path`, `set_max_delay -datapath_only`) references it. Split files by concern if that helps, but be explicit about load order in your project settings.
- **`.xdc` accepts a restricted Tcl subset** — no `if`, no `concat`, no `lsort` (you'll hit `Designutils 20-1307` if you try). If you need conditional or generated constraints, generate the XDC text before it's loaded, or use `PROCESSING_ORDER LATE` on a constraint to push its evaluation to the end of the pass instead of trying to control it with control flow the parser won't accept.
- **Always run `report_clock_interaction` after touching XDC**, not just `report_timing_summary`. The timing summary can look green while clock-group relationships are silently wrong. `report_clock_interaction` shows you what Vivado actually thinks the relationship between each clock pair is — that's where a dropped constraint becomes visible before it becomes a field failure.

The general lesson extends past Vivado: any constraint or configuration language that looks declarative but is actually parsed sequentially will have this failure mode. If the tool tolerates forward references silently instead of erroring, assume it's discarding something rather than deferring it, and verify with whatever the tool's equivalent of `report_clock_interaction` is.
