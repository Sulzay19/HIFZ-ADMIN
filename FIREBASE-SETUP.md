# Connecting the parent portal to the Admissions tab

Both `hifz-school-admin.html` and `parent-portal.html` are still local, browser-only
apps — the only thing that needs to travel between a parent's device and yours is
the application itself. That's what Firebase (a free Google service) is for here:
it's just a shared inbox in the cloud for applications. Nothing else about your
system changes — Students, Classes, Academic Progress, etc. all still live only in
your browser's storage, same as before.

## 1. Create the Firebase project (5 minutes, free)

1. Go to https://console.firebase.google.com and sign in with a Google account.
2. **Add project** → give it a name (e.g. "hifz-admissions") → you can skip
   Google Analytics → **Create project**.
3. In the left sidebar: **Build → Firestore Database → Create database**.
   - Choose a location close to you.
   - Start in **production mode** (we'll paste in our own rules below).
4. Still in the left sidebar: click the **gear icon → Project settings**.
   Scroll to **Your apps**, click the **`</>`** (web) icon, give it a nickname,
   and **Register app**. You'll get a code block that looks like:

   ```js
   const firebaseConfig = {
     apiKey: "AIza...",
     authDomain: "hifz-admissions.firebaseapp.com",
     projectId: "hifz-admissions",
     storageBucket: "hifz-admissions.appspot.com",
     messagingSenderId: "...",
     appId: "..."
   };
   ```

## 2. Paste the config into both files

Open `hifz-school-admin.html` and `parent-portal.html` — near the top of each
you'll find:

```js
const FIREBASE_CONFIG = {
  apiKey: "PASTE_ME",
  ...
};
```

Replace it with the object Firebase gave you (same values in both files).

## 3. Set Firestore security rules

In the Firebase console: **Build → Firestore Database → Rules**, replace the
contents with:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /applications/{appId} {
      allow create: if true;   // anyone can submit an application
      allow read, update: if true;  // anyone with the link can read/mark applications
      allow delete: if false;  // nobody can delete from the client
    }
  }
}
```

**Important caveat:** with `allow read: if true`, anyone who discovers your
Firestore project ID could technically read the applications collection (names,
phone numbers, etc.) — there's no login system in this app yet. For a first
version / small pilot this is a reasonable trade-off (it's still far more
private than a public Google Sheet), but if you're handling this long-term we'd
recommend adding Firebase Authentication (email/password) for staff and
tightening `allow read, update` to `if request.auth != null`. Happy to add that
next if you want it.

## 4. Host the two files somewhere reachable

- `hifz-school-admin.html` — keep this for staff only (don't publish the link).
- `parent-portal.html` — this is the one to share with parents. You can:
  - Email/WhatsApp the file directly, or
  - Put it on any free static host (Firebase Hosting itself, GitHub Pages,
    Netlify) and share the URL.

## 5. Try it

1. Open `parent-portal.html`, fill in a test application, submit.
2. Open `hifz-school-admin.html` → **Admissions → Applications → Refresh**.
   Your test application should appear.
3. Click **Start Evaluation** to pull it into **Candidates**, score it, and
   (if accepted) use **Enroll as Student** to add them to the real Students list.
