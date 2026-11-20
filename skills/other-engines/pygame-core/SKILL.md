---
name: pygame-core
description: Make 2D games with pygame-ce (Community Edition) — game loop with fixed delta, event handling, Surfaces and convert_alpha, sprites and groups, Rects and collisions, FRect, scaling/pixel art, text, sound and music mixer, scene state machine, and packaging with pyinstaller or pygbag for the web. Use for any pygame project.
---

# pygame-ce

Install `pip install pygame-ce` (imports as `pygame`; don't install both `pygame` and `pygame-ce` in one environment).

## Main loop

```python
import pygame

WIDTH, HEIGHT, FPS = 320, 180, 60

def main() -> None:
    pygame.init()
    screen = pygame.display.set_mode((WIDTH, HEIGHT), pygame.SCALED | pygame.RESIZABLE)  # integer-ish scaling, keeps logic res
    pygame.display.set_caption("My Game")
    clock = pygame.Clock()  # pygame-ce alias of pygame.time.Clock
    game = Game()

    running = True
    while running:
        dt = min(clock.tick(FPS) / 1000.0, 0.05)   # seconds, clamped
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            game.handle_event(event)
        game.update(dt)
        screen.fill("#1d1d2b")
        game.draw(screen)
        pygame.display.flip()
    pygame.quit()

if __name__ == "__main__":
    main()
```

`pygame.SCALED` renders at logical resolution and scales to the window — ideal for pixel art.

## Assets

```python
from pathlib import Path
ASSETS = Path(__file__).parent / "assets"
img = pygame.image.load(ASSETS / "hero.png").convert_alpha()   # convert after set_mode! huge speedup
bg = pygame.image.load(ASSETS / "bg.png").convert()
frames = [sheet.subsurface((i * 16, 0, 16, 16)) for i in range(6)]
big = pygame.transform.scale_by(img, 2)                        # nearest by default
```

Load everything once at startup into a dict; never in the loop.

## Sprites, groups, collisions

```python
class Player(pygame.sprite.Sprite):
    def __init__(self, pos, *groups):
        super().__init__(*groups)
        self.image = PLAYER_IMG
        self.rect = self.image.get_frect(center=pos)   # FRect: float positions (pygame-ce)
        self.vel = pygame.Vector2()

    def update(self, dt: float) -> None:
        keys = pygame.key.get_pressed()
        direction = pygame.Vector2(keys[pygame.K_d] - keys[pygame.K_a], keys[pygame.K_s] - keys[pygame.K_w])
        if direction.length_squared():
            direction.normalize_ip()
        self.rect.center += direction * 120 * dt

all_sprites = pygame.sprite.Group()
enemies = pygame.sprite.Group()
player = Player((100, 100), all_sprites)

hits = pygame.sprite.spritecollide(player, enemies, dokill=False)                 # rect test
hits = pygame.sprite.spritecollide(player, enemies, False, pygame.sprite.collide_mask)  # pixel-perfect
all_sprites.update(dt); all_sprites.draw(screen)
```

Tile collision: move X then resolve, move Y then resolve, against a list of wall rects (`rect.collidelistall(walls)`).

## Input

- Discrete actions (jump, shoot, menu): `KEYDOWN` events in `handle_event`.
- Continuous movement: `pygame.key.get_pressed()`.
- Gamepad: `pygame.joystick.Joystick(i)` + `JOYBUTTONDOWN` / `get_axis`; or `pygame._sdl2.controller` for standardized layouts.

## Text & audio

```python
font = pygame.font.Font(ASSETS / "pixel.ttf", 8)
surf = font.render(f"Score {score}", False, "white")   # antialias False for pixel fonts; cache when unchanged

pygame.mixer.music.load(ASSETS / "music.ogg"); pygame.mixer.music.play(-1, fade_ms=1000)
jump = pygame.mixer.Sound(ASSETS / "jump.wav"); jump.set_volume(0.5); jump.play()
```

## Scenes

```python
class Scene:
    def handle_event(self, e): ...
    def update(self, dt): ...
    def draw(self, surf): ...
# Game holds self.scene and swaps it: self.scene = PlayScene(self)
```

## Camera

Draw sprites with an offset: `screen.blit(s.image, s.rect.move(-cam.x, -cam.y))`; clamp camera to level bounds.

## Distribution

- Desktop: `pyinstaller --onefile --windowed --add-data "assets:assets" main.py` (use `sys._MEIPASS` aware path helper).
- Web: **pygbag** (`pip install pygbag; pygbag .`) — requires an `async def main()` with `await asyncio.sleep(0)` each frame.

## Performance

`convert()`/`convert_alpha()` every image; avoid per-frame `transform.rotate/scale` (cache); limit `font.render` calls; use `Group.draw` and dirty rects for heavy scenes; profile with `cProfile`.
