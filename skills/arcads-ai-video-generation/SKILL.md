---
name: arcads-ai-video-generation
description: Create AI marketing videos and static Meta image ads using Arcads API with Seedance 2.0, Sora 2, Veo 3.1, Kling, Nano Banana, and ChatGPT Image 2
triggers:
  - generate an AI video ad with Arcads
  - create UGC video using Seedance
  - make a static Meta image ad
  - generate AI influencer character sheet
  - create Pixar-style animated ad
  - animate product image to video with Veo
  - generate b-roll clip with Kling
  - make claymation ad with Arcads
---

# Arcads AI Video Generation

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

Arcads is a comprehensive AI video and image generation platform for marketing creative. This skill enables agents to orchestrate multi-step ad creation workflows using Seedance 2.0 (flagship video model), Sora 2, Veo 3.1, Kling 3.0, Grok Video, Nano Banana 2/Pro/Edit, ChatGPT Image 2, OmniHuman, and audio-driven models — plus a 37-template static Meta image-ad library and pipelines for Pixar-style and claymation animated ads.

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/krusemediallc/arcads-claude-code.git
cd arcads-claude-code
```

### 2. Run setup

```bash
./scripts/setup.sh
```

This will:
- Prompt for your Arcads API key (get it at [app.arcads.ai/settings/api](https://app.arcads.ai/settings/api))
- Save credentials to `.env` (gitignored)
- Verify API connection
- Create `MASTER_CONTEXT.md` workspace file

### 3. Prerequisites

| Tool | Required for | Install (macOS) |
|---|---|---|
| **Python 3.10+** | All workflows | `brew install python@3.12` |
| **`ffmpeg`** | Video stitching, chroma-key overlay | `brew install ffmpeg` |
| **`jq`** | JSON parsing in bash scripts | `brew install jq` |
| **Node.js** | Caption burn-in (hyperframes) | `brew install node` |
| **`whisper`** | Caption transcription | `pip install openai-whisper` |

Linux: `apt install ffmpeg jq nodejs python3`  
Windows: Use WSL2 (bash required)

### 4. Environment variables

Create `.env` in project root:

```bash
ARCADS_API_KEY=your_api_key_here
```

## API Overview

Base URL: `https://app.arcads.ai/api`

All requests require header:
```
Authorization: Bearer ${ARCADS_API_KEY}
```

### Common response pattern

```json
{
  "id": "gen_abc123",
  "status": "pending",
  "model": "seedance-2",
  "createdAt": "2026-07-08T12:00:00Z"
}
```

Poll with `GET /v1/videos/{id}` or `GET /v1/images/{id}` until `status: "completed"`.

## Core Workflows

### 1. Seedance 2.0 Video (Flagship Model)

**Best for:** UGC product reviews, premium reveals, product heroes, studio lookbooks, feature walkthroughs

#### UGC selfie-style product review

```python
import requests
import os
import time
import json

API_KEY = os.getenv('ARCADS_API_KEY')
BASE_URL = 'https://app.arcads.ai/api'

def create_seedance_ugc(prompt_text, duration=12):
    """
    Generate Seedance 2.0 UGC video using 9-layer formula
    """
    headers = {
        'Authorization': f'Bearer {API_KEY}',
        'Content-Type': 'application/json'
    }
    
    payload = {
        'model': 'seedance-2',
        'prompt': prompt_text,
        'duration': duration,
        'style': 'ugc',  # iPhone-shot aesthetic
        'aspectRatio': '9:16'
    }
    
    response = requests.post(
        f'{BASE_URL}/v1/videos/generate',
        headers=headers,
        json=payload
    )
    
    if response.status_code != 200:
        raise Exception(f"API error: {response.text}")
    
    result = response.json()
    video_id = result['id']
    
    # Poll for completion
    while True:
        status_resp = requests.get(
            f'{BASE_URL}/v1/videos/{video_id}',
            headers=headers
        )
        status = status_resp.json()
        
        if status['status'] == 'completed':
            return status['url']
        elif status['status'] == 'failed':
            raise Exception(f"Generation failed: {status.get('error')}")
        
        print(f"Status: {status['status']}... waiting 10s")
        time.sleep(10)

# Example usage
prompt = """
Woman in modern kitchen, natural lighting, holds product bottle.
Direct eye contact, casual smile. "I used to buy [competitor] until I found this."
Gestures with product. "The difference? Actually works." 
Shows product label to camera. iPhone-shot aesthetic, authentic delivery.
"""

video_url = create_seedance_ugc(prompt, duration=12)
print(f"Video ready: {video_url}")
```

