# Tell umma I'm walking to baekdusan

An interactive parallax essay with layered Korean mountain painting artwork. The text scrolls between a background and foreground layer of traditional mountain illustrations, creating a depth effect.

## Setup

```bash
npm install
npm run dev
```

Then open `http://localhost:5173` in your browser.

## Build for production

```bash
npm run build
```

The built files will be in `dist/`.

## How it works

The essay uses three visual layers:

1. **Background** (`background.png`) — painted mountains with black silhouette framing, fixed position with slow parallax
2. **Text** — the essay content in Cascadia Code, scrolls normally
3. **Foreground** (`foreground.png`) — mountains and clouds on black background that overlay the text, creating the illusion of depth

The foreground image uses a black background which composites over the text. As you scroll, the background and foreground move at different rates to create the parallax effect.

## Images

Place your images in `public/images/`:
- `foreground.png` — mountains and clouds on black (overlaps text)
- `background.png` — full mountain scene with silhouette frame

## Tech

- Vue 3 (Composition API)
- Vite
- CSS parallax via scroll-driven transforms
