---
name: arcads-ai-video-generation
description: Create AI marketing videos and image ads using Arcads API with Seedance 2.0, Sora 2, Veo 3.1, Kling 3.0, Nano Banana, and 37 Meta ad templates
triggers:
  - "generate an arcads video"
  - "create a seedance ugc video"
  - "make a nano banana image ad"
  - "animate this image with veo"
  - "build a pixar style ad campaign"
  - "create ai influencer character sheet"
  - "generate youtube thumbnail variations"
  - "make a claymation animated ad"
---

# Arcads AI Video Generation

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## Overview

Arcads AI Video is a comprehensive creative generation system that provides programmatic access to multiple AI video and image generation models through a unified API. The platform supports Seedance 2.0 (flagship video model), Sora 2, Veo 3.1, Kling 3.0, Grok Video, Nano Banana 2/Pro/Edit, ChatGPT Image 2, OmniHuman, and audio-driven generation. It includes a library of 37 validated static Meta image ad templates and multi-step pipelines for Pixar-style and claymation animated ads.

**Key capabilities:**
- Text-to-video and image-to-video generation across 8+ models
- UGC-style selfie videos with natural dialogue
- AI influencer character sheet creation (10-image consistency)
- Static Meta image ads (typography-heavy and photoreal)
- Multi-step animated ad pipelines with storyboarding
- YouTube thumbnail generation with CTR formulas
- Video caption burn-in and post-processing

## Installation

### Prerequisites

```bash
# Core requirements
python --version  # Must be 3.10+

# Optional tools for advanced pipelines
brew install ffmpeg      # Video stitching, chroma-key overlay
brew install jq          # JSON parsing in bash scripts
brew install node        # For caption burn-in with hyperframes

# Python packages for specific features
pip install openai-whisper  # Caption transcription
pip install requests        # API calls (usually pre-installed)
```

### Setup

```bash
# Clone the repository
git clone https://github.com/krusemediallc/arcads-claude-code.git
cd arcads-claude-code

# Run setup script
./scripts/setup.sh
```

