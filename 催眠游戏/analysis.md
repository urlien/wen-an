# 催眠遊戯 — 加密分析报告

## 游戏信息
- **名称**: 催眠遊戯 (Hypno Game)
- **开发商**: スタッフィング (Stuffing)
- **引擎**: KiriKiri Z
- **Game ID**: crossover

## 文件结构

### GameData 目录
- data0.pack (18KB) — 启动脚本（可部分解密）
- data1.pack (361MB) — 加密游戏数据
- data2.pack (201MB) — 加密游戏数据
- data3.pack (712MB) — 加密游戏数据（最大）
- data4.pack (273MB) — 加密游戏数据
- data5.pack (7MB) — 加密游戏数据
- data6.pack (1.2MB) — 加密游戏数据
- data7.pack (10.9MB) — 加密游戏数据
- data8.pack (2MB) — KeyFile 密钥文件
- System/config.txt — 引擎配置（Shift-JIS编码）

### 已解密文件（从 data0.pack 提取）
- startup.s — 游戏启动脚本
- sysstartup.s — 系统脚本
- system/etc/debugskip.b — 调试配置
- system/etc/game.dat — 游戏设置
- system/etc/message.dat — 系统消息文本
- system/font/fontdata.dat — 字体配置
- system/voice/voicedata.dat — 语音配置

## KeyFile 结构

data8.pack 和 key.fkey 都是 KeyFile ver1.0 格式：

| 偏移 | 大小 | 内容 |
|------|------|------|
| 0x00 | 16 bytes | 魔数: "KeyFile ver1.0" |
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

## 需要解决的问题

需要逆向 crossover 加密算法。关键信息：

1. 密钥文件格式已明确（KeyFile ver1.0）
2. 密钥数据在 0x100 偏移开始
3. 参考 GARbro 源码: `ArcFormats/KiriKiri/` 目录下的加密实现
4. KiriKiri Z 引擎的加密通常在 `KirikiriZ.exe` 或插件中实现

## startup.s 引用的脚本
- Script/ExceptionBackup/main.s
- Script/Logo/main.s
- Script/Title/main.s
- Script/Config/Value.s

## sysstartup.s 引用的脚本
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
