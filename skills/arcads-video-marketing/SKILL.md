---
name: arcads-video-marketing
description: Generate AI marketing videos and static image ads using Arcads external API with Seedance 2.0, Sora 2, Veo 3.1, Kling, Nano Banana, and 37-template Meta image library
triggers:
  - create an arcads video
  - generate ugc video with seedance
  - make a nano banana image
  - create ai influencer character sheet
  - generate meta image ad
  - animate product with veo
  - build pixar style ad
  - make claymation video campaign
---

# Arcads AI Video Marketing Skill

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

Create AI marketing videos and static image ads using the [Arcads](https://arcads.ai) external API. Supports the full creative stack: **Seedance 2.0** (flagship video model), **Sora 2**, **Veo 3.1**, **Kling 3.0**, **Grok Video**, **Nano Banana 2/Pro/Edit**, **ChatGPT Image 2**, **OmniHuman**, and **Audio-driven** video generation, plus a 37-template Meta image-ad library and multi-step pipelines for Pixar-style and claymation animated ads.

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/krusemediallc/arcads-claude-code.git
cd arcads-claude-code
```

### 2. Run setup script

```bash
./scripts/setup.sh
```

This will:
- Prompt for your Arcads API key (get it at [app.arcads.ai/settings/api](https://app.arcads.ai/settings/api))
- Create `.env` file with your credentials
- Verify API connection
- Generate `MASTER_CONTEXT.md` workspace file

### 3. Install optional dependencies (for multi-step pipelines)

```bash
# macOS
brew install ffmpeg jq node

# Linux
apt install ffmpeg jq nodejs python3

# Python packages for specific workflows
pip install openai-whisper  # For caption transcription
pip install -r shared/skills/meta-ad-builder/scripts/requirements.txt  # For Meta API publishing
```

## Core API Structure

All API calls go through the Arcads base URL. The API key is stored in `.env`:

```bash
ARCADS_API_KEY=your_api_key_here
```

### Basic request pattern (Python)

```python
import os
import requests
import time
from dotenv import load_dotenv

load_dotenv()
api_key = os.getenv("ARCADS_API_KEY")
base_url = "https://api.arcads.ai"

headers = {
    "Authorization": f"Bearer {api_key}",
    "Content-Type": "application/json"
}

# Generate video
response = requests.post(
    f"{base_url}/v1/seedance-2/video",
    headers=headers,
    json={
        "prompt": "Woman in kitchen reviewing product, natural iPhone aesthetic",
        "duration": 12,
        "aspectRatio": "9:16"
    }
)

job_id = response.json()["jobId"]

# Poll for completion
while True:
    status_resp = requests.get(
        f"{base_url}/v1/jobs/{job_id}",
        headers=headers
    )
    status = status_resp.json()
    
    if status["status"] == "completed":
        video_url = status["result"]["videoUrl"]
        print(f"Video ready: {video_url}")
        break
    elif status["status"] == "failed":
        print(f"Failed: {status['error']}")
        break
    
    time.sleep(5)
```

## Video Generation Models

### Seedance 2.0 (Flagship Model)

**Endpoints:**
- `POST /v1/seedance-2/video` - Text or image-to-video (4-15s)
- `POST /v1/seedance-2/video-to-video` - Video-to-video transformation

**Key parameters:**
- `prompt` (string, required) - Scene description
- `duration` (int, 4-15) - Video length in seconds
- `aspectRatio` (string) - "16:9", "9:16", "1:1", "4:5"
- `startFrame` (string, optional) - Base64 image to start from
- `referenceImages` (array, optional) - Up to 3 base64 reference images
- `dialogue` (string, optional) - Embedded speech
- `shotStyle` (string, optional) - "static", "dynamic", "cinematic"

**Example: UGC selfie-style product review**

```python
def generate_seedance_ugc(product_name, duration=12):
    """Generate UGC video using 9-layer Seedance formula"""
    
    prompt = f"""
    iPhone selfie video, woman in bright kitchen holding {product_name}.
    Natural eye-contact breaks, authentic delivery, vertical frame.
    Warm lighting, casual outfit, product visible in hand.
    Looking directly at camera, slight hand gestures, genuine smile.
    Background: real kitchen counter, soft focus.
    """
    
    payload = {
        "prompt": prompt.strip(),
        "duration": duration,
        "aspectRatio": "9:16",
        "shotStyle": "static",
        "dialogue": f"I used to buy [competitor] but this {product_name} is so much better"
    }
    
    response = requests.post(
        f"{base_url}/v1/seedance-2/video",
        headers=headers,
        json=payload
    )
    
    return response.json()["jobId"]
```

**Example: Premium product reveal (no person)**

```python
def generate_premium_reveal(product_name, features):
    """Dark void aesthetic with text narrative"""
    
    prompt = f"""
    {product_name} floating in dark void, dramatic spotlight from above.
    Slow 360-degree rotation revealing product details.
    Matte black background, professional studio lighting.
    Text overlays appear: "{features}".
    Cinematic depth, premium aesthetic, no person visible.
    """
    
    payload = {
        "prompt": prompt.strip(),
        "duration": 10,
        "aspectRatio": "1:1",
        "shotStyle": "cinematic"
    }
    
    response = requests.post(
        f"{base_url}/v1/seedance-2/video",
        headers=headers,
        json=payload
    )
    
    return response.json()["jobId"]
```

**Example: Image-to-video with reference**

```python
import base64

def seedance_with_reference(prompt_text, image_path, duration=8):
    """Start from a static image and animate it"""
    
    with open(image_path, "rb") as f:
        img_b64 = base64.b64encode(f.read()).decode()
    
    payload = {
        "prompt": prompt_text,
        "duration": duration,
        "aspectRatio": "9:16",
        "startFrame": img_b64,
        "shotStyle": "dynamic"
    }
    
    response = requests.post(
        f"{base_url}/v1/seedance-2/video",
        headers=headers,
        json=payload
    )
    
    return response.json()["jobId"]
```

### Sora 2 (Long-form text-to-video)

**Endpoint:** `POST /v1/sora2/video`

**Parameters:**
- `prompt` (string, required)
- `duration` (int, 4-20) - Up to 20 seconds
- `aspectRatio` (string)
- `referenceImages` (array, optional) - Style references

```python
def generate_sora_video(scene_description, duration=16):
    """Sora 2 for longer-duration narrative videos"""
    
    payload = {
        "prompt": scene_description,
        "duration": duration,
        "aspectRatio": "16:9"
    }
    
    response = requests.post(
        f"{base_url}/v1/sora2/video",
        headers=headers,
        json=payload
    )
    
    return response.json()["jobId"]
```

### Veo 3.1 (Start-frame animation)

**Endpoint:** `POST /v1/veo3/video`

**Best for:** Animating static UGC images into video with dialogue

```python
def animate_with_veo(image_path, dialogue_text, duration=8):
    """Animate a still image with embedded dialogue"""
    
    with open(image_path, "rb") as f:
        img_b64 = base64.b64encode(f.read()).decode()
    
    payload = {
        "startFrame": img_b64,
        "dialogue": dialogue_text,
        "duration": duration,
        "aspectRatio": "9:16",
        "prompt": "Natural human motion, authentic delivery, maintain starting pose"
    }
    
    response = requests.post(
        f"{base_url}/v1/veo3/video",
        headers=headers,
        json=payload
    )
    
    return response.json()["jobId"]
```

### Kling 3.0 (B-roll and scenes)

**Endpoints:**
- `POST /v1/b-roll` - Quick environmental clips
- `POST /v1/scene` - Narrative scene generation

```python
def generate_broll(scene_type, duration=5):
    """Generate b-roll footage"""
    
    broll_prompts = {
        "coffee": "Steam rising from fresh coffee in ceramic mug, morning sunlight, warm tones",
        "hands": "Hands typing on laptop keyboard, overhead shot, modern workspace",
        "product": "Product on clean white surface, soft shadows, professional lighting"
    }
    
    payload = {
        "prompt": broll_prompts.get(scene_type, scene_type),
        "duration": duration,
        "aspectRatio": "16:9"
    }
    
    response = requests.post(
        f"{base_url}/v1/b-roll",
        headers=headers,
        json=payload
    )
    
    return response.json()["jobId"]
```

### Grok Video

**Endpoint:** `POST /v2/videos/generate`

```python
def generate_grok_video(prompt_text, duration=10):
    """Generate video using Grok Video model"""
    
    payload = {
        "model": "grok-video",
        "prompt": prompt_text,
        "duration": duration,
        "aspectRatio": "16:9"
    }
    
    response = requests.post(
        f"{base_url}/v2/videos/generate",
        headers=headers,
        json=payload
    )
    
    return response.json()["jobId"]
```

## Image Generation

### Nano Banana (Character-consistent images)

**Endpoints:**
- `POST /v1/nano-banana-2` - Default model
- `POST /v1/nano-banana` - Pro version (Gemini 3, tighter identity lock)
- `POST /v1/nano-banana-edit` - Inpainting

**Parameters:**
- `prompt` (string, required)
- `aspectRatio` (string)
- `referenceImages` (array, max 5) - For character consistency
- `refImageAsBase64` (string, optional) - Base reference

**Example: Create AI influencer character sheet**

```python
def create_ai_influencer(description, output_dir="references/influencers/"):
    """Generate 10-image character sheet for consistent AI influencer"""
    
    # Phase 1: Generate hero portrait
    hero_prompt = f"""
    {description}
    Front-facing portrait, direct eye contact, natural smile.
    Professional but approachable lighting, sharp focus on face.
    Neutral background, shoulders visible, genuine expression.
    """
    
    response = requests.post(
        f"{base_url}/v1/nano-banana-2",
        headers=headers,
        json={
            "prompt": hero_prompt.strip(),
            "aspectRatio": "4:5"
        }
    )
    
    hero_job_id = response.json()["jobId"]
    
    # Wait for hero to complete
    hero_img = poll_and_download(hero_job_id)
    
    print("Hero portrait ready. Generating 9 additional angles...")
    
    # Phase 2: Generate 9 additional angles using hero as reference
    with open(hero_img, "rb") as f:
        hero_b64 = base64.b64encode(f.read()).decode()
    
    angles = [
        "3/4 profile view, slight turn to left",
        "3/4 profile view, slight turn to right",
        "Full profile view, side of face",
        "Close-up of face, tighter crop",
        "Laughing expression, natural joy",
        "Serious expression, focused",
        "Upper body, arms visible, casual pose",
        "Holding phone, looking at camera",
        "In different lighting, golden hour"
    ]
    
    job_ids = []
    for angle in angles:
        payload = {
            "prompt": f"{description}. {angle}",
            "aspectRatio": "4:5",
            "referenceImages": [hero_b64]
        }
        
        resp = requests.post(
            f"{base_url}/v1/nano-banana-2",
            headers=headers,
            json=payload
        )
        job_ids.append(resp.json()["jobId"])
    
    return hero_job_id, job_ids
```

**Example: UGC product selfie still**

```python
def generate_ugc_selfie(character_ref_path, product_ref_path, setting="bedroom"):
    """Generate authentic UGC selfie with product"""
    
    # Load reference images
    with open(character_ref_path, "rb") as f:
        char_b64 = base64.b64encode(f.read()).decode()
    
    with open(product_ref_path, "rb") as f:
        prod_b64 = base64.b64encode(f.read()).decode()
    
    # Load UGC aesthetic references
    aesthetic_refs = []
    aesthetic_dir = "references/aesthetics/ugc-selfie/"
    for img_file in os.listdir(aesthetic_dir)[:3]:  # Max 3 aesthetic refs
        with open(os.path.join(aesthetic_dir, img_file), "rb") as f:
            aesthetic_refs.append(base64.b64encode(f.read()).decode())
    
    prompt = f"""
    iPhone selfie in {setting}, natural lighting, authentic feel.
    Person holding product at eye level, slight smile.
    Visible skin texture, natural imperfections, not airbrushed.
    Slightly off-center composition, casual framing.
    Background: real {setting} environment, lived-in feel.
    """
    
    payload = {
        "prompt": prompt.strip(),
        "aspectRatio": "4:5",
        "referenceImages": [char_b64, prod_b64] + aesthetic_refs
    }
    
    response = requests.post(
        f"{base_url}/v1/nano-banana-2",
        headers=headers,
        json=payload
    )
    
    return response.json()["jobId"]
```

### ChatGPT Image 2 (Typography & UI-heavy)

**Endpoint:** `POST /v1/chatgpt-image-2`

**Best for:** UI mockups, text-heavy ads, screenshots

```python
def generate_chatgpt_image(prompt_text, aspect_ratio="1:1"):
    """Generate image using GPT Image 2 (good for text/UI)"""
    
    payload = {
        "prompt": prompt_text,
        "aspectRatio": aspect_ratio
    }
    
    response = requests.post(
        f"{base_url}/v1/chatgpt-image-2",
        headers=headers,
        json=payload
    )
    
    return response.json()["jobId"]
```

## Static Meta Image Ad Library (37 Templates)

### Using the image-ad skills

The repo includes three specialized skills for static Meta image ads:

1. **chatgpt-image-ad** - Typography/UI-heavy templates
2. **nano-banana-image-ad** - Photoreal/lifestyle templates
3. **image-ad-clone** - Reverse-engineer existing ads into templates

**Template categories:**
- UI mockups (Apple Notes, Slack, iMessage, ChatGPT conversation)
- Editorial (Forbes hero, magazine cover, newspaper)
- Comparison tables
- Fake Google search results
- Sticky-note flatlays
- Billboard/outdoor
- Museum exhibit
- Weather forecast UI
- Scratch-off ticket
- Founder letter
- Dating app card
- And 25+ more

**Example: Generate Apple Notes-style ad**

```python
def generate_apple_notes_ad(product_name, benefits_list):
    """Create Apple Notes checklist-style ad"""
    
    notes_text = f"{product_name} Benefits:\n"
    for benefit in benefits_list:
        notes_text += f"☐ {benefit}\n"
    
    prompt = f"""
    iPhone Notes app screenshot, light theme, Helvetica font.
    Title: "{product_name} Benefits"
    Checklist items:
    {notes_text}
    Clean iOS interface, realistic shadows, authentic Notes UI.
    Yellow header bar, white background, black text.
    """
    
    payload = {
        "prompt": prompt.strip(),
        "aspectRatio": "4:5"
    }
    
    response = requests.post(
        f"{base_url}/v1/chatgpt-image-2",
        headers=headers,
        json=payload
    )
    
    return response.json()["jobId"]
```

**Example: Comparison table ad**

```python
def generate_comparison_table(your_product, competitor, comparisons):
    """Generate side-by-side comparison table"""
    
    table_prompt = f"""
    Professional comparison table, clean design, centered layout.
    Two columns: "{your_product}" vs "{competitor}"
    Header row with product names in bold.
    
    Comparison rows:
    """
    
    for feature, your_val, comp_val in comparisons:
        table_prompt += f"\n{feature}: {your_val} | {comp_val}"
    
    table_prompt += """
    
    Green checkmarks for your product, red X for competitor.
    White background, subtle grid lines, modern sans-serif.
    Bottom CTA: "Switch Today"
    """
    
    payload = {
        "prompt": table_prompt.strip(),
        "aspectRatio": "1:1"
    }
    
    response = requests.post(
        f"{base_url}/v1/chatgpt-image-2",
        headers=headers,
        json=payload
    )
    
    return response.json()["jobId"]
```

**Clone existing ad workflow:**

```bash
# Use image-ad-clone skill to reverse-engineer an ad
# 1. User provides reference image path
# 2. Agent analyzes composition, text, style
# 3. Generates reusable prompt template
# 4. Tests against appropriate backend (chatgpt-image-2 or nano-banana-2)
# 5. User validates output
# 6. Template saved to shared/skills/image-ad-prompting/templates/
```

## Job Polling and Status

**Endpoint:** `GET /v1/jobs/{jobId}`

**Response statuses:**
- `pending` - Job queued
- `processing` - Generation in progress
- `completed` - Asset ready
- `failed` - Error occurred

```python
def poll_and_download(job_id, output_path="output/", poll_interval=5):
    """Poll job until complete and download result"""
    
    while True:
        response = requests.get(
            f"{base_url}/v1/jobs/{job_id}",
            headers=headers
        )
        
        status_data = response.json()
        status = status_data["status"]
        
        if status == "completed":
            asset_url = status_data["result"].get("videoUrl") or status_data["result"].get("imageUrl")
            filename = f"{job_id}.{asset_url.split('.')[-1]}"
            
            # Download asset
            asset_response = requests.get(asset_url)
            output_file = os.path.join(output_path, filename)
            
            with open(output_file, "wb") as f:
                f.write(asset_response.content)
            
            print(f"Downloaded: {output_file}")
            return output_file
            
        elif status == "failed":
            error_msg = status_data.get("error", "Unknown error")
            raise Exception(f"Job failed: {error_msg}")
        
        time.sleep(poll_interval)
```

## Multi-Step Pipelines

### Pixar-Style 3D Animated Ad

**Workflow:**
1. Lock character cast sheet (2-3 characters)
2. Generate 8-beat storyboard stills (ChatGPT Image 2)
3. Animate each beat with Seedance 2.0 (i2v)
4. Stitch clips with ffmpeg
5. Burn captions with HyperFrames

```bash
#!/bin/bash
# Generate Pixar-style ad (requires ffmpeg, jq, hyperframes)

PRODUCT_NAME="$1"
OUTPUT_DIR="output/pixar-ads/$(date +%s)"
mkdir -p "$OUTPUT_DIR"

# Phase 1: Generate character cast sheet
python3 << EOF
import requests, os, base64
api_key = os.getenv("ARCADS_API_KEY")
headers = {"Authorization": f"Bearer {api_key}", "Content-Type": "application/json"}

# Hero character
hero_prompt = """
3D Pixar style character: anthropomorphized ${PRODUCT_NAME} mascot.
Large expressive eyes, friendly smile, glossy finish.
Round body shape, cute proportions, vibrant colors.
Full body view, T-pose, clean white background.
"""

resp = requests.post(
    "https://api.arcads.ai/v1/chatgpt-image-2",
    headers=headers,
    json={"prompt": hero_prompt.strip(), "aspectRatio": "1:1"}
)
print(resp.json()["jobId"])
EOF
```

### Claymation/Aardman-Style Ad

**Similar to Pixar pipeline but with clay textures:**

```python
def generate_claymation_beat(beat_description, prior_frame_path=None):
    """Generate one beat of claymation animation"""
    
    clay_prompt = f"""
    Claymation style, sculpted plasticine characters.
    Stop-motion aesthetic, visible fingerprints in clay.
    Warm studio lighting, handcrafted set pieces.
    Slightly imperfect textures, authentic claymation feel.
    
    Scene: {beat_description}
    """
    
    payload = {
        "prompt": clay_prompt.strip(),
        "aspectRatio": "16:9"
    }
    
    # Use prior frame for continuity
    if prior_frame_path:
        with open(prior_frame_path, "rb") as f:
            payload["referenceImages"] = [base64.b64encode(f.read()).decode()]
    
    # Generate still
    still_resp = requests.post(
        f"{base_url}/v1/chatgpt-image-2",
        headers=headers,
        json=payload
    )
    
    still_job = still_resp.json()["jobId"]
    still_path = poll_and_download(still_job)
    
    # Animate with Seedance
    with open(still_path, "rb") as f:
        img_b64 = base64.b64encode(f.read()).decode()
    
    video_resp = requests.post(
        f"{base_url}/v1/seedance-2/video",
        headers=headers,
        json={
            "startFrame": img_b64,
            "prompt": "Subtle claymation motion, stop-motion jitter",
            "duration": 5,
            "aspectRatio": "16:9"
        }
    )
    
    return video_resp.json()["jobId"]
```

### Caption Burn-In Workflow

**Dependencies:** `ffmpeg`, `whisper`, `hyperframes`

```bash
#!/bin/bash
# Burn captions onto video

VIDEO_PATH="$1"
OUTPUT_PATH="${VIDEO_PATH%.mp4}_captioned.mp4"

# Extract audio
ffmpeg -i "$VIDEO_PATH" -vn -acodec pcm_s16le -ar 16000 temp_audio.wav

# Transcribe with Whisper
whisper temp_audio.wav --model medium.en --output_format json

# Generate captions with HyperFrames
npx hyperframes \
  --video "$VIDEO_PATH" \
  --transcript temp_audio.json \
  --output "$OUTPUT_PATH" \
  --style modern \
  --color "#FFFFFF" \
  --bgcolor "#000000"

echo "Captioned video: $OUTPUT_PATH"
```

## Configuration Files

### .env structure

```bash
ARCADS_API_KEY=your_api_key_here

# Optional: For Meta ad publishing
META_ACCESS_TOKEN=your_meta_token_here
META_AD_ACCOUNT_ID=act_123456789
```

### MASTER_CONTEXT.md

Personal workspace file for tracking:
- Active campaigns
- Character reference paths
- Product catalog
- Brand guidelines
- Approved prompts
- Template library additions

## Cost Management

The agent confirms estimated costs before expensive operations:

```python
def estimate_cost(model, duration=None, count=1):
    """Estimate generation cost"""
    
    price_table = {
        "seedance-2": 0.15,  # per video
        "sora-2": 0.20,
        "veo-3": 0.18,
        "kling-3": 0.12,
        "nano-banana-2": 0.05,  # per image
        "chatgpt-image-2": 0.03
    }
    
    unit_cost = price_table.get(model, 0.10)
    total = unit_cost * count
    
    print(f"Estimated cost: ${total:.2f} ({count}x {model})")
    
    confirm = input("Proceed? (y/n): ")
    return confirm.lower() == 'y'
```

## Troubleshooting

### Common Issues

**1. Job stuck in "pending"**
```python
# Check API status
response = requests.get(f"{base_url}/v1/status", headers=headers)
print(response.json())
```

**2. "Invalid aspect ratio" error**

Supported ratios vary by model:
- Seedance 2.0: 16:9, 9:16, 1:1, 4:5
- Sora 2: 16:9, 9:16, 1:1
- Veo 3.1: 9:16, 16:9
- Nano Banana: 1:1, 4:5, 16:9

**3. Character consistency issues**

Use **Nano Banana Pro** instead of default:
```python
payload = {
    "model": "nano-banana",  # Pro version
    "prompt": prompt,
    "referenceImages": [hero_b64]  # Include hero in every call
}
```

**4. Dialogue not audible in Seedance/Veo**

MANDATORY dialogue gate: agent must confirm dialogue text separately before video generation.

**5. Image-ad template not matching reference**

Use `image-ad-clone` skill for precise reverse-engineering:
1. Agent analyzes reference with vision
2. Generates detailed prompt template
3. Tests against correct backend (chatgpt vs nano-banana)
4. User validates before saving to library

## File Organization

**Standard output structure:**

```
output/
├── videos/
│   ├── seedance/
│   ├── sora/
│   ├── veo/
│   └── kling/
├── images/
│   ├── nano-banana/
│   └── chatgpt/
├── campaigns/
│   └── {campaign_name}/
│       ├── stills/
│       ├── videos/
│       └── final/
└── references/
    ├── influencers/
    ├── products/
    └── aesthetics/
```

## Integration with Meta Ads

**Publish static image ad to Meta (paused):**

```python
import subprocess

def publish_to_meta(image_path, ad_name, campaign_id):
    """Publish image ad to Meta Marketing API"""
    
    script_path = "shared/skills/meta-ad-builder/scripts/create_ad.py"
    
    result = subprocess.run([
        "python3", script_path,
        "--image", image_path,
        "--name", ad_name,
        "--campaign", campaign_id,
        "--status", "PAUSED"
    ], capture_output=True, text=True)
    
    if result.returncode == 0:
        print(f"Ad published: {result.stdout}")
    else:
        print(f"Error: {result.stderr}")
```

## Best Practices

1. **Always use references for character consistency** - Generate hero shot first, then use as reference in all subsequent calls
2. **Confirm dialogue separately** - Never skip the dialogue confirmation gate for Seedance/Veo
3. **Choose the right model** - Seedance for flexibility, Veo for start-frame animation, Kling for b-roll
4. **Batch wisely** - Don't fire 50 parallel jobs; queue 5-10 at a time
5. **Save approved prompts** - Add working formulas to `MASTER_CONTEXT.md` for reuse
6. **Test templates before scaling** - Generate 1 proof-of-concept before batch production
7. **Use ChatGPT Image 2 for text-heavy** - It handles typography/UI better than Nano Banana
8. **Use Nano Banana for photoreal** - Better for humans, products, lifestyle shots
9. **Stitch in post** - Multi-beat narratives (Pixar, claymation) need ffmpeg stitching
10. **Version your character sheets** - Keep dated backups of influencer reference sets

## Advanced Patterns

### Batch generate 10 product angles

```python
def generate_product_turnaround(product_image_path):
    """Generate 360-degree product shots"""
    
    with open(product_image_path, "rb") as f:
        prod_b64 = base64.b64encode(f.read()).decode()
    
    angles = [0, 45, 90, 135, 180, 225, 270, 315, "top-down", "hero"]
    job_ids = []
    
    for angle in angles:
        if isinstance(angle, int):
            angle_desc = f"{angle} degree rotation"
        else:
            angle_desc = angle
        
        prompt = f"Product photography, {angle_desc}, clean white background, professional studio lighting"
        
        payload = {
            "prompt": prompt,
            "aspectRatio": "1:1",
            "referenceImages": [prod_b64]
        }
        
        resp = requests.post(
            f"{base_url}/v1/nano-banana-2",
            headers=headers,
            json=payload
        )
        
        job_ids.append((angle, resp.json()["jobId"]))
    
    return job_ids
```

### A/B test 5 thumbnail variations

```python
def generate_thumbnail_variants(base_prompt, face_refs):
    
