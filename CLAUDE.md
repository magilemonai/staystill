# STAYSTILL

A comedy racing game built with Cody on 2026-04-29 → 2026-05-11. Premise: a full Mario Kart 64-style racing game where **the first one to move loses**. Eight racers line up, the lights go 3-2-1-GO, and everyone sits perfectly still while phones ring and texts pile up. Triumphant music plays the entire time.

## The twist (added 2026-07-01) — spoilers, this is dev doc

The game no longer ends when you outlast the field. Outlasting everyone is where it *starts*. It escalates through five acts and three genre shifts, each a reveal:

1. **Act 1 — the race you signed up for.** ~88s. AIs crack one by one, phone notifications pile up. You think winning = being last one standing.
2. **REVEAL 1 (~96s) — "LAP 2 OF 3."** The finish banner drops like the race is ending, then jerks back up and relabels itself "LAP 1 ✓". The DNF'd rivals quietly slide back onto the grid — nobody actually left, they were all just sitting still too. Sky goes to dusk.
3. **Act 2 — the long night.** Time cards ("4 HOURS LATER", "3:00 AM"). Notifications get sadder and more absurd. The stillness tolerance tightens (harder to hold still when exhausted — `moveThreshold` shrinks).
4. **REVEAL 2 (~141s) — horror.** Cody's race track warps down like a dying tape deck (`warpOutRaceMusic`), screen goes near-black with a vignette, red eyes appear on the horizon and slowly approach. Movement tolerance is brutal. Then it decides you're furniture and leaves.
5. **Act 4 (~177s) — the fake crash.** A fake "STAYSTILL is not responding" OS dialog with an OK button. Clicking OK (or moving) = you lose ("YOU FELL FOR IT"). It was never frozen.
6. **REVEAL 3 (~189s) — the narrator + inversion.** The game types out the truth: there is no finish line, you win a race by racing. Controls **invert**: movement detection detaches, and now you must MASH SPACE/click/tap to build speed and cross a real finish line. Music roars back (`restoreRaceMusic`). The whole field un-DNFs and races you, and the strongest rival (Nikko, unless you ARE Nikko) rubber-bands onto your bumper for a **photo finish** (white-flash freeze, "BY 0.04s!!").
   - Mash to the line → **WINNER!** with podium + stats + your winning margin.
   - Start racing then stall for ~3s → the rival takes it → **2ND.** (rueful ending).
   - Refuse to move at all → **THE MONUMENT.** secret ending (they build a statue of you). The rival won't cross without a race, so this stays reachable.

Master pacing knob is the `T` object (all act timings in seconds after GO). Playtest with `index.html?ts=6` to run the whole arc at ~6x so you can feel the comedy timing without sitting 3.5 minutes; `?debug` logs the act schedule.

## Files

Everything lives at the project root.

- `index.html` — the entire game (HTML + CSS + JS in one file)
- `Saturday Brass Parade.mp3` — menu music (title → mode → cup → racer → loading)
- `Pixel Crown.mp3` — race music (crossfade in at race start)
- `NEXT.md` / `LOOPS.md` — steering files for Cody's "magi lemon" command; edit freely, `- [x]` hides a bullet
- No build step, no dependencies. Just `open index.html` in a browser.

## Deployment

Live at **https://magilemonai.github.io/staystill/** (GitHub Pages, serving `main` branch root of https://github.com/magilemonai/staystill — a public repo, so the MP3s are downloadable; Cody's fine with that). To ship a change: commit and `git push`, Pages rebuilds in ~1 minute. Sanity-check by curling the live URL for a string from the new code.

## Architecture

Single-file vanilla JS. Sections inside `index.html` are clearly delimited with comment banners:

- `<style>` — all CSS, split into `SCREENS BASE`, `TITLE`, `RACER SELECT`, `LOADING`, `TRACK`, `HUD`, `COUNTDOWN`, `NOTIFICATIONS`, `INCOMING CALL`, `END SCREEN`
- DOM body — six "screens" (`.screen.active` is shown), plus the persistent `#track`, `#hud`, `#notifications`, `#call`, `#end`, two `<audio>` elements
- `<script>` — sections: DATA, DOM refs, AUDIO, SCREEN STATE MACHINE, RACE, ACT DIRECTOR, FINALE, MOVEMENT DETECTION, helpers, init

