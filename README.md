# Mira — AI Copy Dashboard, Chat + Free Image Studio

Ek fast AI workspace: dark **Content Lab** dashboard (copy tools), chat (Groq + Gemini), **free keyless images (Pollinations Flux)**, aur optional secure backend.

- ⚡ **Groq** — GPT-OSS 120B / 20B, Qwen Vision
- ✨ **Gemini** — Google AI Studio ki key daalte hi available models auto-detect ho jaate hain
- 🧰 **Content Lab** — dark copy dashboard: Business Emails, Social Media Ads, Pitch Generator on Groq LPU
- 🎨 **Image Studio** — unlimited free image generation with Pollinations Flux, no API key
- 🖼️ Chat me image understanding, markdown, code blocks, reasoning, stop / retry / regenerate
- ⬇️ Generated image ka one-click **Download Image**
- 🔐 **Direct mode** (browser key) ya **Secure backend mode** (chat keys server par)
- 🚀 **Netlify-ready** — pure static frontend, koi server zaroori nahi image generation ke liye

---

## 1. Quick start (browser direct mode)

```bash
npm install
npm run dev
```

1. App kholein → top-right **Preferences** (⌘/Ctrl + `,`)
2. **Connection** tab → **Gemini** card
3. [Google AI Studio](https://aistudio.google.com/apikey) se key banayein (`AIza…`)
4. Key paste karein → **Test connection** → **Save changes**
5. Chat ke model pill se **Gemini** model select karein, ya Preferences me **Use Gemini in chat** dabayein

Bas. Key valid hui to app khud Google se model list laata hai, isliye model name guess
karne ki zaroorat nahi. Agar koi model aapki key ke liye available nahi hota, app
automatically working Gemini model par switch kar deta hai.

**Groq key** pehle se configured hai. Apni key use karni ho to usi Connection tab me
Groq card me paste kar dein ([console.groq.com/keys](https://console.groq.com/keys)).

> Direct mode me key browser ke `localStorage` me rehti hai aur seedha Groq/Google ko
> jaati hai. Personal use ke liye theek hai. Public site par niche wala backend use karein.

---

## 2. Secure backend mode (recommended for deployment)

Is folder me ek **zero-dependency Node.js proxy** hai: `server/index.mjs`.
Yehi app ko serve karta hai **aur** provider calls ko proxy karta hai. Keys `.env`
me rehti hain — browser tak kabhi nahi aati.

```bash
cp .env.example .env
# .env me apni keys daalein:
#   GEMINI_API_KEY=AIza...
#   GROQ_API_KEY=gsk_...
#   PORT=3000

npm run build
node server/index.mjs
```

Ab kholен: **http://localhost:3000** — API + app dono isi server se.

Phir app me:

1. **Preferences → Connection**
2. **Secure backend** select karein
3. Backend URL daalein (e.g. `http://localhost:3000` ya `https://your-app.com`)
4. **Test backend** → **Save changes**

App check karta hai ki server par kaun si keys configured hain, aur unke models load
karta hai. Browser me keys rakhne ki zaroorat nahi.

### Deploy kaise karein

Koi bhi Node.js host chalega (Render, Railway, Fly.io, VPS, Docker):

```bash
# build step
npm install && npm run build

# start command
node server/index.mjs
```

`GEMINI_API_KEY` aur `GROQ_API_KEY` host ke **Environment Variables** me set karein.
Netlify/Vercel jaise pure-static hosts par backend nahi chalega — wahan Direct mode
use karein, ya server ko alag deploy karke us ka URL daalein.

### Backend API

| Route | Method | Kaam |
| --- | --- | --- |
| `/api/health` | GET | Server up + kaun si keys configured hain |
| `/api/models?provider=gemini` | GET | Key ke live models |
| `/api/chat` | POST | Streaming chat (SSE) — keys server se lagti hain |

Backend kabhi bhi client se key accept nahi karta; sirf server environment se leta hai.

---

## 3. Supported providers & models

### Groq
- `openai/gpt-oss-120b` — best all-rounder
- `openai/gpt-oss-20b` — fastest
- `qwen/qwen3.8-27b` — image inputs

### Gemini
Gemini models ki fixed list hardcode nahi ki gayi. Jaise hi key save/test hoti hai,
app `GET https://generativelanguage.googleapis.com/v1beta/models` se live list laata
hai, sirf `generateContent` capable models rakhta hai, aur latest **Flash / Pro**
ko prefer karta hai. Isse Google ke model rename/retire hone par bhi app kaam karta
rehta hai.

Internally Gemini ke official **OpenAI-compatible endpoint** ka use hota hai:
`https://generativelanguage.googleapis.com/v1beta/openai/chat/completions`

---

## 4. Troubleshooting

| Problem | Fix |
| --- | --- |
| "That Gemini key didn't work" | Key `.env`/Settings me sahi paste hai? AI Studio me key active hai? Test connection dabayein. |
| Key valid hai but no models | Google AI Studio me Generative Language API enabled ho, aur key par model access ho. |
| "This model isn't available" | **Refresh models** karein. App Gemini par automatically working model try karta hai. |
| 429 / rate limit | Free tier per-minute limit. Thoda ruk kar retry karein. |
| Backend not reachable | `node server/index.mjs` chal raha hai? URL sahi hai? `npm run build` kiya? |
| Backend says key missing | `.env` me `GEMINI_API_KEY` daalein aur server restart karein. |
| "dist/index.html not found" | Project root me `npm run build` chalayein. |
| CORS error | Backend localhost origins allow karta hai. Custom domain ko `ALLOWED_ORIGINS` me add karein. |

---

## 5. Privacy & security

- Direct mode: keys `localStorage` me rehti hain. Ye client-only app hai — public deployment par key extract ho sakti hai.
- Backend mode: keys sirf server par, browser ko sirf streamed text milta hai.
- Chats browser me save hote hain. **Clear all chats** se delete kar sakte hain.
- Kabhi bhi `.env` git me commit na karein (`.gitignore` me add karein).

### Minimal `.gitignore` additions

```gitignore
.env
.env.local
node_modules
dist
```


---

## 6. 🎨 Image Studio — Pollinations Flux (free, no key)

Image generation browser me direct chalti hai. **Koi API key, account, backend ya server
nahi chahiye** — Netlify par deploy karte hi kaam karti hai.

### Endpoint (verified)

```text
GET https://image.pollinations.ai/prompt/<url-encoded prompt>
    ?width=1024&height=1024&model=flux&seed=<random>&nologo=true&enhance=false
```

- Auth header: **none** (anonymous free tier)
- Response: raw image bytes (`image/jpeg`), ~2s typical
- CORS: `Access-Control-Allow-Origin: *` (browser-safe)

App response ko blob me leta hai, `URL.createObjectURL(blob)` banata hai, aur wahi image
chat me render karta hai — JSON/HTML error pages image ki tarah render nahi hote (magic-byte
check included).

### Free tier behaviour

Pollinations ka anonymous free tier **1 image / 45 seconds per IP** allow karta hai.
Mira isse gracefully handle karta hai:

- Free-limit (`402 FREE_LIMIT` ya 429) par `Retry after Ns` ke hisaab se live countdown
  dikha kar **automatically retry** karta hai
- Har successful image ke baad 45s ka **live cooldown countdown** Generate button par dikhta hai
- Queue busy ho to clear message + "Try again" button
- `nologo=true` bheja jaata hai; anonymous generations par chhota “created with” mark aa sakta hai

Koi signup, koi API token, koi monthly credit — bas prompt likhein aur generate karein.
Agar aapko watermark-free ya higher rate limit chahiye, Pollinations ke apne account/API se
optional integration ho sakta hai, lekin Mira ka default flow **100% free** hai.

### Loading & download

Loader real generation time dikhata hai (usually 2–6s; free queue busy ho to zyada). Har image
ke neeche **Download Image** button hai, plus regenerate/copy-prompt. Compressed copy
localStorage me save hoti hai taaki reload ke baad bhi image dikhe.

### Troubleshooting

| Problem | Fix |
| --- | --- |
| “Free image limit reached” | Anonymous tier limit: 1 image / 45s per IP. App countdown dikha kar khud retry karta hai. |
| Slow generation | Free queue par load zyada hai. 10–30s normal ho sakta hai. |
| “Couldn't reach Pollinations” | Internet/adblock check karein; `image.pollinations.ai` reachable hona chahiye. |
| No watermark removal | `nologo=true` best-effort hai; guaranteed removal Pollinations account ke saath. |

### Netlify deploy

Repo me `netlify.toml` already hai:

```toml
[build]
  command = "npm run build"
  publish = "dist"
```

Koi environment variable zaroori nahi — na Groq/Gemini (agar users apni keys paste karte hain),
na image ke liye. Deploy ke baad site kholiye → Image mode → generate. Image endpoint browser
se direct call hota hai aur CORS (`*`) verified hai.
