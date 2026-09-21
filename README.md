# Sol

A minimal, offline-first solar system simulator built with Canvas 2D and plain JavaScript.

Sol is an interactive timeline for exploring the Solar System from its formation through the far future. It combines orbital motion, changing stellar conditions, close-up planet views, satellite paths, and short scientific and cultural stories for each body.

The project is intentionally small and dependency-free. It runs directly from the filesystem, needs no build step, and keeps its rendering work inside one lightweight Canvas 2D loop.

## Features

- Complete Solar System view from the Sun to Neptune
- Time slider spanning approximately 4.6 billion years in the past to the deep future
- Non-linear cosmic timeline that gives important eras usable space
- Play, pause, speed, and orbit controls
- Close-up views for the Sun and every planet
- Separate body files for satellite and body-specific data
- Visible satellite paths and animated satellite positions
- Earth surface changes across deep time
- Sun evolution from protostar to red giant and white dwarf
- Planetary atmosphere, rings, surface bands, clouds, ice, and heat effects
- Era stories covering science, animals, mythology, and imagined futures
- Automatic quality fallback for slower hardware
- Reduced-motion support
- Responsive desktop and mobile layouts
- No external packages, images, fonts, WebGL, or network requests

## Run It

No installation is required.

Open [solar-system.html](solar-system.html) in a modern browser.

The project is designed to work with a `file://` URL, so a local development server is optional. A server can still be useful during development:

```sh
python3 -m http.server 8000
```

Then open:

```text
http://localhost:8000/solar-system.html
```

## Controls

### Pointer and touch

- Click or tap the Sun or a planet to open its close-up view.
- Use the story chips at the bottom to jump between major eras.
- Drag the time slider to move through cosmic history.
- Use the body menu to open a specific body without selecting its small canvas marker.

### Buttons

| Control | Action |
| --- | --- |
| Today | Return to the present moment |
| Bodies | Open the direct body navigator |
| Orbits | Cycle orbital animation speed |
| Quality | Switch between high and lite rendering |
| Start | Jump to the birth of the Solar System |
| Play / Pause | Move the timeline automatically |
| Time speed | Change timeline playback speed |
| Back to the solar system | Leave a close-up view |

### Keyboard

| Key | Action |
| --- | --- |
| `Space` | Play or pause time |
| `Home` | Return to today |
| `B` | Open or close the body navigator |
| `O` | Cycle orbit speed |
| `L` | Toggle rendering quality |
| `Left Arrow` | Move one era backward, or move the system timeline backward |
| `Right Arrow` | Move one era forward, or move the system timeline forward |
| `Escape` | Close the current view or menu |

## Project Layout

```text
.
|-- solar-system.html       Main document, simulation engine, stories, and Canvas renderer
|-- styles.css               Responsive visual styling and typography
|-- bodies/
|   |-- sun.js               Sun metadata and satellite list
|   |-- mercury.js           Mercury metadata and satellite list
|   |-- venus.js             Venus metadata and satellite list
|   |-- earth.js             Earth metadata and Moon data
|   |-- mars.js              Mars metadata and Phobos / Deimos data
|   |-- jupiter.js           Jupiter metadata and Galilean moon data
|   |-- saturn.js            Saturn metadata and major moon data
|   |-- uranus.js            Uranus metadata and major moon data
|   `-- neptune.js           Neptune metadata and Triton / Nereid data
`-- README.md
```

## How It Works

### Rendering

The main page uses a single Canvas 2D context. The renderer uses:

- A small repeated star tile instead of drawing thousands of stars every frame
- Batched orbit strokes
- A capped device pixel ratio
- A lite mode with lower resolution, fewer orbit points, and reduced glow
- Dirty-frame skipping when the scene is idle
- A simple animation clock for orbital and satellite movement
- CSS and DOM overlays only for text and controls

No WebGL, texture pack, shader, chart library, or UI framework is required.

### Cosmic time

The timeline value is normalized to a number from `0` to `1`, then mapped through a set of keyframes. The mapping is intentionally non-linear:

- The early Solar System gets room for formation events.
- Geological and biological eras remain selectable.
- The present is easy to reach.
- The red giant and white dwarf stages remain visible.
- The far future uses logarithmic compression so the range can extend far beyond the lifetime of the current universe.

Times are represented internally as millions of years relative to the present. Negative values are in the past; positive values are in the future.

### Body data

Each file in [bodies](bodies) adds one entry to the shared `SOL_BODY_DATA` registry:

```js
window.SOL_BODY_DATA[3]={name:'Earth',type:'planet',moons:[
 {name:'Moon',a:5.8,size:1.7,period:27.3,phase:.8,color:'#c7ccd5'}
]};
```

Satellite values are display parameters rather than a strict physical scale:

| Property | Meaning |
| --- | --- |
| `name` | Label used in close-up views |
| `a` | Display orbit distance multiplier |
| `size` | Display radius multiplier |
| `period` | Relative animation period |
| `phase` | Starting orbital angle in radians |
| `color` | Satellite display color |

This keeps small moons visible at the same time as distant planets. The simulation is a visual model, not a navigation-grade ephemeris.

## Adding a Body or Satellite

To add a satellite to an existing body:

1. Open the matching file in [bodies](bodies).
2. Add an object to its `moons` array.
3. Set its display properties.
4. Reload the page.

Example:

```js
{ name: 'Example', a: 6, size: 1, period: 12, phase: 0, color: '#bfc7d4' }
```

For a new planet, the renderer also needs a matching entry in the main `BN` and `PL` data, plus a visual style in `LOOK` and a story list in `ERAS`. Keep the numeric body id consistent across all of those structures.

## Stories and Eras

Stories live in `solar-system.html` because they are used by the timeline and close-up panel together.

Each era is created with:

```js
E(startMa, kind, name, description, items, fact)
```

The `kind` value controls the panel label and accent color:

- `physics`
- `fauna`
- `myth`
- `fiction`

Scientific claims are intentionally short and readable. Fictional entries are marked as invented in the interface. Cultural stories are presented as cultural history, not as scientific explanations.

## Visual Model

The simulator favors readable relationships over physical scale:

- Orbital distances are square-root compressed so inner and outer planets fit together.
- Planet sizes are enlarged for recognition.
- Satellite distances and sizes are enlarged so they remain visible.
- The Sun changes radius, color, brightness, and mass over time.
- Lost or engulfed bodies are replaced by a ghost marker where appropriate.
- Close-up views use separate planet drawing rules for bands, clouds, surface areas, rings, atmosphere, ice, and heat.

These choices are deliberate. A strictly scaled Solar System would make most bodies and satellites unreadable on an ordinary screen.

## Accessibility and Responsive Behavior

- The canvas has an accessible label.
- Timeline and buttons use native controls.
- Focus states are visible.
- The panel uses live-region updates for changing story content.
- `prefers-reduced-motion` reduces transitions and close-up animation speed.
- Mobile layouts move the story panel below the close-up and use a wider orbital scale to keep planets separated.
- The body navigator provides an alternative to selecting small objects on the canvas.

## Performance Notes

The project is intended to run on ordinary laptops and phones.

The main performance decisions are:

- No runtime dependencies
- No network requests
- No continuously generated star field
- One Canvas 2D context
- Limited device pixel ratio
- Reused typed arrays for orbit positions
- Batched orbit rendering
- Lite mode fallback after a short performance sample
- Idle-frame skipping when no state is changing

The frame counter in the upper-right corner is a rough rendering indicator, not a benchmark.

## Browser Support

Use a current version of a browser with support for:

- Canvas 2D
- ES2015 JavaScript syntax
- CSS custom properties
- `requestAnimationFrame`
- Pointer events
- `matchMedia`

The project has been tested in a Chromium-based browser at desktop and phone-sized viewports.

## Scientific Scope

Sol is an educational visualization and storytelling project. It simplifies or exaggerates several values so the whole system can be explored in one view. It should not be used for mission planning, precise orbital prediction, climate forecasting, or historical dating.

The far-future timeline contains established stellar-evolution concepts alongside clearly labeled fictional stories. Some long-term planetary outcomes are uncertain; the interface uses approximate dates and language where appropriate.

## Development Principles

- Keep the project offline-capable.
- Prefer browser APIs over dependencies.
- Keep body-specific data in the body files.
- Keep rendering work inside the existing animation loop.
- Avoid adding work to every frame unless it is visible or necessary.
- Preserve keyboard and touch access when adding controls.
- Make visual exaggerations intentional and documented.
- Keep the page usable at both desktop and phone widths.

## License

This project is licensed under the proprietary Attribution-NonCommercial-NoClaim
license in [LICENSE](LICENSE).

The original code and content are owned by `thisisaayu`. You may use, modify,
and share the project for non-commercial personal or community purposes when
the required attribution is preserved. Commercial use of the original code is
not permitted under this license.

Read [LICENSE](LICENSE) for the complete terms, including the rules for
modifications, attribution, redistribution, trademarks, and warranty.
