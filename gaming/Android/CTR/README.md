#### Cut the Rope save migration guide for Android

##### 1. Scope and important notice

This guide sets out the approved procedure for migrating local save data for **Cut the Rope Free** and **Cut the Rope 2** on Android. It is intended solely for the recovery of a player's own save data.

Although both games provide cloud-related features and may integrate with Google Play Games, testing has shown that the relevant save data cannot be uploaded or restored successfully from either service. A complete local migration may therefore be required when replacing a device, changing an Android user profile, reinstalling the operating system, or moving between compatible game installations.

Before carrying out any operation, the operator shall observe the following requirements:

1. Force-stop the game and create a complete backup of its application data.
2. Preserve file contents, directory structure, ownership, permissions, and SELinux contexts.
3. Do not start the destination installation until the copied files and their metadata have been verified.
4. Use root privileges only on a device owned or administered by the operator.
5. Restore the original backup immediately if the game rewrites the save or produces an unexpected result.

##### 2. Why an ordinary file copy can fail

The main level-progress records are ordinary local data and can generally be copied successfully. Coins, hints, magnets, superpowers, and related consumable values are handled differently: each protected count is accompanied by an MD5 integrity value derived from the count and the application's SSAID.

The SSAID is the value returned to the application as `Settings.Secure.ANDROID_ID`. On Android 8.0 and later, it is scoped to the combination of device, Android user, and application signing key. It will therefore normally differ after migration to another device and may also change after a factory reset, a signing-key change, or certain custom-ROM operations.

When save data is copied from a source environment to a destination environment, the XML file still contains hashes derived from the source SSAID. The destination game recalculates the hashes with its own SSAID. If a stored value does not match the expected result, the game treats the protected count as invalid and may replace it with zero or another default value. This explains why level progress can remain intact while coins and power-ups disappear.

The mechanism described in this guide is an integrity check, not reversible encryption. An MD5 digest cannot be decrypted. Recovery requires either preserving the original identity environment or recalculating every affected hash with the destination SSAID and the intended count.

##### 3. Cut the Rope Free

##### 3.1 Version warning

At approximately September 2026, an update renamed **Cut the Rope Free** to **Cut the Rope**. Testing after the update identified material regressions in save continuity and level access. Existing progress may be lost; for example, players who had already completed the game may lose all progress associated with the final box. The updated release also removes conventional level selection and requires each group of three levels to be completed as one continuous sequence. If all three levels are not completed, the next attempt must begin again from the first level in that group.

Continued use of version **3.79.0** is therefore strongly recommended, and the affected update should not be installed. Automatic updates should also be disabled where appropriate. If the game has already been updated, force-stop it before further play and use a trusted backup-and-restore utility to restore save data created before the update. The pre-update backup shall be retained until the restored progress has been fully verified.

##### 3.2 Relevant save data

The Google free edition uses the package name `com.zeptolab.ctr.ads`. For Android user `0`, the relevant preferences file is normally located at:

```text
/data/user/0/com.zeptolab.ctr.ads/shared_prefs/CtrApp.xml
```

The protected values are stored in pairs:

| Item | Count field | Integrity field |
|---|---|---|
| Coins | `PREFS_COINS_COUNT` | `PREFS_COINS_COUNT_HASH` |
| Hints | `PREFS_HINTS_COUNT` | `PREFS_HINTS_COUNT_HASH` |
| Magnets | `PREFS_MAGNETS_COUNT` | `PREFS_MAGNETS_COUNT_HASH` |
| Superpowers | `PREFS_SUPERPOWERS_COUNT` | `PREFS_SUPERPOWERS_COUNT_HASH` |

##### 3.3 Integrity calculation

For every protected count, the game uses the following construction:

```text
MD5(SSAID + "AngryCats" + decimal_count)
```

The concatenation contains no separator. `decimal_count` is the ordinary base-10 representation of the integer, without padding. The resulting digest is written as 32 lowercase hexadecimal characters.

For example, if the destination SSAID is `0123456789abcdef` and the intended count is `50`, the input to MD5 is:

```text
0123456789abcdefAngryCats50
```

Changing only `PREFS_COINS_COUNT` is insufficient. `PREFS_COINS_COUNT_HASH` must be recalculated from the same count and the SSAID visible to the destination installation. The same rule applies independently to hints, magnets, and superpowers.

##### 3.4 Recovery utility

The accompanying `ctr.cpp` utility implements the calculation above. After compilation with Android NDK `clang++`, the binary may be pushed to the device and executed with root privileges. It can locate `CtrApp.xml`, obtain the current SSAID, inspect the four protected count/hash pairs, change selected counts, and regenerate either selected hashes or all hashes.

