# Visible &amp; Trusted

A personal-branding system built on The Brand Company's **Visible &amp; Trusted** method. It takes a professional from a blank page to a clear, distinct personal brand, then distils the result into a **collaborator-ready brief** they can hand to a podcast producer, PR firm, or content writer.

There are two parts:

1. **The Engine** — an interactive, AI-powered web app that challenges the user's answers (it refuses vague positioning, pushes for a single word to own) and synthesises a brief.
2. **The Brief Generator** — a small Python tool that turns the Engine's exported answers into a branded PDF and an editable Word document, in The Brand Company's Swiss visual style.

---

## Who this repo is for

This repo is the **source of truth** for the project: version control, records, and any future developer or deployment work.

It is **not** how non-technical collaborators should receive the tool. For them, the path is to **publish the Engine artifact in Claude and send the link** — they click and use it, nothing to install. See [docs/PROJECT_HANDOFF.md](docs/PROJECT_HANDOFF.md) for the full background on that decision.

---

## Repository layout

```
visible-trusted-repo/
├── README.md                     ← you are here
├── index.html                    ← static project landing page (safe for GitHub Pages)
├── engine/
│   └── visible_trusted_engine.jsx ← the interactive Engine (React; runs as a Claude artifact)
├── generator/
│   ├── build_tbc_brief.py         ← reads an export, writes the PDF (and calls the docx builder)
│   ├── build_brief_docx.py        ← reads an export, writes the editable Word version
│   └── requirements.txt
├── samples/
│   └── sample_export.json         ← an example Engine export, for testing the generator
└── docs/
    └── PROJECT_HANDOFF.md         ← full project history, decisions, and resume notes
```

---

## How the pieces fit together

```
  ┌─────────────┐   user completes,    ┌──────────────────┐   "Export for      ┌────────────────────┐
  │  The Engine │   AI challenges,     │  Distilled brief │    brief" (JSON)   │  Brief Generator   │
  │  (.jsx)     │ ───────────────────▶ │  in the app      │ ─────────────────▶ │  (Python)          │
  └─────────────┘   pushes for a       └──────────────────┘                    └────────────────────┘
                    distinct position                                            │
                    + a word to own                                              ▼
                                                                    TBC-branded PDF  +  editable .docx
```

The Engine and the generator are **deliberately separate technologies**. The Engine is a live React artifact that runs inside Claude (where its AI calls work without a key). The generator is a Python/WeasyPrint + python-docx pipeline. The bridge between them is the **export JSON**, whose exact shape is documented below and demonstrated in `samples/sample_export.json`.

---

## Part 1 — The Engine

`engine/visible_trusted_engine.jsx` is a single-file React component. It is designed to run as an **artifact inside Claude**, because it calls the Anthropic API (`https://api.anthropic.com/v1/messages`) with no API key — which only works inside Claude's runtime.

**To use it:** open the `.jsx` as an artifact in Claude, or paste it into a Claude conversation and run it. To share with non-technical people, publish the artifact and send the link.

**What it does:**
- Walks the user through the 7-day Visible &amp; Trusted method plus a "Word You Own" step and a "Proof &amp; Stories" step.
- Challenges every clarity answer in two layers: an instant rule check that flags nothing-words, and an AI strategist that judges each answer *specific / distinct / true* and pushes back.
- Hard-gates the two that matter most: the positioning statement and the word to own.
- Distils everything into a brief, shown in the app and exportable as JSON.

**Running it outside Claude** (e.g. on a public website) requires a small backend proxy holding an Anthropic API key, because browsers cannot call the Anthropic API directly. That deployment is not included here; see the handoff doc for the recommended approach.

---

## Part 2 — The Brief Generator

Turns an Engine export into two deliverables in The Brand Company style:
- `TBC_<Name>_PersonalBrandBrief.pdf` — print-ready, full Swiss styling
- `TBC_<Name>_PersonalBrandBrief.docx` — editable Word version for collaborators

### Setup

```bash
cd generator
pip install -r requirements.txt
```

### Run

```bash
# uses the bundled sample export, writes both files into the current folder
python3 build_tbc_brief.py

# or point it at a real export and name the person
python3 build_tbc_brief.py /path/to/visible-trusted-brief-data.json "Real Name"
```

`build_tbc_brief.py` writes the PDF and then calls `build_brief_docx.py` to write the Word version. You can also run the Word builder on its own:

```bash
python3 build_brief_docx.py /path/to/export.json "Real Name"
```

> **Note on the AI synthesis.** The rich brief content (bios, sound bites, talking points, angles) is produced by the Engine during its "distil" step and stored in the export's `distill` object. The generator only formats what is already there. If you build a brief from an export whose `distill` section is empty, the generator will produce a thinner document. Generate the brief inside the Engine first, then export.

---

## The export JSON shape

The Engine's "Export for brief" button downloads `visible-trusted-brief-data.json`. The generator reads two top-level keys: `a` (the raw answers) and `distill` (the synthesised brief content). See `samples/sample_export.json` for a complete, working example. The shape, abbreviated:

```jsonc
{
  "a": {
    "direction": "clients | reputation",
    "profile": { "field": "", "goal": "", "audience": "", "where": "" },
    "d1": { "belief": "", "pattern": "" },
    "d2": { "deepdive": "" },
    "d3": { "final": "" },
    "own": { "word": "", "why": "" },
    "d4": { "throughline": "", "b1": "", "b2": "", "b3": "" },
    "d5": { "platform": "", "rhythm": "", "p1": "", "p2": "", "p3": "" },
    "proof": { "credibility": "", "stories": "", "topics_own": "", "topics_avoid": "" }
  },
  "distill": {
    "positioning": "", "ownable_word": "", "ownable_rationale": "", "pull_quote": "",
    "audience": { "line": "", "moment": "", "feel": "", "want": "", "where": "" },
    "message_pillars": [ { "name": "", "covers": "", "angle": "" } ],
    "voice": { "summary": "", "fit": [], "avoid": [] },
    "bios": { "one_line": "", "short": "", "long": "" },
    "talking": { "own": [], "stories": [], "avoid": [] },
    "sound_bites": [],
    "angles": [ { "format": "", "angle": "" } ],
    "first_move": "", "ninety_day": ""
  }
}
```

The generator is defensive: any missing field is simply omitted from the output rather than erroring.

---

## Brand notes

The visual system is The Brand Company's Swiss International Typographic style: black `#0A0A0A`, white, signal red `#E8000F`, with a monospace for labels. The wordmark is set as **solid bold letter-spaced text**, not the supplied outline logo, because that outline pixelates and disappears at small sizes. Full design tokens and the reasoning are in `docs/PROJECT_HANDOFF.md`.

---

## License / ownership

Proprietary to The Brand Company. Internal use.
