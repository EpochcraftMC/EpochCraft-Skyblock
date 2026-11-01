# EpochCraft Skyblock

EpochCraft 空岛服（Skyblock）的完整服务端存档仓库：包含可直接运行的服务端核心、两个世界的存档数据，以及全部插件的配置与运行数据。

仓库定位为 **备份 / 复刻 / 二次开发参考**，不是插件源码工程。克隆后即可本地开服。

| 项目 | 说明 |
| --- | --- |
| 服务端核心 | Spigot 1.8.8（`git-Spigot-db6de12-18fbb24`，NMS `v1_8_R3`） |
| Java 版本 | Java 8（已在 Temurin 8.0.504 上验证启动） |
| 内存 | `-Xmx1G`（见 `START.bat`） |
| 端口 | `25565`（Query 同端口，已启用） |
| 正版验证 | 关闭（`online-mode=false`，离线模式） |
| 最大人数 | 200（`server.properties`；MOTD 插件显示 2022） |
| 世界 | `Server`（主城 / 综合玩法）、`Islands`（空岛世界） |
| 插件数量 | 67 个（全部可正常加载） |
| 交流群 | QQ 群 `139447432` |

## 快速开始

前置条件：安装 64 位 **JDK / JRE 8**，并确保 `java` 已加入 `PATH`。

```bat
:: 方式一：双击 START.bat
:: 方式二：手动启动（nogui 表示不弹出服务端 GUI 窗口）
java -Dfile.encoding=UTF-8 -Xmx1G -jar spigot.jar nogui
```

启动流程：

