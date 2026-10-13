# AD CRAFT RailConfirm: Setup (Hinglish)

## A. Website (GitHub)
1. github.com par repository banayein (Public), is zip ki saari files upload karein.
2. Settings > Pages > Branch `main` / root > Save. Link: https://USERNAME.github.io/REPO/
3. `index.html` me `CFG` wali line (search: `CFG=`) me 5 URL baad me bharenge (neeche C).

## B. n8n me 4 workflow import karein (Workflows > Import from file)
| File | Kaam |
|---|---|
| `n8n/adcraft-workflow.json` | Inquiry + Admin + monthly reset (pehle se) |
| `n8n/railconfirm-agent-workflow.json` | AI chat agent (seat, PNR, Tatkal info) |
| `n8n/railconfirm-tatkal-reminder.json` | Tatkal khulne se 10 min pehle reminder |
| `n8n/railconfirm-seat-watch.json` | Har 30 min seat check, khulte hi Telegram alert |

## C. Keys/URLs (ek baar)
1. **Anthropic key:** console.anthropic.com > API keys. Agent workflow ke dono `Claude:` nodes me `REPLACE_WITH_ANTHROPIC_API_KEY`.
2. **Railway API key (sabse zaroori):** RapidAPI jaisi site par IRCTC-type API subscribe karein. Key `REPLACE_WITH_RAILWAY_API_KEY` ki jagah, **agent workflow ke `Check availability` node aur seat-watch ke `Check watches` node, dono me**. Endpoint naam API ke docs se match karein (`PROVIDERS` block). Doosri API ka backup jodna ho to block copy karke jodein.
3. **Telegram:** BotFather se bot, token n8n credential me, `REPLACE_WITH_TELEGRAM_CHAT_ID` Tatkal aur Seat-watch workflow me.
4. Chaaron workflow **Active** karein. Phir har Webhook ka **Production URL** copy karke `index.html` ke `CFG` me daalein:
   - `webhook` (inquiry), `admin`, `agent` (agent workflow), `tatkal` (reminder), `watch` (seat watch).
5. Pehle test ke liye `Check availability` aur `Check watches` me `const DEMO=true` karein (nakli data). Test ke baad **false** kar dein, warna customers ko nakli seat dikhegi.

## D. Test list
- Website chat: "kal Raipur se Delhi sleeper" (DEMO me "DEMO EXPRESS" aana chahiye).
- PNR box, Tatkal Reminder, Seat Alert: Telegram par message aana chahiye.
- Planner ke WhatsApp button se Google Sheet me row aani chahiye.

## E. Dhyan
- Seat-watch har 30 min me API calls karta hai. Train number likhne se calls kam hoti hain. API ka monthly limit/kharcha dekhte rahein.
- Agent me spam rok hai (20 sawal/ghanta/IP). n8n ke aage Cloudflare lagana behtar hai.
- Workflows asli n8n par test nahi hue; har node "Execute step" se check karein. Seat-watch/Tatkal ke liye n8n ko hamesha chalu rakhein.
- Do-train jodkar route abhi nahi bana. Agent split, nearby hub aur agli date check karta hai.

## Naye features (live status, connection checker, checklist, app jaisa install)
- **Live status:** website ka "Train kahan hai?" box chat agent se jawab leta hai. Agent code me `EP.live` endpoint (`/api/v1/liveTrainStatus`) apni API ke docs se match karein.
- **Connection checker aur Safar checklist:** sirf website par chalte hain, koi setup nahi.
- **App jaisa install:** `manifest.webmanifest` aur `sw.js` ke saath phone ke browser menu me "Add to Home screen" milega (HTTPS, yaani GitHub Pages par). Nayi file upload karne ke baad purani cache ke liye ek baar page refresh karein.
- Naye box ke shabd abhi sirf Hinglish me hain, baaki 12 bhasha me nahi.

## Is version mein badlav
- Bank account number aur IFSC hata diye; sirf UPI ID aur QR (`qr.png`) hain. Bank transfer ke liye customer WhatsApp par maangega.
- Privacy Policy section (`#privacy`), form ke paas sehmati line, footer link joda.
- Jab tak `CFG` mein URL nahi bharte, chat, PNR, Live status, Tatkal Reminder aur Seat Alert boxes apne aap chhupe rehte hain. URL bharte hi dikhne lagte hain.
- FAQ/Planner/Refund ke takrate hue shabd saaf kiye.
- SEO: `canonical`, `og:image`, JSON-LD, `robots.txt`, `sitemap.xml`. **Zaroori:** `index.html`, `robots.txt`, `sitemap.xml` mein `https://USERNAME.github.io/REPO/` ko apne asli GitHub Pages link se badlein.
- QR ki image alag file `qr.png` hai (index.html 215 KB se 130 KB). `qr.png` ko index.html ke saath upload karein.
