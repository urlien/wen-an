# 文案仓库 (wen-an)

游戏文案/剧本的归档仓库，供AI分析和翻译使用。

## 仓库结构

- **催眠游戏/** - 催眠遊戯 (Hypno Game) 加密数据包
  - data0~data8.pack - KiriKiri Z 引擎加密归档
  - data8.pack 是密钥文件 (KeyFile ver1.0, game ID: crossover)
  - large-packs/ - data1~4 按10MB分块上传 (合并后可用)
  - System/config.txt - 引擎配置
- **柠檬工厂/** - Limelight Lemonade Jam 解包文件
  - *.ks.scn - PSB编译剧本 (138个，需反编译为.ks)
  - *.tjs - TJS2系统脚本 (31个)
  - 其他配置和UI文件

## 游戏信息

### 催眠遊戯 (Hypno Game)
- 开发商: スタッフィング (Stuffing)
- 引擎: KiriKiri Z
- 数据格式: .pack (KiriKiri加密归档)
- 密钥: data8.pack (KeyFile ver1.0, game ID: crossover)
- 需要工具: GARbro / KrkrExtract 解密

### 大文件分块合并 (data1~4)
Linux: `cat data1.pack.part* > data1.pack`
Windows PowerShell: `Get-Content data1.pack.part* -Raw -Encoding Byte | Set-Content data1.pack`

### 柠檬工厂 (Limelight Lemonade Jam)
- 引擎: Kirikiri Z
- 来源: unencrypted.xp3 (汉化补丁层，未加密)
- 剧本: .ks.scn 是 PSB 二进制编译格式
- 路线: 主线(バンド)70+章, 月望19章, 杏珠25章, 美玖14章
- 需要工具: GARbro (PSB解码)

## 解密/解包需求
- 催眠游戏: KiriKiri Z pack解密 + KeyFile
- 柠檬工厂: PSB -> .ks 反编译
- 推荐工具: GARbro (同时支持两种格式)