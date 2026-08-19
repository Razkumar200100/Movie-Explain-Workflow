# 🎬 MOVIE EXPLAINER AGENT — COMPLETE FRAMEWORK (v2.0)

> **Purpose:** User sirf movie ka naam (+ file) deta hai → system final
> YouTube-ready video deliver karta hai: script, voiceover, edited clips,
> captions, music, thumbnail, title/description/tags.
> Ye document is system ka **single source of truth** hai.

---

## TABLE OF CONTENTS

1. [Architecture — Agents & Orchestrator](#1-architecture)
2. [Style Rulebook (Script Agent ka core)](#2-style-rulebook)
3. [Workflow — End-to-End (Phase 0–8)](#3-workflow)
4. [Agent Specs](#4-agent-specs)
5. [Manifest Schema (backbone)](#5-manifest-schema)
6. [Prompt Templates](#6-prompt-templates)
7. [Review Queue (human clicks)](#7-review-queue)
8. [Copyright Safety Rules](#8-copyright-safety)
9. [QC Checklists](#9-qc-checklists)
10. [Metadata Formulas](#10-metadata-formulas)
11. [GUI Screens (desktop app)](#11-gui-screens)
12. [Installable EXE Plan](#12-exe-plan)
13. [Project File Structure](#13-file-structure)
14. [Build Phases & Time Budget](#14-build-phases)
15. [Accuracy Targets & Failure Handling](#15-accuracy--failure)

---

## 1. ARCHITECTURE

```
USER (movie name + file)
        │
        ▼
┌──────────────────────────────────────────────┐
│             ORCHESTRATOR AGENT               │
│  manifest.json | checkpoints | routing |     │
│  parallel tasks | crash-resume | GUI events  │
└─────────────┬───────┬───────┬───────┬───────┘
       ▼       ▼       ▼       ▼       ▼
   SCRIPT  UNDER-  MATCH-  PRODUCTION  PUBLISH
   AGENT   STANDING ING     AGENT       AGENT
           AGENT   AGENT
                   + SAFETY AGENT (cross-cutting, har phase mein check)
```

| Agent | Responsibility |
|-------|----------------|
| **Orchestrator** | manifest maintain, step ordering/parallelism, resume-after-crash, GUI progress |
| **Script Agent** | plot → breakdown → 5-chunk script → QC → metadata |
| **Understanding Agent** | movie file → transcript, scenes, keyframes, scene analysis, movie_understanding.json |
| **Matching Agent** | clip DB, 4-stage matching, review queue, duration adaptation |
| **Production Agent** | voice, assembly, transitions, sound, captions, grade, export |
| **Safety Agent** | copyright mods, validation, pre-upload QC |
| **Publish Agent** | thumbnail, title/desc/tags, YouTube upload |

---

## 2. STYLE RULEBOOK

*(Samples se extracted — Script Agent ke SYSTEM prompt mein hamesha inject hota hai)*

1. **LANGUAGE:** Devanagari Hindi, conversational Hindustani. Chhote sentences; ek sentence = ek action. Urdu touch: फरमान, जल्लाद, कत्ल, तहस-नहस, नामोनिशान, चकमा, बेकसूर.
2. **NO META-TALK:** na channel intro, na review, na actor/director baat. Pure third-person storytelling; seedha kahani se shuru.
3. **OPENING (3 types mein se ek):**
   - (a) normal→toot-phoot: "सब कुछ नॉर्मल चल रहा था… लेकिन आज कुछ ऐसा होगा जो दुनिया बदल देगा"
   - (b) concept-first: "यह एक ऐसी दुनिया है जहां…"
   - (c) mystery+sawaal: "शहर वीरान है… तो आखिर सवाल आता है कि…"
4. **WORLD RULES** pehle 500 words mein (maut ka time+jallad / 25 ke baad countdown / Dark Seekers raat mein / time=paisa).
5. **GLUE WORDS** har 3-4 sentences: दरअसल, तभी, इसी दौरान, वहीं दूसरी तरफ, उधर/इधर, अगले ही पल, यह देखकर, इसके बाद.
6. **MICRO-SUSPENSE:** har mod se pehle "लेकिन तभी…" / "तभी अचानक…" (~har 400-500 words).
7. **RHETORICAL QUESTIONS** har 600-800 words: "तो आखिर वह क्या चीज है…", "अब सवाल आता है…"
8. **CHARACTER ENTRY:** naam + ek-line identity/motivation.
9. **EMOTION VOCAB:** पैरों तले जमीन खिसक गई, आग बबूला हो गया, होश उड़ गए, आंखें खुली की खुली रह गईं, दिल टूट गया, सन्न रह गए, मायूस हो गया, खुशी से झूम उठा, गुस्से से लाल हो गया.
10. **MUSIC CUES:** suspense/action/emotional beats pe `[संगीत]` marker.
11. **ENDING:** "इसी के साथ यह कहानी खत्म हो जाती है" + like/subscribe CTA; optional like-bait.
12. **LENGTH:** 3000-4500 words = 20-25 min narration.

---

## 3. WORKFLOW — END TO END

### PHASE 0 — INPUT (human ~2 min)
- Inputs: movie name | movie file (browse) ya legal link (yt-dlp) | duration preset | style preset
- 2+ audio tracks → GUI popup se track select
- `manifest.json` create; sab steps `pending`

### PHASE 1 — UNDERSTANDING (background 45-70 min)
1. ffmpeg → selected audio track extract
2. Demucs → vocal separation (music/noise alag)
3. Whisper large-v3 (`word_timestamps=True`) → `transcript.json`
4. PySceneDetect AdaptiveDetector (threshold 3.0, min_len 15) → `scenes.json`
5. 3 keyframes/scene (start/mid/end)
6. Gemini 1.5 Pro: movie ko 15-min segments + sliding summary → scene analyses
7. Synthesis → `movie_understanding.json` (story arc, characters, rules, turning points **with file timestamps**, emotional journey)
8. **Plot cross-check:** LLM knowledge + file understanding → merged beat-sheet (conflict mein FILE jeetti hai)

### PHASE 2 — SCRIPT (8-12 min)
9. 5 chunks generate (650/1100/650/750/400 words) — rulebook + few-shot + tail continuity
10. Script QC checklist → fail chunk rewrite (max 2 attempts)
11. Metadata: 3 title options, description, tags, thumbnail prompt

### PHASE 3 — VOICE (10-15 min)
12. ElevenLabs per-segment (emotion params) → audio files + exact durations
13. **DURATION LOOP:** total 18-25 min ke bahar → affected chunk rewrite + re-voice

### PHASE 4 — CLIP DB (Phase 2-3 ke PARALLEL, 30-60 min)
14. Scenes → 3-5 sec clips; extraction-time copyright mod; `-an` (original audio dead)
15. CLIP ViT-L/14: 5 frames mean-pool embedding → ChromaDB (+ english_desc metadata)

### PHASE 5 — MATCHING (15-25 min)
16. 4-stage matching per segment
17. Confidence gate: ≥0.6 auto-lock | <0.6 → REVIEW QUEUE
18. **HUMAN TOUCHPOINT #1:** top-3 candidates pe clicks (~10-15 clicks, ~5 min)
19. Duration adaptation (ratio + motion check); unmatched → Ken Burns still + text card

### PHASE 6 — ASSEMBLY + POST (30-50 min)
20. Timeline + transitions (mood matrix) + end card (subscribe animation)
21. Sound: `[संगीत]` pe SFX stings, mood music (−18dB), sidechain ducking
22. Captions: ASS karaoke (Phase 1/3 ke Whisper words reuse) → burn
23. Grade: single LUT/eq chain → visual consistency

### PHASE 7 — QC + APPROVE + EXPORT (~10 min)
24. Auto QC (sync, black frames, clipping, copyright validate, duration, captions)
25. **HUMAN TOUCHPOINT #2:** final preview (2x speed) → approve/tweak
26. Thumbnail generate → Export 1080p

### PHASE 8 — PUBLISH (~5 min)
27. **HUMAN TOUCHPOINT #3 (optional):** title/desc approve
28. YouTube upload (one-time OAuth) ya export-only
29. Manifest `done`; project archive

---

## 4. AGENT SPECS

### 4.1 SCRIPT AGENT
- **Tools:** LLM (GPT-4o / Qwen-Max / Gemini), Wikipedia fetch (unknown movies fallback)
- **Inputs:** movie_name, movie_understanding.json
- **Outputs:** script.json, metadata.json
- **Segment schema:** `{id, section, text, visual, emotion, sfx}`
- **Rules:** §2 Rulebook; chunked generation; QC checklist (§9)

### 4.2 UNDERSTANDING AGENT
| Step | Tool | Params |
|------|------|--------|
| Vocal separation | Demucs | default model |
| Transcription | Whisper | large-v3 (quality) / medium (speed), word_timestamps=True |
| Scene detection | PySceneDetect | AdaptiveDetector, threshold=3.0, min_scene_len=15 |
| Keyframes | OpenCV | 3/scene |
| Scene analysis | Gemini 1.5 Pro | 15-min segments + previous 2-line summary |
| Synthesis | LLM | → movie_understanding.json |

### 4.3 MATCHING AGENT
```python
def match_segment(seg, db, last_ts, review_queue):
    q = english_visual_query(seg["visual"])        # Stage 0 (CLIP Hindi mein weak)
    cands = db.search(q, top_k=15)                 # Stage 1 semantic
    fb = any(w in seg["text"] for w in ["याद","flashback","पहले","bachpan"])
    cands = [c for c in cands if c.start >= last_ts-2 or fb]   # Stage 2 temporal
    scored = [(c, 0.6*c.sim + 0.4*object_overlap(q, c.mid_frame)) for c in cands]  # Stage 3
    best, score = max(scored, key=lambda x: x[1])
    if score < 0.6: review_queue.append({"seg": seg, "top3": scored[:3]})  # Stage 4 gate
    return best, score
```
**Duration adapter:**
```python
r = need / clip.dur;  motion = frame_diff_score(clip)
if r <= 1: TRIM
elif r <= 1.5 and motion < TH: SLOW_MO(1/r)      # calm scenes only
elif r <= 2: BOOMERANG if motion < TH else ALT_ANGLE
elif r <= 3: KEN_BURNS
else: MULTI_CLIP
# unmatched → Ken Burns last-frame + text overlay card
```

### 4.4 PRODUCTION AGENT
**Voice emotion params (ElevenLabs multilingual v2):**
| Emotion | Speed | Stability | Style | Pause after |
|---------|-------|-----------|-------|-------------|
| suspense | 0.85 | 0.4 | 0.7 | 1.5s |
| action | 1.10 | 0.6 | 0.8 | 0.3s |
| emotional | 0.80 | 0.3 | 0.9 | 2.0s |
| calm | 0.95 | 0.7 | 0.5 | 0.8s |
| revelation | 0.90 | 0.35 | 0.85 | 1.5s (+1s before) |

**Transitions (mood matrix):** action→action `whip_pan 0.3s` | calm→suspense `fade_black 0.8s` | suspense→reveal `zoom_cut 0.4s` | default `crossfade 0.5s`

**Sound:** music −18dB, SFX −12dB, sidechain ducking (threshold 0.02, ratio 8)

**Captions:** ASS karaoke `{\k}` word-highlight | Arial Black 52 | white + black outline 3 | bottom | max 6 words/line

**Export:** 1920×1080 | h264 crf 18 | 30fps | aac 192k | +faststart

### 4.5 SAFETY AGENT — see §8
### 4.6 PUBLISH AGENT — see §10

---

## 5. MANIFEST SCHEMA

```json
{
  "project": "in_time_2025",
  "movie_name": "In Time",
  "file": "D:/movies/in_time.mp4",
  "audio_track": 1,
  "steps": {
    "transcribe": "done", "scenes": "done", "understanding": "done",
    "script": "done", "voice": "running", "clipdb": "done",
    "matching": "pending", "assembly": "pending", "post": "pending",
    "qc": "pending", "export": "pending", "upload": "pending"
  },
  "segments": [
    {
      "id": 1, "section": "hook",
      "text": "यह एक ऐसी दुनिया है जहां पैसों की कोई वैल्यू नहीं…",
      "visual": "crowd walking toward giant gate, white mountains",
      "emotion": "suspense", "sfx": "[संगीत]",
      "audio_path": "audio/seg_001.mp3", "audio_dur": 42.5,
      "clip_path": "clips/s003_c012_safe.mp4", "adaptation": "slow_mo",
      "confidence": 0.87, "transition": "crossfade"
    }
  ]
}
```
> Har step complete hone pe status update. Crash → restart pe `done` steps skip (resume).

---

## 6. PROMPT TEMPLATES

### P1 — Plot Fetch (unknown movies)
```
Wikipedia/IMDb se "{movie}" ka scene-by-scene detailed plot nikaalo.
ALL spoilers included. Character names, twists, emotional beats, ending.
Summary NAHI — beat-by-beat retelling.
```

### P2 — Story Breakdown
```
Is plot se JSON banao:
{world_rules, characters:[{name,role,intro_line}],
 beats:[{order,event,emotion,visual}], midpoint_twist, climax,
 resolution, hook_candidates:[3]}
```

### P3 — Script Chunk (CORE)
```
SYSTEM: Tum Hindi movie explainer ho jo 20-25 min ke explanation
scripts likhta hai. Ye STYLE RULEBOOK har line par lagu hai:
{style_rulebook.md}
Reference style ka example:
{sample.txt ka 600-word excerpt}          ← FEW-SHOT (sabse bada lever)

USER:
Movie: {name}
Is chunk ke beats: {breakdown ka relevant hissa}
Pichle chunk ki aakhri 3 lines: {tail}    ← continuity
Likhna hai: {section} | target: {n} words
Har segment: {text (Devanagari), visual, emotion, sfx}
Rules: chhote sentences, glue words, har mod se pehle "लेकिन तभी",
rhetorical question, emotion vocab, koi review/meta-talk nahi.
```

### P4 — English Visual Query (matching ke liye)
```
Is Hindi line ko 1-line English visual search query banao
(subject + action + setting only): "{visual_line}"
```

### P5 — Gemini Scene Analysis
```
Previous context: {summary}
Dialogue transcript: {seg_transcript}
Is movie segment ka JSON analysis do: visual_description, characters,
actions, emotional_tone, camera_work, location, key_objects,
story_relevance_score(1-10), timestamps_of_interest.
```

### P6 — Thumbnail
```
Cinematic YouTube thumbnail: {concept}. GIANT {threat} vs TINY humans,
humans FROM BEHIND (fear), one focal point, high saturation,
dramatic light, NO TEXT, 16:9.
```

---

## 7. REVIEW QUEUE (human clicks)

- Score ≥0.6 → auto-lock (koi click nahi)
- Score <0.6 → queue mein card: segment text + top-3 clip previews (2-2 sec)
- Human sahi clip pe **ek click** karta hai
- 20-min video ≈ 180-220 segments → ~10-15 pending → **10-15 clicks ≈ 5 min**
- Isi se matching 80% → 95%+ hoti hai. Ye ONLY mandatory manual kaam hai.

---

## 8. COPYRIGHT SAFETY

| Rule | Limit |
|------|-------|
| Continuous clip | ≤7 sec (ideal 4) |
| Total movie footage | <10% of video |
| Original audio | NEVER (`-an` at extraction) |
| Voiceover ratio | >70% (commentary = primary content) |
| Description | disclaimer + credits mandatory |

**Extraction-time mod chain (har clip pe):**
```
hflip, crop=iw*0.92:ih*0.92, scale=1920:1080,
eq=brightness=0.02:contrast=1.05:saturation=1.08, fps=30, -an
```
**Validate (pre-upload):** footage %, max continuous clip, original audio detect, VO ratio → fail = block upload.

---

## 9. QC CHECKLISTS

**Script QC:**
- [ ] Pehli 2 lines = hook (3 types)
- [ ] Rules <500 words
- [ ] "लेकिन तभी" har 400-500 words
- [ ] Rhetorical question har 600-800 words
- [ ] Emotion vocab beats pe
- [ ] Har character: naam + intro line
- [ ] `[संगीत]` markers present
- [ ] Ending formula + CTA
- [ ] 3000-4500 words | koi meta-talk nahi

**Pre-upload QC:**
- [ ] A/V sync drift <100ms
- [ ] No black frames | no audio clipping
- [ ] Voice vs music ratio ≥3
- [ ] Resolution consistent 1080p | duration 15-25 min
- [ ] Captions present + synced
- [ ] Copyright validate pass
- ERROR → auto-fix attempt → fail to human flag

---

## 10. METADATA FORMULAS

**TITLE:** `[Hindi curiosity hook — movie name CHHUPA, CAPITAL English keywords] ( [MOVIE NAME] Movie Explained in Hindi/Urdu )`
> ex: "Jab Hamari Duniya Par Zombies Ne Qabza Kar Liya ( I AM LEGEND Movie Explained in Hindi Urdu"

**DESCRIPTION:** 🎬 Movie Credits block — Name / Release Year / Genre / Language / Country / Director (+ Writer / Main Cast) + §8 disclaimer

**THUMBNAIL:** AI concept art — giant threat vs tiny humans, back-view humans, no text, high saturation (§6-P6)

---

## 11. GUI SCREENS

| Screen | Kaam |
|--------|------|
| S1 Setup | API keys, FFmpeg check, model downloads (progress) |
| S2 New Project | name + file/link + track select + presets → START |
| S3 Script | 5 chunk tabs, edit/regenerate, QC badges, metadata |
| S4 Voice | voice picker + test, emotion preview, generate |
| S5 Matching | timeline list, confidence tags, **review clicks**, adaptation tags |
| S6 Preview | rough preview, music/captions/grade toggles → APPROVE |
| S7 Export | QC report, thumbnail, metadata edit, EXPORT / UPLOAD |

---

## 12. EXE PLAN

**Bundle vs download:**
| Cheez | Source | Size |
|-------|--------|------|
| App + GUI (CustomTkinter) + deps | Installer bundled | ~150-250 MB |
| FFmpeg static | Bundled | ~100 MB |
| Torch CPU | Bundled | ~800 MB |
| Fonts/LUTs/music/SFX | Bundled | ~50 MB |
| Whisper + CLIP models | First-run download | 0.4-4 GB (choice) |
| CUDA torch | GUI button (optional) | on-demand |

**Build Step 1 — PyInstaller (onedir, NOT onefile):**
```bat
pyinstaller --noconfirm --onedir --windowed ^
  --name "MovieExplainerStudio" ^
  --add-data "assets;assets" --add-data "bin\ffmpeg;bin\ffmpeg" ^
  --add-data "style_rulebook.md;." --add-data "examples;examples" ^
  --collect-all customtkinter --hidden-import torch ^
  main_gui.py
```

**Build Step 2 — Inno Setup → single installer exe:**
```iss
[Setup]
AppName=Movie Explainer Studio
AppVersion=1.0
DefaultDirName={autopf}\MovieExplainerStudio
OutputBaseFilename=MovieExplainerStudio-Setup-v1.0
ArchitecturesInstallIn64BitMode=x64
PrivilegesRequired=lowest
WizardStyle=modern
[Files]
Source: "dist\MovieExplainerStudio\*"; DestDir: "{app}"; Flags: ignoreversion recursesubdirs
[Icons]
Name: "{autodesktop}\Movie Explainer Studio"; Filename: "{app}\MovieExplainerStudio.exe"
[Run]
Filename: "{app}\MovieExplainerStudio.exe"; Flags: nowait postinstall skipifsilent
```

**Caveats:** torch hidden-imports mein 2-3 din debugging; antivirus flag → code signing; Windows-only build.

---

## 13. FILE STRUCTURE

```
movie-explainer-studio/
├── main_gui.py              # CustomTkinter app (S1-S7)
├── orchestrator.py          # state machine + checkpoints
├── agents/
│   ├── script_agent.py      # P1-P4 + QC
│   ├── understand_agent.py  # demucs/whisper/scenedetect/gemini
│   ├── match_agent.py       # clip DB + matcher + adapter + queue
│   ├── production_agent.py  # voice/assemble/sound/captions/grade/export
│   ├── safety_agent.py      # copyright + QC
│   └── publish_agent.py     # thumbnail + upload
├── style_rulebook.md        # §2
├── examples/sample.txt      # few-shot reference script
├── bin/ffmpeg/              # bundled
├── assets/                  # fonts, luts, music, sfx
├── models/                  # first-run downloads
├── projects/                # per-movie working folders + manifest.json
├── config.json              # API keys + prefs
└── build/                   # build_exe.bat + installer.iss
```

---

## 14. BUILD PHASES & TIME BUDGET

**Dev phases:**
```
Wk 1-2: Script Agent + rulebook → 3 test scripts (samples se compare)
Wk 3:   Understanding Agent
Wk 4:   Matching Agent + review queue
Wk 5:   Production Agent
Wk 6:   GUI S2-S7 (S5 pehle)
Wk 7:   S1 + downloader + Safety + Publish
Wk 8:   PyInstaller + Inno → Setup.exe → test install
```

**Per-video runtime (mid GPU):**
| Phase | Time | Human |
|-------|------|-------|
| Input | 2 min | ✅ |
| Understanding | 45-70 min | ❌ |
| Script | 8-12 min | ❌ |
| Voice | 10-15 min | ❌ |
| Clip DB (parallel) | 30-60 min | ❌ |
| Matching + clicks | 15-25 min | ✅ 5 min |
| Assembly + post | 30-50 min | ❌ |
| QC + preview | 10 min | ✅ 10 min |
| Publish | 5 min | ✅ 2 min |
| **TOTAL** | **~2.5-3.5 hr** | **~20 min** |

---

## 15. ACCURACY & FAILURE

| Component | Auto % | Human touch |
|-----------|--------|-------------|
| Script (rulebook+few-shot+QC) | 90-95% | tone review |
| Understanding | 85-90% | plot verify |
| Matching (+gate+queue) | 80 → 95% | clicks |
| Duration adaptation | 90% | rare |
| Voice/captions | 95% | spot check |

**Failure handling:**
- API fail → 3 retries → step `paused` + GUI notify → [RETRY]
- Crash → manifest resume (done steps skip)
- Unmatched clip → Ken Burns + text card fallback
- Script QC 2x fail → human edit flag (S3)
- QC error → auto-fix → fail to human flag

---

**BOTTOM LINE:** Script quality ka raaz = detailed plot + rulebook + few-shot + chunks + QC.
Close-to-accurate automation ka raaz = file-first understanding + 4-stage matching + review queue + motion-aware adaptation.
Human ka total kaam = 4 chhote touchpoints (~20 min/video). Baaki sab background.