### Act director

`scheduleActs()` (called from `startRace`) schedules every reveal off the `T` timeline via the `atT(sec, fn)` helper (which respects `TIMESCALE`). Key functions: `reviveRivalsForLongNight`, `warpOutRaceMusic`/`restoreRaceMusic`, `startFakeCrash`/`endFakeCrash`, `runNarratorSequence` (types lines with `typeNarrator`), `beginFinaleUnlock` → `startFinaleLoop` (the inverted mash-to-win loop) → `crossFinishLine` or `finishMonument`.

`phase` (act1 | lap2 | night | horror | crash | narrator | finale) gates behavior; `gameState` gains a `"finale"` value where movement no longer loses.

Atmosphere is CSS-class-driven on `#track` (`dusk`/`night`/`horror`/`dawn`/`driving`) plus overlay elements `#sky-tint`, `#stars`, `#eyes`, `#vignette`, `#announcer`, `#narrator`, `#timecard`, `#glitch` (fake crash), `#score-bonus`.

### Score

Passive stillness points tick (`startScoreTick`), notifications and ignored calls add points, multiplier ramps each act (`scoreState.mult`). Shown live in the HUD and summarized on the end screen (`#end-stats`, podium, plaque, COPY RESULT button).

### Near-miss reactivity

`startScoreTick`'s 500ms pulse doubles as the near-miss engine: mouse drift **decays** ~2.4px/s (you can recover by freezing), your kart wobbles at 45% of `moveThreshold` (`.drift1`) and trembles at 78% (`.drift2`), the horror eyes `.lean` in when drift > 50%, and `NEAR_MISS_TEXTS` fires one phase-appropriate text per level per act ("I saw that flinch."). Suppressed during crash/narrator phases so the frozen-game gag stays intact.

### Memory (localStorage `staystill_memory`)

`loadMemory()`/`saveMemory()`; written at the end of `endGame`. Tracks races / wins / seconds / monuments / losses / lastMargin / lastLossReason. Effects: title subtitle becomes "LAP 4?" + copyright shows your winning margin after a win, "ATTEMPT #N" otherwise; a statue of you joins the attract loop after a monument; Nikko's racer-select blurb becomes "lost by X.XXs. thinks about it hourly."; loading tips half the time quote your last death ("Last time you twitched the mouse. Don't."). Clear it with `localStorage.removeItem("staystill_memory")` in the console.

## Screen flow

```
title (PRESS START)
  → mode-screen   (GRAND PRIX selectable; TT/VS/BATTLE locked)
  → cup-screen    (MUSHROOM selectable; FLOWER/STAR/SPECIAL locked)
  → racer-screen  (8 characters, all selectable, stats panel)
  → loading-screen (track preview, loading bar, random tip)
  → countdown 3-2-1-GO
  → racing
  → end (win or lose) → RACE AGAIN returns to title
```

Navigation: arrow keys + Enter (or A) confirm, Esc (or B) back. Click works on every option.

State is driven by `screen` (string) and per-screen `enterXxx()` functions. Each `enterXxx()` calls `detachMenuHandlers()` then sets up its own key handler.

## Music

Two `<audio>` elements, crossfaded via a small `fadeAudio(el, from, to, ms)` helper.

- `startMenuMusic()` — plays brass parade. Called once on page load (autoplay attempt) and again on first user gesture (in case autoplay was blocked).
- `switchToRaceMusic()` — called from `enterRace()`. Fades brass out and pixel crown in over 700ms. Both happen at the moment the track screen becomes visible, just before the countdown.
- `stopAllMusic(fade)` — called from `endGame()`.