#### Premium product reveal (no person)

```python
def create_premium_reveal(product_name, product_image_path=None):
    """
    Dark-void aesthetic with text narrative overlays
    """
    prompt = f"""
    {product_name} floating in dark void, dramatic spotlight.
    Text overlay: "Introducing {product_name}".
    Slow 360° rotation. Mist effect, subtle glow.
    Text: "Engineered for [key benefit]".
    Hero shot, product center frame. Premium minimal aesthetic.
    """
    
    payload = {
        'model': 'seedance-2',
        'prompt': prompt,
        'duration': 10,
        'style': 'premium-reveal',
        'aspectRatio': '1:1'
    }
    
    # Optional: add product reference image
    if product_image_path:
        import base64
        with open(product_image_path, 'rb') as f:
            img_b64 = base64.b64encode(f.read()).decode()
        payload['referenceImages'] = [img_b64]
    
    # Same polling pattern as above...
```

**Seedance 2.0 shot styles:** `ugc`, `premium-reveal`, `product-hero`, `studio-lookbook`, `feature-walkthrough`

See prompt library at `skills/arcads-external-api/prompting/prompt-library/seedance-2-*.md`

### 2. Image-to-Video (Veo 3.1 with Start Frame)

**Best for:** Animating UGC stills into videos with dialogue

```python
import base64

def animate_still_to_video(image_path, dialogue_script, duration=8):
    """
    Veo 3.1 with startFrame — animate a Nano Banana still
    """
    headers = {
        'Authorization': f'Bearer {API_KEY}',
        'Content-Type': 'application/json'
    }
    
    # Load image as base64
    with open(image_path, 'rb') as f:
        start_frame_b64 = base64.b64encode(f.read()).decode()
    
    payload = {
        'model': 'veo-3.1',
        'startFrame': start_frame_b64,
        'prompt': f"""
        Natural human motion from starting frame.
        Person speaks: "{dialogue_script}"
        Lip-sync to dialogue, subtle head movements, authentic delivery.
        """,
        'duration': duration,
        'aspectRatio': '9:16'
    }
    
    response = requests.post(
        f'{BASE_URL}/v1/veo/animate',
        headers=headers,
        json=payload
    )
    
    result = response.json()
    
    # Poll until complete (same pattern)
    # ...
    return result['id']
```

**MANDATORY dialogue gate:** Veo 3.1 with dialogue requires separate user confirmation before generation starts.

### 3. AI Influencer Character Sheet (Nano Banana 2)

**Best for:** Creating reusable AI person libraries

```python
def create_ai_influencer(description, name):
    """
    Two-pass workflow:
    1. Generate hero portrait
    2. Generate 9 additional angles with hero as reference
    """
    headers = {
        'Authorization': f'Bearer {API_KEY}',
        'Content-Type': 'application/json'
    }
    
    # Phase 1: Hero portrait
    hero_prompt = f"""
    Front-facing portrait. {description}
    Natural lighting, eye contact, slight smile.
    Photoreal skin texture, authentic appearance.
    High detail, commercial photography quality.
    """
    
    hero_payload = {
        'model': 'nano-banana-2',
        'prompt': hero_prompt,
        'aspectRatio': '1:1'
    }
    
    hero_resp = requests.post(
        f'{BASE_URL}/v1/images/generate',
        headers=headers,
        json=hero_payload
    )
    
    hero_result = hero_resp.json()
    hero_id = hero_result['id']
    
    # Poll for hero completion
    while True:
        status_resp = requests.get(
            f'{BASE_URL}/v1/images/{hero_id}',
            headers=headers
        )
        status = status_resp.json()
        
        if status['status'] == 'completed':
            hero_url = status['url']
            hero_b64 = status['imageBase64']
            break
        
        time.sleep(5)
    
    print(f"Hero portrait: {hero_url}")
    print("User approval required before generating additional angles...")
    
    # Phase 2: Generate 9 additional angles
    angles = [
        "3/4 view left, slight smile",
        "3/4 view right, thoughtful expression",
        "Profile left side, natural lighting",
        "Profile right side, soft focus background",
        "Close-up face, direct eye contact",
        "Mid-distance shot, arms crossed",
        "Looking away, candid moment",
        "Laughing naturally, genuine expression",
        "Serious expression, professional demeanor"
    ]
    
    character_sheet = {'hero': hero_url, 'angles': {}}
    
    for i, angle_desc in enumerate(angles):
        angle_prompt = f"{description}. {angle_desc}. Match likeness and style from reference."
        
        angle_payload = {
            'model': 'nano-banana-2',
            'prompt': angle_prompt,
            'referenceImages': [hero_b64],  # Use hero as reference
            'aspectRatio': '1:1'
        }
        
        angle_resp = requests.post(
            f'{BASE_URL}/v1/images/generate',
            headers=headers,
            json=angle_payload
        )
        
        # Poll and collect
        # ...
        character_sheet['angles'][f'angle_{i+1}'] = angle_url
    
    # Save to references/influencers/
    output_dir = f'references/influencers/{name}'
    os.makedirs(output_dir, exist_ok=True)
    
    with open(f'{output_dir}/manifest.json', 'w') as f:
        json.dump(character_sheet, f, indent=2)
    
    return character_sheet
```

