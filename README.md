# BOOP·SCAPE

A chunky, bright, slightly weird music maker for phones. Tap pixel critters onto a staff, hit play, and watch them hop in time. It's one self-contained HTML file with no build step and no dependencies, made for the [71dpi](https://71dpi.github.io) site.

BOOP·SCAPE is inspired by the "drop little characters on a staff" idea of old-school console music toys. Everything in it is original: the critters, the sounds, the layout and the code. No third-party artwork, characters or audio samples are used.

---

## What it is

- **A tap-to-compose staff.** 13 steps from C4 up to A5, scrolling sideways, 2 to 32 bars of 4/4. One column is an eighth note.
- **Eight critters, each a synthesized instrument.** Everything is generated live with the Web Audio API, so there are no sound files.
- **Built-in songs.** Jingle Bells, Canon in D and Ba Ba Black Sheep, all public domain and simplified for the staff.
- **Sharing.** A link that contains the whole song, a WAV export, and a text export.
- **Chatbot import.** Copy a prompt, ask any chatbot for a public-domain song, paste its reply, and it plays.
- **Mobile first.** Big touch targets, safe-area padding, portrait and landscape layouts, and autosave.

### The critters

| Critter | Role | Sound |
|---|---|---|
| **BLIP** | lead | square wave through a low-pass filter |
| **WUB** | bass | triangle plus sine one and two octaves below the written pitch, with a closing filter sweep |
| **ZAP** | lead / effect | sawtooth with a fast pitch drop through a band-pass filter |
| **PLING** | bell lead | two-operator FM bell with a long decay |
| **THUD** | kick | sine with a falling pitch, plus a short noise click |
| **HISS** | hi-hat | band-passed noise; higher notes give shorter ticks |
| **BOING** | bass | sine that bends up into pitch with a wobble, plus a quiet triangle an octave up |
| **OOOH** | pad / choir | two detuned sawtooths, low-passed, with a slow swell |

---

## Using it

| Control | What it does |
|---|---|
| Tap the staff | Drop the selected critter on that note and beat. Tap it again to remove it, or tap with a different critter to swap it. |
| Critter palette | Pick a critter. Tapping one plays a preview. The red X is the eraser. |
| ♮ ♯ ♭ | Accidental applied to the next notes you place. |
| Bar ruler (top of the staff) | Tap a bar number to set where playback starts. |
| ▶ PLAY / ■ STOP | Starts and stops playback. The playhead eye follows along and notes hop as they sound. |
| LOOP | Loops from the start bar back around the song. |
| Tempo slider | 60 to 240 BPM, counted in quarter notes. |
| − / + (header) | Fewer or more bars. |
| UNDO | Up to 60 steps. Loading a song or an import can be undone too, including its bar count and tempo. |
| CLEAR | Two taps, so it can't happen by accident. |
| ♫ SONGS | Built-in songs and the import sheet. |
| ↗ SHARE | Link, WAV, boop text and .txt file. |

Limits: up to 5 notes per beat, 32 bars (256 columns). On a keyboard, Space plays and stops, Ctrl/Cmd+Z undoes, and Esc closes any sheet.

Your song autosaves to the browser (`localStorage`, key `boopscape.v1`), so it's there when you come back.

---

## Sharing and exporting

**Link.** The whole song is stored in the URL hash (`#s=…`), so no server is involved. Opening the link loads the song straight into the app. A 13-bar song is about 400 characters, short enough to text. If the person opening it already had a song saved, UNDO brings theirs back. A malformed link falls back to the normal start-up song.

**WAV.** The song is rendered offline with an `OfflineAudioContext` (mono, 44.1 kHz, 16-bit) and normalized. The first tap renders. The second tap hands the file to the phone's share sheet where available, or downloads it. It's two taps because browsers only allow the share sheet right after a tap.

**Boop text and .txt.** "Copy as boop text" gives you the song as editable text (see below). The .txt file contains the link plus the same text.

### Link format

Base64url of a small byte array: `[version=1, bars, bpm−60]`, followed by 3 bytes per note. Each note packs 17 bits: column (8), row (4), critter (3) and accidental+1 (2).

---

## Boop notation

A plain-text format that is easy for people and chatbots to read and write. This is what the import sheet accepts and what "copy as boop text" produces.

```
BOOP 1
title: Ba Ba Black Sheep
bpm: 104
bars: 4

[PLING]
C5:2 C5:2 G5:2 G5:2 A5:2 A5:2 G5:4 F5:2 F5:2 E5:2 E5:2 D5:2 D5:2 C5:4

[BOING]
C4:4 C4:4 F4:4 C4:4 F4:4 C4:4 G4:4 C4:4
```

- **Time** is a grid of columns. One column is an eighth note, and a 4/4 bar is 8 columns.
- **NOTE:LENGTH** is a note name with an optional `#` or `b`, then an octave, then a length in columns (eighth = 1, quarter = 2, half = 4, whole = 8). Valid pitches run from C4 to A5.
- **Rests** are `-:LENGTH`. **Chords** join notes with `+`, as in `C4+E4+G4:4`.
- **Parts.** Each `[INSTRUMENT]` section starts at the beginning of the song and reads left to right. Repeat a header to layer another part.
- **Lenient reading.** The importer ignores chatter and code fences around the notation. It accepts `|` bar lines and comments, and it moves out-of-range notes by octaves instead of rejecting them. It warns about anything it had to guess: unknown instruments, missing lengths, unreadable tokens and dropped notes.

