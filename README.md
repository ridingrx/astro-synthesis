# 🔮 Astro-Synthesis — Claude Skill

A multi-tradition astrology skill for Claude that synthesizes **Western**, **Vedic/Jyotish**, **Chinese BaZi**, and **Numerology** (Pythagorean + Chaldean) into a single, non-contradictory reading — with real remedies.

Built by **ridingrx** ([@ridingrx](https://github.com/ridingrx))

---

## Why This Skill Exists

Most AI astrology tools do one of two things:
- Give you vague sun-sign fluff with zero specificity
- Give you parallel reports from multiple traditions that directly contradict each other — and leave you confused

This skill is built around a different principle: **every tradition is a different lens on the same person, not a competing report.** When Vedic says one thing and numerology says another, there's always a reason — and a synthesis. This skill finds that synthesis every time.

It also includes something almost no AI astrology tool does: **actual remedies.** Mantras, gemstones, fasting protocols, havan instructions (including anar dana in agni for releasing blockages), BaZi elemental adjustments, Western psychological integration practices. Because knowing your chart without knowing what to *do* with it is only half the work.

---

## What It Covers

- **Natal chart interpretation** — planets, houses, aspects, dignity, yogas
- **Transit readings** — current planetary weather and how it activates your chart
- **Dasha/Antardasha** — Vedic predictive timing
- **BaZi luck pillars** — Chinese 10-year cycle analysis
- **Synastry** — compatibility between two charts
- **Specific placement deep-dives** — "what does Moon in Scorpio in the 8th house mean"
- **Quick lookups** — "what does Venus square Saturn mean"
- **Remedies** — Vedic upayas, Western integration practices, BaZi elemental adjustments, numerological alignment
- **Conflict resolution** — when traditions say opposite things, explains why and synthesizes

---

## Skill Structure

```
astro-synthesis/
├── SKILL.md                          # Main skill file (loads into Claude)
├── README.md                         # This file
├── LICENSE                           # MIT License
└── references/
    ├── western.md                    # Western astrology framework
    ├── vedic.md                      # Vedic/Jyotish framework
    ├── chinese-bazi.md               # BaZi (Four Pillars) framework
    ├── numerology.md                 # Pythagorean + Chaldean numerology
    ├── remedies.md                   # Full remedy library across traditions
    └── synthesis-protocol.md        # How to reconcile conflicting traditions
```

---

## How to Install

### On Claude.ai
1. Download `astro-synthesis.skill` from the [Releases](../../releases) page
2. Go to **Claude.ai → Settings → Skills**
3. Click **Install from file** and select the `.skill` file
4. The skill will now appear in Claude's available skills

### Manual Install (Claude Code or custom setup)
Clone this repo and point Claude to the `SKILL.md` file directly.

```bash
git clone https://github.com/ridingrx/astro-synthesis
```
## Requirements (for chart calculator)

```bash
pip install pyswisseph geopy timezonefinder pytz
```

---

## Calculate a Chart

Once dependencies are installed, run the calculator directly from your terminal:

```bash
python scripts/calculate_chart.py \
  --date "1994-11-19" \
  --time "09:45" \
  --lat 21.1458 \
  --lon 79.0882 \
  --tz "Asia/Kolkata" \
  --name "Your Full Birth Name"
```

If you don't know your coordinates, pass `--place` instead:

```bash
python scripts/calculate_chart.py \
  --date "1994-11-19" \
  --time "09:45" \
  --place "Nagpur, India"
```

The script outputs your complete Western, Vedic, BaZi, Dasha, and Numerology chart in one go. Paste the output into Claude with the skill active for a full interpretation.

For raw JSON output (useful for developers):
```bash
python scripts/calculate_chart.py --date "..." --time "..." --place "..." --json
```
---

## How to Use

Once installed, just talk to Claude naturally:

> "Do a full birth chart reading for me — DOB 19 November 1994, 9:45 AM, Nagpur India"

> "What does Moon in Scorpio in the 8th house mean across traditions?"

> "My Vedic chart says X but my numerology says Y — which one is right?"

> "What remedies should I do for Saturn Mahadasha?"

> "Do a synastry reading between [person A details] and [person B details]"

For best results on a natal reading, calculate your chart first:
- **Western**: [astro.com](https://astro.com) → Free Horoscopes → Extended Chart Selection → Placidus
- **Vedic**: [astrosage.com](https://astrosage.com) or Jagannatha Hora (free desktop app)
- **BaZi**: [Joey Yap's calculator](https://bazi.masteryacademy.com)

Then paste the chart data into Claude with the skill active.

---

## Accuracy Philosophy

This skill uses:
- **Actual orb rules** (not generic "planets affect each other" handwaving)
- **Dignity and debility** assessment (domicile, exaltation, fall, combustion)
- **Neecha bhanga** — cancellation of debilitation (one of the most important and commonly missed Jyotish calculations)
- **Full 10 Gods system** for BaZi (not just animal sign)
- **Both Pythagorean AND Chaldean** numerology (with an explanation of why they differ)
- **Vimshottari Dasha** for Vedic timing
- **Nakshatra-level specificity** (two people with Moon in Scorpio behave very differently based on nakshatra)

It does **not** give vague statements like "You are creative and intuitive." It gives specific placements with specific meanings and specific things to do.

---

## The Synthesis Protocol

The core differentiator. When traditions appear to contradict each other, the skill classifies the conflict as one of four types:

1. **Different timeframes** — one system is describing a 10-year chapter, another a 3-month window
2. **Different life domains** — one system is addressing career, another intimate partnership
3. **Different layers of the same truth** — the what vs. the how
4. **Genuine internal tension** — the person actually carries this tension; both traditions are correct

Each type resolves differently. No reading ends with unresolved contradictions.

---

## Contributing

Found a missing remedy? A tradition that should be added? An error in a planetary dignity table?

PRs welcome. Please keep the accuracy standard high — cite sources for any corrections to traditional astrology content.

---

## License

MIT License — free to use, modify, and distribute with attribution.

---

## Author

**Ridingrx**
Head of Marketing & Strategy | Physiotherapist | Biker | Builder

*"Most astrology content tells you what your chart says. This one tells you what to do about it."*
