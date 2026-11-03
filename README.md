# Illusion: Creation

Adds various funny things to Illusion.

## Block outline

The block selection outline can be recoloured and resized in the client config:
`config/dxr_mc_extras-client.toml`, or in-game through **Mods → Illusion: Creation → Config**.

| Key | Default | Range | Meaning |
| --- | --- | --- | --- |
| `outline.enabled` | `true` | | Apply the settings below; `false` keeps the vanilla outline. |
| `outline.red` / `outline.green` / `outline.blue` | `0` | 0–255 | Colour channels. `0/0/0` together with the default alpha is the vanilla black. |
| `outline.alpha` | `102` | 0–255 | Opacity. Below 255 the outline stays translucent and is drawn after terrain, so it shows through blocks; 255 moves it to the solid queue, where terrain depth-tests and can hide it. |
| `outline.width` | `5` | 1–20 | Line width. 5 is the vanilla thickness; the value scales the width vanilla would use for the current window, so the outline keeps the same visual weight on HiDPI displays. |

## Port notes: ColorMeOutlines (1.19.x) → 26.3

The feature is a rewrite of [ColorMeOutlines](https://github.com/LordDeatHunter/ColorMeOutlines) (MIT) for
NeoForge 26.3. The upstream Fabric mod needed a lot of plumbing that 26.3 no longer has:

- Upstream drew through its own `RenderLayer` (`CustomRenderLayer.OUTLINE`, `VertexFormats.LINES`, a
  `RenderPhase.LineWidth` subclass) and had to mixin a `BufferBuilder` into `BlockBufferBuilderStorage`,
  swapping the vertex consumer through `@ModifyArgs` on `WorldRenderer.drawBlockOutline`.
- Its line width was applied by `RenderSystem.lineWidth()` around the draw, which in turn forced an access
  widener for `RenderPhase$LineWidth`, `RenderLayer$MultiPhase` and `RenderLayer$MultiPhaseParameters`.

In 26.3 the colour and width are plain parameters of the submit path: `LevelRenderer.submitBlockOutline`
passes `int colour` and `float width` to `submitHitOutline`, and `ShapeOutlineFeatureRenderer` writes the
width per vertex (`.setLineWidth(width)`). Both values are changed with a single pair of `@ModifyArg`
handlers in `LevelRendererOutlineMixin` (Kotlin, like every mixin in this mod):

- `submitBlockOutline` calls `submitHitOutline` twice: ordinal 0 is the black high-contrast halo, ordinal 1
  is the regular outline. Only ordinal 1 is modified, so high-contrast mode keeps its own look.
- No access widener, no custom render layer, no `RenderSystem.lineWidth` call, and no
  `BlockBufferBuilderStorage` mixin are needed any more.
