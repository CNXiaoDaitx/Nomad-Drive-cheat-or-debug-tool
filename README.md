[README.md](https://github.com/user-attachments/files/35204438/README.md)
# Nomad Drive 调试面板

一个给 **Nomad Drive**（Unity 单人游戏，Early Access 1.0）用的运行时调试面板：悬浮在游戏画面上，能刷物品、改人物状态、调车辆参数/油量/冷却液、切天气、开飞行、无限使用物品。

不需要 BepInEx / MelonLoader，不需要 .NET SDK，不改游戏本体文件 —— 拷 4 个文件进游戏目录就能用，删掉就干净卸载。

![物品生成面板](images/panel-player.png)

---

## 功能一览

| 模块 | 能做什么 |
|---|---|
| **物品生成** | 227 个物品的中文名列表（括号内是游戏里的英文称呼），支持搜索 / 分类筛选，点一下就在准星前方生成，位置会自动贴地 |
| **玩家状态** | 血量、饱食度、水分、精力、中毒值实时滑条 + 0/半/满 快捷键、解毒、全部回满、原地复活、有效血量/受伤显示 |
| **车辆** | 自动锁定离你最近的车；车速/里程/位置、油量与油箱容量、引擎启停与状况、手刹、急停、冷却液量与容量、换水/换冷却液 |
| **天气** | 显示当前天气（Enviro）与温度/湿度/风/日照，一键锁定 15 种天气，支持"立即/渐变"，可恢复自动 |
| **飞行模式** | 人物飞行、车辆飞行各有开关和速度滑条，WASD 平移 / 空格上升 / Ctrl 下降 / Shift 加速 |
| **无限使用物品** | 开关式：手持液体自动补满（喝水/倒油不掉）、喷漆罐无限、食物吃掉自动补一个 |
| **窗口** | 可拖动、右下角拖拽缩放、透明度滑条、最小化成小条、隐藏后按 F10 或点左下角按钮叫回 |

<p align="center">
  <img src="images/panel-vehicle.png" width="49%" />
  <img src="images/panel-weather.png" width="49%" />
</p>

<p align="center">
  <img src="images/spawn-demo.png" width="49%" />
  <img src="images/flight-player.png" width="49%" />
</p>

---

## 安装

> 只适用于 **Nomad Drive V1.0**，游戏根目录指 `Nomad Drive.exe` 所在文件夹。

1. **备份**（可选）：把游戏目录复制一份，或备份存档 `%USERPROFILE%\AppData\LocalLow\Indie Devil\Nomad Drive\`。
2. **拷贝 4 个文件到游戏根目录**：

   ```
   GAME17038\
   ├─ Nomad Drive.exe
   ├─ Nomad Drive_Data\
   ├─ winhttp.dll              ← 新增（Doorstop 代理）
   ├─ doorstop_config.ini      ← 新增
   ├─ .doorstop_version        ← 新增（4.6.0）
   └─ doorstop\                ← 新增
      └─ LootDebugPanel.dll    ← 新增（面板本体）
   ```

3. **检查 `doorstop_config.ini`**（用记事本打开，确保这几行）：

   ```ini
   [General]
   enabled = true
   target_assembly=doorstop\LootDebugPanel.dll
   ```

4. **启动游戏**，进主菜单后按 **F10** —— 面板出现即安装成功。

   验证：`doorstop\panel.log` 里应有 `=== Doorstop entry called ===` 和 `panel GameObject created`。

> 如果仓库的 **Releases** 里提供了打包好的压缩包，直接解压到游戏根目录覆盖即可，效果和上面的手动拷贝一样。

更详细（常见问题、卸载、重编译）见 👉 [安装教程.md](安装教程.md)

---

## 使用

| 按键 | 作用 |
|---|---|
| **F10** | 显示 / 隐藏面板（注意：F8 是游戏自带的"报告问题"，别按错） |
| **F5** | 一键开始单人游戏（自动点掉弹窗 → 单人游戏 → 创建） |
| **F2** | 自检：打印物品表绑定状态与人物五项数值 |
| **F7** | 车辆 / 散热器诊断，结果写进日志区 |
| **F12** | 把当前所有控件的坐标和 id 转储到日志（改 UI 时很有用） |

面板分三页：**玩家 / 车辆 / 天气**；左列永远是物品列表，想刷什么就搜什么再点一下。

完整说明（每一项参数的含义、已知限制）见 👉 [物品调试面板-使用说明.md](物品调试面板-使用说明.md)
物品中文名对照表见 👉 [物品清单.md](物品清单.md)

---

## 二次开发

**欢迎二次开发**。仓库里的 [使用说明](物品调试面板-使用说明.md)、[安装教程](安装教程.md)、[物品清单](物品清单.md) 就是按"交接文档"写的：任何人拿到这个仓库，都可以照着这些文档接着改，不需要再问我。

### 代码结构

```
_work/panel_src/
├─ Panel.cs       # 全部逻辑：Doorstop 入口、钩子、自绘 IMGUI 控件、
│                 # 刷物品、玩家/车辆/天气/飞行/无限物品
├─ ZhNames.cs     # 227 条物品中文名映射（BEGIN/END AUTO 之间自动生成）
└─ build.ps1      # 编译脚本（Roslyn csc，直接引用游戏托管程序集）
```

### 编译

本机只需要 .NET **运行时**（不需要 SDK）：

```powershell
powershell -File "路径\_work\panel_src\build.ps1"
```

脚本会用 `_work\roslyn` 里的 `csc.dll` 编译，并直接引用游戏的 `Nomad Drive_Data\Managed\` 下的程序集（`Assembly-CSharp`、`UnityEngine.*`、`QFSW.QC`、`NWH.*`、`ECM2`、`Mirror`、`Enviro3.Runtime` 等），产物覆盖 `doorstop\LootDebugPanel.dll`，重启游戏生效。

### 几个踩过的坑（省你时间）

- **别用 BepInEx 5**：Unity 6 的 mscorlib 没有 `System.Reflection.Module.GetPEKind`，BepInEx 5 的 preloader 会直接 `MissingMethodException`。这里用的是 Doorstop 4.6 + 自写 `Doorstop.Entrypoint.Start()` 入口。
- **游戏内 `GUI.Button` 永远不触发**：游戏每帧会重置 `GUIUtility.hotControl`，标准 IMGUI 控件收不到按下/抬起配对。所以面板所有控件都是**自绘 + 自己判定鼠标矩形**（`Btn` / `Slider` / `TextField`）。
- **`GUI.Window` 的标题栏会吞鼠标事件**：按钮别放在标题栏那 24px 里，放到下面一条工具栏；拖动/缩放也改成用全局鼠标状态（`Input.GetMouseButton` + 屏幕坐标），因为 IMGUI 只在鼠标位于窗口内时才派发事件。
- **车辆飞行不能直接写 `rb.velocity`**：NWH 的 `VehicleController` 每帧会覆盖；要关掉重力 + 临时禁用控制器 + 用 `Rigidbody.MovePosition` 位移。
- **刷物品走 `WorldGenerator.CmdDebugSpawnLootByGuid(guid, pos)`** 比游戏自带的 `world.spawnLoot` 更可控（后者固定打到 50 米外）。

### 还可以做什么

- 载入时自动打开面板、记住窗口位置/大小
- 物品搜索支持中文（目前只能英文名 / 内部 key）
- 批量生成、按分类一键生成一箱
- 存档编辑 / 时间与昼夜控制

---

## 版本兼容

> ⚠️ **目前版本只适用于 Nomad Drive V1.0**（Unity `6000.0.73f1`，Mono 后端）。

面板是直接引用游戏托管程序集编译的，游戏一旦更新，`Assembly-CSharp` 里的类型/方法签名可能变化，需要重新对照源码修改并重编译。不同版本的游戏请勿直接套用本 DLL。

---

## 注意事项

- 刷物品、改天气走的是服务器授权接口，**只有单人模式或作为主机时生效**；进别人的房间点了没反应是正常的。
- 面板会修改人物数值、车辆状态等运行时数据，**可能影响成就/进度判定**，请自行备份存档，介意的话先复制一份存档。
- 仅供单机调试与学习使用，请勿用于联机游戏或商业用途。

---

## 许可证

MIT License —— 可以自由使用、修改、分发；如需改成别的协议，直接改这一段即可。

## 致谢

- [Unity Doorstop](https://github.com/Neighlyd/doorstop) —— 注入 loader
- **Quantum Console**（QFSW）—— 游戏内置控制台，本项目的命令调用依赖它
- 游戏本体 © Indie Devil
