# Vibe Coding for HR workshop site

Static site: `index.html` plus `slides/01.jpg` to `slides/12.jpg`. No build step.

## 1. Publish on GitHub Pages

1. Create a new public repository on GitHub, for example `vibe-coding-hr`.
2. Upload the contents of this folder (`index.html`, `README.md` and the `slides` folder) to the repository root.
3. In the repository, open Settings, then Pages. Under "Build and deployment" choose "Deploy from a branch", branch `main`, folder `/ (root)`, and save.
4. After a minute the site is live at `https://<your-user>.github.io/vibe-coding-hr/`.

Until step 2 below is done, the site runs in demo mode: the slide viewer works, and submitted cases are saved in each visitor's own browser only, so people do not see each other's cases.

## 2. Share cases between all visitors (Firebase Realtime Database, free tier)

GitHub Pages only serves static files, so shared cases need a small database. This site talks to Firebase Realtime Database over plain HTTPS, with no SDK and no sign-in.

1. Go to https://console.firebase.google.com and create a project (Google Analytics is not needed).
2. Open Build, then Realtime Database, then Create Database. Pick a location near your audience and start in locked mode.
3. Open the Rules tab, paste the rules below, and Publish.
4. Copy the database URL shown at the top of the Data tab, for example `https://vibe-coding-hr-default-rtdb.europe-west1.firebasedatabase.app`.
5. In `index.html`, set `var DB_URL = "<that URL>";` near the top of the script, commit, and wait for Pages to redeploy.

Rules: anyone can read the list and add a case, nobody can change or delete one from the website, and each case must be 8 to 200 characters.

```json
{
  "rules": {
    "cases": {
      ".read": true,
      ".write": false,
      "$id": {
        ".write": "!data.exists()",
        ".validate": "newData.hasChildren(['t','ts']) && newData.child('t').isString() && newData.child('t').val().length >= 8 && newData.child('t').val().length <= 200 && newData.child('ts').isNumber()",
        "t": {},
        "ts": {},
        "$other": { ".validate": false }
      }
    }
  }
}
```

To clear test cases or remove an inappropriate one, delete the entry under `cases` in the Firebase console (Data tab).

## Notes

- Cases are public to anyone who has the link. Tell attendees not to include names or confidential details.
- The page checks for new cases every 4 seconds (`POLL_MS` in `index.html`).
- Anyone can send requests to the database URL directly, so this setup suits a workshop, not a long-lived public page. After the event you can set `".write": false` on `cases` to freeze it.
- The slide images were exported from the Gamma PDF. To update the deck, replace the files in `slides/` (same names, JPEG).
