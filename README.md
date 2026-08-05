# Campus Exchange Hub

> A real-time campus exchange platform for students — built with vanilla HTML/CSS/JS and Firebase.

**Campus Exchange Hub** lets students post and browse campus transfer listings in seconds. No account required: just fill in your name, campuses, and phone number — your listing goes live instantly, visible to anyone looking for a match.

---

## Features

- **Publish a listing** — name, current campus, desired campus, phone number
- **Real-time updates** — listings appear and disappear without refreshing the page
- **Filter by campus** — quickly find people coming from the campus you want
- **Secure deletion** — each listing is protected by a personal security code; only the creator can delete it
- **Anonymous authentication** — Firebase Auth silently assigns a `uid` to each visitor, stored as `ownerUid` for ownership checks
- **Firestore security rules** — write operations are validated server-side; no raw open access

---

## Project structure

```
Campus-Exchange-Hub/
├── index.html            # Main page (UI + Firebase logic)
├── config.example.js     # Firebase config template (copy → config.js)
├── config.js             # Your local config — gitignored, never committed
├── firestore.rules       # Recommended Firestore security rules
└── logo.png              # App logo
```

---

## Getting started

### Prerequisites

- A Firebase project with **Firestore** and **Anonymous Authentication** enabled
- A local HTTP server (e.g. the VS Code [Live Server](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer) extension)

### Local setup

1. Clone the repository:
   ```bash
   git clone https://github.com/Ziad-Yousfi/Campus-Exchange-Hub.git
   cd Campus-Exchange-Hub
   ```

2. Copy the config template and fill in your Firebase credentials:
   ```bash
   cp config.example.js config.js
   ```
   ```js
   // config.js  —  never committed (see .gitignore)
   window.firebaseConfig = {
     apiKey: "YOUR_API_KEY",
     authDomain: "your-project.firebaseapp.com",
     projectId: "your-project",
     storageBucket: "your-project.appspot.com",
     messagingSenderId: "000000000000",
     appId: "1:000000000000:web:xxxxxxxxxxxx",
     measurementId: "G-XXXXXXXXXX"
   };
   ```

3. Open `index.html` with Live Server (right-click → *Open with Live Server*) and navigate to `http://127.0.0.1:5500`.

> **Note:** The Firebase Web `apiKey` is a *public identifier*, not a secret. Real security is enforced by Firestore Rules and Auth — not by hiding this key.

---

## Firestore security rules

Apply the rules in `firestore.rules` to your project:

- **Read** — public, no authentication required
- **Create** — requires a valid Firebase Auth session; all required fields must be present and valid
- **Update / Delete** — restricted to the original creator (`ownerUid == request.auth.uid`)

**To deploy:**

```bash
firebase deploy --only firestore:rules
```

Or paste the content directly into **Firebase Console → Firestore → Rules**.

---

## Authentication

The app uses **Firebase Anonymous Authentication** to silently assign a unique `uid` to each visitor. This `uid` is stored in the `ownerUid` field of each listing, enabling ownership-based access control without requiring users to create an account.

Enable it in **Firebase Console → Authentication → Sign-in method → Anonymous**.

---

## Deployment on GitHub Pages

The project is designed to work on GitHub Pages with a CI pipeline that injects the Firebase config from GitHub Secrets — so `config.js` is never stored in the repository.

**1. Add your credentials to GitHub Secrets** (Settings → Secrets and variables → Actions):

| Secret name | Value |
|---|---|
| `FIREBASE_API_KEY` | Your API key |
| `FIREBASE_AUTH_DOMAIN` | `your-project.firebaseapp.com` |
| `FIREBASE_PROJECT_ID` | `your-project` |
| `FIREBASE_STORAGE_BUCKET` | `your-project.appspot.com` |
| `FIREBASE_MESSAGING_SENDER_ID` | Your sender ID |
| `FIREBASE_APP_ID` | Your app ID |
| `FIREBASE_MEASUREMENT_ID` | `G-XXXXXXXXXX` |

**2. Create the workflow** at `.github/workflows/deploy.yml`:

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Inject Firebase config from secrets
        run: |
          cat > config.js << EOF
          window.firebaseConfig = {
            apiKey: "${{ secrets.FIREBASE_API_KEY }}",
            authDomain: "${{ secrets.FIREBASE_AUTH_DOMAIN }}",
            projectId: "${{ secrets.FIREBASE_PROJECT_ID }}",
            storageBucket: "${{ secrets.FIREBASE_STORAGE_BUCKET }}",
            messagingSenderId: "${{ secrets.FIREBASE_MESSAGING_SENDER_ID }}",
            appId: "${{ secrets.FIREBASE_APP_ID }}",
            measurementId: "${{ secrets.FIREBASE_MEASUREMENT_ID }}"
          };
          EOF

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./
```

**3. Push to `main`** — GitHub Actions will build and deploy automatically.

---

## Security checklist

- [x] `config.js` is in `.gitignore` — never committed
- [x] Firestore Rules enforce authentication and field validation on every write
- [x] Anonymous Auth prevents unauthenticated writes
- [ ] (Optional) Enable [Firebase App Check](https://firebase.google.com/docs/app-check) (reCAPTCHA v3) to block automated abuse
- [ ] (Optional) Restrict the API key to your domain in [Google Cloud Console → APIs & Services → Credentials](https://console.cloud.google.com/apis/credentials)
- [ ] (Optional) Set up Firebase budget alerts to monitor unexpected usage

---

## Contributing

Contributions are welcome. Fork the repo, create a feature branch, and open a pull request. For significant changes (moderation system, backend, etc.), please open an issue first to discuss the approach.

---

## License

[MIT](LICENSE) © 2025 Ziad Yousfi
