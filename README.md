# FinSight — all your bank accounts, cards and loans in one place

FinSight is a private Android app. It reads your **SMS bank alerts**, your **Gmail** (read-only), and **WhatsApp / Telegram / bank-app notifications**, and combines them into one dashboard:

| Section | What it shows |
|---|---|
| Net position | Bank balances + deposits − card dues − loans outstanding |
| Bank accounts | Each account (bank + last digits), latest balance and when it was seen, recent debits/credits |
| Credit cards | Total due, minimum due, due date, credit limit, available limit, "paid" once the bill payment SMS arrives |
| Loans | EMI, next due date, principal outstanding, overdue |
| Deposits | FD / RD amount, maturity value and date |
| Upcoming dues | Card bills and EMIs due in the next 45 days, with reminder notifications |

**Privacy:** everything stays on your phone. There is no server. SMS are read live from your inbox and never copied; from Gmail, notifications and imports only messages with financial details are saved, in the app's private storage. The app can be locked with fingerprint / screen lock (on by default).

---

## Part A — Get the APK (one time, about 10 minutes, easiest on your MacBook)

The APK is built for free by GitHub. You only upload this folder once.

1. Create a free account at **github.com** (skip if you have one).
2. Click **+ → New repository**. Name: `finsight`. Choose **Private**. Click **Create repository**.
3. Upload the project:
   - On the new repository page click **uploading an existing file**.
   - On the Mac, unzip `FinSight.zip`, open the `FinSight` folder in Finder and press **⌘ + Shift + .** so the hidden `.github` folder becomes visible.
   - Select **everything inside** the `FinSight` folder (including `.github`) and drag it onto the GitHub page. Click **Commit changes**.
   - *Alternative:* open Claude Code on the MacBook in the unzipped folder and ask: "push this folder to my private GitHub repo finsight".
4. Open the **Actions** tab. The "Build APK" job starts by itself and takes about 5 minutes. Wait for the green tick.
   - If you don't see a run, check that the file `.github/workflows/build.yml` was uploaded. If not: Actions → "set up a workflow yourself", paste the contents of `build.yml`, and commit.
5. On the repository's main page, open **Releases** (right side) → **FinSight build 1** → download **FinSight.apk**. Do this in your phone's browser while signed in to GitHub.
6. Open the downloaded APK on the phone. Allow **Install unknown apps** for your browser/Files app when asked, then tap **Install**. If Play Protect warns that the app is unknown, choose **More details → Install anyway** (it warns because the app reads SMS and is not from the Play Store).

Every time a change is pushed to the repository, a new build appears in Releases. Install it over the old one; your data stays.

---

## Part B — Connect your sources inside the app

Open FinSight → **Sources**.

### 1. SMS (most important)
Tap **Allow SMS access**. If Android says it is a *restricted setting*: Settings → Apps → FinSight → **⋮ (top right) → Allow restricted settings**, then tap the button again.

### 2. Gmail (read-only, using your own free Google key)
Do this once on console.cloud.google.com, signed in with the same Gmail:

1. **Create a project:** top bar → Select a project → **New project** → name `FinSight` → Create.
2. **Enable the Gmail API:** ☰ → APIs & Services → **Library** → search "Gmail API" → **Enable**.
3. **Consent screen:** ☰ → APIs & Services → **OAuth consent screen** (also called Google Auth Platform) → Get started. App name `FinSight`, your e-mail, Audience **External**, finish.
4. **Add yourself as a test user:** **Audience** → Test users → **Add users** → your Gmail address → Save.
5. **Create the key:** **Clients** → **Create client** → Application type **Desktop app** → name `FinSight` → Create. Copy the **Client ID** and **Client secret**.
6. In FinSight → Sources → Gmail: paste both and tap **Sign in with Google**.
   - Google shows "Google hasn't verified this app": that is your own app → **Continue**, tick the Gmail permission, **Continue**.
   - Tap **Return to FinSight** on the page that follows.
   - If the browser shows an error page instead, copy the full address from the address bar (it starts with `http://127.0.0.1`) and paste it into the box under the button, then tap **Finish sign-in with pasted address**.

The first check reads bank e-mails from the last 2 years. Later checks only fetch new ones (automatically every few hours when you open the app, or with **Refresh**).

Note: while your Google project is in "Testing", Google asks you to sign in again roughly once a week. That is normal.

### 3. WhatsApp, Telegram and bank-app notifications
WhatsApp and Telegram do not allow any app to read old chats directly. Tap **Turn on notification access** and switch FinSight on. From then on, new bank messages that arrive as notifications (WhatsApp banking, Telegram, your banks' own apps) are saved — only the financial ones. If the switch is greyed out, use **Allow restricted settings** as above.

### 4. Old chats
- **WhatsApp:** open the bank's chat → ⋮ → More → **Export chat** → **Without media** → choose **FinSight** (or "Import to FinSight").
- **Telegram:** on a computer, Telegram Desktop → open the chat → ⋮ → **Export chat history** → format **JSON**. Copy `result.json` to the phone and use **Import a file**. On the phone you can also copy messages and use **Paste messages**.

### 5. Background checks and reminders
On by default (Sources → **Background checks**). Every 4 hours, even when the app is closed, FinSight reads new SMS, fetches new Gmail bank e-mails and updates the dashboard. It sends a notification **3 days before** and **on the day** a credit card bill or EMI is due, and tells you if Gmail needs you to sign in again. Allow notifications when asked. **Check now** runs a check immediately. Android may delay checks when the battery is low; if your phone has aggressive battery saving (Xiaomi, Vivo, Oppo, Realme…), set FinSight's battery usage to **Unrestricted** in App info.

---

## Good to know
- Figures are the **latest value mentioned in a message** for each account, with the date it was seen. If a bank never sends the balance in its alerts, the account shows its transactions but no balance.
- Accounts are matched by bank + last 3–4 digits of the account / card number. Promotional messages ("pre-approved loan…") and OTPs are ignored.
- Delete everything any time: Sources → **Delete all saved messages**. Disconnecting Gmail also revokes the app's access at Google.
- The signing key (`app/finsight.jks`) is in the repository so every build installs as an update. Keep the repository **private**.

## Project layout
- `app/src/main/java/com/abhishek/finsight/core/` — message parser and account combiner (plain Java)
- `app/src/main/java/com/abhishek/finsight/` — Android screens, SMS reader, Gmail client, notification listener, importer
- `.github/workflows/build.yml` — builds the APK on GitHub and publishes it under Releases
