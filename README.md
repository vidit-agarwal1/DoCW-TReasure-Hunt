# Treasure Hunt Leaderboard

A single-file scoreboard for your event:

- **Public leaderboard** — anyone with the link can view live standings. No login needed.
- **Committee console** (`yoursite.com/#admin`) — password-protected screen where organisers add teams and award points.
- Quick point buttons for **+50 / +100 / +150** (easy / medium / hard tasks), plus a custom **±points** box for anything else, and a one-tap **Undo** if someone mis-clicks.
- Built to handle ~75 teams: search boxes on both the public board and the console, and a **bulk add** box so you can paste all your team names in one go instead of adding them one by one.
- Works out of the box in **demo mode** (scores saved only in your browser) so you can try it immediately — then takes about 5 minutes to connect to a free Firebase database so scores sync live to every device.

---

## 1. Try it immediately (demo mode)

Just open `index.html` in a browser — it works right away, no setup. Add a few teams from the Committee tab (default password: `treasure2026`) and try the point buttons. In this mode, data only lives in that one browser/device, so it's for testing only — do this step before the event, not during it.

## 2. Make it live for everyone (~5 minutes, free)

This uses **Firebase Firestore**, Google's free real-time database. It requires no coding — just copy some keys.

1. Go to [console.firebase.google.com](https://console.firebase.google.com) and click **Add project**. Name it anything (e.g. "treasure-hunt-2026"). You can skip Google Analytics.
2. In the left sidebar, click **Build → Firestore Database → Create database**. Choose **Start in test mode** for now (we'll tighten this in step 4), pick any region, and click Enable.
3. Click the gear icon (⚙) → **Project settings**. Under "Your apps", click the **</> (web)** icon to register a new web app (any nickname is fine). Firebase will show you a `firebaseConfig` object like this:

   ```js
   const firebaseConfig = {
     apiKey: "AIza...",
     authDomain: "treasure-hunt-2026.firebaseapp.com",
     projectId: "treasure-hunt-2026",
     storageBucket: "treasure-hunt-2026.appspot.com",
     messagingSenderId: "...",
     appId: "..."
   };
   ```

4. Open `index.html` in a text editor, find the `firebaseConfig` block near the top of the `<script>` section, and paste your values in over the `"REPLACE_ME"` placeholders. Save the file.
5. Back in the Firebase console, go to **Firestore Database → Rules** and replace the default rules with:

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /teams/{teamId} {
         allow read: if true;
         allow write: if request.resource.data.keys().hasOnly(['name', 'score', 'createdAt'])
                      && request.resource.data.name is string
                      && request.resource.data.score is number;
       }
     }
   }
   ```

   This keeps the leaderboard publicly viewable, while at least requiring writes to look like valid team data.

That's it — reload the page and the status pill in the header should switch from "Demo mode" to "Live".

## 3. Set your committee password

Open `index.html`, find this line near the top of the script:

```js
const ADMIN_PASSWORD = "treasure2026";
```

Change it to whatever you want, save, and re-publish the file. Share that password only with your organising committee.

> **A note on security:** this is a simple, friendly lock — good enough to stop curious participants from wandering into the scoring console. It is not the same as a real login system, because a static site with no backend can't fully hide anything in its own source code. For a casual event scoreboard this is normally a fine trade-off. If you need stronger protection (e.g. this is a large public leaderboard with prize money on the line), consider adding Firebase Authentication, which is a bigger but more robust setup.

## 4. Host it on GitHub Pages (free)

1. Create a new GitHub repository (public or private both work for Pages on a free personal account, though private repos need GitHub Pro/Team/Enterprise for Pages).
2. Upload `index.html` (and this README, optional) to the repo — either drag-and-drop on the GitHub website, or:
   ```bash
   git init
   git add index.html README.md
   git commit -m "Treasure hunt leaderboard"
   git branch -M main
   git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO.git
   git push -u origin main
   ```
3. In the repo, go to **Settings → Pages**. Under "Build and deployment", set **Source** to "Deploy from a branch", pick branch `main` and folder `/ (root)`, then Save.
4. After a minute or two, your site will be live at:
   `https://YOUR_USERNAME.github.io/YOUR_REPO/`

Share that link with all teams for the public leaderboard, and `https://YOUR_USERNAME.github.io/YOUR_REPO/#admin` privately with your committee.

## 5. Running the event

- **Add all teams upfront:** Committee tab → paste all team names (one per line) into the bulk box → "Add all from list".
- **Award points:** find a team in the search box, tap +50 / +100 / +150 for the task difficulty, or type a custom number (positive or negative — you can also dock points) and hit Apply.
- **Made a mistake?** A toast appears after every change with an **Undo** link for a few seconds.
- **Removing a team:** the ✕ button on its row in the console (asks for confirmation first).

## Customizing

- **Event name:** edit the `<h1 id="eventTitle">Treasure Hunt</h1>` line in `index.html`.
- **Point values:** the +50/+100/+150 buttons are set in the `renderAdminList()` function — search for `data-delta="50"` etc. to change the numbers.
- **Colors/fonts:** all defined as CSS variables at the top of the `<style>` block (`--jungle`, `--gold`, `--parchment`, etc.).
