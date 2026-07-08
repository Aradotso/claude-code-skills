---
name: arcads-ai-video-generation
description: Create AI marketing videos and static image ads using Arcads API with Seedance 2.0, Sora 2, Veo 3.1, Kling 3.0, Nano Banana, and 37-template Meta ad library
triggers:
  - generate an arcads video
  - create AI marketing video with seedance
  - make a ugc video with arcads
  - generate nano banana product image
  - create meta image ad with arcads
  - animate still to video with veo
  - build pixar style ad campaign
  - make claymation ad with arcads
---

# Arcads AI Video Generation

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

Create AI marketing videos and static image ads using the Arcads external API. Supports **Seedance 2.0** (flagship video model), **Sora 2**, **Veo 3.1**, **Kling 3.0**, **Grok Video**, **Nano Banana 2/Pro/Edit**, **ChatGPT Image 2**, **OmniHuman**, and **Audio-driven** lip-sync. Also includes a 37-template static Meta image-ad library and multi-step pipelines for Pixar-style and claymation animated ads.

## Installation

### Clone and Setup

```bash
git clone https://github.com/krusemediallc/arcads-claude-code.git
cd arcads-claude-code
./scripts/setup.sh
```

The setup script will:
- Prompt for your Arcads API key (get it at [app.arcads.ai/settings/api](https://app.arcads.ai/settings/api))
- Save credentials to `.env` (gitignored)
- Create `MASTER_CONTEXT.md` workspace file
- Verify API connection

### Prerequisites

| Tool | Required For | Install (macOS) |
|------|-------------|-----------------|
| Python 3.10+ | Core functionality | `brew install python@3.12` |
| ffmpeg | Video stitching, chroma-key | `brew install ffmpeg` |
| jq | Shell script parsing | `brew install jq` |
| Node.js | Caption burn-in (hyperframes) | `brew install node` |
| whisper | Transcription | `pip install openai-whisper` |

### Environment Variables

Create `.env` in project root:

```bash
ARCADS_API_KEY=your_api_key_here
```

## Core API Patterns

### Base Configuration

All API calls use:
- **Base URL**: `https://app.arcads.ai/api`
- **Authentication**: `x-api-key` header
- **Content-Type**: `application/json`

### Python Request Template

```python
import os
import requests
from dotenv import load_dotenv

load_dotenv()

API_KEY = os.getenv('ARCADS_API_KEY')
BASE_URL = 'https://app.arcads.ai/api'

headers = {
    'x-api-key': API_KEY,
    'Content-Type': 'application/json'
}

def make_arcads_request(endpoint, payload):
    """Standard Arcads API request with error handling."""
    response = requests.post(f'{BASE_URL}{endpoint}', 
                            json=payload, 
                            headers=headers)
    response.raise_for_status()
    return response.json()
```

### Job Polling Pattern

Most video/image generation is async. Standard polling:

```python
import time

def poll_job(job_id, endpoint='/v1/job/status'):
    """Poll job until complete or failed."""
    while True:
        response = requests.get(
            f'{BASE_URL}{endpoint}/{job_id}',
            headers=headers
        )
        data = response.json()
        status = data.get('status')
        
        if status == 'completed':
            return data.get('output_url')
        elif status in ['failed', 'error']:
            raise Exception(f"Job failed: {data.get('error')}")
        
        print(f"Status: {status}... waiting 5s")
        time.sleep(5)
```

## Video Generation

### Seedance 2.0 (Flagship Model)

**Best for**: UGC-style ads, product reveals, feature demos (4–15s clips)

#### UGC Selfie-Style Product Review

```python
# 9-layer UGC formula for Seedance 2.0
payload = {
    "model": "seedance-2",
    "prompt": """
    A 12-second UGC video. Scene: bright kitchen, natural daylight. 
    Woman in casual tee holds [product]. iPhone selfie camera angle.
    
    She looks directly at camera: "I used to buy [competitor] until I tried this."
    Holds product closer. Natural eye-contact breaks. Casual delivery.
    Subtle background motion (steam from coffee mug). 
    Authentic imperfect framing. No polish.
    """,
    "duration": 12,
    "aspectRatio": "9:16",
    "audio": True
}

response = make_arcads_request('/v1/seedance-2/generate', payload)
job_id = response['jobId']
video_url = poll_job(job_id)
print(f"Video ready: {video_url}")
```

**Prompt templates** (5 formulas included):
- UGC selfie (`seedance-2-ugc.md`)
- Premium reveal (`seedance-2-premium-reveal.md`)
- Product hero w/ effects (`seedance-2-product-hero.md`)
- Studio lookbook (`seedance-2-studio-lookbook.md`)
- Feature walkthrough (`seedance-2-feature-walkthrough.md`)

#### With Reference Image (Image-to-Video)

```python
import base64

def encode_image(image_path):
    with open(image_path, 'rb') as f:
        return base64.b64encode(f.read()).decode('utf-8')

payload = {
    "model": "seedance-2",
    "prompt": "Animate this product shot with water splash and slow rotation",
    "duration": 8,
    "startFrame": encode_image('product.jpg'),
    "aspectRatio": "1:1"
}

response = make_arcads_request('/v1/seedance-2/generate', payload)
```

### Veo 3.1 (Still → Video Animation)

**Best for**: Animating Nano Banana stills into video with dialogue

```python
# Step 1: Generate still with Nano Banana (see Image Generation)
# Step 2: Animate with Veo 3.1

payload = {
    "prompt": """
    Natural human motion. Woman smiles, gestures with hand holding product.
    She says: "This changed everything for me."
    Authentic UGC delivery. Slight head tilt. Eye contact with camera.
    """,
    "startFrame": encode_image('ugc_still.jpg'),
    "duration": 8,
    "aspectRatio": "9:16"
}

response = make_arcads_request('/v1/veo-3.1/generate', payload)
job_id = response['jobId']
video_url = poll_job(job_id)
```

**MANDATORY dialogue gate**: Always confirm dialogue separately before generating Veo videos.

### Sora 2 (Long-Form Text-to-Video)

**Best for**: Longer clips (up to 20s), cinematic scenes

```python
payload = {
    "prompt": "Wide shot of a modern kitchen. Golden hour light streams through window. Product sits on marble counter. Camera slowly dollies forward.",
    "duration": 16,  # Auto-calculated from word count (~2.5 words/sec)
    "aspectRatio": "16:9",
    "styleReference": encode_image('style_ref.jpg')  # Optional
}

response = make_arcads_request('/v1/sora2/generate', payload)
```

#### Sora 2 Remix

```python
payload = {
    "videoUrl": "https://arcads.ai/output/original_video.mp4",
    "prompt": "Same scene but at sunset with warmer tones"
}

response = make_arcads_request('/v1/sora2/remix/video', payload)
```

### Kling 3.0 (B-Roll & Scenes)

**Best for**: Quick b-roll clips, environmental shots

```python
# B-roll endpoint
payload = {
    "prompt": "Close-up of hands pouring coffee. Steam rising. Soft morning light.",
    "duration": 5
}
response = make_arcads_request('/v1/b-roll', payload)

# Scene endpoint
payload = {
    "prompt": "Modern minimalist living room. Large windows. Afternoon light.",
    "duration": 8
}
response = make_arcads_request('/v1/scene', payload)
```

### Other Video Models

```python
# Grok Video
payload = {
    "model": "grok-video",
    "prompt": "Product floating in space with particle effects",
    "duration": 10
}
response = make_arcads_request('/v2/videos/generate', payload)

# OmniHuman (Talking Avatar)
payload = {
    "avatarImage": encode_image('avatar.jpg'),
    "script": "Hi, I'm Sarah and I want to tell you about this amazing product.",
    "duration": 12
}
response = make_arcads_request('/v1/omnihuman', payload)

# Audio-Driven (Lip Sync)
payload = {
    "videoUrl": "https://example.com/base_video.mp4",
    "audioFile": encode_image('voiceover.mp3')
}
response = make_arcads_request('/v1/audio-driven', payload)
```

## Image Generation

### Nano Banana (Photoreal Product Images)

**Models**: `nano-banana-2` (default), `nano-banana` (Pro/Gemini 3), `nano-banana-edit` (inpainting)

#### UGC Product Selfie Still

```python
payload = {
    "model": "nano-banana-2",
    "prompt": """
    iPhone selfie of 22-year-old woman in bedroom. Natural lighting.
    She holds [product] near her face, smiling genuinely.
    Freckles visible. Hair slightly messy. Authentic UGC aesthetic.
    Phone camera POV. Slight blur on edges. Real skin texture.
    """,
    "aspectRatio": "9:16",
    "referenceImages": [
        encode_image('references/influencers/sofia_hero.jpg'),
        encode_image('references/products/product_angle1.jpg'),
        encode_image('references/aesthetics/ugc-selfie/style1.jpg')
    ]
}

response = make_arcads_request('/v1/nano-banana/generate', payload)
job_id = response['jobId']
image_url = poll_job(job_id, endpoint='/v1/job/status')
```

#### Create AI Influencer (10-Image Character Sheet)

```python
def create_influencer(description):
    """Generate 10-angle character sheet for reuse."""
    
    # Step 1: Hero front portrait
    hero_payload = {
        "model": "nano-banana-2",
        "prompt": f"""
        Front-facing portrait. {description}
        Studio lighting. Direct eye contact. Neutral expression.
        High detail on facial features. Clean background.
        """,
        "aspectRatio": "1:1"
    }
    
    hero_response = make_arcads_request('/v1/nano-banana/generate', hero_payload)
    hero_url = poll_job(hero_response['jobId'])
    
    # Download hero for reference
    hero_img = requests.get(hero_url).content
    with open('hero.jpg', 'wb') as f:
        f.write(hero_img)
    
    hero_base64 = encode_image('hero.jpg')
    
    # Step 2: Generate 9 additional angles
    angles = [
        "3/4 view looking left",
        "3/4 view looking right",
        "Profile view left side",
        "Profile view right side",
        "Close-up of face",
        "Smiling expression",
        "Thoughtful expression",
        "Upper body shot",
        "Full body standing"
    ]
    
    for i, angle in enumerate(angles):
        payload = {
            "model": "nano-banana-2",
            "prompt": f"{description}. {angle}. Same person, consistent features.",
            "aspectRatio": "1:1",
            "referenceImages": [hero_base64]
        }
        response = make_arcads_request('/v1/nano-banana/generate', payload)
        img_url = poll_job(response['jobId'])
        
        # Save to references/influencers/
        img_data = requests.get(img_url).content
        with open(f'references/influencers/char_{i+1}.jpg', 'wb') as f:
            f.write(img_data)
    
    return hero_url

# Usage
create_influencer("22-year-old woman with freckles, golden hair, green eyes")
```

#### Nano Banana Pro (Higher Fidelity)

```python
payload = {
    "model": "nano-banana",  # Pro model (Gemini 3 Pro Image)
    "prompt": "Studio product shot with precise lighting setup",
    "aspectRatio": "1:1",
    "referenceImages": [encode_image('ref1.jpg'), encode_image('ref2.jpg')]
}
```

### ChatGPT Image 2 (Typography & UI-Heavy)

**Best for**: Text-heavy ads, UI mockups, editorial layouts

```python
payload = {
    "model": "gpt-image-2",
    "prompt": """
    Apple Notes style ad. iPhone notes app interface.
    Title: "Why I switched to [Product]"
    Bulleted list with 5 benefits. Clean iOS typography.
    Screenshot aesthetic. Realistic app chrome.
    """,
    "aspectRatio": "9:16"
}

response = make_arcads_request('/v1/chatgpt-image/generate', payload)
```

## Static Meta Image Ad Library (37 Templates)

### Three-Skill Family

**Skills included**:
- `chatgpt-image-ad` — Typography/UI-heavy (GPT Image 2 backend)
- `nano-banana-image-ad` — Photoreal/lifestyle (Nano Banana backend)
- `image-ad-clone` — Reverse-engineer existing ads into templates

### Template Examples

```python
# Apple Notes List Ad
payload = {
    "template": "apple-notes-list",
    "product": "Acme Protein Powder",
    "headline": "5 reasons I ditched [competitor]",
    "bullets": [
        "30g protein per serving",
        "No artificial sweeteners",
        "Mixes instantly",
        "Actually tastes good",
        "Costs 40% less"
    ],
    "aspectRatio": "9:16"
}

# Forbes Editorial Hero
payload = {
    "template": "forbes-editorial",
    "headline": "The $2B Startup Disrupting [Industry]",
    "subhead": "How Acme's founder built a unicorn in 18 months",
    "productImage": encode_image('product.jpg'),
    "aspectRatio": "1:1"
}

# Comparison Table
payload = {
    "template": "comparison-table",
    "columns": ["Us", "Competitor A", "Competitor B"],
    "rows": [
        {"feature": "Price", "values": ["$29", "$49", "$59"]},
        {"feature": "Servings", "values": ["30", "20", "25"]},
        {"feature": "Protein", "values": ["30g", "25g", "28g"]}
    ],
    "aspectRatio": "1:1"
}
```

### Clone Existing Ad

```python
def clone_ad_template(reference_image_path):
    """Reverse-engineer ad into reusable template."""
    
    payload = {
        "action": "analyze",
        "referenceImage": encode_image(reference_image_path)
    }
    
    response = make_arcads_request('/v1/image-ad-clone', payload)
    
    # Returns template structure
    template = response['template']
    print(f"Detected template: {template['type']}")
    print(f"Layout: {template['layout']}")
    print(f"Typography: {template['typography']}")
    
    # Generate new version with your content
    payload = {
        "action": "generate",
        "template": template,
        "content": {
            "headline": "Your New Headline",
            "body": "Your body copy"
        },
        "backend": "nano-banana-2"  # or "gpt-image-2"
    }
    
    new_ad = make_arcads_request('/v1/image-ad-clone', payload)
    return poll_job(new_ad['jobId'])
```

### Backend Selection Matrix

| Template Type | Recommended Backend | Aspect Ratios |
|---------------|-------------------|---------------|
| Apple Notes, UI mockups | `gpt-image-2` | 9:16, 1:1 |
| Photoreal lifestyle | `nano-banana-2` | All |
| Editorial hero | `nano-banana` (Pro) | 1:1, 4:5 |
| Comparison tables | `gpt-image-2` | 1:1 |
| Sticky-note flatlay | `nano-banana-2` | 4:5, 1:1 |

Read `shared/skills/image-ad-prompting/OVERVIEW.md` for full template library and decision tree.

## Multi-Step Animated Ad Pipelines

### Pixar-Style 3D Animated Ad

**Pipeline**: Cast sheet → ChatGPT Image 2 storyboard → Seedance 2.0 i2v → ffmpeg stitch + captions

```bash
# Full pipeline via bash script
./shared/skills/pixar-style-ad/scripts/generate.sh \
  --product "Acme Coffee" \
  --story "Tired bean meets energizing coffee bag, transforms into superhero bean" \
  --duration 32 \
  --output output/pixar-ad.mp4
```

**Python workflow**:

```python
def generate_pixar_ad(product_name, story_arc, duration=32):
    """Generate Pixar-style 8-beat animated ad."""
    
    beats = 8
    seconds_per_beat = duration / beats
    
    # Step 1: Lock cast sheet
    cast_payload = {
        "model": "gpt-image-2",
        "prompt": f"""
        Character design sheet for Pixar-style 3D animation.
        Product: {product_name}
        Characters: Anthropomorphized {product_name} mascot.
        Front, 3/4, side views. Expressive features. Pixar aesthetic.
        """,
        "aspectRatio": "16:9"
    }
    cast_response = make_arcads_request('/v1/chatgpt-image/generate', cast_payload)
    cast_url = poll_job(cast_response['jobId'])
    cast_base64 = encode_image(download_file(cast_url, 'cast.jpg'))
    
    # Step 2: Generate 8-beat storyboard stills
    storyboard = []
    prev_frame = None
    
    for beat in range(1, beats + 1):
        prompt = f"""
        Pixar-style 3D render. Beat {beat} of {beats}.
        {story_arc}. Scene: [describe beat {beat}].
        Consistent character from cast sheet. Pixar lighting and textures.
        """
        
        refs = [cast_base64]
        if prev_frame:
            refs.append(prev_frame)
        
        payload = {
            "model": "gpt-image-2",
            "prompt": prompt,
            "aspectRatio": "16:9",
            "referenceImages": refs[:5]  # Max 5 references
        }
        
        response = make_arcads_request('/v1/chatgpt-image/generate', payload)
        still_url = poll_job(response['jobId'])
        still_path = download_file(still_url, f'beat_{beat}.jpg')
        prev_frame = encode_image(still_path)
        
        storyboard.append(still_path)
    
    # Step 3: Animate each still with Seedance 2.0
    clips = []
    for i, still_path in enumerate(storyboard):
        payload = {
            "model": "seedance-2",
            "prompt": f"Animate this Pixar scene. Subtle character motion. Pixar-quality animation.",
            "duration": int(seconds_per_beat),
            "startFrame": encode_image(still_path),
            "aspectRatio": "16:9",
            "audio": False
        }
        
        response = make_arcads_request('/v1/seedance-2/generate', payload)
        clip_url = poll_job(response['jobId'])
        clip_path = download_file(clip_url, f'clip_{i}.mp4')
        clips.append(clip_path)
    
    # Step 4: Stitch with ffmpeg
    stitch_clips(clips, 'output/pixar_stitched.mp4')
    
    # Step 5: Burn captions (optional)
    # Uses hyperframes + whisper - see Caption Workflow below
    
    return 'output/pixar_stitched.mp4'

def stitch_clips(clips, output):
    """Stitch clips with ffmpeg."""
    import subprocess
    
    # Create concat file
    with open('concat.txt', 'w') as f:
        for clip in clips:
            f.write(f"file '{clip}'\n")
    
    subprocess.run([
        'ffmpeg', '-f', 'concat', '-safe', '0', 
        '-i', 'concat.txt', '-c', 'copy', output
    ])
```

### Claymation / Aardman-Style Ad

**Same 8-beat structure** with clay textures and stop-motion aesthetic:

```python
# Modify storyboard generation
payload = {
    "model": "gpt-image-2",
    "prompt": f"""
    Claymation character made of plasticine clay.
    Visible sculpting marks. Aardman Animations style.
    Scene: {beat_description}
    Handcrafted texture. Tactile surface. Stop-motion lighting.
    """,
    "aspectRatio": "16:9"
}

# After stitching, add stop-motion judder
subprocess.run([
    'ffmpeg', '-i', 'stitched.mp4', 
    '-vf', 'fps=12,fps=24',  # 12fps judder
    'claymation_final.mp4'
])
```

### Caption Burn-In Workflow

**Dependencies**: `whisper`, `hyperframes` (Node.js)

```python
def burn_captions(video_path, output_path):
    """Transcribe and burn captions onto video."""
    import subprocess
    import json
    
    # Step 1: Extract audio
    subprocess.run([
        'ffmpeg', '-i', video_path, 
        '-vn', '-acodec', 'pcm_s16le', 'audio.wav'
    ])
    
    # Step 2: Transcribe with Whisper
    subprocess.run([
        'whisper', 'audio.wav', 
        '--model', 'medium.en', 
        '--output_format', 'json'
    ])
    
    # Step 3: Parse transcript
    with open('audio.json') as f:
        transcript = json.load(f)
    
    # Group words into reading phrases (3-5 words)
    phrases = group_words(transcript['words'], max_words=4)
    
    # Step 4: Generate caption overlays with hyperframes
    captions_config = {
        "phrases": phrases,
        "style": {
            "fontFamily": "Inter",
            "fontSize": 48,
            "fontWeight": "bold",
            "color": "#FFFFFF",
            "backgroundColor": "rgba(0,0,0,0.7)",
            "padding": 10
        }
    }
    
    with open('captions.json', 'w') as f:
        json.dump(captions_config, f)
    
    subprocess.run([
        'npx', 'hyperframes', 
        '--input', video_path,
        '--captions', 'captions.json',
        '--output', output_path
    ])
    
    return output_path

def group_words(words, max_words=4):
    """Group word-level transcript into reading phrases."""
    phrases = []
    current = []
    
    for word in words:
        current.append(word)
        if len(current) >= max_words or word['text'].endswith(('.', '!', '?')):
            phrases.append({
                "text": " ".join([w['text'] for w in current]),
                "start": current[0]['start'],
                "end": current[-1]['end']
            })
            current = []
    
    return phrases
```

## Publishing to Meta (via Meta Ad Builder Skill)

**Prerequisite**: Install Meta Ad Builder dependencies:

```bash
pip install -r shared/skills/meta-ad-builder/scripts/requirements.txt
```

**Environment variables**:

```bash
META_ACCESS_TOKEN=your_meta_access_token
META_AD_ACCOUNT_ID=act_123456789
```

### Publish Image Ad

```python
import sys
sys.path.append('shared/skills/meta-ad-builder/scripts')
from meta_publisher import publish_ad

ad_config = {
    "name": "UGC Selfie - Product Launch",
    "image_path": "output/ugc_selfie.jpg",
    "body": "I used to buy [competitor] until I tried this. 50% off today only 👇",
    "link": "https://example.com/product",
    "call_to_action": "SHOP_NOW",
    "status": "PAUSED"  # Always create paused
}

ad_id = publish_ad(ad_config)
print(f"Ad created (paused): {ad_id}")
```

### Publish Video Ad

```python
ad_config = {
    "name": "Seedance UGC Video - Kitchen Scene",
    "video_path": "output/seedance_ugc.mp4",
    "body": "This is how I start my mornings now 💪",
    "link": "https://example.com/product",
    "call_to_action": "LEARN_MORE",
    "status": "PAUSED"
}

ad_id = publish_ad(ad_config)
```

## Configuration Files

### `.env` Structure

```bash
# Required
ARCADS_API_KEY=your_api_key

# Optional (for Meta publishing)
META_ACCESS_TOKEN=your_token
META_AD_ACCOUNT_ID=act_123456789

# Optional (for external services)
ELEVENLABS_API_KEY=your_elevenlabs_key
```

### `MASTER_CONTEXT.md`

Personal workspace file created by setup. Track projects, campaigns, and reference assets:

```markdown
# Master Context - Arcads Workspace

## Active Campaigns
- Q1 Product Launch (UGC + Pixar combo)
- Retargeting (static image ads)

## AI Influencers
- Sofia: 22F, freckles, golden hair (refs in `references/influencers/sofia/`)
- Marcus: 28M, athletic, stubble (refs in `references/influencers/marcus/`)

## Products
- Acme Coffee: refs in `references/products/coffee/`
- Acme Protein: refs in `references/products/protein/`

## Validated Templates (Image Ads)
- Apple Notes list (9:16, gpt-image-2) ✅
- Forbes editorial (1:1, nano-banana-pro) ✅
- Comparison table (1:1, gpt-image-2) ✅
```

## Common Patterns

### Pattern 1: Still → Video (Two-Step UGC)

```python
def ugc_still_to_video(influencer_refs, product_ref, script):
    """Generate UGC still → animate to video."""
    
    # Step 1: Generate still
    still_payload = {
        "model": "nano-banana-2",
        "prompt": f"""
        iPhone selfie. Woman in kitchen holding product.
        Natural lighting. Authentic UGC aesthetic.
        """,
        "aspectRatio": "9:16",
        "referenceImages": influencer_refs + [product_ref]
    }
    
    still_response = make_arcads_request('/v1/nano-banana/generate', still_payload)
    still_url = poll_job(still_response['jobId'])
    still_base64 = encode_image(download_file(still_url, 'still.jpg'))
    
    # Step 2: Animate with Veo 3.1
    video_payload = {
        "prompt": f"""
        Natural motion. Woman smiles, gestures with product.
        She says: "{script}"
        Authentic delivery. Slight head movement. Eye contact.
        """,
        "startFrame": still_base64,
        "duration": len(script.split()) / 2.5,  # ~2.5 words/sec
        "aspectRatio": "9:16"
    }
    
    video_response = make_arcads_request('/v1/veo-3.1/generate', video_payload)
    return poll_job(video_response['jobId'])
```

### Pattern 2: Batch Image Generation

```python
def generate_image_variants(base_prompt, variations, model="nano-banana-2"):
    """Generate multiple image variations in parallel."""
    
    jobs = []
    for variation in variations:
        payload = {
            "model": model,
            "prompt": f"{base_prompt}. Variation: {variation}",
            "aspectRatio": "1:1"
        }
        response = make_arcads_request('/v1/nano-banana/generate', payload)
        jobs.append(response['jobId'])
    
    # Poll all jobs
    results = []
    for job_id in jobs:
        url = poll_job(job_id)
        results.append(url)
    
    return results

# Usage
variants = generate_image_variants(
    "Product on marble counter",
    ["morning light", "golden hour", "dramatic side light"]
)
```

### Pattern 3: Reference Library Builder

```python
def build_reference_library(category, descriptions):
    """Build organized reference library."""
    import os
    
    category_path = f'references/{category}'
    os.makedirs(category_path, exist_ok=True)
    
    for i, desc in enumerate(descriptions):
        payload = {
            "model": "nano-banana-2",
            "prompt": desc,
            "aspectRatio": "1:1"
        }
        
        response = make_arcads_request('/v1/nano-banana/generate', payload)
        img_url = poll_job(response['jobId'])
        
        img_data = requests.get(img_url).content
        filename = desc.replace(' ', '_')[:30]
        with open(f'{category_path}/{filename}_{i}.jpg', 'wb') as f:
            f.write(img_data)
    
    print(f"Library built: {category_path}/")

# Usage
build_reference_library('aesthetics/lifestyle
