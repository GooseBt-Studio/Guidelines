#### Cut the Rope save migration guide for Android

This guide provides instructions on migrating local save data for **Cut the Rope Free**, **Cut the Rope 2**, and **Cut the Rope Time Travel** on the Android platform. 
It is only applicable to restoring game data owned by the player or that has been authorized for manipulation. 

##### 1. Purpose and preparation

Although **Cut the Rope Free**, **Cut the Rope 2**, and **Cut the Rope Time Travel** on Android provide cloud-sync-related features and can integrate with Google Play Games, 
testing has shown that the relevant save data cannot be uploaded to or restored from either service successfully. 
Users may therefore still need to perform a complete local data migration when replacing a device. 

After migration, level progress is usually retained, while coins, hints, and similar items—collectively referred to below as protected counts—are reset to zero. 
Further action is required to complete the migration correctly, which is the reason for this guide. 

The methods described in this guide require root access on the destination device; root access on the source device is also recommended. 
Swift Backup is recommended, with the Restore SSAID option enabled on its settings page. Tools such as MT Manager may also be used. 

##### 2. Save migration

##### 2.1 Migration on the same device

When the game is not in use, players can generally suspend it by using the archive feature available in newer Android versions, 
the disable feature provided by Swift Backup (or shell commands) executed as root, or the freeze feature offered by applications such as Tombstone. 
The application can then be restored, enabled, or unfrozen when it is needed again. 

In some circumstances, however, a player may uninstall and reinstall the application. In that case, use the following procedure. 

1. Before uninstalling, use Swift Backup (recommended), Titanium Backup (an option only on older devices and earlier Android versions), MT Manager (not recommended because file permission information may be lost), or another suitable tool to back up ``/data/user/${user}/${package}/``. Here, ``${user}`` is the Android user ID, normally ``0`` for the device owner, commonly ``999`` for a cloned-app space, or ``10`` for a work profile. External data under ``/sdcard/Android/`` does not need to be backed up because it consists mainly of advertising caches. ``${package}`` is the application package name. 
2. Reinstall the application when restoration of the game progress is required. 
3. In Swift Backup, enable Restore SSAID and perform the restoration; a device restart may be required. Alternatively, use another tool to write the backup back to ``/data/user/${user}/${package}/`` and correct the permissions, owner (UID), and group (GID). If this metadata has been lost, set files to ``0644``, directories to ``0755``, and the owner and group of every file and directory to the UID of ``${package}``. 

##### 2.2 Migration between devices

The following procedure applies to the migration of local save data for **Cut the Rope Free**, **Cut the Rope 2**, and **Cut the Rope Time Travel** between devices. 

1. Force-stop the game on the source device. Use Swift Backup or another backup-and-restore utility capable of cross-device restoration to back up the application and its data; external data does not need to be included. Alternatively, use MT Manager to extract the installation package and compress all directories and files under ``/data/user/${user}/${package}/``. 
2. Record the SSAID visible to ``${package}`` on the source device and retain the original preferences XML as evidence of the intended protected counts. 
3. Install the same compatible version of the game on the destination device by restoring it with the backup utility, installing it through MT Manager, or obtaining it from Google Play (not recommended). Do not begin normal play. 
4. Force-stop the destination installation, then restore or extract the complete source package data into the destination package directory. 
5. Correct the permissions of all copied files as described in section 2.1, and set their UID and GID to those assigned to ``${package}`` on the destination device. 
6. If the game is opened at this stage, level progress will normally remain intact while the protected counts are reset to zero. Recalculate the protected-count hashes in ``/data/user/${user}/com.zeptolab.ctr.ads/shared_prefs/CtrApp.xml`` for Cut the Rope Free, ``/data/user/${user}/com.zeptolab.ctr2.f2p.google/shared_prefs/CTR2.xml`` for Cut the Rope 2, or ``/data/user/${user}/com.zeptolab.timetravel.free.google/shared_prefs/TIMETRAVEL.xml`` for Cut the Rope Time Travel. 
7. After making the changes, use MT Manager or another text editor to inspect the relevant XML manually and verify that every intended count has a corresponding new hash. 
8. Start the game and verify both the level progress and the protected item counts. 

##### 2.3 Recalculating protected-count hashes

Research shows that the save data is concentrated mainly in the XML files identified above. Core level progress is stored as ordinary local data and can generally be copied directly. 
Protected counts—such as coins, hints, magnets, and superpowers in Cut the Rope Free—are handled differently: each protected count is accompanied by an integrity value. 

