---
name: arcads-ai-video-generator
description: Create AI marketing videos and images using Arcads external API with Seedance 2.0, Sora 2, Veo 3.1, Kling 3.0, Nano Banana, and 37 static ad templates
triggers:
  - generate an arcads video
  - create a seedance ugc video
  - make a nano banana image ad
  - generate a veo animation from this image
  - create a pixar style ad campaign
  - build static meta ad creative
  - animate this still with arcads
  - create ai influencer character sheet
---

# Arcads AI Video Generator

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## What It Does

Arcads is a creative automation stack for AI-generated marketing videos and images. This skill pack provides direct API access to:

- **Video models**: Seedance 2.0 (flagship), Sora 2, Veo 3.1, Kling 3.0, Grok Video, OmniHuman, Audio-driven
- **Image models**: Nano Banana 2/Pro/Edit, ChatGPT Image 2
- **Static ad library**: 37 validated Meta image-ad templates
- **Multi-step pipelines**: Pixar-style animated ads, claymation workflows, YouTube thumbnails

The agent handles API calls, polling for completion, prompt engineering using validated formulas, file organization, and cost confirmation.

## Installation

### 1. Clone and Setup

```bash
git clone https://github.com/krusemediallc/arcads-claude-code.git
cd arcads-claude-code
./scripts/setup.sh
```