### 4. UGC Product Selfie Still

**Best for:** Authentic-looking iPhone selfie frames with AI influencer + product

```python
def generate_ugc_selfie(influencer_name, product_image_path, scene_description):
    """
    Combine character reference + product photo + UGC aesthetic references
    """
    # Load influencer hero
    with open(f'references/influencers/{influencer_name}/manifest.json') as f:
        influencer_data = json.load(f)
    
    hero_path = influencer_data['hero']
    
    # Load product image
    with open(product_image_path, 'rb') as f:
        product_b64 = base64.b64encode(f.read()).decode()
    
    # Load UGC aesthetic references
    ugc_ref_dir = 'references/aesthetics/ugc-selfie/'
    style_refs = []
    for ref_file in os.listdir(ugc_ref_dir):
        if ref_file.endswith(('.jpg', '.png')):
            with open(os.path.join(ugc_ref_dir, ref_file), 'rb') as f:
                style_refs.append(base64.b64encode(f.read()).decode())
    
    prompt = f"""
    iPhone selfie frame grab. {scene_description}
    Person holds product naturally, casual selfie angle.
    Soft skin realism, natural blemishes, authentic lighting.
    Slight camera grain, real iPhone aesthetic.
    Product visible but organic to scene.
    """
    
    payload = {
        'model': 'nano-banana-2',
        'prompt': prompt,
        'referenceImages': style_refs[:3] + [product_b64],  # Max 5 total
        'aspectRatio': '9:16'
    }
    
    # Generate and poll...
```

**Nano Banana model variants:**
- `nano-banana-2` — default (Gemini 3 Flash Image)
- `nano-banana` — **Nano Banana Pro** (Gemini 3 Pro Image, tighter identity lock)
- `nano-banana-edit` — inpainting/editing

### 5. Sora 2 Text-to-Video (Long-form)

**Best for:** Up to 20-second narrative videos

```python
def generate_sora_video(scene_description, duration=16):
    """
    Sora 2 for longer-duration text-to-video
    Auto-calculate duration from script word count (~2.5 words/sec)
    """
    word_count = len(scene_description.split())
    estimated_duration = min(20, max(5, word_count / 2.5))
    
    payload = {
        'model': 'sora-2',
        'prompt': scene_description,
        'duration': int(estimated_duration),
        'aspectRatio': '16:9'
    }
    
    headers = {
        'Authorization': f'Bearer {API_KEY}',
        'Content-Type': 'application/json'
    }
    
    response = requests.post(
        f'{BASE_URL}/v1/sora2/generate',
        headers=headers,
        json=payload
    )
    
    # Poll pattern...
```

**Sora 2 remix:** Use `POST /v1/sora2/remix/video` with existing video ID to remix an asset.

### 6. B-roll & Scene Generation (Kling 3.0)

```python
def generate_broll(scene_description, duration=5):
    """
    Kling 3.0 b-roll clips
    """
    payload = {
        'model': 'kling-3.0',
        'prompt': scene_description,
        'duration': duration,
        'aspectRatio': '16:9'
    }
    
    response = requests.post(
        f'{BASE_URL}/v1/b-roll',
        headers={'Authorization': f'Bearer {API_KEY}'},
        json=payload
    )
    
    # Poll...

def generate_scene(environment_description):
    """
    Kling 3.0 environmental scenes
    """
    payload = {
        'model': 'kling-3.0',
        'prompt': environment_description,
        'duration': 8
    }
    
    response = requests.post(
        f'{BASE_URL}/v1/scene',
        headers={'Authorization': f'Bearer {API_KEY}'},
        json=payload
    )
    
    # Poll...
```