The preferred recovery procedure is to retain the destination SSAID and regenerate the hashes. Changing the system SSAID is normally unnecessary and has a wider effect on applications. Before using any write option, run the utility's help command and retain an untouched copy of `CtrApp.xml`.

##### 4. Cut the Rope 2

##### 4.1 Relevant save data

The free Google edition uses the package name `com.zeptolab.ctr2.f2p.google`. For Android user `0`, the relevant preferences file is normally located at:

```text
/data/user/0/com.zeptolab.ctr2.f2p.google/shared_prefs/CTR2.xml
```

The protected values are stored as follows:

| Item | Count field | Integrity field | Logical key used by the hash |
|---|---|---|---|
| Coins | `com.zeptolab.ctr2.f2p.coins` | `com.zeptolab.ctr2.f2p.coins_HASH` | `f2p.coins` |
| Free coins | `com.zeptolab.ctr2.f2p.coins_free` | `com.zeptolab.ctr2.f2p.coins_free_HASH` | `f2p.coins_free` |
| Unlimited coins | `com.zeptolab.ctr2.f2p.coins_unlim` | `com.zeptolab.ctr2.f2p.coins_unlim_HASH` | `f2p.coins_unlim` |

##### 4.2 Integrity calculation

Cut the Rope 2 uses the same general design as the first game but applies a different and field-specific construction. Let `N` be the base-10 representation of the count and let `K` be the logical key shown in the table above. The hash is:

```text
MD5(N + "!don'thackthis!" + K + "!" + N + "!" + SSAID + "!ctr2.")
```

For ordinary coins with a count of `50` and an SSAID of `0123456789abcdef`, the exact MD5 input is:

```text
50!don'thackthis!f2p.coins!50!0123456789abcdef!ctr2.
```

The count occurs twice, and the logical key forms part of the digest. Consequently, a valid hash for ordinary coins cannot be reused for free coins or unlimited coins, even when the numerical values are identical. As with the first game, copying `CTR2.xml` to a device with a different SSAID leaves the original hashes invalid and can cause the protected counts to be reset.

##### 4.3 Recovery utility

The accompanying `ctr2.cpp` utility implements the Cut the Rope 2 construction. After compilation with Android NDK `clang++`, the binary may be pushed to the device and executed with root privileges. It can locate `CTR2.xml`, obtain the current SSAID, inspect all three count/hash pairs, change selected counts, and regenerate selected or all integrity hashes.

The operator should normally keep the destination SSAID and update the hashes in `CTR2.xml`. The source and destination XML files shall be backed up separately, because starting the game with mismatched values may cause the original counts to be overwritten before recovery is attempted.

##### 5. Standard migration procedure

The following procedure applies to both games:

1. On the source device, force-stop the game and back up its complete package directory.
2. Record the source SSAID and retain the original preferences XML as evidence of the intended counts.
3. On the destination device, install the same compatible game edition but do not begin normal play.
4. Force-stop the destination game and copy the complete source package data into the destination package directory.
5. Set every copied file to the destination application's UID and GID. Apply suitable permissions—commonly `0600` or `0644` for files and `0700` or `0755` for directories, subject to the original layout—and restore SELinux contexts.
6. Use `ctr` for `CtrApp.xml` or `ctr2` for `CTR2.xml` to recalculate the protected hashes with the destination SSAID.
7. Reopen the XML and verify that every intended count has a corresponding newly calculated hash.
8. Start and enter the game to confirm the level progress and protected item counts.

If level progress is missing as well, the failure is not limited to the SSAID-bound integrity fields. In that case, the operator shall restore the backup and inspect the completeness of the copied application directory, file ownership, permissions, SELinux contexts, game version, package name, and signing compatibility before attempting another migration.

---

#### Android 版《割绳子》存档迁移指南

##### 1. 适用范围及重要提示

本指南规定了 Android 平台 **Cut the Rope Free（割绳子免费版）**与 **Cut the Rope 2（割绳子 2）**本地存档的迁移方法，仅适用于恢复玩家本人所有的游戏数据。

上述两款游戏虽提供云同步相关功能，并可与 Google Play 游戏服务联动，但经测试，相关存档并不能通过任一服务成功上传或恢复。因此，在更换设备、切换 Android 用户、重装操作系统或者在兼容的游戏安装之间迁移时，可能仍须执行完整的本地数据迁移。

