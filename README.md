# One product photo. One prompt. One finished UGC ad.

Claude Code skill that generates complete multi-shot UGC video ads using Seedance 2.0 on fal.ai.

Drop in a product image, pick an ad angle, and it writes the dialogue, fires each shot to the API in parallel, downloads the clips, and stitches them into one ready-to-upload vertical ad.

Built for DTC brands and agencies paying $150-$300 per creator video and waiting days for footage.

---

## What's in the repo

- `SKILL.md` - the Claude Code skill. Install it once and run it with `/seedance-ugc-ads` in any project.
- `scripts/generate.py` - parallel multi-shot generator (up to 5 concurrent shots, auto cost estimate)
- `scripts/stitch.sh` - ffmpeg concat to 1080x1920 vertical
- `references/angles.md` - detailed shot-by-shot breakdowns for all 6 preset angles
- `references/fal-api.md` - fal.ai API schemas, pricing, auth, parallel generation pattern
- `references/prompting.md` - how to write Seedance prompts that actually work

## What the skill eliminates

- Writing creator briefs
- Waiting for footage
- $200/video platform fees
- Revision rounds

## What the pipeline does

1. Takes your product image + ad angle
2. Writes a multi-shot UGC script with dialogue, shot direction, and pacing
3. Shows you the shot list for approval before touching the API
4. Uploads your product image to fal.ai
5. Fires shots in parallel
6. Downloads clips into an organized folder
7. Stitches clips into one finished vertical ad

## The 6 preset angles

| Angle | Shots | Best for |
|---|---|---|
| `unboxing` | 3 | Products with strong packaging or reveal appeal |
| `testimonial` | 4 | Supplements, skincare, before/after health claims |
| `lifestyle-demo` | 5 | Apparel, accessories, home goods |
| `problem-solution` | 4 | Products solving a specific nameable problem |
| `before-after` | 3 | Visual transformations: skincare, hair, fitness |
| `hold-and-show` | 3 | Makeup, skincare application, small packaged products |

Freeform also works. Describe the angle and the skill infers shot count and structure.

## Cost estimate

| Config | Per 5-sec clip | 3-shot ad | 5-shot ad |
|---|---|---|---|
| 720p, image/text reference | $1.51 | ~$4.50 | ~$7.50 |
| 720p, video reference (shots 2+) | $0.91 | ~$2.90 | ~$4.70 |

Using character consistency (passing shot 1's output as reference for shots 2+) is both higher quality AND 40% cheaper. The skill uses this pattern by default for multi-shot ads.

Budget $10 in fal.ai credits to start. Check current rates at [fal.ai/models/bytedance/seedance-2.0](https://fal.ai/models/bytedance/seedance-2.0/image-to-video) - they update occasionally.

---

## Prerequisites

1. **Claude Code** - [claude.ai/code](https://claude.ai/code)
2. **fal.ai account with API key** - [fal.ai](https://fal.ai) -> Settings -> API Keys -> Create key. Load at least $10 in credits.
3. **Python 3.8+** with:
   ```bash
   pip install fal-client python-dotenv requests
   ```
4. **ffmpeg**:
   - macOS: `brew install ffmpeg`
   - Ubuntu/Debian: `sudo apt install ffmpeg`
   - Windows: [ffmpeg.org](https://ffmpeg.org)

---

## Setup

**1. Clone the repo:**
```bash
git clone https://github.com/iliyanivanovmp-stack/seedance-ugc-ad-generator
cd seedance-ugc-ad-generator
```

**2. Add your fal.ai key:**
```bash
cp .env.example .env
# open .env and paste your FAL_KEY
```

**3. Install the Claude Code skill:**
```bash
mkdir -p ~/.claude/skills/seedance-ugc-ads
cp SKILL.md ~/.claude/skills/seedance-ugc-ads/SKILL.md
cp -r scripts references ~/.claude/skills/seedance-ugc-ads/
```

**4. Restart Claude Code.** The skill registers as `/seedance-ugc-ads` in the slash menu.

---

## Running the skill

In Claude Code, type:
```
/seedance-ugc-ads
```

You'll be asked for:
- Product image path (required)
- Ad angle - pick one of the presets or describe freeform
- Product name and one-line description (optional, sharpens the dialogue)
- Duration per clip (default: 5 seconds)

The skill shows you the full shot list and dialogue for approval before firing to the API. Revising dialogue here is free. Regenerating clips is not.

---

## Running without the skill (scripts only)

If you want to use the scripts directly without installing the skill:

**1. Create a manifest:**
```json
{
  "project_name": "my-product-ad",
  "output_dir": "outputs/my-product-ad",
  "shots": [
    {
      "shot_number": 1,
      "mode": "image_to_video",
      "prompt": "...",
      "image_url": "<fal-uploaded-url>",
      "duration": "5",
      "resolution": "720p",
      "aspect_ratio": "9:16",
      "generate_audio": true
    }
  ]
}
```

**2. Generate clips:**
```bash
python scripts/generate.py --manifest outputs/my-product-ad/manifest.json
```

**3. Stitch:**
```bash
./scripts/stitch.sh outputs/my-product-ad/clips outputs/my-product-ad/final-ad.mp4
```

See `references/fal-api.md` for the full manifest schema and parameter reference.

---

## Output structure

```
outputs/<project>/
├── shot-list.md       # approved script
├── manifest.json      # prompts, results, cost
├── clips/
│   ├── shot-1.mp4
│   ├── shot-2.mp4
│   └── shot-N.mp4
└── final-ad.mp4       # stitched, ready to upload
```

---

## Honest limits

- Text rendering on small packaging garbles below 720p. Use 720p or 1080p if packaging text matters.
- Character consistency across shots requires passing shot 1's output as `@Video1` in reference-to-video mode. Just passing the product image is not enough.
- Makeup/skincare application angles require restrictive prompting. For high-risk categories, `hold-and-show` is more reliable. See `references/prompting.md` for the restrictive negation pattern.
- Seedance generates dialogue audio. It's solid but not always perfect. Set `generate_audio: false` and dub with ElevenLabs afterward if you need precise voice control.

---

## Questions

`iliyan.ivanov.mp@gmail.com` · [aiessentials.us](https://aiessentials.us)

If you want this kind of AI system built into your operations end-to-end, the [AI Operating System](https://aiessentials.us/ai-operating-system) is how we do it.
