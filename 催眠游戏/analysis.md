# 给逆向分析 AI 的任务说明

## 目标

逆向催眠遊戯（Hypno Game）的 `.pack` 文件加密算法，实现解密脚本。

## 你需要的文件

已上传到本仓库 `催眠游戏/re-for-ai/` 目录：

| 文件 | 用途 |
|------|------|
| `催眠遊戯.exe` (1.6MB) | **核心目标** — 包含解密算法的二进制代码 |
| `エンジン設定.exe` (634KB) | 引擎配置工具，可能包含相关代码 |
| `DLL/key.fkey` (4KB) | KeyFile 密钥文件（data8.pack 的精简版，内容相同） |
| `GameData/data0.pack` (18KB) | 最小的加密包，可用来验证解密是否成功 |
| `GameData/data5.pack` (7MB) | 中等大小的加密包，验证用 |
| `GameData/data6.pack` (1.2MB) | 中等大小的加密包，验证用 |
| `GameData/data7.pack` (10.9MB) | 中等大小的加密包，验证用 |
| `GameData/data8.pack` (2MB) | 完整 KeyFile（与 key.fkey 内容相同） |
| `GameData/System/config.txt` | 引擎配置 |

大文件（data1-4）已在 `催眠游戏/large-packs/` 中以 10MB 分块上传。

## 已知信息

### KeyFile 结构（data8.pack / key.fkey）
```
偏移 0x00: "KeyFile ver1.0\x00\x00" (16字节魔数)
偏移 0x10: version = 1 (uint32 LE)
偏移 0x14: entry_size = 4096 (uint32 LE)
偏移 0x18: key_offset = 20 (uint32 LE)
偏移 0x20: game_id = "crossover" (8字节)
偏移 0x100+: 密钥数据 (~2MB)
```

### 催眠遊戯.exe 中的关键字符串
- `KeyFile ver1.0` — 出现在偏移 1463436 和 1464704
- `PACK_KeyFile_kfueheish15538fa9or.key` — 出现在偏移 1472409
- `inflate 1.1.4` — zlib 解压（偏移 1505333）

### 加密流程（推测）
1. 游戏读取 KeyFile（key.fkey 或 data8.pack）
2. 使用 KeyFile 中的密钥数据对 .pack 文件进行 crossover 解密
3. 解密后用 zlib（inflate）解压得到明文数据

### 已尝试失败的方法
- 简单 XOR — 结果无效
- GARbro — 不支持 crossover 方案
- KrkrExtract DLL 注入 — 未提取到脚本
- 内存扫描 — 脚本在内存中仍加密

## 建议的分析步骤

1. **用 Ghidra / IDA Pro 分析催眠遊戯.exe**
   - 搜索字符串 `PACK_KeyFile_kfueheish15538fa9or.key`
   - 追踪引用该字符串的函数
   - 分析 .pack 文件读取和解密逻辑

2. **对比 data0.pack 的已知解密结果**
   - data0.pack 可被 GARbro 部分解密（提取出 startup.s 等）
   - 比较解密前后的数据，推断算法

3. **参考已有实现**
   - GARbro 源码: `ArcFormats/KiriKiri/CryptAlgorithms.cs` 和 `KiriKiriCx.cs`
   - GalgameReverse: `YuriSizuku/GalgameReverse/project/krkr/src/krkr_hxcrypt.py`
   - 这些实现了 KiriKiri Cx 加密，crossover 可能是其变种

4. **编写 Python 解密脚本**
   - 输入: .pack 文件 + key.fkey
   - 输出: 解密后的 .xp3 或明文数据
   - 验证: data0.pack 解密后应与 GARbro 提取结果一致

## 仓库结构

```
wen-an/
├── 柠檬工厂/        — 已解包的 Limelight Lemonade Jam
├── 催眠游戏/
│   ├── large-packs/ — data1-4 的 10MB 分块
│   ├── re-for-ai/   — 逆向分析用的关键文件 ← 从这里开始
│   ├── analysis.md  — 本文件
│   └── *.pack       — 小文件直接上传
├── tools/
│   ├── GARbro/      — 通用解包工具
│   └── KrkrExtract/ — 内存提取工具
└── README.md
```