执行任何操作前，操作人员应当遵守下列要求：

1. 强制停止游戏，并完整备份其应用数据。
2. 保留文件内容、目录结构、所有者、权限以及 SELinux 上下文。
3. 在确认所复制的文件及其元数据无误前，不得启动目标安装。
4. 仅可在操作人员本人所有或者负责管理的设备上使用 root 权限。
5. 如游戏改写存档或者出现异常结果，应当立即恢复原始备份。

##### 2. 普通文件复制可能失败的原因

主要关卡进度以普通本地数据形式保存，通常能够直接复制。金币、提示、磁铁、超级能力及其他相关消耗品则采用不同的处理方式：每项受保护数量均附有一个 MD5 完整性校验值，该值由相应数量与应用程序的 SSAID 共同派生。

SSAID 是应用程序通过 `Settings.Secure.ANDROID_ID` 获取的值。在 Android 8.0 及更高版本中，该值由设备、Android 用户及应用签名密钥的组合共同确定。因此，将数据迁移至另一台设备后，SSAID 通常会发生变化；恢复出厂设置、更换签名密钥或者执行某些自定义系统操作，也可能造成该值变化。

将存档从源环境复制到目标环境后，XML 文件中仍保存着使用源 SSAID 计算所得的散列值，而目标游戏会使用自身可见的 SSAID 重新计算散列值。如果存储值与预期结果不符，游戏将相应受保护数量判定为无效，并可能将其重置为零或者其他默认值。因此，迁移后可能出现关卡进度仍然存在，但金币及道具数量消失的情况。

本指南所述机制属于完整性校验，而非可逆加密。MD5 摘要无法解密。恢复数据时，应当保留原有身份环境，或者根据目标设备的 SSAID 及预期数量重新计算全部受影响的散列值。

##### 3. Cut the Rope Free（割绳子免费版）

##### 3.1 版本警告

约于 2026 年 9 月，**Cut the Rope Free（割绳子免费版）**通过一次更新更名为 **Cut the Rope（割绳子）**。经测试，该次更新在存档连续性和关卡访问方面存在较为严重的倒退：既有进度可能发生丢失；例如，已经打通全部关卡的玩家可能丢失最后一个盒子的全部进度数据。更新后的版本还取消了常规选关功能，要求玩家将每组三关作为一个连续流程完成；如未能完成全部三关，下次进入时只能从该组三关的第一关重新开始。

据此，强烈建议继续使用 **3.79.0** 版本，不要安装上述更新，并酌情关闭应用商店的自动更新功能。如已完成更新，应当在继续游玩前强制停止游戏，并使用可信的备份还原软件恢复更新前创建的存档。在确认恢复后的游戏进度完整无误前，不得删除更新前的原始备份。

##### 3.2 相关存档数据

谷歌免费版的包名为 `com.zeptolab.ctr.ads`。对于 Android 用户 `0`，相关首选项文件通常位于：

```text
/data/user/0/com.zeptolab.ctr.ads/shared_prefs/CtrApp.xml
```

受保护数据按下列数量字段与完整性字段成对存储：

| 项目 | 数量字段 | 完整性字段 |
|---|---|---|
| 金币 | `PREFS_COINS_COUNT` | `PREFS_COINS_COUNT_HASH` |
| 提示 | `PREFS_HINTS_COUNT` | `PREFS_HINTS_COUNT_HASH` |
| 磁铁 | `PREFS_MAGNETS_COUNT` | `PREFS_MAGNETS_COUNT_HASH` |
| 超级能力 | `PREFS_SUPERPOWERS_COUNT` | `PREFS_SUPERPOWERS_COUNT_HASH` |

##### 3.3 完整性校验计算方法

对于每一项受保护数量，游戏均采用下列结构计算散列值：

```text
MD5(SSAID + "AngryCats" + decimal_count)
```

各部分直接连接，不使用分隔符。`decimal_count` 为相应整数的普通十进制表示，不添加前导零。最终摘要以 32 个小写十六进制字符写入文件。

例如，目标 SSAID 为 `0123456789abcdef`，预期数量为 `50` 时，MD5 的输入内容为：

```text
0123456789abcdefAngryCats50
```

仅修改 `PREFS_COINS_COUNT` 并不能使数据生效。必须使用相同的数量和目标安装实际可见的 SSAID，重新计算 `PREFS_COINS_COUNT_HASH`。提示、磁铁和超级能力亦须分别按照同一规则处理。

##### 3.4 恢复工具

