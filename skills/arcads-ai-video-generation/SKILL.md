---
name: arcads-ai-video-generation
description: Generate AI marketing videos and images using Arcads API with Seedance 2.0, Sora 2, Veo 3.1, Kling 3.0, Nano Banana, and 37-template static ad library
triggers:
  - create an arcads video
  - generate ai marketing video
  - make seedance ugc video
  - create nano banana image ad
  - generate veo animation
  - build pixar style ad campaign
  - use arcads api
  - create ai influencer character
---

# Arcads AI Video Generation

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## What This Project Does

Arcads is an AI video and image generation platform for creating marketing content. This repo provides agent-ready skills for the full Arcads creative stack:

- **Video models**: Seedance 2.0 (flagship), Sora 2, Veo 3.1, Kling 3.0, Grok Video, OmniHuman, Audio-driven
- **Image models**: Nano Banana 2/Pro/Edit, ChatGPT Image 2
- **Static ad library**: 37 validated prompt templates for Meta image ads
- **Multi-step pipelines**: Pixar-style animated ads, claymation ads, YouTube thumbnails

The agent handles API calls, polling, prompt engineering, file organization, and cost confirmation.

## Installation

```bash
# Clone the repository
git clone https://github.com/krusemediallc/arcads-claude-code.git
cd arcads-claude-code

# Run setup (interactive)
./scripts/setup.sh
```

