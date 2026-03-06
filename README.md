# AquaFlow

AquaFlow is an end-to-end IoT system for monitoring water/tank conditions. It includes:

- **Firmware (device)** that reads sensors (humidity, temperature, **distance/tank level**) and publishes readings to the cloud
- **Backend API (server)** that authenticates users, receives IoT data (via AWS IoT Core), and stores it in MongoDB
- **Web app** dashboard for viewing data (and login/auth flows)
- **Mobile app** (work-in-progress) for on-the-go access

> Branch note: your repository’s default branch is **`master`** (not `main`).

---

## Architecture (how everything works)

### Data flow (high level)

1. **Device/Firmware** connects to Wi‑Fi.
2. Firmware connects to **AWS IoT Core** over MQTT (TLS certificates).
3. Firmware publishes sensor readings to an MQTT topic (example: `aquaflow/sensor/data`).
4. **Backend** connects/subscribes to AWS IoT Core, receives the sensor messages, and saves them into **MongoDB**.
5. **Web app / Mobile app** call the backend REST API to display latest readings and history.

### Expected IoT message format

The backend expects JSON messages like:

```json
{
  "humidity": 20.0,
  "temperature": 29.0,
  "distance": 23.0,
  "timestamp": "Thu Oct  2 00:18:03 2025\n"
}
```

- `distance` typically represents **tank level / distance sensor reading**
- `timestamp` is the device timestamp (string in your current implementation)

---

## Repository structure

At the repo root you have four main folders:

- `backend/` — Node.js + Express REST API, MongoDB (Mongoose), JWT authentication, AWS IoT Core ingestion
- `firmware/` — PlatformIO-based firmware project (ESP32-style workflow) that publishes data to AWS IoT Core
- `webapp/` — React (Vite) dashboard app (inside `webapp/web-app/`)
- `mobile/` — Mobile app project (currently minimal docs)

---

## Quickstart (local development)

### Prerequisites

- **Node.js 18+** (for backend + webapp)
- **MongoDB** (local or cloud)
- **AWS IoT Core** configured (for real device → cloud flow)
- PlatformIO environment (VS Code + PlatformIO) if you will build/upload firmware

---

## Backend (API) — `backend/`

### What it does

- User authentication (JWT)
- REST API endpoints
- MongoDB persistence
- AWS IoT Core integration to receive sensor readings and store them

### Install & run

```bash
cd backend
npm install
cp .env.example .env
npm run dev
```

Your test API base URL (as you specified):

- `http://localhost:5000/api`

### Environment variables

From `backend/README.md`, important variables include:

- `PORT` (default: `5000`)
- `MONGO_URI` (MongoDB connection string)
- `JWT_SECRET` (secret for signing tokens)
- `JWT_EXPIRES_IN` (token lifetime)
- `NODE_ENV` (development/production)

### AWS IoT Core settings (backend)

From `backend/IOT_README.md`:

```env
AWS_IOT_ENDPOINT=your-iot-endpoint.iot.us-east-1.amazonaws.com
AWS_IOT_TOPIC=aquaflow/sensor/data
```

It also references certificates placed in a `crt/` folder (ensure the backend code points at the correct cert path used in your repo/setup).

### IoT API endpoints (backend)

From `backend/IOT_README.md`:

- `GET /api/iot/latest`  
  Returns the most recently received sensor data.

- `GET /api/iot/all?page=1&limit=50`  
  Returns paginated list of all received sensor data.

- `GET /api/iot/status`  
  Returns AWS IoT Core connection status.

- `POST /api/iot/connect`  
  Manually trigger connection (testing).

### Auth endpoints (backend)

From `backend/README.md`:

- `POST /api/auth/register` — Register user
- `POST /api/auth/login` — Login user

---

## Firmware (Device) — `firmware/`

### What it does

- Connects to Wi‑Fi securely
- Connects to AWS IoT Core using MQTT over TLS (cert-based auth)
- Syncs time (NTP)
- Reads sensors and publishes:

  - humidity
  - temperature
  - **distance** (tank/level distance sensor reading)

### Typical setup steps

1. Open `firmware/` in VS Code with **PlatformIO** installed.
2. Check `platformio.ini` to confirm board/environment settings.
3. Update credentials:
   - Wi‑Fi SSID/password
   - AWS IoT endpoint
   - certificates/private key

Your firmware documentation mentions configuration headers like `config.h` and `secrets.h` (usually under `include/`).

### Uploading

Using PlatformIO:
- Build the project
- Upload to the board (USB)

(Exact commands depend on your PlatformIO environment; most commonly: “Build” and “Upload” buttons in PlatformIO, or `pio run` / `pio run -t upload`.)

---

## Web App (Dashboard) — `webapp/web-app/`

### What it is

A React + Vite + Tailwind CSS dashboard app that talks to the backend API.

### Run locally

```bash
cd webapp/web-app
npm install
npm run dev
```

Then open the dev server (commonly `http://localhost:3000` based on your webapp README).

### Backend API URL for the webapp

For local testing, set the API URL to your backend:

- `http://localhost:5000/api`

> Important: Vite typically expects environment variables prefixed with `VITE_` (example: `VITE_API_URL`).  
> Your repo currently also contains a `.env` using `REACT_APP_API_URL`. If something doesn’t work, align the env var name with what your webapp code actually reads.

---

## Mobile App — `mobile/`

The mobile folder contains a JavaScript/TypeScript mobile app project (likely Expo/React Native style based on `app.json` and structure). Its `mobile/README.md` currently only contains a Bolt link:

- `https://bolt.new/~/sb1-jchbigq3`

If you want a “full explanation” here too, the next step is to document:
- how to install dependencies (`npm install`)
- how to run (`npm run start` / `expo start`)
- how to configure the backend API base URL

---

## Deployment notes (current state)

- Your webapp `.env` points to a deployed backend:
  - `https://aqua-flow-backend.vercel.app/api`

So it looks like the backend is deployed (likely on Vercel). For local testing you will use:

- `http://localhost:5000/api`

---

## Security notes (important)

Your repo currently contains committed `.env` files (example: `backend/.env`, `webapp/web-app/.env`). This is risky because `.env` files often contain secrets.

Recommended best practice:
- keep `.env.example` committed
- keep real `.env` **untracked** (in `.gitignore`)
- rotate credentials if anything sensitive has already been pushed

---

## Component READMEs (existing)

- `backend/README.md` — backend setup + features
- `backend/IOT_README.md` — AWS IoT Core integration + IoT endpoints
- `firmware/readme.md` — firmware overview + setup
- `webapp/web-app/README.md` — webapp features + running instructions
- `mobile/README.md` — currently minimal

