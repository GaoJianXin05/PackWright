<div align="center">

<img src="docs/images/logo.png" alt="" width="160">

# PackWright：<br>让编程 Agent 写出能直接运行的 Minecraft 数据包

![Minecraft Java 1.21.8](https://img.shields.io/badge/Minecraft_Java-1.21.8-62a83f?style=flat-square)
![Python 3.9+](https://img.shields.io/badge/Python-3.9%2B-3776ab?style=flat-square&logo=python&logoColor=white)
![No dependencies](https://img.shields.io/badge/dependencies-none-4c9a8a?style=flat-square)
![License MIT](https://img.shields.io/badge/license-MIT-5b6ee1?style=flat-square)

中文 · [English](README.en.md)

</div>

## 📖 简介

PackWright 取自 pack（数据包）和 wright（工匠），意思是造数据包的工匠。

编程 Agent 写 Minecraft 命令时，经常把不同版本的写法混在一起，比如花括号里的物品 NBT、写成 JSON 字符串的文本组件、带 `generic.` 前缀的属性。这些写法放进 1.21.8，不是报错就是被游戏静默忽略。

PackWright 把 1.21.8 的知识库、官方游戏数据和一个静态检查器放进同一个文件夹，交给 Codex、Claude Code 等 Agent 使用。你用一句话描述想要的武器、Boss 或玩法，Agent 先查资料再动手，写完用检查器自查，交给你的数据包放进 `datapacks/`、执行 `/reload` 就能玩。

## ✨ 特性

- 📚 **知识库**：28 篇主题笔记，从 execute、记分板、NBT 到技能武器、Boss 战，还有一份常见的坑
- 🗃️ **1.21.8 官方数据**：命令树、注册表、标签、方块状态、物品组件和全部原版数据文件，ID 可以用中文名搜索
- 🔍 **检查器**：检查命令、函数、宏和 JSON 里的语法、ID、字段和旧写法，并给出改法
- 🎮 **完整示例**：一把三技能武器和一个两阶段 Boss，可以直接玩，也可以照着写
- 🪶 **轻量离线**：只需要 Python 3.9+，不用装依赖，不用联网

## 🎮 示例

### ⚡ 风暴之刃

> 做一把雷电剑：右键向前冲刺并伤害路径上的怪，潜行右键放冰霜新星，攻击时 30% 概率触发连锁闪电，冷却显示在动作栏。

- 🌀 右键：雷霆冲刺，冷却 3 秒
- ❄️ 潜行右键：冰霜新星，冷却 10 秒
- ⛓️ 命中时：30% 概率触发连锁闪电
- 🔨 合成：避雷针、钻石剑、旋风棒竖排
- 🎁 获取：`/function storm:give`

### 👑 风暴之王

> 做一个两阶段 Boss：有血条，会预警后落雷、召唤小怪，半血进入第二阶段，打败后掉落专属战利品并解锁进度。

- ❤️ 400 点生命，带 Boss 血条
- 🌩️ 先在玩家脚下标出预警，再落雷
- 💥 冲击波把附近的玩家震到空中
- 🌪️ 半血进入第二阶段，开始召唤小怪
- 🏆 玩家击杀后掉落战利品，解锁进度
- 🌤️ 战斗结束后天气恢复原样
- 🎁 召唤：`/function king:spawn`

两个示例都在 `examples/` 里，复制到存档的 `datapacks/` 后执行 `/reload` 即可。卸载用 `/function storm:uninstall` 和 `/function king:uninstall`。

## 🔍 旧写法检查

检查器能认出这些常见的旧写法，并给出 1.21.8 的写法：

- ❌ `diamond_sword{Enchantments:[...]}` → ✅ `diamond_sword[enchantments={sharpness:5}]`
- ❌ `CustomName:'{"text":"Guard"}'` → ✅ `CustomName:{text:"Guard"}`
- ❌ `HandItems:[...]` → ✅ `equipment:{mainhand:{...}}`
- ❌ `Count:1b` → ✅ `count:1`
- ❌ `generic.max_health` → ✅ `max_health`
- ❌ `particle dust 1 0 0 1` → ✅ `particle dust{color:[1.0,0.0,0.0],scale:1.0}`
- ❌ 每个维度写一行 `execute in <维度> as @e[...]` → ✅ 一行 `execute as @e[...] at @s`
- ❌ `tick` 函数里直接用 `@s` → ✅ `execute as @a at @s run function ...`

<details>
<summary>一次真实的检查输出</summary>

<br>

```mcfunction
give @s diamond_sword{Enchantments:[{id:"sharpness",lvl:5}]} 1
summon zombie ~ ~ ~ {CustomName:'{"text":"Guard"}',HandItems:[{id:"iron_axe",Count:1b}]}
attribute @s generic.max_health base set 40
execute in minecraft:the_nether as @e[type=blaze] run effect give @s glowing 5
```

```console
$ python3 PackWright/tools/pw.py check reward.mcfunction
ERROR reward.mcfunction:1:22 [LEGACY_ITEM_NBT] item NBT in curly braces was removed in 1.20.5
      | give @s diamond_sword{Enchantments:[{id:"sharpness",lvl:5}]} 1
      |                      ^
      hint: use components in square brackets, e.g. diamond_sword[custom_name='x']
ERROR reward.mcfunction:2:21 [ENTITY_NBT] zombie data HandItems: 'HandItems' is old syntax and is silently ignored in 1.21.8
      | summon zombie ~ ~ ~ {CustomName:'{"text":"Guard"}',HandItems:[{id:"iron_axe",Count:1b}]}
      |                     ^
      hint: use equipment:{mainhand:{...},offhand:{...}} (1.21.5+)
WARN  reward.mcfunction:2:21 [TEXT_JSON_STRING] zombie data CustomName: '{"text":"Guard"}' is a plain string, so the game shows it as literal text including the braces
      | summon zombie ~ ~ ~ {CustomName:'{"text":"Guard"}',HandItems:[{id:"iron_axe",Count:1b}]}
      |                     ^
      hint: since 1.21.5 write the component itself, without the outer quotes, e.g. {text:'Name',color:'gold'}
ERROR reward.mcfunction:3:14 [UNKNOWN_ID] unknown attribute 'minecraft:generic.max_health'
      | attribute @s generic.max_health base set 40
      |              ^
      hint: attribute ids lost their 'generic.' prefix in 1.21.2: use minecraft:max_health
WARN  reward.mcfunction:4:36 [EXECUTE_IN_SELECTOR] 'execute in' does not limit @e: a selector without distance/x/y/z/dx/dy/dz still loops over entities of every dimension
      | execute in minecraft:the_nether as @e[type=blaze] run effect give @s glowing 5
      |                                    ^
      hint: writing one such line per dimension runs everything once per dimension; use a single 'execute as @e[...] at @s run ...' (at @s moves into the entity's own dimension) or limit the selector with distance/x/y/z/dx/dy/dz (distance=0.. keeps the search in the source's dimension)
ERRORS: 3 errors, 2 warnings, 0 notes (4 commands) [reward.mcfunction]
```

</details>

## 🚀 快速开始

1. 📦 下载 [PackWright.zip](PackWright.zip) 并解压，得到 `PackWright` 文件夹
2. 🤖 交给 Agent
   - Claude Code：放进 `~/.claude/skills/`（所有项目可用）或项目的 `.claude/skills/`，遇到 Minecraft 需求时自动启用
   - Codex 和其他 Agent：放进工作目录，开头告诉它「先阅读 PackWright/SKILL.md」
3. 💬 用自然语言描述需求
4. 🎮 把生成的数据包放进 `存档/datapacks/`，进游戏执行 `/reload`

## 💡 可以这样提需求

- 🛒 做一个用 /trigger 打开的商店菜单，玩家点按钮用绿宝石换物品
- ⏱️ 做一个跑酷计时器：踩下起点的压力板开始计时，到终点显示用时和个人最好成绩
- 📜 用对话框做新手引导，玩家第一次进入世界时弹出，可以选择职业
- 🏹 做一个自定义附魔「雷击」：弓箭命中时有概率召唤闪电，等级越高概率越大

## 🧭 工作方式

<p align="center">💬 需求 → 📚 查资料 → 🧩 设计 → ✍️ 写数据包 → 🔍 检查 → 📦 打包交付</p>

- 📚 查资料：读知识库里对应的笔记，用官方数据确认命令语法、组件字段和原版写法
- 🧩 设计：先定好事件、状态和每个函数的执行上下文，再动手写
- 🔍 检查：没通过就回去改，通过后再打包
- 🎯 武器、Boss 这类需求，会先读对应的模式笔记和示例包

## 🛠️ 检查器和查询工具

Agent 会自己调用这些命令，你也可以直接用。下面的 `pw` 是 `python3 PackWright/tools/pw.py` 的简写，Windows 上把 `python3` 换成 `python` 或 `py`。

- ✅ 检查一条命令：`pw cmd "give @s diamond_sword[enchantments={sharpness:5}]"`
- 📦 检查一个数据包：`pw check 我的数据包/`
- 🧩 多个包合在一起检查：`pw check --merge 包A.zip 包B.zip 包C.zip`
- 🗜️ 检查并打包成 zip：`pw pack 我的数据包/`
- 🔎 按中文名查 ID：`pw id 烈焰棒`
- 📚 搜索知识库：`pw find 冷却`

常用选项：

- `--merge`：同一张地图的多个包按一个整体检查，包之间的函数、计分项和 tag 引用互相解析
- `--strict-names`：计分项、队伍和 tag 的名字只允许 `A-Za-z0-9_-.+`
- `--json`：输出 JSON

<details>
<summary>检查范围</summary>

<br>

- 命令按 1.21.8 的命令树解析，检查 ID、选择器参数、NBT 和物品组件字段
- 战利品表、配方、进度、谓词、附魔、对话框等 JSON 文件按官方结构定义检查
- 宏行里没有闭合的 `$(` 和没有用到变量的宏行
- 调用了不存在的函数，重复定义的函数
- 没有创建的计分项和队伍，没有添加的 tag
- `tick`、`load`、`schedule` 启动的函数里直接使用 `@s`、`execute on` 或相对坐标
- 花括号物品 NBT、JSON 字符串形式的文本组件、`Count`、`generic.` 属性、`HandItems` / `ArmorItems`、旧粒子参数
- 名字里的全角标点、全角空格和零宽字符

</details>

## 📁 目录结构

- 📄 `SKILL.md`：Agent 的入口说明，`AGENTS.md` 和 `CLAUDE.md` 指向它
- 📚 `knowledge/`：知识库
- 🗃️ `reference/`：1.21.8 官方数据
- 🛠️ `tools/pw.py`：检查器和查询工具
- 🎮 `examples/`：风暴之刃和风暴之王
- 🧪 `tests/run.py`：回归测试，运行 `python3 PackWright/tests/run.py`

<details>
<summary>知识库目录</summary>

<br>

- `commands/`：execute、选择器、记分板、NBT 与 storage、文本组件、物品命令与物品谓词、世界与实体命令
- `datapack/`：数据包结构、函数与宏、进度、谓词、配方与注册表、自定义附魔、对话框
- `entities/`：实体 NBT、展示实体 / 交互实体 / 标记、运动与弹射物
- `items/`：物品组件、自定义物品、物品修饰器与战利品表
- `patterns/`：数据包架构、技能武器、Boss 战、射线检测、计时冷却与提示、数学
- 根目录：`INDEX.md` 索引、`pitfalls.md` 常见的坑、`version-1.21.8.md` 版本差异

</details>

## ⚖️ 许可

- [MIT](LICENSE)
- 结构定义编译自 [SpyglassMC/vanilla-mcdoc](https://github.com/SpyglassMC/vanilla-mcdoc)
- 游戏数据由 [misode/mcmeta](https://github.com/misode/mcmeta) 从 Minecraft 1.21.8 生成，详见 [NOTICE](NOTICE)
- 本项目与 Mojang、Microsoft 无关