配套的 `ctr.cpp` 工具已经实现上述计算方法。使用 Android NDK 的 `clang++` 编译后，可将所得二进制文件推送至设备并以 root 权限运行。该工具能够定位 `CtrApp.xml`、获取当前 SSAID、检查四组受保护的数量与散列值、修改指定数量，并重新生成指定项目或者全部项目的散列值。

恢复时，原则上应当保留目标设备的 SSAID，并重新生成相应散列值。修改系统 SSAID 通常没有必要，且会对其他应用产生更广泛的影响。使用任何写入选项前，应当先查看该工具的帮助信息，并保留一份未经修改的 `CtrApp.xml`。

##### 4. Cut the Rope 2（割绳子 2）

##### 4.1 相关存档数据

谷歌免费版的包名为 `com.zeptolab.ctr2.f2p.google`。对于 Android 用户 `0`，相关首选项文件通常位于：

```text
/data/user/0/com.zeptolab.ctr2.f2p.google/shared_prefs/CTR2.xml
```

受保护数据按照下表存储：

| 项目 | 数量字段 | 完整性字段 | 参与散列计算的逻辑键名 |
|---|---|---|---|
| 金币 | `com.zeptolab.ctr2.f2p.coins` | `com.zeptolab.ctr2.f2p.coins_HASH` | `f2p.coins` |
| 免费金币 | `com.zeptolab.ctr2.f2p.coins_free` | `com.zeptolab.ctr2.f2p.coins_free_HASH` | `f2p.coins_free` |
| 无限金币 | `com.zeptolab.ctr2.f2p.coins_unlim` | `com.zeptolab.ctr2.f2p.coins_unlim_HASH` | `f2p.coins_unlim` |

##### 4.2 完整性校验计算方法

《割绳子 2》与第一代采用相同的总体设计，但使用了另一种针对具体字段的计算结构。设 `N` 为数量的十进制表示，`K` 为上表所列逻辑键名，则散列值为：

```text
MD5(N + "!don'thackthis!" + K + "!" + N + "!" + SSAID + "!ctr2.")
```

例如，普通金币数量为 `50`，SSAID 为 `0123456789abcdef` 时，MD5 的准确输入内容为：

```text
50!don'thackthis!f2p.coins!50!0123456789abcdef!ctr2.
```

数量在输入内容中出现两次，逻辑键名也参与摘要计算。因此，即使数值相同，普通金币的有效散列值也不能用于免费金币或者无限金币。与第一代相同，将 `CTR2.xml` 复制到 SSAID 不同的设备后，原散列值将不再有效，并可能导致受保护数量被重置。

##### 4.3 恢复工具

配套的 `ctr2.cpp` 工具已经实现《割绳子 2》的计算方法。使用 Android NDK 的 `clang++` 编译后，可将所得二进制文件推送至设备并以 root 权限运行。该工具能够定位 `CTR2.xml`、获取当前 SSAID、检查三组数量与散列值、修改指定数量，并重新生成指定项目或者全部项目的完整性散列值。

操作人员原则上应当保留目标设备的 SSAID，并更新 `CTR2.xml` 中的散列值。源 XML 与目标 XML 应当分别备份，因为使用不匹配的数值启动游戏后，原有数量可能在恢复操作开始前即被覆盖。

##### 5. 标准迁移程序

下列程序同时适用于两款游戏：

1. 在源设备上强制停止游戏，并完整备份其软件包目录。
2. 记录源 SSAID，并保留原始首选项 XML，作为预期数量的依据。
3. 在目标设备上安装同一兼容版本的游戏，但不得开始正常游玩。
4. 强制停止目标游戏，将源软件包的完整数据复制到目标软件包目录。
5. 将全部复制文件的 UID 与 GID 设置为目标应用的 UID 与 GID。按照原有目录结构设置适当权限——文件通常为 `0600` 或 `0644`，目录通常为 `0700` 或 `0755`——并恢复 SELinux 上下文。
6. 对于 `CtrApp.xml` 使用 `ctr`，对于 `CTR2.xml` 使用 `ctr2`，根据目标 SSAID 重新计算受保护数据的散列值。
7. 重新打开 XML，确认每一项预期数量均有相应的新散列值。
8. 启动并进入游戏，核验关卡进度和受保护道具数量。

如果关卡进度也同时丢失，则故障并非仅限于与 SSAID 绑定的完整性字段。此时，操作人员应当恢复备份，并检查所复制应用目录的完整性、文件所有者、权限、SELinux 上下文、游戏版本、包名及签名兼容性，确认无误后方可再次执行迁移。
