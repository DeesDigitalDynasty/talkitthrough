# Talk It Through 🤍

> AI-powered emotional support companion — voice & text, 5 languages, private by design.

**Built by:** [Your Name / Studio]  
**Stack:** Vanilla HTML/CSS/JS frontend · Vercel Edge Functions backend · OpenRouter AI

---

## Files

```
talk-it-through/
├── index.html       ← Landing page (animated hero)
├── register.html    ← Sign up
├── login.html       ← Log in
├── chat.html        ← Main chat interface (voice + text)
└── README.md        ← This file
```

---

## Hosting Guide (Free)

### Step 1 — Set up accounts (all free)

| Service | What for | Sign up |
|---------|----------|---------|
| **GitHub** | Store your code | github.com |
| **Vercel** | Host frontend + backend | vercel.com |
| **Supabase** | Users & auth (optional) | supabase.com |
| **OpenRouter** | AI (pay-per-use, starts free) | openrouter.ai |

---

### Step 2 — Get your AI API key

1. Go to [openrouter.ai](https://openrouter.ai) → Sign up
2. Go to **Keys** → **Create Key**
3. Copy it — you'll use it in Step 4
4. Many models (including `openai/gpt-4o-mini`) are very cheap (~$0.15/million tokens)

---

### Step 3 — Push to GitHub

```bash
# In your project folder
git init
git add .
git commit -m "Initial commit: Talk It Through"

# Create a new repo on github.com, then:
git remote add origin https://github.com/YOUR_USERNAME/talk-it-through.git
git push -u origin main
```

---

### Step 4 — Deploy to Vercel

1. Go to [vercel.com](https://vercel.com) → **Add New Project**
2. Import your GitHub repo
3. Click **Deploy** (Vercel auto-detects static sites)
4. Go to **Settings → Environment Variables** and add:

```
OPENROUTER_API_KEY = your_key_here
AUTH_SECRET = any_random_32_char_string
```

Generate a random AUTH_SECRET:
```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

5. Your site is live at `https://your-project.vercel.app` 🎉

---

### Step 5 — Wire the backend (chat API)

The `chat.ts` file from your original project is already set up as a Vercel Edge Function.

Place it at:
```
api/
└── chat.ts     ← Vercel will expose this as /api/chat
```

Then in `chat.html`, update:
```js
const CONFIG = {
  apiEndpoint: '/api/chat',   // ← already correct
  model: 'openai/gpt-4o-mini'
};
```

And uncomment the `fetch` block in the `callAI()` function (look for the `WIRE YOUR API HERE` comment).

---

### Step 6 — Wire auth (optional, recommended)

For user accounts use **Supabase** (free tier: 50,000 users):

1. Create project at [supabase.com](https://supabase.com)
2. Go to **Authentication → Settings** → enable Email auth
3. Get your `SUPABASE_URL` and `SUPABASE_ANON_KEY`
4. Add the Supabase JS client to `register.html` and `login.html`:

```html
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
```

```js
const supabase = supabase.createClient('YOUR_URL', 'YOUR_ANON_KEY');

// Register
const { data, error } = await supabase.auth.signUp({ email, password });

// Login
const { data, error } = await supabase.auth.signInWithPassword({ email, password });
```

Replace the `// WIRE YOUR API HERE` blocks in both `register.html` and `login.html`.

---

## Security Checklist ✅

- [x] API key stored only as Vercel env variable (never in frontend code)
- [x] No conversation data stored by default (toggle in settings)
- [x] HTTPS enforced by Vercel automatically
- [x] Crisis resources prominently shown
- [x] No user profiling or ad tracking
- [ ] Add rate limiting to `/api/chat` (prevent abuse)
- [ ] Add Supabase Row Level Security (RLS) if storing conversations
- [ ] Add Content Security Policy headers in `vercel.json`

### Recommended `vercel.json` (add to project root):

```json
{
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        { "key": "X-Content-Type-Options", "value": "nosniff" },
        { "key": "X-Frame-Options", "value": "DENY" },
        { "key": "Referrer-Policy", "value": "strict-origin-when-cross-origin" },
        {
          "key": "Content-Security-Policy",
          "value": "default-src 'self'; script-src 'self' 'unsafe-inline' cdn.jsdelivr.net fonts.googleapis.com; style-src 'self' 'unsafe-inline' fonts.googleapis.com fonts.gstatic.com; font-src fonts.gstatic.com; connect-src 'self' *.supabase.co openrouter.ai"
        }
      ]
    }
  ]
}
```

---

## Languages Supported

| Language | Code | Voice (STT/TTS) |
|----------|------|-----------------|
| English | `en` | ✅ Full |
| Français | `fr` | ✅ Full |
| Español | `es` | ✅ Full |
| Português | `pt` | ✅ Full |
| Yorùbá | `yo` | ⚠️ Limited (browser support varies) |

> **Note:** Voice (STT/TTS) uses the browser's built-in Web Speech API — no extra cost, no external service. Works best in Chrome and Edge.

---

## The `callAI()` function — full example

```js
async function callAI(userMessage) {
  const res = await fetch('/api/chat', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      message: userMessage,
      lang: currentLang,
      model: 'openai/gpt-4o-mini',
      system: SYSTEM_PROMPTS[currentLang],
      // Pass last 10 messages for context
      history: conversationHistory.slice(-10)
    })
  });
  const data = await res.json();
  if (!data.ok) throw new Error(data.error || 'AI error');
  return data.reply;
}
```

---

## Crisis Resources

Hardcoded in the UI. Consider updating for your primary audience:

- **Nigeria:** NEEM Foundation — 08060601824
- **US/Canada:** 988 Lifeline
- **UK:** Samaritans — 116 123
- **Global:** findahelpline.com

---

*For emotional support — not a substitute for professional mental health therapy.*