### Importing from a chatbot

1. Open **♫ SONGS → ↓ IMPORT FROM TEXT**.
2. Optionally type a song name, then tap **COPY THE PROMPT**.
3. Paste the prompt into any chatbot. It explains the format, the C4–A5 range, the instruments and the public-domain rule, and includes a worked example.
4. Paste the chatbot's reply back into the sheet. A live summary shows bars, tempo, note count and any warnings.
5. Tap **LOAD IT**. Chatbots do get notes wrong, so play it and fix it by ear. You can also copy your song back out as boop text and ask the chatbot to revise it.

---

## How it works

The whole app is `boopscape.html` (about 1,000 lines): HTML, CSS and one script in an IIFE, with no framework and no bundler.

- **State.** A `grid` of 256 columns, each holding up to five `{row, critter, accidental}` notes, plus bars, tempo, loop and start bar. Rows 0–12 map to C4–A5 on a C-major scale, and the accidental shifts a note by a semitone.
- **Rendering.** The staff is an SVG background built for the current size, with note sprites as absolutely positioned elements. The layout recomputes on resize and rotation.
- **Art.** The critters are 8×8 pixel sprites defined as strings of characters. At start-up each is drawn to a tiny canvas and turned into a data URL, then shown with `image-rendering: pixelated`.
- **Audio.** Each critter is a small function that builds Web Audio nodes at a scheduled time. A master gain and compressor keep dense passages from clipping.
- **Playback.** A look-ahead scheduler runs every 25 ms and schedules notes about 150 ms ahead on the audio clock, so timing stays steady even if the screen stutters. The playhead and the note hops are driven by `requestAnimationFrame` against the same clock, compensating for output latency.
- **WAV export** reuses the same critter functions against an `OfflineAudioContext`. They borrow the shared audio variables for the render and give them back afterward, and live playback is blocked while that happens.
- **Autosave** is debounced to 250 ms, and loading validates everything it reads back.
- **Sheets.** Songs, share and import are bottom sheets over a dimmed overlay. Keyboard shortcuts are suspended while one is open, so Space types a space in the text boxes.

### Built-in songs

Each song is written with a tiny token format (`E5:2` means E in octave 5 for two columns) and laid onto the grid by a helper, with extra helpers for drum hits. They are simplified arrangements:

- **Jingle Bells** (Pierpont, 1857): Blip melody, Wub bass, Thud and Hiss drums.
- **Canon in D** (Pachelbel): Pling melody that grows from half notes to quarters to arpeggios, with an Oooh pad carrying the famous descending line. Because the staff stops at C4, the bass sits higher than in the original, and Wub's octave drop brings it back down.
- **Ba Ba Black Sheep**: Pling melody, Boing bass, Thud.

---

## Running it

Open `boopscape.html` in a browser, or host it as a static file. No install is needed.

- It uses the Press Start 2P font from Google Fonts and falls back to a system monospace font offline.
- The Web Audio API needs a user tap to start sound, which the app handles on the first tap.
- On iPhone, the ring/silent switch can mute web audio. Flip it to ring if you hear nothing.
- Share links point at wherever the file is hosted. They only work for other people once it's online, for example at 71dpi.github.io.

---

## How it was made

BOOP·SCAPE was built in a back-and-forth conversation with Claude (Anthropic's AI model) and written and tested in a Claude Code-style sandbox.

1. **The brief.** It began as a request for a 1:1 clone of a classic console music toy, optimized for mobile. A literal clone would mean copying someone else's artwork and sound samples, so the project became an original instrument in the same spirit, with its own critters and synthesized sounds, in a "chunky, bright, retro weirdness" style.
2. **The first build** was the staff, the eight critters and their synth voices, the scheduler and playhead, tempo and bar controls, undo, and autosave.
3. **Songs menu.** Three public-domain examples were added, written as short note strings and checked to fit the staff range with no dropped notes.
4. **Share and export** followed: the song-in-a-link encoding, the offline WAV render and the text export.
5. **Chatbot import** came last: boop notation, the prompt, a forgiving parser with warnings, and the matching export.

Features were checked in a real browser (headless Chromium driven by Playwright) rather than assumed:

- a shared link opened in a fresh browser reproduces the song exactly;
- the exported WAV is a valid file with real audio, and live playback still works afterward;
- the import prompt's own example parses with no warnings;
- messy and garbage pastes are handled;
- each built-in song survives export and re-import identically.

Those test scripts were ad hoc and are not part of this repo. One thing testing caught was status messages ("LINK COPIED!") rendering underneath the open sheets, which was fixed.

Things the tests couldn't cover: the phone's native share sheet and real chatbots' output. Both depend on the device and the model, so try them on a real phone and with a few chatbots.

---

## Ideas for later

- Note length and held notes (right now every note is a hit that rings out on its own).
- A song name that carries into shared links and files.
- More critters, or a pitch-bend and effects row.
- Time signatures other than 4/4.
- A gallery of community songs shared as links.

## License

None chosen yet.