**Setup process:**
1. Creates `.env` file for your Arcads API key (get from [app.arcads.ai/settings/api](https://app.arcads.ai/settings/api))
2. Verifies API connection
3. Creates `MASTER_CONTEXT.md` workspace file
4. Installs dependencies (if needed)

**Required dependencies:**

```bash
# Core (required)
python3.10+  # Preinstalled on macOS

# Optional (per workflow)
brew install ffmpeg jq node  # macOS
apt install ffmpeg jq nodejs python3  # Linux

pip install openai-whisper  # Caption transcription
pip install -r shared/skills/meta-ad-builder/scripts/requirements.txt  # Meta publishing
```

## Configuration

**Environment variables** (`.env` file):

```bash
ARCADS_API_KEY=your_api_key_here
ARCADS_API_BASE=https://api.arcads.ai  # Default
```

**Workspace file** (`MASTER_CONTEXT.md`):
- Personal notes, brand voice, product details
- Referenced by agent for context-aware prompting

## Core API Patterns

### Authentication

All requests require the API key in headers:

```python
import os
import requests

API_KEY = os.getenv("ARCADS_API_KEY")
BASE_URL = os.getenv("ARCADS_API_BASE", "https://api.arcads.ai")

headers = {
    "Authorization": f"Bearer {API_KEY}",
    "Content-Type": "application/json"
}
```

### Polling Pattern

Most generation endpoints return a job ID. Poll for completion:

```python
def poll_job(job_id, endpoint="/v1/jobs/status"):
    url = f"{BASE_URL}{endpoint}/{job_id}"
    
    while True:
        response = requests.get(url, headers=headers)
        data = response.json()
        
        status = data.get("status")
        
        if status == "completed":
            return data.get("result")
        elif status == "failed":
            raise Exception(f"Job failed: {data.get('error')}")
        
        time.sleep(5)  # Poll every 5 seconds
```

## Video Generation

### Seedance 2.0 (Flagship Model)

**UGC selfie-style product review:**

```python
def generate_seedance_ugc(product_name, competitor_name):
    payload = {
        "model": "seedance-2",
        "duration": 12,
        "prompt": f"""
UGC creator video - kitchen setting, natural light:

VISUAL: Young woman (25-32), casual athleisure, holding {product_name} product, 
direct eye contact with camera, authentic smile. iPhone selfie aesthetic - 
slight camera shake, auto-focus shifts.

DIALOGUE: "Okay so I used to buy {competitor_name} every month, but then I 
tried {product_name} and honestly? I'm never going back. The difference is crazy."

PERFORMANCE: Natural vocal fry, conversational pacing, brief eye-contact breaks 
to look at product, genuine enthusiasm (not overselling).

STYLE: Soft kitchen lighting, blurred background (shallow DOF), warm color grade.
        """.strip(),
        "shotType": "ugc-selfie"
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/seedance2/generate",
        headers=headers,
        json=payload
    )
    
    job_id = response.json()["jobId"]
    return poll_job(job_id)
```

**Premium product reveal (no person):**

```python
def generate_premium_reveal(product_name, tagline):
    payload = {
        "model": "seedance-2",
        "duration": 8,
        "prompt": f"""
Premium product reveal - dark void aesthetic:

VISUAL: {product_name} suspended in black void, dramatic spotlight from top-right, 
slow 360° rotation (20 sec/rotation). Product pristine, hero lighting reveals 
texture and finish.

TEXT OVERLAYS (sequenced):
- Beat 1 (0-2s): "{tagline}" (serif font, white, fade in)
- Beat 2 (3-5s): Product rotates, light catches details
- Beat 3 (6-8s): Brand logo materializes bottom-right

STYLE: Cinematic depth of field, no background elements, luxury silence 
(no dialogue/music).
        """.strip(),
        "shotType": "product-hero"
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/seedance2/generate",
        headers=headers,
        json=payload
    )
    
    return poll_job(response.json()["jobId"])
```

**Image-to-video with reference:**

```python
import base64

def seedance_image_to_video(image_path, animation_prompt):
    with open(image_path, "rb") as f:
        image_b64 = base64.b64encode(f.read()).decode()
    
    payload = {
        "model": "seedance-2",
        "duration": 8,
        "startFrame": image_b64,
        "prompt": animation_prompt,
        "shotType": "animation"
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/seedance2/i2v",
        headers=headers,
        json=payload
    )
    
    return poll_job(response.json()["jobId"])
```

### Sora 2 (Text-to-Video)

```python
def generate_sora_video(scene_description, duration=16):
    payload = {
        "model": "sora-2",
        "prompt": scene_description,
        "duration": duration,  # Up to 20 seconds
        "resolution": "1080p"
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/sora2/generate",
        headers=headers,
        json=payload
    )
    
    return poll_job(response.json()["jobId"])
```

**With style reference:**

```python
def generate_sora_with_reference(scene, reference_image_path):
    with open(reference_image_path, "rb") as f:
        ref_b64 = base64.b64encode(f.read()).decode()
    
    payload = {
        "model": "sora-2",
        "prompt": scene,
        "duration": 16,
        "styleReference": ref_b64
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/sora2/generate",
        headers=headers,
        json=payload
    )
    
    return poll_job(response.json()["jobId"])
```

### Veo 3.1 (Starting Frame Animation)

Standard path for **UGC stills → video**:

```python
def animate_with_veo(still_image_path, dialogue_script):
    with open(still_image_path, "rb") as f:
        image_b64 = base64.b64encode(f.read()).decode()
    
    payload = {
        "model": "veo-3.1",
        "startFrame": image_b64,
        "dialogue": dialogue_script,
        "duration": 8,
        "motionIntensity": "natural"  # or "subtle", "dynamic"
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/veo31/animate",
        headers=headers,
        json=payload
    )
    
    return poll_job(response.json()["jobId"])
```

### Kling 3.0 (B-roll/Scene)

```python
def generate_kling_broll(scene_description):
    payload = {
        "prompt": scene_description,
        "duration": 5,
        "category": "broll"  # or "scene"
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/b-roll",
        headers=headers,
        json=payload
    )
    
    return poll_job(response.json()["jobId"])

def generate_kling_scene(environment):
    payload = {
        "prompt": environment,
        "duration": 5
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/scene",
        headers=headers,
        json=payload
    )
    
    return poll_job(response.json()["jobId"])
```

## Image Generation

### Nano Banana (Character/Product Stills)

**Create AI influencer (10-image character sheet):**

```python
def create_ai_influencer(character_description):
    # Step 1: Generate hero portrait
    hero_payload = {
        "model": "nano-banana-2",  # or "nano-banana" for Pro
        "prompt": f"""
Front-facing portrait: {character_description}

CAMERA: iPhone 14 Pro selfie, natural lighting, golden hour
COMPOSITION: Head and shoulders, direct eye contact
DETAILS: Skin texture visible, natural imperfections, authentic expression
STYLE: Warm color grade, soft focus background
        """.strip(),
        "aspectRatio": "9:16",
        "quality": "high"
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/nano-banana/generate",
        headers=headers,
        json=hero_payload
    )
    hero_result = poll_job(response.json()["jobId"])
    hero_url = hero_result["imageUrl"]
    
    # Download hero for reference
    hero_image = requests.get(hero_url).content
    hero_b64 = base64.b64encode(hero_image).decode()
    
    # Step 2: Generate 9 additional angles
    angles = [
        "3/4 profile left, slight smile",
        "3/4 profile right, looking at camera",
        "Full profile left, serious expression",
        "Close-up face, genuine laugh",
        "Upper body, hands visible, casual pose",
        "Full body standing, natural posture",
        "Looking down then up at camera",
        "Candid expression, mid-speech",
        "Side angle, hair visible, natural light"
    ]
    
    additional_images = []
    for angle in angles:
        payload = {
            "model": "nano-banana-2",
            "prompt": f"{character_description} - {angle}",
            "referenceImages": [hero_b64],
            "aspectRatio": "9:16"
        }
        
        response = requests.post(
            f"{BASE_URL}/v1/nano-banana/generate",
            headers=headers,
            json=payload
        )
        result = poll_job(response.json()["jobId"])
        additional_images.append(result["imageUrl"])
    
    return {
        "hero": hero_url,
        "additional": additional_images
    }
```

**UGC product selfie still:**

```python
def generate_ugc_selfie(character_hero_path, product_image_path, product_name):
    with open(character_hero_path, "rb") as f:
        char_b64 = base64.b64encode(f.read()).decode()
    
    with open(product_image_path, "rb") as f:
        prod_b64 = base64.b64encode(f.read()).decode()
    
    payload = {
        "model": "nano-banana-2",
        "prompt": f"""
iPhone selfie - bedroom setting:

CHARACTER: Using provided reference likeness exactly
PRODUCT: Holding {product_name} (using provided product image)
POSE: Arm extended (selfie angle), product at chest height, friendly smile
LIGHTING: Natural window light, soft shadows, warm color temperature
REALISM: Visible skin texture, minor camera shake blur, authentic expression
COMPOSITION: 9:16 vertical, character's face and product both in focus
        """.strip(),
        "referenceImages": [char_b64, prod_b64],
        "aspectRatio": "9:16",
        "styleWeight": 0.7  # Balance character likeness vs scene composition
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/nano-banana/generate",
        headers=headers,
        json=payload
    )
    
    return poll_job(response.json()["jobId"])
```

**Nano Banana Edit (inpainting):**

```python
def edit_image_region(base_image_path, mask_image_path, edit_prompt):
    with open(base_image_path, "rb") as f:
        base_b64 = base64.b64encode(f.read()).decode()
    
    with open(mask_image_path, "rb") as f:
        mask_b64 = base64.b64encode(f.read()).decode()
    
    payload = {
        "model": "nano-banana-edit",
        "baseImage": base_b64,
        "maskImage": mask_b64,  # White = edit region, black = preserve
        "prompt": edit_prompt
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/nano-banana/edit",
        headers=headers,
        json=payload
    )
    
    return poll_job(response.json()["jobId"])
```

### ChatGPT Image 2 (Typography/UI-Heavy)

```python
def generate_chatgpt_image(prompt, aspect_ratio="1:1"):
    payload = {
        "model": "gpt-image-2",
        "prompt": prompt,
        "aspectRatio": aspect_ratio,
        "quality": "hd"
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/chatgpt-image/generate",
        headers=headers,
        json=payload
    )
    
    return poll_job(response.json()["jobId"])
```

## Static Meta Image Ads (37-Template Library)

### Apple Notes List Ad

```python
def generate_apple_notes_ad(headline, bullet_points):
    prompt = f"""
Apple Notes app screenshot - iOS 17 aesthetic:

HEADER: "{headline}" (San Francisco font, bold, 28pt)

LIST ITEMS:
{chr(10).join(f"• {point}" for point in bullet_points)}

STYLE:
- Cream background (#FFFEF7)
- Default iOS Notes styling
- Checkboxes unchecked
- Realistic screen glare (subtle)
- Battery/signal indicators top-right
- Time shows 9:41 AM

COMPOSITION: Portrait 9:16, centered text, margins match iOS defaults
    """.strip()
    
    return generate_chatgpt_image(prompt, "9:16")
```

### Comparison Table Ad

```python
def generate_comparison_table(your_product, competitor, features):
    prompt = f"""
Product comparison table - editorial style:

HEADER: Bold sans-serif, "{your_product} vs {competitor}"

TABLE:
| Feature | {your_product} | {competitor} |
|---------|----------------|--------------|
{chr(10).join(f"| {feat['name']} | {feat['yours']} ✓ | {feat['theirs']} ✗ |" for feat in features)}

STYLE:
- Clean grid lines
- Green checkmarks for your product
- Red X's for competitor
- Professional color scheme (navy header, white bg)
- Subtle drop shadow on table

COMPOSITION: Square 1:1, table fills 80% of canvas
    """.strip()
    
    return generate_chatgpt_image(prompt, "1:1")
```

### Fake iMessage Screenshot

```python
def generate_imessage_ad(sender_name, messages):
    prompt = f"""
iPhone iMessage conversation screenshot - iOS 17:

CONTACT: "{sender_name}" (top, with profile pic circle)

MESSAGES (chronological, alternating gray/blue bubbles):
{chr(10).join(f"{'[Them]' if i % 2 == 0 else '[You]'}: {msg}" for i, msg in enumerate(messages))}

DETAILS:
- Blue bubbles (your messages) on right
- Gray bubbles (their messages) on left
- Timestamps every 2-3 messages
- "Delivered" under last message
- iOS keyboard visible at bottom
- Realistic screen bezel

COMPOSITION: 9:16 portrait, authentic iOS spacing
    """.strip()
    
    return generate_chatgpt_image(prompt, "9:16")
```

## Multi-Step Pipelines

### Pixar-Style Animated Ad

```bash
# Using the bash script wrapper
./shared/skills/pixar-style-ad/scripts/generate.sh \
  --product "SuperWidget Pro" \
  --duration 60 \
  --beats 8 \
  --output ./output/pixar-ad/
```

**Python implementation:**

```python
def generate_pixar_ad(product_name, story_beats):
    # Step 1: Lock character cast
    cast = [
        {"role": "hero", "description": "Anthropomorphic product mascot, Pixar style"},
        {"role": "sidekick", "description": "Friendly companion character"}
    ]
    
    # Step 2: Generate storyboard stills (ChatGPT Image 2)
    storyboard = []
    previous_frame = None
    
    for i, beat in enumerate(story_beats):
        refs = [previous_frame] if previous_frame else []
        
        prompt = f"""
Pixar 3D animated style - Beat {i+1}/{len(story_beats)}:

SCENE: {beat['scene']}
CHARACTERS: {beat['characters']}
ACTION: {beat['action']}

STYLE: Pixar-quality 3D render, vibrant colors, expressive faces, 
soft lighting, shallow depth of field.

COMPOSITION: 16:9 cinematic, rule of thirds.
        """.strip()
        
        payload = {
            "model": "gpt-image-2",
            "prompt": prompt,
            "referenceImages": refs[:5],  # Max 5 refs
            "aspectRatio": "16:9"
        }
        
        response = requests.post(
            f"{BASE_URL}/v1/chatgpt-image/generate",
            headers=headers,
            json=payload
        )
        
        result = poll_job(response.json()["jobId"])
        storyboard.append(result["imageUrl"])
        previous_frame = base64.b64encode(requests.get(result["imageUrl"]).content).decode()
    
    # Step 3: Animate each still (Seedance 2.0 i2v)
    video_clips = []
    for i, still_url in enumerate(storyboard):
        still_b64 = base64.b64encode(requests.get(still_url).content).decode()
        
        anim_payload = {
            "model": "seedance-2",
            "duration": 8,
            "startFrame": still_b64,
            "prompt": f"Animate: {story_beats[i]['action']}",
            "shotType": "animation"
        }
        
        response = requests.post(
            f"{BASE_URL}/v1/seedance2/i2v",
            headers=headers,
            json=anim_payload
        )
        
        clip = poll_job(response.json()["jobId"])
        video_clips.append(clip["videoUrl"])
    
    # Step 4: Stitch with ffmpeg (external)
    return video_clips  # Stitch externally with ffmpeg concat
```

### YouTube Thumbnail Generation

```python
def generate_youtube_thumbnail_batch(face_refs, product_image, headline, formula="peace-sign"):
    formulas = {
        "peace-sign": "Person making peace sign, big smile, product visible, energetic",
        "real-vs-ai": "Split screen: left = 'REAL', right = 'AI', dramatic comparison",
        "reaction-shock": "Exaggerated shocked expression, mouth open, pointing at product",
        "before-after": "Side-by-side: before (dull) vs after (vibrant)",
        "terminal-flow": "Terminal/code aesthetic, tech vibes, product in foreground"
    }
    
    formula_prompt = formulas.get(formula, formulas["peace-sign"])
    
    # Encode face references (5+ for likeness lock)
    face_b64s = []
    for ref_path in face_refs:
        with open(ref_path, "rb") as f:
            face_b64s.append(base64.b64encode(f.read()).decode())
    
    with open(product_image, "rb") as f:
        prod_b64 = base64.b64encode(f.read()).decode()
    
    # Generate 6 variations in parallel
    jobs = []
    for i in range(6):
        payload = {
            "model": "nano-banana-2",
            "prompt": f"""
YouTube thumbnail - {formula_prompt}

TEXT OVERLAY: "{headline}" (bold, yellow outline, top or bottom)
PERSON: Using exact likeness from references
PRODUCT: {product_image} prominent in frame
STYLE: High contrast, saturated colors, eye-catching
COMPOSITION: 16:9, face + text + product all visible

Variation {i+1}: {"Slight angle change" if i > 0 else "Primary composition"}
            """.strip(),
            "referenceImages": face_b64s + [prod_b64],
            "aspectRatio": "16:9",
            "quality": "high"
        }
        
        response = requests.post(
            f"{BASE_URL}/v1/nano-banana/generate",
            headers=headers,
            json=payload
        )
        jobs.append(response.json()["jobId"])
    
    # Poll all jobs
    return [poll_job(job_id) for job_id in jobs]
```

## Advanced Workflows

### Caption Burn-In (Post-Processing)

```python
import subprocess
import json

def burn_captions(video_path, output_path):
    # Step 1: Transcribe with Whisper
    whisper_result = subprocess.run(
        ["whisper", video_path, "--model", "medium.en", "--output_format", "json"],
        capture_output=True,
        text=True
    )
    
    transcript = json.loads(whisper_result.stdout)
    
    # Step 2: Group into reading phrases (external lib or custom logic)
    phrases = group_into_phrases(transcript["segments"])
    
    # Step 3: Render with HyperFrames
    hyperframes_config = {
        "video": video_path,
        "captions": [
            {"start": p["start"], "end": p["end"], "text": p["text"]}
            for p in phrases
        ],
        "style": {
            "fontFamily": "Inter",
            "fontSize": 48,
            "color": "#FFFFFF",
            "backgroundColor": "#000000",
            "position": "bottom"
        }
    }
    
    with open("captions_config.json", "w") as f:
        json.dump(hyperframes_config, f)
    
    subprocess.run([
        "npx", "hyperframes",
        "--config", "captions_config.json",
        "--output", output_path
    ])
    
    return output_path

def group_into_phrases(segments, max_words=5):
    phrases = []
    current = {"text": "", "start": 0, "end": 0, "words": 0}
    
    for seg in segments:
        words = seg["text"].split()
        
        if current["words"] + len(words) > max_words:
            if current["text"]:
                phrases.append(current)
            current = {
                "text": seg["text"],
                "start": seg["start"],
                "end": seg["end"],
                "words": len(words)
            }
        else:
            current["text"] += " " + seg["text"]
            current["end"] = seg["end"]
            current["words"] += len(words)
    
    if current["text"]:
        phrases.append(current)
    
    return phrases
```

### Meta Ad Publishing (via Marketing API)

```python
# Requires: pip install facebook-business
from facebook_business.api import FacebookAdsApi
from facebook_business.adobjects.adcreative import AdCreative
from facebook_business.adobjects.adimage import AdImage

def publish_to_meta(image_url, headline, ad_account_id):
    # Initialize Meta API (requires META_ACCESS_TOKEN env var)
    access_token = os.getenv("META_ACCESS_TOKEN")
    app_id = os.getenv("META_APP_ID")
    app_secret = os.getenv("META_APP_SECRET")
    
    FacebookAdsApi.init(app_id, app_secret, access_token)
    
    # Download image
    image_data = requests.get(image_url).content
    
    # Upload to Meta
    image = AdImage(parent_id=ad_account_id)
    image[AdImage.Field.bytes] = image_data
    image.remote_create()
    
    # Create ad creative (paused)
    creative = AdCreative(parent_id=ad_account_id)
    creative[AdCreative.Field.name] = headline
    creative[AdCreative.Field.object_story_spec] = {
        "page_id": os.getenv("META_PAGE_ID"),
        "link_data": {
            "image_hash": image[AdImage.Field.hash],
            "link": os.getenv("LANDING_PAGE_URL"),
            "message": headline
        }
    }
    creative.remote_create()
    
    return creative.get_id()
```

## Common Patterns

### Batch Processing with Rate Limiting

```python
import time
from concurrent.futures import ThreadPoolExecutor, as_completed

def batch_generate(prompts, model="seedance-2", max_workers=3):
    results = []
    
    with ThreadPoolExecutor(max_workers=max_workers) as executor:
        futures = []
        
        for prompt in prompts:
            payload = {
                "model": model,
                "prompt": prompt,
                "duration": 8
            }
            
            future = executor.submit(
                requests.post,
                f"{BASE_URL}/v1/{model}/generate",
                headers=headers,
                json=payload
            )
            futures.append(future)
            
            # Rate limit: 3 req/sec
            time.sleep(0.33)
        
        for future in as_completed(futures):
            response = future.result()
            job_id = response.json()["jobId"]
            result = poll_job(job_id)
            results.append(result)
    
    return results
```

### Cost Estimation Before Generation

```python
PRICING = {
    "seedance-2": 0.15,  # per second
    "sora-2": 0.20,
    "veo-3.1": 0.12,
    "kling-3.0": 0.08,
    "nano-banana-2": 0.05,  # per image
    "gpt-image-2": 0.04
}

def estimate_cost(model, duration=None, count=1):
    if model in ["nano-banana-2", "gpt-image-2"]:
        return PRICING[model] * count
    else:
        return PRICING[model] * duration * count

def confirm_generation(model, duration, count=1):
    cost = estimate_cost(model, duration, count)
    print(f"Estimated cost: ${cost:.2f} ({count}x {duration}s {model})")
    response = input("Proceed? (y/n): ")
    return response.lower() == "y"
```

### Error Handling and Retries

```python
import time
from requests.exceptions import RequestException

def safe_api_call(method, url, max_retries=3, **kwargs):
    for attempt in range(max_retries):
        try:
            response = method(url, **kwargs)
            response.raise_for_status()
            return response.json()
        
        except RequestException as e:
            if attempt == max_retries - 1:
                raise Exception(f"API call failed after {max_retries} attempts: {e}")
            
            wait_time = 2 ** attempt  # Exponential backoff
            print(f"Retry {attempt + 1}/{max_retries} after {wait_time}s...")
            time.sleep(wait_time)

def poll_job_safe(job_id, timeout=600):
    start_time = time.time()
    
    while time.time() - start_time < timeout:
        try:
            data = safe_api_call(
                requests.get,
                f"{BASE_URL}/v1/jobs/status/{job_id}",
                headers=headers
            )
            
            if data["status"] == "completed":
                return data["result"]
            elif data["status"] == "failed":
                raise Exception(f"Job failed: {data.get('error')}")
            
            time.sleep(5)
        
        except Exception as e:
            print(f"Polling error: {e}")
            time.sleep(10)
    
    raise TimeoutError(f"Job {job_id} timed out after {timeout}s")
```

## Troubleshooting

### API Key Issues

```python
def verify_api_key():
    try:
        response = requests.get(
            f"{BASE_URL}/v1/account",
            headers=headers
        )
        
        if response.status_code == 401:
            print("❌ Invalid API key. Get yours at: https://app.arcads.ai/settings/api")
            return False
        
        data = response.json()
        print(f"✓ Connected as: {data['email']}")
        print(f"✓ Credits remaining: {data['credits']}")
        return True
    
    except Exception as e:
        print(f"❌ Connection error: {e}")
        return False
```

### Job Stuck in Processing
