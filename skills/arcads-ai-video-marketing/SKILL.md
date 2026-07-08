```markdown
---
name: arcads-ai-video-marketing
description: Generate AI marketing videos and images using Arcads API with Seedance 2.0, Sora 2, Veo 3.1, Kling, Nano Banana, and 37-template static ad library
triggers:
  - create an AI video ad
  - generate arcads video
  - make a seedance video
  - create nano banana image
  - build meta image ad
  - generate ugc video with arcads
  - make pixar style animated ad
  - create AI influencer character sheet
---

# Arcads AI Video Marketing

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## Overview

Arcads is a comprehensive AI video and image generation platform for marketing creatives. This skill pack provides agent-level access to the full Arcads creative stack including:

- **Video models**: Seedance 2.0 (flagship), Sora 2, Veo 3.1, Kling 3.0, Grok Video, OmniHuman, Audio-driven
- **Image models**: Nano Banana 2/Pro/Edit, ChatGPT Image 2
- **Static ad library**: 37 validated Meta image ad templates
- **Multi-step pipelines**: Pixar-style animations, claymation ads, YouTube thumbnails

The API handles creative generation, polling, file management, and cost confirmation across all endpoints.

## Installation

**Prerequisites**:
- Python 3.10+
- Arcads API key from [app.arcads.ai/settings/api](https://app.arcads.ai/settings/api)
- Optional: `ffmpeg`, `jq`, Node.js (for multi-step pipelines)

**Setup**:

```bash
# Clone and setup
git clone https://github.com/krusemediallc/arcads-claude-code.git
cd arcads-claude-code
./scripts/setup.sh
```

The setup script will:
1. Prompt for your Arcads API key
2. Save to `.env` (never committed)
3. Verify API connection
4. Create `MASTER_CONTEXT.md` workspace file

**Manual `.env` configuration**:

```bash
# .env
ARCADS_API_KEY=your_api_key_here
```

## Core API Structure

All Arcads API calls follow this pattern:

```python
import os
import requests
import time

API_BASE = "https://api.arcads.ai"
API_KEY = os.environ["ARCADS_API_KEY"]

headers = {
    "Authorization": f"Bearer {API_KEY}",
    "Content-Type": "application/json"
}

# POST to generate
response = requests.post(
    f"{API_BASE}/v1/endpoint",
    headers=headers,
    json=payload
)

job_id = response.json()["jobId"]

# Poll for completion
while True:
    status_response = requests.get(
        f"{API_BASE}/v1/job/{job_id}",
        headers=headers
    )
    job = status_response.json()
    
    if job["status"] in ["completed", "failed"]:
        break
    
    time.sleep(5)

if job["status"] == "completed":
    video_url = job["videoUrl"]  # or imageUrl
```

## Video Generation

### Seedance 2.0 (Flagship Model)

**Best for**: 4-15s clips with native audio, image-to-video, video-to-video, multiple shot styles

**Text-to-video**:

```python
import requests
import os

headers = {
    "Authorization": f"Bearer {os.environ['ARCADS_API_KEY']}",
    "Content-Type": "application/json"
}

# UGC selfie-style product review
payload = {
    "prompt": """Young woman in modern kitchen, natural lighting from window. 
    She holds [PRODUCT] at chest level, makes eye contact with camera.
    "I stopped buying [COMPETITOR] after trying this."
    iPhone-shot aesthetic, casual delivery, authentic excitement.
    Duration: 12 seconds.""",
    "duration": 12,
    "aspectRatio": "9:16"
}

response = requests.post(
    "https://api.arcads.ai/v1/seedance-2/video",
    headers=headers,
    json=payload
)

job_id = response.json()["jobId"]
```

**Image-to-video** (start from a still):

```python
import base64

# Read reference image
with open("product_still.jpg", "rb") as f:
    image_b64 = base64.b64encode(f.read()).decode()

payload = {
    "prompt": "Woman picks up product, smiles naturally, camera pulls focus",
    "startFrame": image_b64,  # Base64 encoded image
    "duration": 8,
    "aspectRatio": "9:16"
}

response = requests.post(
    "https://api.arcads.ai/v1/seedance-2/video",
    headers=headers,
    json=payload
)
```

**Seedance 2.0 Prompt Formulas** (in `skills/arcads-external-api/prompting/prompt-library/`):

- `seedance-2-ugc.md`: 9-layer UGC formula (iPhone aesthetic, eye-contact breaks)
- `seedance-2-premium-reveal.md`: Dark void, text narrative, no person
- `seedance-2-product-hero.md`: Elemental effects (water, mist, rotation)
- `seedance-2-studio-lookbook.md`: Editorial multi-shot with voiceover
- `seedance-2-feature-walkthrough.md`: Fast-paced demo cuts

### Veo 3.1

**Best for**: Animating existing stills into video with dialogue (UGC stills → video)

```python
payload = {
    "prompt": "Woman delivers testimonial: 'This changed everything for me.' Natural gestures, authentic delivery.",
    "startFrame": image_b64,  # Base64 image
    "duration": 8,
    "aspectRatio": "9:16"
}

response = requests.post(
    "https://api.arcads.ai/v1/veo-3-1/video",
    headers=headers,
    json=payload
)
```

**Dialogue gate**: Always confirm dialogue separately before generating Veo videos.

### Sora 2

**Best for**: Longer text-to-video (up to 20s)

```python
payload = {
    "prompt": "Cinematic product reveal in futuristic environment, hero lighting, slow camera push",
    "duration": 16,  # Auto-calculated from script (~2.5 words/sec)
    "aspectRatio": "16:9"
}

response = requests.post(
    "https://api.arcads.ai/v1/sora2/video",
    headers=headers,
    json=payload
)
```

**Remix existing video**:

```python
payload = {
    "videoUrl": "https://cdn.arcads.ai/original-video.mp4",
    "prompt": "Same scene but at sunset with warmer tones"
}

response = requests.post(
    "https://api.arcads.ai/v1/sora2/remix/video",
    headers=headers,
    json=payload
)
```

### Kling 3.0

**Best for**: B-roll and scene generation

```python
# B-roll clip
payload = {
    "prompt": "Close-up coffee pour in slow motion, steam rising, cafe background blur",
    "duration": 5,
    "aspectRatio": "16:9"
}

response = requests.post(
    "https://api.arcads.ai/v1/b-roll",
    headers=headers,
    json=payload
)

# Scene generation
payload = {
    "prompt": "Modern coworking space, people collaborating, natural light",
    "duration": 8
}

response = requests.post(
    "https://api.arcads.ai/v1/scene",
    headers=headers,
    json=payload
)
```

### Grok Video

```python
payload = {
    "model": "grok-video",
    "prompt": "Product demonstration in clean studio environment",
    "duration": 10
}

response = requests.post(
    "https://api.arcads.ai/v2/videos/generate",
    headers=headers,
    json=payload
)
```

### OmniHuman (Talking Avatar)

```python
payload = {
    "prompt": "Professional woman, business attire, delivers pitch: 'Our platform increases ROI by 300%'",
    "duration": 12
}

response = requests.post(
    "https://api.arcads.ai/v1/omnihuman",
    headers=headers,
    json=payload
)
```

### Audio-Driven (Lip Sync)

```python
with open("voiceover.mp3", "rb") as f:
    audio_b64 = base64.b64encode(f.read()).decode()

payload = {
    "videoUrl": "https://cdn.arcads.ai/base-video.mp4",
    "audioBase64": audio_b64
}

response = requests.post(
    "https://api.arcads.ai/v1/audio-driven",
    headers=headers,
    json=payload
)
```

## Image Generation

### Nano Banana (Character & Product Stills)

**Models**:
- `nano-banana-2`: Default, balanced quality/speed
- `nano-banana`: Nano Banana Pro (Gemini 3 Pro Image) — tighter identity lock
- `nano-banana-edit`: Inpainting

**Create AI influencer (10-image character sheet)**:

```python
# Phase 1: Hero portrait
hero_payload = {
    "model": "nano-banana-2",
    "prompt": """Front-facing portrait, 22-year-old woman with freckles, 
    auburn hair in loose waves, green eyes, warm smile. 
    Natural makeup, cream sweater, golden hour kitchen lighting.
    Sharp focus on face, shallow depth of field.""",
    "aspectRatio": "9:16",
    "quality": "high"
}

hero_response = requests.post(
    "https://api.arcads.ai/v1/nano-banana/image",
    headers=headers,
    json=hero_payload
)

# Poll and get hero image...
hero_url = job["imageUrl"]

# Phase 2: 9 additional angles using hero as reference
angles = [
    "3/4 profile view, same person, same lighting",
    "Side profile, same person, looking off-camera",
    "Close-up, eyes and smile, same person",
    "Laughing expression, same person, same setting",
    # ... etc
]

