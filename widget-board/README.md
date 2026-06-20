# Widget Board

A single-file widget dashboard — like a personal Windows widgets board, but it runs in any browser. One `index.html`, no build step, no install.

## Widgets

- **Clock & date** — live, updates every second
- **Weather** — current conditions via [Open-Meteo](https://open-meteo.com/) (free, no API key). Uses your location, or set a city in Settings.
- **To-do** — add / check off / delete tasks, saved on your device
- **Quick notes** — a scratchpad that autosaves
- **GitHub** — your latest repos, pulled live from the GitHub API (works out of the box)
- **Market movers** — live Fed Funds rate + latest Trump-related headlines
- **Notion** — connect a Notion database via a small proxy (see below)

All personal data (tasks, notes, settings) is stored in your browser's `localStorage` — nothing is uploaded anywhere.

## Run it

Just open `index.html` in any browser. To run it as a clean "board" window on Windows, open it in a browser and use **Install as app** (Chrome/Edge: ⋮ → Cast, save, share → Install this site as an app), which gives you a frameless always-available window.

## Publish (optional)

Push this repo and enable **GitHub Pages** → it'll be live at
`https://edwinjr44.github.io/Github/widget-board/`.

## Connecting Notion

Browsers can't call Notion's API directly — the API blocks cross-origin requests and your secret token must never sit in client-side code. The fix is a tiny **proxy**: it holds the token server-side and returns plain JSON the widget can read. Paste the proxy URL into **⚙ Settings → Notion proxy URL**.

The widget expects JSON shaped like:

```json
[{ "title": "Buy milk" }, { "title": "Ship feature" }]
```

(It also accepts `{ "results": [...] }` and reads `title`, `name`, or `text` fields.)

### Example proxy — Cloudflare Worker (free)

```js
export default {
  async fetch(req, env) {
    const r = await fetch(`https://api.notion.com/v1/databases/${env.DB_ID}/query`, {
      method: "POST",
      headers: {
        "Authorization": `Bearer ${env.NOTION_TOKEN}`,
        "Notion-Version": "2022-06-28",
        "Content-Type": "application/json"
      }
    });
    const data = await r.json();
    const items = (data.results || []).map(p => {
      const props = Object.values(p.properties);
      const titleProp = props.find(x => x.type === "title");
      return { title: titleProp?.title?.[0]?.plain_text || "(untitled)" };
    });
    return new Response(JSON.stringify(items), {
      headers: { "Content-Type": "application/json", "Access-Control-Allow-Origin": "*" }
    });
  }
};
```

Set `NOTION_TOKEN` and `DB_ID` as Worker variables (get the token from a Notion integration at notion.so/my-integrations, and share the database with that integration). The same pattern works for other services — swap the upstream `fetch`.

## Customize

Open `index.html` — it's plain HTML/CSS/JS in one file. Each widget is one `.card` block plus one function in the `<script>`. Add your own by copying a card and writing a loader.

---
Single file · plain HTML/CSS/JS · data stays on your device.
