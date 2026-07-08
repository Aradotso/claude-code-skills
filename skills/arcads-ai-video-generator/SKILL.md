---
name: arcads-ai-video-generator
description: Create AI marketing videos and static Meta image ads using the Arcads API with Seedance 2.0, Sora 2, Veo 3.1, Kling, Nano Banana, ChatGPT Image, and 37 static ad templates
triggers:
  - generate an AI video ad
  - create a UGC video with Seedance
  - make a product reveal video
  - generate AI influencer images
  - create static Meta image ads
  - build a Pixar-style animated ad
  - animate a still image with Veo
  - generate YouTube thumbnails
---

# Arcads AI Video Generator

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

Create AI marketing videos and images using the Arcads API. Supports the full creative stack: **Seedance 2.0** (flagship video), **Sora 2**, **Veo 3.1**, **Kling 3.0**, **Grok Video**, **Nano Banana 2/Pro/Edit**, **ChatGPT Image 2**, **OmniHuman**, **Audio-driven** models, plus a 37-template static Meta image-ad library and multi-step pipelines for Pixar-style and claymation animated ads.

## Installation

```bash
git clone https://github.com/krusemediallc/arcads-claude-code.git
cd arcads-claude-code
./scripts/setup.sh
```

The setup script will:
- Prompt for your Arcads API key (get it at [app.arcads.ai/settings/api](https://app.arcads.ai/settings/api))
- Save credentials to `.env`
- Verify API connection
- Create `MASTER_CONTEXT.md` workspace file

### Prerequisites

| Tool | Required for | Install |
|---|---|---|
| Python 3.10+ | Core API operations | `brew install python@3.12` |
| `ffmpeg` | Video stitching, chroma-key | `brew install ffmpeg` |
| `jq` | JSON parsing in bash scripts | `brew install jq` |
| Node.js | Caption burn-in (hyperframes) | `brew install node` |
| `whisper` | Caption transcription | `pip install openai-whisper` |

### Environment Setup

Create `.env` in the project root:

```bash
ARCADS_API_KEY=your_api_key_here
```

## Core API Patterns

### Video Generation (Seedance 2.0)

**Endpoint:** `POST https://api.arcads.ai/v1/seedance2/video`

```python
import requests
import os
import time

API_KEY = os.getenv('ARCADS_API_KEY')
BASE_URL = 'https://api.arcads.ai'

def generate_seedance_video(prompt, duration=12):
    """Generate a Seedance 2.0 video."""
    headers = {
        'Authorization': f'Bearer {API_KEY}',
        'Content-Type': 'application/json'
    }
    
    payload = {
        'prompt': prompt,
        'duration': duration,
        'shotStyle': 'ugc-selfie'  # or 'premium-reveal', 'product-hero', etc.
    }
    
    # Start generation
    response = requests.post(
        f'{BASE_URL}/v1/seedance2/video',
        headers=headers,
        json=payload
    )
    
    job = response.json()
    job_id = job['id']
    
    # Poll until complete
    while True:
        status_response = requests.get(
            f'{BASE_URL}/v1/jobs/{job_id}',
            headers=headers
        )
        status = status_response.json()
        
        if status['status'] == 'completed':
            return status['videoUrl']
        elif status['status'] == 'failed':
            raise Exception(f"Generation failed: {status.get('error')}")
        
        time.sleep(5)

# Example: UGC product review
video_url = generate_seedance_video(
    prompt="""
    [SCENE: NATURAL KITCHEN LIGHTING]
    Medium shot, slight handheld motion.
    Woman, early 30s, messy bun, casual tank top.
    Holding [PRODUCT] at chest level, genuine smile.
    
    [DIALOGUE]
    "Okay so I used to buy [COMPETITOR] every month but then I found THIS—"
    *lifts product slightly, direct eye contact*
    "—and honestly? I'm never going back."
    
    [VISUAL BEATS]
    0-3s: Establishing shot, product reveal
    3-7s: Close on face during key claim
    7-12s: Pull back, casual product demo gesture
    """,
    duration=12
)
print(f"Video ready: {video_url}")
```

### Seedance 2.0 Prompt Formulas

Five battle-tested formulas ship with the skill:

#### 1. UGC Selfie-Style Review
```
[SCENE: NATURAL KITCHEN LIGHTING]
Medium shot, slight handheld motion.
Woman, early 30s, messy bun, casual tank top.
Holding [PRODUCT] at chest level, genuine smile.

[DIALOGUE]
"Okay so I used to buy [COMPETITOR] every month..."

[VISUAL BEATS]
0-3s: Establishing shot
3-7s: Close on face
7-12s: Product demo gesture
```

#### 2. Premium Product Reveal (No Person)
```
[SCENE: DARK VOID]
No human. Product only.
Black background, dramatic side lighting.

[TEXT OVERLAYS]
Beat 1 (0-3s): "Most [CATEGORY] products..."
Beat 2 (3-6s): "[PRODUCT] is different."
Beat 3 (6-9s): Hero rotation + key feature callout

[CAMERA]
Slow push-in, 360° rotation at beat 3
```

#### 3. Product Hero with Elements
```
[SCENE: ELEMENTAL SHOWCASE]
Product suspended mid-frame.
Water splash from below, mist rising.
Slow rotation (15°/sec).

[VISUAL FX]
0-2s: Water splash entry
2-5s: Mist buildup
5-8s: Light rays piercing mist
8-12s: Hero rotation + feature close-up
```

### Image Generation (Nano Banana)

**Endpoint:** `POST https://api.arcads.ai/v1/nano-banana/image`

```python
import base64

def generate_nano_banana_image(prompt, reference_image_path=None, model='nano-banana-2'):
    """Generate a Nano Banana image with optional reference."""
    headers = {
        'Authorization': f'Bearer {API_KEY}',
        'Content-Type': 'application/json'
    }
    
    payload = {
        'prompt': prompt,
        'model': model,  # 'nano-banana-2' or 'nano-banana' (Pro) or 'nano-banana-edit'
        'aspectRatio': '9:16'
    }
    
    # Add reference image if provided
    if reference_image_path:
        with open(reference_image_path, 'rb') as f:
            img_base64 = base64.b64encode(f.read()).decode('utf-8')
        payload['refImageAsBase64'] = img_base64
    
    response = requests.post(
        f'{BASE_URL}/v1/nano-banana/image',
        headers=headers,
        json=payload
    )
    
    job = response.json()
    job_id = job['id']
    
    # Poll for completion
    while True:
        status_response = requests.get(
            f'{BASE_URL}/v1/jobs/{job_id}',
            headers=headers
        )
        status = status_response.json()
        
        if status['status'] == 'completed':
            return status['imageUrl']
        elif status['status'] == 'failed':
            raise Exception(f"Generation failed: {status.get('error')}")
        
        time.sleep(3)

# Example: UGC selfie with product
image_url = generate_nano_banana_image(
    prompt="""
    iPhone selfie camera, natural bedroom lighting.
    Woman, 22, freckles, golden hour glow.
    Holding [PRODUCT] at chest level, genuine smile.
    Messy hair, no makeup, casual tank top.
    Slight camera shake, realistic skin texture.
    Background: unmade bed, string lights, plants.
    """,
    reference_image_path='references/influencers/sofia_hero.jpg'
)
```

### Veo 3.1 (Start-Frame Animation)

**Endpoint:** `POST https://api.arcads.ai/v1/veo3/video`

```python
def animate_with_veo(start_frame_path, prompt, duration=8):
    """Animate a still image with Veo 3.1."""
    headers = {
        'Authorization': f'Bearer {API_KEY}',
        'Content-Type': 'application/json'
    }
    
    # Load start frame
    with open(start_frame_path, 'rb') as f:
        img_base64 = base64.b64encode(f.read()).decode('utf-8')
    
    payload = {
        'prompt': prompt,
        'startFrame': img_base64,
        'duration': duration,
        'dialogue': 'I switched to this product and never looked back'
    }
    
    response = requests.post(
        f'{BASE_URL}/v1/veo3/video',
        headers=headers,
        json=payload
    )
    
    job = response.json()
    return poll_job(job['id'])

def poll_job(job_id):
    """Generic job polling."""
    headers = {'Authorization': f'Bearer {API_KEY}'}
    
    while True:
        response = requests.get(
            f'{BASE_URL}/v1/jobs/{job_id}',
            headers=headers
        )
        status = response.json()
        
        if status['status'] == 'completed':
            return status.get('videoUrl') or status.get('imageUrl')
        elif status['status'] == 'failed':
            raise Exception(f"Job failed: {status.get('error')}")
        
        time.sleep(5)
```

### Sora 2 Video Generation

**Endpoint:** `POST https://api.arcads.ai/v1/sora2/video`

```python
def generate_sora_video(prompt, duration=16, reference_image_path=None):
    """Generate a Sora 2 video (up to 20s)."""
    headers = {
        'Authorization': f'Bearer {API_KEY}',
        'Content-Type': 'application/json'
    }
    
    payload = {
        'prompt': prompt,
        'duration': duration
    }
    
    if reference_image_path:
        with open(reference_image_path, 'rb') as f:
            img_base64 = base64.b64encode(f.read()).decode('utf-8')
        payload['styleReference'] = img_base64
    
    response = requests.post(
        f'{BASE_URL}/v1/sora2/video',
        headers=headers,
        json=payload
    )
    
    return poll_job(response.json()['id'])

# Example: Longer narrative scene
video_url = generate_sora_video(
    prompt="""
    Cozy coffee shop interior, warm afternoon light.
    Camera tracks across table as woman opens laptop.
    Smooth dolly shot, shallow depth of field.
    Steam rising from coffee mug in foreground.
    Natural color grading, 24fps cinematic feel.
    """,
    duration=16
)
```

### B-Roll / Scene Generation (Kling 3.0)

**Endpoint:** `POST https://api.arcads.ai/v1/b-roll` or `POST https://api.arcads.ai/v1/scene`

```python
def generate_broll(prompt, duration=5):
    """Generate b-roll footage with Kling 3.0."""
    headers = {
        'Authorization': f'Bearer {API_KEY}',
        'Content-Type': 'application/json'
    }
    
    payload = {
        'prompt': prompt,
        'duration': duration
    }
    
    response = requests.post(
        f'{BASE_URL}/v1/b-roll',
        headers=headers,
        json=payload
    )
    
    return poll_job(response.json()['id'])

# Example: Product environment b-roll
broll_url = generate_broll(
    prompt="""
    Close-up of coffee beans being poured into grinder.
    Slow motion, 120fps.
    Natural light from window, wooden countertop.
    Shallow focus on falling beans.
    """
)
```

## Static Meta Image Ad Library

The repo includes **37 validated prompt templates** for static Meta image ads. Two primary generators:

### ChatGPT Image 2 (Typography/UI-Heavy)

**Best for:** Apple Notes lists, editorial layouts, comparison tables, UI mockups, text-heavy designs

```python
def generate_chatgpt_image_ad(template_name, product_details):
    """Generate static ad using ChatGPT Image 2."""
    # Load template from library
    template_path = f'shared/skills/image-ad-prompting/library/{template_name}.md'
    
    with open(template_path, 'r') as f:
        template = f.read()
    
    # Fill template with product details
    prompt = template.format(**product_details)
    
    headers = {
        'Authorization': f'Bearer {API_KEY}',
        'Content-Type': 'application/json'
    }
    
    payload = {
        'prompt': prompt,
        'model': 'gpt-image-2',
        'aspectRatio': '4:5'  # Standard Meta image ad ratio
    }
    
    response = requests.post(
        f'{BASE_URL}/v1/chatgpt-image/generate',
        headers=headers,
        json=payload
    )
    
    return poll_job(response.json()['id'])

# Example: Apple Notes style ad
image_url = generate_chatgpt_image_ad(
    template_name='apple-notes-list',
    product_details={
        'product_name': 'FocusFlow',
        'category': 'productivity app',
        'benefits': [
            'Blocks distractions automatically',
            'AI suggests optimal focus times',
            'Syncs across all devices'
        ],
        'cta': 'Download free today'
    }
)
```

### Nano Banana (Photoreal/Lifestyle)

**Best for:** Lifestyle product shots, before/after, founder letters with portraits, real-world settings

```python
def generate_nano_banana_ad(template_name, product_details, reference_images=None):
    """Generate static ad using Nano Banana."""
    template_path = f'shared/skills/image-ad-prompting/library/{template_name}.md'
    
    with open(template_path, 'r') as f:
        template = f.read()
    
    prompt = template.format(**product_details)
    
    headers = {
        'Authorization': f'Bearer {API_KEY}',
        'Content-Type': 'application/json'
    }
    
    payload = {
        'prompt': prompt,
        'model': 'nano-banana-2',
        'aspectRatio': '4:5'
    }
    
    # Add reference images if provided
    if reference_images:
        ref_images_b64 = []
        for img_path in reference_images:
            with open(img_path, 'rb') as f:
                ref_images_b64.append(base64.b64encode(f.read()).decode('utf-8'))
        payload['referenceImages'] = ref_images_b64
    
    response = requests.post(
        f'{BASE_URL}/v1/nano-banana/image',
        headers=headers,
        json=payload
    )
    
    return poll_job(response.json()['id'])
```

### Available Templates (37 total)

Key templates in the library:
- `apple-notes-list` — Simple checklist aesthetic
- `forbes-editorial` — Magazine hero layout
- `comparison-table` — Side-by-side feature grid
- `fake-google-search` — Search results UI mockup
- `sticky-note-flatlay` — Scattered notes on desk
- `imessage-screenshot` — Text conversation mockup
- `chatgpt-conversation` — AI chat interface
- `weather-forecast-ui` — Weather app parody
- `dating-app-card` — Swipe-style card layout
- `billboard-mockup` — Outdoor advertising placement
- `museum-exhibit` — Art gallery presentation
- `scratch-off-ticket` — Lottery/game aesthetic

See `shared/skills/image-ad-prompting/OVERVIEW.md` for full template list and compatibility matrix.

## Multi-Step Pipelines

### Pixar-Style 3D Animated Ad

Creates an 8-beat story arc with anthropomorphized characters, stitched into a final video with captions.

**Workflow:**
1. Define character sheet (mascot design)
2. Generate storyboard stills (ChatGPT Image 2)
3. Animate each beat (Seedance 2.0 i2v)
4. Stitch segments (ffmpeg)
5. Burn captions (HyperFrames)

```bash
# Run the full pipeline
./shared/skills/pixar-style-ad/pipeline.sh \
  --product "EcoBottle" \
  --duration 60 \
  --output output/pixar-ad.mp4
```

**Python API approach:**

```python
def create_pixar_ad(product_name, story_beats):
    """Generate a Pixar-style animated ad."""
    # Step 1: Generate character design
    character_prompt = f"""
    3D Pixar style mascot for {product_name}.
    Anthropomorphized product with expressive face.
    Big eyes, friendly smile, rounded shapes.
    Soft lighting, vibrant colors, clean geometry.
    T-pose reference, white background.
    """
    
    character_img = generate_chatgpt_image(character_prompt)
    
    # Step 2: Generate storyboard frames
    storyboard_frames = []
    prev_frame = character_img
    
    for beat in story_beats:
        frame_prompt = f"""
        {beat['scene_description']}
        Character from reference image.
        Pixar 3D render style, consistent lighting.
        """
        
        frame_url = generate_chatgpt_image(
            prompt=frame_prompt,
            reference_images=[prev_frame]
        )
        storyboard_frames.append(frame_url)
        prev_frame = frame_url
    
    # Step 3: Animate each frame with Seedance 2.0
    video_segments = []
    for i, frame in enumerate(storyboard_frames):
        video_url = animate_with_seedance_i2v(
            start_frame=frame,
            duration=story_beats[i]['duration'],
            motion_prompt=story_beats[i]['motion']
        )
        video_segments.append(video_url)
    
    # Step 4: Stitch (requires ffmpeg)
    final_video = stitch_videos(video_segments)
    
    return final_video

def animate_with_seedance_i2v(start_frame, duration, motion_prompt):
    """Animate a still frame with Seedance 2.0 image-to-video."""
    headers = {
        'Authorization': f'Bearer {API_KEY}',
        'Content-Type': 'application/json'
    }
    
    # Download and encode start frame
    import requests as req
    img_data = req.get(start_frame).content
    img_base64 = base64.b64encode(img_data).decode('utf-8')
    
    payload = {
        'startFrame': img_base64,
        'motionPrompt': motion_prompt,
        'duration': duration
    }
    
    response = requests.post(
        f'{BASE_URL}/v1/seedance2/video',
        headers=headers,
        json=payload
    )
    
    return poll_job(response.json()['id'])

# Example usage
pixar_ad = create_pixar_ad(
    product_name='EcoBottle',
    story_beats=[
        {
            'scene_description': 'Mascot looking sad at pile of plastic waste',
            'duration': 6,
            'motion': 'Subtle head turn, eyes downcast'
        },
        {
            'scene_description': 'Mascot has idea, lightbulb appears',
            'duration': 4,
            'motion': 'Excited jump, arms raised'
        },
        {
            'scene_description': 'Mascot presents EcoBottle to camera',
            'duration': 8,
            'motion': 'Slow rotation of bottle, proud smile'
        }
        # ... 5 more beats
    ]
)
```

### AI Influencer Creation (10-Image Character Sheet)

Generates a consistent AI influencer with multiple angles for reuse.

```python
def create_ai_influencer(description):
    """Create a 10-image character sheet for an AI influencer."""
    # Step 1: Generate hero front portrait
    hero_prompt = f"""
    Front-facing portrait, iPhone selfie camera.
    {description}
    Natural lighting, genuine smile.
    9:16 aspect ratio, eye-level camera.
    Realistic skin texture, no AI smoothing.
    """
    
    hero_url = generate_nano_banana_image(
        prompt=hero_prompt,
        model='nano-banana'  # Use Pro for tighter identity lock
    )
    
    print(f"Hero generated: {hero_url}")
    print("Review and approve before generating additional angles...")
    
    # Wait for user approval (in agent context)
    input("Press Enter when approved...")
    
    # Step 2: Generate 9 additional angles
    angles = [
        ('3/4 view, slight smile, looking slightly left', 3),
        ('Profile view, side lighting', 3),
        ('Close-up of face, neutral expression', 2),
        ('Full body shot, standing pose', 1),
        ('Laughing expression, candid moment', 2),
        ('Serious expression, professional', 2)
        # ... 3 more angles
    ]
    
    character_sheet = [hero_url]
    
    for angle_desc, count in angles:
        for _ in range(count):
            angle_prompt = f"""
            {description}
            {angle_desc}
            Match character from reference image exactly.
            Same lighting style, same person.
            """
            
            img_url = generate_nano_banana_image(
                prompt=angle_prompt,
                reference_image_path=download_temp(hero_url),
                model='nano-banana'  # Pro for consistency
            )
            character_sheet.append(img_url)
    
    # Save to references folder
    save_character_sheet('sofia', character_sheet)
    
    return character_sheet

# Example
influencer = create_ai_influencer(
    description="""
    Woman, 24 years old, warm smile.
    Light freckles, wavy brown hair.
    College student aesthetic.
    Natural makeup, approachable energy.
    """
)
```

### Caption Burn-In Workflow

Add captions to any finished video using Whisper + HyperFrames.

```bash
# Requires: pip install openai-whisper, npm install -g hyperframes

./shared/skills/caption-video/burn_captions.sh \
  --input output/video.mp4 \
  --output output/video_captioned.mp4 \
  --style "viral"
```

**Python implementation:**

```python
import whisper
import subprocess
import json

def burn_captions(video_path, output_path, style='viral'):
    """Add captions to video using Whisper + HyperFrames."""
    # Step 1: Transcribe with Whisper
    model = whisper.load_model("medium.en")
    result = model.transcribe(video_path, word_timestamps=True)
    
    # Step 2: Group words into reading phrases
    phrases = group_into_phrases(result['segments'])
    
    # Step 3: Generate HyperFrames caption JSON
    caption_json = generate_hyperframes_config(phrases, style)
    
    with open('captions_config.json', 'w') as f:
        json.dump(caption_json, f)
    
    # Step 4: Render with HyperFrames
    subprocess.run([
        'npx', 'hyperframes',
        '--input', video_path,
        '--config', 'captions_config.json',
        '--output', output_path
    ])
    
    return output_path

def group_into_phrases(segments, max_words=4):
    """Group word-level timestamps into reading phrases."""
    phrases = []
    current_phrase = []
    
    for segment in segments:
        for word in segment.get('words', []):
            current_phrase.append(word)
            
            if len(current_phrase) >= max_words or word['word'].endswith(('.', '!', '?')):
                phrases.append({
                    'text': ' '.join([w['word'] for w in current_phrase]),
                    'start': current_phrase[0]['start'],
                    'end': current_phrase[-1]['end']
                })
                current_phrase = []
    
    return phrases

def generate_hyperframes_config(phrases, style):
    """Generate HyperFrames caption configuration."""
    styles = {
        'viral': {
            'font': 'Montserrat-ExtraBold',
            'fontSize': 72,
            'color': '#FFFFFF',
            'stroke': '#000000',
            'strokeWidth': 6,
            'position': 'center'
        },
        'minimal': {
            'font': 'Helvetica-Neue',
            'fontSize': 48,
            'color': '#FFFFFF',
            'stroke': 'none',
            'position': 'bottom'
        }
    }
    
    return {
        'captions': phrases,
        'style': styles.get(style, styles['viral'])
    }
```

## Configuration

### API Base URL

Default: `https://api.arcads.ai`

Override via environment variable:
```bash
export ARCADS_API_BASE_URL=https://custom-endpoint.com
```

### Model Selection

```python
# Video models
SEEDANCE_2 = 'seedance-2'
SORA_2 = 'sora-2'
VEO_3_1 = 'veo-3.1'
KLING_3 = 'kling-3.0'
GROK_VIDEO = 'grok-video'

# Image models
NANO_BANANA_2 = 'nano-banana-2'        # Standard
NANO_BANANA_PRO = 'nano-banana'        # Gemini 3 Pro (higher fidelity)
NANO_BANANA_EDIT = 'nano-banana-edit'  # Inpainting
CHATGPT_IMAGE_2 = 'gpt-image-2'
```

### Aspect Ratios

```python
ASPECT_RATIOS = {
    'vertical': '9:16',    # Stories, Reels
    'square': '1:1',       # Feed posts
    'meta-image': '4:5',   # Meta image ads
    'landscape': '16:9'    # YouTube, horizontal
}
```

### Duration Limits

```python
DURATION_LIMITS = {
    'seedance-2': (4, 15),
    'sora-2': (1, 20),
    'veo-3.1': (4, 15),
    'kling-3.0': (1, 10),
    'grok-video': (1, 10)
}
```

## Common Patterns

### Pattern: UGC Workflow (Still → Video)

```python
def ugc_workflow(influencer_ref, product_ref, script):
    """Complete UGC workflow: still generation → video animation."""
    # Step 1: Generate UGC still
    still_prompt = f"""
    iPhone selfie camera, natural bedroom lighting.
    {influencer_ref['description']}
    Holding product at chest level, genuine smile.
    Messy hair, casual tank top.
    Background: unmade bed, string lights.
    """
    
    still_url = generate_nano_banana_image(
        prompt=still_prompt,
        reference_image_path=influencer_ref['hero_path']
    )
    
    print(f"Still generated: {still_url}")
    input("Review and approve...")
    
    # Step 2: Animate with Veo 3.1
    video_url = animate_with_veo(
        start_frame_path=download_temp(still_url),
        prompt=f"""
        Natural head and hand movement.
        Maintains eye contact with camera.
        Slight smile, authentic energy.
        Product stays in frame throughout.
        """,
        duration=12
    )
    
    return video_url
```

### Pattern: Batch Generation

```python
def batch_generate_variations(base_prompt, variations, model='nano-banana-2'):
    """Generate multiple variations in parallel."""
    import concurrent.futures
    
    def generate_variation(variation):
        full_prompt = f"{base_prompt}\n\nVariation: {variation}"
        return generate_nano_banana_image(full_prompt, model=model)
    
    with concurrent.futures.ThreadPoolExecutor(max_workers=5) as executor:
        urls = list(executor.map(generate_variation, variations))
    
    return urls

# Example: Generate 6 thumbnail variations
thumbnails = batch_generate_variations(
    base_prompt="""
    YouTube thumbnail, 16:9 aspect ratio.
    Man, 30s, excited expression, peace sign.
    Product held in other hand.
    Bold text overlay: "THIS CHANGED EVERYTHING"
    High contrast, vibrant colors.
    """,
    variations=[
        'Blue background, indoor studio lighting',
        'Yellow background, warm tone',
        'Outdoor natural light, green background',
        'Split-screen comparison layout',
        'Close-up face, product in foreground',
        'Full body shot, product placement emphasized'
    ]
)
```

### Pattern: Cost Estimation

```python
COST_TABLE = {
    'seedance-2': 0.25,      # per second
    'sora-2': 0.30,
    'veo-3.1': 0.20,
    'kling-3.0': 0.15,
    'nano-banana-2': 0.08,   # per image
    'nano-banana': 0.12,     # Pro
    'gpt-image-2': 0.10
}

def estimate_cost(model, duration=None, image_count=None):
    """Estimate generation cost."""
    unit_cost = COST_TABLE.get(model, 0)
    
    if duration:
        return unit_cost * duration
    elif image_count:
        return unit_cost * image_count
    
    return unit_cost

# Example
cost = estimate_cost('seedance-2', duration=12)
print(f"Estimated cost: ${cost:.2f}")

user_input = input(f"Proceed with generation? (${cost:.2f}) [y/N]: ")