for angle_prompt in angles:
    angle_payload = {
        "model": "nano-banana-2",
        "prompt": angle_prompt,
        "referenceImages": [hero_url],  # Lock identity to hero
        "aspectRatio": "9:16"
    }
    
    requests.post(
        "https://api.arcads.ai/v1/nano-banana/image",
        headers=headers,
        json=angle_payload
    )
```

**UGC product selfie**:

```python
# Combine character + product + aesthetic references
payload = {
    "model": "nano-banana-2",
    "prompt": """Sofia in her bedroom, holding [PRODUCT] at face level, 
    iPhone selfie angle, natural lighting from window, authentic smile.
    Visible skin texture, slight camera shake blur, casual makeup.""",
    "referenceImages": [
        "https://cdn.arcads.ai/characters/sofia-hero.jpg",  # Character
        "https://cdn.arcads.ai/products/product-photo.jpg",  # Product
        "references/aesthetics/ugc-selfie/sample-1.jpg"     # Style ref
    ],
    "aspectRatio": "9:16"
}
```

**Recreate influencer from reference**:

```python
with open("reference_photo.jpg", "rb") as f:
    ref_b64 = base64.b64encode(f.read()).decode()

payload = {
    "model": "nano-banana",  # Use Pro for tighter likeness
    "prompt": "Same person, similar lighting and pose, holding [PRODUCT]",
    "refImageAsBase64": ref_b64,
    "aspectRatio": "9:16"
}
```

### ChatGPT Image 2

**Best for**: Typography-heavy, UI mimicry, illustration

```python
payload = {
    "model": "gpt-image-2",
    "prompt": "Apple Notes screenshot: handwritten checklist titled 'Why I switched to [PRODUCT]' with 6 checkmarked items",
    "aspectRatio": "1:1",
    "quality": "high"
}

response = requests.post(
    "https://api.arcads.ai/v1/chatgpt/image",
    headers=headers,
    json=payload
)
```

## Static Meta Image Ad Library

**37 validated templates** across three generator skills:

1. **chatgpt-image-ad**: Typography/UI mimicry (Apple Notes, Forbes editorial, Google search, Slack threads, etc.)
2. **nano-banana-image-ad**: Photoreal/lifestyle (lifestyle hero, comparison table, sticky-note flatlay, etc.)
3. **image-ad-clone**: Reverse-engineer any ad into a template

**Location**: `shared/skills/image-ad-prompting/library/`

**Standard workflow**:

```python
# 1. Select template from library
template_path = "shared/skills/image-ad-prompting/library/apple-notes-list.md"

# 2. Read template structure
with open(template_path) as f:
    template = f.read()

# 3. Customize prompt with product details
customized_prompt = template.replace("[PRODUCT]", "Sleep Supplement")
customized_prompt = customized_prompt.replace("[HOOK]", "Why I finally sleep 8 hours")

# 4. Generate via appropriate backend (see OVERVIEW.md for routing)
payload = {
    "model": "gpt-image-2",  # Apple Notes → ChatGPT Image 2
    "prompt": customized_prompt,
    "aspectRatio": "1:1"
}

response = requests.post(
    "https://api.arcads.ai/v1/chatgpt/image",
    headers=headers,
    json=payload
)
```

**Template categories**:
- **UI Screenshots**: Apple Notes, Slack, iMessage, ChatGPT conversation, Google search results
- **Editorial**: Forbes cover, magazine spread, newspaper article, billboard
- **Comparison**: Side-by-side, table format, before/after
- **Novelty**: Weather forecast UI, dating app card, scratch-off ticket, museum exhibit label

**Read first**: `shared/skills/image-ad-prompting/OVERVIEW.md` — decision tree for backend selection, aspect-ratio compatibility, validation workflows.

## Multi-Step Pipelines

### Pixar-Style 3D Animated Ad

**Pipeline**: Cast sheet → ChatGPT Image 2 storyboard → Seedance 2.0 i2v → ffmpeg stitch

```bash
# Requires: ffmpeg, jq
shared/skills/pixar-style-ad/scripts/generate-pixar-ad.sh \
  --product "Coffee Maker" \
  --duration 30 \
  --output output/pixar-ad.mp4
```

**Storyboard generation** (8-beat structure):

```python
beats = [
    "Beat 1: Coffee Maker sits alone on kitchen counter, sad expression",
    "Beat 2: Owner walks in, frustrated with old coffee maker",
    "Beat 3: Coffee Maker's face lights up, idea spark above head",
    # ... etc (8 total)
]

