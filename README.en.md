<div align="center">

<img src="docs/images/logo.png" alt="" width="160">

# PackWright:<br>Agent-Written Minecraft Data Packs That Work

![Minecraft Java 1.21.8](https://img.shields.io/badge/Minecraft_Java-1.21.8-62a83f?style=flat-square)
![Python 3.9+](https://img.shields.io/badge/Python-3.9%2B-3776ab?style=flat-square&logo=python&logoColor=white)
![No dependencies](https://img.shields.io/badge/dependencies-none-4c9a8a?style=flat-square)
![License MIT](https://img.shields.io/badge/license-MIT-5b6ee1?style=flat-square)

[中文](README.md) · English

</div>

## 📖 About

The name joins pack (data pack) and wright (an old word for a maker, as in shipwright): a maker of data packs.

Coding agents often mix syntax from different Minecraft versions when they write commands: item NBT in curly braces, text components written as JSON strings, attributes with the `generic.` prefix. In 1.21.8 these either fail or are silently ignored.

PackWright puts a 1.21.8 knowledge base, the official game data and a static checker into one folder for Codex, Claude Code and other agents. You describe the weapon, boss or mechanic you want in a sentence; the agent looks things up before writing, checks its own work, and hands you a data pack that runs once it is in `datapacks/` and you run `/reload`.

## ✨ Features

- 📚 **Knowledge base**: 28 topic notes, from execute, scoreboards and NBT to skill weapons and boss fights, plus a list of common pitfalls
- 🗃️ **Official 1.21.8 data**: command tree, registries, tags, block states, item components and every vanilla data file, ids searchable by Chinese name too
- 🔍 **Checker**: checks syntax, ids, fields and outdated formats in commands, functions, macros and JSON, and suggests the fix
- 🎮 **Complete examples**: a three-skill weapon and a two-phase boss, ready to play or to learn from
- 🪶 **Lightweight**: Python 3.9+ only, no dependencies, works offline

## 🎮 Examples

### ⚡ Storm Blade

> Make a lightning sword: right click dashes forward and damages mobs in the path, sneak + right click casts a frost nova, hits have a 30% chance to chain lightning, and cooldowns show in the actionbar.

- 🌀 Right click: Thunder Dash, 3 s cooldown
- ❄️ Sneak + right click: Frost Nova, 10 s cooldown
- ⛓️ On hit: 30% chance of chain lightning
- 🔨 Crafting: lightning rod, diamond sword and breeze rod in one column
- 🎁 Get it: `/function storm:give`

### 👑 Storm King

> Make a two-phase boss with a health bar, telegraphed lightning strikes and minions. It enters phase two at half health and drops unique loot and an advancement when defeated.

- ❤️ 400 health with a boss bar
- 🌩️ Marks the ground under each player, then strikes it with lightning
- 💥 A shockwave throws nearby players into the air
- 🌪️ Enters phase two at half health and starts summoning minions
- 🏆 A player kill drops loot and grants an advancement
- 🌤️ The weather returns to what it was before the fight
- 🎁 Summon: `/function king:spawn`

Both examples are in `examples/`. Copy them into your world's `datapacks/` and run `/reload`. `/function storm:uninstall` and `/function king:uninstall` remove them.

## 🔍 Outdated syntax

The checker recognizes these common outdated forms and gives the 1.21.8 form:

- ❌ `diamond_sword{Enchantments:[...]}` → ✅ `diamond_sword[enchantments={sharpness:5}]`
- ❌ `CustomName:'{"text":"Guard"}'` → ✅ `CustomName:{text:"Guard"}`
- ❌ `HandItems:[...]` → ✅ `equipment:{mainhand:{...}}`
- ❌ `Count:1b` → ✅ `count:1`
- ❌ `generic.max_health` → ✅ `max_health`
- ❌ `particle dust 1 0 0 1` → ✅ `particle dust{color:[1.0,0.0,0.0],scale:1.0}`
- ❌ one `execute in <dimension> as @e[...]` line per dimension → ✅ a single `execute as @e[...] at @s`
- ❌ `@s` straight in a `tick` function → ✅ `execute as @a at @s run function ...`

<details>
<summary>A real checker run</summary>

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

## 🚀 Quick start

1. 📦 Download [PackWright.zip](PackWright.zip) and extract it to get the `PackWright` folder
2. 🤖 Give it to your agent
   - Claude Code: put the folder in `~/.claude/skills/` (all projects) or in a project's `.claude/skills/`; it is picked up automatically for Minecraft requests
   - Codex and other agents: put the folder in the working directory and start with "Read PackWright/SKILL.md first"
3. 💬 Describe what you want
4. 🎮 Put the generated data pack in `<world>/datapacks/` and run `/reload` in game

## 💡 Example requests

- 🛒 Make a shop menu opened with /trigger where players click buttons to trade emeralds for items
- ⏱️ Make a parkour timer: stepping on the start pressure plate starts it, and the finish shows the time and the player's personal best
- 📜 Use a dialog for a welcome screen that opens the first time a player joins and lets them pick a class
- 🏹 Make a custom enchantment "Thunderstrike": arrows that hit have a chance to call down lightning, higher levels raise the chance

## 🧭 How it works

<p align="center">💬 Request → 📚 Look up → 🧩 Design → ✍️ Write → 🔍 Check → 📦 Pack</p>

- 📚 Look up: read the matching notes, then confirm command syntax, component fields and vanilla patterns in the official data
- 🧩 Design: settle events, state and the execution context of every function before writing
- 🔍 Check: fix what the checker reports, then pack
- 🎯 For a weapon or a boss, the agent reads the matching pattern note and example pack first

## 🛠️ Checker and lookup tools

The agent runs these itself, and you can use them too. `pw` below is short for `python3 PackWright/tools/pw.py`; on Windows use `python` or `py` instead of `python3`.

- ✅ Check one command: `pw cmd "give @s diamond_sword[enchantments={sharpness:5}]"`
- 📦 Check a data pack: `pw check my_pack/`
- 🧩 Check several packs together: `pw check --merge pack_a.zip pack_b.zip pack_c.zip`
- 🗜️ Check and zip a pack: `pw pack my_pack/`
- 🔎 Look up an id: `pw id "blaze rod"`
- 📚 Search the knowledge base: `pw find cooldown`

Options:

- `--merge`: check the packs of one map as a whole; references to functions, objectives and tags across packs resolve
- `--strict-names`: allow only `A-Za-z0-9_-.+` in objective, team and tag names
- `--json`: machine-readable output

<details>
<summary>What gets checked</summary>

<br>

- Commands are parsed against the 1.21.8 command tree, including ids, selector arguments, NBT and item component fields
- Loot tables, recipes, advancements, predicates, enchantments, dialogs and other JSON files are checked against the official schemas
- Unclosed `$(` in macro lines, and macro lines without a variable
- Calls to functions that do not exist, functions defined twice
- Objectives and teams that are never created, tags that are never added
- `@s`, `execute on` or relative coordinates straight in functions started by `tick`, `load` or `schedule`
- Item NBT in curly braces, text components written as JSON strings, `Count`, `generic.` attributes, `HandItems` / `ArmorItems`, old particle arguments
- Full-width punctuation, ideographic spaces and zero-width characters inside names

</details>

## 📁 Folder layout

- 📄 `SKILL.md`: entry point for agents; `AGENTS.md` and `CLAUDE.md` point to it
- 📚 `knowledge/`: knowledge base
- 🗃️ `reference/`: official 1.21.8 data
- 🛠️ `tools/pw.py`: checker and lookup tools
- 🎮 `examples/`: Storm Blade and Storm King
- 🧪 `tests/run.py`: regression tests, run `python3 PackWright/tests/run.py`

<details>
<summary>Knowledge base topics</summary>

<br>

- `commands/`: execute, selectors, scoreboards, NBT and storage, text components, item commands and item predicates, world and entity commands
- `datapack/`: pack structure, functions and macros, advancements, predicates, recipes and registries, custom enchantments, dialogs
- `entities/`: entity NBT, display / interaction / marker entities, motion and projectiles
- `items/`: item components, custom items, item modifiers and loot tables
- `patterns/`: pack architecture, skill weapons, boss fights, raycasts, timers, cooldowns and feedback, math
- top level: `INDEX.md` index, `pitfalls.md` common pitfalls, `version-1.21.8.md` version differences

</details>

## ⚖️ License

- [MIT](LICENSE)
- Schemas are compiled from [SpyglassMC/vanilla-mcdoc](https://github.com/SpyglassMC/vanilla-mcdoc)
- Game data is generated from Minecraft 1.21.8 by [misode/mcmeta](https://github.com/misode/mcmeta); see [NOTICE](NOTICE)
- Not affiliated with Mojang or Microsoft
