---
name: arcads-ai-video-agent
description: Create AI marketing videos and images using Arcads API with Seedance 2.0, Sora 2, Veo 3.1, Kling 3.0, Nano Banana, and 37 static Meta ad templates
triggers:
  - generate an arcads video
  - create ai marketing video with seedance
  - make a nano banana image ad
  - generate ugc video with arcads
  - create pixar style animated ad
  - make meta image ad with arcads
  - use arcads api for video generation
  - create ai influencer character sheet
---

# Arcads AI Video Agent

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

This skill enables AI agents to create marketing videos and images using the [Arcads](https://arcads.ai/?via=claude-code) platform. It supports the full creative stack: **Seedance 2.0** (flagship video), **Sora 2**, **Veo 3.1**, **Kling 3.0**, **Grok Video**, **Nano Banana 2/Pro/Edit**, **ChatGPT Image 2**, **OmniHuman**, and **Audio-driven** models, plus 37 validated static Meta image-ad templates and multi-step pipelines for Pixar-style and claymation animated ads.

## Prerequisites

- Python 3.10+
- Arcads API key from [app.arcads.ai/settings/api](https://app.arcads.ai/settings/api)
- Optional tools for advanced workflows:
  - `ffmpeg` (video stitching, chroma-key)
  - `jq` (JSON parsing in bash scripts)
  - Node.js + `npx hyperframes` (caption burn-in)
  - `openai-whisper` (transcription: `pip install openai-whisper`)

## Installation

```bash
# Clone the repository
git clone https://github.com/krusemediallc/arcads-claude-code.git
cd arcads-claude-code

# Run setup script
./scripts/setup.sh
```

The setup script will:
1. Prompt for your Arcads API key
2. Create `.env` file with `ARCADS_API_KEY=your_key_here`
3. Verify API connection
4. Create `MASTER_CONTEXT.md` workspace file

**Manual setup** (if you skip the script):

```bash
# Create .env file
echo "ARCADS_API_KEY=your_api_key_here" > .env
```

## Core API Patterns

All Arcads API calls follow this pattern:

```python
import os
import requests
import time
from dotenv import load_dotenv

load_dotenv()

BASE_URL = "https://api.arcads.ai"
API_KEY = os.getenv("ARCADS_API_KEY")

headers = {
    "Authorization": f"Bearer {API_KEY}",
    "Content-Type": "application/json"
}
```

### Standard Generation Flow

1. **Submit job** → get `jobId`
2. **Poll status** until `completed` or `failed`
3. **Download result**

```python
def submit_job(endpoint, payload):
    """Submit a generation job to Arcads API"""
    response = requests.post(
        f"{BASE_URL}{endpoint}",
        headers=headers,
        json=payload
    )
    response.raise_for_status()
    return response.json()["jobId"]

def poll_status(job_id, timeout=600, interval=10):
    """Poll job status until complete"""
    start_time = time.time()
    while time.time() - start_time < timeout:
        response = requests.get(
            f"{BASE_URL}/v1/jobs/{job_id}",
            headers=headers
        )
        data = response.json()
        status = data["status"]
        
        if status == "completed":
            return data["result"]
        elif status == "failed":
            raise Exception(f"Job failed: {data.get('error')}")
        
        time.sleep(interval)
    
    raise TimeoutError(f"Job {job_id} timed out after {timeout}s")

def download_file(url, output_path):
    """Download generated asset"""
    response = requests.get(url, stream=True)
    response.raise_for_status()
    with open(output_path, 'wb') as f:
        for chunk in response.iter_content(chunk_size=8192):
            f.write(chunk)
```

## Video Generation

### Seedance 2.0 (Flagship Model)

**Best for:** UGC videos, product reveals, hero shots, lookbooks, feature demos (4-15s)

```python
def generate_seedance_video(prompt, duration=10, style="ugc"):
    """Generate Seedance 2.0 video with prompt engineering"""
    
    # UGC formula (9-layer structure)
    if style == "ugc":
        full_prompt = f"""
[PERSON] Woman, late 20s, natural makeup, casual kitchen setting
[OPENER] Direct camera eye contact, authentic energy, "Hey guys!"
[HOOK] Attention-grabbing statement about {prompt}
[PROBLEM] Relatable pain point, conversational tone
[SOLUTION] Product introduction, natural hand gestures
[DEMO] Show product in use, realistic interaction
[BENEFIT] Key value prop, maintains eye contact
[SOCIAL_PROOF] Brief testimonial feel
[CTA] Natural call to action, warm energy
[STYLE] iPhone selfie aesthetic, natural lighting, slight camera shake
"""
    else:
        full_prompt = prompt
    
    payload = {
        "prompt": full_prompt,
        "duration": duration,
        "aspectRatio": "9:16"  # or "16:9", "1:1"
    }
    
    job_id = submit_job("/v2/videos/generate", payload)
    print(f"Seedance job submitted: {job_id}")
    
    result = poll_status(job_id)
    video_url = result["videoUrl"]
    
    download_file(video_url, f"output/seedance_{job_id}.mp4")
    return video_url
```

**Example usage:**

```python
# UGC product review
video = generate_seedance_video(
    "skin serum that reduced dark circles in 2 weeks",
    duration=12,
    style="ugc"
)

# Premium product reveal
payload = {
    "prompt": """
[SCENE] Dark void, single spotlight
[PRODUCT] Luxury perfume bottle materializes
[MOTION] Slow 360° rotation, golden light rays
[TEXT] Overlay: "Crafted for those who dare"
[AESTHETIC] High-contrast, cinematic, no human presence
""",
    "duration": 8,
    "aspectRatio": "9:16"
}
job_id = submit_job("/v2/videos/generate", payload)
```

### Veo 3.1 (Image-to-Video with Dialogue)

**Best for:** Animating Nano Banana stills into talking UGC videos

```python
def animate_with_veo(image_path, dialogue, duration=8):
    """Animate a still image with Veo 3.1 + dialogue"""
    
    import base64
    
    # Load and encode starting frame
    with open(image_path, "rb") as f:
        image_b64 = base64.b64encode(f.read()).decode()
    
    payload = {
        "startFrame": image_b64,
        "prompt": f"""
Natural human motion, authentic energy, person speaks:
"{dialogue}"
Maintain character likeness from starting frame.
iPhone selfie aesthetic, slight head movement, natural eye contact.
""",
        "dialogue": dialogue,  # MANDATORY for dialogue videos
        "duration": duration,
        "aspectRatio": "9:16"
    }
    
    job_id = submit_job("/v1/veo3-1/video", payload)
    result = poll_status(job_id, timeout=900)  # Veo takes longer
    
    download_file(result["videoUrl"], f"output/veo_{job_id}.mp4")
    return result["videoUrl"]
```

### Sora 2 (Text-to-Video, Longer Durations)

**Best for:** Cinematic scenes, B-roll, up to 20s

```python
def generate_sora_video(prompt, duration=16):
    """Generate Sora 2 video (supports longer durations)"""
    
    payload = {
        "prompt": prompt,
        "duration": duration,  # Auto-calculated from word count if omitted
        "aspectRatio": "16:9"
    }
    
    # Optional: add style reference image
    # payload["styleReference"] = base64_encoded_image
    
    job_id = submit_job("/v1/sora2/video", payload)
    result = poll_status(job_id, timeout=1200)
    
    download_file(result["videoUrl"], f"output/sora_{job_id}.mp4")
    return result["videoUrl"]
```

### Kling 3.0 (B-Roll & Scenes)

**Best for:** Scene generation, environmental b-roll

```python
def generate_broll(scene_description):
    """Generate b-roll clip with Kling 3.0"""
    
    payload = {
        "prompt": scene_description,
        "duration": 5
    }
    
    job_id = submit_job("/v1/b-roll", payload)
    result = poll_status(job_id)
    
    download_file(result["videoUrl"], f"output/broll_{job_id}.mp4")
    return result["videoUrl"]

# Example
generate_broll("Golden hour beach waves, slow motion, cinematic")
```

## Image Generation

### Nano Banana (Character Creation & Product Stills)

**Best for:** AI influencers, UGC stills, photoreal product shots

```python
def create_nano_banana_image(prompt, reference_images=None, model="nano-banana-2"):
    """
    Generate image with Nano Banana
    model options: "nano-banana-2" (default), "nano-banana" (Pro), "nano-banana-edit"
    """
    
    payload = {
        "prompt": prompt,
        "model": model,
        "aspectRatio": "9:16"
    }
    
    # Add reference images for character consistency
    if reference_images:
        import base64
        refs = []
        for img_path in reference_images:
            with open(img_path, "rb") as f:
                refs.append(base64.b64encode(f.read()).decode())
        payload["referenceImages"] = refs
    
    job_id = submit_job("/v1/nano-banana/image", payload)
    result = poll_status(job_id)
    
    download_file(result["imageUrl"], f"output/nano_{job_id}.png")
    return result["imageUrl"]
```

**Example: Create AI Influencer (10-image character sheet)**

```python
def create_ai_influencer(description):
    """Generate 10-angle character sheet for AI influencer"""
    
    # Step 1: Generate hero front portrait
    hero_prompt = f"""
{description}
Front-facing portrait, natural expression, golden hour lighting.
Photoreal skin texture, freckles, pores visible.
Soft focus background, kitchen setting.
"""
    
    hero_url = create_nano_banana_image(hero_prompt)
    print(f"Hero portrait: {hero_url}")
    print("Review and approve hero before generating remaining 9 angles.")
    
    # Step 2: Generate 9 additional angles using hero as reference
    angles = [
        "3/4 view left, slight smile",
        "3/4 view right, natural expression",
        "Profile left, looking away",
        "Profile right, looking forward",
        "Close-up, eyes focused on camera",
        "Full body, standing casual pose",
        "Laughing, animated expression",
        "Serious expression, direct gaze",
        "Lifestyle shot, holding coffee mug"
    ]
    
    results = []
    for angle in angles:
        prompt = f"{description}\n{angle}\nMaintain exact character likeness."
        url = create_nano_banana_image(
            prompt,
            reference_images=["output/hero_portrait.png"],
            model="nano-banana"  # Use Pro for tighter identity lock
        )
        results.append(url)
    
    return results
```

### ChatGPT Image 2 (Typography & UI-Heavy Ads)

**Best for:** Apple Notes lists, fake Slack threads, editorial layouts, comparison tables

```python
def generate_chatgpt_image(prompt, aspect_ratio="1:1"):
    """Generate image with ChatGPT Image 2 (gpt-image-2)"""
    
    payload = {
        "prompt": prompt,
        "model": "gpt-image-2",
        "aspectRatio": aspect_ratio  # "1:1", "4:5", "16:9"
    }
    
    job_id = submit_job("/v1/image/generate", payload)
    result = poll_status(job_id)
    
    download_file(result["imageUrl"], f"output/chatgpt_{job_id}.png")
    return result["imageUrl"]
```

## Static Meta Image Ad Templates (37-Template Library)

The repo includes **37 validated prompt templates** for static Meta image ads. Use the specialized skills:

```python
# Apple Notes-style list ad
def generate_apple_notes_ad(product, benefits):
    """Generate Apple Notes-style ad (ChatGPT Image 2)"""
    
    prompt = f"""
iPhone Notes app interface, cream background.
Title: "why i switched to {product}"
Bulleted list:
{chr(10).join(f'• {b}' for b in benefits)}

Footer: handwritten-style signature.
Clean iOS typography, authentic spacing, no visible phone edges.
"""
    
    return generate_chatgpt_image(prompt, aspect_ratio="4:5")

# Photoreal UGC selfie ad
def generate_ugc_selfie_ad(influencer_ref, product_ref):
    """Generate UGC selfie with product (Nano Banana)"""
    
    prompt = """
iPhone selfie, natural bedroom lighting.
[influencer] holding [product], casual smile.
Authentic skin texture, slight motion blur.
Visible pores, flyaway hairs, iPhone camera imperfections.
Product visible and recognizable, natural hand position.
"""
    
    return create_nano_banana_image(
        prompt,
        reference_images=[influencer_ref, product_ref],
        model="nano-banana-2"
    )
```

**Template categories** (see `shared/skills/image-ad-prompting/library/` for full list):

- **UI/App Mimicry:** Apple Notes, Slack threads, iMessage screenshots, ChatGPT conversations, Google search results
- **Editorial:** Forbes hero, magazine cover, billboard, museum exhibit
- **Comparison:** Feature tables, before/after splits, versus cards
- **Lifestyle:** Sticky-note flatlays, founder letter, dating-app card, weather forecast UI

## Multi-Step Animated Ad Pipelines

### Pixar-Style 3D Animated Ad

**8-beat story arc** → ChatGPT Image 2 storyboard → Seedance 2.0 i2v per beat → ffmpeg stitch + captions

```python
def generate_pixar_ad(product, story_arc):
    """
    Generate Pixar-style animated ad
    story_arc: list of 8 beat descriptions
    """
    
    # Step 1: Lock cast sheet (character designs)
    cast_prompt = """
3D Pixar style character sheet.
Anthropomorphized [product mascot], friendly expression.
Multiple angles: front, 3/4, profile, back.
Consistent lighting, white background.
"""
    
    cast_url = generate_chatgpt_image(cast_prompt)
    print(f"Cast sheet: {cast_url}")
    
    # Step 2: Generate storyboard stills (8 beats)
    stills = []
    prev_frame = None
    
    for i, beat in enumerate(story_arc):
        prompt = f"""
3D Pixar animation style frame.
Beat {i+1}: {beat}
Maintain character consistency from cast sheet.
Cinematic lighting, shallow depth of field.
"""
        
        refs = [cast_url]
        if prev_frame:
            refs.append(prev_frame)  # Max 5 references total
        
        # Use ChatGPT Image 2 for sequential stills
        job_id = submit_job("/v1/image/generate", {
            "prompt": prompt,
            "model": "gpt-image-2",
            "referenceImages": refs,
            "aspectRatio": "16:9"
        })
        
        result = poll_status(job_id)
        still_path = f"output/beat_{i+1}.png"
        download_file(result["imageUrl"], still_path)
        
        stills.append(still_path)
        prev_frame = result["imageUrl"]
    
    # Step 3: Animate each still with Seedance 2.0 i2v
    videos = []
    for i, still in enumerate(stills):
        video = animate_still_with_seedance(still, story_arc[i])
        videos.append(video)
    
    # Step 4: Stitch with ffmpeg
    stitch_videos(videos, "output/pixar_ad_final.mp4")
    
    # Step 5: Burn captions (optional)
    # burn_captions("output/pixar_ad_final.mp4", "output/pixar_ad_captioned.mp4")
    
    return "output/pixar_ad_final.mp4"

def animate_still_with_seedance(image_path, motion_description):
    """Animate a still image with Seedance 2.0 i2v"""
    
    import base64
    with open(image_path, "rb") as f:
        img_b64 = base64.b64encode(f.read()).decode()
    
    payload = {
        "startFrame": img_b64,
        "prompt": f"{motion_description}\nMaintain scene composition, add subtle motion.",
        "duration": 3
    }
    
    job_id = submit_job("/v2/videos/generate", payload)
    result = poll_status(job_id)
    
    output_path = f"output/animated_{int(time.time())}.mp4"
    download_file(result["videoUrl"], output_path)
    return output_path

def stitch_videos(video_paths, output_path):
    """Stitch multiple videos with ffmpeg"""
    
    import subprocess
    
    # Create concat file
    with open("concat_list.txt", "w") as f:
        for path in video_paths:
            f.write(f"file '{path}'\n")
    
    subprocess.run([
        "ffmpeg", "-f", "concat", "-safe", "0",
        "-i", "concat_list.txt",
        "-c", "copy", output_path
    ], check=True)
```

### Caption Burn-In Workflow

```python
def burn_captions(video_path, output_path):
    """Transcribe and burn captions onto video"""
    
    import subprocess
    import whisper
    
    # Step 1: Transcribe with Whisper
    model = whisper.load_model("medium.en")
    result = model.transcribe(video_path, word_timestamps=True)
    
    # Step 2: Group word-level timestamps into phrases
    phrases = group_into_phrases(result["segments"])
    
    # Step 3: Generate caption JSON for HyperFrames
    captions_json = {
        "captions": [
            {
                "start": p["start"],
                "end": p["end"],
                "text": p["text"]
            }
            for p in phrases
        ]
    }
    
    with open("captions.json", "w") as f:
        import json
        json.dump(captions_json, f)
    
    # Step 4: Render with HyperFrames
    subprocess.run([
        "npx", "hyperframes",
        "--input", video_path,
        "--captions", "captions.json",
        "--output", output_path,
        "--style", "modern"  # or "classic", "minimal"
    ], check=True)

def group_into_phrases(segments, max_words=5):
    """Group word-level timestamps into reading phrases"""
    
    phrases = []
    current_phrase = {"words": [], "start": 0, "end": 0}
    
    for segment in segments:
        for word_info in segment.get("words", []):
            current_phrase["words"].append(word_info["word"])
            
            if not current_phrase["start"]:
                current_phrase["start"] = word_info["start"]
            
            current_phrase["end"] = word_info["end"]
            
            if len(current_phrase["words"]) >= max_words:
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
    
    return phrases
```

## Configuration

All configuration is in `.env`:

```bash
ARCADS_API_KEY=your_api_key_here

# Optional: default models
DEFAULT_VIDEO_MODEL=seedance-2
DEFAULT_IMAGE_MODEL=nano-banana-2

# Optional: output directories
OUTPUT_DIR=output
REFERENCES_DIR=references

# Optional: API timeouts
JOB_TIMEOUT=600
POLL_INTERVAL=10
```

## Common Workflows

### Full UGC Video Workflow (Still → Video)

```python
def create_ugc_video_from_scratch(influencer_desc, product_ref, script):
    """Complete UGC workflow: character → still → video"""
    
    # Step 1: Create AI influencer if needed
    influencer_refs = create_ai_influencer(influencer_desc)
    hero_ref = influencer_refs[0]
    
    # Step 2: Generate UGC still with product
    still_prompt = f"""
{influencer_desc}
iPhone selfie, bedroom setting, natural lighting.
Holding {product_ref}, casual smile, looking at camera.
Authentic skin texture, slight motion blur.
"""
    
    still_url = create_nano_banana_image(
        still_prompt,
        reference_images=[hero_ref, product_ref]
    )
    
    download_file(still_url, "temp_still.png")
    
    # Step 3: Animate with Veo 3.1 + dialogue
    video_url = animate_with_veo("temp_still.png", script, duration=12)
    
    return video_url
```

### Batch Video Generation

```python
def batch_generate_videos(prompts, model="seedance-2"):
    """Submit multiple video jobs in parallel"""
    
    job_ids = []
    
    # Submit all jobs
    for prompt in prompts:
        payload = {
            "prompt": prompt,
            "duration": 10,
            "aspectRatio": "9:16"
        }
        job_id = submit_job("/v2/videos/generate", payload)
        job_ids.append(job_id)
        print(f"Submitted: {job_id}")
    
    # Poll all jobs
    results = []
    for job_id in job_ids:
        try:
            result = poll_status(job_id, timeout=900)
            video_path = f"output/batch_{job_id}.mp4"
            download_file(result["videoUrl"], video_path)
            results.append(video_path)
        except Exception as e:
            print(f"Job {job_id} failed: {e}")
    
    return results
```

### Cost Estimation

```python
def estimate_cost(model, duration=10):
    """Estimate generation cost before submitting"""
    
    costs = {
        "seedance-2": 0.10 * (duration / 10),  # $0.10 per 10s
        "veo-3-1": 0.15 * (duration / 10),
        "sora-2": 0.20 * (duration / 10),
        "kling-3": 0.08 * (duration / 10),
        "nano-banana-2": 0.05,  # per image
        "nano-banana": 0.08,  # Pro
        "gpt-image-2": 0.04
    }
    
    return costs.get(model, 0.10)

# Confirm before generating
cost = estimate_cost("seedance-2", duration=12)
print(f"Estimated cost: ${cost:.2f}")
response = input("Proceed? (y/n): ")
if response.lower() == "y":
    generate_seedance_video(prompt, duration=12)
```

## Troubleshooting

### API Key Issues

```python
def verify_api_connection():
    """Test API key validity"""
    
    try:
        response = requests.get(
            f"{BASE_URL}/v1/account",
            headers=headers
        )
        response.raise_for_status()
        print("✓ API connection successful")
        print(f"Account: {response.json()['email']}")
        return True
    except requests.exceptions.HTTPError as e:
        if e.response.status_code == 401:
            print("✗ Invalid API key")
            print("Get your key: https://app.arcads.ai/settings/api")
        else:
            print(f"✗ API error: {e}")
        return False
```

### Job Timeout Issues

If jobs are timing out (Veo/Sora can take 15+ minutes):

```python
# Increase timeout for slower models
result = poll_status(job_id, timeout=1800, interval=30)  # 30min, check every 30s
```

### Character Consistency Issues

For better character lock across multiple images:

```python
# Use Nano Banana Pro instead of default
create_nano_banana_image(
    prompt,
    reference_images=[hero_ref],
    model="nano-banana"  # Pro model (Gemini 3 Pro Image)
)

# Provide 3-5 reference images from different angles
refs = [
    "references/influencers/sofia_front.png",
    "references/influencers/sofia_3_4.png",
    "references/influencers/sofia_profile.png"
]
```

### Video Quality Issues

If Seedance videos lack motion or feel static:

```python
# Add explicit motion cues to prompt
prompt = """
[SCENE] Woman in kitchen, dynamic camera movement
[MOTION] Head turns toward camera, hand gestures, product raise
[ACTION] Walks forward 2 steps, smiles broadly
[ENERGY] Animated, enthusiastic, constant subtle movement
"""
```

### ffmpeg Not Found

```bash
# macOS
brew install ffmpeg

# Ubuntu/Debian
sudo apt install ffmpeg

# Verify installation
ffmpeg -version
```

## Advanced: Publishing to Meta Ads

The repo includes a `meta-ad-builder` skill for publishing generated images as paused Meta ads:

```python
# Install dependencies first
# pip install -r shared/skills/meta-ad-builder/scripts/requirements.txt

def publish_to_meta(image_path, ad_account_id, campaign_name):
    """Publish generated image as paused Meta ad"""
    
    import subprocess
    
    # Set required env vars
    os.environ["META_ACCESS_TOKEN"] = os.getenv("META_ACCESS_TOKEN")
    os.environ["AD_ACCOUNT_ID"] = ad_account_id
    
    # Run meta-ad-builder script
    subprocess.run([
        "python",
        "shared/skills/meta-ad-builder/scripts/create_ad.py",
        "--image", image_path,
        "--campaign", campaign_name,
        "--status", "PAUSED"
    ], check=True)
```

Requires `META_ACCESS_TOKEN` and `AD_ACCOUNT_ID` in `.env`.

## File Organization

The agent auto-organizes outputs:

```
project/
├── output/                    # Generated assets
│   ├── seedance_*.mp4
│   ├── nano_*.png
│   └── pixar_ad_final.mp4
├── references/                # Reference libraries
│   ├── influencers/          # AI character sheets
│   ├── products/             # Product photos
│   └── aesthetics/           # Style references
├── .env                       # API keys (gitignored)
└── MASTER_CONTEXT.md         # Project workspace file
```

## Prompt Engineering Tips

### Seedance 2.0 UGC Formula

Use the 9-layer structure for authentic UGC:

```
[PERSON] demographics + appearance
[OPENER] eye contact + greeting
[HOOK] attention-grabbing statement
[PROBLEM] relatable pain point
[SOLUTION] product intro
[DEMO] product in action
[BENEFIT] key value prop
[SOCIAL_PROOF] testimonial feel
[CTA] natural call to action
[STYLE] iPhone aesthetic + lighting
```

### Nano Banana Character Consistency

For AI influencers that look the same across 100+ images:

1. Generate hero portrait with detailed description
2. Use hero as `referenceImages[0]` for all subsequent generations
3. Use **Nano Banana Pro** (`model="nano-banana"`) for tighter lock
4. Include "maintain exact character likeness" in every prompt
5. Lock lighting/background when possible (e.g., "golden hour kitchen" across all shots)

### Veo 3.1 Dialogue Best Practices

- **Always include** `dialogue` field (mandatory for talking videos)
- Keep dialogue under 20 words for 8s clips (~2.5 words/sec)
- Prompt for "natural head movement, slight pauses, eye contact breaks"
- Starting frame should show person's face clearly (not profile/back)

## API Endpoints Reference

```python
# Video generation
"/v2/videos/generate"           # Seedance 2.0, Sora 2, Grok Video
"/v1/veo3-1/video"              # Veo 3.1 with startFrame + dialogue
"/v1/sora2/video"               # Sora 2 text-to-video
"/v1/sora2/remix/video"         # Sora 2 remix
"/v1/b-roll"                    # Kling 3.0 b-roll
"/v1/scene"                     # Kling 3.0 scene
"/v1/omnihuman"                 # OmniHuman avatar
"/v1/audio-driven"              # Audio-driven lip-sync

# Image generation
"/v1/nano-banana/image"         # Nano Banana 2/Pro/Edit
"/v1/image/generate"            # ChatGPT Image 2

# Job management
"/v1/jobs/{jobId}"              # Poll job status
"/v1/account"                   # Verify API key
```

## Cost Management

Always confirm costs before batch operations:

```python
def confirm_batch_cost(num_videos, model="seedance-2", duration=10):
    """Confirm cost before batch generation
