# Lattice + Firebase plugin store

Files in this folder:

| File | Purpose |
|---|---|
| `index.html` | The Lattice notes app (same as before, plus icon-set plugins and a Plugin store link) |
| `store.html` | The plugin store, in a single file: Firebase setup, Google sign-in, live catalog, publish, install |
| `firebase-config.example.js` | Optional: copy to `firebase-config.js` and fill in your config object |
| `firestore.rules` | Security rules to publish in your Firebase project |
| `firebase.json` | Ready for `firebase deploy` (hosting + rules) |

## Important: where this can run

This must be served from **your own https address (or http://localhost)**. It will not work on a claude.ai published page, which blocks requests to Firebase and Google sign-in. Notes and the store must be on the **same origin** (same domain), because the store hands plugins to the notes app through the browser's shared storage.

## Setup

1. **Create a Firebase project** at https://console.firebase.google.com.
2. **Add a web app** (Project settings > Your apps > `</>`). Copy the `firebaseConfig` object it shows.
3. **Authentication**: Build > Authentication > Get started > Sign-in method > enable **Google**.
4. **Authorized domains**: Authentication > Settings > Authorized domains. `localhost` is there by default. Add your hosting domain if it is not.
5. **Firestore**: Build > Firestore Database > Create database (production mode).
6. **Rules**: paste the contents of `firestore.rules` into Firestore > Rules and publish (or run `firebase deploy --only firestore:rules`).
7. **Host the folder**
   - Quick local test: run `python3 -m http.server 8000` in this folder, then open http://localhost:8000/store.html
   - Firebase Hosting: `npm i -g firebase-tools`, `firebase login`, `firebase init` (reuse existing files), `firebase deploy`
8. **Give Firebase your config**: open `store.html`. The first screen asks for the config object. Paste it exactly as the console shows it, then press **Save and connect**. (Alternative: copy `firebase-config.example.js` to `firebase-config.js` and fill it in.)
9. **Make yourself an admin**: sign in once, copy your UID from Authentication > Users, then in Firestore create collection `admins` with a document whose ID is that UID (any field). A **Review** tab then appears for you.

## How it works

- **Themes, fonts, icon sets and highlighters** are JSON or YAML data. They publish instantly and cannot run code.
- **Scripts** (`.js` plugins of any type) are saved as `pending` and appear in the store only after an admin approves them in the Review tab. Editing an approved script sends it back to review.
- **Install** copies the plugin into the shared install list. Lattice picks it up on load, and live if it is already open in another tab. Installed features start switched off.
- Each plugin stores a SHA-256 fingerprint. The store verifies it before installing.
- The plugin id is the Firestore document id, so ids are unique and only the original author can update a plugin.

## Troubleshooting

- **Nothing but the header shows:** a red banner now explains any error. Make sure you opened the file through `http://localhost` or your https address, not by double-clicking it.
- **The setup box shows but saving does nothing:** paste the whole object including the braces. The error text appears under the box.
- **"Could not load ...gstatic.com":** the browser cannot reach Google's servers. Check your connection or any content blocker.
- **"Firestore denied the request":** publish `firestore.rules` (step 6).
- **The pop-up closes straight away:** add your domain under Authentication > Settings > Authorized domains.
- To change projects later, press **Firebase** in the store header.

## Notes

- The Firebase `apiKey` is not a secret. The rules are what protect your data.
- Rules cannot inspect script contents. Read scripts before approving them, since they run with full access to a user's notes.
- Installs are local to each browser. There are no install counts or ratings yet.
- I could not test against a live Firebase project from here. If a step fails, the store shows the Firebase error text.
