---
name: arcads-ai-video-generation
description: Create AI marketing videos and static image ads using the Arcads API with agent-powered workflows for Seedance 2.0, Sora 2, Veo 3.1, Kling 3.0, Nano Banana, and 37 Meta image-ad templates
triggers:
  - "generate an Arcads video"
  - "create a UGC selfie video with Seedance"
  - "make a Nano Banana product image"
  - "build a Pixar-style animated ad"
  - "create AI influencer character sheet"
  - "generate Meta image ad from template"
  - "animate this still with Veo"
  - "make a claymation ad with Arcads"
---

# Arcads AI Video Generation

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

Arcads is an AI video and image generation platform for creating marketing creative. This skill provides agent-powered workflows for the full Arcads creative stack: **Seedance 2.0** (flagship video model), **Sora 2**, **Veo 3.1**, **Kling 3.0**, **Grok Video**, **Nano Banana 2/Pro/Edit**, **ChatGPT Image 2**, **OmniHuman**, **Audio-driven**, plus 37 validated static Meta image-ad templates and multi-step pipelines for Pixar-style and claymation animated ads.

## Installation

### 1. Clone and setup

```bash
git clone https://github.com/krusemediallc/arcads-claude-code.git
cd arcads-claude-code
./scripts/setup.sh
```