After clearing the game's data and observing the first two completed levels, identical counts on the same device were found to produce identical integrity values. 
The values also match the characteristics of an MD5 digest: a 128-bit value represented by 32 hexadecimal characters. 
This indicates that each integrity value is an MD5 digest derived jointly from the corresponding count and the application's SSAID. 

The SSAID is the value obtained by the application through ``Settings.Secure.ANDROID_ID``. 
On Android 8.0 and later, it is determined by the combination of the device, Android user, and application signing key. 
It will therefore normally change when data is migrated to another device. A factory reset, a change of signing key, or certain custom-ROM operations may also change it. 

After a save is copied from the source environment to the destination environment, the XML file still contains hashes calculated with the source SSAID, 
while the destination game recalculates them using the SSAID visible in its own environment. If a stored value differs from the expected result, 
the game treats the corresponding protected count as invalid and may reset it to zero. This is why migration generally preserves level progress while clearing protected counts. 

This mechanism is an integrity check rather than reversible encryption. 
Since MD5 is a one-way function, recovery should therefore recalculate each protected count's integrity value from the count, rather than attempt to derive the count from its integrity value. 

As the calculation depends on both the protected count and the SSAID, either of the following approaches may be used during restoration. 

1. Change the SSAID visible to the application on the destination device to the SSAID that was visible on the source device, thereby avoiding XML modification or hash recalculation. 
2. Retain the SSAID visible on the destination device and recalculate every protected count individually. 

The latter method is described in the following sections. 

##### 3. Cut the Rope Free

##### 3.1 Version warning

In approximately September 2026, an update renamed **Cut the Rope Free** to **Cut the Rope**. Testing indicates that this update introduced serious regressions in save continuity and level access. Progress may be lost after updating; for example, players who had completed all levels may lose all progress associated with the final box. The update also removes conventional level selection and requires each group of three levels to be completed as a continuous sequence. If all three levels are not completed, the next attempt must begin again from the first level in that group. 

Continued use of version **3.79.0** is therefore strongly recommended. Do not install the affected update, and consider disabling automatic updates in the application store or using HMA or one of its variants to hide the application from Google Play. If the update has already been installed, use Core Patch to permit application downgrades, downgrade the application, and, where necessary, restore the earlier data with a backup-and-restore utility. 

##### 3.2 Relevant save data

The Google free edition uses the package name ``com.zeptolab.ctr.ads``. For Android user ``0``, the relevant preferences file is normally located at ``/data/user/0/com.zeptolab.ctr.ads/shared_prefs/CtrApp.xml``. Protected counts are stored as count/integrity-value pairs as follows. 

| Protected count | Count field | Integrity-value field |
| - | - | - |
| Coins | ``PREFS_COINS_COUNT`` | ``PREFS_COINS_COUNT_HASH`` |
| Hints | ``PREFS_HINTS_COUNT`` | ``PREFS_HINTS_COUNT_HASH`` |
| Magnets | ``PREFS_MAGNETS_COUNT`` | ``PREFS_MAGNETS_COUNT_HASH`` |
| Superpowers | ``PREFS_SUPERPOWERS_COUNT`` | ``PREFS_SUPERPOWERS_COUNT_HASH`` |

##### 3.3 Integrity value calculation

For every protected count, the game calculates the integrity value using the following construction. 

```text
MD5(SSAID + "AngryCats" + decimal_count)
```

The components are concatenated directly without a separator. ``decimal_count`` is the ordinary base-10 representation of the corresponding integer and contains no leading zeroes. The resulting digest is written to the file as 32 lowercase hexadecimal characters. 

For example, if the destination SSAID is ``0123456789abcdef`` and the intended count is ``50``, the exact MD5 input is as follows. 

```text
0123456789abcdefAngryCats50
```

For coins, changing only ``PREFS_COINS_COUNT`` is insufficient. ``PREFS_COINS_COUNT_HASH`` must be recalculated from the same count and the SSAID actually visible to the destination installation. 
Otherwise, the protected count will be reset to 0. The same rule applies to the other protected counts. 

##### 3.4 Recovery utility

