# Lenovo TB373FU / Xiaoxin Pad Pro 2025 Touchscreen Issue – ReK Recovery and Possible Long-Term Fix

## Summary

I experienced a serious intermittent touchscreen issue on the Lenovo TB373FU
(Xiaoxin Pad Pro 2025 / related Lenovo tablet platform).

The touchscreen would sometimes become extremely insensitive, especially after
sleep/wake. In the bad state, normal touches were often ignored or required
unusually strong pressure.

Turning the screen off and on could sometimes temporarily recover the
touchscreen, which suggested that this might not be a simple hardware failure.

After investigating the Novatek touchscreen driver (`nt36532`), I found that
forcing a ReK (re-calibration) operation immediately restored normal touchscreen
behavior.

This recovery was reproduced twice while the touchscreen was actually in the
bad state.

More interestingly, after additional touchscreen diagnostic mode transitions,
the issue has not returned for approximately two weeks.

The exact reason for the long-term improvement is not yet proven, but the
results strongly suggest that calibration / controller state may be involved.

---

## Device

- Device: Lenovo TB373FU
- Product family: Xiaoxin Pad Pro 2025 / Idea Tab Pro platform
- Android: 16
- ZUI: 17.5.10.057
- Root: Magisk
- Touchscreen driver: Novatek `nt36532`

---

## Symptoms

The touchscreen issue was intermittent.

Typical behavior:

- Touch input suddenly became very insensitive
- Light touches were frequently ignored
- Stronger pressure sometimes worked
- The problem often appeared after sleep/wake
- Turning the display off and on could temporarily restore normal operation
- Cleaning/wiping the screen sometimes appeared to improve sensitivity
- The frequency varied significantly

Because display power cycling could recover the touchscreen, I suspected that
the problem might involve touchscreen controller state or calibration rather
than permanent digitizer hardware damage.

---

## Investigation

The kernel exposed the following interface:

    /proc/game_mode

During testing, I modified the `nt36532` touchscreen driver so that writing:

    echo 2 > /proc/game_mode

triggered the controller's ReK / recalibration command:

    0x23 0x00

The important test was performed while the touchscreen was already in the bad
state.

Result:

    Touch BAD
        ↓
    Trigger ReK
        ↓
    Touch immediately returns to normal

This was reproduced twice.

No display power cycle was required between the bad state and recovery.

That makes ReK a strong candidate for explaining the immediate recovery.

---

## Unexpected Long-Term Result

After the ReK experiments, I also used the touchscreen diagnostic functions,
including transitions involving:

    TEST_MODE_2
        ↓
    NORMAL
        ↓
    HOST_READY

These operations were used while reading/testing touchscreen diagnostic data
such as baseline/raw/diff values.

After these diagnostic mode transitions, the intermittent touchscreen problem
stopped occurring.

At the time of writing, the device has operated for approximately two weeks
without another occurrence.

Before this experiment, the problem had occurred repeatedly and unpredictably.

---

## What Is Confirmed

The following behavior was directly reproduced:

1. The touchscreen entered the bad / insensitive state.
2. ReK was triggered without power-cycling the display.
3. Touch sensitivity immediately returned to normal.
4. This recovery was reproduced twice.

Therefore, ReK can recover the touchscreen from the problematic state on my
device.

---

## What Is NOT Yet Confirmed

I cannot yet claim that ReK permanently fixes the problem.

The long-term disappearance of the issue happened after several related
operations, including ReK and diagnostic mode transitions.

Possible explanations include:

- ReK itself corrected persistent calibration data
- TEST_MODE_2 → NORMAL → HOST_READY reset an abnormal controller state
- The combination of ReK and diagnostic operations caused the improvement
- Another side effect of the testing reset the controller state
- The lack of recurrence is coincidental

More testing on other affected TB373FU devices is needed.

---

## Why I Am Publishing This

I am publishing these findings because other TB373FU / Xiaoxin Pad Pro users
may be experiencing similar intermittent touchscreen problems.

If your device shows symptoms such as:

- touchscreen sensitivity becoming extremely poor after sleep
- temporary recovery after screen off/on
- inconsistent sensitivity without obvious physical damage

then the problem may involve touchscreen controller calibration or internal
state rather than necessarily being a defective touch panel.

I would especially like to know whether ReK or touchscreen diagnostic mode
transitions produce the same recovery on other affected devices.

---

## Warning

The tests described here involved:

- root access
- kernel / touchscreen driver investigation
- modified driver behavior
- direct interaction with touchscreen controller functions

Do NOT blindly run commands from this document on another device.

The meaning of `/proc/game_mode` depends on the kernel/driver implementation.
Writing arbitrary values to kernel interfaces can cause instability or other
problems.

The `echo 2 > /proc/game_mode` behavior described here worked because the
driver used during my experiment had been specifically modified to trigger ReK.

This document is primarily intended as technical information for investigation
and reproduction.

---

## Current Status

**Immediate ReK recovery:** reproduced twice

**Long-term status:** no recurrence for approximately two weeks after the
diagnostic-mode experiments

**Permanent fix:** not yet proven

If the issue returns, I will update this document with additional logs and test
results.
