---
name: love2d-core
description: Make 2D games with LÖVE (love2d) 11.x in Lua — conf.lua, love.load/update/draw callbacks, input callbacks, images/quads/SpriteBatch, pixel-perfect scaling with canvases, love.physics (Box2D), audio sources, game states, require modules, and distribution as .love / fused executables / web. Use for any LÖVE project.
---

# LÖVE 11.x

## Files

```lua
-- conf.lua
function love.conf(t)
  t.identity = "mygame"              -- save directory name
  t.version = "11.5"
  t.window.title = "My Game"
  t.window.width, t.window.height = 1280, 720
  t.window.resizable = true
  t.window.vsync = 1
end
```

```lua
-- main.lua
local Player = require("src.player")
local player
local W, H = 320, 180
local canvas

function love.load()
  love.graphics.setDefaultFilter("nearest", "nearest")   -- pixel art (before loading images)
  canvas = love.graphics.newCanvas(W, H)
  player = Player.new(100, 100)
end

function love.update(dt)
  dt = math.min(dt, 1/30)          -- avoid huge steps after hitches
  player:update(dt)
end

function love.draw()
  love.graphics.setCanvas(canvas)
  love.graphics.clear(0.11, 0.11, 0.17)
  player:draw()
  love.graphics.setCanvas()
  local s = math.floor(math.min(love.graphics.getWidth() / W, love.graphics.getHeight() / H))
  local ox = (love.graphics.getWidth() - W * s) / 2
  local oy = (love.graphics.getHeight() - H * s) / 2
  love.graphics.draw(canvas, ox, oy, 0, s, s)       -- integer scale, letterboxed
end

function love.keypressed(key)
  if key == "escape" then love.event.quit() end
  if key == "space" then player:jump() end
end
```

Colors are **0–1** in LÖVE 11 (not 0–255).

## Modules / classes

```lua
-- src/player.lua
local Player = {}
Player.__index = Player

function Player.new(x, y)
  local self = setmetatable({}, Player)
  self.x, self.y, self.vx, self.vy = x, y, 0, 0
  self.image = love.graphics.newImage("assets/hero.png")
  return self
end

function Player:update(dt)
  local dx = 0
  if love.keyboard.isDown("a", "left") then dx = dx - 1 end
  if love.keyboard.isDown("d", "right") then dx = dx + 1 end
  self.x = self.x + dx * 90 * dt
end

function Player:draw()
  love.graphics.draw(self.image, math.floor(self.x), math.floor(self.y))
end

return Player
```

Libraries commonly used: `classic` (OOP), `bump.lua` (AABB collision for platformers), `anim8` (sprite animation), `hump` (gamestate, timer, camera), `sti` (Tiled maps), `windfield`/`love.physics` (Box2D).

## Sprites

```lua
local sheet = love.graphics.newImage("assets/sheet.png")
local quads = {}
for i = 0, 5 do quads[#quads+1] = love.graphics.newQuad(i*16, 0, 16, 16, sheet:getDimensions()) end
love.graphics.draw(sheet, quads[frame], x, y, rot, sx, sy, 8, 8)   -- ox/oy = origin

local batch = love.graphics.newSpriteBatch(sheet, 1000)   -- tilemaps: one draw call
```

## Physics (Box2D)

```lua
love.physics.setMeter(32)
world = love.physics.newWorld(0, 9.81 * 32, true)
local body = love.physics.newBody(world, x, y, "dynamic")
local shape = love.physics.newRectangleShape(16, 24)
local fixture = love.physics.newFixture(body, shape)
body:setFixedRotation(true)
world:setCallbacks(beginContact, endContact)
-- update: world:update(dt)
```

For tight platformer feel, prefer `bump.lua` with custom movement over Box2D.

## Audio

```lua
local music = love.audio.newSource("assets/music.ogg", "stream")
music:setLooping(true); music:play()
local sfx = love.audio.newSource("assets/jump.wav", "static")
sfx:clone():play()         -- overlapping plays
```

## Saving

`love.filesystem.write("save.lua", serialized)`, `love.filesystem.read`; files go to the identity save dir. Serialize with a JSON lib (`rxi/json.lua`) or `binser`. Never `loadstring` untrusted saves.

## Distribution

- `.love`: zip the project contents (main.lua at zip root) → rename `game.love`.
- Windows fused: `copy /b love.exe+game.love game.exe` and ship with LÖVE DLLs + license.
- macOS: put `game.love` inside `love.app/Contents/Resources/` and edit Info.plist.
- Web: `love.js` (npm `love.js`) — compatibility mode for no-threads hosting.
- Tooling: `makelove`, `love-release`.

## Pitfalls

- Creating images/fonts in `love.draw` every frame → memory leak.
- Non-integer positions with nearest filtering cause shimmering — floor positions or draw to a canvas.
- `love.keypressed` for discrete actions; `isDown` for held actions.
