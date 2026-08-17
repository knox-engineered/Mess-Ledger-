# Mess Ledger — setup guide

This app installs on Android like a native app. Anyone can create a new,
independent mess — each one gets its own unique **Mess ID** and its own
private set of members, meals, guest meals, fund contributions and
expenses. Other people join a specific mess by entering its ID, then log
in with their name and PIN. Everything syncs in real time using a free
Firebase project as the backend.

There are two one-time steps: **(A)** connect your own free Firebase
project, and **(B)** host the files somewhere with a real web address so
Chrome will let people install it. Neither needs any coding.

---

## Part A — Create your free Firebase project (~5 minutes)

1. Go to **https://console.firebase.google.com** and sign in with any
   Google account. Click **Add project**, give it any name (e.g.
   `mess-ledger`), and finish creation. No credit card is required — this
   uses Firebase's free "Spark" plan.

2. On the project's home screen, click the **web icon (`</>`)** to register
   a web app. Give it a nickname (e.g. `mess-ledger-web`) and click
   **Register app**. Firebase will show you a code block that looks like:

   ```js
   const firebaseConfig = {
     apiKey: "AIzaSy...",
     authDomain: "mess-ledger-xxxxx.firebaseapp.com",
     projectId: "mess-ledger-xxxxx",
     storageBucket: "mess-ledger-xxxxx.appspot.com",
     messagingSenderId: "123456789",
     appId: "1:123456789:web:abcdef"
   };
   ```

   Keep this tab open — you'll copy these six values into `index.html` in
   step 5.

3. In the left sidebar, go to **Build → Authentication → Get started**.
   Under the **Sign-in method** tab, click **Anonymous**, toggle it
   **Enable**, and **Save**. (This lets each phone connect without anyone
   typing an email or password — the app has its own simple name+PIN
   login on top of this, per mess.)

4. In the left sidebar, go to **Build → Firestore Database → Create
   database**. Pick a location close to you (e.g. `asia-south1` for
   India), choose **Start in production mode**, and click **Create**.

   Once created, open the **Rules** tab and replace everything with:

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /messes/{messId} {
         allow get: if request.auth != null;
         allow list: if false;
         allow create: if request.auth != null;
         allow update, delete: if request.auth != null;

         match /{document=**} {
           allow read, write: if request.auth != null;
         }
       }
     }
   }
   ```

   Click **Publish**.

   (`allow list: if false` stops anyone from browsing every mess that
   exists in your project — a mess can only be opened by someone who
   already has its exact ID.)

5. Open `index.html` from this folder in any text editor (Notepad, VS
   Code, etc.). Near the top of the `<script type="module">` block, find:

   ```js
   const firebaseConfig = {
     apiKey: "YOUR_API_KEY",
     ...
   };
   ```

   Replace all six placeholder values with the ones from step 2, then
   save the file.

That's it for Firebase — the free tier (50,000 reads and 20,000 writes a
day) is far more than any number of small messes will ever use.

---

## Part B — Host it and install it on Android (~2 minutes)

Chrome will only offer to "install" a web app if it's served over
**https://**, not opened as a local file — so the folder needs a home on
the web. The easiest free option, no account needed:

1. Go to **https://app.netlify.com/drop** on a computer.
2. Drag the whole `mess-ledger-app` folder (with the now-edited
   `index.html`, plus `manifest.json`, `service-worker.js`, and the two
   icon files) onto the page.
3. Netlify instantly gives you a live address like
   `https://random-name-123.netlify.app`. That's the app's link — share
   it with anyone who might want to create or join a mess.
   - Optional: create a free Netlify account afterwards to keep this
     link permanently and give it a custom name.
   - Alternative: GitHub Pages works too if you'd rather use a GitHub
     repo — just enable Pages in the repo settings.

4. Each person opens the link in **Chrome on Android**, taps the **⋮**
   menu in the top right, and taps **"Add to Home screen"** (or Chrome
   may show an **"Install app"** banner automatically). This adds a real
   app icon that opens full-screen, with no browser bar.

---

## How a mess works

- **Creating a mess**: the first person taps **"Create a new mess"**,
  enters a mess name and everyone's name (with an optional 4-digit PIN
  each). The app generates a unique **6-character Mess ID** (e.g.
  `ALPHA7`) and shows it on screen with a copy button — share this with
  the mess's members however you like (WhatsApp, etc.).
- **Joining a mess**: everyone else opens the same app link, taps
  **"Join this mess"**, and types in the Mess ID they were given. They'll
  then see the member list for that specific mess and log in by tapping
  their name (and PIN, if one was set).
- Each mess's data — meals, guest meals, fund contributions, expenses —
  is completely separate from every other mess created with this app,
  and updates live on every phone in that mess.
- The app also works offline: it queues your changes and syncs them
  automatically once you're back online.
- **Switching or leaving**: the menu next to your name lets you switch
  which member you're logged in as (same mess), or leave the mess
  entirely on that device (the mess and its data aren't affected —
  anyone can rejoin later with the ID). Settings → Danger zone also has
  an option to permanently delete a mess and free up its ID.

## A note on privacy

A mess's data is private in the sense that it's only reachable by
someone who has its exact Mess ID — Firestore is configured to refuse
any request to list or browse messes, so an ID can't be guessed by
browsing. PINs work the same way: a light way to pick the right name on
a shared mess, not a password in the cryptographic sense.

This is **not end-to-end encryption** — Firestore itself can still read
the underlying data (as could you, from the Firebase console), and
anyone who obtains a mess's ID could technically read or write to it.
For a small trusted group tracking meal expenses, this level of privacy
is normally more than enough; it's the same model many "anyone with the
link" tools use. If you need stronger guarantees than that, this app
isn't the right fit as-is.

## Troubleshooting

- **"Couldn't connect" screen** → double-check Anonymous sign-in is
  enabled and the Firestore rules above are published.
- **"Mess ID not found"** → double-check the ID exactly as shared; it's
  case-insensitive but must otherwise match exactly.
- **No install prompt on Android** → make sure you're using the
  `https://` link (not a local file), and that you're on Chrome.
- **Changes not appearing on another phone** → check that phone has an
  internet connection; offline changes sync once it reconnects.
