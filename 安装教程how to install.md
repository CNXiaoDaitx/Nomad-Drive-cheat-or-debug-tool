# Nomad Drive 调试面板 — 安装教程

> 这套面板是"绿色注入式"的：不改游戏文件、不装框架、不需要 .NET SDK，拷 4 个文件进去就能用，删掉 4 个文件就干净卸载。

---

## 一、装之前先确认

| 项目 | 要求 |
|---|---|
| 游戏 | Nomad Drive（Early Access 1.0，Unity 6 / Mono 后端），已能正常启动 |
| 系统 | Windows 10 / 11，64 位 |
| 权限 | 能往游戏目录写文件（不要装在 `Program Files` 里，否则用管理员权限） |
| 不需要 | BepInEx、MelonLoader、.NET SDK、Visual Studio、任何运行库 |

> 面板走的是 **Unity Doorstop 4.6**（`winhttp.dll` 代理 + 一个自写 dll），游戏本体一个字都没改。

---

## 二、安装（4 步）

### 第 1 步：备份（可跳过，但建议）

把整个游戏目录复制一份，或者至少备份 `winhttp.dll` 不存在这一点（原版游戏没有这个文件，所以删掉就等于还原）。

### 第 2 步：把文件拷进游戏根目录

游戏根目录 = `Nomad Drive.exe` 所在的那个文件夹（例如 `E:\你妈的drive\GAME17038\`）。需要放进这些东西：

```
GAME17038\
├─ Nomad Drive.exe              ← 已有的
├─ Nomad Drive_Data\            ← 已有的
│
├─ winhttp.dll                  ← 【新增】Doorstop 代理 DLL（必需）
├─ doorstop_config.ini          ← 【新增】Doorstop 配置（必需）
├─ .doorstop_version            ← 【新增】版本标记 4.6.0（建议一并放）
└─ doorstop\                    ← 【新增】文件夹
   ├─ LootDebugPanel.dll        ← 面板本体（必需）
   ├─ panel.log                 ← 运行后自动生成，不用拷
   ├─ items.tsv                 ← 运行后按 F4 导出，不用拷
   └─ zh.tsv                    ← 运行后按 F1 导出，不用拷
```

> 只需要拷 **`winhttp.dll`、`doorstop_config.ini`、`.doorstop_version`、`doorstop\LootDebugPanel.dll`** 这 4 个（`doorstop` 文件夹里其它文件是运行/导出产物，可以不拷）。

### 第 3 步：检查 `doorstop_config.ini`

用记事本打开，确认这几行（其它行保持默认即可）：

```ini
[General]
enabled = true
target_assembly=doorstop\LootDebugPanel.dll
redirect_output_log = false

