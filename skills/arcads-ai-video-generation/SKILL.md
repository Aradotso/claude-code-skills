---
name: arcads-ai-video-generation
description: Create AI marketing videos and images using Arcads API with Seedance 2.0, Sora 2, Veo 3.1, Kling, Nano Banana, and 37 static ad templates
triggers:
  - "generate an Arcads video"
  - "create UGC content with Seedance"
  - "make a Nano Banana AI influencer"
  - "build a Pixar-style animated ad"
  - "generate static Meta image ads"
  - "create claymation advertisement"
  - "animate product showcase with Veo"
  - "poll Arcads generation status"
---

# Arcads AI Video Generation

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

Create AI marketing videos and images using the Arcads API. Supports the full creative stack: **Seedance 2.0** (flagship video model), **Sora 2**, **Veo 3.1**, **Kling 3.0**, **Grok Video**, **Nano Banana 2/Pro/Edit**, **ChatGPT Image 2**, **OmniHuman**, and **Audio-driven** lip-sync. Also includes a 37-template static Meta image-ad library and multi-step pipelines for Pixar-style and claymation animated ads.

## Installation

Clone the repository and run setup:

```bash
git clone https://github.com/krusemediallc/arcads-claude-code.git
cd arcads-claude-code
./scripts/setup.sh
```