SFX use a Web Audio `AudioContext` and are separate from the MP3s: `sfxBeep`, `sfxMove`, `sfxConfirm`, `sfxDeny`, `sfxRing`, `sfxBuzz`, `sfxLose`, `sfxWin`.

## Characters and break mechanic

8 characters in `CHARACTERS` (keyed by `id`). Each has `name`, `color`, `stillness` (1-100, shown as a stat bar in racer-select), `weight`, `blurb`.

`AI_BREAK_TIMES` maps character `id` → seconds after GO when that character cracks. Three break almost immediately (dale 0.4, gerard 0.9, jim 1.7); the rest are graduated up to nikko at 88s. The player's chosen character is excluded from the schedule.

Being last one standing (nikko cracks at 88s) no longer wins — `breakRacer` no longer ends the game, and `WIN_TIME` is dead code. It's the trigger for the "LAP 2" reveal instead. The only wins are the finale (mash across the real line) and the monument (never move even after unlock).

The lineup puts the player at slot 3 (gold "YOU" highlight), other 7 fill remaining slots with a light shuffle so neighbors vary per race.

## Movement detection (the loss conditions)

Attached in `attachMovementDetection()` only during `gameState === "racing"`:

- Any keydown → "pressed X."
- Any mousedown → "clicked the mouse."
- Mouse motion > 80px total → "twitched the mouse."
- Touchstart → "touched the screen."
- Window blur (tabbing away) → "looked away from the race."
- Wheel/scroll → "scrolled."

All cleaned up in `detachMovementDetection()`. Loss reasons and the mouse threshold shift by `phase` (horror gives spookier copy and a much tighter threshold). **In the finale (`gameState === "finale"`) this is fully detached** — the controls invert and `attachFinaleControls()` takes over, where input builds speed instead of losing.

## Notifications

`NOTIFICATION_POOL` is an array of `{ t, type, from, urgent, body }`. `type: "call"` triggers the full-screen incoming call UI (`showCall`); everything else slides in on the right (`pushNotification`). Calls auto-dismiss after 4.5s.

Generic characters in notifications: **Sarah (Ex)**, **Jason (8)**, **Mom**, **Kate** (wife), **Landlord**, **boss@megacorp.com**, **IRS**, **UNKNOWN**, **?? UNKNOWN**. No real names — Zvika/Innovid/Molly were intentionally removed in an earlier pass.

## Tuning knobs

- `T` — **master act timeline** (seconds after GO). Every reveal keys off this. This is the main pacing dial now.
- `NARRATOR_LINES` — the fourth-wall monologue right before the finale unlock (each `{t, text}`; `t` is advisory, they play in sequence after `T.crashClear`). `text` can be a function evaluated at speak-time — the census line ("Four went home — Gerard has a family...") counts `aliveRacers` so it stays true no matter who rejoined or who you're playing.
- `FINALE_DISTANCE` — meters to the real finish in the mash-to-win phase (friction bleeds `finale.speed` ~2.2/tick, +7 per tap).
- `AI_BREAK_TIMES` — when each AI cracks in Act 1. Pacing: 3 immediate, then 14 / 28 / 45 / 65 / 88.
- `NOTIFICATION_POOL` — three tonal waves (Act 1 chaos, Act 2 long-night despair, Act 3 horror). Add jokes, retime urgency.
- `moveThreshold` (global) — mouse-drift px allowed before you lose. Starts 80, tightens to 46 (3 AM) then 30 (horror).
- `BREAK_REASONS` / `LOADING_TIPS` — random pools.
- `MENU_VOL` / `RACE_VOL` — music levels.
- `WIN_TIME` — legacy, now unused (outlasting no longer ends the game).
- `MODES`, `CUPS` — flip `enabled: true` to unlock more options.
- Dev: `?ts=N` runs the whole arc at N× speed for pacing playtests; `?debug` logs the act schedule.

## What we built today

In rough chronological order:

