# Sketchy Memory: Happy Hour Edition

A browser drawing game for the xFusion team's happy hour. Each round flashes a shape for two seconds, then the screen goes dark and you draw it from memory. The reveal lays your attempt over the real thing and gives it a precise score. Five rounds, five different shapes, one total on a shared leaderboard. Everyone thinks they can draw a square until they see a 61%.

Inspired by https://neal.fun/perfect-circle/ (credited at the bottom of the leaderboard panel). Sister game to Type Blaster and built the same way. For embedding in the handbook, see `handbook-integration.md`.

## Files

| File | Purpose |
|---|---|
| `sketchy-memory.html` | Source of truth: the whole game (HTML, CSS and JS in one file) |
| `index.html` | Identical copy of the above, used for deployment |
| `supabase-setup.sql` | Creates the shared `shape_scores` table (run once in Supabase → SQL Editor) |
| `supabase-accuracy-decimal.sql` | Switches `acc` to one decimal (score ÷ 500) on a table made with the first setup script |
| `handbook-integration.md` | Guide for embedding in the handbook and passing the Slack name and photo (same message contract as Type Blaster) |

After editing `sketchy-memory.html`, copy it over `index.html`.

## Hosting

GitHub (`reggierendal/sketchy-memory`, files uploaded manually) → Cloudflare Pages (`*.pages.dev`). Uploading a new `index.html` to GitHub redeploys the site. Hard-refresh (Ctrl+Shift+R) to see changes.

Shared data lives in Sketchy Memory's own Supabase project (`mzncpsdvkfgxzcjujnge`), separate from Type Blaster's:
- Table `shape_scores` for the leaderboard (live updates). Each finished game stores name, photo, ring colour, `score` (out of 500), `acc` (accuracy %), the score per shape, and the best drawing.
- Realtime presence channel `happy-hour-shapes` for "Playing now" (nothing stored).

The project URL and publishable key are already in the game file. The key is safe to publish; row-level security allows only reading and adding scores.

If the table hasn't been created yet, the game still works: the Playing now panel shows "Setup needed" and scores are saved in that browser only. If the file is opened without the Supabase settings (for example the claude.ai preview link), it runs in local mode: scores stay in that browser, and Playing now only shows other tabs on the same computer.

## Gameplay rules

- A game is 5 rounds, one for each active shape (straight line, circle, square, heart, infinity), in a shuffled order.
- Every round has three steps:
  1. **Peek:** the target shape appears for 2 seconds, at a random size, position and rotation.
  2. **Dark:** the screen goes dark and the shape disappears. Draw it from memory in **one continuous stroke**: press down, draw the whole shape, and only lift the mouse button (or finger) when you are done. Lifting ends the drawing, so you cannot pick up and add more strokes. Your stroke shows as a plain white line, with no score or colour hints while you draw.
  3. **Reveal:** the real shape fades back in under your drawing. Your stroke turns red to green along its length to show where you drifted, and the round score counts up.
- Scoring compares your stroke with the exact shape you were shown, including its size, position and rotation, so remembering where it was matters as much as drawing it well. The score also checks that you covered the whole shape, so drawing half of it neatly doesn't win.
- Game score is the total of the 5 rounds, out of 500 (for example 412.6).
- Accuracy is how close your drawings were: your score as a percentage of 500, to one decimal. For example, 375.4 / 500 is 75.1%. During a game it is the average of the rounds so far. Redraws don't affect it.
- A drawing is rejected, with no penalty, and you get a fresh peek and redraw when it is:
  - Too short (usually from lifting the mouse before finishing) or too small.
  - Too slow (over 10 seconds).
  - Not closed, for closed shapes (the end is too far from the start).