Setup will:
- Prompt for your **Arcads API key** from [app.arcads.ai/settings/api](https://app.arcads.ai/settings/api)
- Save it securely in `.env` (never committed)
- Verify API connection
- Create `MASTER_CONTEXT.md` workspace file

### Prerequisites

| Tool | Required for | Install |
|------|-------------|---------|
| Python 3.10+ | Core functionality | `brew install python@3.12` |
| ffmpeg | Pixar/claymation pipelines, video stitching | `brew install ffmpeg` |
| jq | Bash workflow scripts | `brew install jq` |
| Node.js | Caption burn-in (hyperframes) | `brew install node` |
| whisper | Caption transcription | `pip install openai-whisper` |

Image-ad generators (`chatgpt-image-ad`, `nano-banana-image-ad`) are stdlib-only — no pip installs required.

## Configuration

### Environment Variables

```bash
# .env (created by setup.sh)
ARCADS_API_KEY=your_api_key_here
```

### API Base URL

```python
BASE_URL = "https://api.arcads.ai"
```

All endpoints require the `Authorization: Bearer <API_KEY>` header.

## Core API Usage

### Python API Client Pattern

```python
import os
import requests
import time
from dotenv import load_dotenv

load_dotenv()
API_KEY = os.getenv("ARCADS_API_KEY")
BASE_URL = "https://api.arcads.ai"

headers = {
    "Authorization": f"Bearer {API_KEY}",
    "Content-Type": "application/json"
}

def poll_generation(generation_id, endpoint_type="videos"):
    """Poll until generation completes"""
    poll_url = f"{BASE_URL}/v1/{endpoint_type}/{generation_id}"
    
    while True:
        response = requests.get(poll_url, headers=headers)
        data = response.json()
        
        status = data.get("status")
        if status == "completed":
            return data.get("videoUrl") or data.get("imageUrl")
        elif status == "failed":
            raise Exception(f"Generation failed: {data.get('error')}")
        
        time.sleep(5)  # Poll every 5 seconds
```

## Video Generation Models

### Seedance 2.0 (Flagship Model)

4-15 second clips with native audio, image-to-video or video-to-video, reference images support.

**Text-to-Video:**

```python
def generate_seedance_video(prompt, duration=10, aspect_ratio="9:16"):
    """Generate Seedance 2.0 video from text prompt"""
    payload = {
        "prompt": prompt,
        "duration": duration,
        "aspectRatio": aspect_ratio,
        "model": "seedance-2"
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/videos/generate",
        headers=headers,
        json=payload
    )
    
    data = response.json()
    generation_id = data["generationId"]
    
    # Poll for completion
    return poll_generation(generation_id)

# Example: UGC selfie-style product review
prompt = """
Medium shot, 25-year-old woman in bright kitchen, iPhone selfie angle, 
holding product at chest height. Natural eye contact with camera, 
casual smile. Says: "I stopped buying [competitor] after trying this."
Warm morning light, authentic UGC aesthetic, slight camera shake.
"""

video_url = generate_seedance_video(prompt, duration=12, aspect_ratio="9:16")
print(f"Video ready: {video_url}")
```

**Image-to-Video (Start Frame):**

```python
import base64

def seedance_from_image(image_path, prompt, duration=10):
    """Animate a still image with Seedance 2.0"""
    with open(image_path, "rb") as f:
        image_base64 = base64.b64encode(f.read()).decode()
    
    payload = {
        "prompt": prompt,
        "duration": duration,
        "aspectRatio": "9:16",
        "model": "seedance-2",
        "startFrame": image_base64
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/videos/generate",
        headers=headers,
        json=payload
    )
    
    return poll_generation(response.json()["generationId"])

# Animate product showcase still
video_url = seedance_from_image(
    "product_showcase.jpg",
    "Woman rotates product, examines features, natural motion",
    duration=8
)
```

**Five Seedance Prompt Formulas:**

1. **UGC selfie-style** — `seedance-2-ugc.md`
2. **Premium product reveal** (no person) — `seedance-2-premium-reveal.md`
3. **Product hero with elemental effects** — `seedance-2-product-hero.md`
4. **Studio lookbook** — `seedance-2-studio-lookbook.md`
5. **Feature walkthrough** — `seedance-2-feature-walkthrough.md`

See `skills/arcads-external-api/prompting/prompt-library/` for full templates.

### Sora 2 (Long-Form)

Up to 20 seconds, text-to-video with optional style reference.

```python
def generate_sora_video(prompt, duration=16, reference_image=None):
    """Generate Sora 2 video (up to 20s)"""
    payload = {
        "prompt": prompt,
        "duration": duration,
        "model": "sora-2"
    }
    
    if reference_image:
        with open(reference_image, "rb") as f:
            payload["referenceImage"] = base64.b64encode(f.read()).decode()
    
    response = requests.post(
        f"{BASE_URL}/v1/sora2/generate",
        headers=headers,
        json=payload
    )
    
    return poll_generation(response.json()["generationId"], "sora2")

# Generate longer narrative scene
video_url = generate_sora_video(
    "Cinematic drone shot over coastal highway at sunset, "
    "car drives along cliff edge, waves crashing below",
    duration=18
)
```

**Sora 2 Remix:**

```python
def remix_sora_video(video_url, new_prompt):
    """Remix existing Sora video with new prompt"""
    payload = {
        "videoUrl": video_url,
        "prompt": new_prompt
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/sora2/remix/video",
        headers=headers,
        json=payload
    )
    
    return poll_generation(response.json()["generationId"], "sora2")
```

### Veo 3.1 (Starting Frame Animation)

Ideal for **still → video** with embedded dialogue.

```python
def generate_veo_video(start_frame_path, prompt, dialogue, duration=8):
    """Animate still image with Veo 3.1 + dialogue"""
    with open(start_frame_path, "rb") as f:
        start_frame = base64.b64encode(f.read()).decode()
    
    payload = {
        "startFrame": start_frame,
        "prompt": prompt,
        "dialogue": dialogue,
        "duration": duration,
        "aspectRatio": "9:16"
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/veo3/generate",
        headers=headers,
        json=payload
    )
    
    return poll_generation(response.json()["generationId"], "veo3")

# Convert UGC still to video with speech
video_url = generate_veo_video(
    "ugc_selfie.jpg",
    "Woman speaks directly to camera, natural hand gestures, "
    "slight head movement, authentic UGC motion",
    dialogue="This is the best skincare product I've ever used",
    duration=8
)
```

**Dialogue Gate:** The API enforces dialogue validation — user must approve the dialogue text separately before Veo generation proceeds.

### Kling 3.0 (B-Roll & Scenes)

**B-Roll Clips:**

```python
def generate_kling_broll(prompt, duration=5):
    """Generate b-roll clip with Kling 3.0"""
    payload = {
        "prompt": prompt,
        "duration": duration
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/b-roll",
        headers=headers,
        json=payload
    )
    
    return poll_generation(response.json()["generationId"], "videos")

# Generate product lifestyle b-roll
broll_url = generate_kling_broll(
    "Coffee being poured into white mug on marble counter, "
    "steam rising, morning sunlight, slow motion",
    duration=6
)
```

**Scene Generation:**

```python
def generate_kling_scene(prompt):
    """Generate environment scene with Kling 3.0"""
    payload = {"prompt": prompt}
    
    response = requests.post(
        f"{BASE_URL}/v1/scene",
        headers=headers,
        json=payload
    )
    
    return poll_generation(response.json()["generationId"], "videos")
```

### Grok Video

```python
def generate_grok_video(prompt, duration=10):
    """Generate video with Grok Video model"""
    payload = {
        "prompt": prompt,
        "duration": duration,
        "model": "grok-video"
    }
    
    response = requests.post(
        f"{BASE_URL}/v2/videos/generate",
        headers=headers,
        json=payload
    )
    
    return poll_generation(response.json()["generationId"])
```

### OmniHuman & Audio-Driven

**OmniHuman Avatar:**

```python
def generate_omnihuman(person_description, script):
    """Generate talking avatar with OmniHuman"""
    payload = {
        "personDescription": person_description,
        "script": script
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/omnihuman",
        headers=headers,
        json=payload
    )
    
    return poll_generation(response.json()["generationId"], "omnihuman")
```

**Audio-Driven Lip-Sync:**

```python
def generate_audio_driven(video_path, audio_path):
    """Lip-sync video to audio file"""
    with open(video_path, "rb") as vf, open(audio_path, "rb") as af:
        video_base64 = base64.b64encode(vf.read()).decode()
        audio_base64 = base64.b64encode(af.read()).decode()
    
    payload = {
        "videoBase64": video_base64,
        "audioBase64": audio_base64
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/audio-driven",
        headers=headers,
        json=payload
    )
    
    return poll_generation(response.json()["generationId"], "audio-driven")
```

## Image Generation

### Nano Banana (Character Creation)

**Create AI Influencer (10-Image Character Sheet):**

```python
def create_ai_influencer(description, output_dir="references/influencers/"):
    """Generate 10-image character sheet for AI influencer"""
    os.makedirs(output_dir, exist_ok=True)
    
    # Step 1: Generate hero front portrait
    hero_payload = {
        "prompt": f"{description}. Direct front view, eye contact, "
                  f"even lighting, neutral expression, high detail.",
        "model": "nano-banana-2",
        "aspectRatio": "1:1"
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/images/generate",
        headers=headers,
        json=hero_payload
    )
    
    hero_image_url = poll_generation(
        response.json()["generationId"], 
        "images"
    )
    
    # Download hero
    hero_path = f"{output_dir}hero.jpg"
    with open(hero_path, "wb") as f:
        f.write(requests.get(hero_image_url).content)
    
    print(f"Hero generated: {hero_path}")
    print("Review hero before continuing...")
    input("Press Enter to generate 9 additional angles...")
    
    # Step 2: Generate 9 additional angles with hero as reference
    with open(hero_path, "rb") as f:
        hero_base64 = base64.b64encode(f.read()).decode()
    
    angles = [
        ("3/4 view left", "three_quarter_left.jpg"),
        ("3/4 view right", "three_quarter_right.jpg"),
        ("Profile left", "profile_left.jpg"),
        ("Profile right", "profile_right.jpg"),
        ("Close-up face", "closeup.jpg"),
        ("Smiling expression", "smiling.jpg"),
        ("Serious expression", "serious.jpg"),
        ("Full body standing", "full_body.jpg"),
        ("Upper body gesturing", "upper_body.jpg")
    ]
    
    for angle_desc, filename in angles:
        payload = {
            "prompt": f"{description}. {angle_desc}. Same person, "
                      f"consistent identity, same lighting style.",
            "model": "nano-banana-2",
            "referenceImages": [hero_base64],
            "aspectRatio": "1:1"
        }
        
        response = requests.post(
            f"{BASE_URL}/v1/images/generate",
            headers=headers,
            json=payload
        )
        
        image_url = poll_generation(
            response.json()["generationId"],
            "images"
        )
        
        # Download
        angle_path = f"{output_dir}{filename}"
        with open(angle_path, "wb") as f:
            f.write(requests.get(image_url).content)
        
        print(f"Generated: {angle_path}")
    
    print(f"\n✓ Character sheet complete: {output_dir}")
    return output_dir

# Create new influencer
create_ai_influencer(
    "22-year-old woman with freckles, hazel eyes, shoulder-length "
    "auburn hair, casual college student aesthetic"
)
```

**UGC Product Selfie Still:**

```python
def generate_ugc_selfie(character_ref, product_ref, prompt):
    """Generate UGC-style product selfie"""
    # Load reference images
    refs = []
    for path in [character_ref, product_ref]:
        with open(path, "rb") as f:
            refs.append(base64.b64encode(f.read()).decode())
    
    # Add UGC aesthetic references
    ugc_aesthetic_dir = "references/aesthetics/ugc-selfie/"
    for aesthetic_file in os.listdir(ugc_aesthetic_dir):
        with open(f"{ugc_aesthetic_dir}{aesthetic_file}", "rb") as f:
            refs.append(base64.b64encode(f.read()).decode())
    
    payload = {
        "prompt": f"{prompt}. iPhone selfie angle, natural bedroom lighting, "
                  f"slight motion blur, realistic skin texture, pores visible, "
                  f"casual messy hair, authentic UGC aesthetic, camera imperfections.",
        "model": "nano-banana-2",
        "referenceImages": refs[:5],  # Max 5 references
        "aspectRatio": "9:16"
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/images/generate",
        headers=headers,
        json=payload
    )
    
    return poll_generation(response.json()["generationId"], "images")

# Generate UGC selfie
image_url = generate_ugc_selfie(
    "references/influencers/sofia/hero.jpg",
    "references/products/skincare_bottle.jpg",
    "Sofia holding skincare product at eye level in bedroom, "
    "smiling at camera, morning golden hour"
)
```

**Model Selection:**

- `nano-banana-2` — Default, fast, good quality
- `nano-banana` — **Nano Banana Pro** (Gemini 3 Pro Image), higher fidelity, tighter identity lock across batches
- `nano-banana-edit` — Inpainting/editing

```python
# Use Nano Banana Pro for critical identity lock
payload = {
    "prompt": prompt,
    "model": "nano-banana",  # Pro model
    "referenceImages": refs
}
```

### ChatGPT Image 2 (Typography & UI)

```python
def generate_chatgpt_image(prompt, aspect_ratio="1:1"):
    """Generate image with ChatGPT Image 2 (gpt-image-2)"""
    payload = {
        "prompt": prompt,
        "model": "gpt-image-2",
        "aspectRatio": aspect_ratio
    }
    
    response = requests.post(
        f"{BASE_URL}/v1/images/generate",
        headers=headers,
        json=payload
    )
    
    return poll_generation(response.json()["generationId"], "images")

# Generate Apple Notes-style ad
image_url = generate_chatgpt_image(
    "iPhone Notes app screenshot. Title: '3 Reasons I Switched'. "
    "Numbered list with check marks. Clean typography, iOS aesthetic, "
    "realistic UI elements, subtle shadows.",
    aspect_ratio="1:1"
)
```

## Static Meta Image Ads (37-Template Library)

Three specialized skills for static image ads:

### chatgpt-image-ad (Typography/UI-Heavy)

Handles: Apple Notes lists, Forbes editorial, Google search mockups, comparison tables, Slack threads, ChatGPT conversations, iMessage screenshots, weather forecast UI.

```python
# Use the installed skill directly
# Example via CLI wrapper:
# python shared/skills/chatgpt-image-ad/generate.py \
#   --template apple-notes \
#   --product "meditation app" \
#   --aspect-ratio 1:1
```

### nano-banana-image-ad (Photoreal/Lifestyle)

Handles: Product photography, lifestyle scenes, multi-reference composites, founder portraits, magazine covers.

```python
# Example via CLI wrapper:
# python shared/skills/nano-banana-image-ad/generate.py \
#   --template magazine-cover \
#   --product "fitness tracker" \
#   --references product.jpg,lifestyle.jpg
```

### image-ad-clone (Reverse-Engineer Existing Ad)

```python
# Clone any ad image into a reusable template
# python shared/skills/image-ad-clone/clone.py \
#   --input competitor_ad.jpg \
#   --backend nano-banana \
#   --validate
```

**Read `shared/skills/image-ad-prompting/OVERVIEW.md` first** for:
- Decision tree (which backend for which template)
- Aspect-ratio compatibility matrix
- Standard generate/clone workflows

Output is image files. Pair with `meta-ad-builder` skill to publish as paused Meta ads.

## Multi-Step Pipelines

### Pixar-Style 3D Animated Ad

8-beat story arc, anthropomorphized mascot, ChatGPT Image 2 storyboard → Seedance 2.0 i2v → ffmpeg stitch.

```bash
# Run Pixar pipeline
./shared/skills/pixar-style-ad/scripts/generate.sh \
  --product "coffee maker" \
  --mascot "cheerful coffee bean character" \
  --duration 60
```

**Python API Pattern:**

```python
def generate_pixar_ad(product, mascot_description, beats=8):
    """Generate Pixar-style animated ad"""
    
    # 1. Lock cast sheet (ChatGPT Image 2)
    cast_prompt = f"3D Pixar character sheet. {mascot_description}. " \
                  f"Front, 3/4, side view. Consistent turnaround."
    
    cast_payload = {
        "prompt": cast_prompt,
        "model": "gpt-image-2",
        "aspectRatio": "16:9"
    }
    
    cast_response = requests.post(
        f"{BASE_URL}/v1/images/generate",
        headers=headers,
        json=cast_payload
    )
    cast_image_url = poll_generation(
        cast_response.json()["generationId"],
        "images"
    )
    
    # Download cast sheet
    cast_path = "output/pixar/cast.jpg"
    with open(cast_path, "wb") as f:
        f.write(requests.get(cast_image_url).content)
    
    with open(cast_path, "rb") as f:
        cast_base64 = base64.b64encode(f.read()).decode()
    
    # 2. Generate 8 storyboard stills (sequential with prior frame ref)
    story_beats = [
        "Beat 1: Character wakes up tired in kitchen",
        "Beat 2: Character sees product on counter",
        "Beat 3: Character's eyes light up with excitement",
        "Beat 4: Character uses product, magical sparkles",
        "Beat 5: Character drinks coffee, energized glow",
        "Beat 6: Character dances happily around kitchen",
        "Beat 7: Character shows product to camera proudly",
        "Beat 8: Product hero shot with logo reveal"
    ]
    
    stills = []
    prior_frame = None
    
    for i, beat in enumerate(story_beats):
        refs = [cast_base64]
        if prior_frame:
            refs.append(prior_frame)
        
        still_prompt = f"3D Pixar animation style. {beat}. " \
                       f"Vibrant colors, soft lighting, emotional expression. " \
                       f"Same character as cast sheet, consistent identity."
        
        payload = {
            "prompt": still_prompt,
            "model": "gpt-image-2",
            "referenceImages": refs[:5],
            "aspectRatio": "16:9"
        }
        
        response = requests.post(
            f"{BASE_URL}/v1/images/generate",
            headers=headers,
            json=payload
        )
        
        still_url = poll_generation(
            response.json()["generationId"],
            "images"
        )
        
        still_path = f"output/pixar/beat_{i+1}.jpg"
        with open(still_path, "wb") as f:
            content = requests.get(still_url).content
            f.write(content)
        
        stills.append(still_path)
        
        # Update prior frame for next beat
        with open(still_path, "rb") as f:
            prior_frame = base64.b64encode(f.read()).decode()
        
        print(f"Generated beat {i+1}/8")
    
    # 3. Animate each still with Seedance 2.0 (i2v)
    clips = []
    for i, still_path in enumerate(stills):
        with open(still_path, "rb") as f:
            start_frame = base64.b64encode(f.read()).decode()
        
        video_prompt = f"{story_beats[i]}. Smooth character motion, " \
                       f"natural animation, Pixar-quality movement."
        
        payload = {
            "prompt": video_prompt,
            "startFrame": start_frame,
            "duration": 7,  # 7s per beat
            "aspectRatio": "16:9",
            "model": "seedance-2"
        }
        
        response = requests.post(
            f"{BASE_URL}/v1/videos/generate",
            headers=headers,
            json=payload
        )
        
        video_url = poll_generation(response.json()["generationId"])
        
        clip_path = f"output/pixar/clip_{i+1}.mp4"
        with open(clip_path, "wb") as f:
            f.write(requests.get(video_url).content)
        
        clips.append(clip_path)
        print(f"Animated beat {i+1}/8")
    
    # 4. Stitch clips with ffmpeg
    import subprocess
    
    concat_file = "output/pixar/concat.txt"
    with open(concat_file, "w") as f:
        for clip in clips:
            f.write(f"file '{clip}'\n")
    
    final_output = "output/pixar/final.mp4"
    subprocess.run([
        "ffmpeg", "-f", "concat", "-safe", "0",
        "-i", concat_file,
        "-c", "copy",
        final_output
    ])
    
    print(f"\n✓ Pixar ad complete: {final_output}")
    return final_output

# Generate
generate_pixar_ad(
    "premium coffee maker",
    "Anthropomorphic coffee bean with big eyes and tiny arms"
)
```

See `shared/skills/pixar-style-ad/prompting/guide.md` for full workflow.

### Claymation/Aardman-Style Ad

Same 8-beat arc with clay textures, stop-motion judder, narrator-driven.

```bash
./shared/skills/claymation-ad/scripts/generate.sh \
  --product "organic snack bars" \
  --character "sculpted clay fox" \
  --duration 90
```

**Stop-Motion Effect:**

```bash
# Add judder in post
ffmpeg -i final.mp4 \
  -vf "fps=12,fps=24" \
  -c:v libx264 \
  output_claymation_judder.mp4
```

See `shared/skills/claymation-ad/prompting/guide.md`.

### Burn Captions onto Video

Post-processing step for any video source (Pixar, claymation, UGC, b-roll).

```bash
# Transcribe with Whisper + render with HyperFrames
python shared/skills/caption-video/burn_captions.py \
  --video output/pixar/final.mp4 \
  --output output/pixar/final_captioned.mp4
```

**Python Implementation:**

```python
import subprocess
import json

def burn_captions(video_path, output_path):
    """Add captions to video using Whisper + HyperFrames"""
    
    # 1. Extract audio
    audio_path = "temp_audio.wav"
    subprocess.run([
        "ffmpeg", "-i", video_path,
        "-vn", "-acodec", "pcm_s16le",
        audio_path
    ])
    
    # 2. Transcribe with Whisper
    import whisper
    model = whisper.load_model("medium.en")
    result = model.transcribe(audio_path, word_timestamps=True)
    
    # 3. Group words into reading phrases
    phrases = []
    current_phrase = {"words": [], "start": 0, "end": 0}
    
    for segment in result["segments"]:
        for word in segment["words"]:
            current_phrase["words"].append(word["word"])
            current_phrase["end"] = word["end"]
            
            if not current_phrase["start"]:
                current_phrase["start"] = word["start"]
            
            # Group ~3-5 words per caption
            if len(current_phrase["words"]) >= 4:
                phrases.append({
                    "text": " ".join(current_phrase["words"]),
                    "start": current_phrase["start"],
                    "end": current_phrase["end"]
                })
                current_phrase = {"words": [], "start": 0, "end": 0}
    
    # Add remaining words
    if current_phrase["words"]:
        phrases.append({
            "text": " ".join(current_phrase["words"]),
            "start": current_phrase["start"],
            "end": current_phrase["end"]
        })
    
    # 4. Create HyperFrames caption JSON
    caption_json = []
    for phrase in phrases:
        caption_json.append({
            "text": phrase["text"],
            "start": phrase["start"],
            "end": phrase["end"],
            "style": {
                "fontSize": 48,
                "fontWeight": "bold",
                "color": "white",
                "backgroundColor": "black",
                "padding": 10,
                "position": "bottom"
            }
        })
    
    captions_file = "temp_captions.json"
    with open(captions_file, "w") as f:
        json.dump(caption_json, f)
    
    # 5. Render captions with HyperFrames
    subprocess.run([
        "npx", "hyperframes",
        "--video", video_path,
        "--captions", captions_file,
        "--output", output_path
    ])
    
    # Cleanup
    os.remove(audio_path)
    os.remove(captions_file)
    
    print(f"✓ Captions burned: {output_path}")
    return output_path
```

### YouTube Thumbnails (5 CTR Formulas)

```python
def generate_youtube_thumbnail(face_refs, product_ref, formula="peace-sign"):
    """Generate YouTube thumbnail with CTR formula"""
    
    # Load 5+ face references for likeness lock
    refs = []
    for face_path in face_refs:
        with open(face_path, "rb") as f:
            refs.append(base64.b64encode(f.read()).decode())
    
    with open(product_ref
