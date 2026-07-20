# 文案仓库 (wen-an)

游戏文案/剧本的归档仓库，供AI分析和翻译使用。

## 仓库结构

- **柠檬工厂/** — Limelight Lemonade Jam 解包文件 (203文件)
- **催眠游戏/** — 催眠遊戯 原始加密数据 + 解包资源 (166文件)
- **tools/** — 解密/解包工具
  - **tools/GARbro/** — KiriKiri/KiriKiriZ 通用解包工具
  - **tools/KrkrExtract/** — KiriKiri 游戏内存提取工具

## 工具说明

### GARbro (推荐先试这个)

**用途**: 解包 KiriKiri/KiriKiriZ 引擎的游戏归档文件（.xp3, .pack, .sf等）

**特点**:
- 支持几百种视觉小说引擎的文件格式
- GUI 界面，直接导航到游戏目录即可识别
- 内置多种解密方案，部分游戏可自动解密
- 不需要游戏运行

**使用方法**:
1. 下载 `tools/GARbro/` 目录全部文件到本地
2. 双击 `GARbro.GUI.exe`
3. 在左侧文件浏览器导航到游戏的 .pack 或 .xp3 文件所在目录
4. 右键 → 提取（Extract）→ 选择输出目录

**已知限制**:
- 本仓库催眠遊戯的 crossover 加密方案不在 GARbro 支持范围内
- data0.pack 可以部分解密，data1~7.pack 无法解密

**来源**: https://github.com/morkt/GARbro (v1.5.44)

### KrkrExtract

**用途**: 从运行中的 KiriKiri/KiriKiriZ 游戏进程里实时提取解密后的脚本和资源

**特点**:
- Hook 游戏进程，在内存中拦截解密后的数据
- 不需要逆向加密算法，利用游戏自身的解密逻辑
- 支持提取脚本(.ks)、图片、音频等所有游戏资源

**使用方法**:
1. 下载 `tools/KrkrExtract/` 目录全部文件到本地
2. 将三个文件复制到游戏目录（和游戏exe同级）
3. 双击 `krkr_launcher.exe`
4. 在弹出窗口选择游戏的 .exe 文件
5. 游戏启动后会自动 hook 并导出文件

**已知限制**:
- 需要在 Windows 上运行（依赖 Windows API）
- 杀毒软件可能误报拦截（已知 Trojan:Win32/Tnega!MSR 误报）
- 本仓库催眠遊戯测试时 DLL 注入成功但未提取到脚本
- 脚本按需解密，仅 hook 启动阶段可能不够

**来源**: https://github.com/xmoezzz/KrkrExtract (v5.0.0.2)

### 选择建议

| 场景 | 推荐工具 |
|------|----------|
| 拿到游戏安装包，想快速查看文件 | GARbro |
| GARbro 无法解密 | KrkrExtract |
| 两个都不行 | 可能需要逆向加密算法 |

## 催眠遊戯 (Hypno Game)

**游戏信息**:
- 开发商: スタッフィング (Stuffing)
- 引擎: KiriKiri Z
- Game ID: crossover (加密方案名)

**加密情况**:
- data0.pack — 可被 GARbro 部分解密
- data1~7.pack — crossover 加密，目前无法解密
- data8.pack — KeyFile 密钥文件，非压缩包

**已解密内容**:
- startup.s / sysstartup.s（启动脚本）
- system/ 配置文件（game.dat, message.dat 等）

**详细分析**: 见 `催眠游戏/analysis.md`

### 大文件分块

data1~4 因超过 GitHub 100MB 限制，在 `催眠游戏/large-packs/` 按 10MB 分块上传。

合并命令:
```bash
# Linux/Mac
cat 催眠游戏/large-packs/data1.pack.part* > data1.pack

# Windows PowerShell
Get-Content 催眠游戏\large-packs\data1.pack.part* -Raw -Encoding Byte | Set-Content data1.pack
```

## 柠檬工厂 (Limelight Lemonade Jam)

**游戏信息**:
- 引擎: Kirikiri Z
- 来源: unencrypted.xp3（汉化补丁层，未加密）

**文件说明**:
- *.ks.scn — PSB 编译剧本文件（138个，需反编译为 .ks 纯文本）
- *.tjs — TJS2 系统脚本（31个，可直接读取）
- *.txt — 帮助文档（6个，可直接读取）
- *.tlg — UI 贴图（26个，非文本）

**剧本路线**:
- 主线(バンド) — 70+ 章
- 月望路线 — 19 章
- 杏珠路线 — 25 章
- 美玖路线 — 14 章

**PSB 解码**: .ks.scn 是 Polygonal String Binary 编译格式，参考 GARbro 源码中的 PSB 解析器。


## 工作日志

### 2026-07-21：催眠遊戯文案提取

**突破性发现：data8.pack (KeyFile) 直接包含解密后的全部脚本**

之前尝试的方法全部失败（XOR、GARbro、KrkrExtract、内存扫描），因为一直在尝试解密 .pack 文件。实际上 KeyFile 里已经有完整的解密脚本，不需要逆向加密算法。

**提取结果：**
- 来源：data8.pack (KeyFile ver1.0, game ID: crossover)
- 总条目：512 个
- 总对话：14,003 行
- 角色数：42 个（出场≥5次的有 26 个）
- 编码：Shift-JIS → UTF-8

**主要角色出场统计：**
| 角色 | 对话数 |
|------|--------|
| 柳（男主） | 6,465 |
| 舞夜 | 2,477 |
| 流衣 | 1,954 |
| 蛍火 | 1,400 |
| 沙流 | 467 |
| 紫云 | 365 |
| 汐見 | 294 |

**章节结构：**
- 01COM：公共路线
- 02MAI/05MAI/08MAI：舞夜路线（3周目）
- 03RUI：流衣路线
- 04KEI/06KEI/07KEI：蛍火路线（3周目）

**输出文件：** `催眠游戏/文案/` 目录
- 完整对话录.txt（14,003行）
- 角色_XXX.txt（26个角色分类文件）
- 摘要.md

**技术细节：**
- KeyFile 偏移 0x100 开始，每 4096 字节一个条目
- 条目 0 是加密头，条目 1-511 是脚本内容
- 用 Shift-JIS 解码可得到繁体中文翻译文本
- 脚本格式：KiriKiri 标签命令（\jmp, \ret, \sub 等）+ 对话（「」）
