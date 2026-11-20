---
name: defold-core
description: Make games with Defold — game objects, components and collections, Lua script lifecycle (init/update/fixed_update/on_message/on_input), message passing and URLs, input bindings, factories and collection factories, go.animate, collision objects and physics messages, GUI scripts, and bundling for HTML5/mobile/desktop. Use for any Defold project.
---

# Defold

## Core concepts

- **Game object** (GO): position/rotation/scale + components (sprite, script, collision object, sound, factory, label, tilemap, model).
- **Collection**: a tree of GOs (a level, a player prefab). The bootstrap collection is set in `game.project`.
- Components communicate via **message passing** using URLs: `"socket:/path#fragment"` → e.g. `"main:/player#script"`, `"."` (this GO), `"#sprite"` (component on this GO).

## Script lifecycle

```lua
-- player.script
go.property("speed", 150)            -- editable in the editor, accessible as self.speed

function init(self)
    msg.post(".", "acquire_input_focus")
    self.velocity = vmath.vector3()
    self.input = vmath.vector3()
end

function fixed_update(self, dt)       -- enable Use Fixed Timestep in game.project physics
    local pos = go.get_position()
    self.velocity.x = self.input.x * self.speed
    go.set_position(pos + self.velocity * dt)
    self.input = vmath.vector3()
end

function on_input(self, action_id, action)
    if action_id == hash("left") then self.input.x = -1
    elseif action_id == hash("right") then self.input.x = 1
    elseif action_id == hash("jump") and action.pressed then self:jump() end
end

function on_message(self, message_id, message, sender)
    if message_id == hash("contact_point_response") then
        -- separate from walls: go.set_position(go.get_position() + message.normal * message.distance)
    elseif message_id == hash("trigger_response") and message.enter then
        msg.post(message.other_id, "collected")
    end
end

function final(self)
    msg.post(".", "release_input_focus")
end
```

Pre-hash ids at file top for speed: `local JUMP = hash("jump")`.

## Input bindings

`input/game.input_binding`: map keys/gamepad/touch to actions (`left`, `right`, `jump`, `touch`). `action.pressed`, `action.released`, `action.repeated`, `action.value`, `action.x/y` (touch/mouse).

## Spawning

```lua
local id = factory.create("#bullet_factory", pos, rot, { speed = 400 }, 1)   -- properties override go.property
go.delete(id)

local ids = collectionfactory.create("#enemy_factory", pos)   -- returns table path → id
```

Mark factories *Load Dynamically* for async resource loading.

## Animation

```lua
sprite.play_flipbook("#sprite", hash("run"))
sprite.set_hflip("#sprite", self.velocity.x < 0)
go.animate(".", "position.y", go.PLAYBACK_ONCE_PINGPONG, pos.y + 10, go.EASING_OUTQUAD, 0.3)
go.animate("#sprite", "tint.w", go.PLAYBACK_ONCE_FORWARD, 0, go.EASING_LINEAR, 0.5, 0, function() go.delete() end)
```

## Physics

Collision object types: *Static*, *Dynamic*, *Kinematic* (you move it, receive contact messages to resolve), *Trigger*. Set **group** and **mask** names. Platformer characters typically use Kinematic + manual separation using `contact_point_response` normals/distance. Raycasts: `physics.raycast(from, to, { hash("world") })`.

## GUI

`.gui` + `.gui_script` render in screen space, independent of the game camera. Nodes: box, text, pie, template. `gui.get_node("score")`, `gui.set_text(node, "10")`, `gui.pick_node(node, action.x, action.y)` for buttons, `gui.animate`. Send messages from game scripts: `msg.post("main:/hud#gui", "set_score", { score = 10 })`.

## Camera & rendering

Camera component (+ `acquire_camera_focus` in older versions; recent versions use the camera component's enabled state). Customize `render_script` for pixel-perfect scaling or post-processing. Popular: `rendercam`/`defold-orthographic` extensions.

## Bundling

Project → Bundle → HTML5 / Windows / macOS / Linux / Android / iOS. CLI: `java -jar bob.jar --archive --platform js-web --variant release resolve build bundle`. HTML5 builds are small and fast — well suited to web portals. Native extensions (e.g. Steamworks, Poki/CrazyGames SDKs) are pulled in as dependencies in `game.project`.

## Pitfalls

- Sending messages to a URL that doesn't exist yet (spawned this frame) → error. Store ids from `factory.create`.
- `go.get_position()` of another GO is fine; *setting* another GO's position from outside is discouraged — send a message.
- Forgetting `acquire_input_focus` → no `on_input` calls.
- Max counts (sprites, collision objects, game objects) in `game.project` cap spawning — raise them for bullet-heavy games.