### 7. Static Meta Image Ads (37-Template Library)

**Three generation skills:**
1. **`chatgpt-image-ad`** — Typography-heavy, UI mimicry (gpt-image-2)
2. **`nano-banana-image-ad`** — Photoreal, lifestyle, multi-reference (Nano Banana 2/Pro)
3. **`image-ad-clone`** — Reverse-engineer existing ads into templates

#### Generate Apple Notes-style ad

```python
def generate_apple_notes_ad(product_name, benefit_list):
    """
    ChatGPT Image 2 backend (typography-focused)
    """
    prompt = f"""
    iPhone Notes app screenshot.
    Title: "{product_name} — why I switched"
    Bulleted list:
    {chr(10).join(f'• {b}' for b in benefit_list)}
    
    iOS system font, cream background, realistic UI elements.
    Subtle screen glare, authentic iPhone screenshot aesthetic.
    """
    
    payload = {
        'model': 'gpt-image-2',
        'prompt': prompt,
        'aspectRatio': '1:1',
        'templateId': 'apple-notes-list'  # From library
    }
    
    response = requests.post(
        f'{BASE_URL}/v1/images/generate',
        headers={'Authorization': f'Bearer {API_KEY}'},
        json=payload
    )
    
    # Poll...
```

#### Generate Forbes editorial ad

```python
def generate_forbes_editorial(product_name, headline):
    """
    Nano Banana 2 backend (photoreal lifestyle)
    """
    prompt = f"""
    Forbes magazine editorial layout.
    Hero image: {product_name} in premium setting, editorial photography.
    Headline: "{headline}"
    Forbes logo top, article preview text, professional layout.
    High-end commercial photography aesthetic.
    """
    
    payload = {
        'model': 'nano-banana-2',
        'prompt': prompt,
        'aspectRatio': '4:5',
        'templateId': 'forbes-editorial'
    }
    
    # Same pattern...
```

**37 template IDs:** `apple-notes-list`, `forbes-editorial`, `google-search-fake`, `comparison-table`, `sticky-note-flatlay`, `slack-thread-fake`, `chatgpt-conversation`, `imessage-screenshot`, `magazine-cover`, `billboard-outdoor`, `museum-exhibit`, `weather-forecast-ui`, `scratch-off-ticket`, `founder-letter`, `dating-app-card`, and more.

See `shared/skills/image-ad-prompting/OVERVIEW.md` for:
- Backend decision tree (which model for which template)
- Aspect-ratio compatibility matrix
- Standard generate/clone workflows

### 8. Clone Existing Ad as Template

```python
def clone_image_ad(reference_image_path, backend='auto'):
    """
    Reverse-engineer existing ad into reusable template
    Phase 1: Detect structure → Phase 8: Cross-validate
    """
    with open(reference_image_path, 'rb') as f:
        ref_b64 = base64.b64encode(f.read()).decode()
    
    # Phase 1: Structure detection
    analyze_payload = {
        'imageBase64': ref_b64,
        'backend': backend  # 'gpt-image-2', 'nano-banana-2', or 'auto'
    }
    
    analyze_resp = requests.post(
        f'{BASE_URL}/v1/image-ad/clone/analyze',
        headers={'Authorization': f'Bearer {API_KEY}'},
        json=analyze_payload
    )
    
    structure = analyze_resp.json()
    
    # Agent extracts: layout type, color scheme, typography, composition
    template_id = structure['suggestedTemplateId']
    
    # Phase 8: Cross-validate against other backend
    # ...
    
    return template_id
```

## Multi-Step Animated Ad Pipelines

### 9. Pixar-Style 3D Animated Ad

```bash
#!/bin/bash
# shared/skills/pixar-style-ad/scripts/generate.sh

set -e

PRODUCT=$1
STORY_ARC=$2  # 8-beat JSON file

# Step 1: Lock cast sheet (ChatGPT Image 2 character designs)
echo "Generating Pixar character designs..."
python3 shared/skills/pixar-style-ad/scripts/lock_cast.py "$PRODUCT"

# Step 2: Generate storyboard stills (sequential, prior frame as ref)
echo "Creating storyboard..."
python3 shared/skills/pixar-style-ad/scripts/storyboard.py "$STORY_ARC"

# Step 3: Seedance 2.0 image-to-video per beat
echo "Animating beats..."
for beat in $(seq 1 8); do
    python3 shared/skills/pixar-style-ad/scripts/animate_beat.py "$beat"
done

# Step 4: Stitch with ffmpeg + burn captions
echo "Stitching final video..."
ffmpeg -f concat -safe 0 -i beats_concat.txt -c copy pixar_ad_raw.mp4

# Step 5: Captions (hyperframes + whisper)
npx hyperframes --input pixar_ad_raw.mp4 --output pixar_ad_final.mp4 \
    --transcript "$(python3 -c "import whisper; print(whisper.load_model('medium.en').transcribe('pixar_ad_raw.mp4')['text'])")"

echo "Done: pixar_ad_final.mp4"
```

