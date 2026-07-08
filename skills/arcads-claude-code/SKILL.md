---
name: arcads-claude-code
description: Create AI marketing videos and images using Arcads API — Seedance, Sora, Veo, Kling, Nano Banana, ChatGPT Image, and multi-step ad pipelines
triggers:
  - "generate an AI video ad"
  - "create a UGC video with Seedance"
  - "make a Nano Banana product image"
  - "build a Pixar-style animated ad"
  - "create static Meta image ads"
  - "animate this still with Veo"
  - "generate YouTube thumbnails"
  - "make a claymation ad campaign"
---

# arcads-claude-code

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## What it does

`arcads-claude-code` is a comprehensive skill pack for generating AI marketing videos and images through the Arcads API. It supports the full Arcads creative stack:

- **Video models**: Seedance 2.0, Sora 2, Veo 3.1, Kling 3.0, Grok Video, OmniHuman, Audio-driven
- **Image models**: Nano Banana 2/Pro/Edit, ChatGPT Image 2
- **Static ad library**: 37 validated Meta image-ad templates
- **Multi-step pipelines**: Pixar-style animated ads, claymation ads, YouTube thumbnails, caption workflows

Built for AI agents in Claude Code and Cursor to handle API calls, polling, prompt engineering, file organization, and cost confirmation.

## Installation

### 1. Clone and setup

```bash
git clone https://github.com/krusemediallc/arcads-claude-code.git
cd arcads-claude-code
./scripts/setup.sh
```