1. **One-shot first build**: full single-file game with synthesized Mario-Kart-ish music (Web Audio: kick/snare/hat/bass/square-arp over an 8-bar C/Am/F/G loop).
2. **Music swap**: Cody dropped in `Pixel Crown.mp3`. Removed the synth music code; kept SFX. MP3 plays at race start, fades on end.
3. **Generic-ifying**: removed Zvika / Innovid / Molly references. Boss → `boss@megacorp.com`, "1:1 with Zvika" → "1:1 with the CFO", Molly → Kate.
4. **MK64-style lead-up**: added title (PRESS START with car attract loop) → mode select → cup select → racer select (8 character cards with stats) → loading screen → race. Rebuilt the data model so AI break times are keyed to character identity instead of slot index.
5. **Two-track music**: added `Saturday Brass Parade.mp3` for the menus. Crossfades to `Pixel Crown.mp3` at the moment the track screen appears (just before countdown).
6. **Autoplay attempt**: brass parade tries to start on page load; falls back to first-gesture if the browser blocks autoplay.

### 2026-07-02 — polish round + deployment (all fixes from Cody's live playtest)

- **Warped-music bug**: dying during/after the horror act left `playbackRate` at ~0.45–0.9, so the next race's Pixel Crown played in slow motion. `switchToRaceMusic` now resets it.
- **Sky swap behind the timecard**: dusk→night used to crossfade visibly after the "4 HOURS LATER" card. `snapSky()` now swaps it instantly while the screen is fully black (via `showTimecard`'s `atBlack` callback).
- **Readability**: narrator monologue got a dark panel; end screen backdrop deepened with text shadows.
- **End screen clipping**: flex-centering clipped "WINNER!" on short windows. Now `flex-start` + `overflow-y: auto` with auto margins on first/last children — centers when it fits, scrolls when it doesn't.
- **Narrator census**: "Seven went home. Dale has a family" was wrong once racers rejoined (or if you WERE Dale). The line is now computed at speak-time from `aliveRacers`/`standings`.
- **Shipped**: public repo + GitHub Pages (see Deployment).

### 2026-07-01 — the genre-twist rebuild

Turned the ~95s one-note bit into a five-act escalation with three reveals (see "The twist" up top). Added: the act director + `T` timeline, lap-2 fakeout, night/3AM time cards, a horror act (music tape-warp, vignette, approaching eyes, tightening move tolerance), a fake OS-crash dialog trap, a typewriter narrator that breaks the fourth wall, and an inverted finale where you finally have to MASH to cross a real finish line — with a WINNER podium ending and a secret MONUMENT ending for refusing to move even then. Also a live stillness-points score, COPY RESULT share card, and `?ts=`/`?debug` dev hooks. Kept it single-file vanilla; kept Cody's two MP3s (the horror act bends `Pixel Crown` rather than replacing it). **Fixed a latent infinite loop** in `pushNotification`'s cap logic (deferred `.remove()` inside a `while (children.length > 5)` never terminated once the denser notification pool overflowed the cap) — verified via a jsdom harness that drove all endings.

## Possible next steps Cody might want

(The live list lives in `NEXT.md` — keep that one current.)

- Playtest the full five-act arc at real speed (not `?ts=6`) to tune comedy timing — still not done as of 2026-07-02.
- Hooking up the locked modes/cups for variations (TIME TRIAL = race solo against a clock, VS = pick the AI roster, BATTLE = ???).
- More notification jokes — the pool has room and the comedy is in the writing.
- Mobile pass: the racer-select grid (4 columns) probably needs to collapse to 2 columns at narrow widths; touch = instant loss also makes phones rough.
- Difficulty knob (50cc / 100cc / 150cc) that scales the AI break times.

Done since this list was written: winner podium (photo-finish ending), shareable result (COPY RESULT button).

## Cody-specific notes

- Cody composes for theater; the music quality matters to him. Pixel Crown and Saturday Brass Parade are his own tracks. Don't suggest replacing them or generating synthetic music unless he asks.
- The joke lives or dies on pacing. If you change the break schedule, playtest the full ~90 seconds before declaring it tuned.
- He works in vanilla web tech here on purpose. Don't suggest React, build tools, or frameworks for this project.