**Key dependencies:** `ffmpeg`, `jq`, `npx hyperframes`, `whisper`

See `shared/skills/pixar-style-ad/prompting/guide.md` for 8-beat story arc structure.

### 10. Claymation / Aardman-Style Ad

```bash
#!/bin/bash
# shared/skills/claymation-ad/scripts/generate.sh

set -e

PRODUCT=$1
NARRATOR_SCRIPT=$2

# Same backbone as Pixar with clay textures
# ChatGPT Image 2 storyboard → Seedance 2.0 i2v

# Step 4: Stitch with stop-motion judder
ffmpeg -f concat -safe 0 -i beats_concat.txt \
    -vf "fps=12,fps=24" \  # Stop-motion judder effect
    -c:v libx264 claymation_ad.mp4

# Mix external voiceover (ElevenLabs, etc.)
ffmpeg -i claymation_ad.mp4 -i narrator_vo.mp3 \
    -c:v copy -c:a aac -map 0:v:0 -map 1:a:0 \
    claymation_ad_final.mp4
```

See `shared/skills/claymation-ad/prompting/guide.md`.

### 11. Burn Captions onto Finished Video

```python
import subprocess
import json

def burn_captions(video_path, output_path):
    """
    Out-of-band post-step: Whisper transcription → HyperFrames render
    Works on any source (Pixar, UGC, B-roll, etc.)
    """
    # Step 1: Transcribe with Whisper
    import whisper
    model = whisper.load_model("medium.en")
    result = model.transcribe(video_path)
    
    # Step 2: Group word-level into reading phrases
    transcript_json = json.dumps(result['segments'])
    
    # Step 3: HyperFrames render
    subprocess.run([
        'npx', 'hyperframes',
        '--input', video_path,
        '--output', output_path,
        '--transcript', transcript_json,
        '--style', 'modern-bold'
    ], check=True)
    
    print(f"Captions burned: {output_path}")
```

## Configuration

### Workspace files

- **`MASTER_CONTEXT.md`** — Personal workspace file (product info, brand voice, active campaigns)
- **`references/influencers/`** — AI character libraries (10-image sets per person)
- **`references/aesthetics/`** — Style reference images (UGC selfie, premium, etc.)
- **`references/products/`** — Product photography for multi-step workflows

### Cost controls

The agent confirms generation costs before API calls:

```
💰 Cost estimate: $0.45 (Seedance 2.0, 12s, 9:16)
Proceed? (y/n):
```

Typical costs (as of 2026):
- Seedance 2.0: $0.03–0.05/sec
- Veo 3.1: $0.08/sec
- Nano Banana 2: $0.02/image
- ChatGPT Image 2: $0.04/image

## Common Patterns

### Pattern 1: UGC Still → Video Pipeline

```python
# 1. Generate UGC selfie still (Nano Banana 2)
still_url = generate_ugc_selfie('Sofia', 'product.jpg', 'bedroom, golden hour')

# 2. User approves still

# 3. Animate with Veo 3.1 (start frame + dialogue)
video_url = animate_still_to_video(still_url, "This changed my morning routine", duration=8)
```

### Pattern 2: Multi-Reference Character Lock

```python
# Use 5 references (max) for tight identity lock
character_refs = [
    'references/influencers/Alex/hero.jpg',
    'references/influencers/Alex/angle_1.jpg',
    'references/influencers/Alex/angle_2.jpg',
    'references/aesthetics/lighting/golden-hour-1.jpg',
    'product.jpg'
]

ref_b64_list = []
for ref in character_refs:
    with open(ref, 'rb') as f:
        ref_b64_list.append(base64.b64encode(f.read()).decode())

payload = {
    'model': 'nano-banana',  # Pro for tighter lock
    'prompt': prompt,
    'referenceImages': ref_b64_list
}
```

### Pattern 3: Batch Parallel Firing

