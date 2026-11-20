---
name: ebitengine-game
description: Build 2D games in Go with Ebitengine v2 — the Game interface (Update/Draw/Layout), fixed-tick logic, input via inpututil, images and sub-images, DrawImageOptions and GeoM, text/v2 rendering, audio, scenes, testing game logic, and builds for desktop, WebAssembly and mobile. Use for any Ebitengine (ebiten) project.
---

# Ebitengine (Go)

`go get github.com/hajimehoshi/ebiten/v2@latest` (pin the version in go.mod, e.g. v2.10.x).

## Skeleton

```go
package main

import (
	"log"

	"github.com/hajimehoshi/ebiten/v2"
	"github.com/hajimehoshi/ebiten/v2/inpututil"
)

const screenW, screenH = 320, 180

type Game struct {
	player Player
}

func (g *Game) Update() error {          // fixed 60 TPS by default; logic only
	if inpututil.IsKeyJustPressed(ebiten.KeyEscape) {
		return ebiten.Termination           // clean exit
	}
	g.player.Update()
	return nil
}

func (g *Game) Draw(screen *ebiten.Image) { // may run at a different rate; no state changes
	g.player.Draw(screen)
}

func (g *Game) Layout(outsideW, outsideH int) (int, int) {
	return screenW, screenH                 // logical resolution, scaled to window
}

func main() {
	ebiten.SetWindowSize(screenW*4, screenH*4)
	ebiten.SetWindowTitle("My Game")
	ebiten.SetWindowResizingMode(ebiten.WindowResizingModeEnabled)
	if err := ebiten.RunGame(&Game{}); err != nil {
		log.Fatal(err)
	}
}
```

`Update` runs at `ebiten.TPS()` (default 60) — use a fixed `dt = 1.0/60` rather than wall-clock time. `Draw` must not mutate game state.

## Input

```go
dx := 0.0
if ebiten.IsKeyPressed(ebiten.KeyArrowLeft) || ebiten.IsKeyPressed(ebiten.KeyA) { dx-- }
if ebiten.IsKeyPressed(ebiten.KeyArrowRight) || ebiten.IsKeyPressed(ebiten.KeyD) { dx++ }
jump := inpututil.IsKeyJustPressed(ebiten.KeySpace)

mx, my := ebiten.CursorPosition()        // in Layout coordinates
clicked := inpututil.IsMouseButtonJustPressed(ebiten.MouseButtonLeft)

for _, id := range ebiten.AppendGamepadIDs(nil) {
	if ebiten.IsStandardGamepadLayoutAvailable(id) {
		x := ebiten.StandardGamepadAxisValue(id, ebiten.StandardGamepadAxisLeftStickHorizontal)
		a := inpututil.IsStandardGamepadButtonJustPressed(id, ebiten.StandardGamepadButtonRightBottom)
		_, _ = x, a
	}
}
touches := inpututil.AppendJustPressedTouchIDs(nil)
```

## Images & drawing

```go
//go:embed assets/*
var assets embed.FS

func loadImage(path string) *ebiten.Image {
	f, _ := assets.Open(path)
	defer f.Close()
	img, _, err := image.Decode(f)   // import _ "image/png"
	if err != nil { log.Fatal(err) }
	return ebiten.NewImageFromImage(img)
}

sheet := loadImage("assets/hero.png")
frame := sheet.SubImage(image.Rect(i*16, 0, i*16+16, 16)).(*ebiten.Image)

op := &ebiten.DrawImageOptions{}
op.GeoM.Translate(-8, -8)            // origin to center
if facingLeft { op.GeoM.Scale(-1, 1) }
op.GeoM.Translate(p.X, p.Y)          // order: transforms apply in call order
op.ColorScale.ScaleAlpha(0.5)
screen.DrawImage(frame, op)
```

- Load images once, never inside `Draw`/`Update`.
- Sub-images of one atlas batch into a single draw call; keep draws in texture order.
- `vector` package for shapes (`vector.DrawFilledRect`, `vector.StrokeLine`); `ebiten.NewImage(w,h)` for offscreen buffers; Kage shaders (`ebiten.NewShader`, `DrawRectShader`) for effects.

## Text (text/v2)

```go
src, _ := text.NewGoTextFaceSource(bytes.NewReader(fontTTF))
face := &text.GoTextFace{Source: src, Size: 12}
op := &text.DrawOptions{}
op.GeoM.Translate(8, 8)
op.ColorScale.ScaleWithColor(color.White)
text.Draw(screen, "Score: 42", face, op)
```

For quick debug text: `ebitenutil.DebugPrint(screen, msg)`.

## Audio

```go
ctx := audio.NewContext(44100)                      // exactly one per program
stream, _ := vorbis.DecodeF32(bytes.NewReader(musicOgg))
loop := audio.NewInfiniteLoopF32(stream, stream.Length())
music, _ := ctx.NewPlayerF32(loop)
music.SetVolume(0.6)
music.Play()

// SFX: decode once into bytes, create short-lived players per play
sfx, _ := wav.DecodeF32(bytes.NewReader(jumpWav))
data, _ := io.ReadAll(sfx)
ctx.NewPlayerF32FromBytes(data).Play()
```

## Scenes

```go
type Scene interface { Update() (Scene, error); Draw(*ebiten.Image) }
```

`Game.Update` calls `current.Update()` and swaps when a new scene is returned.

## Testing

Keep game logic in plain Go types (no ebiten calls) and test with `go test ./...`. Inject input as an interface (`type Input interface{ Left() bool; JumpPressed() bool }`) so `Update` logic is testable. Rendering tests are rarely worth it.

## Builds

```bash
go build -o build/game ./cmd/game                                  # desktop (cgo needed on Linux: libgl, xorg dev pkgs)
GOOS=windows GOARCH=amd64 go build -ldflags "-H=windowsgui" -o game.exe ./cmd/game   # no console window
GOOS=js GOARCH=wasm go build -o web/game.wasm ./cmd/game
cp "$(go env GOROOT)/lib/wasm/wasm_exec.js" web/                   # Go ≥1.24 (older: misc/wasm)
go run github.com/hajimehoshi/ebiten/v2/cmd/ebitenmobile bind -target android -javapkg com.example.game -o game.aar ./mobile
```

Web: serve `index.html` loading `wasm_exec.js` + `WebAssembly.instantiateStreaming`. itch.io accepts the zipped folder.

## Pitfalls

- Creating `ebiten.NewImage` every frame → GPU memory churn.
- Using `time.Now()` deltas in `Update` → nondeterminism; rely on fixed ticks.
- Pixel-art blur: draw at Layout resolution; set `op.Filter = ebiten.FilterNearest` (default) and avoid fractional translations for crispness.
