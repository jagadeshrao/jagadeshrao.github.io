---
layout: default
title: "A Passing Checksum Isn't Proof Your Data Is Right"
---

A multi-lane high-speed link — SERDES, HiSPi, MIPI, anything with N parallel channels feeding a deserializer — has two very different failure modes that produce the same symptom: wrong pixels, wrong bytes, wrong *something* downstream. It's worth being precise about the difference, because chasing the wrong one wastes real time.

**Failure mode one: a bit is wrong.** Noise, marginal signal integrity, a bad termination. The payload itself is corrupted.

**Failure mode two: the bits are all correct, but they're in the wrong place.** A swapped pair of lanes, a column/pixel remapping bug, a connector with two pins crossed. Every bit that arrives is a bit that some valid part of the frame actually produced — it's just not the bit that belongs there.

A checksum or CRC is built to catch failure mode one. It is *not* built to catch failure mode two, and this is easy to forget under pressure. CRC is a function of the bit pattern in a frame; if a fault reorders which valid data lands in which slot, the frame can still contain exactly the set of bits the CRC expects — just permuted. The CRC passes. All the "is my link healthy" indicators say yes. And the image (or packet, or record) is still wrong.

The tell is usually structural: errors that repeat with a fixed spatial or positional period — every Nth column, every 2nd block, a pattern that tracks lane count or PHY grouping instead of looking like random noise. Random corruption doesn't respect your architecture's boundaries. A wiring or mapping fault does, exactly, because it *is* your architecture's boundary, just connected wrong.

Two practical consequences:

**1. Not every test pattern can reveal a positional fault.** A pattern that repeats the same value across all channels (all-ones, all-zeros, a fixed constant) looks identical whether or not two channels got swapped — wrong-but-plausible data is indistinguishable from right data. You need a pattern where every lane/position carries a *distinct* value, so a swap produces a value that's wrong for that position specifically. A clean result on a low-entropy pattern is not evidence of anything; it means the pattern couldn't have shown you the fault even if it exists.

**2. Don't reach for signal-integrity fixes first.** Tap sweeps, deskew, IDELAY tuning — all of that fixes failure mode one. If your evidence points at mode two (all channels report locked, all CRCs pass, errors are structurally periodic), sweeping taps will not fix a connector with two pins physically swapped, and time spent doing it is time not spent checking the thing that's actually wrong.

The general principle: a passing integrity check tells you the bits weren't corrupted in transit. It tells you nothing about whether they were correctly *addressed*. Those are different guarantees, and most link-layer checks only give you the first one.
