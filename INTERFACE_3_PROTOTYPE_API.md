🌌 Space AI: Interface Specification — Phase 3 Prototype (API)

> **Live Interactive Prototype:**
> 👉 **Launch Space AI (API)** → https://liborbenes-us.github.io/Space-AI/space_ai_api.html

![Space AI Prototype Interface API Layout][blueprint]

A browser-native design workspace where a **stealth micro-agent** appears on hover over a live UI element and lets you restyle it in real time using **natural language**. Unlike the Phase 2 local prototype, commands are interpreted by **Google Gemini 3.8 Flash** through a small **Cloudflare Worker** proxy. When the API is unavailable, the tool falls back silently to the local keyword parser — the canvas never breaks.


📄 Metadata & Simulation Specifications

Field: Value

• **Document:** `INTERFACE_3_PROTOTYPE_API.md`

• **Principal Designer & Visionary:** Dr. Libor Benes, M.A.

• **Original visual specification:** Formulated with Google AI

• **Reference implementation:** Orchestrated with Claude 3.5 Sonnet and DeepSeek AI

• **Architecture Phase:** Phase 3 — LLM-Driven Parsing & Public Deployment

• **Prototype status:** ✅ Working — Gemini-backed, with local fallback

• **System Core:** Single-file HTML/CSS/JS app + Cloudflare Worker proxy

• **Parser mode:** Google Gemini 3.8 Flash via Cloudflare Worker

• **Companion documents:**

  • `INTERFACE_1_STATIC.md` — static image specification

  • `INTERFACE_2_PROTOTYPE_LOCAL.md` — local, offline prototype


🕹️ What It Is

Space AI Phase 3 is the **same interface** as Phase 2 with one crucial upgrade: the natural-language parser is now a real LLM. Type a mood or reference — *"make it feel like a 1990s ATM button"*, *"luxury watch brand"*, *"friendly for a kids' app"* — and Gemini translates that into the same style-token object the local parser produces.

Everything else is identical:

• The same hovering agent panel with clickable chips

• The same live property sidebar

• The same Copy CSS export

• The same state object and renderer

The difference is **what's inside `callLLM()`**. Phase 2 uses keyword matching. Phase 3 sends the prompt to Gemini with a system prompt that teaches it the schema. That's the entire architectural shift.


🏗️ Architecture

Three components, connected in a chain:

```
[ Browser ]
     │  POST with prompt + API key
     ▼
[ Cloudflare Worker ]   ◄── CORS proxy, no data stored
     │  forwards as-is
     ▼
[ Google Gemini API ]
     │  returns JSON style patch
     ▼
[ Cloudflare Worker ]
     │  adds CORS headers, returns to browser
     ▼
[ Browser: applyPatch() → render() → canvas updates ]
```

Why the Worker is necessary

The Gemini API does not send CORS headers. A browser cannot call it directly from a static page — the request is blocked before it leaves the machine. The Cloudflare Worker sits in the middle, forwards the request, and adds the headers the browser needs. It is a **transparent proxy**: it inspects nothing, stores nothing, and knows nothing about the
API key beyond forwarding it.


🔐 The API Key Model — Option B

This prototype uses the **"bring your own key"** pattern. Every user supplies their own free Gemini API key and their own Cloudflare Worker URL. Both are stored only in the browser's `localStorage` and are sent only to Google, via the Worker the user themselves deployed.

Why this design

• **No secrets in the page.** The HTML file contains no API key. Nothing to leak, nothing   to rotate, nothing to steal.

• **No shared quota.** Each user's requests count against their own free tier. One heavy   user cannot exhaust anyone else's allowance.

• **No server to maintain.** There is no backend, no database, no session state. The tool   is a static file plus a user-deployed Worker.

• **Graceful when unconfigured.** If a visitor doesn't enter a key, the tool falls back   to the local parser and still works.

What the user needs

```
What                             | Where to get it                                                                                          | Cost       |
Gemini API key             | [aistudio.google.com/app/apikey](https://aistudio.google.com/app/apikey) | Free tier |
Cloudflare Worker URL | Deployed by the user (see Setup below)                                                    | Free tier |
Browser                         | Chrome, Firefox, Safari, or Edge                                                               |      —     |
```

🧠 The System Prompt

The parser's behavior is defined by a single system prompt that teaches Gemini the style-token schema. It is short — about 40 lines — and it covers:

• Every valid property and its allowed values

• The rule that output must be **JSON only** (no explanation, no markdown)

• Rules for named colors → hex conversion

• Rules for mood-based prompts ("1990s ATM" → sensible multi-property response)

• The exact casing of `"3D"`

If you want to extend the schema — add `textColor`, `fontFamily`, `borderWidth`, etc. — you edit the system prompt, the state object, and the renderer. Everything else stays the same.