storyboard_frames = []

for i, beat in enumerate(beats):
    prior_refs = storyboard_frames[-2:] if i > 0 else []  # Max 5 refs
    
    payload = {
        "model": "gpt-image-2",
        "prompt": f"{beat}. Pixar 3D animation style, expressive characters, vibrant colors.",
        "referenceImages": prior_refs,  # Identity lock across sequence
        "aspectRatio": "16:9"
    }
    
    response = requests.post(
        "https://api.arcads.ai/v1/chatgpt/image",
        headers=headers,
        json=payload
    )
    
    # Poll, get image URL, append to storyboard_frames
```

**Animate each frame**:

```python
for frame_url in storyboard_frames:
    payload = {
        "prompt": "Gentle character movement, maintain Pixar style consistency",
        "startFrame": frame_url,  # Or base64
        "duration": 4,
        "aspectRatio": "16:9"
    }
    
    requests.post(
        "https://api.arcads.ai/v1/seedance-2/video",
        headers=headers,
        json=payload
    )
```

**Stitch with ffmpeg**:

```bash
# Create concat file
for video in beat_*.mp4; do
  echo "file '$video'" >> concat.txt
done

# Stitch
ffmpeg -f concat -safe 0 -i concat.txt -c copy output.mp4

# Burn captions (optional)
npx hyperframes add-captions output.mp4 captions.srt final.mp4
```

### Claymation Ad

Same 8-beat structure as Pixar, with clay texture prompts:

```python
beat_prompt = """Plasticine clay character with visible fingerprint textures, 
sculpted features, stop-motion aesthetic. [BEAT_DESCRIPTION]"""

# Optional: Add stop-motion judder in post
# ffmpeg -i input.mp4 -vf "fps=12,fps=24" output.mp4
```

### YouTube Thumbnails (5 CTR Formulas)

**Skill**: `generate-youtube-thumbnail`

**Formulas**:
1. Peace sign + branding
2. Real vs AI comparison
3. Terminal/code flow
4. Reaction shock
5. Before/after split

```python
# Lock likeness with 5+ face references
face_refs = [
    "face-front.jpg",
    "face-3-4.jpg",
    "face-profile.jpg",
    "face-expression-1.jpg",
    "face-expression-2.jpg"
]

payload = {
    "model": "nano-banana-2",
    "prompt": """Peace sign gesture, person in center, large bold text overlay 
    '[HOOK]', vibrant background, product in corner, shocked expression""",
    "referenceImages": face_refs,
    "aspectRatio": "16:9",
    "quality": "high"
}

# Generate 6 variations in parallel
for variation in range(6):
    requests.post(
        "https://api.arcads.ai/v1/nano-banana/image",
        headers=headers,
        json={**payload, "seed": variation}
    )
```

### Caption Burn-In Workflow

**Requirements**: `ffmpeg`, `whisper`, `npx hyperframes`

```bash
# 1. Transcribe
whisper input.mp4 --model medium.en --output_format srt

# 2. Burn captions
npx hyperframes add-captions input.mp4 input.srt output.mp4 \
  --style '{"fontSize": 48, "fontFamily": "Arial Black", "color": "#FFFFFF"}'
```

**Python wrapper**:

```python
import subprocess

def burn_captions(video_path, output_path):
    # Transcribe
    subprocess.run([
        "whisper", video_path,
        "--model", "medium.en",
        "--output_format", "srt"
    ])
    
    srt_path = video_path.replace(".mp4", ".srt")
    
    # Burn captions
    subprocess.run([
        "npx", "hyperframes", "add-captions",
        video_path, srt_path, output_path,
        "--style", '{"fontSize": 48, "color": "#FFFFFF"}'
    ])
```

## Job Polling Pattern

All async endpoints return a `jobId`. Poll until completion:

```python
def poll_job(job_id, timeout=300):
    """Poll job status until completed or timeout"""
    import time
    
    headers = {
        "Authorization": f"Bearer {os.environ['ARCADS_API_KEY']}"
    }
    
    start = time.time()
    
    while time.time() - start < timeout:
        response = requests.get(
            f"https://api.arcads.ai/v1/job/{job_id}",
            headers=headers
        )
        
        job = response.json()
        status = job["status"]
        
        print(f"Job {job_id}: {status}")
        
        if status == "completed":
            return {
                "success": True,
                "url": job.get("videoUrl") or job.get("imageUrl"),
                "job": job
            }
        
        elif status == "failed":
            return {
                "success": False,
                "error": job.get("error", "Unknown error")
            }
        
        time.sleep(5)
    
    return {"success": False, "error": "Timeout"}