The accompanying [``ctr.cpp``](https://github.com/LRFP-Team/Bypasser/blob/main/cpp/ctr.cpp) implements the integrity value calculation above. 
After compiling it for the appropriate architecture with Android NDK ``clang++``, push the resulting binary to the device and run it with root privileges, preferably through MT Manager or Termux. 
The utility can locate ``CtrApp.xml`` automatically, obtain the current SSAID, correct protected-count integrity values, change the SSAID, and set protected counts to specified values. 

Changing a system-level SSAID shared by all applications would affect other applications and is therefore not recommended. 
Before using any write option, review the utility's help information with ``ctr --help`` and retain an untouched copy of ``CtrApp.xml``. 

##### 4. Cut the Rope 2

##### 4.1 Relevant save data

The Google free edition uses the package name ``com.zeptolab.ctr2.f2p.google``. For Android user ``0``, the relevant preferences file is normally located at ``/data/user/0/com.zeptolab.ctr2.f2p.google/shared_prefs/CTR2.xml``. Protected counts are stored as follows. 

| Protected count | Count field | Integrity-value field | Logical key used in the hash |
| - | - | - | - |
| Balloons | ``com.zeptolab.ctr2.f2p.balloons`` | ``com.zeptolab.ctr2.f2p.balloons_HASH`` | ``com.zeptolab.ctr2.f2p.balloons`` |
| Bombs | ``com.zeptolab.ctr2.f2p.bombs`` | ``com.zeptolab.ctr2.f2p.bombs_HASH`` | ``com.zeptolab.ctr2.f2p.bombs`` |
| Coins | ``com.zeptolab.ctr2.f2p.coins`` | ``com.zeptolab.ctr2.f2p.coins_HASH`` | ``com.zeptolab.ctr2.f2p.coins`` |
| Free coins | ``com.zeptolab.ctr2.f2p.coins_free`` | ``com.zeptolab.ctr2.f2p.coins_free_HASH`` | ``com.zeptolab.ctr2.f2p.coins_free`` |
| Hints | ``com.zeptolab.ctr2.f2p.hints`` | ``com.zeptolab.ctr2.f2p.hints_HASH`` | ``com.zeptolab.ctr2.f2p.hints`` |
| Teleports | ``com.zeptolab.ctr2.f2p.teleports`` | ``com.zeptolab.ctr2.f2p.teleports_HASH`` | ``com.zeptolab.ctr2.f2p.teleports`` |
| Unlimited coins | ``com.zeptolab.ctr2.f2p.coins_unlim`` | ``com.zeptolab.ctr2.f2p.coins_unlim_HASH`` | ``f2p.coins_unlim`` |

Except for unlimited coins, each protected count is also represented by ``_BOUGHT``, ``_EARNED_FOR_FREE``, and ``_USED`` fields. The base count must agree with ``current = bought + earned_for_free - used``. When changing a protected count, reconcile these fields as well as the integrity value. 

##### 4.2 Integrity value calculation

**Cut the Rope 2** uses the same general design as the first game, but with a different field-specific construction. 
Let ``N`` be the base-10 representation of the count, and let ``K`` be the logical key shown in the table above. 
For ordinary protected counts, ``K`` is the complete preferences key, including the ``com.zeptolab.ctr2.`` prefix, that is, ``com.zeptolab.ctr2.f2p.coins_unlim``. 
The count of the unlimited coins is the sole exception, which uses ``f2p.coins_unlim``. The integrity value is calculated as follows. 

```text
MD5(N + "!don'thackthis!" + K + "!" + N + "!" + SSAID + "!ctr2.")
```

For ordinary coins with a count of ``50`` and an SSAID of ``0123456789abcdef``, the exact MD5 input is as follows. 

```text
50!don'thackthis!com.zeptolab.ctr2.f2p.coins!50!0123456789abcdef!ctr2.
```

The count occurs twice in the input, and the logical key also participates in the digest. 
Consequently, even when the numerical values are identical, a valid integrity value for ordinary coins cannot be used for free coins or unlimited coins. 
As with the first game, copying ``CTR2.xml`` to a device with a different SSAID invalidates the original integrity values and may cause the protected counts to be reset. 

Static comparison of versions **1.46.0** and **1.47.2**, followed by an offline device test, confirms that both versions use the same format template and source ``K`` from the protected field object. The previously documented shortened keys for ordinary fields were therefore incorrect; they were not introduced by a later game update. 

##### 4.3 Recovery utility

The accompanying [``ctr2.cpp``](https://github.com/LRFP-Team/Bypasser/blob/main/cpp/ctr2.cpp) implements the integrity value calculation above. 
After compiling it for the appropriate architecture with Android NDK ``clang++``, push the resulting binary to the device and run it with root privileges, preferably through MT Manager or Termux. 
The utility can locate ``CTR2.xml`` automatically, obtain the current SSAID, correct protected-count integrity values, change the SSAID, and set protected counts to specified values. 

For example, the following command sets coins to ``100`` and hints to ``50``, reconciles every supported ``bought + earned - used`` equation, and updates the two affected integrity values. 

```bash
ctr2 -s coin 100 hint 50 -e all -u coin hint
```

Changing a system-level SSAID shared by all applications would affect other applications and is therefore not recommended. 
Before using any write option, review the utility's help information with ``ctr2 --help`` and retain an untouched copy of ``CTR2.xml``. 

##### 5. Cut the Rope Time Travel

##### 5.1 Relevant save data

The Google free edition uses the package name ``com.zeptolab.timetravel.free.google``. For Android user ``0``, the relevant preferences file is normally located at ``/data/user/0/com.zeptolab.timetravel.free.google/shared_prefs/TIMETRAVEL.xml``. In version **1.20.0**, the protected item counts identified in this file are stored as follows. 

| Protected count | Count field | Integrity-value field |
| - | - | - |
| Hints | ``PREFS_HINTS_COUNT`` | ``PREFS_HINTS_COUNT_HASH`` |
| Superpowers | ``PREFS_SUPERPOWERS_COUNT`` | ``PREFS_SUPERPOWERS_COUNT_HASH`` |

##### 5.2 Integrity value calculation

**Cut the Rope Time Travel** follows the same general design as the other two games, but uses its own fixed strings. For both protected counts, let ``decimal_count`` be the ordinary base-10 representation of the count. The integrity value is calculated as follows. 

```text
MD5(SSAID + "ctr_tt_hd" + "_Shame" + " 0n You " + decimal_count)
```

The components are concatenated directly. The spaces before and after ``0n You`` are part of the input, and the first character is the digit zero rather than the letter O. The count contains no leading zeroes. The resulting digest is written as 32 lowercase hexadecimal characters. 

For a count of ``50`` and an SSAID of ``0123456789abcdef``, the exact MD5 input is as follows. 

```text
0123456789abcdefctr_tt_hd_Shame 0n You 50
```

Static analysis of version **1.20.0** shows that the native code obtains the value through ``Device::getUDID()``, which ultimately calls ``Settings.Secure.getString(..., "android_id")``. 
It then formats the SSAID, fixed strings, and count, converts the result to UTF-8, calculates MD5, and compares the lowercase hexadecimal digest with the value stored in the XML file. 
Despite the native method name, the device-dependent component is therefore the application's SSAID. 

The same construction is used for hints and superpowers. The field name does not participate in the digest. 
Consequently, equal hint and superpower counts under the same SSAID produce equal integrity values, as also observed in the supplied save where both counts are zero. 
Copying ``TIMETRAVEL.xml`` to an installation with a different SSAID invalidates both stored values and may cause both item counts to be reset. 

##### 5.3 Recovery utility

The accompanying [``ctrtt.cpp``](https://github.com/LRFP-Team/Bypasser/blob/main/cpp/ctrtt.cpp) implements the integrity value calculation above. 
After compiling it for the appropriate architecture with Android NDK ``clang++``, push the resulting ``ctrtt`` binary to the device and run it with root privileges, preferably through MT Manager or Termux. 
The utility can locate ``TIMETRAVEL.xml`` automatically, obtain the current SSAID, correct the integrity values for hints and superpowers, change the SSAID, and set either protected count to a specified value. 

Changing a system-level SSAID shared by all applications would affect other applications and is therefore not recommended. 
Before using any write option, review the utility's help information with ``ctrtt --help`` and retain an untouched copy of ``TIMETRAVEL.xml``. 

##### 6. Appeal

The game publisher should provide an appropriate and accessible method for ordinary users to migrate their save data. 

---

#### Android 版《割绳子》存档迁移指南

本指南提供了 Android 平台 **Cut the Rope Free（割绳子免费版）**、**Cut the Rope 2（割绳子 2）**与 **Cut the Rope Time Travel（割绳子：时光旅行）**本地存档的迁移方法，仅适用于恢复玩家本人所有的或已被授权操作的游戏数据。

##### 1. 动机及预备

Android 平台 **Cut the Rope Free（割绳子免费版）**、**Cut the Rope 2（割绳子 2）**与 **Cut the Rope Time Travel（割绳子：时光旅行）**虽在游戏内提供了云同步相关功能，并可与 Google Play 游戏服务联动，
但经测试，相关存档并不能通过任一服务成功上传至云端或从云端恢复。因此，用户在更换设备时，仍可能须手动执行完整的本地数据迁移。
然而，迁移后通常会出现关卡进度正常而金币及提示等道具数量（以下统称受保护数量）被清 0 的现象，需要进一步操作才能完成正确的迁移，因而写下本教程。

本教程所使用的方法要求目标设备具有 root 权限（建议源设备也具有 root 权限），推荐使用 Swift Backup 并在其设置页中开启还原 SSAID 功能，也可以使用使用 MT 管理器等工具。

##### 2. 存档迁移

##### 2.1 在同一设备上

一般地，玩家在不使用该游戏时可以通过较高版本的安卓系统自带的归档、Swift Backup（或 shell 命令）提供的停用功能（需要 root 身份）以及墓碑等软件的冻结功能来暂时停用该应用，待下次再玩时，相应地进行恢复、启用或解冻即可。
在某些情况下，玩家可能会将其卸载重装，这时候就需要做以下操作。

1. 卸载前需要使用 Swift Backup（推荐）、钛备份（仅在较早的时间和较低的安卓系统此选项可选）或 MT 管理器（可能会丢失文件的权限信息因此不建议）等工具备份数据 ``/data/user/${user}/${package}/``，其中 ${user} 为 Android 用户，通常为 0（机主），多用户模式下通常为 999（双开空间）或 10（工作账号），不需要备份位于 ``/sdcard/Android/`` 中的外部数据（基本上为广告缓存），${package} 为应用包名。
2. 因某些原因卸载了此应用，随后希望重新安装此应用并恢复游戏进度。
3. 使用 Swift Backup 开启还原 SSAID 后执行还原（可能需要重启手机），或使用其它工具将备份回写 ``/data/user/${user}/${package}/`` 并修正权限、所有者（UID）和用户组（GID）。如此类信息丢失，请设置所有文件权限为 ``0644``、所有目录权限为 ``0755``、及所有文件和目录的所有者和用户组为应用 ${package} 的 UID。

##### 2.2 跨设备迁移

下列程序同时适用于 **Cut the Rope Free（割绳子免费版）**、**Cut the Rope 2（割绳子 2）**与 **Cut the Rope Time Travel（割绳子：时光旅行）**本地存档的跨设备迁移。

1. 在源设备上强制停止游戏，使用 Swift Backup 等备份还原软件备份应用及数据（要求能够跨设备还原），无需备份外部数据，或使用 MT 管理器提取安装包，并全选 ``/data/user/${user}/${package}/`` 中的目录和文件进行压缩。
2. 记录源设备上 ${package} 的 SSAID，并保留原始首选项 XML，作为预期数量的依据。
3. 通过备份还原软件还原、MT 管理器安装或谷歌商店（不推荐）在目标设备上安装同一兼容版本的游戏，但不得开始正常游玩。
4. 在目标设备上强制停止目标游戏，将源软件包的完整数据还原或解压到目标软件包目录。
5. 将全部复制文件的权限修正（可参考 2.1），将 UID 与 GID 设置为目标设备上 ${package} 的 UID 与 GID。
6. 此时打开游戏会发现游戏关卡进度正常而受保护数量被置 0 的现象，于是需要重算 ``/data/user/${user}/com.zeptolab.ctr.ads/shared_prefs/CtrApp.xml``（Cut the Rope Free）、``/data/user/${user}/com.zeptolab.ctr2.f2p.google/shared_prefs/CTR2.xml``（Cut the Rope 2）或 ``/data/user/${user}/com.zeptolab.timetravel.free.google/shared_prefs/TIMETRAVEL.xml``（Cut the Rope Time Travel）中受保护数量的哈希值。
7. 修改完成后可以通过 MT 管理器手动打开上述 XML，确认每一项预期数量均有相应的新散列值。
8. 启动并进入游戏，核验关卡进度和受保护道具数量。

##### 2.3 重算受保护数量的哈希值

经研究，游戏存档主要集中于上述的 XML 文件中，其中主要关卡进度以普通本地数据形式保存，通常能够直接复制，而受保护数量（如 Cut the Rope Free 中的金币、提示、磁铁及超能力）则采用不同的处理方式，即每项受保护数量均附有一个完整性校验值。
通过清空游戏数据并完成前两关观察发现，同一设备上相同数量的完整性检验值相同，且完整性检验值符合 MD5 摘要（由 32 个十六进制字符表示的 128 个比特）的特征，推断该值为由相应数量与应用程序的 SSAID 共同派生的 MD5 摘要。

SSAID 是应用程序通过 ``Settings.Secure.ANDROID_ID`` 获取的值。在 Android 8.0 及更高版本中，该值由设备、Android 用户及应用签名密钥的组合共同确定。因此，将数据迁移至另一台设备后，SSAID 通常会发生变化。恢复出厂设置、更换签名密钥或者执行某些自定义系统操作，也可能造成该值变化。

将存档从源环境复制到目标环境后，XML 文件中仍保存着使用源 SSAID 计算所得的散列值，而目标游戏会使用自身可见的 SSAID 重新计算散列值。如果存储值与预期结果不符，游戏将相应受保护数量判定为无效，并可能将其重置为零。因此，跨迁移后基本会出现关卡进度仍然存在，但受保护数量被清零的情况。

上述机制属于完整性校验，而非可逆加密，且MD5 是单向函数，因此在恢复数据时，应当考虑从受保护数量重算受保护数量的完整性检验值而非从受保护数量的完整性检验值推算受保护数量。
考虑到受保护数值和 SSAID 两个要素，在恢复数据时，可以直接修改目标设备上应用的 SSAID 为源设备上应用读取到的 SSAID 以避免修改 XML 文件或重算完整性检验值，
也可以选择不更改目标设备上应用读取到的 SSAID，逐个重算受保护数值。有关后者完整性检验值重算的信息，请参阅下节。

##### 3. Cut the Rope Free（割绳子免费版）

##### 3.1 版本警告

约于 2026 年 9 月，**Cut the Rope Free（割绳子免费版）**通过一次更新更名为 **Cut the Rope（割绳子）**。
经测试，该次更新在存档连续性和关卡访问方面存在较为严重的倒退，更新后的版本发生进度丢失（如已经打通全部关卡的玩家可能丢失最后一个盒子的全部进度数据），
取消了常规选关功能，并要求玩家将每组三关作为一个连续流程完成（如未能完成全部三关下次进入时只能从该组三关的第一关重新开始）。

据此，强烈建议继续使用 **3.79.0** 版本，不要安装上述更新，并酌情关闭应用商店的自动更新功能（或通过 HMA 或其变体排除谷歌应用商店对该应用的可见性）。
如已完成更新，建议使用核心破解启用允许降级安装应用后降级安装应用，必要时可进一步使用备份还原软件还原数据。

##### 3.2 相关存档数据

谷歌免费版的包名为 ``com.zeptolab.ctr.ads``。对于 Android 用户 ``0``，相关首选项文件通常位于 ``/data/user/0/com.zeptolab.ctr.ads/shared_prefs/CtrApp.xml``，受保护数量按下列数量字段与完整性字段成对存储。

| 受保护数量 | 数量字段 | 完整性检验值字段 |
| - | - | - |
| 金币（coin） | ``PREFS_COINS_COUNT`` | ``PREFS_COINS_COUNT_HASH`` |
| 提示（hint） | ``PREFS_HINTS_COUNT`` | ``PREFS_HINTS_COUNT_HASH`` |
| 磁铁（magnet） | ``PREFS_MAGNETS_COUNT`` | ``PREFS_MAGNETS_COUNT_HASH`` |
| 超级能力（superpower） | ``PREFS_SUPERPOWERS_COUNT`` | ``PREFS_SUPERPOWERS_COUNT_HASH`` |

##### 3.3 完整性校验值计算方法

对于每一项受保护数量，游戏均采用下列结构计算完整性校验值。

```text
MD5(SSAID + "AngryCats" + decimal_count)
```

各部分直接连接，不使用分隔符，``decimal_count`` 为相应整数的普通十进制表示，不添加前导零。最终摘要以 32 个小写十六进制字符写入文件。

例如，目标 SSAID 为 ``0123456789abcdef``，预期数量为 ``50`` 时，MD5 的输入内容如下。

```text
0123456789abcdefAngryCats50
```

也就是说，以金币为例，仅修改 ``PREFS_COINS_COUNT`` 并不能使数据生效，必须使用相同的数量和目标安装实际可见的 SSAID，重新计算 ``PREFS_COINS_COUNT_HASH``，否则，受保护数量将会被重置为 0，其余受保护数量类似。

##### 3.4 恢复工具

配套的 [``ctr.cpp``](https://github.com/LRFP-Team/Bypasser/blob/main/cpp/ctr.cpp) 已经实现上述完整性检验值的计算方法。
使用 Android NDK 对应架构的 ``clang++`` 编译生成二进制文件 ``ctr`` 后，可将所得二进制文件推送至设备并以 root 权限运行（推荐通过 MT 管理器或 Termux 执行）。
该工具能够自动定位 ``CtrApp.xml``、获取当前 SSAID、更正受保护数量的完整性检验值、修改 SSAID 及修改受保护数量为指定数量等。

若系统所有应用共用一个系统层面的 SSAID，修改系统 SSAID 会对其它应用产生影响，因此不推荐。使用任何写入选项前，应当先查看该工具的帮助信息（``ctr --help``），并保留一份未经修改的 ``CtrApp.xml``。

##### 4. Cut the Rope 2（割绳子 2）

##### 4.1 相关存档数据

谷歌免费版的包名为 ``com.zeptolab.ctr2.f2p.google``。对于 Android 用户 ``0``，相关首选项文件通常位于 ``/data/user/0/com.zeptolab.ctr2.f2p.google/shared_prefs/CTR2.xml``，受保护数量按照下表存储。

| 受保护数量 | 数量字段 | 完整性检验值字段 | 参与散列计算的逻辑键名 |
| - | - | - | - |
| 气球 | ``com.zeptolab.ctr2.f2p.balloons`` | ``com.zeptolab.ctr2.f2p.balloons_HASH`` | ``com.zeptolab.ctr2.f2p.balloons`` |
| 炸弹 | ``com.zeptolab.ctr2.f2p.bombs`` | ``com.zeptolab.ctr2.f2p.bombs_HASH`` | ``com.zeptolab.ctr2.f2p.bombs`` |
| 金币 | ``com.zeptolab.ctr2.f2p.coins`` | ``com.zeptolab.ctr2.f2p.coins_HASH`` | ``com.zeptolab.ctr2.f2p.coins`` |
| 免费金币 | ``com.zeptolab.ctr2.f2p.coins_free`` | ``com.zeptolab.ctr2.f2p.coins_free_HASH`` | ``com.zeptolab.ctr2.f2p.coins_free`` |
| 提示 | ``com.zeptolab.ctr2.f2p.hints`` | ``com.zeptolab.ctr2.f2p.hints_HASH`` | ``com.zeptolab.ctr2.f2p.hints`` |
| 传送 | ``com.zeptolab.ctr2.f2p.teleports`` | ``com.zeptolab.ctr2.f2p.teleports_HASH`` | ``com.zeptolab.ctr2.f2p.teleports`` |
| 无限金币 | ``com.zeptolab.ctr2.f2p.coins_unlim`` | ``com.zeptolab.ctr2.f2p.coins_unlim_HASH`` | ``f2p.coins_unlim`` |

除无限金币外，每项受保护数量还对应 ``_BOUGHT``、``_EARNED_FOR_FREE`` 与 ``_USED`` 三个字段，基础数量须满足 ``current = bought + earned_for_free - used``。
修改受保护数量时，除更新完整性检验值外，还须使上述等式字段相互一致。

##### 4.2 完整性校验值计算方法

《Cut the Rope 2》与第一代采用相同的总体设计，但使用了另一种针对具体字段的计算结构。设 ``N`` 为数量的十进制表示，``K`` 为上表所列逻辑键名。
对于普通受保护数量，``K`` 是包含 ``com.zeptolab.ctr2.`` 前缀的完整首选项键名，即 ``com.zeptolab.ctr2.f2p.coins_unlim``。唯一的例外是无限金币，其使用 ``f2p.coins_unlim``。完整性校验值计算公式如下。

```text
MD5(N + "!don'thackthis!" + K + "!" + N + "!" + SSAID + "!ctr2.")
```

例如，普通金币数量为 ``50``，SSAID 为 ``0123456789abcdef`` 时，MD5 的准确输入内容如下。

```text
50!don'thackthis!com.zeptolab.ctr2.f2p.coins!50!0123456789abcdef!ctr2.
```

数量在输入内容中出现两次，逻辑键名也参与摘要计算。因此，即使数值相同，普通金币的有效完整性检验值也不能用于免费金币或者无限金币。
与第一代相同，将 ``CTR2.xml`` 复制到 SSAID 不同的设备后，原完整性检验值将不再有效，并可能导致受保护数量被重置。

对 **1.46.0** 与 **1.47.2** 两个版本进行静态差分并在断网实机上验证后，可以确认两版使用相同的格式模板，且均从受保护字段对象中取得 ``K``。此前文档对普通字段使用短键名的说法属于分析错误，并非后续游戏更新引入的变化。

##### 4.3 恢复工具

配套的 [``ctr2.cpp``](https://github.com/LRFP-Team/Bypasser/blob/main/cpp/ctr2.cpp) 已经实现上述完整性检验值的计算方法。
使用 Android NDK 对应架构的 ``clang++`` 编译生成二进制文件 ``ctr2`` 后，可将所得二进制文件推送至设备并以 root 权限运行（推荐通过 MT 管理器或 Termux 执行）。
该工具能够自动定位 ``CTR2.xml``、获取当前 SSAID、更正受保护数量的完整性检验值、修改 SSAID 及修改受保护数量为指定数量等。

例如，下列命令会将金币设为 ``100``、提示设为 ``50``，使全部受支持的 ``bought + earned - used`` 等式与当前数量一致，并更新上述两项的完整性检验值。

```bash
ctr2 -s coin 100 hint 50 -e all -u coin hint
```

若系统所有应用共用一个系统层面的 SSAID，修改系统 SSAID 会对其它应用产生影响，因此不推荐。使用任何写入选项前，应当先查看该工具的帮助信息（``ctr2 --help``），并保留一份未经修改的 ``CTR2.xml``。

##### 5. Cut the Rope Time Travel（割绳子：时光旅行）

##### 5.1 相关存档数据

谷歌免费版的包名为 ``com.zeptolab.timetravel.free.google``。对于 Android 用户 ``0``，相关首选项文件通常位于 ``/data/user/0/com.zeptolab.timetravel.free.google/shared_prefs/TIMETRAVEL.xml``。在 **1.20.0** 版本中，于该文件内确认的受保护道具数量按照下表存储。

| 受保护数量 | 数量字段 | 完整性检验值字段 |
| - | - | - |
| 提示（hint） | ``PREFS_HINTS_COUNT`` | ``PREFS_HINTS_COUNT_HASH`` |
| 超级能力（superpower） | ``PREFS_SUPERPOWERS_COUNT`` | ``PREFS_SUPERPOWERS_COUNT_HASH`` |

##### 5.2 完整性校验值计算方法

**Cut the Rope Time Travel（割绳子：时光旅行）**与上述两款游戏采用相同的总体设计，但使用了自身专有的固定字符串。对于两项受保护数量，设 ``decimal_count`` 为数量的普通十进制表示，则完整性校验值计算公式如下。

```text
MD5(SSAID + "ctr_tt_hd" + "_Shame" + " 0n You " + decimal_count)
```

各部分直接连接。``0n You`` 前后的空格均属于输入内容，且首字符是数字 0，而非英文字母 O。数量不添加前导零，最终摘要以 32 个小写十六进制字符写入文件。

例如，数量为 ``50``，SSAID 为 ``0123456789abcdef`` 时，MD5 的准确输入内容如下。

```text
0123456789abcdefctr_tt_hd_Shame 0n You 50
```

对 **1.20.0** 版本的静态分析表明，原生代码通过 ``Device::getUDID()`` 获取相应数值，而该调用最终执行 ``Settings.Secure.getString(..., "android_id")``。
随后，程序将 SSAID、固定字符串与数量格式化为字符串，将其转换为 UTF-8，计算 MD5，并将所得小写十六进制摘要与 XML 文件中的存储值进行比较。
因此，尽管原生方法名中包含 UDID，实际参与计算的设备相关要素仍是应用程序的 SSAID。

提示与超能力共用上述计算结构，字段名不参与摘要计算。因此，在同一 SSAID 下，两项道具数量相同时会生成相同的完整性校验值。
所提供存档中两者均为 0 且存储值相同，也与该结论一致。
将 ``TIMETRAVEL.xml`` 复制到 SSAID 不同的安装环境后，两项原始存储值均会失效，并可能导致相应道具数量被重置。

##### 5.3 恢复工具

配套的 [``ctrtt.cpp``](https://github.com/LRFP-Team/Bypasser/blob/main/cpp/ctrtt.cpp) 已经实现上述完整性检验值的计算方法。
使用 Android NDK 对应架构的 ``clang++`` 编译生成二进制文件 ``ctrtt`` 后，可将所得二进制文件推送至设备并以 root 权限运行（推荐通过 MT 管理器或 Termux 执行）。
该工具能够自动定位 ``TIMETRAVEL.xml``、获取当前 SSAID、更正提示与超级能力的完整性检验值、修改 SSAID，以及将任一受保护数量修改为指定值。

若系统所有应用共用一个系统层面的 SSAID，修改系统 SSAID 会对其它应用产生影响，因此不推荐。使用任何写入选项前，应当先查看该工具的帮助信息（``ctrtt --help``），并保留一份未经修改的 ``TIMETRAVEL.xml``。

##### 6. 呼吁

希望官方应当提供合适的方式供普通用户进行游戏存档迁移。
