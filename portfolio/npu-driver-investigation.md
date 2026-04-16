---
layout: default
title: "NPU Driver Investigation — amd/xdna-driver #1257"
date: 2026-04-15
category: portfolio
tags: [amd, npu, linux-kernel, driver, hardware, bug-report, root-cause-analysis]
---

# NPU Driver Investigation — amd/xdna-driver #1257

**Project:** Bug report + root cause analysis for AMD XDNA NPU driver on Strix Point  
**Repository:** [amd/xdna-driver #1257](https://github.com/amd/xdna-driver/issues/1257)  
**Date:** April 15, 2026

---

## What I Did

My GPD Pocket 4 (AMD Ryzen AI 9 HX 370, Strix Point) has an NPU that should work with the mainline `amdxdna` kernel driver — but it didn't. Every cold boot, the driver loaded, attempted to probe the NPU PCI device, and failed with `aie2_smu_init: Access power failed, ret -22`. The SMU power handshake never completed, `/dev/accel/` stayed empty, and no NPU runtime could use the hardware.

I ran a **cold-boot A/B test** to isolate the variable:

- **Firmware 1.1.2.64** (the version shipped with the newer `linux-firmware-other` package): driver probe fails every time, `ret -22`.
- **Firmware 1.0.0.63** (the older version): driver probe succeeds, `xrt-smi examine` reports the NPU device, `/dev/accel/0` appears.

This was a binary search on one variable — firmware version — across cold boots (warm reboots didn't reproduce the failure). The methodology mattered because the failure was in the SMU init path, which only runs on a cold power cycle.

## What I Found

The root cause was in the driver's **POWER_OFF precheck** — a code path in the out-of-tree driver that checked firmware version before attempting SMU initialization. The precheck was wrong for Strix Point's PCI device ID (`17f0`), causing it to reject firmware 1.1.2.64 and abort the probe before the hardware ever got a chance to initialize.

A patched `.ko` built from the out-of-tree source (with the precheck disabled) loads clean on cold boot regardless of firmware version. The NPU is now visible to user-space runtimes.

I also filed a **secondary bug** with CachyOS (`CachyOS-PKGBUILDS #1311`) — the `linux-firmware-other` package had a symlink inversion that put the wrong firmware blob at the path the driver expected.

## What I Learned

**The biggest catch came after I filed the report.** In my initial draft reply to AMD engineer Max Zhen, I proposed backporting a `min_fw_version` check from the out-of-tree driver into the mainline tree. It sounded reasonable — the out-of-tree driver had this check, the mainline driver didn't, so backport it, right?

Wrong. A parallel review caught that **Lizhi Hou had already merged a fix** into mainline (commit `75c151ceaacf`, merged 2026-02-25) that used a completely different mechanism — a firmware-name fallback to `npu_7.sbin` — not a minimum version check. If I'd pushed the backport suggestion, it would have conflicted with an already-merged fix and made me look like I hadn't read the tree.

**The lesson:** Before proposing any backport, grep the upstream tree for already-merged fixes. This is now a saved rule: `feedback_check_merged_fixes.md`.

Max Zhen engaged within 4 hours of the original report, answered both follow-up questions cleanly, and the thread closed gracefully. AMD is paying attention to their open-source driver issues — that's worth knowing.

## Current State

- **Driver level:** Unblocked. Patched `.ko` loads on cold boot.
- **Inference level:** Still blocked — FastFlowLM can't handle protocol-7 opcodes needed by Qwen3/GGUF models.
- **Upstream:** Waiting for the POWER_OFF precheck fix to land in the mainline kernel.

---

*This investigation was a solo effort with parallel AI-assisted review. The methodology (cold-boot A/B on firmware version) and the mistake catch (proposed backport vs. already-merged fix) are both documented here because both matter.*