```

## Cost Confirmation Pattern

Always confirm costs before expensive operations:

```python
def confirm_cost(operation, estimated_cost):
    """Confirm cost with user before proceeding"""
    print(f"\n{'='*60}")
    print(f"Operation: {operation}")
    print(f"Estimated cost: ${estimated_cost:.2f}")
    print(f"{'='*60}")
    
    response = input("Proceed? (yes/no): ").strip().lower()
    
    return response == "yes"

# Usage
if confirm_cost("Generate 10-image character sheet", 2.50):
    # Proceed with generation
    pass
```

## File Organization

**Recommended structure**:

```
project/
├── references/
│   ├── influencers/          # AI character sheets
│   │   ├── sofia/
│   │   │   ├── hero.jpg
│   │   │   ├── 3-4-view.jpg
│   │   │   └── ...
│   ├── products/             # Product photos
│   │   ├── product-1.jpg
│   │   └── product-2.jpg
│   └── aesthetics/           # Style references
│       ├── ugc-selfie/
│       └── premium/
├── output/
│   ├── videos/
│   ├── images/
│   └── campaigns/
└── MASTER_CONTEXT.md         # Workspace context
```

## Configuration

**Environment variables** (`.env`):

```bash
# Required
ARCADS_API_KEY=your_api_key

# Optional - Meta Marketing API (for meta-ad-builder skill)
META_ACCESS_TOKEN=your_meta_token
META_AD_ACCOUNT_ID=act_123456789

# Optional - ElevenLabs (for voiceover)
ELEVENLABS_API_KEY=your_elevenlabs_key
```

**MASTER_CONTEXT.md** (workspace state):

```markdown
# Project Context

## Current Campaign
- Product: [name]
- Target audience: [description]
- Key messages: [list]

## AI Influencers
- Sofia: 22yo college student, freckles, auburn hair
  - Hero: references/influencers/sofia/hero.jpg
  - Angles: 9 additional views

## Active Workflows
- [ ] Generate 5 UGC videos (Seedance 2.0)
- [ ] Create 10 static image ads (Meta library)
- [ ] Build Pixar-style 30s animated ad
```

## Common Patterns

### Pattern: Character-Locked UGC Video Series

Generate 5 UGC videos with same character:

```python
# 1. Generate character still
character_payload = {
    "model": "nano-banana-2",
    "prompt": "Woman, 28, casual style, natural makeup, bright home setting",
    "aspectRatio": "9:16"
}

char_response = requests.post(
    "https://api.arcads.ai/v1/nano-banana/image",
    headers=headers,
    json=character_payload
)

# Poll and get character URL
char_url = poll_job(char_response.json()["jobId"])["url"]

# 2. Generate 5 videos using character as startFrame
scripts = [
    "This product changed my morning routine",
    "Here's why I switched from [competitor]",
    "Three things I love about this",
    "My honest review after 30 days",
    "Why everyone's talking about this"
]

for script in scripts:
    video_payload = {
        "prompt": f"Woman delivers testimonial: '{script}'. Natural gestures, authentic delivery.",
        "startFrame": char_url,  # Lock to same character
        "duration": 10,
        "aspectRatio": "9:16"
    }
    
    requests.post(
        "https://api.arcads.ai/v1/veo-3-1/video",
        headers=headers,
        json=video_payload
    )
```

### Pattern: Product Reveal (Still → Video)

```python
# 1. Generate product hero still
still_payload = {
    "model": "nano-banana-2",
    "prompt": "Premium product on dark surface, dramatic side lighting, shallow DOF",
    "aspectRatio": "9:16"
}

still_url = poll_job(
    requests.post(
        "https://api.arcads.ai/v1/nano-banana/image",
        headers=headers,
        json=still_payload
    ).json()["jobId"]
)["url"]

# 2. Animate into reveal
video_payload = {
    "prompt": "Slow 360 rotation, light sweeps across product, mist rises from base",
    "startFrame": still_url,
    "duration": 8,
    "aspectRatio": "9:16"
}

requests.post(
    "https://api.arcads.ai/v1/seedance-2/video",
    headers=headers,
    json=video_payload
)
```

### Pattern: A/B Test Creative Variations

Generate 3 variations of same concept:

```python
base_prompt = "Woman in kitchen, holding product, testimonial delivery"

