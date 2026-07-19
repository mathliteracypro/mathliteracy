# 🛠️ Developer Setup & Installation Guide

This guide walks you through setting up a complete local development environment for the MathLiteracy Pro platform.

---

## 📋 Prerequisites

Before proceeding, ensure you have the following software installed:

- **Node.js**: Version 18.x or 20.x (Recommended)
- **npm**: Version 9.x or higher
- **Git**: For version control
- **Docker**: Optional, for container-based builds

---

## 🚀 Step-by-Step Local Setup

### 1. Repository Setup
Clone the platform code and navigate into the workspace root directory:

```bash
git clone https://github.com/mathliteracypro/platform.git
cd platform
```

### 2. Configure Environment Variables
Copy the example environment template into a local `.env` configuration file:

```bash
cp .env.example .env
```

Open `.env` in your text editor and populate the required API keys and Firestore credentials:

```env
# ==============================================================================
# 🔥 FIREBASE SERVICES CONFIGURATION
# ==============================================================================
VITE_FIREBASE_API_KEY=AIzaSyA1...
VITE_FIREBASE_AUTH_DOMAIN=mathlit-pro.firebaseapp.com
VITE_FIREBASE_PROJECT_ID=mathlit-pro
VITE_FIREBASE_STORAGE_BUCKET=mathlit-pro.appspot.com
VITE_FIREBASE_MESSAGING_SENDER_ID=2891...
VITE_FIREBASE_APP_ID=1:2891...

# ==============================================================================
# 🧠 SERVERSIDE API SECRETS (NEVER EXPOSE TO THE BROWSER)
# ==============================================================================
GEMINI_API_KEY=AIzaSyB9...
PORT=3000
NODE_ENV=development
```

### 3. Install Dependencies
Install all core NPM packages and build dependencies:

```bash
npm install
```

### 4. Fire Up the Development Server
Launch the unified Express + Vite dev server. It is strictly configured to serve the app on port `3000`:

```bash
npm run dev
```

Your terminal should display local server parameters:
```text
> tsx server.ts
Server running on port 3000
Vite dev server mounted on http://localhost:3000
```

Open `http://localhost:3000` in your browser to view your live workspace.

---

## 📦 Script Commands Reference

| Script Command | Purpose / Action |
| :--- | :--- |
| `npm run dev` | Boots up the development server with hot reload capabilities |
| `npm run lint` | Runs the linter rules (`tsc --noEmit`) to verify TS files have zero bugs |
| `npm run build` | Compiles public static assets and bundles the backend into `/dist` |
| `npm run start` | Launches the production-ready server directly from `/dist` |