```python
import concurrent.futures

def batch_generate_variations(base_prompt, variations):
    """
    Fire 6 parallel requests (YouTube thumbnail variations, A/B test stills, etc.)
    """
    def generate_single(variation_text):
        payload = {
            'model': 'nano-banana-2',
            'prompt': f"{base_prompt} {variation_text}",
            'aspectRatio': '16:9'
        }
        response = requests.post(
            f'{BASE_URL}/v1/images/generate',
            headers={'Authorization': f'Bearer {API_KEY}'},
            json=payload
        )
        return response.json()['id']
    
    with concurrent.futures.ThreadPoolExecutor(max_workers=6) as executor:
        futures = {executor.submit(generate_single, v): v for v in variations}
        
        results = {}
        for future in concurrent.futures.as_completed(futures):
            variation = futures[future]
            gen_id = future.result()
            results[variation] = gen_id
    
    return results
```

## Troubleshooting

### Issue: "Authorization failed"

**Cause:** Invalid or missing API key

**Fix:**
```bash
# Verify .env exists and has correct key
cat .env | grep ARCADS_API_KEY

# Re-run setup if needed
./scripts/setup.sh
```

### Issue: Generation stuck in "pending" status

**Cause:** Model overloaded or request timeout

**Fix:**
- Poll interval: Wait 10–15 seconds between status checks (not 5s)
- Check status manually: `GET /v1/videos/{id}` or `/v1/images/{id}`
- Timeout: Set max polling duration (5 minutes for video, 2 minutes for image)

```python
import time

def poll_with_timeout(endpoint, timeout=300):
    start = time.time()
    while time.time() - start < timeout:
        status = requests.get(endpoint, headers={'Authorization': f'Bearer {API_KEY}'}).json()
        if status['status'] in ['completed', 'failed']:
            return status
        time.sleep(10)
    raise TimeoutError(f"Generation timeout after {timeout}s")
```

### Issue: "Reference image limit exceeded"

**Cause:** More than 5 `referenceImages` in payload

**Fix:**
```python
# Max 5 references per request
payload['referenceImages'] = ref_list[:5]
```

### Issue: Veo 3.1 dialogue not matching script

**Cause:** Skipped MANDATORY dialogue confirmation gate

**Fix:**
```python
# Before Veo 3.1 generation with dialogue:
print(f"Dialogue script: '{dialogue_text}'")
confirm = input("Confirm dialogue before generating? (y/n): ")
if confirm.lower() != 'y':
    raise Exception("Dialogue confirmation required")
```

### Issue: Pixar/claymation pipeline fails at stitch step

**Cause:** Missing `ffmpeg` or malformed concat file

**Fix:**
```bash
# Verify ffmpeg installed
which ffmpeg

# Check concat file format
cat beats_concat.txt
# Should be:
# file 'beat_1.mp4'
# file 'beat_2.mp4'
# ...

# Manually test stitch
ffmpeg -f concat -safe 0 -i beats_concat.txt -c copy test_output.mp4
```

### Issue: Caption burn-in fails

**Cause:** Missing `npx hyperframes` or `whisper`

**Fix:**
```bash
# Install Node.js (includes npx)
brew install node

# Install Whisper
pip3 install openai-whisper

# Test manually
npx hyperframes --version
python3 -c "import whisper; print(whisper.__version__)"
```

### Issue: Image ad template not rendering correctly

**Cause:** Wrong backend for template type

**Fix:** Check `shared/skills/image-ad-prompting/OVERVIEW.md` decision tree:

- Typography-heavy (Notes, ChatGPT conversation, Slack) → `gpt-image-2`
- Photoreal lifestyle (Forbes, magazine, billboard) → `nano-banana-2`
- UI mockups with text → `gpt-image-2`

```python
# Use correct model per template
if template_id in ['apple-notes-list', 'chatgpt-conversation', 'slack-thread-fake']:
    model = 'gpt-image-2'
else:
    model = 'nano-banana-2'
```

## Additional Resources

- **Prompt library:** `skills/arcads-external-api/prompting/prompt-library/`
- **Image ad templates:** `shared/skills/image-ad-prompting/OVERVIEW.md`
- **Video walkthrough:** [YouTube tutorial by Mr. Paid Social](https://youtu.be/HHGQN9Zqaxo)
- **API docs:** [app.arcads.ai/docs/api](https://app.arcads.ai/docs/api)
- **Community:** [The AI Ad Alchemists Skool](https://skool.