variations = [
    f"{base_prompt}. Bright natural lighting, casual dress.",
    f"{base_prompt}. Warm golden hour lighting, athleisure.",
    f"{base_prompt}. Soft window light, cozy sweater."
]

job_ids = []

for var_prompt in variations:
    response = requests.post(
        "https://api.arcads.ai/v1/seedance-2/video",
        headers=headers,
        json={
            "prompt": var_prompt,
            "duration": 10,
            "aspectRatio": "9:16"
        }
    )
    job_ids.append(response.json()["jobId"])

# Poll all jobs
results = [poll_job(jid) for jid in job_ids]
```

## Troubleshooting

### Job fails with "prompt too complex"

**Solution**: Simplify prompt, remove excessive detail. Seedance 2.0 prompts should be ~50-150 words.

```python
# Too complex
"Young woman, 22 years old, with freckles on her nose and cheeks, ..."

# Better
"Woman with freckles, natural look, casual style"
```

### Identity drift across multi-image sequences

**Solution**: Use `referenceImages` with prior frames (max 5 for ChatGPT Image 2):

```python
payload = {
    "prompt": "Same person, next scene",
    "referenceImages": prior_frames[-2:],  # Last 2 frames
    "model": "gpt-image-2"
}
```

For Nano Banana, use `nano-banana` (Pro) model for tighter identity lock.

### Video generation times out

**Solution**: Increase timeout, check job status manually:

```bash
curl -H "Authorization: Bearer $ARCADS_API_KEY" \
  https://api.arcads.ai/v1/job/{jobId}
```

Typical generation times:
- Seedance 2.0: 3-8 minutes
- Sora 2: 5-12 minutes
- Veo 3.1: 4-10 minutes
- Nano Banana: 30-90 seconds

### Audio/dialogue not matching video

**Solution**: For Veo 3.1, always include dialogue in prompt explicitly:

```python
payload = {
    "prompt": """Woman says: 'This is the exact line of dialogue.'
    Natural lip sync, appropriate gestures for the words.""",
    "startFrame": image_url
}
```

### Static ad template renders incorrectly

**Solution**: Check aspect ratio compatibility in `shared/skills/image-ad-prompting/OVERVIEW.md`. Some templates only work with specific backends:

- Apple Notes, Slack → ChatGPT Image 2, 1:1 or 4:5
- Lifestyle hero → Nano Banana 2, 9:16
- Comparison table → ChatGPT Image 2, 1:1

### ffmpeg stitch has frame rate mismatches

**Solution**: Re-encode all clips to same specs before concat:

```bash
for f in beat_*.mp4; do
  ffmpeg -i "$f" -r 30 -c:v libx264 -preset fast -crf 23 \
    -vf scale=1080:1920 "normalized_$f"
done
```

## Quick Reference

**Video Endpoints**:
- `POST /v1/seedance-2/video` — Flagship, 4-15s, i2v/t2v
- `POST /v1/veo-3-1/video` — Animate stills with dialogue
- `POST /v1/sora2/video` — Long-form t2v (up to 20s)
- `POST /v1/sora2/remix/video` — Remix existing video
- `POST /v1/b-roll` — Kling 3.0 b-roll
- `POST /v1/scene` — Kling 3.0 scene
- `POST /v2/videos/generate` — Grok Video

**Image Endpoints**:
- `POST /v1/nano-banana/image` — Character/product stills
- `POST /v1/chatgpt/image` — Typography/UI/illustration

**Job Management**:
- `GET /v1/job/{jobId}` — Poll job status

**Aspect Ratios**:
- `9:16` — Vertical (Instagram Reels, TikTok)
- `16:9` — Horizontal (YouTube)
- `1:1` — Square (Meta feed)
- `4:5` — Vertical feed (Meta/Instagram)

**Cost Estimates**:
- Seedance 2.0: ~$0.20-0.40/video
- Nano Banana 2: ~$0.10-0.20/image
- ChatGPT Image 2: ~$0.15-0.25/image
- Veo 3.1: ~$0.25-0.50/video

**Key Files**:
- `.env` — API keys (never commit)
- `MASTER_CONTEXT.md` — Workspace context
- `shared/skills/image-ad-prompting/library/` — 37 static ad templates
- `skills/arcads-external-api/prompting/prompt-library/` — Video prompt formulas

**Documentation**:
- Main guide: `skills/arcads-external-api/GUIDE.