The setup script will:
- Prompt for your Arcads API key from [app.arcads.ai/settings/api](https://app.arcads.ai/settings/api)
- Save credentials to `.env` (gitignored)
- Create `MASTER_CONTEXT.md` workspace file
- Verify API connection

### 2. Environment Configuration

Create `.env` in project root:

```bash
ARCADS_API_KEY=your_api_key_here
```

### 3. Dependencies

**Core (required):**
- Python 3.10+

**Optional (for multi-step pipelines):**
```bash
# macOS
brew install ffmpeg jq node

# Python packages for specific workflows
pip install openai-whisper  # Caption transcription
pip install -r shared/skills/meta-ad-builder/scripts/requirements.txt  # Meta publishing
```

**Linux:**
```bash
apt install ffmpeg jq nodejs python3
```

## Core API Patterns

### Base Request Structure

```python
import requests
import os
import time

API_KEY = os.getenv("ARCADS_API_KEY")
BASE_URL = "https://api.arcads.ai"

headers = {
    "Authorization": f"Bearer {API_KEY}",
    "Content-Type": "application/json"
}
```

### Polling Pattern (All Video/Image Jobs)

```python
def poll_until_complete(job_id, endpoint_type="video"):
    """
    Poll job status until complete or failed.
    endpoint_type: 'video' or 'image'
    """
    poll_url = f"{BASE_URL}/v1/{endpoint_type}s/{job_id}"
    
    while True:
        response = requests.get(poll_url, headers=headers)
        data = response.json()
        
        status = data.get("status")
        if status == "completed":
            return data.get("output_url") or data.get("outputUrl")
        elif status == "failed":
            raise Exception(f"Job failed: {data.get('error')}")
        
        time.sleep(10)  # Poll every 10 seconds
```

## Video Generation

### Seedance 2.0 (Flagship Model)

**UGC Selfie-Style Product Review:**

```python
def generate_seedance_ugc(product_name, competitor_name):
    """
    9-layer UGC formula for Seedance 2.0
    See: skills/arcads-external-api/prompting/prompt-library/seedance-2-ugc.md
    """
    prompt = f"""
    Medium shot of a woman in her late 20s in a bright modern kitchen. 
    She's holding {product_name} in her hands, looking directly at camera 
    with genuine excitement. Natural iPhone aesthetic - slight grain, 
    authentic lighting. She speaks casually: "I used to buy {competitor_name} 
    but this is so much better." Eye contact breaks naturally as she 
    demonstrates the product. Handheld stability with minor drift.
    """
    
    payload = {
        "prompt": prompt,
        "duration": 12,  # 4-15 seconds supported
        "aspectRatio": "9:16",  # or "16:9", "1:1"
        "model": "seedance-2"
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/videos/generate",
        headers=headers,
        json=payload
    )
    
    job_id = response.json()["id"]
    return poll_until_complete(job_id, "video")
```

**Premium Product Reveal (No Person):**

```python
def generate_premium_reveal(product_name, key_benefit):
    """
    Dark void aesthetic, text narrative, hero rotation
    See: skills/arcads-external-api/prompting/prompt-library/seedance-2-premium-reveal.md
    """
    prompt = f"""
    {product_name} floating in dark void background. Cinematic lighting 
    with dramatic shadows. Text overlay fades in: "{key_benefit}". 
    Product slowly rotates 360 degrees. High-end commercial aesthetic. 
    Depth of field with product in sharp focus, background bokeh.
    """
    
    payload = {
        "prompt": prompt,
        "duration": 8,
        "aspectRatio": "1:1",
        "model": "seedance-2"
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/videos/generate",
        headers=headers,
        json=payload
    )
    
    return poll_until_complete(response.json()["id"], "video")
```

**Product Hero with Elemental Effects:**

```python
def generate_product_hero(product_description):
    """
    Splash, mist, light rays, slow rotation
    See: skills/arcads-external-api/prompting/prompt-library/seedance-2-product-hero.md
    """
    prompt = f"""
    {product_description} on white pedestal. Water splash erupts from 
    base in slow motion. Atmospheric mist swirls around product. 
    Volumetric light rays pierce through from top-right. Product 
    rotates slowly (30 degrees total). Hyper-realistic physics, 
    high-speed camera aesthetic.
    """
    
    payload = {
        "prompt": prompt,
        "duration": 10,
        "aspectRatio": "16:9",
        "model": "seedance-2"
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/videos/generate",
        headers=headers,
        json=payload
    )
    
    return poll_until_complete(response.json()["id"], "video")
```

### Veo 3.1 (Start-Frame Animation)

```python
def animate_with_veo(image_path, dialogue_script, duration=8):
    """
    Animate a still image with natural motion and dialogue.
    Standard path for UGC stills → video.
    """
    import base64
    
    with open(image_path, "rb") as f:
        image_b64 = base64.b64encode(f.read()).decode()
    
    prompt = f"""
    Natural human motion and expressions. Person delivers dialogue: 
    "{dialogue_script}". Subtle head movements, eye blinks, 
    authentic micro-expressions. Lip sync matches audio perfectly.
    """
    
    payload = {
        "prompt": prompt,
        "startFrame": image_b64,
        "duration": duration,
        "aspectRatio": "9:16",
        "model": "veo-3.1",
        "dialogue": dialogue_script  # MANDATORY dialogue gate
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/videos/generate",
        headers=headers,
        json=payload
    )
    
    return poll_until_complete(response.json()["id"], "video")
```

### Sora 2 (Text-to-Video, Up to 20s)

```python
def generate_sora_video(scene_description, duration=16):
    """
    Longer-form text-to-video. Auto-calculates duration from script (~2.5 words/sec).
    """
    payload = {
        "prompt": scene_description,
        "duration": duration,  # Up to 20 seconds
        "aspectRatio": "16:9",
        "model": "sora-2"
    }
    
    # Optional: add style reference image
    # payload["referenceImages"] = [base64_encoded_image]
    
    response = requests.post(
        f"{BASE_URL}/v1/videos/generate",
        headers=headers,
        json=payload
    )
    
    return poll_until_complete(response.json()["id"], "video")
```

### Kling 3.0 (B-Roll / Scene)

```python
def generate_broll(scene_description):
    """B-roll clips for editing into longer content"""
    payload = {
        "prompt": scene_description,
        "duration": 5,
        "aspectRatio": "16:9"
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/b-roll",
        headers=headers,
        json=payload
    )
    
    return poll_until_complete(response.json()["id"], "video")

def generate_scene(environment_description):
    """Environment/atmosphere shots"""
    payload = {
        "prompt": environment_description,
        "duration": 5,
        "aspectRatio": "16:9"
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/scene",
        headers=headers,
        json=payload
    )
    
    return poll_until_complete(response.json()["id"], "video")
```

## Image Generation

### Nano Banana (Character Creation & Product Stills)

**Create AI Influencer Character Sheet:**

```python
def create_ai_influencer(character_description):
    """
    Two-pass workflow:
    1. Generate hero front portrait
    2. Generate 9 additional angles with hero as reference
    """
    # Pass 1: Hero portrait
    hero_prompt = f"""
    Front-facing portrait of {character_description}. Direct eye contact, 
    natural expression, soft studio lighting. Professional headshot quality.
    """
    
    payload = {
        "prompt": hero_prompt,
        "model": "nano-banana-2",  # or "nano-banana" for Pro
        "aspectRatio": "1:1",
        "numberOfImages": 1
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/images/generate",
        headers=headers,
        json=payload
    )
    
    hero_url = poll_until_complete(response.json()["id"], "image")
    
    # Download hero for reference
    hero_image = requests.get(hero_url).content
    hero_b64 = base64.b64encode(hero_image).decode()
    
    # Pass 2: Additional angles
    angles = [
        "3/4 view facing left",
        "3/4 view facing right", 
        "profile view left side",
        "profile view right side",
        "close-up face",
        "smiling expression",
        "serious expression",
        "full body standing",
        "waist-up portrait"
    ]
    
    reference_images = []
    for angle in angles:
        prompt = f"{character_description}, {angle}. Match exact appearance, features, and styling from reference."
        
        payload = {
            "prompt": prompt,
            "model": "nano-banana-2",
            "aspectRatio": "1:1",
            "refImageAsBase64": hero_b64,
            "numberOfImages": 1
        }
        
        response = requests.post(
            f"{BASE_URL}/v1/images/generate",
            headers=headers,
            json=payload
        )
        
        url = poll_until_complete(response.json()["id"], "image")
        reference_images.append(url)
    
    return {
        "hero": hero_url,
        "references": reference_images
    }
```

**UGC Product Selfie Still:**

```python
def generate_ugc_selfie(character_ref_path, product_ref_path, scene_description):
    """
    Authentic iPhone selfie with character + product + aesthetic refs
    """
    import base64
    
    # Load references
    with open(character_ref_path, "rb") as f:
        character_b64 = base64.b64encode(f.read()).decode()
    
    with open(product_ref_path, "rb") as f:
        product_b64 = base64.b64encode(f.read()).decode()
    
    prompt = f"""
    iPhone selfie of person holding product. {scene_description}. 
    Natural bedroom lighting, slight grain, authentic camera imperfections. 
    Casual angle, arm extended. Skin texture visible with pores and slight 
    blemishes - fight AI polish. Match character appearance exactly from reference.
    """
    
    payload = {
        "prompt": prompt,
        "model": "nano-banana-2",
        "aspectRatio": "9:16",
        "referenceImages": [character_b64, product_b64],
        "numberOfImages": 1
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/images/generate",
        headers=headers,
        json=payload
    )
    
    return poll_until_complete(response.json()["id"], "image")
```

**Nano Banana Edit (Inpainting):**

```python
def edit_image_region(base_image_path, mask_image_path, edit_prompt):
    """
    Inpaint/edit specific region of existing image
    """
    import base64
    
    with open(base_image_path, "rb") as f:
        base_b64 = base64.b64encode(f.read()).decode()
    
    with open(mask_image_path, "rb") as f:
        mask_b64 = base64.b64encode(f.read()).decode()
    
    payload = {
        "prompt": edit_prompt,
        "model": "nano-banana-edit",
        "baseImage": base_b64,
        "maskImage": mask_b64,
        "aspectRatio": "1:1"
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/images/generate",
        headers=headers,
        json=payload
    )
    
    return poll_until_complete(response.json()["id"], "image")
```

### ChatGPT Image 2 (Typography & UI Mimicry)

```python
def generate_chatgpt_image_ad(template_type, product_name, benefit_list):
    """
    Typography-heavy static ad creative
    Templates: apple-notes, forbes-editorial, fake-google-search, etc.
    """
    template_prompts = {
        "apple-notes": f"""
            iPhone Notes app screenshot. Title: "Why I switched to {product_name}". 
            Bulleted list with checkmarks:
            {chr(10).join(f"✓ {benefit}" for benefit in benefit_list)}
            Light mode, SF Pro font, realistic iOS UI.
        """,
        
        "forbes-editorial": f"""
            Forbes magazine editorial layout. Headline: "The {product_name} Revolution". 
            Professional business photo with overlay text listing:
            {chr(10).join(f"• {benefit}" for benefit in benefit_list)}
            Forbes branding, premium typography, editorial design.
        """,
        
        "comparison-table": f"""
            Clean comparison table. Left column: "Other Products", right column: "{product_name}".
            Rows showing:
            {chr(10).join(f"{benefit}: X vs ✓" for benefit in benefit_list)}
            Modern UI, clear hierarchy, visual checkmarks and X marks.
        """
    }
    
    prompt = template_prompts.get(template_type, template_prompts["apple-notes"])
    
    payload = {
        "prompt": prompt,
        "model": "gpt-image-2",
        "aspectRatio": "1:1",
        "numberOfImages": 1
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/images/generate",
        headers=headers,
        json=payload
    )
    
    return poll_until_complete(response.json()["id"], "image")
```

## Static Meta Image Ad Library

The repo includes 37 validated prompt templates for static Meta ads. Three main skills:

### 1. ChatGPT Image Ad (Typography-Heavy)

```bash
# Located at: shared/skills/chatgpt-image-ad/
python shared/skills/chatgpt-image-ad/scripts/generate.py \
  --template apple-notes \
  --product "SuperWidget" \
  --benefits "Saves time" "Costs less" "Works better"
```

**Supported templates:** apple-notes, forbes-editorial, fake-google-search, comparison-table, sticky-note-flatlay, fake-slack-thread, chatgpt-conversation, imessage-screenshot, magazine-cover, billboard, weather-forecast-ui, scratch-off-ticket, founder-letter, dating-app-card

### 2. Nano Banana Image Ad (Photoreal/Lifestyle)

```bash
# Located at: shared/skills/nano-banana-image-ad/
python shared/skills/nano-banana-image-ad/scripts/generate.py \
  --template lifestyle-hero \
  --product "SuperWidget" \
  --character-ref references/influencers/sofia/hero.jpg \
  --product-ref references/products/superwidget.jpg
```

**Supported templates:** lifestyle-hero, ugc-selfie, product-showcase, editorial-flatlay, before-after-split

### 3. Image Ad Clone (Reverse Engineer)

```bash
# Located at: shared/skills/image-ad-clone/
python shared/skills/image-ad-clone/scripts/clone.py \
  --input competitor-ad.jpg \
  --backend nano-banana-2  # or gpt-image-2
```

This skill:
1. Analyzes the input ad structure
2. Extracts layout, color palette, typography style
3. Creates a new template entry
4. Validates against chosen backend
5. Optionally cross-validates against alternate backend

**Read `shared/skills/image-ad-prompting/OVERVIEW.md` for:**
- Decision tree (which backend for which template)
- Aspect ratio compatibility matrix
- Standard generate/clone workflows

## Multi-Step Pipelines

### Pixar-Style 3D Animated Ad

```bash
# Located at: shared/skills/pixar-style-ad/
./shared/skills/pixar-style-ad/scripts/generate.sh \
  --product "SuperWidget" \
  --mascot "Anthropomorphic widget with big eyes" \
  --story-arc "discovery, skepticism, transformation, celebration"
```

**Pipeline:**
1. Lock character cast sheet (ChatGPT Image 2)
2. Generate 8-beat storyboard (sequential, prior frame as ref)
3. Seedance 2.0 image-to-video per beat
4. FFmpeg stitch + burn captions
5. Export final 60-90s animated ad

**Requirements:** `ffmpeg`, `jq`, `whisper` (pip)

See: `shared/skills/pixar-style-ad/prompting/guide.md`

### Claymation / Aardman-Style Ad

```bash
# Located at: shared/skills/claymation-ad/
./shared/skills/claymation-ad/scripts/generate.sh \
  --product "SuperWidget" \
  --characters "Widget, User, Problem Monster" \
  --narrator-script "Meet Widget. It solves the problem..."
```

**Pipeline:**
1. 8-beat narrator-driven story arc
2. ChatGPT Image 2 storyboard (clay texture prompt)
3. Seedance 2.0 i2v with clay aesthetic
4. FFmpeg stitch with optional fps=12 stop-motion judder
5. External VO (ElevenLabs) mixed in post

**Requirements:** `ffmpeg`, `jq`

See: `shared/skills/claymation-ad/prompting/guide.md`

### Burn Captions onto Finished Video

```bash
# Located at: shared/skills/caption-video/
./shared/skills/caption-video/scripts/add_captions.sh input.mp4 output.mp4
```

**Pipeline:**
1. Whisper transcription (word-level timestamps)
2. Group words into reading phrases
3. HyperFrames render with style config
4. FFmpeg overlay onto source video

**Requirements:** `ffmpeg`, `node`, `whisper` (pip)

Works on any video source (Pixar, claymation, UGC, B-roll).

## Publishing to Meta Ads

```bash
# Located at: shared/skills/meta-ad-builder/
pip install -r shared/skills/meta-ad-builder/scripts/requirements.txt

# Set Meta credentials in .env
FACEBOOK_ACCESS_TOKEN=your_token
FACEBOOK_AD_ACCOUNT_ID=act_123456789

# Publish paused ad
python shared/skills/meta-ad-builder/scripts/publish.py \
  --creative-path output/final-ad.mp4 \
  --headline "Get SuperWidget Today" \
  --description "The widget that changed everything" \
  --cta-type LEARN_MORE \
  --paused
```

## Configuration Files

### MASTER_CONTEXT.md

Personal workspace file created by setup. Use it to store:
- Active campaign context
- Character reference inventory
- Product shot library
- Brand voice guidelines
- Running prompt variations

```markdown
# Current Campaign: SuperWidget Q1 Launch

## Active Characters
- Sofia: references/influencers/sofia/ (10 angles)
- Marcus: references/influencers/marcus/ (10 angles)

## Product Assets
- Hero shot: references/products/superwidget-hero.jpg
- Lifestyle: references/products/superwidget-lifestyle-*.jpg

## Voice
- Casual, benefit-driven, "game changer" positioning
- Avoid: overly technical, corporate speak
```

### Prompt Libraries

Located at: `skills/arcads-external-api/prompting/prompt-library/`

- `seedance-2-ugc.md` — 9-layer UGC formula
- `seedance-2-premium-reveal.md` — Dark void hero
- `seedance-2-product-hero.md` — Elemental effects
- `seedance-2-studio-lookbook.md` — Editorial multi-shot
- `seedance-2-feature-walkthrough.md` — Fast-paced demo

**Usage pattern:**
```python
with open("skills/arcads-external-api/prompting/prompt-library/seedance-2-ugc.md") as f:
    ugc_formula = f.read()

# Inject product-specific details into formula
prompt = ugc_formula.replace("{{PRODUCT}}", "SuperWidget")
```

## Cost Estimation

Always confirm costs before generation:

```python
def estimate_cost(model, duration_or_count, resolution="1080p"):
    """
    Rough cost estimates (check current API pricing):
    - Seedance 2.0: ~$0.10-0.30 per video (varies by duration)
    - Sora 2: ~$0.50-1.00 per video
    - Veo 3.1: ~$0.20-0.40 per video
    - Nano Banana 2: ~$0.05 per image
    - ChatGPT Image 2: ~$0.04 per image
    """
    cost_table = {
        "seedance-2": 0.20 * (duration_or_count / 10),
        "sora-2": 0.75 * (duration_or_count / 16),
        "veo-3.1": 0.30 * (duration_or_count / 8),
        "nano-banana-2": 0.05 * duration_or_count,
        "gpt-image-2": 0.04 * duration_or_count
    }
    
    return cost_table.get(model, 0)

# Example usage
cost = estimate_cost("seedance-2", duration_or_count=12)
print(f"Estimated cost: ${cost:.2f}")
user_confirm = input("Proceed? (y/n): ")
if user_confirm.lower() != 'y':
    exit()
```

## Common Patterns

### Pattern 1: Still → Video Pipeline

```python
def still_to_video_pipeline(product_name, character_description, dialogue):
    """
    1. Generate Nano Banana still
    2. User approval
    3. Animate with Veo 3.1
    """
    # Step 1: Generate still
    still_prompt = f"""
    {character_description} holding {product_name}. Natural expression, 
    looking at camera. Soft lighting, authentic aesthetic.
    """
    
    still_payload = {
        "prompt": still_prompt,
        "model": "nano-banana-2",
        "aspectRatio": "9:16",
        "numberOfImages": 1
    }
    
    still_response = requests.post(
        f"{BASE_URL}/v1/images/generate",
        headers=headers,
        json=still_payload
    )
    
    still_url = poll_until_complete(still_response.json()["id"], "image")
    
    # Step 2: User approval (in agent flow)
    print(f"Review still: {still_url}")
    approval = input("Approve? (y/n): ")
    if approval.lower() != 'y':
        return None
    
    # Step 3: Animate with Veo
    still_image = requests.get(still_url).content
    still_b64 = base64.b64encode(still_image).decode()
    
    video_payload = {
        "prompt": f"Natural motion, delivers: {dialogue}",
        "startFrame": still_b64,
        "duration": 8,
        "aspectRatio": "9:16",
        "model": "veo-3.1",
        "dialogue": dialogue
    }
    
    video_response = requests.post(
        f"{BASE_URL}/v1/videos/generate",
        headers=headers,
        json=video_payload
    )
    
    return poll_until_complete(video_response.json()["id"], "video")
```

### Pattern 2: Batch Generate with References

```python
def batch_generate_with_lockdown(base_prompt, reference_image_path, variations):
    """
    Generate multiple variations while maintaining character consistency
    """
    import base64
    
    with open(reference_image_path, "rb") as f:
        ref_b64 = base64.b64encode(f.read()).decode()
    
    results = []
    for variation in variations:
        prompt = f"{base_prompt}, {variation}. Match exact appearance from reference."
        
        payload = {
            "prompt": prompt,
            "model": "nano-banana-2",
            "aspectRatio": "1:1",
            "refImageAsBase64": ref_b64,
            "numberOfImages": 1
        }
        
        response = requests.post(
            f"{BASE_URL}/v1/images/generate",
            headers=headers,
            json=payload
        )
        
        url = poll_until_complete(response.json()["id"], "image")
        results.append({"variation": variation, "url": url})
    
    return results
```

### Pattern 3: A/B Test Creative Variants

```python
def generate_ab_test_variants(product_name, base_scenes, num_variants=3):
    """
    Generate multiple creative variants for A/B testing
    """
    all_variants = []
    
    for scene_idx, scene in enumerate(base_scenes):
        for variant_idx in range(num_variants):
            # Add variation to prompt
            variations = [
                "with dramatic lighting",
                "with soft natural lighting",
                "with high-energy vibe"
            ]
            
            prompt = f"{scene} {variations[variant_idx]}"
            
            payload = {
                "prompt": prompt,
                "duration": 10,
                "aspectRatio": "9:16",
                "model": "seedance-2"
            }
            
            response = requests.post(
                f"{BASE_URL}/v1/videos/generate",
                headers=headers,
                json=payload
            )
            
            url = poll_until_complete(response.json()["id"], "video")
            
            all_variants.append({
                "scene": scene_idx,
                "variant": variant_idx,
                "style": variations[variant_idx],
                "url": url
            })
    
    return all_variants
```

## Troubleshooting

### Job Stuck in Processing

```python
def poll_with_timeout(job_id, endpoint_type="video", timeout_minutes=10):
    """Enhanced polling with timeout"""
    import time
    
    start_time = time.time()
    timeout_seconds = timeout_minutes * 60
    
    while True:
        elapsed = time.time() - start_time
        if elapsed > timeout_seconds:
            raise TimeoutError(f"Job {job_id} exceeded {timeout_minutes}min timeout")
        
        response = requests.get(
            f"{BASE_URL}/v1/{endpoint_type}s/{job_id}",
            headers=headers
        )
        data = response.json()
        
        status = data.get("status")
        if status == "completed":
            return data.get("output_url") or data.get("outputUrl")
        elif status == "failed":
            error_msg = data.get("error", "Unknown error")
            raise Exception(f"Job failed: {error_msg}")
        
        # Exponential backoff
        wait_time = min(30, 5 * (1 + elapsed // 60))
        time.sleep(wait_time)
```

### API Rate Limiting

```python
import time
from functools import wraps

def rate_limit(calls_per_minute=10):
    """Decorator to rate-limit API calls"""
    min_interval = 60.0 / calls_per_minute
    last_called = [0.0]
    
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            elapsed = time.time() - last_called[0]
            left_to_wait = min_interval - elapsed
            if left_to_wait > 0:
                time.sleep(left_to_wait)
            
            result = func(*args, **kwargs)
            last_called[0] = time.time()
            return result
        return wrapper
    return decorator

@rate_limit(calls_per_minute=10)
def make_api_call(endpoint, payload):
    return requests.post(endpoint, headers=headers, json=payload)
```

### Character Consistency Issues

When character appearance drifts across images:

1. **Use Nano Banana Pro** (`model:
