# AD CRAFT: GitHub + n8n setup

Is folder mein:
- `index.html`: poori website (GitHub Pages par chalti hai)
- `favicon.svg`: browser tab ka logo
- `n8n/adcraft-workflow.json`: n8n workflow (inquiry, admin list, mahine ka reset)
- `n8n/Inquiries_header.csv`: Google Sheet ki pehli row (headers)
- `.nojekyll`: GitHub ko batata hai ki file jaisi hai waisi hi dikhaye

## Kaise kaam karta hai
Customer form bharta hai -> website n8n webhook ko bhejti hai -> n8n Google Sheet mein save karta hai aur Telegram par alert bhejta hai -> aap Admin Panel (ya Sheet) mein dekhte ho -> har mahine ki 1 tareekh ko n8n purani entries `Archive` tab mein copy karke `Inquiries` tab saaf kar deta hai (reset).
Agar n8n band ho ya URL na bhara ho, to form apne aap WhatsApp par khul jata hai, taaki inquiry khoye nahi.

## Step 1: Website GitHub par live karein
1. github.com par account banayein, **New repository** banayein (naam jaise `adcraft`, Public).
2. Is folder ki saari files (`index.html`, `favicon.svg`, `.nojekyll`, `n8n` folder) repository mein upload karein.
3. **Settings > Pages > Build and deployment**: Source = *Deploy from a branch*, Branch = `main`, folder = `/ (root)`. Save karein.
4. 1-2 minute baad aapka link milega: `https://<aapka-username>.github.io/adcraft/`

## Step 2: n8n taiyar karein
1. **Google Sheet** banayein. Do tab rakhein: `Inquiries` aur `Archive`. Dono mein pehli row yeh rakhein: `time, type, mobile, message, page` (alag-alag columns mein).
2. n8n mein **Workflows > Import from file** se `n8n/adcraft-workflow.json` chunein.
3. Har Google Sheets node (5 node: Save to Inquiries sheet, Read Inquiries, Read all Inquiries, Copy to Archive, Clear Inquiries) mein: apna **Google Sheets credential** jodein aur sahi **Sheet** aur **tab** chuniye (documentId "REPLACE_WITH_GOOGLE_SHEET_URL" ki jagah apni Sheet ka link).
4. **Telegram alert** node: @BotFather se bot banayein, token n8n credential mein daalein, aur `REPLACE_WITH_TELEGRAM_CHAT_ID` ki jagah apna chat ID likhein. (Telegram nahi chahiye to is node ko hata dein ya Gmail/Email node lagayein.)
5. **Key correct?** node mein `REPLACE_WITH_YOUR_SECRET_KEY` ki jagah apni lambi, mushkil secret key likhein (yahi admin password hai).
6. Workflow ko **Active** karein. Phir dono webhook nodes mein **Production URL** copy karein:
   - `Inquiry Webhook` ka URL
   - `Admin Webhook` ka URL

## Step 3: Website mein URL lagayein
`index.html` kholein, `REPLACE_WITH_N8N_WEBHOOK_URL` aur `REPLACE_WITH_N8N_ADMIN_URL` dhundhein (`CFG` wali line) aur apne Production URL paste karein. Commit karein.

## Step 4: Test
Website par koi form (jaise Payment wala) bharke bhejein. Google Sheet mein row aani chahiye aur Telegram par message.

## Admin Panel
Website ka link kholein aur aage `?admin=1` lagayein: `https://<aapka-link>/?admin=1`. Upar **Admin** chip dikhega. Dabane par admin key puchhi jayegi (jo Step 2.5 mein rakhi).
- **Inquiry Inbox:** n8n/Google Sheet se aati hai.
- **Hisaab register** (booking aur payment ki entries): sirf usi phone/browser mein rehta hai jahan aap use bharte ho. Isliye CSV download karke backup rakhein.
Admin chip customers ko kabhi nahi dikhta (sirf `?admin=1` wale link par).

## Zaroori suraksha baatein
- **Webhook URL public hota hai.** Koi spam bhej sakta hai. Isliye n8n ke aage Cloudflare ya rate limit lagayein, aur Sheet par nazar rakhein.
- Admin key sirf n8n workflow mein rakhein, website ki file mein nahi.
- `index.html` mein jo UPI ID aur QR public hongi (bank account number hata diya gaya hai). Yeh aapne dene ka faisla kiya hai.
- Customers ka naam/mobile Google Sheet mein jaata hai. Sirf zaroorat jitna rakhein aur purana data mahine mein saaf hone dein.
- Apna UPI PIN, bank password ya n8n ka password kisi ke saath share na karein.

## Dhyan dein: kya test hua, kya nahi
- Website ka code (form bhejna, admin inbox, hisaab register) maine mock server ke saath test kiya hai.
- **n8n workflow maine asli n8n par chalakar test nahi kiya.** Alag n8n versions mein kuch node ke fields alag ho sakte hain. Import ke baad har node ko ek baar kholkar check karein aur "Execute step" se test karein.
- Mahine ka reset Google Sheet ke `Inquiries` tab ko poora saaf karta hai (header chhodkar) aur pehle `Archive` mein copy karta hai.

## Apna domain (optional)
GitHub Pages ke **Settings > Pages > Custom domain** mein apna domain (jaise `adcrafttravels.in`) daalein aur domain provider par DNS record badlein. GitHub ki guide dekhein.

## Naya: RailConfirm AI Planner
Website me `#railconfirm` section jud gaya hai (nav me "Confirm Seat AI"). Isme route planner (split, nearby station, alternate train, Tatkal), WL confirm andaaza, aur FAQ hain. "Asli availability check" button wahi WhatsApp/n8n inquiry flow use karta hai, to koi naya setup nahi chahiye.
Dhyan dein: planner abhi rule-based hai, live railway data nahi dikhata. Live availability ke liye baad me authorized railway data API aur AI agent (brief ka Phase 1-2) jodna hoga.

## RailConfirm AI Agent (chat) setup
1. n8n me **Import** karein: `n8n/railconfirm-agent-workflow.json`.
2. Do jagah key daalein: dono `Claude:` HTTP nodes me `REPLACE_WITH_ANTHROPIC_API_KEY` (console.anthropic.com se), aur `Check availability` code node me `REPLACE_WITH_RAILWAY_API_KEY`.
3. Railway data API (jaise RapidAPI par IRCTC-type API) lein. Code node ke upar `EP` me endpoint ke naam apni API ke docs se match karein. Yeh unofficial APIs hain, rate limit aur terms dekh lein.
4. Workflow **Active** karein, `Agent Webhook` ka Production URL copy karke `index.html` ki `CFG` me `REPLACE_WITH_N8N_AGENT_URL` ki jagah paste karein.
5. Test: website ke chat box me "kal Raipur se Delhi sleeper" likhein.
Dhyan: yeh workflow maine asli n8n/API par chalakar test nahi kiya. Har node "Execute step" se check karein. Har chat me ~30 API calls tak ho sakti hain, to cost aur limit par nazar rakhein.

## PNR Status aur Tatkal Reminder
- **PNR:** website ka "PNR Status" box chat agent se jawab leta hai (agent workflow me PNR endpoint `EP.pnr`, docs se verify karein). PNR/naam store nahi hote.
- **Tatkal Reminder:** `n8n/railconfirm-tatkal-reminder.json` import karein, Telegram node me chat ID aur credential daalein, Active karein, uska Production URL `index.html` ki `CFG` me `REPLACE_WITH_N8N_TATKAL_URL` ki jagah paste karein. Tatkal khulne se 10 minute pehle aapko Telegram par reminder aata hai (Wait node, n8n execution saved rehni chahiye). URL na bhara ho to form WhatsApp par chala jata hai.
- Dono ab tak asli n8n par test nahi hue. Webhook public hai, spam par nazar rakhein.

## Khaas (unique) features
- **Quota Finder:** Defence/Jawan, Senior-Lower berth, Mahila, Divyangjan quota ke tips (aam apps sirf General dekhte hain). Agent bhi quota samajhta hai.
- **Local boarding hubs:** Kota/Bilaspur/Raipur/Korba/Durg/Champa ke aas-paas ke bade station aur Delhi/Mumbai/Kolkata/Varanasi ke alag station. `HB` aur `DH` list `index.html` me badha sakte hain.
- **Group helper:** 4+ yatri par PNR limit aur split-booking salah.
Quota/age ke rules badal sakte hain, booking se pehle verify karein.
