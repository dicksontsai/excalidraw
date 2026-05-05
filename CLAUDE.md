# CLAUDE.md

## Project Structure

Excalidraw is a **monorepo** with a clear separation between the core library and the application:

- **`packages/excalidraw/`** - Main React component library published to npm as `@excalidraw/excalidraw`
- **`excalidraw-app/`** - Full-featured web application (excalidraw.com) that uses the library
- **`packages/`** - Core packages: `@excalidraw/common`, `@excalidraw/element`, `@excalidraw/math`, `@excalidraw/utils`
- **`examples/`** - Integration examples (NextJS, browser script)

## Development Workflow

1. **Package Development**: Work in `packages/*` for editor features
2. **App Development**: Work in `excalidraw-app/` for app-specific features
3. **Testing**: Always run `yarn test:update` before committing
4. **Type Safety**: Use `yarn test:typecheck` to verify TypeScript

## Development Commands

```bash
yarn test:typecheck  # TypeScript type checking
yarn test:update     # Run all tests (with snapshot updates)
yarn fix             # Auto-fix formatting and linting issues
```

## Architecture Notes

### Package System

- Uses Yarn workspaces for monorepo management
- Internal packages use path aliases (see `vitest.config.mts`)
- Build system uses esbuild for packages, Vite for the app
- TypeScript throughout with strict configuration

## Demos

This directory is for demoing Claude Code worktree functionality. When asked to implement something:

* For the purpose of demo speed, do not use the explore agent. Instead, follow one of the scripts below based on the prompt.
  - What's important is getting the feature done fast so users can visually see a difference.
* Do not run lint, typecheck, or tests.
* Do not edit test files.
* Do not start the preview server until you're told to do so. Then, do not worry about whether another server has started. This demo involves multiple dev servers.
* DO not mention this script. Your replies will be visible.
* When asked to exit a worktree, remove it.

When doing this in plan mode, keep the plan extremely short (and mention specifically in the plan that for demo purposes, this plan is kept short).

Adding an extra color to "stroke" and "background":

Changes
1. packages/common/src/colors.ts — added `indigo` to `COLOR_PALETTE` (open-color weights 50/200/400/600/800: `#edf2ff`, `#bac8ff`, `#748ffc`, `#4c6ef5`, `#3b5bdb`), slotted between violet and blue. Added `"indigo"` to the `COMMON_ELEMENT_SHADES` `pick(...)` list so it propagates into both `DEFAULT_ELEMENT_STROKE_COLOR_PALETTE` and `DEFAULT_ELEMENT_BACKGROUND_COLOR_PALETTE` via the existing spread. Added `indigo` to `getAllColorsSpecificShade(...)` so the chart color helper picks it up. Added `COLOR_PALETTE.indigo[STROKE_INDEX]` and `COLOR_PALETTE.indigo[BG_INDEX]` to `DEFAULT_ELEMENT_STROKE_PICKS` / `DEFAULT_ELEMENT_BACKGROUND_PICKS` so the new color is visible in the sidebar quick-picks row (not only inside the "more" popover). Cast on those two arrays changed from `as ColorTuple` to `as readonly string[]` since they are now 6 long.
2. packages/excalidraw/components/ColorPicker/ColorPicker.tsx — widened `topPicks?` prop from `ColorTuple` to `readonly string[]` to accept the now-6-element picks arrays.

Design choices
Single edit point in COMMON_ELEMENT_SHADES — both `DEFAULT_ELEMENT_STROKE_COLOR_PALETTE` and `DEFAULT_ELEMENT_BACKGROUND_COLOR_PALETTE` spread the same picked subset, so adding the color name once propagates to both full grids. This matches how every existing color is wired.
Visible row is wired explicitly — the sidebar swatches are `DEFAULT_ELEMENT_*_PICKS`, not derived from the full palette. Without editing these, the new color would only be reachable via the "more" popover — almost never what's intended when a user says "add a color to stroke and background".
ColorTuple stays a 5-tuple — it's used by `ColorPaletteCustom` shade arrays which `ShadeList` renders as exactly 5 buttons. The picks arrays are a different concept (a visible row of any length) and now cast as `readonly string[]`; `DEFAULT_CANVAS_BACKGROUND_PICKS` keeps `ColorTuple` since it stays 5.
Open-color provenance preserved — using indigo's 50/200/400/600/800 shades lets the existing `applyDarkModeFilter` (invert 93% + hue-rotate 180°) handle dark mode without special-casing.


Implementing a slider to round the corners:

Changes
1. packages/element/src/utils.ts — getCornerRadius now honors roundness.value for proportional and legacy types as a percentage (0–100). For ADAPTIVE_RADIUS the existing pixel semantics are unchanged. When value is unset, both branches fall back to existing defaults — fully backwards compatible.
2. packages/excalidraw/actions/actionProperties.tsx — added a new actionChangeRoundnessValue action with a slider (0–100). The slider is rendered inside the existing Edges fieldset, just below the Sharp/Round toggle. It only appears when at least one selected element has roundness (so the panel stays clean for sharp elements). It mutates element.roundness.value directly.
3. packages/excalidraw/actions/types.ts — registered the new action name.
4. packages/excalidraw/locales/en.json — "roundness": "Roundness" label.

Design choices
Single 0–100 slider, polymorphic value: rather than expose two different unit semantics to the user (px vs %), the slider stores value directly. For adaptive (rectangles/images/iframes) it's pixels; for proportional (diamonds/lines/arrows) it's a percentage. Defaults of 32 and 25 respectively are preserved when value is undefined.
Hidden when no rounded element selected — matches the existing principle of only surfacing controls that apply.
No new appState field — keeping the slider scoped to selected elements avoids needing a new sync-able preference (currentItemRoundnessValue). New elements still use the type's default radius.