✨ What the API Unlocks (beyond Phase 2)

The chip rows are identical to Phase 2, but the prompt now understands prompts the local parser can never handle:

```
make it feel like a 1990s ATM button
style it for a luxury watch brand — understated, matte, precise
friendly for a children's app
cyberpunk but not cliché — think Blade Runner, not neon signs
make it accessible: high contrast, clear focus state
warm 1970s, mustard and brown
cold nordic minimalism, pale blue and off-white
Tokyo at night
```

None of those map to keyword lists. All of them are things an LLM can reason about and translate into the schema.


🔄 Graceful Fallback

If any part of the API chain fails, the tool does not break:

```
Failure                                        | Behavior
No API key entered                     | Falls back to local parser, badge shows `LOCAL`
No Worker URL entered              | Falls back to local parser, badge shows `LOCAL`
Gemini returns `503` (overload)  | Falls back to local parser, badge shows `LOCAL`
Gemini returns `429` (rate limit) | Falls back to local parser, badge shows `LOCAL`
Worker unreachable                    | Falls back to local parser, badge shows `LOCAL`
Malformed JSON response         | Falls back to local parser, badge shows `LOCAL`
```

The status badge in the agent header always tells you which path was taken:
`GEMINI` for a successful API call, `LOCAL` for fallback.


🧰 Setup — Three Steps

Step 1: Get a Gemini API Key