The setup script will:
- Prompt for your Arcads API key (get it at [app.arcads.ai/settings/api](https://app.arcads.ai/settings/api))
- Save it to `.env` (never committed)
- Verify API connection
- Create `MASTER_CONTEXT.md` workspace file

### 2. Install dependencies (optional, based on workflows)

Basic video/image generation requires only **Python 3.10+**. Multi-step pipelines need:

```bash
# macOS
brew install ffmpeg jq node

# Python packages for specific workflows
pip install openai-whisper  # Caption transcription
pip install -r shared/skills/meta-ad-builder/scripts/requirements.txt  # Meta publishing
```

Linux:
```bash
apt install ffmpeg jq nodejs python3 python3-pip
```

### 3. Environment variables

`.env` file structure:
```bash
ARCADS_API_KEY=your_api_key_here
```

Never hardcode keys — always use `os.environ['ARCADS_API_KEY']`.

## Core API patterns

### Authentication

All requests require `Authorization: Bearer $ARCADS_API_KEY` header.

```python
import os
import requests

BASE_URL = "https://api.arcads.ai"
headers = {
    "Authorization": f"Bearer {os.environ['ARCADS_API_KEY']}",
    "Content-Type": "application/json"
}
```

### Polling workflow

Most video/image generation is asynchronous:
1. POST to generate endpoint → receive `jobId`
2. Poll GET `/v1/job/{jobId}` until `status: "completed"`
3. Extract `outputUrl` or `imageUrl`

```python
import time

def poll_job(job_id, timeout=600):
    """Poll Arcads job until completion."""
    start = time.time()
    while time.time() - start < timeout:
        resp = requests.get(
            f"{BASE_URL}/v1/job/{job_id}",
            headers=headers
        )
        data = resp.json()
        
        if data['status'] == 'completed':
            return data
        elif data['status'] == 'failed':
            raise Exception(f"Job failed: {data.get('error')}")
        
        time.sleep(5)
    
    raise TimeoutError(f"Job {job_id} timeout after {timeout}s")
```

## Video generation

### Seedance 2.0 (flagship model)

**Key features**: 4–15s clips, native audio, image-to-video, video-to-video, reference images, multiple shot styles.

#### Text-to-video with dialogue

```python
def generate_seedance_ugc(prompt, duration=12):
    """Generate UGC-style Seedance video."""
    payload = {
        "prompt": prompt,
        "duration": duration,
        "model": "seedance-2.0",
        "aspectRatio": "9:16",
        "dialogue": "I stopped buying [competitor] after I found this"
    }
    
    resp = requests.post(
        f"{BASE_URL}/v1/seedance/generate",
        headers=headers,
        json=payload
    )
    job_id = resp.json()['jobId']
    result = poll_job(job_id)
    return result['outputUrl']

# Usage
video_url = generate_seedance_ugc(
    "Woman in kitchen, natural lighting, holding product bottle, iPhone selfie aesthetic"
)
```

#### Image-to-video (product reveal)

```python
import base64

def seedance_image_to_video(image_path, prompt, duration=8):
    """Animate static image with Seedance."""
    with open(image_path, 'rb') as f:
        image_b64 = base64.b64encode(f.read()).decode()
    
    payload = {
        "prompt": prompt,
        "duration": duration,
        "startFrame": image_b64,
        "aspectRatio": "1:1"
    }
    
    resp = requests.post(
        f"{BASE_URL}/v1/seedance/generate",
        headers=headers,
        json=payload
    )
    return poll_job(resp.json()['jobId'])
```

**Prompt formulas** (see `skills/arcads-external-api/prompting/prompt-library/`):
- `seedance-2-ugc.md` — 9-layer UGC selfie formula
- `seedance-2-premium-reveal.md` — Dark void product reveal
- `seedance-2-product-hero.md` — Elemental effects (water, mist)
- `seedance-2-studio-lookbook.md` — Editorial multi-shot
- `seedance-2-feature-walkthrough.md` — Fast-paced demo

### Sora 2

**Text-to-video, up to 20s.**

```python
def generate_sora(prompt, duration=16, reference_image=None):
    """Generate Sora 2 video."""
    payload = {
        "prompt": prompt,
        "duration": duration,
        "aspectRatio": "16:9"
    }
    
    if reference_image:
        with open(reference_image, 'rb') as f:
            payload['referenceImage'] = base64.b64encode(f.read()).decode()
    
    resp = requests.post(
        f"{BASE_URL}/v1/sora2/generate",
        headers=headers,
        json=payload
    )
    return poll_job(resp.json()['jobId'])
```

**Sora 2 remix** (restyle existing video):

```python
def sora_remix(video_path, new_prompt):
    """Remix existing video with new style."""
    with open(video_path, 'rb') as f:
        video_b64 = base64.b64encode(f.read()).decode()
    
    payload = {
        "videoAsBase64": video_b64,
        "prompt": new_prompt
    }
    
    resp = requests.post(
        f"{BASE_URL}/v1/sora2/remix/video",
        headers=headers,
        json=payload
    )
    return poll_job(resp.json()['jobId'])
```

### Veo 3.1 (start-frame animation)

**Primary use case**: Animate static UGC stills with natural motion + dialogue.

```python
def veo_animate_still(image_path, prompt, dialogue, duration=8):
    """Animate static image with Veo 3.1."""
    with open(image_path, 'rb') as f:
        start_frame = base64.b64encode(f.read()).decode()
    
    payload = {
        "prompt": prompt,
        "startFrame": start_frame,
        "dialogue": dialogue,
        "duration": duration,
        "aspectRatio": "9:16"
    }
    
    resp = requests.post(
        f"{BASE_URL}/v1/veo/generate",
        headers=headers,
        json=payload
    )
    return poll_job(resp.json()['jobId'])
```

**MANDATORY dialogue gate**: Agent must confirm dialogue separately before generating.

### Kling 3.0 (b-roll / scene)

```python
def generate_kling_broll(scene_description, duration=5):
    """Generate b-roll clip with Kling 3.0."""
    payload = {
        "prompt": scene_description,
        "duration": duration,
        "type": "b-roll"
    }
    
    resp = requests.post(
        f"{BASE_URL}/v1/b-roll",
        headers=headers,
        json=payload
    )
    return poll_job(resp.json()['jobId'])

def generate_kling_scene(environment, duration=8):
    """Generate scene with Kling 3.0."""
    payload = {
        "prompt": environment,
        "duration": duration
    }
    
    resp = requests.post(
        f"{BASE_URL}/v1/scene",
        headers=headers,
        json=payload
    )
    return poll_job(resp.json()['jobId'])
```

### Grok Video

```python
def generate_grok(prompt, duration=10):
    """Generate Grok video."""
    payload = {
        "model": "grok-video",
        "prompt": prompt,
        "duration": duration
    }
    
    resp = requests.post(
        f"{BASE_URL}/v2/videos/generate",
        headers=headers,
        json=payload
    )
    return poll_job(resp.json()['jobId'])
```

### OmniHuman / Audio-driven

```python
def generate_omnihuman(avatar_description, script):
    """Generate talking avatar."""
    payload = {
        "avatar": avatar_description,
        "script": script
    }
    
    resp = requests.post(
        f"{BASE_URL}/v1/omnihuman",
        headers=headers,
        json=payload
    )
    return poll_job(resp.json()['jobId'])

def generate_audio_driven(video_path, audio_path):
    """Lip-sync video to audio file."""
    with open(video_path, 'rb') as f:
        video_b64 = base64.b64encode(f.read()).decode()
    with open(audio_path, 'rb') as f:
        audio_b64 = base64.b64encode(f.read()).decode()
    
    payload = {
        "videoAsBase64": video_b64,
        "audioAsBase64": audio_b64
    }
    
    resp = requests.post(
        f"{BASE_URL}/v1/audio-driven",
        headers=headers,
        json=payload
    )
    return poll_job(resp.json()['jobId'])
```

## Image generation

### Nano Banana (photoreal / lifestyle)

**Models**: `nano-banana-2` (default), `nano-banana` (Pro — tighter identity lock), `nano-banana-edit` (inpainting).

#### Basic generation

```python
def generate_nano_banana(prompt, model="nano-banana-2", aspect_ratio="1:1"):
    """Generate Nano Banana image."""
    payload = {
        "prompt": prompt,
        "model": model,
        "aspectRatio": aspect_ratio
    }
    
    resp = requests.post(
        f"{BASE_URL}/v1/nano-banana/generate",
        headers=headers,
        json=payload
    )
    return poll_job(resp.json()['jobId'])
```

#### With reference images (character consistency)

```python
def generate_with_references(prompt, reference_paths, model="nano-banana-2"):
    """Generate with multiple reference images for character lock."""
    references = []
    for path in reference_paths:
        with open(path, 'rb') as f:
            references.append(base64.b64encode(f.read()).decode())
    
    payload = {
        "prompt": prompt,
        "model": model,
        "referenceImages": references,
        "aspectRatio": "9:16"
    }
    
    resp = requests.post(
        f"{BASE_URL}/v1/nano-banana/generate",
        headers=headers,
        json=payload
    )
    return poll_job(resp.json()['jobId'])
```

#### Create AI influencer (10-image character sheet)

```python
def create_influencer_sheet(description, output_dir="references/influencers/"):
    """Generate 10-image character sheet."""
    import os
    os.makedirs(output_dir, exist_ok=True)
    
    # Step 1: Hero portrait
    hero_prompt = f"{description}, front-facing portrait, neutral expression, soft natural lighting"
    hero_result = generate_nano_banana(hero_prompt, aspect_ratio="1:1")
    hero_path = f"{output_dir}hero.png"
    download_image(hero_result['imageUrl'], hero_path)
    
    # Step 2: 9 additional angles
    angles = [
        "3/4 view left", "3/4 view right", "profile left", "profile right",
        "close-up face", "full body", "laughing expression", "serious expression",
        "over-the-shoulder"
    ]
    
    for i, angle in enumerate(angles, 1):
        prompt = f"{description}, {angle}, same person, consistent features"
        result = generate_with_references(prompt, [hero_path])
        download_image(result['imageUrl'], f"{output_dir}angle_{i:02d}.png")
    
    return output_dir

def download_image(url, path):
    """Helper to download image."""
    resp = requests.get(url)
    with open(path, 'wb') as f:
        f.write(resp.content)
```

#### UGC product selfie

```python
def generate_ugc_selfie(influencer_ref, product_ref, setting):
    """Generate UGC selfie with influencer + product."""
    refs = [influencer_ref, product_ref]
    
    # Add aesthetic references
    ugc_aesthetic_dir = "references/aesthetics/ugc-selfie/"
    for ref_file in os.listdir(ugc_aesthetic_dir):
        refs.append(f"{ugc_aesthetic_dir}{ref_file}")
    
    prompt = f"""
    iPhone selfie, {setting}, natural lighting, slightly grainy,
    person holding product visible in frame, authentic candid moment,
    skin texture visible, minor lens distortion, informal composition
    """
    
    return generate_with_references(prompt, refs, model="nano-banana")
```

### ChatGPT Image 2 (typography / UI mimicry)

```python
def generate_chatgpt_image(prompt, aspect_ratio="1:1"):
    """Generate ChatGPT Image 2."""
    payload = {
        "prompt": prompt,
        "aspectRatio": aspect_ratio
    }
    
    resp = requests.post(
        f"{BASE_URL}/v1/chatgpt-image/generate",
        headers=headers,
        json=payload
    )
    return poll_job(resp.json()['jobId'])
```

## Static Meta image ads (37-template library)

### Decision tree

- **Typography-heavy / UI mimicry** → `chatgpt-image-ad` (Apple Notes, fake Slack, comparison tables, ChatGPT conversation)
- **Photoreal / lifestyle / multi-reference** → `nano-banana-image-ad` (founder letter with portrait, product flat-lay, editorial hero)
- **Reverse-engineer existing ad** → `image-ad-clone`

### Generate from template library

```python
def generate_apple_notes_ad(product_name, benefits_list):
    """Apple Notes template — 8-12 bullet points."""
    prompt = f"""
    iOS Apple Notes app screenshot style:
    - Title: "{product_name}" in San Francisco font
    - Yellow header background (#FFD60A)
    - Clean white note background
    - Numbered list (emoji + text):
      {chr(10).join(f"  {i+1}. {benefit}" for i, benefit in enumerate(benefits_list))}
    - Timestamp "Just now" bottom right
    - Authentic iOS UI chrome (back button, share icon)
    - Screenshot aspect ratio
    """
    
    return generate_chatgpt_image(prompt, aspect_ratio="9:16")

def generate_forbes_editorial_ad(product_ref, headline, subhead):
    """Forbes editorial hero template."""
    with open(product_ref, 'rb') as f:
        ref_b64 = base64.b64encode(f.read()).decode()
    
    prompt = f"""
    Forbes magazine editorial layout:
    - Hero product image (centered, premium lighting)
    - Forbes logo top left
    - Headline: "{headline}" (Freight Big Pro, 48pt)
    - Subheadline: "{subhead}" (Untitled Sans, 18pt)
    - Clean white background, minimal typography, luxury aesthetic
    - Article byline and date bottom
    """
    
    payload = {
        "prompt": prompt,
        "referenceImages": [ref_b64],
        "aspectRatio": "4:5"
    }
    
    resp = requests.post(
        f"{BASE_URL}/v1/nano-banana/generate",
        headers=headers,
        json=payload
    )
    return poll_job(resp.json()['jobId'])
```

### Clone existing ad into template

```python
def clone_ad_as_template(source_image_path, backend="chatgpt-image"):
    """Reverse-engineer existing ad into library template."""
    # Phase 1: Visual analysis
    with open(source_image_path, 'rb') as f:
        image_b64 = base64.b64encode(f.read()).decode()
    
    # Use vision API to extract structure
    analysis = analyze_ad_structure(image_b64)
    
    # Phase 2: Template construction
    template_name = input("Template name (kebab-case): ")
    template_dir = f"shared/skills/image-ad-prompting/templates/{template_name}/"
    os.makedirs(template_dir, exist_ok=True)
    
    # Write template files
    with open(f"{template_dir}prompt.md", 'w') as f:
        f.write(analysis['prompt_formula'])
    
    with open(f"{template_dir}metadata.json", 'w') as f:
        json.dump({
            "backend": backend,
            "aspectRatios": analysis['supported_ratios'],
            "category": analysis['category']
        }, f, indent=2)
    
    # Phase 3: Test generation
    test_result = generate_from_template(template_name, backend)
    
    return template_dir
```

**Read `shared/skills/image-ad-prompting/OVERVIEW.md` first** for full workflow and aspect-ratio compatibility matrix.

## Multi-step pipelines

### Pixar-style 3D animated ad

**Requirements**: `ffmpeg`, `jq`, `npx hyperframes`, `whisper`

```bash
# Orchestrate from bash
./shared/skills/pixar-style-ad/scripts/generate.sh
```

**Python workflow**:

```python
def generate_pixar_ad(product_name, story_beats):
    """
    Generate Pixar-style animated ad.
    
    Args:
        product_name: Product name
        story_beats: List of 8 story beat descriptions
    """
    output_dir = f"output/pixar-ads/{product_name.lower().replace(' ', '-')}/"
    os.makedirs(output_dir, exist_ok=True)
    
    # Step 1: Cast sheet (mascot design)
    mascot_prompt = f"""
    Pixar-style 3D character sheet: {product_name} mascot,
    anthropomorphized, large expressive eyes, rounded friendly shapes,
    soft studio lighting, white backdrop, turnaround views
    """
    cast_result = generate_chatgpt_image(mascot_prompt, "1:1")
    cast_path = f"{output_dir}cast_sheet.png"
    download_image(cast_result['imageUrl'], cast_path)
    
    # Step 2: Storyboard stills (8 beats, sequential reference)
    stills = []
    prev_frame = None
    
    for i, beat in enumerate(story_beats):
        refs = [cast_path]
        if prev_frame:
            refs.append(prev_frame)
        
        # Max 5 referenceImages
        refs = refs[:5]
        
        prompt = f"Pixar 3D render: {beat}, same character from cast sheet, consistent style"
        
        with open(cast_path, 'rb') as f:
            cast_b64 = base64.b64encode(f.read()).decode()
        
        ref_data = [cast_b64]
        if prev_frame:
            with open(prev_frame, 'rb') as f:
                ref_data.append(base64.b64encode(f.read()).decode())
        
        payload = {
            "prompt": prompt,
            "referenceImages": ref_data,
            "aspectRatio": "16:9"
        }
        
        resp = requests.post(
            f"{BASE_URL}/v1/chatgpt-image/generate",
            headers=headers,
            json=payload
        )
        result = poll_job(resp.json()['jobId'])
        
        still_path = f"{output_dir}beat_{i+1:02d}.png"
        download_image(result['imageUrl'], still_path)
        stills.append(still_path)
        prev_frame = still_path
    
    # Step 3: Image-to-video per beat (Seedance 2.0)
    clips = []
    for i, still_path in enumerate(stills):
        video_result = seedance_image_to_video(
            still_path,
            prompt=f"Subtle animation, character movement, Pixar style",
            duration=3
        )
        clip_path = f"{output_dir}clip_{i+1:02d}.mp4"
        download_video(video_result['outputUrl'], clip_path)
        clips.append(clip_path)
    
    # Step 4: Stitch with ffmpeg
    final_path = f"{output_dir}final_stitched.mp4"
    stitch_videos(clips, final_path)
    
    # Step 5: Burn captions (optional)
    captioned_path = f"{output_dir}final_captioned.mp4"
    burn_captions(final_path, captioned_path)
    
    return captioned_path

def stitch_videos(clip_paths, output_path):
    """Stitch multiple clips with ffmpeg."""
    import subprocess
    
    concat_file = "/tmp/concat_list.txt"
    with open(concat_file, 'w') as f:
        for path in clip_paths:
            f.write(f"file '{os.path.abspath(path)}'\n")
    
    subprocess.run([
        "ffmpeg", "-f", "concat", "-safe", "0",
        "-i", concat_file, "-c", "copy", output_path
    ], check=True)

def burn_captions(video_path, output_path):
    """Transcribe and burn captions with HyperFrames."""
    import subprocess
    
    # Transcribe with Whisper
    import whisper
    model = whisper.load_model("medium.en")
    result = model.transcribe(video_path)
    
    # Write SRT
    srt_path = "/tmp/captions.srt"
    write_srt(result['segments'], srt_path)
    
    # Burn with HyperFrames
    subprocess.run([
        "npx", "hyperframes", "burn",
        "--input", video_path,
        "--srt", srt_path,
        "--output", output_path,
        "--style", "bold",
        "--color", "yellow"
    ], check=True)

def write_srt(segments, output_path):
    """Convert Whisper segments to SRT format."""
    with open(output_path, 'w') as f:
        for i, seg in enumerate(segments, 1):
            start = format_timestamp(seg['start'])
            end = format_timestamp(seg['end'])
            text = seg['text'].strip()
            f.write(f"{i}\n{start} --> {end}\n{text}\n\n")

def format_timestamp(seconds):
    """Format seconds to SRT timestamp."""
    hours = int(seconds // 3600)
    minutes = int((seconds % 3600) // 60)
    secs = int(seconds % 60)
    millis = int((seconds % 1) * 1000)
    return f"{hours:02d}:{minutes:02d}:{secs:02d},{millis:03d}"
```

### Claymation / Aardman-style ad

Same backbone as Pixar with clay textures and narrator-driven script.

```python
def generate_claymation_ad(product_name, narrator_script):
    """Generate claymation ad (60-115s)."""
    # 8-beat story arc from narrator script
    beats = split_into_beats(narrator_script, num_beats=8)
    
    # Cast sheet: plasticine characters
    cast_prompt = f"""
    Claymation character sheet: {product_name} mascot,
    sculpted plasticine texture, visible fingerprints,
    simple shapes, Aardman style, white backdrop
    """
    # ... similar workflow to Pixar
    
    # Stitch with optional stop-motion judder
    subprocess.run([
        "ffmpeg", "-i", "stitched.mp4",
        "-vf", "fps=12,fps=24",  # Stop-motion effect
        "final_claymation.mp4"
    ])
```

### YouTube thumbnails (5 CTR formulas)

```python
def generate_youtube_thumbnails(face_refs, product_ref, formula="peace-sign"):
    """
    Generate YouTube thumbnail batch.
    
    Formulas: peace-sign, comparison, terminal, reaction, before-after
    """
    formulas = {
        "peace-sign": "Person doing peace sign, product visible, bold text overlay",
        "comparison": "Split screen: Real vs AI, contrasting colors, VS text",
        "terminal": "Code terminal aesthetic, matrix green, tech vibes",
        "reaction": "Shocked expression, mouth open, surprised eyes, product reveal",
        "before-after": "Vertical split: before (dull) | after (vibrant)"
    }
    
    prompt = f"""
    YouTube thumbnail, 16:9 aspect ratio:
    - {formulas[formula]}
    - High contrast, bold colors (red/yellow/green)
    - Large readable text (Arial Black, 72pt+)
    - Person's face takes 40% of frame (likeness lock from references)
    - Product visible in hand or foreground
    - Slight lens distortion (wide-angle effect)
    - Vibrant saturation boost
    """
    
    # Batch generate 6 variations
    refs = face_refs + [product_ref]
    results = []
    
    for i in range(6):
        result = generate_with_references(
            f"{prompt}\nVariation {i+1}: minor composition shift",
            refs,
            model="nano-banana"
        )
        results.append(result)
    
    return results
```

## Configuration

### MASTER_CONTEXT.md

Your personal workspace file for campaign state:

```markdown
# Current Campaign: [Product Name]

## Active AI Influencers
- **Sofia** — 22, college student, freckles, golden-hour kitchen
  - References: `references/influencers/sofia/`
  - Used in: UGC batch 2024-01-15

## Product Assets
- Hero shot: `assets/products/bottle_hero.png`
- Lifestyle: `assets/products/in_use_kitchen.png`

## Active Prompts
- Seedance UGC: [link to prompt-library/seedance-2-ugc.md]
- Static ad: Apple Notes template

## Cost Tracking
- Seedance 2.0: $0.50/video (12s)
- Nano Banana 2: $0.10/image
- Sora 2: $1.20/video (16s)

## Notes
- Veo 3.1 requires explicit dialogue confirmation
- Nano Banana Pro locks identity better for influencer sheets
```

### Prompt library structure

```
skills/arcads-external-api/prompting/prompt-library/
├── seedance-2-ugc.md              # 9-layer UGC formula
├── seedance-2-premium-reveal.md   # Dark void product
├── seedance-2-product-hero.md     # Elemental effects
├── seedance-2-studio-lookbook.md  # Editorial multi-shot
└── seedance-2-feature-walkthrough.md  # Demo cuts
```

### Static ad template library

```
shared/skills/image-ad-prompting/templates/
├── apple-notes/
│   ├── prompt.md
│   └── metadata.json
├── forbes-editorial/
├── comparison-table/
├── fake-slack-thread/
├── chatgpt-conversation/
├── iphone-message/
└── ... (37 total)
```

## Common patterns

### Character consistency across multi-shot campaigns

```python
def maintain_character_consistency(influencer_name, new_scene):
    """Use influencer reference library for consistent character."""
    ref_dir = f"references/influencers/{influencer_name}/"
    
    # Load all reference angles
    refs = [f"{ref_dir}{f}" for f in os.listdir(ref_dir) if f.endswith('.png')]
    
    # Use hero + 4 most relevant angles (max 5 refs)
    primary_refs = [f"{ref_dir}hero.png"] + refs[1:5]
    
    result = generate_with_references(
        f"Same person from references, {new_scene}, consistent features and style",
        primary_refs,
        model="nano-banana"  # Pro model for tighter identity lock
    )
    
    return result
```

### Cost estimation before batch generation

```python
def estimate_cost(job_type, count):
    """Estimate cost before batch generation."""
    pricing = {
        "seedance-2.0": {"12s": 0.50, "8s": 0.35, "15s": 0.65},
        "sora-2": {"16s": 1.20, "20s": 1.50},
        "veo-3.1": {"8s": 0.80},
        "nano-banana-2": 0.10,
        "nano-banana": 0.15,  # Pro
        "chatgpt-image-2": 0.08
    }
    
    unit_cost = pricing.get
