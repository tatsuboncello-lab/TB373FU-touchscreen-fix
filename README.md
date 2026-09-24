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

---

---

# 日本語 / Japanese

## 概要

Lenovo TB373FU（Idea Tab Pro / Xiaoxin Pad Pro 2025）で発生していた、断続的なタッチスクリーン不良について調査した記録です。

私の端末では、特にスリープからの復帰後などに、タッチスクリーンの感度が極端に悪化することがありました。症状発生中は通常のタッチが無視されたり、かなり強く押さないと反応しない状態になることがありました。

画面をOFF→ONすると一時的に正常へ戻ることがあったため、単純なタッチパネルのハードウェア故障ではなく、タッチコントローラの状態やキャリブレーションに関係する可能性を疑いました。

Novatekのタッチスクリーンドライバ `nt36532` を調査し、ReK（再キャリブレーション）を強制的に実行したところ、**不具合が実際に発生している状態から、その場で正常なタッチ操作へ復帰することを2回再現できました。**

さらに、その後の調査で診断モードの切り替えを行って以降、執筆時点で**約2週間、一度も症状が再発していません。**

ただし、長期的に再発しなくなった直接の原因については、まだ特定できていません。

---

## 検証環境

- 端末：Lenovo TB373FU
- 製品：Lenovo Idea Tab Pro / Xiaoxin Pad Pro 2025 系
- Android：16
- ZUI：17.5.10.057
- Root：Magisk
- 調査したタッチスクリーンドライバ：Novatek `nt36532`

---

## 症状

確認していた主な症状は以下です。

- 突然タッチ感度が極端に悪くなる
- 軽いタッチをほとんど認識しなくなる
- 強く押すと反応する場合がある
- スリープからの復帰後に発生することが多い
- 画面OFF→ONで一時的に正常へ戻る場合がある
- 発生頻度にはかなりばらつきがある

画面OFF→ONだけで復旧することがあったため、デジタイザ自体の恒久的なハードウェア故障とは異なる可能性があると考えました。

---

## ReK（再キャリブレーション）の検証

`nt36532` タッチスクリーンドライバを調査し、実験用にドライバを変更して、

```sh
echo 2 > /proc/game_mode
```

を実行すると、タッチコントローラにReK（再キャリブレーション）の

```text
0x23 0x00
```

を送るようにしました。

重要なのは、**正常な状態で試したのではなく、実際にタッチ不良が発生している最中に実行した**ことです。

結果は、

```text
タッチ不良（BAD）
       ↓
ReKを実行
       ↓
即座に正常復帰
```

となりました。

途中で画面OFF→ONは行っていません。

この復旧を**2回再現**できました。

そのため、少なくとも私の端末では、ReKによって異常なタッチ状態から復旧できることを確認しています。

---

## `/proc/game_mode` についての重要な注意

**通常のTB373FUで、単純に次のコマンドを実行すれば直るという意味ではありません。**

```sh
echo 2 > /proc/game_mode
```

私の検証環境では、**この操作によってReKが実行されるよう、タッチスクリーンドライバを実験用に変更していました。**

`/proc/game_mode` の動作はカーネルやドライバの実装によって異なります。

他の端末でこのコマンドをそのまま実行しないでください。

---

## 診断モード切り替え後の経過

その後、baseline / raw / diff などのタッチスクリーン診断情報を調査する過程で、コントローラについて

```text
TEST_MODE_2
    ↓
NORMAL
    ↓
HOST_READY
```

などのモード遷移を行いました。

すると、その後、それまで繰り返し発生していたタッチ不良が発生しなくなりました。

このREADMEを書いている時点で、**約2週間、一度も再発していません。**

---

## 確認できたこと

以下については実機で確認しています。

1. タッチスクリーンが異常な低感度状態になった
2. 画面OFF→ONを行わずReKを実行した
3. 直後に正常なタッチ感度へ復帰した
4. この復旧を2回再現した
5. その後の診断モード検証以降、約2週間再発していない

したがって、**発症中のReKによる即時復旧については再現性を確認できています。**

---

## まだ分かっていないこと

現時点では、**恒久的に修正できたとは断定していません。**

長期間再発しなくなった理由としては、

- ReKによって異常なキャリブレーション状態が修正された
- `TEST_MODE_2 → NORMAL → HOST_READY` の遷移によってコントローラ内部の異常状態がリセットされた
- ReKと診断操作の組み合わせが影響した
- その他の検証操作による副作用
- 偶然、再発していないだけ

などが考えられます。

他のTB373FUで同じ結果が再現できるか確認できれば、非常に有用だと考えています。

---

## この情報を公開する理由

TB373FU / Idea Tab Pro / Xiaoxin Pad Pro 2025で、似たようなタッチスクリーン不良に困っているユーザーがいるため、調査結果を公開することにしました。

特に、

- スリープ復帰後にタッチ感度が極端に悪化する
- 画面OFF→ONで一時的に復旧する
- タッチパネルに物理的な破損がないのに症状が断続的に発生する

といった症状がある場合、タッチパネルそのものの故障だけでなく、**タッチコントローラの内部状態やキャリブレーション**が関係している可能性があります。

同様の症状がある方や、別のTB373FUで再現できた方がいれば、GitHubのIssueで情報を共有してもらえると助かります。

---

## 注意事項

この調査ではroot権限、タッチスクリーンドライバの解析・変更、カーネルインターフェースへの直接アクセスを行っています。

ここに記載したコマンドは、**他の端末でそのまま実行するための手順ではなく、技術的な調査記録として掲載しています。**

誤ったカーネルインターフェース操作やドライバ変更は、端末の不安定化などを引き起こす可能性があります。

---

## 現在の状況

- **ReKによる即時復旧：2回再現**
- **診断モード検証後：約2週間再発なし**
- **恒久的な修正かどうか：現時点では未確定**

症状が再発した場合や、新しい情報が得られた場合は、このREADMEを更新します。
