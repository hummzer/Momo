# System media

Place screenshots and walkthrough videos here. The UI looks for:

| Field | Example path | Notes |
|-------|----------------|-------|
| `imageUrl` | `/media/01.jpg` | Chart / EA screenshot (jpg/png/webp) |
| `videoUrl` | YouTube URL or `/media/01.mp4` | Backtest walkthrough |

Naming convention used in `App.vue`:

- Strategies: `01.jpg` … `12.jpg`
- Indicators: `13.jpg` … `22.jpg`

If a file is missing, a generated SVG poster is shown automatically.

To attach video, set `videoUrl` on that system object in `src/App.vue`, e.g.:

```js
videoUrl: 'https://www.youtube.com/watch?v=XXXXXXXXXXX'
// or
videoUrl: '/media/01-walkthrough.mp4'
```