- One counted attempt per round. Rejected drawings don't count towards the score.
- Switching windows mid-round restarts that round with a fresh peek.
- Choose **Play for Score** or **Practice** each time. Practice is marked "(recommended)" on the start screen for first-timers. Practice lets you pick any shape, can turn the dark step off, and is never saved. Scores are only saved when all 5 rounds are done.
- The results screen shows all 5 reveals side by side, ready for the shared screen.
- The start screen has a "How to play" rulebook with these rules, including the one-stroke rule.
- After **Play for Score**, a "Quick tips before you start" screen reminds players to memorise the shape (size, position, angle) and to draw in one continuous stroke, with **I'm ready, let's draw**, **Practice first** and **Back** buttons. **Play again** skips it.
- Everyone gets the same 5 shapes each day (`SHAPE_MODE = 'event'`). Set it to `'random'` for a new 5 each game.

## Sound

After each drawing there is a short pencil-lift swish, then a chime whose notes depend on the score: 90+ plays a rising four-note run, 75+ three notes, 50+ two notes, and lower scores a friendly falling "boop". Rejected drawings get a soft double blip. All sounds are generated in the browser (Web Audio), with no audio files and nothing copied from neal.fun. The 🔊 button at the bottom left of the play area mutes it, and the choice is remembered.

## Shapes

The first batch uses 5 shapes. Star and spiral are built but switched off; to bring them back (or add new ones), edit `ACTIVE_SHAPES` near the top of the shapes section in the game file.

| Shape | Type | Notes |
|---|---|---|
| Straight line | Open | Angle and length both count |
| Circle | Closed | |
| Square | Closed | Rectangles score lower |
| Heart | Closed | Point at the bottom |
| Star | Closed | 5 points, one stroke. **Parked for a later batch** |
| Infinity | Closed | Must cross in the middle |
| Spiral | Open | Number of turns counts. **Parked for a later batch** |

## Players

- Name and photo are required. Names show as "Reggie" (first letter capital).
- The photo auto-centres on the face and can be zoomed or dragged (same code as Type Blaster). It shows on the reveal card, the leaderboard and Playing now.
- Ring colour is assigned at random (no picker) and kept for the player's next visit. It frames their photo everywhere it appears.
- Play for Score and Practice stay greyed out until a name and photo are added. Hovering them shows a tooltip saying what's missing ("Upload a photo first", "Add your name first", or both) and highlights the Upload photo button or name field.
- Hovering a photo or name in the Leaderboard, Shape champions or Playing now shows a larger photo card (112px) with the name. It closes as soon as the pointer moves away and never opens while a game is in progress.
- Hovering the score, accuracy or best-shape cells of a leaderboard row shows that player's best drawing.

## Leaderboard and Playing now

- Leaderboard: same table as Type Blaster. Top 15 games with medals for 1st to 3rd, and columns #, photo, Name, Score, Acc and Best (icon of the best shape; hover a row to see that drawing). It updates live. No sample players: it starts empty ("No scores yet — be the first!").
- Shape champions: one row above the leaderboard with the best round score ever for each active shape and who drew it.
- Playing now: same pattern as Type Blaster. White rows with photo, name, a pulsing live dot and "Round 3/5 · Heart · 294.3 · 80%" (or "Practice · Heart"), sorted by running total. Rows update in place, a heartbeat is sent every 2.5 seconds, and the channel reconnects by itself if it drops. Only a 56px photo thumbnail is shared. People leave the list when they disconnect, refresh, quit or finish.

## Handbook integration

Works exactly like Type Blaster: the handbook embeds the game in an iframe, the game sends `happyhour:ready`, and the handbook replies with `happyhour:profile` (name and Slack photo). The handbook owner adds a second page (Happy Hour → Sketchy Memory) with the same embed code and this game's address. Add the handbook's address to `HANDBOOK_ORIGINS` in the game file first. Details: `handbook-integration.md`.

## Still to do

- Run `supabase-accuracy-decimal.sql` once in Supabase → SQL Editor so `acc` stores one decimal (the game still saves a whole number until then).
- Create the GitHub repo and the Cloudflare Pages project.
- Send the new game address to the handbook owner for the second embed page.