The setup script will:
1. Prompt for your Arcads API key (get it from https://app.arcads.ai/settings/api)
2. Create a `.env` file with your credentials
3. Verify API connectivity
4. Create `MASTER_CONTEXT.md` workspace file

**Manual `.env` setup:**

```bash
# .env
ARCADS_API_KEY=your_api_key_here
```

## API Configuration

Base URL: `https://api.arcads.ai`

Authentication via header:
```
x-api-key: <your_api_key>
```

All API calls return JSON with standard structure:
```json
{
  "success": true,
  "data": { "id": "gen_xxx", "status": "queued" },
  "error": null
}
```

## Video Generation

### Seedance 2.0 (Flagship Model)

Seedance 2.0 supports 4–15 second clips with native audio, image-to-video, video-to-video, and multiple shot styles.

**Text-to-video:**

```python
import requests
import os
import time

API_KEY = os.environ.get("ARCADS_API_KEY")
BASE_URL = "https://api.arcads.ai"

def generate_seedance_video(prompt, duration=10, style="ugc"):
    """Generate a Seedance 2.0 video."""
    headers = {"x-api-key": API_KEY, "Content-Type": "application/json"}
    
    payload = {
        "prompt": prompt,
        "duration": duration,
        "style": style,  # Options: "ugc", "premium", "hero", "lookbook", "walkthrough"
        "aspectRatio": "9:16"  # Options: "16:9", "9:16", "1:1"
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/seedance2/generate",
        headers=headers,
        json=payload
    )
    
    data = response.json()
    if not data.get("success"):
        raise Exception(f"Generation failed: {data.get('error')}")
    
    return data["data"]["id"]

def poll_generation_status(generation_id):
    """Poll generation status until complete."""
    headers = {"x-api-key": API_KEY}
    
    while True:
        response = requests.get(
            f"{BASE_URL}/v1/generation/{generation_id}",
            headers=headers
        )
        
        data = response.json()["data"]
        status = data["status"]
        
        if status == "completed":
            return data["videoUrl"]
        elif status == "failed":
            raise Exception(f"Generation failed: {data.get('error')}")
        
        print(f"Status: {status} - waiting 5s...")
        time.sleep(5)

# Example: UGC selfie-style product review
ugc_prompt = """
Woman in bright kitchen, natural lighting, holding skincare product.
Direct eye contact, iPhone selfie angle.
Says: "I stopped buying expensive moisturizers after finding this."
Casual delivery, natural hand gestures.
"""

generation_id = generate_seedance_video(ugc_prompt, duration=12, style="ugc")
video_url = poll_generation_status(generation_id)
print(f"Video ready: {video_url}")
```

**Image-to-video with Seedance 2.0:**

```python
import base64

def image_to_video_seedance(image_path, prompt, duration=8):
    """Animate a still image with Seedance 2.0."""
    headers = {"x-api-key": API_KEY, "Content-Type": "application/json"}
    
    # Encode image as base64
    with open(image_path, "rb") as f:
        image_base64 = base64.b64encode(f.read()).decode("utf-8")
    
    payload = {
        "prompt": prompt,
        "startFrame": image_base64,  # Starting frame for animation
        "duration": duration,
        "aspectRatio": "9:16"
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/seedance2/image-to-video",
        headers=headers,
        json=payload
    )
    
    return response.json()["data"]["id"]

# Example: Animate product photo
generation_id = image_to_video_seedance(
    "references/products/moisturizer.png",
    "Product slowly rotates, light mist effect, water droplets appear",
    duration=8
)
```

**Seedance 2.0 prompt formulas** (from `prompting/prompt-library/`):

1. **UGC selfie-style** (`seedance-2-ugc.md`): 9-layer formula with iPhone aesthetic, eye-contact breaks, casual delivery
2. **Premium reveal** (`seedance-2-premium-reveal.md`): Dark void, text narrative, hero rotation
3. **Product hero** (`seedance-2-product-hero.md`): Elemental effects (splash, mist), slow rotation
4. **Studio lookbook** (`seedance-2-studio-lookbook.md`): Editorial multi-shot with voiceover
5. **Feature walkthrough** (`seedance-2-feature-walkthrough.md`): Fast-paced product demo

### Veo 3.1 (Starting-Frame Animation)

Veo 3.1 specializes in animating still images with natural human motion and embedded dialogue.

```python
def generate_veo_video(start_frame_path, dialogue, duration=8):
    """Generate Veo 3.1 video from starting frame."""
    headers = {"x-api-key": API_KEY, "Content-Type": "application/json"}
    
    with open(start_frame_path, "rb") as f:
        start_frame = base64.b64encode(f.read()).decode("utf-8")
    
    payload = {
        "startFrame": start_frame,
        "dialogue": dialogue,  # MANDATORY for UGC animations
        "duration": duration,
        "aspectRatio": "9:16"
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/veo3/generate",
        headers=headers,
        json=payload
    )
    
    return response.json()["data"]["id"]

# Example: Animate UGC still with dialogue
generation_id = generate_veo_video(
    "output/influencers/sofia_ugc_001.png",
    "This product changed my morning routine completely",
    duration=8
)
```

### Sora 2 (Long-Form Text-to-Video)

Sora 2 supports up to 20 seconds with automatic duration calculation from script word count.

```python
def generate_sora_video(prompt, style_reference_path=None):
    """Generate Sora 2 video with optional style reference."""
    headers = {"x-api-key": API_KEY, "Content-Type": "application/json"}
    
    # Auto-calculate duration (~2.5 words/second)
    word_count = len(prompt.split())
    duration = min(20, max(5, int(word_count / 2.5)))
    
    payload = {
        "prompt": prompt,
        "duration": duration,
        "aspectRatio": "16:9"
    }
    
    if style_reference_path:
        with open(style_reference_path, "rb") as f:
            payload["styleReference"] = base64.b64encode(f.read()).decode("utf-8")
    
    response = requests.post(
        f"{BASE_URL}/v1/sora2/generate",
        headers=headers,
        json=payload
    )
    
    return response.json()["data"]["id"]

# Example: Cinematic product launch
sora_prompt = """
Cinematic reveal of new smartphone on minimalist pedestal.
Camera orbits slowly. Dramatic lighting with volumetric fog.
Product glows subtly as features materialize in AR text overlays.
Professional studio environment with depth of field.
"""

generation_id = generate_sora_video(
    sora_prompt,
    style_reference_path="references/aesthetics/premium-tech.png"
)
```

### Kling 3.0 (B-Roll & Scenes)

Kling 3.0 optimized for 5-second b-roll clips and environmental scenes.

```python
def generate_kling_broll(scene_description):
    """Generate b-roll clip with Kling 3.0."""
    headers = {"x-api-key": API_KEY, "Content-Type": "application/json"}
    
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
    
    return response.json()["data"]["id"]

# Example: Coffee shop b-roll
generation_id = generate_kling_broll(
    "Cozy coffee shop interior, steam rising from latte, warm lighting, bokeh background"
)
```

### Other Video Models

**Grok Video:**

```python
def generate_grok_video(prompt, duration=10):
    """Generate video with Grok model."""
    headers = {"x-api-key": API_KEY, "Content-Type": "application/json"}
    
    payload = {
        "model": "grok-video",
        "prompt": prompt,
        "duration": duration
    }
    
    response = requests.post(
        f"{BASE_URL}/v2/videos/generate",
        headers=headers,
        json=payload
    )
    
    return response.json()["data"]["id"]
```

**OmniHuman (Talking Avatar):**

```python
def generate_omnihuman(person_description, script):
    """Generate talking avatar video."""
    headers = {"x-api-key": API_KEY, "Content-Type": "application/json"}
    
    payload = {
        "personDescription": person_description,
        "script": script,
        "aspectRatio": "9:16"
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/omnihuman",
        headers=headers,
        json=payload
    )
    
    return response.json()["data"]["id"]
```

## Image Generation

### Nano Banana (AI Influencer & UGC)

Nano Banana 2 is the default model. Use `nano-banana` (Pro) for tighter identity lock or `nano-banana-edit` for inpainting.

**Create AI influencer character sheet (10 images):**

```python
def create_ai_influencer(description, name):
    """Generate 10-image character sheet with consistent identity."""
    headers = {"x-api-key": API_KEY, "Content-Type": "application/json"}
    
    # Phase 1: Generate hero portrait
    hero_payload = {
        "model": "nano-banana-2",
        "prompt": f"{description}. Front-facing portrait, soft natural light, 50mm lens, eye contact.",
        "aspectRatio": "9:16",
        "quality": "high"
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/nano-banana/generate",
        headers=headers,
        json=hero_payload
    )
    
    hero_id = response.json()["data"]["id"]
    
    # Poll for hero completion
    hero_url = poll_generation_status(hero_id)
    
    # Download hero image for reference
    hero_image = requests.get(hero_url).content
    hero_base64 = base64.b64encode(hero_image).decode("utf-8")
    
    # Phase 2: Generate 9 additional angles with hero as reference
    angles = [
        "3/4 view from right, slight smile",
        "Profile view from left, looking away",
        "Close-up of face, expression of surprise",
        "Full body shot, casual pose, arms crossed",
        "Over-the-shoulder glance back at camera",
        "Laughing candid moment, head tilted",
        "Serious expression, direct eye contact",
        "Side profile with hair details visible",
        "Relaxed pose, hand near face, thoughtful"
    ]
    
    generation_ids = []
    
    for angle in angles:
        payload = {
            "model": "nano-banana-2",
            "prompt": f"{description}. {angle}",
            "referenceImages": [hero_base64],  # Lock identity
            "aspectRatio": "9:16",
            "quality": "high"
        }
        
        response = requests.post(
            f"{BASE_URL}/v1/nano-banana/generate",
            headers=headers,
            json=payload
        )
        
        generation_ids.append(response.json()["data"]["id"])
    
    return hero_id, generation_ids

# Example: Create college student influencer
hero_id, angle_ids = create_ai_influencer(
    "22-year-old college student, freckles, wavy auburn hair, casual style, warm smile",
    "sofia"
)
```

**UGC product selfie still:**

```python
def generate_ugc_selfie(character_ref_path, product_ref_path, prompt):
    """Generate authentic UGC selfie with character and product."""
    headers = {"x-api-key": API_KEY, "Content-Type": "application/json"}
    
    # Load reference images
    with open(character_ref_path, "rb") as f:
        character_base64 = base64.b64encode(f.read()).decode("utf-8")
    
    with open(product_ref_path, "rb") as f:
        product_base64 = base64.b64encode(f.read()).decode("utf-8")
    
    # Load UGC aesthetic references
    aesthetic_refs = []
    aesthetic_dir = "references/aesthetics/ugc-selfie/"
    for img in os.listdir(aesthetic_dir)[:3]:
        with open(os.path.join(aesthetic_dir, img), "rb") as f:
            aesthetic_refs.append(base64.b64encode(f.read()).decode("utf-8"))
    
    payload = {
        "model": "nano-banana-2",
        "prompt": f"""{prompt}
        iPhone selfie aesthetic. Slight motion blur. Natural skin texture with pores visible.
        Imperfect framing. Bedroom or bathroom background slightly out of focus.
        Casual lighting from window. Authentic expression, not posed.
        """,
        "referenceImages": [character_base64, product_base64] + aesthetic_refs,
        "aspectRatio": "9:16",
        "quality": "high"
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/nano-banana/generate",
        headers=headers,
        json=payload
    )
    
    return response.json()["data"]["id"]

# Example: Sofia holding skincare product
generation_id = generate_ugc_selfie(
    "references/influencers/sofia_hero.png",
    "references/products/moisturizer.png",
    "Sofia holding moisturizer bottle, examining it with interest, soft morning light"
)
```

**Nano Banana Pro (tighter identity lock):**

```python
def generate_with_nano_banana_pro(prompt, reference_images):
    """Use Nano Banana Pro for maximum identity consistency."""
    headers = {"x-api-key": API_KEY, "Content-Type": "application/json"}
    
    payload = {
        "model": "nano-banana",  # Pro model
        "prompt": prompt,
        "referenceImages": reference_images,  # Up to 5 references
        "aspectRatio": "9:16",
        "quality": "high"
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/nano-banana/generate",
        headers=headers,
        json=payload
    )
    
    return response.json()["data"]["id"]
```

### ChatGPT Image 2 (Typography & UI)

ChatGPT Image 2 excels at typography-heavy and UI-mimicry creatives.

```python
def generate_chatgpt_image(prompt, aspect_ratio="1:1"):
    """Generate image with ChatGPT Image 2."""
    headers = {"x-api-key": API_KEY, "Content-Type": "application/json"}
    
    payload = {
        "model": "gpt-image-2",
        "prompt": prompt,
        "aspectRatio": aspect_ratio,  # "1:1", "16:9", "9:16"
        "quality": "high"
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/chatgpt-image/generate",
        headers=headers,
        json=payload
    )
    
    return response.json()["data"]["id"]

# Example: Apple Notes list ad
notes_prompt = """
iPhone screenshot of Apple Notes app.
Title: "Why I Switched to [Product Name]"
Bulleted list:
• Saves me 2 hours every day
• Cut my costs by 60%
• Actually works (unlike competitors)
• Setup took 5 minutes
• ROI in first week

Clean iOS UI, light mode, realistic phone bezels.
"""

generation_id = generate_chatgpt_image(notes_prompt, aspect_ratio="9:16")
```

## Static Meta Image Ads (37 Template Library)

The repo includes 37 validated prompt templates for static Meta image ads. Templates are stored in `shared/skills/image-ad-prompting/templates/`.

**Available template categories:**
- Typography & lists (Apple Notes, sticky notes, Forbes editorial)
- UI mimicry (Google search, Slack threads, ChatGPT conversations, iMessage)
- Comparison & social proof (comparison tables, before/after, reviews)
- Editorial & lifestyle (magazine covers, billboards, museum exhibits)
- Gamification (scratch-off tickets, weather forecast, dating app cards)

**Decision tree:**

| Template Type | Backend | Aspect Ratios |
|---|---|---|
| Typography-heavy, UI | ChatGPT Image 2 | 1:1, 9:16, 16:9 |
| Photoreal, lifestyle | Nano Banana 2 | 1:1, 9:16, 4:5 |
| Multi-reference | Nano Banana Pro | All |

**Generate from template:**

```python
import json

def load_template(template_name):
    """Load prompt template from library."""
    template_path = f"shared/skills/image-ad-prompting/templates/{template_name}.json"
    with open(template_path, "r") as f:
        return json.load(f)

def generate_from_template(template_name, variables):
    """Generate image ad from template with variable substitution."""
    template = load_template(template_name)
    
    # Substitute variables in prompt
    prompt = template["prompt"]
    for key, value in variables.items():
        prompt = prompt.replace(f"{{{key}}}", value)
    
    # Determine backend
    backend = template.get("backend", "nano-banana-2")
    
    if backend == "gpt-image-2":
        return generate_chatgpt_image(prompt, template.get("aspectRatio", "1:1"))
    else:
        headers = {"x-api-key": API_KEY, "Content-Type": "application/json"}
        payload = {
            "model": backend,
            "prompt": prompt,
            "aspectRatio": template.get("aspectRatio", "1:1"),
            "quality": "high"
        }
        
        response = requests.post(
            f"{BASE_URL}/v1/nano-banana/generate",
            headers=headers,
            json=payload
        )
        
        return response.json()["data"]["id"]

# Example: Apple Notes template
generation_id = generate_from_template(
    "apple-notes-list",
    {
        "product_name": "AI Writing Assistant",
        "benefit_1": "Writes better than ChatGPT",
        "benefit_2": "Knows your brand voice",
        "benefit_3": "10x faster than hiring",
        "benefit_4": "Pays for itself in week 1"
    }
)
```

**Clone existing ad as template:**

```python
def clone_ad_as_template(image_path, backend="nano-banana-2"):
    """Reverse-engineer existing ad into reusable template."""
    headers = {"x-api-key": API_KEY, "Content-Type": "application/json"}
    
    with open(image_path, "rb") as f:
        image_base64 = base64.b64encode(f.read()).decode("utf-8")
    
    # Phase 1: Analyze image structure
    analyze_payload = {
        "image": image_base64,
        "analysisType": "ad-structure"
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/analyze",
        headers=headers,
        json=analyze_payload
    )
    
    structure = response.json()["data"]
    
    # Phase 2: Extract template prompt
    template_prompt = structure["reconstructedPrompt"]
    variables = structure["identifiedVariables"]
    
    # Phase 3: Validate by regenerating
    validation_id = generate_chatgpt_image(template_prompt) if backend == "gpt-image-2" else generate_with_nano_banana_pro(template_prompt, [])
    
    return {
        "template": template_prompt,
        "variables": variables,
        "validationId": validation_id
    }

# Example: Clone competitor ad
template_data = clone_ad_as_template(
    "references/competitor-ads/comparison-table.png",
    backend="gpt-image-2"
)
```

## Multi-Step Animated Ad Pipelines

### Pixar-Style 3D Animated Ad

8-beat story arc with anthropomorphized characters, storyboarded with ChatGPT Image 2, animated with Seedance 2.0.

```python
import subprocess

def generate_pixar_ad(product_name, story_beats):
    """Generate Pixar-style animated ad with 8-beat story."""
    headers = {"x-api-key": API_KEY, "Content-Type": "application/json"}
    
    # Phase 1: Generate cast sheet (character designs)
    cast_prompt = f"""
    3D Pixar character design sheet for {product_name} mascot.
    Anthropomorphized product with expressive eyes, friendly smile.
    Multiple angles: front, 3/4, side, back.
    Turnaround reference. Clean white background.
    Professional character design, animation-ready.
    """
    
    cast_id = generate_chatgpt_image(cast_prompt, aspect_ratio="16:9")
    cast_url = poll_generation_status(cast_id)
    
    # Download cast reference
    cast_image = requests.get(cast_url).content
    cast_base64 = base64.b64encode(cast_image).decode("utf-8")
    
    # Phase 2: Generate storyboard stills (sequential with identity lock)
    storyboard_ids = []
    previous_frame = None
    
    for i, beat in enumerate(story_beats):
        refs = [cast_base64]
        if previous_frame:
            refs.append(previous_frame)  # Sequential identity lock
        
        # Limit to 5 reference images (ChatGPT Image 2 constraint)
        refs = refs[:5]
        
        frame_prompt = f"""
        Pixar-style 3D animation frame. {beat}
        Same character from cast sheet. Vibrant colors, soft lighting.
        Expressive pose, clear storytelling. Clean composition.
        """
        
        payload = {
            "model": "gpt-image-2",
            "prompt": frame_prompt,
            "referenceImages": refs,
            "aspectRatio": "16:9",
            "quality": "high"
        }
        
        response = requests.post(
            f"{BASE_URL}/v1/chatgpt-image/generate",
            headers=headers,
            json=payload
        )
        
        frame_id = response.json()["data"]["id"]
        storyboard_ids.append(frame_id)
        
        # Get frame for next iteration's reference
        frame_url = poll_generation_status(frame_id)
        frame_image = requests.get(frame_url).content
        previous_frame = base64.b64encode(frame_image).decode("utf-8")
    
    # Phase 3: Animate each frame with Seedance 2.0 i2v
    video_ids = []
    
    for frame_id in storyboard_ids:
        frame_url = poll_generation_status(frame_id)
        frame_image = requests.get(frame_url).content
        frame_base64 = base64.b64encode(frame_image).decode("utf-8")
        
        animate_payload = {
            "prompt": "Smooth natural motion, character movement, Pixar-style animation",
            "startFrame": frame_base64,
            "duration": 3,
            "aspectRatio": "16:9"
        }
        
        response = requests.post(
            f"{BASE_URL}/v1/seedance2/image-to-video",
            headers=headers,
            json=animate_payload
        )
        
        video_ids.append(response.json()["data"]["id"])
    
    # Phase 4: Download video clips
    video_clips = []
    for video_id in video_ids:
        video_url = poll_generation_status(video_id)
        video_path = f"output/pixar_beat_{video_ids.index(video_id)}.mp4"
        
        with open(video_path, "wb") as f:
            f.write(requests.get(video_url).content)
        
        video_clips.append(video_path)
    
    # Phase 5: Stitch with ffmpeg
    concat_file = "output/concat_list.txt"
    with open(concat_file, "w") as f:
        for clip in video_clips:
            f.write(f"file '{clip}'\n")
    
    output_path = f"output/{product_name}_pixar_ad.mp4"
    subprocess.run([
        "ffmpeg", "-f", "concat", "-safe", "0",
        "-i", concat_file,
        "-c", "copy", output_path
    ])
    
    return output_path

# Example: 8-beat product story
story_beats = [
    "Character struggles with messy desk, papers everywhere, frustrated expression",
    "Discovers magical product box, eyes light up with curiosity",
    "Opens box, product glows with warm light, character amazed",
    "Character uses product, desk organizes itself with sparkles",
    "Happy character working efficiently, smooth workflow",
    "Friends arrive, see organized space, expressions of surprise",
    "Group celebration, product in center, everyone cheering",
    "Character winks at camera, product logo visible, fade to brand colors"
]

final_video = generate_pixar_ad("TaskMaster Pro", story_beats)
print(f"Pixar ad complete: {final_video}")
```

### YouTube Thumbnail Generation

Generate CTR-optimized thumbnails with 5 proven formulas.

```python
def generate_youtube_thumbnails(face_refs, product_ref, formula="peace-sign"):
    """Generate YouTube thumbnail with CTR formula."""
    headers = {"x-api-key": API_KEY, "Content-Type": "application/json"}
    
    # Load face references (5+ for tight likeness lock)
    face_base64_list = []
    for face_path in face_refs:
        with open(face_path, "rb") as f:
            face_base64_list.append(base64.b64encode(f.read()).decode("utf-8"))
    
    # Load product reference
    with open(product_ref, "rb") as f:
        product_base64 = base64.b64encode(f.read()).decode("utf-8")
    
    formulas = {
        "peace-sign": """
        YouTube thumbnail. Person making peace sign, pointing at product with other hand.
        Bright background (solid color or subtle gradient). Big genuine smile.
        Eye contact with camera. High contrast, vibrant colors. Product prominent.
        """,
        "real-vs-ai": """
        YouTube thumbnail split-screen. Left: "REAL" label, actual product photo.
        Right: "AI" label, generated version. Big text overlays. Dramatic comparison.
        Person in center pointing at both sides, shocked expression. High contrast.
        """,
        "terminal-flow": """
        YouTube thumbnail. Person centered, code terminal visible in background.
        Glowing green/blue terminal text. Product floating holographically.
        Tech aesthetic, clean composition. Person gesturing toward screen.
        """,
        "reaction-shock": """
        YouTube thumbnail. Person with exaggerated shocked expression, hands on face.
        Product visible with dramatic lighting or glow effect. Bold text overlay.
        High energy, bright colors, eye-catching. Clear focal point.
        """,
        "before-after": """
        YouTube thumbnail split vertically. Left: "BEFORE"