1. `eula.txt` 已置为 `eula=true`（表示同意 [Minecraft EULA](https://aka.ms/MinecraftEULA)）。
2. 首次生成 `ops.json`、`banned-players.json`、`whitelist.json` 等文件属于正常现象。
3. 看到 `Done (20.509s)! For help, type "help" or "?"` 即启动完成，客户端直连 `localhost:25565` 即可。
4. 无需 BungeeCord / Velocity 代理，单体服务端即可运行。

> 建议把 `-Xmx1G` 调到 `2G`~`4G`：当前配置同时加载 67 个插件、世界数据较多，1G 在生产环境容易触发 Full GC。

## 目录结构

```
.
├── START.bat              启动脚本（已 chcp 65001 处理中文编码）
├── spigot.jar             Spigot 1.8.8 服务端核心
├── server.properties      端口、难度、人数、离线模式等
├── bukkit.yml             刷怪上限、区块 GC、别名、关服提示
├── spigot.yml             Spigot 主配置
├── paper.yml / taco.yml / beer.yml / options.yml / knockback.yml
│                          历史遗留：PaperSpigot / TacoSpigot / BeerSpigot / ySpigot 的配置
│                          （当前 spigot.jar 不会读取，保留仅供追溯）
├── wepif.yml              WEPIF 权限配置（WorldEdit 系插件使用）
├── help.yml               Bukkit 帮助页
├── commands.yml           原版命令别名
├── permissions.yml        Bukkit 权限
├── server-icon.png        服务器图标
├── Server/                主城世界（含 DIM-1 / DIM1 / stats / data）
├── Islands/               空岛世界（IridiumSkyblock 生成器）
├── plugins/               67 个插件 jar + 插件数据目录
└── LICENSE                GNU GPL v3
```

## 世界说明

| 世界 | 用途 | 关键设置 |
| --- | --- | --- |
| `Server` | 主城 / 综合玩法世界 | 出生点 `0 / 88 / 0`，饥饿关闭，PvP 开启，难度 Easy |
| `Islands` | 空岛世界 | 生成器 `IridiumSkyblock`，出生点 `0 / 100 / 0`，饥饿开启，PvP 开启 |

- 下界与末地均已禁用：`allow-nether=false`、`bukkit.yml → allow-end: false`、Iridium 的 `netherIslands=false`。
- `Server/` 内存在 `level.dat_mcr`，说明该世界由远古（MCRegion 时代）存档逐代升级而来。
- 当前数据：**11 个岛屿**、**24 个 NPC**（`auction-master`、`weaponsmith`、`fisher`、`bank`、`rewards`、`sirius` 等）。

## 玩法系统

### 空岛核心

| 插件 | 版本 | 作用 |
| --- | --- | --- |
| IridiumSkyblock | 2.5.0 | 空岛主体：`/is` 建岛、等级、价值、任务、升级、银行、水晶、传送点 |
| IslandAdditions | 1.0.0 | 空岛扩展功能补充 |
| Skyblock（UltraSkills） | 3.2.0 | 技能、收集图鉴、村民交易、宠物、末影龙、矿场、符文、灵魂、护甲 |
| HypixelMinionsCore | 1.0 | Hypixel 风格小兵（Minions）系统 |
| HypixelRegions / HypixelRegionBlocks / HypixelWorldProtect | 1.1 / 1.0 / 1.2 | 区域划分、区域方块与破坏保护 |
| FirstRegionJoin | 1.0 | 玩家首次进入某区域时的引导 |

IridiumSkyblock 关键配置（`plugins/IridiumSkyblock/config.json`）：

- 岛屿世界 `Islands`，间距 `distance: 251`，单岛范围约 150×150。
- 方块限制：`SPAWNER` 最多 10 个、`HOPPER` 最多 50 个。
- 禁用实体：`PRIMED_TNT`、`MINECART_TNT`、`FIREBALL`、`SMALL_FIREBALL`、`ENDER_PEARL`。
- 岛上禁止 PvP（`disablePvPOnIslands`）、禁止爆炸（`allowExplosions: false`）。
- 岛屿重生成冷却 1 小时；每 60 分钟自动备份，7 天后清理旧备份。

### 经济与交易

| 插件 | 版本 | 作用 |
| --- | --- | --- |
| RoyaleEconomy | 1.4.2 | 经济核心：钱包 / 银行 / 共享银行 / 利息 / 击杀金币 / 护符 / 硬币袋 / 黑市 |
| Vault | 1.7.2-b107 | 经济与权限桥接 |
| PickupMoney | 2.5.2 | 击杀生物直接掉金币 |
| AuctionMaster + AuctionMasterItemDisplay | 2.3 / 1.1 | 拍卖行与展示实体 |
| DeluxeBazaar | 2.3.5 | 集市（Bazaar）买卖菜单 |
| DarkAuctions | 1.1.5 | 暗拍 / 黑市拍卖 |
| TradeSystem | B5 | 玩家间安全交易 |
| Rewards | 1.4 | 在线奖励 / 签到类奖励 |

### 物品与装备

`HypixelItems`(5.0.5)、`HypixelArmor`/Armor+(2.2.4)、`HypixelLore`(1.6)、`HypixelMenus`(1.10.5)、`HypixelBackpacks`(1.6.7)、`HypixelBags`(1.0)、`HypixelBooks`(1.4.2)、`HandyOrbs`(2.1)、`SpecialItems`(1.7)、`FancyCrafting`(1.0.0)，底层依赖 `NBTAPI`(2.5.0)。

### 战斗与移动

`HypixelCombat`(9.9.3.4)、`CannonPatches`(1.0，TNT 炮修复)、`GrapplingHook`(2.3，钩爪)、`JumpPads`(1.0)、`AdvanceRespawn`(7.1.3)、`DeathTeleport`(1.0)、`AntiDrop`(2.2.1)、`DamageIndicators`(1.0.0)、`HoloMobHealth`(2.0.7)。

### 界面、社交与信息

`TAB`(2.6.5)、`QuickBoard`(4.4.1，侧边栏)、`HypixelChat`(2.18)、`HypixelFriends`(1.5.12)、`HypixelCommands`(1.9.0，命令别名/伪装未知命令)、`HypixelMOTD`(0.2.1)、`HypixelLag`(3.1.6)、`RealTime`(1.0-C)、`TimeEmoji`(1.0)、`HolographicDisplays`(2.4.3)、`ImageOnMap`(3.1)、`Replay`(1.6)、`MessageFilter`(0.3.2)、`BetterProfiles`(1.3-BETA)、`SkinsRestorer`(14.2.3)。

### 管理与基础库

`LuckPerms`(5.1.26)、`Essentials`(2.x)、`WorldEdit`(6.1.7)、`FastAsyncWorldEdit`(19.11.13)、`WorldGuard`(6.1.3) + `WGRegionEvents`(1.0)、`Multiverse-Core`(2.5-b699)、`WorldSpawn`(1.4.4)、`Citizens`(2.0.27) + `CommandNPC`(1.9.1)、`Skript`(2.1.2)、`PlaceholderAPI`(2.10.5)、`ProtocolLib`(4.5.1)、`ConsoleSpamFix`(1.8.2)、`bStats`。

## 常用命令

| 命令 | 说明 |
| --- | --- |
| `/is`（别名 `/island`） | 空岛主命令，子命令：`create` `home` `join` `leave` `invite` `kick` `coop` `missions` `upgrade` `booster` `crystals` `value` `top` `warps` `visit` `public` `private` `bank` `shop` `biome` `fly` `permissions` 等 |
| `/trades` `/collections` | 村民交易界面 / 收集图鉴（UltraSkills） |
| `/auction`（`/ah` `/auctions` `/auctionhouse`） | 拍卖行 |
| `/bazaar`（`/bz`） | 集市买卖 |
| `/darkauction`（`/da`） | 暗拍 / 黑市拍卖 |
| `/trade <玩家>` `/tradeaccept` `/tradedeny` | 玩家间交易 |
| `/balance`（`/bal` `/purse` `/money`）/ `/bank` / `/sharedbank` / `/balancetop` / `/pay` | RoyaleEconomy 钱包、银行与转账 |
| `/interest` `/killcoins` `/talismans` `/moneybag` `/piggybank` `/reshop` `/blackmarket` | RoyaleEconomy 利息、击杀统计、护符、硬币袋、商店、黑市 |
| `/reward`（`/rw`） | 在线 / 周期奖励界面 |
| `/spawn` `/sethome` `/home` `/tpa` | Essentials + WorldSpawn 基础传送 |
| `/combatlogx` | 战斗记录（脱战计时等） |
| `/ssetup` `/usbc` | Skyblock（UltraSkills）管理命令 |

## 配置要点

- `server.properties`：`online-mode=false`（允许非正版登录）、`allow-flight=true`、`spawn-protection=0`、`difficulty=1`、`view-distance=6`、`enable-command-block=true`、`level-name=Server`。
- `bukkit.yml`：全部刷怪上限归零（`monsters/animals/water-animals/ambient: 0`），刷怪完全交给插件控制；区块 GC `400` tick；连接限流关闭。
- `plugins/RoyaleEconomy/config.yml`：默认钱包与银行余额均为 0，银行利息开启、冷却 31 小时，40 天不活跃自动清除数据。
- `plugins/WorldSpawn/config.yml`：主城 `Server 0/88/0`，重生回出生点（`spawn-on-respawn: true`）。
- `plugins/ConsoleSpamFix/config.yml`：屏蔽了大量插件控制台输出（含各插件的 `Enabling ...` 行），**排查启动问题时优先检查此文件**。

## 已知问题与注意事项

1. **重复的 RoyaleEconomy**：`plugins/RoyaleEconomy.jar`(1.4.2) 与 `plugins/EconomyFix.jar`(1.3.4) 的插件名相同，启动时会报
   `Ambiguous plugin name 'RoyaleEconomy' for files plugins\RoyaleEconomy.jar and plugins\EconomyFix.jar`。
   建议删除旧版 `EconomyFix.jar`。
2. **控制台日志被静音**：ConsoleSpamFix 的过滤器会吞掉大量日志，线上排错时容易误判"插件未启用"。
3. **内核版本与插件年代不一致**：多数插件声明 `api-version: 1.13+`，但实际运行在 1.8.8 上（服务端会忽略该字段）。当前版本组合可正常加载，但升级任何插件前建议先在测试服验证。
4. **内存偏小**：`START.bat` 仅分配 1G，长期运行建议增加。
5. **SkinsRestorer 会自动更新**：会自行写入 `plugins/update/SkinsRestorer.jar`，如需锁定版本请关闭更新检查。
6. **仓库包含玩家数据**：`LuckPerms` 的 H2 数据库、经济/岛屿/NPC 数据、玩家 UUID 等均在其中，公开或分享前请评估隐私与安全风险（也便于他人解密/复用数据）。
7. **jar 与存档直接入库**：仓库未提供 `.gitignore`，约 124 MB，包含 114 个 jar 文件；提交大文件时注意 Git 历史膨胀。
8. **插件第三方许可**：各插件 jar 分别遵循其原作者许可，本仓库的 GPL-3.0 仅覆盖仓库自身内容。

## 备份与迁移

需要备份的核心数据：

- 世界存档：`Server/`、`Islands/`
- 插件数据：`plugins/*/`（尤其 `LuckPerms/luckperms-h2.mv.db`、`IridiumSkyblock/islandmanager.json`、`RoyaleEconomy/`、`Essentials/`、`Citizens/saves.yml`）
- 配置：根目录 `*.yml`、`server.properties`、`plugins/**/config.yml`

迁移到新服时，请先统一内核版本（例如保留 1.8.8，或整体升级到对应插件都支持的 1.13+ / 1.16 内核），再按顺序迁移：世界存档 → 插件 jar → 插件数据。

## 许可与声明

- 本仓库内容以 [GNU GPL v3](./LICENSE) 发布。
- 本项目与 Mojang Studios / Microsoft 无关联，Minecraft 为其各自所有者的商标。
- 使用本仓库架设公开服务器所产生的一切后果由使用者自行承担。