[UnityMono]
dll_search_path_override =
debug_enabled = false
```

注意：
- `target_assembly` 用**反斜杠**、**相对路径**，不要写绝对路径；
- 别给 `target_assembly` 加引号；
- 路径大小写不敏感，但 `doorstop` 文件夹名要和这里一致。

### 第 4 步：启动游戏

直接双击 `Nomad Drive.exe`（不要用 Steam 启动器抢注入，Steam 启动也一样能用，因为 Doorstop 是进程内代理）。

进入主菜单后：

1. 按 **F10** → 屏幕上应该出现「Nomad Drive 物品生成 / 参数调试面板」；
2. 看 `GAME17038\doorstop\panel.log`，正常内容开头是：

```
=== Doorstop entry called (build v2) ===
hooked Camera.onPreRender + sceneLoaded (startup)
scene loaded: Bootstrap
PanelBehaviour.Awake
font: Microsoft YaHei UI
panel GameObject created
```

看到 `Doorstop entry called` 就说明注入成功；看到 `panel GameObject created` 就说明面板已经挂在游戏里了。

---

## 三、验证清单（装完花 30 秒确认）

| 检查 | 期望 |
|---|---|
| 按 F10 | 面板出现 / 收起，不弹游戏的「报告问题」框 |
| 主菜单就能看到左列物品 | 显示 `物品生成  227 / 227 件` |
| 进单人游戏后 | 右栏「玩家」页出现血量/饱食度等数值 |
| 点任意一行物品 | 底部状态行出现 `已生成 xxx  位置 x,y,z`，准星前方出现物品 |
| 点「天气」页 | 显示 `当前 晴 [自动] Enviro: Clear Sky`，下面有 15 个天气按钮 |

---

## 四、常见问题

**1. 按 F10 没反应，`panel.log` 也没有**

- `winhttp.dll` 被杀毒软件/Windows Defender 删了 → 加白名单后重新拷；
- `doorstop_config.ini` 里 `enabled = false` 或 `target_assembly` 路径写错；
- 游戏不是从本目录启动的（有些启动器会 cd 到别处）→ 用 `Start-Process "…\Nomad Drive.exe" -WorkingDirectory "…\GAME17038"`；
- 确认是 64 位游戏（本作是 x64，`winhttp.dll` 也是 x64 版）。

**2. 日志里有 `BepInEx ... MissingMethodException: Module.GetPEKind`**

说明游戏目录里还有旧的 BepInEx（`BepInEx\` 文件夹 + `winhttp.dll` 指向 BepInEx 的 preloader）。BepInEx 5 在 Unity 6 上跑不起来，**直接删掉 `BepInEx\` 文件夹和 `preloader_*.log`**，保留 Doorstop 的那套文件即可。

**3. 面板出现了，但按 F8 弹游戏的「报告问题」**

正常现象：F8 是游戏自带的报告快捷键。面板的显隐键是 **F10**。

**4. 面板打开时鼠标不能转视角**

正常现象：游戏把光标锁在屏幕中心，IMGUI 收不到鼠标事件，面板必须强制解锁光标。要转视角先按 F10 收起面板。

**5. 中文变成方框/乱码**

面板用的是系统字体 `Microsoft YaHei UI`（找不到会退到 `Microsoft YaHei` / `SimHei` / `SimSun`）。如果系统精简掉了这些字体，装一个「微软雅黑」即可。

**6. 联机时刷不出东西**

刷物品 / 改天气走的是服务器授权接口，**只有主机（或单人）生效**，客机点了没反应是正常的。

---

## 五、卸载

三选一：

| 方式 | 操作 |
|---|---|
| 彻底卸载 | 删掉 `winhttp.dll`、`doorstop_config.ini`、`.doorstop_version`、`doorstop\` 四样 |
| 临时关闭 | `doorstop_config.ini` 里 `enabled = true` 改成 `false`（文件留着，下次改回来就行） |
| 只关面板 | 进游戏按 F10 收起，或者 `enabled=false` |

删完游戏就是原版，存档、成就判定都不受影响（面板只是运行时注入，不改游戏文件）。

---

## 六、进阶：自己改面板 / 重新编译

源码在 `_work\panel_src\`：

| 文件 | 作用 |
|---|---|
| `Panel.cs` | 面板全部逻辑（入口、钩子、自绘 IMGUI 控件、刷物品、玩家/车辆/天气/飞行） |
| `ZhNames.cs` | 227 条物品中文名映射（`// BEGIN AUTO ... // END AUTO` 之间是自动生成的） |
| `build.ps1` | 编译脚本（用 `_work\roslyn` 里的 Roslyn `csc.dll`，因为本机只有 .NET 运行时、没有 SDK） |

重新编译：

```powershell
powershell -File "E:\你妈的drive\_work\panel_src\build.ps1"
```

成功后会直接覆盖 `GAME17038\doorstop\LootDebugPanel.dll`，重启游戏生效。

编译时引用的是游戏自己的托管程序集（`Nomad Drive_Data\Managed\` 下的 `Assembly-CSharp.dll`、`UnityEngine.*.dll`、`QFSW.QC.dll`、`NWH.*.dll`、`ECM2.dll`、`Mirror.dll`、`Enviro3.Runtime.dll` 等），所以**游戏更新后如果类型签名变了，需要重新对照改代码**。

---

## 七、文件清单速查

| 文件 | 大小参考 | 必须 |
|---|---|---|
| `LootDebugPanel.dll` | ~64 KB | ✅ |
| `winhttp.dll` | Doorstop 4.6 自带 | ✅ |
| `doorstop_config.ini` | < 1 KB | ✅ |
| `.doorstop_version` | 6 字节（`4.6.0`） | 建议 |
| `doorstop\panel.log` | 运行生成 | ❌ |
| `doorstop\items.tsv` | F4 导出 | ❌ |
| `doorstop\zh.tsv` | F1 导出 | ❌ |
