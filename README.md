# 💅 NailVision

**Capture · Choose · Get your perfect look.**
Scan your hand with the camera, pick up to three polish colors, and NailVision paints **5 nail designs** on your own nails.

| | |
|---|---|
| 🖼️ **Frontend** | Vue 3 + Vite. Finds your nails right in the browser. |
| 🎨 **Design service** | A Python FastAPI server (`backend/`) that paints the designs with Stable Diffusion inpainting on Cloudflare Workers AI (or a local model). |

```
NailVision/
├── frontend/   Vue + Vite app, Capacitor Android project (frontend/android)
├── backend/    FastAPI design service, Dockerfile, tests, local model in backend/models (not in git)
├── supabase/   database schema
└── README.md
```

---

## 📖 Table of contents

1. [Get the code (clone)](#-get-the-code-clone)
2. [How to use NailVision](#-how-to-use-nailvision)
3. [Quick start](#-quick-start)
4. [Pages at a glance](#-pages-at-a-glance)
5. [Settings explained](#-settings-explained)
6. [Troubleshooting](#-troubleshooting)
7. [Mobile app (Android, Ionic Capacitor)](#-mobile-app-android-ionic-capacitor)
8. [Deploy](#-deploy)

---

## 📥 Get the code (clone)

### 🧰 Before you start
Install **Git** from <https://git-scm.com/downloads> and check it works:

```sh
git --version
```

### 📋 Clone it

```sh
git clone https://github.com/nikopolistico/NailVision.git
cd NailVision
```

This creates a `NailVision` folder with the whole project. Then continue with the [Quick start](#-quick-start).

### 🔄 Get the latest updates later
Run this inside the `NailVision` folder:

```sh
git pull
cd frontend
npm install
```

(`npm install` picks up any new packages.)

### 📦 What you do **not** get from the clone
These are too big or private, so they are not in the repository:

| Missing | What to do |
|---|---|
| `frontend/node_modules/` | Run `npm install` in `frontend/`. |
| The Python environment (`backend/.venv`) | Create it in [Quick start → Step 2](#-step-2--start-the-design-service). |
| The local Stable Diffusion model (~2 GB) | Only for `GEN_BACKEND=local`. It downloads into `backend/models/` the first time it is used. |
| Your `.env` file | Create `frontend/.env` with `VITE_SD_URL=http://localhost:8000`. |

> 🔐 Never commit real passwords or tokens. Keep them in `.env`, not in the code.

### 🍴 Want your own copy to change?
Click **Fork** on GitHub first, then clone **your** fork's address instead:

```sh
git clone https://github.com/<your-username>/NailVision.git
```

---

## 🌸 How to use NailVision

Follow these steps in the **Studio** page.

### 1️⃣ 📸 Scan your hand
Click **Start scan** and allow the camera. A **5-second countdown** gives you time to place your hand inside the frame.
Then NailVision looks for nails live. When it sees the same nails steadily, it takes the photo **by itself**: no button to press.
Click **Cancel** to close the camera, or **Rescan** later to try again.
> 💡 Tip: lay your hand flat, nails facing the camera, in even light.
>
> 🔒 The camera only works on `localhost`, on an `https` address, or in the Android app.

### 2️⃣ 🔍 Check the detected nails
NailVision finds each nail and numbers it from left to right. Up to **5 nails** are used.
- Nail missing? ⬇️ Lower the **Detection confidence** slider.
- Something that is not a nail got picked up? ⬆️ Raise it.

### 3️⃣ 🎨 Choose your polishes
Click any of the **3 color swatches** to pick a color. The strip above them shows your whole palette at once.
The first three designs use your colors. The other two are suggestions picked for you.

### 4️⃣ ⚙️ Pick the design quality

| Option | Best for |
|---|---|
| ⚡ **Quick** | A fast preview. Looks a bit rougher. |
| ✅ **Balanced** | Recommended for most photos. |
| 💎 **Best** | The finest result. Slowest. |

### 5️⃣ ✨ Press **Generate 5 designs**
- 📊 A progress bar shows how far along you are and about how long is left.
- 🖼️ Each design appears **as soon as it is finished**. You do not have to wait for all five.
- 🔀 You can visit other pages while it runs. It keeps going.

### 6️⃣ ⭐ Rate and save your favorites
- Click the **stars** under a design to rate it. Click the same star again to clear it.
- 🔎 Click a design image to view it **full screen** (press `Esc` to close).
- ⬇️ Use the download icon to save the full photo.
- 💾 Every design is saved automatically in your browser. Find them under **Previous designs**.

---

## 🚀 Quick start

### ✅ You need
- 🟢 **Node.js** 22 or newer, with npm
- 🐍 **Python** 3.12 (only to run the design service on your computer)
- 💽 About **2 GB** of free disk for the model (not needed in test mode)

### 🖥️ Step 1 · Start the app

```sh
cd frontend
npm install
npm run dev
```

Open the address it prints (usually <http://localhost:5173>).

> The app talks to the design service at
> For local use create a `frontend/.env` file with:
> `VITE_SD_URL=http://localhost:8000`
> `VITE_SUPABASE_URL: = `
> `VITE_SUPABASE_ANON_KEY: = ` 

> ### 🎨 Step 2 · Start the design service

Open a **second terminal**:

```powershell
cd backend
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
python -m uvicorn server:app --port 8000
```

> 🧠 That is enough for Cloudflare (the default). For the local Stable Diffusion model (`GEN_BACKEND=local` or `GEN_FALLBACK_LOCAL=true`) also run
> `pip install torch --index-url https://download.pytorch.org/whl/cpu` and `pip install -r requirements-local.txt`.

> 🍎🐧 On macOS or Linux use `source .venv/bin/activate` instead of the `Activate.ps1` line.
>
> ⚡ Have an NVIDIA GPU? Install the CUDA build of PyTorch from <https://pytorch.org> instead of the CPU one. The server picks the GPU by itself.

✅ **Check it works:** open <http://localhost:8000/health>. The top bar of the app should say **Design service ready** and show **GPU** or **CPU**.

### ☁️ Paint on Cloudflare Workers AI (default)
By default the server sends each nail to Cloudflare's `@cf/runwayml/stable-diffusion-v1-5-inpainting` model instead of running Stable Diffusion itself. That is fast even on a small server.

1. In the Cloudflare dashboard, copy your **Account ID** (Workers & Pages → right side).
2. Create an API token: **My Profile → API Tokens → Create Token → Workers AI** template.
3. Put them in `backend/.env.local` (copy `backend/.env.example`); the server reads it on start. Or set them in PowerShell (on Render: the Environment tab):
   ```powershell
   $env:CF_ACCOUNT_ID = "your-account-id"
   $env:CF_API_TOKEN  = "your-token"
   $env:GEN_FALLBACK_LOCAL = "true"   # optional: paint locally if Cloudflare fails
   ```
4. Check it with one real request. It saves `smoke_out.png`:
   ```powershell
   cd backend
   python -m scripts.smoke_cloudflare
   ```

To use only the local model, set `$env:GEN_BACKEND = "local"`. Cloudflare always uses 20 steps, so the **Design quality** choice only affects the local model.

Each design in `GET /jobs/{id}` records `backend` (`cloudflare`, `local` or `mock`), `seed`, `prompt`, `latency_ms` (time spent in the model), `delta_e` and `on_palette`.

Unit tests: `cd backend` then `python -m unittest discover tests`.

### 🧪 Just want to try it quickly? Use test mode
Test mode skips Stable Diffusion and returns fast placeholder designs.

```powershell
$env:MOCK = "1"
python -m uvicorn server:app --port 8000
```

(macOS/Linux: `export MOCK=1`.) Remove it for real designs. The first real run downloads the model (~2 GB).

---

## 🧭 Pages at a glance

| Page | What it is for |
|---|---|
| 🏠 **Dashboard** | Your overview: designs made, average rating, recent designs, how you rate designs, and service status. |
| 🎨 **Studio** | Scan your hand, choose polishes, and generate designs. |
| 🗂️ **Previous designs** | Every design you saved. Rate, filter (All / Rated / Unrated), view full screen, delete, and browse by page. **Book at a salon** sends a design to a company. |
| 📅 **My bookings** (client) | Designs you sent to a salon, their status, and the technician's rating and advice. |
| 💼 **Design reviews** (technician) | Bookings for your company. Set the status, rate if the design is worth it, and leave advice. |

Clients use pages under `#/client`, nail technicians under `#/tech`. Each account only sees its own area.

### 🔐 Accounts (Supabase)

1. Create a free project at <https://supabase.com>.
2. Open **SQL Editor**, paste [`supabase/schema.sql`](supabase/schema.sql), and run it. It creates the tables and the security rules. Accounts live in the app's own `credentials` table (passwords stored as bcrypt hashes), so Supabase Auth, its emails and its rate limits are not used. [`supabase/reset.sql`](supabase/reset.sql) removes everything again.
3. Copy **Project Settings → API** values into `frontend/.env`:
   ```sh
   VITE_SUPABASE_URL=https://your-project.supabase.co
   VITE_SUPABASE_ANON_KEY=your-anon-key
   ```
4. Sign up twice with a username and password: once as **Client**, once as **Nail technician** (type your salon's name and address). Each technician sign-up adds that salon to the list clients book from.

---

## 🎛️ Settings explained

| Setting | Where | What it does |
|---|---|---|
| `VITE_SD_URL` | `frontend/.env` | Address of the design service. |
| `VITE_SD_TOKEN` | `frontend/.env` | Optional password. Must match `API_TOKEN` on the server. |
| `GEN_BACKEND` | server | `cloudflare` (default) paints on Cloudflare Workers AI. `local` uses Stable Diffusion on the server itself. |
| `GEN_FALLBACK_LOCAL` | server | `true` = if Cloudflare fails for a design, paint that design locally instead. |
| `CF_ACCOUNT_ID`, `CF_API_TOKEN` | server | Cloudflare login for Workers AI. Keep them on the server only, never in `.env` of the app. |
| `MOCK=1` | server | Test mode, no Stable Diffusion. |
| `DEVICE` | server | Force `cuda`, `mps` or `cpu`. Normally leave it empty. |
| `MODEL_DIR` | server | Where the model is stored. |
| `ALLOWED_ORIGINS` | server | Websites allowed to use the server (comma separated). |

More examples are in `frontend/.env.example` and `backend/.env.example`.

---

## 🛠️ Troubleshooting

**🔴 "Design service offline" in the top bar**
The design service is not running. Start it (Step 2) and click the status pill to recheck.

**🔒 PowerShell says running scripts is disabled when I activate the venv**
Run this first, then activate again:
```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy RemoteSigned
```
Or skip activation and run Python directly:
```powershell
.\.venv\Scripts\python.exe -m uvicorn server:app --port 8000
```

**❓ `python server.py` does nothing**
`server.py` is a web app. Start it with `python -m uvicorn server:app --port 8000`.

**🐢 Designs take minutes**
That is normal on a CPU. A GPU is much faster. You can also pick **Quick** quality.

**🙈 My saved designs disappeared**
Designs are stored in your browser, and only the latest **30** are kept. Clearing the browser's site data removes them.

---

## 📱 Mobile app (Android, Ionic Capacitor)

NailVision is wrapped with [Capacitor](https://capacitorjs.com), Ionic's tool for turning a web app into a native app. The Android project lives in `frontend/android/`.

### 🧰 What to install

| # | Install | Why |
|---|---|---|
| 1 | 🟢 **Node.js 22+** (you already have it from [Quick start](#-quick-start)) | Builds the web app. |
| 2 | 🤖 **[Android Studio](https://developer.android.com/studio)**, latest version | Builds and runs the Android app. It also brings **Java (JDK 21)**, which Capacitor 8 needs. |
| 3 | 📦 **Android SDK** (inside Android Studio) | The parts listed below. |
| 4 | 📲 **Capacitor packages** | Already in `package.json`: `npm install` in `frontend/` installs them. |
| 5 | ⚡ **Ionic CLI** (optional) | Not needed for this project. Install it only if you want the `ionic` commands: `npm install -g @ionic/cli`. |

**Android SDK parts.** In Android Studio open **More Actions → SDK Manager** (or **Settings → Languages & Frameworks → Android SDK**):
- **SDK Platforms** tab: tick **Android 16 (API 36)**. The app targets API 36 and runs on Android 7 (API 24) or newer.
- **SDK Tools** tab: tick **Android SDK Build-Tools**, **Android SDK Platform-Tools**, **Android Emulator**, and **Android SDK Command-line Tools**.

**Tell your computer where the SDK is** (needed when you build from a terminal). On Windows PowerShell:

```powershell
[Environment]::SetEnvironmentVariable("ANDROID_HOME", "$env:LOCALAPPDATA\Android\Sdk", "User")
[Environment]::SetEnvironmentVariable("JAVA_HOME", "C:\Program Files\Android\Android Studio\jbr", "User")
```

Close and reopen the terminal, then check:

```powershell
& "$env:ANDROID_HOME\platform-tools\adb.exe" version
& "$env:JAVA_HOME\bin\java.exe" -version   # should say 21
```

(🍎 macOS: add `export ANDROID_HOME=$HOME/Library/Android/sdk` to `~/.zshrc`.)

> 🆕 Starting a new project from scratch instead of this one? These are the commands this project was set up with:
> `npm install @capacitor/core @capacitor/android` · `npm install -D @capacitor/cli` · `npx cap init NailVision com.nailvision.app --web-dir dist` · `npx cap add android`.
> You do **not** need to run them here.

### 📲 Get a phone or emulator ready
- **Real phone:** open **Settings → About phone** and tap **Build number** 7 times. Then in **Developer options** turn on **USB debugging** and plug the phone in. Accept the "Allow USB debugging?" prompt.
- **Emulator:** in Android Studio open **Device Manager → Create Virtual Device**, pick a phone, and choose an API 36 system image.

### ▶️ Build and run
1. Make sure `frontend/.env` points `VITE_SD_URL` at a public **https** design service. `localhost` on your PC is not reachable from the phone.
2. Run:
   ```sh
   cd frontend
   npm install
   npm run android
   ```
   It builds the web app, copies it into `frontend/android/` (`cap sync`), and opens Android Studio. Wait for Gradle to finish syncing, pick your device at the top, and press ▶ **Run**.
3. The first time you tap **Start scan**, the app asks for **camera** access. Tap **Allow**.

Run `npm run android` again after every code change. iOS needs a Mac with Xcode (`npm install @capacitor/ios && npx cap add ios`).

### 🛠️ Android troubleshooting
- **`SDK location not found`**: set `ANDROID_HOME` (above), or create `frontend/android/local.properties` with `sdk.dir=C:\\Users\\<you>\\AppData\\Local\\Android\\Sdk`.
- **`Unsupported class file major version` or a Java error**: Gradle is using the wrong Java. In Android Studio open **Settings → Build, Execution, Deployment → Build Tools → Gradle** and set **Gradle JDK** to the bundled **jbr-21**.
- **Camera does not open in the app**: open the phone's **Settings → Apps → NailVision → Permissions** and allow **Camera**.
- **Phone not listed**: check the USB cable carries data, and run `adb devices`. It should show your phone as `device`, not `unauthorized`.

---

## ☁️ Deploy

- 🐳 **Design service → Render (free plan works):** New → Web Service → connect the GitHub repo → **Root Directory** `backend`, **Language** Docker. The image has no torch, so it fits the free 512 MB. Environment variables: `CF_ACCOUNT_ID`, `CF_API_TOKEN`, `ALLOWED_ORIGINS` (see `backend/.env.example`). Leave `GEN_FALLBACK_LOCAL` unset there.
- ▲ **Frontend → Vercel:** set **Root Directory** to `frontend`, then set `VITE_SD_URL` to your Render address, and optionally `VITE_SD_TOKEN`. On Render, set `ALLOWED_ORIGINS` to your Vercel address.

> ⏱️ A free Render service sleeps after 15 minutes without visits. The first request then takes about a minute to wake it up.

---

Made with 💖 for nail lovers. Questions or ideas? Open an issue.