1. Open [aistudio.google.com/app/apikey](https://aistudio.google.com/app/apikey)

2. Sign in with a Google account

3. Click **Create API key** → choose **Create API key in new project**

4. Copy the key immediately — it looks like `AIzaSy...`, about 39 characters

Store it somewhere safe. Do not commit it to GitHub. Treat it like a password.


Step 2: Deploy the Cloudflare Worker

This is a one-time setup. The Worker is a ~40-line file that adds CORS headers and forwards requests to Gemini.

**Via the Cloudflare dashboard:**

1. Open the [Cloudflare dashboard](https://dash.cloudflare.com)

2. Go to **Workers & Pages**

3. Click **Create application** → **Start with Hello World!**

4. Name it `space-ai-proxy` and click **Deploy**

5. Click **Edit code**

6. Delete the default code and paste the Worker source (see appendix)

7. Click **Deploy**

8. Copy the URL Cloudflare assigns — it looks like

   `https://space-ai-proxy.YOUR-SUBDOMAIN.workers.dev`

**Verify it works:** open that URL in a browser. You should see `Method not allowed`. That confirms the Worker is live and responding.

Step 3: Use the Tool

1. Open the prototype:

   https://liborbenes-us.github.io/Space-AI/space_ai_api.html

2. Hover the green button → the agent panel appears

3. Paste your Gemini API key → click **Save** (badge turns green)

4. Paste your Cloudflare Worker URL → click **Save** (badge turns green)

5. Type a prompt and press Enter

Both values are stored in the browser's `localStorage`. You enter them once per browser. They are never sent to any server other than Google, via your own Worker.


🧪 Verifying It Works

After entering both values, try this prompt:

```
make it feel like a 1990s ATM button
```

Watch the status badge in the agent header:

• **`🧠 THINKING...`** → then **`GEMINI`** — the API path worked

• **`🧠 THINKING...`** → then **`LOCAL`** — the API failed, fallback used

Click **"show raw patch"** under the key inputs to see exactly what Gemini returned, or the error message if the call failed. This is the single most useful diagnostic in the prototype.


⚠️ Known Issues & Limitations

**Gemini 503 errors.** Gemini 3.8 Flash is a new, high-demand model. Google's API occasionally returns `503 Service Unavailable` when the model is overloaded. This is a server-side issue, not a bug in the tool. The fallback parser keeps the tool usable while it clears. Waiting a few minutes and retrying resolves it.

**Free tier rate limits.** Google's free tier allows roughly **15 requests per minute** and a daily cap. Rapid-fire prompts will trip the limit and trigger fallback. Space out your prompts by a few seconds for smooth testing.

**Schema is fixed.** The LLM can only produce properties that exist in the system prompt. If a user asks for something the schema doesn't cover — text color, font family, border color — it will either be ignored or mapped onto the nearest existing property. Extending the schema requires editing the system prompt.

**One element only.** The canvas still holds a single button. Multi-element selection is planned for a later phase.

**No persistence across browsers.** The key and Worker URL are stored per-browser in `localStorage`. Switching browsers means entering them again.


🖥️ Features at a Glance

🎨 **15 colors** + synonyms + hex + darker/lighter modifiers

🔷 **4 shapes**, 4 sizes, taller/shorter

🌗 **Shade** ramp in 5 directions, auto-tinted to the current color

🎭 **10 named themes** for multi-color palettes

✨ **7 effects** (glow, shadow, stripes, glass, outline, 3D, alive) — all composable

✍️ **6 text styles**, 3 hover behaviors, 15 icons

🧠 **Gemini 3.8 Flash** parsing via Cloudflare Worker

🛡️ **Local fallback** — the tool never breaks, it just gets simpler

📋 **Copy CSS** export — production-ready output

📊 **Live properties sidebar** and **command log**

🔍 **Raw patch debug panel** — see exactly what the LLM returned

🪶 **Zero backend** — a static file plus a user-deployed Worker


🔒 Security & Privacy

• **No keys in the code.** The HTML contains no API key. Users supply their own.

• **No server-side storage.** The Worker is stateless. It forwards requests and forgets  them.

• **No analytics, no tracking.** Nothing is collected, logged, or transmitted beyond the   Gemini request itself.

• **Browser-only storage.** Keys and Worker URLs live in `localStorage` and can be   cleared from the browser at any time.

• **No `eval()`, no `innerHTML`.** All chip rows and log entries are built with   `document.createElement` and `textContent`.

• **Clipboard write only.** The Copy CSS button writes to the clipboard. The clipboard is   never read.

**A note on Google's free tier.** Google states that prompts and responses on the free tier may be used to improve their models. Users who need stricter privacy should review Google's terms or consider a paid tier.


🛠️ Technical Details

Property: Value

• **Language:** HTML + CSS + vanilla JavaScript (no build step)

• **Companion service:** Cloudflare Worker (~40 lines of JavaScript)

• **Model:** Google Gemini 3.8 Flash

• **File size:** ~40 KB, single HTML file

• **Dependencies:** None (browser-side)

• **Compatibility:** Any modern desktop browser

• **Hosting:** GitHub Pages (static) + Cloudflare Workers (proxy)

• **Tests:** Manual — validated on Firefox 140+, Chrome 120+


📁 File Structure

```
Space-AI/
├── README.md
├── INTERFACE_1_STATIC.md
├── INTERFACE_2_PROTOTYPE_LOCAL.md
├── INTERFACE_3_PROTOTYPE_API.md      ← this document
├── space_ai.html                     ← local prototype
├── space_ai_api.html                 ← API prototype (this file)
└── assets/
    └── blueprint.png
```


🔭 What's Next — Phase 4

Phase 4 extends the pattern beyond a single button:

• **Multiple elements** on the canvas — cards, forms, navigation, modals

• **Element selection** — the left sidebar becomes functional; clicking an element  focuses its agent

• **Expanded schema** — text color, font family, border properties, spacing, layout

• **Undo / redo** — every patch pushes onto a history stack

• **State persistence** — save a design as JSON, reload it later, share via URL

• **Hosted proxy** — a shared Worker so visitors don't need to deploy their own

The Phase 3 architecture — one state object, one parser, one renderer, one DOM writer — carries forward unchanged. Only the canvas grows.


⚖️ Legal Disclaimer

Space AI is an independent interface prototype developed for informational and design exploration purposes and a proof-of-concept in the new AI paradigm for software creation. It is not affiliated with, endorsed by, or sponsored by Google, Anthropic, Cloudflare, Mozilla, or any other entity or brand mentioned within the project. The software is provided "as-is" without warranties of any kind. The developer assumes no
liability for any issues arising from its use.


Appendix — Cloudflare Worker Source

```javascript
export default {
  async fetch(request, env, ctx) {
    const corsHeaders = {
      "Access-Control-Allow-Origin": "*",
      "Access-Control-Allow-Methods": "POST, OPTIONS",
      "Access-Control-Allow-Headers": "Content-Type",
    };

    if (request.method === "OPTIONS") {
      return new Response(null, { headers: corsHeaders });
    }

    if (request.method !== "POST") {
      return new Response("Method not allowed", {
        status: 405,
        headers: corsHeaders
      });
    }

    const url = new URL(request.url);
    const targetUrl = "https://generativelanguage.googleapis.com" + url.pathname + url.search;

    try {
      const body = await request.text();

      const response = await fetch(targetUrl, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: body,
      });

      const responseBody = await response.text();

      return new Response(responseBody, {
        status: response.status,
        headers: {
          ...corsHeaders,
          "Content-Type": "application/json",
        },
      });
    } catch (err) {
      return new Response(JSON.stringify({ error: err.message }), {
        status: 500,
        headers: { ...corsHeaders, "Content-Type": "application/json" },
      });
    }
  },
};
```


• Developed in collaboration DeepSeek AI, in a follow-up to the previous phases developed with Google AI and Claude 3.5 Sonnet, and DeepSeek AI.

• Last updated: Tuesday, September 22, 2026.

[blueprint]: https://github.com/LiborBenes-US/Space-AI/blob/main/INTERFACE_3_PROTOTYPE_API.png