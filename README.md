# Your GitHub Learning Lab Repository for Introducing GitHub

Welcome to **your** repository for your GitHub Learning Lab course. This repository will be used during the different activities that I will be guiding you through. See a word you don't understand? We've included an emoji 📖 next to some key terms. Click on it to see its definition.

Oh! I haven't introduced myself...

I'm the GitHub Learning Lab bot and I'm here to help guide you in your journey to learn and master the various topics covered in this course. I will be using Issue and Pull Request comments to communicate with you. In fact, I already added an issue for you to check out.

![issue tab](https://lab.github.com/public/images/issue_tab.png)

I'll meet you over there, can't wait to get started!

This course is using the :sparkles: open source project [reveal.js](https://github.com/hakimel/reveal.js/). In some cases we’ve made changes to the history so it would behave during class, so head to the original project repo to learn more about the cool people behind this project.

## Game-dev Agent Skills (Claude Code plugin marketplace)

This repository also hosts **awesome-gamedev-agent-skills**, a Claude Code plugin marketplace with a master router and 79 cross-engine game-development skills.

```
.claude-plugin/marketplace.json   # marketplace manifest (10 plugins)
router/SKILL.md                   # master router: detects engine + task, loads the right skills
skills/
  godot/          16 skills  (Godot 4: GDScript, C#, scenes, signals, 2D/3D, physics, UI, shaders, multiplayer, export, headless tests)
  unity/           9 skills  (Unity 6: C#, Input System, physics, animation, ScriptableObjects, tilemaps, NavMesh, builds)
  unreal/          6 skills  (UE5: Blueprints, C++, Enhanced Input, Behavior Trees/EQS/StateTree, Niagara, packaging)
  ebitengine/      1 skill   (Go 2D games)
  web-engines/     6 skills  (Phaser, PixiJS, three.js)
  other-engines/  12 skills  (Bevy, pygame-ce, LÖVE, Defold, Roblox ×8)
  disciplines/    15 skills  (assets, AI, procgen, dialogue, saves, audio, shaders, physics, levels, input, feel, UI/UX, cameras, performance, BT/utility AI)
  genres/          9 skills  (platformer, roguelike, RPG, FPS, tower defense, card game, visual novel, survival, puzzle)
  workflows/       5 skills  (game jam, fast prototyping, Steam, itch.io, game docs)
```

Install in Claude Code (2.1.275+) — `gamedev` installs everything, or pick a single group:

```
/plugin install gamedev --marketplace Loup08/github-slideshow
/plugin install router --marketplace Loup08/github-slideshow
/plugin install godot --marketplace Loup08/github-slideshow
/plugin install unity --marketplace Loup08/github-slideshow
/plugin install unreal --marketplace Loup08/github-slideshow
/plugin install ebitengine --marketplace Loup08/github-slideshow
/plugin install web-engines --marketplace Loup08/github-slideshow
/plugin install other-engines --marketplace Loup08/github-slideshow
/plugin install disciplines --marketplace Loup08/github-slideshow
/plugin install genres --marketplace Loup08/github-slideshow
/plugin install workflows --marketplace Loup08/github-slideshow
```

On older Claude Code versions, add the marketplace first with `/plugin marketplace add Loup08/github-slideshow`, then run `/plugin install <name>@awesome-gamedev-agent-skills`.
