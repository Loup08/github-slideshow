---
name: bevy-ecs
description: Build games in Rust with Bevy's ECS — App/plugins, components (required components), resources, systems and queries with filters, Commands, events/messages and observers, States and schedules, fixed timestep, assets, and compile-time tips. Use for any Bevy project; check the version's migration guide because Bevy APIs change every release.
---

# Bevy ECS

Bevy changes APIs each minor release. Read `Cargo.toml` for the version and consult the official migration guide (bevyengine.org/learn/migration-guides) for anything that doesn't compile. Examples below follow the modern (0.15+) style: required components, `Sprite`/`Camera2d` as components, `delta_secs()`.

## Setup

```toml
[dependencies]
bevy = "0.x"   # match the project

[profile.dev]
opt-level = 1
[profile.dev.package."*"]
opt-level = 3            # fast deps in debug builds — essential
```

Enable `bevy/dynamic_linking` feature during development for faster incremental builds (not for release).

## App & plugins

```rust
use bevy::prelude::*;

fn main() {
    App::new()
        .add_plugins(DefaultPlugins.set(ImagePlugin::default_nearest())) // pixel art
        .add_plugins(PlayerPlugin)
        .run();
}

pub struct PlayerPlugin;
impl Plugin for PlayerPlugin {
    fn build(&self, app: &mut App) {
        app.add_systems(Startup, spawn_player)
           .add_systems(Update, (read_input, move_player).chain());
    }
}
```

One plugin per feature (player, enemies, ui, audio).

## Components, resources, spawning

```rust
#[derive(Component)]
#[require(Transform, Velocity)]          // required components auto-inserted
struct Player { speed: f32 }

#[derive(Component, Default)]
struct Velocity(Vec2);

#[derive(Resource, Default)]
struct Score(u32);

fn spawn_player(mut commands: Commands, assets: Res<AssetServer>) {
    commands.spawn(Camera2d);
    commands.spawn((
        Player { speed: 200.0 },
        Sprite::from_image(assets.load("hero.png")),
        Transform::from_xyz(0.0, 0.0, 1.0),
    ));
}
```

## Systems & queries

```rust
fn read_input(keys: Res<ButtonInput<KeyCode>>, mut q: Query<(&Player, &mut Velocity)>) {
    let mut dir = Vec2::ZERO;
    if keys.pressed(KeyCode::KeyA) { dir.x -= 1.0; }
    if keys.pressed(KeyCode::KeyD) { dir.x += 1.0; }
    for (player, mut vel) in &mut q {
        vel.0 = dir.normalize_or_zero() * player.speed;
    }
}

fn move_player(time: Res<Time>, mut q: Query<(&Velocity, &mut Transform)>) {
    for (vel, mut tf) in &mut q {
        tf.translation += vel.0.extend(0.0) * time.delta_secs();
    }
}
```

- Filters: `With<T>`, `Without<T>`, `Added<T>`, `Changed<T>`, `Or<(..)>`.
- Two queries touching the same component mutably conflict → add `Without<>` filters or use a `ParamSet`.
- Single entity: `q.single()` / `single_mut()` returns a `Result` in recent versions — handle the error or use `let Ok(x) = q.single_mut() else { return };`.
- Ordering: `.chain()`, `.before(sys)`, `.after(sys)`, or `SystemSet`s. Unordered systems run in parallel.
- Run conditions: `.run_if(in_state(GameState::Playing))`, `.run_if(resource_exists::<T>)`.

## Communication

- **Buffered events/messages** for many-to-many decoupling. In older versions: `#[derive(Event)]`, `EventWriter<T>`/`EventReader<T>`, `app.add_event::<T>()`. From 0.17 the buffered kind is called **Message** (`#[derive(Message)]`, `MessageWriter`, `MessageReader`, `add_message`), and `Event` is used with **observers**.
- **Observers**: `commands.entity(e).observe(|trigger: On<Damage>, ...| { ... })` (older: `Trigger<Damage>`) — immediate reactions targeted at entities; also component hooks (`on_add`, `on_remove`).

## States

```rust
#[derive(States, Default, Debug, Clone, PartialEq, Eq, Hash)]
enum GameState { #[default] Menu, Playing, GameOver }

app.init_state::<GameState>()
   .add_systems(OnEnter(GameState::Playing), spawn_level)
   .add_systems(Update, gameplay.run_if(in_state(GameState::Playing)));
// change: next.set(GameState::GameOver) with ResMut<NextState<GameState>>
```

Tag level entities with `StateScoped(GameState::Playing)` / `DespawnOnExit` (name depends on version) to auto-despawn on exit.

## Fixed timestep

Put physics/gameplay simulation in `FixedUpdate` (default 64 Hz; `Time<Fixed>`), visuals in `Update`. Use `Time` inside FixedUpdate — it reports the fixed delta.

## Ecosystem

Physics: Avian (ECS-native) or bevy_rapier. Input mapping: leafwing-input-manager. Tilemaps: bevy_ecs_tilemap, bevy_ecs_ldtk. Inspector: bevy-inspector-egui. Check crate compatibility tables against your Bevy version.

## Testing

```rust
#[test]
fn moves_player() {
    let mut app = App::new();
    app.add_plugins(MinimalPlugins).add_systems(Update, move_player);
    let e = app.world_mut().spawn((Velocity(Vec2::X), Transform::default())).id();
    app.update();
    assert!(app.world().get::<Transform>(e).unwrap().translation.x >= 0.0);
}
```

## Pitfalls

- `Commands` apply at sync points — a spawned entity isn't queryable in the same system.
- Huge compile times: use dynamic linking + a fast linker (mold/lld) in dev; split crates.
- Despawning parents: `despawn()` now despawns children recursively in recent versions (older: `despawn_recursive`).
