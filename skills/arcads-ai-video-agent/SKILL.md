---
name: arcads-ai-video-agent
description: Create AI marketing videos and images using Arcads external API with agent skills for video generation, image ads, and animated campaigns
triggers:
  - "create an AI video ad"
  - "generate Seedance video"
  - "make a UGC selfie video"
  - "create Nano Banana image ad"
  - "build Pixar style animated ad"
  - "generate Meta image ad creative"
  - "create AI influencer character"
  - "animate image to video with Veo"
---

# Arcads AI Video Agent Skill

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## Overview

Arcads AI Video Agent provides programmatic access to the Arcads creative stack for generating AI marketing videos and images. Supports **Seedance 2.0**, **Sora 2**, **Veo 3.1**, **Kling 3.0**, **Grok Video**, **Nano Banana 2/Pro/Edit**, **ChatGPT Image 2**, **OmniHuman**, and **Audio-driven** video generation, plus 37 static Meta image-ad templates and multi-step pipelines for Pixar-style and claymation animated ads.

## Installation

### 1. Clone and setup

```bash
git clone https://github.com/krusemediallc/arcads-claude-code.git
cd arcads-claude-code
./scripts/setup.sh
```

The setup script will:
- Prompt for your Arcads API key (get it at [app.arcads.ai/settings/api](https://app.arcads.ai/settings/api))
- Save credentials to `.env` (git-ignored)
- Verify API connection
- Create `MASTER_CONTEXT.md` workspace file

### 2. Prerequisites

```bash
# Core requirement
python3.10+

# Optional tools for multi-step pipelines
brew install ffmpeg jq node

# Python packages (as needed)
pip install openai-whisper
pip install -r shared/skills/meta-ad-builder/scripts/requirements.txt
```

### 3. Environment configuration

`.env` file structure:
```bash
ARCADS_API_KEY=your_api_key_here
ARCADS_BASE_URL=https://api.arcads.ai
```

## Project Structure

```
arcads-claude-code/
├── shared/
│   └── skills/
│       ├── arcads-external-api/     # Core API skill
│       ├── image-ad-prompting/      # 37-template library
│       ├── chatgpt-image-ad/        # Typography/UI ads
│       ├── nano-banana-image-ad/    # Photoreal lifestyle ads
│       ├── image-ad-clone/          # Ad reverse engineering
│       ├── pixar-style-ad/          # 3D animated pipeline
│       ├── claymation-ad/           # Stop-motion pipeline
│       ├── generate-youtube-thumbnail/
│       └── meta-ad-builder/         # Meta Marketing API publisher
├── references/
│   ├── influencers/                 # Character sheets
│   ├── aesthetics/                  # Style references
│   └── products/                    # Product photos
├── scripts/
│   └── setup.sh
└── MASTER_CONTEXT.md               # Personal workspace
```

## Core API Usage

### Python API Client Pattern

```python
import os
import requests
import json
from typing import Dict, Any

class ArcadsClient:
    def __init__(self):
        self.api_key = os.getenv('ARCADS_API_KEY')
        self.base_url = os.getenv('ARCADS_BASE_URL', 'https://api.arcads.ai')
        self.headers = {
            'Authorization': f'Bearer {self.api_key}',
            'Content-Type': 'application/json'
        }
    
    def generate_video(self, endpoint: str, payload: Dict[str, Any]) -> Dict:
        """Generic video generation request"""
        response = requests.post(
            f'{self.base_url}{endpoint}',
            headers=self.headers,
            json=payload
        )
        response.raise_for_status()
        return response.json()
    
    def poll_job(self, job_id: str) -> Dict:
        """Poll job status until completion"""
        import time
        while True:
            response = requests.get(
                f'{self.base_url}/v1/jobs/{job_id}',
                headers=self.headers
            )
            data = response.json()
            
            if data['status'] in ['completed', 'failed']:
                return data
            
            time.sleep(5)  # Poll every 5 seconds
```

### Seedance 2.0 Video Generation

```python
client = ArcadsClient()

# UGC selfie-style product review
ugc_payload = {
    "model": "seedance-2",
    "duration": 12,
    "prompt": """
    iPhone selfie POV — 28-year-old woman, kitchen counter, natural light from window.
    She's holding [PRODUCT NAME] at chest level, looking directly at camera.
    
    0-3s: "So I used to buy [competitor brand]..."
    Eye contact breaks naturally every 2-3 seconds, shifts weight, small hand gestures.
    
    4-8s: "...but then I found [YOUR BRAND]."
    Slight smile, brings product closer to camera, genuine enthusiasm.
    
    9-12s: "It's literally half the price and works even better."
    Nods affirmatively, maintains eye contact, casual confidence.
    
    Aesthetic: Natural kitchen lighting, iPhone 14 Pro quality, casual delivery,
    authentic micro-expressions, no polish.
    """,
    "aspectRatio": "9:16",
    "style": "ugc-selfie"
}

result = client.generate_video('/v1/seedance-2/generate', ugc_payload)
job_id = result['jobId']

# Poll for completion
final_result = client.poll_job(job_id)
video_url = final_result['output']['videoUrl']
print(f"Video ready: {video_url}")
```

### Premium Product Reveal (No Person)

```python
premium_payload = {
    "model": "seedance-2",
    "duration": 15,
    "prompt": """
    Dark void background (matte black, no visible floor or walls).
    
    0-3s: Text overlay fades in: "The problem with [category]..."
    Product enters frame slowly from right, soft spotlight.
    
    4-7s: Text: "...is that you're paying for the brand."
    Product rotates 90°, hero angle, cinematic lighting (key + rim).
    
    8-11s: Text: "This does the exact same thing."
    Zoom in 20%, product details visible, mist effect at base.
    
    12-15s: Text: "For 60% less." + logo lockup bottom right.
    Product holds center, slow rotation continues.
    
    Aesthetic: Premium commercial, dark void, text-driven narrative,
    no human presence, hero product focus.
    """,
    "aspectRatio": "1:1",
    "style": "premium-reveal"
}

result = client.generate_video('/v1/seedance-2/generate', premium_payload)
```

### Image-to-Video with Reference

```python
import base64

# Load product image
with open('references/products/bottle.png', 'rb') as f:
    image_b64 = base64.b64encode(f.read()).decode('utf-8')

i2v_payload = {
    "model": "seedance-2",
    "duration": 8,
    "startFrame": image_b64,  # Base64 image
    "prompt": """
    Product hero shot — water splash rising from bottom, droplets suspended mid-air,
    slow 180° rotation, backlit with blue-green gradient, mist wisps around base.
    
    Camera: Locked position, product rotates in place.
    Lighting: Dramatic backlight, rim light on edges, subtle fill from front.
    """,
    "aspectRatio": "4:5"
}

result = client.generate_video('/v1/seedance-2/image-to-video', i2v_payload)
```

## Veo 3.1 Starting-Frame Animation

```python
# Animate a Nano Banana still into video with dialogue
with open('outputs/ugc-still.png', 'rb') as f:
    still_b64 = base64.b64encode(f.read()).decode('utf-8')

veo_payload = {
    "model": "veo-3.1",
    "duration": 8,
    "startFrame": still_b64,
    "dialogue": "This product changed everything for me — I'm never going back.",
    "prompt": """
    Natural human motion: slight head nods, blinks, hand gestures while speaking.
    Maintains eye contact with camera, genuine enthusiasm in expression.
    Background: kitchen, soft natural light, shallow depth of field.
    """,
    "aspectRatio": "9:16"
}

result = client.generate_video('/v1/veo-3.1/animate', veo_payload)
```

## Sora 2 Text-to-Video

```python
# Longer-form video (up to 20s)
sora_payload = {
    "model": "sora-2",
    "duration": 16,  # Auto-calculated from word count
    "prompt": """
    Busy coffee shop interior, morning light through large windows.
    Young professional (30s, business casual) sits at corner table with laptop.
    
    Camera: Slow dolly-in from medium to medium-close-up over 16 seconds.
    
    She looks up from screen, slight smile of realization, picks up phone,
    taps a few times (implied: ordering your product), sets it down,
    returns to laptop with satisfied expression.
    
    Background: Barista making drinks (soft focus), other patrons at tables,
    ambient cafe motion. Warm color grade, natural lighting, authentic location.
    """,
    "aspectRatio": "16:9"
}

result = client.generate_video('/v2/videos/generate', sora_payload)
```

## Nano Banana Image Generation

### Create AI Influencer Character Sheet

```python
def create_influencer(name: str, description: str) -> list:
    """Generate 10-image character sheet"""
    client = ArcadsClient()
    
    # Phase 1: Hero front portrait
    hero_payload = {
        "model": "nano-banana-2",
        "prompt": f"""
        {description}
        
        Shot: Direct front-facing portrait, eye-level camera, natural expression.
        Lighting: Soft diffused light (golden hour quality), no harsh shadows.
        Focus: Face fills 60% of frame, sharp focus on eyes.
        Style: iPhone 14 Pro portrait mode, realistic skin texture, freckles visible,
        natural pores and fine lines, no AI polish.
        """,
        "aspectRatio": "4:5",
        "negativePrompt": "smooth skin, plastic, doll-like, perfect, polished, studio lighting"
    }
    
    response = client.generate_video('/v1/nano-banana/generate', hero_payload)
    hero_job = client.poll_job(response['jobId'])
    hero_url = hero_job['output']['imageUrl']
    
    # Download hero for reference
    hero_image = requests.get(hero_url).content
    hero_b64 = base64.b64encode(hero_image).decode('utf-8')
    
    # Phase 2: Generate 9 additional angles
    angles = [
        "3/4 view left, slight smile",
        "3/4 view right, neutral expression",
        "Profile left, looking slightly forward",
        "Profile right, looking slightly forward",
        "Close-up face, happy expression",
        "Close-up face, surprised expression",
        "Upper body, arms crossed, confident",
        "Full body, casual stance, jeans and t-shirt",
        "Environmental portrait, sitting on couch"
    ]
    
    character_sheet = [hero_url]
    
    for angle_desc in angles:
        angle_payload = {
            "model": "nano-banana-2",
            "prompt": f"{description}\n\nShot: {angle_desc}",
            "referenceImages": [hero_b64],  # Lock character identity
            "referenceStrength": 0.85,
            "aspectRatio": "4:5"
        }
        
        response = client.generate_video('/v1/nano-banana/generate', angle_payload)
        job = client.poll_job(response['jobId'])
        character_sheet.append(job['output']['imageUrl'])
    
    return character_sheet

# Usage
character_urls = create_influencer(
    name="Sofia",
    description="22-year-old college student, shoulder-length brown hair with subtle highlights, light freckles, hazel eyes, natural makeup, wearing casual college attire"
)
```

### UGC Product Selfie Still

```python
def generate_ugc_selfie(character_ref: str, product_ref: str, scene: str) -> str:
    """Generate authentic UGC-style product selfie"""
    client = ArcadsClient()
    
    # Load references
    with open(character_ref, 'rb') as f:
        char_b64 = base64.b64encode(f.read()).decode('utf-8')
    with open(product_ref, 'rb') as f:
        prod_b64 = base64.b64encode(f.read()).decode('utf-8')
    
    payload = {
        "model": "nano-banana-2",
        "prompt": f"""
        iPhone selfie POV — person holding phone with one hand, taking selfie.
        Other hand holds product at chest level, visible in frame.
        
        Scene: {scene}
        
        Expression: Natural smile, direct eye contact with camera lens.
        Lighting: Natural window light, slightly overexposed (authentic iPhone auto-exposure).
        Framing: Slight upward angle (typical selfie), person fills 70% of frame.
        
        Product: Clearly visible, recognizable, held naturally (not staged).
        Background: Real bedroom/kitchen, slight blur, authentic mess/lived-in look.
        
        Imperfections: Slight motion blur on hand, natural skin texture, 
        casual hair (not styled), real lighting conditions.
        """,
        "referenceImages": [char_b64, prod_b64],
        "referenceStrength": 0.75,
        "aspectRatio": "4:5",
        "negativePrompt": "professional photography, studio lighting, posed, perfect, polished"
    }
    
    response = client.generate_video('/v1/nano-banana/generate', payload)
    job = client.poll_job(response['jobId'])
    return job['output']['imageUrl']

# Usage
selfie_url = generate_ugc_selfie(
    character_ref='references/influencers/sofia/hero.png',
    product_ref='references/products/serum-bottle.png',
    scene='bedroom, morning light from window, unmade bed visible in background'
)
```

## ChatGPT Image 2 Static Ads

### Apple Notes List Ad

```python
def generate_apple_notes_ad(headline: str, benefits: list, cta: str) -> str:
    """Generate Apple Notes-style list ad"""
    client = ArcadsClient()
    
    # Format benefits as list items
    benefits_text = "\n".join([f"• {b}" for b in benefits])
    
    payload = {
        "model": "gpt-image-2",
        "prompt": f"""
        Apple Notes app screenshot on iPhone.
        
        Top: Note title in default font: "{headline}"
        Creation date: "Today 8:42 AM"
        
        Body (clean list):
        {benefits_text}
        
        Bottom: Empty line, then "{cta}" in bold.
        
        UI: Exact Apple Notes interface (iOS 17+), yellow background color,
        default system font (SF Pro), authentic margins and line spacing,
        realistic iPhone screen with slight rounded corners visible.
        
        No extra elements, no mockup frame, just the note interface.
        """,
        "aspectRatio": "9:16",
        "style": "ui-screenshot"
    }
    
    response = client.generate_video('/v1/chatgpt-image/generate', payload)
    job = client.poll_job(response['jobId'])
    return job['output']['imageUrl']

# Usage
ad_url = generate_apple_notes_ad(
    headline="Why I switched to [Brand]",
    benefits=[
        "Same ingredients as [competitor]",
        "60% cheaper (seriously)",
        "Subscribe & save another 15%",
        "Free shipping on orders $35+",
        "30-day money-back guarantee"
    ],
    cta="Try it risk-free → [brand].com"
)
```

### Comparison Table Ad

```python
def generate_comparison_table(your_brand: dict, competitors: list) -> str:
    """Generate side-by-side comparison table"""
    client = ArcadsClient()
    
    payload = {
        "model": "gpt-image-2",
        "prompt": f"""
        Clean comparison table — 3 columns, white background.
        
        Header row (bold):
        Feature | {your_brand['name']} | {competitors[0]['name']} | {competitors[1]['name']}
        
        Row 1: Price
        {your_brand['price']} ✓ | {competitors[0]['price']} | {competitors[1]['price']}
        
        Row 2: {your_brand['feature1_name']}
        ✓ Included | ✗ Not included | ✗ Not included
        
        Row 3: {your_brand['feature2_name']}
        ✓ Free | $12/month | $19/month
        
        Row 4: Shipping
        ✓ Free | $8.99 | $6.99
        
        Bottom: "{your_brand['name']} = ${your_brand['total_value']} in value" (highlighted)
        
        Style: Clean sans-serif font, generous padding, subtle borders,
        green checkmarks (✓), red X marks (✗), your brand column highlighted 
        with light yellow background.
        """,
        "aspectRatio": "1:1"
    }
    
    response = client.generate_video('/v1/chatgpt-image/generate', payload)
    job = client.poll_job(response['jobId'])
    return job['output']['imageUrl']
```

## Multi-Step Animated Pipelines

### Pixar-Style 3D Animated Ad

```python
import subprocess
import json

def generate_pixar_ad(product: str, story_beats: list) -> str:
    """Generate 8-beat Pixar-style animated ad"""
    client = ArcadsClient()
    
    # Step 1: Cast sheet
    cast_payload = {
        "model": "gpt-image-2",
        "prompt": f"""
        Character design sheet for Pixar-style 3D animation.
        
        Character: Anthropomorphized {product} — round friendly face,
        big expressive eyes, small arms and legs, vibrant colors.
        
        Sheet layout:
        - Front view (neutral)
        - 3/4 view (happy expression)
        - Side profile
        - Expression studies (happy, sad, surprised, determined)
        - Scale reference (compared to everyday objects)
        
        Style: Pixar character design, clean white background, professional
        character sheet layout, consistent lighting across all views.
        """,
        "aspectRatio": "16:9"
    }
    
    cast_response = client.generate_video('/v1/chatgpt-image/generate', cast_payload)
    cast_job = client.poll_job(cast_response['jobId'])
    cast_url = cast_job['output']['imageUrl']
    
    # Download cast sheet for reference
    cast_image = requests.get(cast_url).content
    cast_b64 = base64.b64encode(cast_image).decode('utf-8')
    
    # Step 2: Generate storyboard stills (8 beats)
    stills = []
    previous_frame = None
    
    for i, beat in enumerate(story_beats):
        refs = [cast_b64]
        if previous_frame:
            refs.append(previous_frame)
        
        still_payload = {
            "model": "gpt-image-2",
            "prompt": f"""
            Pixar-style 3D animation still — Beat {i+1}/8
            
            {beat['description']}
            
            Character: Same design from cast sheet, consistent appearance.
            Environment: {beat['environment']}
            Camera: {beat['camera_angle']}
            Lighting: Pixar-quality volumetric lighting, vibrant colors, soft shadows.
            
            Style: Pixar/Disney 3D animation, high-quality render, expressive character,
            clean composition, storytelling focus.
            """,
            "referenceImages": refs[:5],  # Max 5 refs
            "aspectRatio": "16:9"
        }
        
        response = client.generate_video('/v1/chatgpt-image/generate', still_payload)
        job = client.poll_job(response['jobId'])
        still_url = job['output']['imageUrl']
        stills.append(still_url)
        
        # Download for next iteration
        still_image = requests.get(still_url).content
        previous_frame = base64.b64encode(still_image).decode('utf-8')
    
    # Step 3: Animate each still with Seedance 2.0
    video_clips = []
    
    for i, still_url in enumerate(stills):
        still_image = requests.get(still_url).content
        still_b64 = base64.b64encode(still_image).decode('utf-8')
        
        animate_payload = {
            "model": "seedance-2",
            "duration": 3,  # 3s per beat
            "startFrame": still_b64,
            "prompt": f"""
            Animate this Pixar-style frame with natural motion:
            {story_beats[i]['motion']}
            
            Camera: {story_beats[i].get('camera_motion', 'Static hold')}
            Character: Maintain consistent appearance, smooth animation.
            """,
            "aspectRatio": "16:9"
        }
        
        response = client.generate_video('/v1/seedance-2/image-to-video', animate_payload)
        job = client.poll_job(response['jobId'])
        video_url = job['output']['videoUrl']
        
        # Download clip
        clip_path = f'temp/beat_{i+1}.mp4'
        with open(clip_path, 'wb') as f:
            f.write(requests.get(video_url).content)
        video_clips.append(clip_path)
    
    # Step 4: Stitch with ffmpeg
    concat_file = 'temp/concat.txt'
    with open(concat_file, 'w') as f:
        for clip in video_clips:
            f.write(f"file '{clip}'\n")
    
    output_path = 'outputs/pixar_ad_final.mp4'
    subprocess.run([
        'ffmpeg', '-f', 'concat', '-safe', '0',
        '-i', concat_file,
        '-c', 'copy', output_path
    ], check=True)
    
    return output_path

# Usage
story = [
    {
        "description": "Character wakes up in messy bedroom, alarm clock ringing",
        "environment": "Cozy bedroom, morning light through window",
        "camera_angle": "Wide shot from doorway",
        "motion": "Character sits up, rubs eyes, stretches"
    },
    {
        "description": "Character looks in mirror, sees problem (messy hair/tired eyes)",
        "environment": "Bathroom, mirror reflection",
        "camera_angle": "Over-shoulder medium shot",
        "motion": "Character leans closer to mirror, disappointed expression"
    },
    # ... 6 more beats
]

final_video = generate_pixar_ad(
    product="coffee mug",
    story_beats=story
)
```

### Burn Captions onto Video

```bash
#!/bin/bash
# Caption any video using Whisper + HyperFrames

VIDEO_PATH=$1
OUTPUT_PATH="outputs/$(basename "$VIDEO_PATH" .mp4)_captioned.mp4"

# Extract audio
ffmpeg -i "$VIDEO_PATH" -vn -acodec pcm_s16le -ar 16000 -ac 1 temp/audio.wav

# Transcribe with Whisper
whisper temp/audio.wav \
  --model medium.en \
  --output_format json \
  --output_dir temp/

# Generate captions with HyperFrames
npx hyperframes \
  --input "$VIDEO_PATH" \
  --transcript temp/audio.json \
  --output "$OUTPUT_PATH" \
  --style "modern" \
  --font "Arial Black" \
  --color "#FFFFFF" \
  --outline "#000000" \
  --position "center"

echo "Captioned video: $OUTPUT_PATH"
```

## Image Ad Cloning Workflow

```python
def clone_ad_as_template(source_image_path: str, template_name: str) -> dict:
    """Reverse-engineer existing ad into reusable template"""
    client = ArcadsClient()
    
    # Phase 1: Analyze ad structure
    with open(source_image_path, 'rb') as f:
        source_b64 = base64.b64encode(f.read()).decode('utf-8')
    
    analysis_prompt = f"""
    Analyze this ad creative and extract the prompt template.
    
    Identify:
    1. Visual structure (layout, composition, aspect ratio)
    2. Typography (font styles, hierarchy, placement)
    3. Color palette (background, text, accent colors)
    4. UI elements (if screenshot-style)
    5. Content pattern (what's variable vs. fixed)
    
    Output a prompt template with {{variables}} for customizable elements.
    """
    
    # Use GPT-4 Vision or similar to analyze
    # (Implementation depends on your analysis pipeline)
    
    # Phase 2: Validate against both backends
    test_payload_chatgpt = {
        "model": "gpt-image-2",
        "prompt": "{{GENERATED_TEMPLATE_PROMPT}}",
        "aspectRatio": "{{DETECTED_ASPECT_RATIO}}"
    }
    
    test_payload_nano = {
        "model": "nano-banana-2",
        "prompt": "{{GENERATED_TEMPLATE_PROMPT}}",
        "aspectRatio": "{{DETECTED_ASPECT_RATIO}}"
    }
    
    # Generate test images with both backends
    # Compare quality/fidelity
    # Save winning template to library
    
    template = {
        "name": template_name,
        "backend": "chatgpt-image-2",  # or "nano-banana-2"
        "prompt": "{{TEMPLATE}}",
        "aspectRatio": "9:16",
        "variables": ["headline", "bullet_points", "cta"],
        "validated": True
    }
    
    # Save to library
    library_path = f'shared/skills/image-ad-prompting/templates/{template_name}.json'
    with open(library_path, 'w') as f:
        json.dump(template, f, indent=2)
    
    return template
```

## Meta Ad Publishing

```python
def publish_to_meta(image_path: str, ad_config: dict) -> str:
    """Publish image ad to Meta Marketing API as paused ad"""
    
    # Upload image to Meta
    upload_response = subprocess.run([
        'python', 'shared/skills/meta-ad-builder/scripts/upload_image.py',
        '--image', image_path,
        '--account_id', os.getenv('META_AD_ACCOUNT_ID')
    ], capture_output=True, text=True, check=True)
    
    image_hash = upload_response.stdout.strip()
    
    # Create ad
    create_response = subprocess.run([
        'python', 'shared/skills/meta-ad-builder/scripts/create_ad.py',
        '--image_hash', image_hash,
        '--headline', ad_config['headline'],
        '--body', ad_config['body'],
        '--cta', ad_config['cta'],
        '--url', ad_config['landing_url'],
        '--status', 'PAUSED'  # Always start paused
    ], capture_output=True, text=True, check=True)
    
    ad_id = create_response.stdout.strip()
    return ad_id
```

## Configuration

### Model Selection Guide

| Use Case | Recommended Model | Duration | Notes |
|----------|------------------|----------|-------|
| UGC selfie video | Seedance 2.0 | 4-15s | Native audio, natural motion |
| Premium product reveal | Seedance 2.0 | 8-15s | Best for controlled studio shots |
| Long-form narrative | Sora 2 | up to 20s | Better coherence over time |
| Still → video with dialogue | Veo 3.1 | 5-10s | Best for `startFrame` + dialogue |
| B-roll / scene | Kling 3.0 | 5-10s | Dedicated b-roll endpoint |
| Talking avatar | OmniHuman | 5-30s | Lip-sync from text |
| Typography/UI ads | ChatGPT Image 2 | N/A | Best for text-heavy layouts |
| Photoreal lifestyle | Nano Banana 2 | N/A | Multi-reference character lock |
| High-fidelity portraits | Nano Banana Pro | N/A | Gemini 3 Pro backend |

### Aspect Ratio Compatibility

**Video models:**
- `9:16` (vertical) — recommended for UGC, Meta/TikTok/IG Reels
- `16:9` (horizontal) — YouTube pre-roll, landscape ads
- `1:1` (square) — Meta feed, universal fallback
- `4:5` (tall) — Instagram feed

**Image models:**
- ChatGPT Image 2: `1:1`, `9:16`, `16:9`
- Nano Banana: `1:1`, `4:5`, `9:16`, `16:9`

## Common Patterns

### Sequential Character Identity Lock

```python
def generate_multi_shot_sequence(character_ref: str, shots: list) -> list:
    """Generate sequence with consistent character across shots"""
    client = ArcadsClient()
    
    with open(character_ref, 'rb') as f:
        char_b64 = base64.b64encode(f.read()).decode('utf-8')
    
    outputs = []
    previous_frames = [char_b64]  # Start with hero reference
    
    for shot in shots:
        payload = {
            "model": "nano-banana-2",
            "prompt": shot['prompt'],
            "referenceImages": previous_frames[-5:],  # Last 5 frames max
            "referenceStrength": 0.85,
            "aspectRatio": shot['aspectRatio']
        }
        
        response = client.generate_video('/v1/nano-banana/generate', payload)
        job = client.poll_job(response['jobId'])
        output_url = job['output']['imageUrl
