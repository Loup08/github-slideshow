---
name: godot-multiplayer
description: Online multiplayer in Godot 4 with the high-level multiplayer API — ENet/WebSocket/WebRTC peers, authority, @rpc annotations, MultiplayerSpawner and MultiplayerSynchronizer, server-authoritative design, and lag handling. Use for co-op, versus, lobbies, and dedicated servers.
---

# Godot Multiplayer

## Topology

- **Listen server** (one player hosts) — simplest for co-op.
- **Dedicated server** — export with `--headless`, competitive or persistent games.
- Default to **server-authoritative**: clients send input; the server simulates and replicates.

## Peer setup

```gdscript
# autoload Network
const PORT := 7777
signal player_joined(id: int)
signal player_left(id: int)

func host(max_clients := 8) -> Error:
	var peer := ENetMultiplayerPeer.new()
	var err := peer.create_server(PORT, max_clients)
	if err: return err
	multiplayer.multiplayer_peer = peer
	multiplayer.peer_connected.connect(func(id): player_joined.emit(id))
	multiplayer.peer_disconnected.connect(func(id): player_left.emit(id))
	return OK

func join(address: String) -> Error:
	var peer := ENetMultiplayerPeer.new()
	var err := peer.create_client(address, PORT)
	if err: return err
	multiplayer.multiplayer_peer = peer
	multiplayer.connected_to_server.connect(func(): print("connected"))
	multiplayer.connection_failed.connect(func(): print("failed"))
	return OK
```

Server ID is always `1`. `multiplayer.get_unique_id()`, `multiplayer.is_server()`. Web builds: `WebSocketMultiplayerPeer` or `WebRTCMultiplayerPeer` (ENet is UDP, not available in browsers).

## RPCs

```gdscript
@rpc("any_peer", "call_local", "reliable")
func request_fire(dir: Vector2) -> void:
	if not multiplayer.is_server(): return
	var sender := multiplayer.get_remote_sender_id()
	# validate sender owns this player, rate-limit, then act
	_spawn_bullet.rpc(dir)

@rpc("authority", "call_local", "unreliable_ordered")
func _spawn_bullet(dir: Vector2) -> void: ...
```

Call with `request_fire.rpc_id(1, dir)` (to server) or `func.rpc(args)` (everyone). Modes: `authority` (only the node's authority may call) vs `any_peer`. Transfer: `reliable` (events), `unreliable` (positions), `unreliable_ordered`. Use `channel` to separate streams.

Both sides must have the RPC function **with the same signature at the same node path**.

## Spawning & sync nodes

- `MultiplayerSpawner`: set *Spawn Path* + auto-spawn scene list; the server `add_child`s with unique names (`str(peer_id)`) and clients mirror it. Use `spawn_function` for custom data.
- `MultiplayerSynchronizer`: replicate properties listed in its Replication config (`position`, `velocity`, `animation`), with sync vs on-change, visibility per peer, and `replication_interval`.
- Authority per node: `set_multiplayer_authority(peer_id)` — e.g. let each client own its input-synchronizer while the server owns the body.

Common split: `Player` (server authority, synced position) + `PlayerInput` child (client authority, synced input vector).

## Lag

- Interpolate remote entities between snapshots (render ~100 ms in the past).
- Client-side prediction + reconciliation for the local player in fast games.
- Never trust client values (damage, position jumps) — validate on server.

## Testing locally

Debug → *Customize Run Instances* → run 2-4 instances, with launch args like `--server`. Use `OS.get_cmdline_args()` to auto-host/join.

## Pitfalls

- Spawned node names must match across peers or RPCs/sync fail ("node not found").
- Calling RPCs before the peer is connected or before the node exists on the remote side.
- NAT: players behind routers can't host without port forwarding; use a relay/dedicated server, Steam networking (GodotSteam), or WebRTC with signaling/TURN.