The setup script will:
- Prompt for your Arcads API key (get it at [app.arcads.ai/settings/api](https://app.arcads.ai/settings/api))
- Create `.env` file with your credentials
- Verify API connection
- Create `MASTER_CONTEXT.md` workspace file

### 2. Prerequisites

**Core requirements:**
- Python 3.10+
- Arcads API key

**Optional (for multi-step pipelines):**
```bash
# macOS
brew install ffmpeg jq node

# Linux
apt install ffmpeg jq nodejs python3

# Python packages
pip install openai-whisper
pip install -r shared/skills/meta-ad-builder/scripts/requirements.txt
```

### 3. Environment configuration

Create `.env` in the project root:

```bash
ARCADS_API_KEY=your_api_key_here
ARCADS_API_BASE=https://api.arcads.ai
```

## Core API patterns

### Base request structure

All Arcads API calls follow this pattern:

```python
import requests
import os
import time

API_KEY = os.getenv('ARCADS_API_KEY')
API_BASE = os.getenv('ARCADS_API_BASE', 'https://api.arcads.ai')

headers = {
    'Authorization': f'Bearer {API_KEY}',
    'Content-Type': 'application/json'
}

# Make request
response = requests.post(f'{API_BASE}/v1/endpoint', headers=headers, json=payload)
job_id = response.json()['jobId']

# Poll for completion
while True:
    status_response = requests.get(f'{API_BASE}/v1/jobs/{job_id}', headers=headers)
    status_data = status_response.json()
    
    if status_data['status'] == 'completed':
        video_url = status_data['outputUrl']
        break
    elif status_data['status'] == 'failed':
        raise Exception(f"Job failed: {status_data.get('error')}")
    
    time.sleep(10)
```

## Video generation workflows

### Seedance 2.0 (flagship model)

**Best for:** UGC product reviews, premium product reveals, studio lookbooks, feature demos (4-15s clips with native audio)

```python
# UGC selfie-style product review
payload = {
    "prompt": """
    [SCENE SETUP]
    iPhone selfie-style. Woman, mid-20s, kitchen counter background, natural window light.
    Holding [product] at chest level, label facing camera.
    
    [PERFORMANCE DIRECTION]
    Casual, authentic delivery. Eye contact 70% of time, natural breaks to glance at product.
    Hand gestures: open palm emphasis on "this", light product tap on key features.
    
    [DIALOGUE]
    "Okay so I stopped buying [competitor] because this literally changed everything.
    The [feature] is insane and it's like half the price. I'm obsessed."
    
    [AESTHETIC]
    Slight iPhone grain, natural color grade, realistic skin texture with minor blemishes.
    Casual home environment, not overly staged.
    """,
    "duration": 12,
    "model": "seedance-2",
    "aspectRatio": "9:16"
}

response = requests.post(f'{API_BASE}/v1/seedance2/video', headers=headers, json=payload)
```

**Premium product reveal (no person):**

```python
payload = {
    "prompt": """
    [SCENE]
    Pure black void. [Product] suspended center frame, dramatic spotlight from above.
    
    [CAMERA]
    Slow 360° orbit. Start front, rotate clockwise 8s total.
    
    [TEXT OVERLAYS]
    0-3s: "THE PROBLEM" (fade in top-left)
    3-6s: "EVERYONE ELSE'S SOLUTION" (fade in bottom-right)  
    6-9s: "OUR ANSWER" (fade in center, larger)
    9-12s: Product name and logo (fade in bottom-center)
    
    [LIGHTING]
    High-contrast. Key light top, rim light backside creates glow edge.
    Mist/atmosphere catches light rays.
    
    [AESTHETIC]
    Premium, minimal, Apple-keynote inspired.
    """,
    "duration": 12,
    "model": "seedance-2",
    "aspectRatio": "9:16"
}
```

**Image-to-video with Seedance 2.0:**

```python
import base64

# Load starting image
with open('product_still.png', 'rb') as f:
    image_b64 = base64.b64encode(f.read()).decode('utf-8')

payload = {
    "prompt": "Woman picks up product, examines it, smiles at camera and nods approvingly",
    "startFrame": image_b64,
    "duration": 8,
    "model": "seedance-2",
    "aspectRatio": "9:16"
}

response = requests.post(f'{API_BASE}/v1/seedance2/video', headers=headers, json=payload)
```

### Sora 2 (text-to-video, up to 20s)

```python
payload = {
    "prompt": "Aerial drone shot ascending over misty mountain valley at golden hour. Camera rises through low clouds revealing peaks bathed in warm sunlight. Cinematic, 4K quality.",
    "duration": 16,
    "aspectRatio": "16:9"
}

response = requests.post(f'{API_BASE}/v1/sora2/video', headers=headers, json=payload)
```

**Sora 2 with style reference:**

```python
with open('style_reference.jpg', 'rb') as f:
    style_ref_b64 = base64.b64encode(f.read()).decode('utf-8')

payload = {
    "prompt": "Product floating in ethereal space with particle effects",
    "duration": 12,
    "styleReference": style_ref_b64,
    "aspectRatio": "1:1"
}
```

### Veo 3.1 (start-frame animation)

**Best for:** Animating UGC stills with dialogue, bringing Nano Banana images to life

```python
# MANDATORY: Confirm dialogue separately before generating
dialogue = "This product literally saved my morning routine. Game changer."

# Generate video with start frame and dialogue
with open('ugc_selfie_still.png', 'rb') as f:
    start_frame_b64 = base64.b64encode(f.read()).decode('utf-8')

payload = {
    "startFrame": start_frame_b64,
    "prompt": f"""
    Natural head movement, slight smile, eyes make contact with camera then glance at product.
    Hand gestures: open palm on "saved", point to product on "this".
    Authentic UGC delivery, not overly performative.
    """,
    "dialogue": dialogue,
    "duration": 8,
    "aspectRatio": "9:16"
}

response = requests.post(f'{API_BASE}/v1/veo3/video', headers=headers, json=payload)
```

### Kling 3.0 (b-roll and scene generation)

```python
# B-roll endpoint
payload = {
    "prompt": "Close-up of coffee being poured into white ceramic mug, steam rising, warm morning light",
    "duration": 5,
    "aspectRatio": "16:9"
}

response = requests.post(f'{API_BASE}/v1/b-roll', headers=headers, json=payload)

# Scene endpoint  
payload = {
    "prompt": "Cozy coffee shop interior, wooden tables, plants by window, soft afternoon light filtering through, indie cafe atmosphere",
    "duration": 8,
    "aspectRatio": "16:9"
}

response = requests.post(f'{API_BASE}/v1/scene', headers=headers, json=payload)
```

### OmniHuman (talking avatar)

```python
payload = {
    "personDescription": "Professional woman, 30s, business casual, neutral office background",
    "script": "Welcome to our Q4 product update. Today I'm excited to share three major features that our team has been working on.",
    "duration": 15,
    "aspectRatio": "16:9"
}

response = requests.post(f'{API_BASE}/v1/omnihuman', headers=headers, json=payload)
```

## Image generation workflows

### Nano Banana (AI influencer and product stills)

**Create new AI influencer (10-image character sheet):**

```python
# Phase 1: Generate hero portrait
hero_payload = {
    "prompt": """
    Front-facing portrait, centered, eye contact with camera.
    
    CHARACTER:
    Woman, 22, college student aesthetic. Light freckles across nose and cheeks.
    Warm brown eyes, shoulder-length wavy auburn hair with natural highlights.
    Genuine smile showing slight tooth gap (adds authenticity).
    
    SETTING:
    Her kitchen, golden hour light through window (soft, warm, directional from left).
    Blurred background: white subway tile, plants on windowsill.
    
    CLOTHING:
    Cream oversized knit sweater, casual, lived-in.
    
    CAMERA:
    iPhone 14 selfie aesthetic. Slight skin texture, natural pores visible.
    Warm color grade, slightly underexposed for authentic feel.
    """,
    "model": "nano-banana-2",
    "aspectRatio": "9:16",
    "outputFormat": "png"
}

response = requests.post(f'{API_BASE}/v1/images/generate', headers=headers, json=hero_payload)
hero_job_id = response.json()['jobId']

# Poll and save hero
# ... (polling logic)

# Phase 2: Generate 9 additional angles using hero as reference
with open('hero_portrait.png', 'rb') as f:
    hero_ref_b64 = base64.b64encode(f.read()).decode('utf-8')

angles = [
    "3/4 view from right side, looking slightly off-camera left",
    "Profile view from left side, hair tucked behind ear",
    "Close-up from chest up, laughing genuinely",
    "Overhead angle, looking up at camera with surprised expression",
    "Low angle from below, confident expression",
    "3/4 view from left side, holding coffee mug",
    "Full body shot standing, same outfit, relaxed pose",
    "Close-up of face, serious/thoughtful expression",
    "Candid looking away from camera, natural moment"
]

for i, angle_prompt in enumerate(angles):
    payload = {
        "prompt": f"{angle_prompt}. MAINTAIN: Same person, same freckle pattern, same eye color, same hair texture from reference.",
        "referenceImages": [hero_ref_b64],
        "model": "nano-banana-2",
        "aspectRatio": "9:16"
    }
    # Generate and save to references/influencers/[name]/
```

**UGC product selfie still:**

```python
# Load character reference + product photo + aesthetic references
with open('references/influencers/sofia/hero.png', 'rb') as f:
    character_ref_b64 = base64.b64encode(f.read()).decode('utf-8')

with open('product_photo.png', 'rb') as f:
    product_b64 = base64.b64encode(f.read()).decode('utf-8')

with open('references/aesthetics/ugc-selfie/style1.png', 'rb') as f:
    style_ref_b64 = base64.b64encode(f.read()).decode('utf-8')

payload = {
    "prompt": """
    iPhone selfie in bedroom. Sofia holding [product] at chest level, label facing camera.
    Sitting on edge of bed, natural window light from right.
    
    Genuine smile, eye contact with camera. Hair slightly messy (authentic).
    Wearing casual tank top.
    
    Background: blurred bedroom, fairy lights, poster on wall.
    
    REALISM DETAILS:
    - Slight iPhone grain/noise
    - Natural skin texture, visible pores on nose/forehead
    - Minor blemish on left cheek (adds authenticity)
    - Warm but not oversaturated color grade
    - Slightly soft focus (characteristic of front-facing phone camera)
    """,
    "referenceImages": [character_ref_b64, product_b64, style_ref_b64],
    "model": "nano-banana-2",
    "aspectRatio": "9:16"
}

response = requests.post(f'{API_BASE}/v1/images/generate', headers=headers, json=payload)
```

**Nano Banana Pro (higher fidelity, tighter identity lock):**

```python
payload = {
    "prompt": "Same character description...",
    "referenceImages": [ref1_b64, ref2_b64],
    "model": "nano-banana",  # Pro model
    "aspectRatio": "9:16"
}
```

### ChatGPT Image 2 (typography/UI-heavy creatives)

```python
# Apple Notes style ad
payload = {
    "prompt": """
    iPhone Notes app screenshot.
    
    Top: "Shopping List" in Notes app header font
    
    List items (with strikethrough on completed):
    ☑︎ Expensive gym membership
    ☑︎ Meal prep containers I never use  
    ☑︎ Supplements that don't work
    ☐ [Your Product] ← Actually worth it
    
    Bottom timestamp: "Edited 2 hours ago"
    
    Authentic iOS Notes aesthetic: SF font, cream background, correct UI spacing.
    """,
    "model": "gpt-image-2",
    "aspectRatio": "9:16",
    "outputFormat": "png"
}

response = requests.post(f'{API_BASE}/v1/images/generate', headers=headers, json=payload)
```

## Static Meta image ad library

The repo includes 37 validated prompt templates in `shared/skills/image-ad-prompting/prompt-library/`:

**Decision tree:**
- Typography-heavy / UI mimicry → `chatgpt-image-ad` skill (gpt-image-2)
- Photoreal / lifestyle / multi-reference → `nano-banana-image-ad` skill (Nano Banana 2/Pro)
- Reverse-engineer existing ad → `image-ad-clone` skill (backend-agnostic)

**Template examples:**

```python
# Apple Notes list (chatgpt-image-ad)
# Template: shared/skills/image-ad-prompting/prompt-library/chatgpt-image/apple-notes-list.md

# Forbes editorial hero (nano-banana-image-ad)
# Template: shared/skills/image-ad-prompting/prompt-library/nano-banana/forbes-editorial.md

# Fake Google search (chatgpt-image-ad)
# Template: shared/skills/image-ad-prompting/prompt-library/chatgpt-image/fake-google-search.md

# Comparison table (chatgpt-image-ad)
# Template: shared/skills/image-ad-prompting/prompt-library/chatgpt-image/comparison-table.md

# Sticky note flatlay (nano-banana-image-ad)
# Template: shared/skills/image-ad-prompting/prompt-library/nano-banana/sticky-note-flatlay.md
```

**Standard generate workflow:**

```bash
# Read OVERVIEW.md first
cat shared/skills/image-ad-prompting/OVERVIEW.md

# Generate from template
python shared/skills/chatgpt-image-ad/generate.py \
  --template apple-notes-list \
  --product "Your Product Name" \
  --aspect-ratio 9:16
```

**Clone existing ad:**

```bash
# Reverse-engineer ad image into new template
python shared/skills/image-ad-clone/clone.py \
  --input existing_ad.png \
  --backend nano-banana-2 \
  --output-name my-custom-template
```

## Multi-step animated ad pipelines

### Pixar-style 3D animated ad

**8-beat story arc workflow:**

```python
# Step 1: Define story beats
beats = [
    "Hero character discovers problem (frustrated expression)",
    "Problem escalates (comedic exaggeration)", 
    "Discovery of your product (eyes widen, lightbulb moment)",
    "Character tries product (hopeful, cautious)",
    "Product delivers results (surprised delight)",
    "Character celebrates (joy, relief)",
    "Final transformation (confidence, satisfaction)",
    "Product showcase with branding (hero shot)"
]

# Step 2: Lock character design
character_sheet_prompt = """
Pixar-style 3D character. Anthropomorphized [mascot concept].
Big expressive eyes, exaggerated proportions, appealing silhouette.
Color palette: [primary], [secondary], [accent].
Render: Pixar's RenderMan aesthetic, subsurface scattering on skin, soft global illumination.
"""

# Step 3: Generate storyboard stills via ChatGPT Image 2
reference_images = []
for i, beat in enumerate(beats):
    payload = {
        "prompt": f"""
        Pixar 3D animation still. Beat {i+1}/8.
        
        CHARACTER: {character_sheet_prompt}
        
        ACTION: {beat}
        
        COMPOSITION: Rule of thirds, depth of field, cinematic lighting.
        """,
        "referenceImages": reference_images[-5:] if reference_images else [],  # Max 5 refs
        "model": "gpt-image-2",
        "aspectRatio": "16:9"
    }
    response = requests.post(f'{API_BASE}/v1/images/generate', headers=headers, json=payload)
    # Poll, save, add to reference_images for identity lock

# Step 4: Animate each still via Seedance 2.0 i2v
video_clips = []
for i, still_path in enumerate(stills):
    with open(still_path, 'rb') as f:
        still_b64 = base64.b64encode(f.read()).decode('utf-8')
    
    payload = {
        "startFrame": still_b64,
        "prompt": f"Subtle Pixar-style animation. {beats[i]}. Natural character movement, expressive faces.",
        "duration": 3,
        "model": "seedance-2",
        "aspectRatio": "16:9"
    }
    # Generate and save clip

# Step 5: Stitch with ffmpeg
import subprocess

concat_list = "\n".join([f"file '{clip}'" for clip in video_clips])
with open('concat_list.txt', 'w') as f:
    f.write(concat_list)

subprocess.run([
    'ffmpeg', '-f', 'concat', '-safe', '0', 
    '-i', 'concat_list.txt', '-c', 'copy', 'output.mp4'
])

# Step 6: Burn captions (optional)
# Uses HyperFrames + Whisper for transcription
subprocess.run([
    'npx', 'hyperframes', 
    '--input', 'output.mp4',
    '--output', 'output_captioned.mp4',
    '--style', 'modern'
])
```

### Claymation / Aardman-style ad

Same backbone as Pixar with clay texture prompts:

```python
character_prompt = """
Claymation character. Sculpted plasticine aesthetic.
Visible fingerprint textures, slight surface imperfections.
Aardman Studios style: expressive eyes, exaggerated mouth movements.
Stop-motion lighting: soft, even, slightly matte surface.
"""

# Storyboard generation same as Pixar but with clay aesthetic
# Post-processing: add stop-motion judder
subprocess.run([
    'ffmpeg', '-i', 'output.mp4', 
    '-vf', 'fps=12,fps=24',  # 12fps feel, output 24fps
    'output_claymation.mp4'
])
```

### YouTube thumbnail generation

```python
# 5 CTR formulas: peace-sign, comparison, terminal, reaction, before/after

# Peace sign + branding formula
with open('references/face/portrait1.png', 'rb') as f:
    face_ref1 = base64.b64encode(f.read()).decode('utf-8')
# Load 4 more face references for likeness lock

payload = {
    "prompt": """
    YouTube thumbnail, 16:9.
    
    LEFT SIDE: [Person] from chest up, peace sign hand gesture at face level.
    Big genuine smile, direct eye contact. Bright engaging expression.
    
    RIGHT SIDE: [Product] hero shot with glowing accent light.
    
    BACKGROUND: Vibrant gradient [brand color 1] to [brand color 2].
    
    TEXT OVERLAY TOP: "[Hook Text]" in bold sans-serif, white with black stroke.
    
    LIGHTING: High-key, saturated, contrasty (YouTube CTR optimization).
    """,
    "referenceImages": [face_ref1, face_ref2, face_ref3, face_ref4, face_ref5],
    "model": "nano-banana-2",
    "aspectRatio": "16:9"
}

# Generate 6 variations in parallel
```

## Polling and job management

**Standard polling pattern:**

```python
def poll_job(job_id, timeout=600, interval=10):
    """Poll Arcads job until completion or timeout."""
    start_time = time.time()
    
    while True:
        if time.time() - start_time > timeout:
            raise TimeoutError(f"Job {job_id} timed out after {timeout}s")
        
        response = requests.get(
            f'{API_BASE}/v1/jobs/{job_id}',
            headers=headers
        )
        data = response.json()
        
        status = data['status']
        if status == 'completed':
            return data['outputUrl']
        elif status == 'failed':
            raise Exception(f"Job failed: {data.get('error', 'Unknown error')}")
        elif status in ['pending', 'processing']:
            print(f"Job {job_id}: {status}... {data.get('progress', 0)}%")
            time.sleep(interval)
        else:
            raise Exception(f"Unknown status: {status}")

# Usage
job_id = response.json()['jobId']
output_url = poll_job(job_id)
```

**Batch job management:**

```python
def poll_jobs_parallel(job_ids):
    """Poll multiple jobs, return results as they complete."""
    results = {}
    pending = set(job_ids)
    
    while pending:
        for job_id in list(pending):
            response = requests.get(f'{API_BASE}/v1/jobs/{job_id}', headers=headers)
            data = response.json()
            
            if data['status'] == 'completed':
                results[job_id] = data['outputUrl']
                pending.remove(job_id)
            elif data['status'] == 'failed':
                results[job_id] = {'error': data.get('error')}
                pending.remove(job_id)
        
        if pending:
            time.sleep(10)
    
    return results
```

## Cost estimation and confirmation

**Pre-generation cost check:**

```python
def estimate_cost(model, duration=None):
    """Estimate cost before generation."""
    costs = {
        'seedance-2': 0.15,  # per video
        'sora-2': 0.25,
        'veo-3': 0.20,
        'kling-3': 0.10,
        'nano-banana-2': 0.05,  # per image
        'gpt-image-2': 0.04
    }
    return costs.get(model, 0)

# Confirm before batch operations
total_cost = estimate_cost('seedance-2') * len(beats)
confirm = input(f"Generate {len(beats)} Seedance videos for ~${total_cost:.2f}? (y/n): ")
if confirm.lower() != 'y':
    exit()
```

## File organization

**Standard output structure:**

```
output/
├── videos/
│   ├── seedance/
│   │   ├── ugc-selfie-v1.mp4
│   │   └── product-reveal-v2.mp4
│   ├── sora/
│   ├── veo/
│   └── kling/
├── images/
│   ├── nano-banana/
│   │   ├── influencer-shots/
│   │   └── product-stills/
│   └── chatgpt-image/
│       └── static-ads/
├── references/
│   ├── influencers/
│   │   └── sofia/
│   │       ├── hero.png
│   │       └── angles/
│   ├── products/
│   └── aesthetics/
│       └── ugc-selfie/
└── projects/
    ├── pixar-campaign-q4/
    └── claymation-launch/
```

## Prompt engineering guidelines

### Seedance 2.0 UGC formula (9-layer structure)

```
[SCENE SETUP]
Camera type, framing, background, lighting specifics.

[CHARACTER]
Age, appearance details, clothing, demeanor.

[PERFORMANCE DIRECTION]
Eye contact patterns, gesture timing, delivery style.

[DIALOGUE]
Exact spoken words with emphasis markers.

[CAMERA MOVEMENT]
Static vs. handheld, any motion.

[AESTHETIC]
Color grade, texture, grain, technical specs.

[AUDIO]
Voice characteristics, pacing, ambient sound.

[TIMING BEATS]
0-3s: [action], 3-6s: [action], etc.

[ANTI-PATTERNS]
Things to avoid (overly polished, unnatural poses).
```

### Reference image hierarchy

**Identity lock priority:**
1. Hero portrait (front-facing, centered, eye contact)
2. 3/4 angles with same lighting
3. Expressions with feature consistency
4. Full-body with environment context
5. Style references (max 5 total per request)

### Dialogue integration (Veo 3.1)

```python
# MANDATORY: Separate dialogue confirmation
dialogue = "This literally saved me hours every week"

# Then pass to payload
payload = {
    "startFrame": image_b64,
    "prompt": "Performance direction (gestures, expressions) without dialogue text",
    "dialogue": dialogue,  # Separate field
    "duration": 8
}
```

## Common workflows

### Complete UGC ad pipeline

```python
# 1. Create AI influencer
# 2. Generate product selfie still with Nano Banana
# 3. Animate still with Veo 3.1 + dialogue
# 4. Burn captions with HyperFrames
# 5. Export final MP4

# Step 1: Influencer
# (Use 10-image character sheet workflow above)

# Step 2: Product still
ugc_still_payload = {
    "prompt": "UGC selfie prompt...",
    "referenceImages": [character_ref, product_ref],
    "model": "nano-banana-2"
}
still_response = requests.post(f'{API_BASE}/v1/images/generate', headers=headers, json=ugc_still_payload)
still_url = poll_job(still_response.json()['jobId'])

# Download still
still_local = 'ugc_still.png'
requests.get(still_url).content
with open(still_local, 'wb') as f:
    f.write(requests.get(still_url).content)

# Step 3: Animate with dialogue
with open(still_local, 'rb') as f:
    still_b64 = base64.b64encode(f.read()).decode('utf-8')

video_payload = {
    "startFrame": still_b64,
    "prompt": "Natural head movement, hand gestures on key words, authentic delivery",
    "dialogue": "This literally saved me hours every week",
    "duration": 8,
    "aspectRatio": "9:16"
}
video_response = requests.post(f'{API_BASE}/v1/veo3/video', headers=headers, json=video_payload)
video_url = poll_job(video_response.json()['jobId'])

# Download video
video_local = 'ugc_video.mp4'
with open(video_local, 'wb') as f:
    f.write(requests.get(video_url).content)

# Step 4: Captions
subprocess.run(['npx', 'hyperframes', '--input', video_local, '--output', 'final.mp4'])
```

### Batch image generation (thumbnails, variations)

```python
prompts = [
    "Variation 1: peace sign formula",
    "Variation 2: comparison formula",
    "Variation 3: reaction formula",
    "Variation 4: before/after split",
    "Variation 5: terminal flow",
    "Variation 6: branded product showcase"
]

job_ids = []
for prompt in prompts:
    payload = {
        "prompt": prompt,
        "referenceImages": face_refs,
        "model": "nano-banana-2",
        "aspectRatio": "16:9"
    }
    response = requests.post(f'{API_BASE}/v1/images/generate', headers=headers, json=payload)
    job_ids.append(response.json()['jobId'])

# Poll all in parallel
results = poll_jobs_parallel(job_ids)
```

## Troubleshooting

### Authentication errors

```python
# Verify API key is set
import os
api_key = os.getenv('ARCADS_API_KEY')
if not api_key:
    raise ValueError("ARCADS_API_KEY not set in .env")

# Test connection
response = requests.get(f'{API_BASE}/v1/account', headers
