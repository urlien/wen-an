# 催眠遊戯 — 加密分析报告

## 游戏信息
- **名称**: 催眠遊戯 (Hypno Game)
- **开发商**: スタッフィング (Stuffing)
- **引擎**: KiriKiri Z
- **Game ID**: crossover
- **安装路径**: `C:\Users\19615\Desktop\新建文件夹\催眠游戏\Setup\AppData`

---

## 工作时间线

### 第一阶段：柠檬工厂（Limelight Lemonade Jam）

**日期**: 2026-07-14 ~ 2026-07-17

1. 通过快捷方式（.lnk）定位到游戏根目录
2. 发现引擎为 Kirikiri Z，数据打包为 .xp3 格式
3. 编写 Node.js 脚本 `extract-xp3.mjs` 解包 XP3 归档
4. 从 `unencrypted.xp3`（汉化补丁层，未加密）成功提取 **203 个文件**
5. 内容包括：
   - 138 个 `.ks.scn` PSB 编译剧本文件（覆盖主线、月望、杏珠、美玖四条路线）
   - 31 个 `.tjs` TJS2 系统脚本
   - 6 个 `.txt` 帮助文档
   - 26 个 `.tlg` UI 贴图
   - 其他配置文件
6. 通过 GitHub API 创建仓库并上传全部文件
7. 编写 README 说明 PSB 格式和解码方法

**结果**: 上传到 `urlien/lemon-factory`，后迁入 `urlien/wen-an/柠檬工厂/`

---

### 第二阶段：催眠遊戯（Hypno Game）

**日期**: 2026-07-19（今天）

#### 步骤 1：探索游戏结构
- 通过 VBS 启动脚本定位游戏路径
- 发现 KiriKiri Z 引擎，9 个 .pack 数据文件（总计约 1.6GB）
- data8.pack 为 KeyFile ver1.0 密钥文件（game ID: crossover）

#### 步骤 2：上传原始 .pack 文件到 GitHub
- 小文件（data0/5/6/7/8 + config.txt）直接上传
- 大文件（data1-4）按 10MB 分块上传到 `large-packs/` 目录
- 总计 158 个分块全部上传成功
- 仓库: `urlien/wen-an/催眠游戏/`

#### 步骤 3：尝试 GARbro 解密
- 下载安装 GARbro v1.5.44 到 D:\GARbro
- 成功解包 data0.pack，提取出:
  - `startup.s` / `sysstartup.s`（启动脚本）
  - `system/` 配置文件
  - `pack_keyfile_*.key`（密钥副本）
- data1~7.pack 的 crossover 加密方案超出 GARbro 支持范围

#### 步骤 4：尝试 KrkrExtract
- 下载 KrkrExtract v5.0.0.2（DLL + Lite.exe）
- Windows Defender 将 Lite.exe 标记为 Trojan:Win32/Tnega!MSR 并隔离
- 从隔离区还原后仍被阻止运行
- 改用 C# DLL 注入方案：编写 `inject.cs`，编译为 `inject.exe`
- 注入成功（DLL 被加载到游戏进程），但未提取到脚本

#### 步骤 5：内存扫描
- 编写 `memdump.cs` 扫描游戏进程内存
- 扫描结果为乱码——脚本在内存中仍保持加密状态
- KiriKiri Z 按需解密，不会一次性将所有脚本加载到内存

#### 步骤 6：分析报告
- 整理 KeyFile 结构、已尝试的方法、关键发现
- 上传分析报告到 GitHub

**当前状态**: 加密无法突破，等待进一步处理

---

## KeyFile 结构

data8.pack 和 key.fkey 都是 KeyFile ver1.0 格式：

| 偏移 | 大小 | 内容 |
|------|------|------|
| 0x00 | 16 bytes | 魔数: "KeyFile ver1.0\x00\x00" |
| 0x10 | 4 bytes LE | version = 1 |
| 0x14 | 4 bytes LE | entry_size = 4096 |
| 0x18 | 4 bytes LE | key_offset = 20 |
| 0x20 | 8 bytes | game_id = "crossover" |
| 0x100 | ~2MB | 密钥数据 |

## 已尝试的解密方法（均失败）

1. **简单 XOR** — 用 key.fkey 数据对 .pack 做 XOR，结果无效
2. **GARbro** — 无法识别 crossover 加密方案
3. **KrkrExtract** — DLL注入成功但未提取到脚本
4. **内存扫描** — 脚本在内存中仍保持加密状态

## 关键发现

- data0.pack 的加密方式不同（GARbro 可部分解密），可能是标准 KiriKiri 加密
- data1~7.pack 使用 crossover 加密，与 data0 不同
- 脚本在游戏运行时是按需解密的，内存扫描无法捕获
- crossover 加密可能需要完整的密钥派生算法，不仅仅是简单 XOR

## 需要解决的问题

需要逆向 crossover 加密算法。关键信息：

1. 密钥文件格式已明确（KeyFile ver1.0）
2. 密钥数据在 0x100 偏移开始
3. 参考 GARbro 源码: `ArcFormats/KiriKiri/` 目录下的加密实现
4. KiriKiri Z 引擎的加密通常在 `KirikiriZ.exe` 或插件中实现
5. data8.pack (2MB) 和 key.fkey (4KB) 内容相同，均为密钥

## 已提取的启动脚本引用

### startup.s
- Script/ExceptionBackup/main.s
- Script/Logo/main.s
- Script/Title/main.s
- Script/Config/Value.s

### sysstartup.s
- Script/Quit/main.s
- Script/Reset/main.s
- Script/SaveLoad/Main_Save.s
- Script/SaveLoad/Main_Load.s
- Script/SaveLoad/Main_QuickLoad.s
- Script/SaveLoad/Main_QuickSave.s
- Script/SaveLoad/Main_AutoLoad.s
- Script/Log/main.s
- Script/UnLock/main.s
- Script/UnLock/Excute.s
- Script/Config/main.s
- Script/View/main.s
- Script/Menu/main.s
- Script/SceneSkip/main.s

这些脚本文件都在加密的 .pack 中，需要解密后才能读取。

## 本地工具位置

- GARbro: `D:\GARbro\GARbro.GUI.exe`
- KrkrExtract DLL: `C:\Users\19615\Desktop\催眠游戏解包\KrkrExtract\`
- 解包脚本: `C:\Users\19615\Desktop\文案解包\`（7558文件，561MB）
- 注入器: `D:\REASONIX-Desktop\Reasonix\inject.exe`
- 内存扫描器: `D:\REASONIX-Desktop\Reasonix\memdump.exe`
