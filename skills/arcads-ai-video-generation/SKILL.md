---
name: arcads-ai-video-generation
description: Generate AI marketing videos and static image ads using the Arcads API with skills for Seedance 2.0, Sora 2, Veo 3.1, Kling 3.0, Nano Banana, and 37 Meta ad templates
triggers:
  - "create an Arcads video"
  - "generate a UGC video with Seedance"
  - "make a Nano Banana product image"
  - "build a Meta image ad"
  - "animate this still with Veo"
  - "create a Pixar-style animated ad"
  - "generate AI influencer images"
  - "make a claymation ad video"
---

# Arcads AI Video Generation

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## What this project does

**arcads-claude-code** is a Python-based agent skill pack that provides programmatic access to the full Arcads creative stack for generating AI marketing videos and images. It includes:

- **Video models**: Seedance 2.0 (flagship), Sora 2, Veo 3.1, Kling 3.0, Grok Video, OmniHuman, Audio-driven
- **Image models**: Nano Banana 2/Pro/Edit, ChatGPT Image 2
- **37 static Meta image ad templates** with dedicated generators
- **Multi-step pipelines**: Pixar-style ads, claymation ads, YouTube thumbnails
- **Agent-native workflows**: polling, cost gates, prompt engineering, file organization

The project is designed for AI coding agents (Claude Code, Cursor) to autonomously generate marketing creative through natural language commands.

## Installation

### 1. Clone and setup

```bash
git clone https://github.com/krusemediallc/arcads-claude-code.git
cd arcads-claude-code
./scripts/setup.sh
```

The setup script will:
- Prompt for your Arcads API key (get it from [app.arcads.ai/settings/api](https://app.arcads.ai/settings/api))
- Create `.env` with `ARCADS_API_KEY=your_key_here`
- Verify API connection
- Generate `MASTER_CONTEXT.md` workspace file

### 2. Install dependencies

**Core (required for all workflows):**
```bash
python3 -m pip install requests python-dotenv
```

**Optional (for specific pipelines):**
```bash
# For video stitching and Pixar/claymation workflows
brew install ffmpeg jq

# For caption burn-in
brew install node
pip install openai-whisper

# For Meta ad publishing
pip install -r shared/skills/meta-ad-builder/scripts/requirements.txt
```

### 3. Environment variables

Create `.env` in the project root:
```bash
ARCADS_API_KEY=your_api_key_here
```

## Core API patterns

### Base configuration

```python
import os
import requests
from dotenv import load_dotenv

load_dotenv()

API_KEY = os.getenv('ARCADS_API_KEY')
BASE_URL = 'https://api.arcads.ai'

headers = {
    'Authorization': f'Bearer {API_KEY}',
    'Content-Type': 'application/json'
}
```

### Standard video generation flow

```python
# 1. Submit generation request
def generate_video(prompt, model='seedance-2', duration=12):
    response = requests.post(
        f'{BASE_URL}/v1/videos/generate',
        headers=headers,
        json={
            'prompt': prompt,
            'model': model,
            'duration': duration
        }
    )
    return response.json()

# 2. Poll for completion
def poll_video(job_id, interval=10):
    import time
    while True:
        response = requests.get(
            f'{BASE_URL}/v1/videos/{job_id}',
            headers=headers
        )
        data = response.json()
        
        if data['status'] == 'completed':
            return data['videoUrl']
        elif data['status'] == 'failed':
            raise Exception(f"Generation failed: {data.get('error')}")
        
        time.sleep(interval)

# 3. Download result
def download_video(url, output_path):
    response = requests.get(url)
    with open(output_path, 'wb') as f:
        f.write(response.content)
```

### Full example workflow

```python
# Generate a 12-second Seedance UGC video
result = generate_video(
    prompt="""
    A young woman in her mid-20s sits in a cozy kitchen, natural morning light
    streaming through a window. She holds up a skincare bottle, speaking directly
    to camera with natural eye contact breaks. iPhone-shot aesthetic, authentic
    and casual delivery.
    """,
    model='seedance-2',
    duration=12
)

job_id = result['jobId']
print(f"Job submitted: {job_id}")

# Poll until complete
video_url = poll_video(job_id)
print(f"Video ready: {video_url}")

# Download
download_video(video_url, 'output/ugc_skincare.mp4')
```

## Video models

### Seedance 2.0 (flagship model)

**Best for:** UGC content, product reveals, feature walkthroughs, 4-15s clips with native audio

```python
# UGC selfie-style product review (9-layer formula)
response = requests.post(
    f'{BASE_URL}/v1/videos/generate',
    headers=headers,
    json={
        'model': 'seedance-2',
        'duration': 12,
        'prompt': """
        Shot on iPhone 14 Pro in natural light. A woman in her late 20s sits
        in a modern kitchen, holding [PRODUCT]. She speaks directly to camera
        with natural pauses and eye-contact breaks. Casual, authentic delivery.
        "I used to buy [COMPETITOR] until I found this..."
        """,
        'style': 'ugc'
    }
)
```

**Premium product reveal (no person):**
```python
response = requests.post(
    f'{BASE_URL}/v1/videos/generate',
    headers=headers,
    json={
        'model': 'seedance-2',
        'duration': 10,
        'prompt': """
        Dark void background. Premium watch floats and rotates slowly.
        Text overlay appears: "Swiss precision. 40-hour power reserve."
        Dramatic lighting with subtle reflections. Hero product reveal.
        """,
        'style': 'premium'
    }
)
```

**Image-to-video with reference:**
```python
import base64

with open('product_hero.jpg', 'rb') as f:
    img_b64 = base64.b64encode(f.read()).decode('utf-8')

response = requests.post(
    f'{BASE_URL}/v1/videos/generate',
    headers=headers,
    json={
        'model': 'seedance-2',
        'duration': 8,
        'prompt': 'Zoom into the product label, then pan around showing texture details',
        'startFrame': img_b64
    }
)
```

### Veo 3.1 (start-frame animation)

**Best for:** Animating stills into videos with dialogue, UGC still → video pipeline

```python
# Animate a Nano Banana still with dialogue
with open('ugc_still.jpg', 'rb') as f:
    start_frame = base64.b64encode(f.read()).decode('utf-8')

response = requests.post(
    f'{BASE_URL}/v1/veo3/animate',
    headers=headers,
    json={
        'startFrame': start_frame,
        'duration': 8,
        'prompt': 'Natural head movement, blinking, slight smile',
        'dialogue': "This serum changed my entire skincare routine"
    }
)
```

**IMPORTANT:** Veo 3.1 requires explicit dialogue confirmation before generation:
```python
def confirm_dialogue(script):
    """Agent must get user approval for dialogue before Veo generation"""
    print(f"Dialogue to be embedded:\n{script}\n")
    confirm = input("Approve dialogue? (yes/no): ")
    return confirm.lower() == 'yes'

if confirm_dialogue(dialogue_text):
    # proceed with generation
```

### Sora 2 (text-to-video, up to 20s)

**Best for:** Longer scenes, cinematic establishing shots

```python
response = requests.post(
    f'{BASE_URL}/v1/sora2/generate',
    headers=headers,
    json={
        'prompt': """
        Aerial drone shot: sunrise over a mountain lake. Camera slowly descends
        revealing a lone figure standing at the water's edge. Golden hour light,
        mist rising from the water. Cinematic, 24fps feel.
        """,
        'duration': 16,
        'aspectRatio': '16:9'
    }
)
```

**Sora 2 remix (restyle existing video):**
```python
response = requests.post(
    f'{BASE_URL}/v1/sora2/remix/video',
    headers=headers,
    json={
        'sourceVideoUrl': 'https://example.com/original.mp4',
        'prompt': 'Transform into cyberpunk aesthetic with neon colors',
        'strength': 0.7  # 0.0-1.0, higher = more transformation
    }
)
```

### Kling 3.0 (B-roll and scene generation)

**Best for:** Background footage, establishing shots, 5-10s clips

```python
# B-roll clip
response = requests.post(
    f'{BASE_URL}/v1/b-roll',
    headers=headers,
    json={
        'prompt': 'Coffee being poured into a white mug, steam rising, macro shot',
        'duration': 5
    }
)

# Scene generation
response = requests.post(
    f'{BASE_URL}/v1/scene',
    headers=headers,
    json={
        'prompt': 'Modern minimalist office space, large windows, afternoon light',
        'duration': 8
    }
)
```

### Other models

```python
# Grok Video
response = requests.post(
    f'{BASE_URL}/v2/videos/generate',
    headers=headers,
    json={
        'model': 'grok-video',
        'prompt': 'Your scene description',
        'duration': 10
    }
)

# OmniHuman (talking avatar)
response = requests.post(
    f'{BASE_URL}/v1/omnihuman',
    headers=headers,
    json={
        'avatarImage': avatar_base64,
        'script': 'Welcome to our product demo...',
        'voiceId': 'professional-female'
    }
)

# Audio-driven (lip sync)
response = requests.post(
    f'{BASE_URL}/v1/audio-driven',
    headers=headers,
    json={
        'videoUrl': 'https://example.com/person_silent.mp4',
        'audioUrl': 'https://example.com/voiceover.mp3'
    }
)
```

## Image generation

### Nano Banana (photoreal images)

**Model variants:**
- `nano-banana-2`: Default, fast, good quality
- `nano-banana` (Pro): Gemini 3 Pro Image — higher fidelity, better character consistency
- `nano-banana-edit`: Inpainting/editing

```python
# Generate a UGC product selfie
response = requests.post(
    f'{BASE_URL}/v1/images/generate',
    headers=headers,
    json={
        'model': 'nano-banana-2',
        'prompt': """
        iPhone selfie shot. Young woman, 24, freckles, natural makeup, holding
        skincare bottle. Bedroom background, soft morning light through curtain.
        Authentic, unfiltered aesthetic. Slight lens distortion, natural grain.
        """,
        'aspectRatio': '9:16',
        'numImages': 1
    }
)
```

**With reference images for character consistency:**
```python
import base64

# Load reference images
refs = []
for img_path in ['hero_front.jpg', 'hero_3quarter.jpg', 'hero_profile.jpg']:
    with open(f'references/influencers/{img_path}', 'rb') as f:
        refs.append(base64.b64encode(f.read()).decode('utf-8'))

response = requests.post(
    f'{BASE_URL}/v1/images/generate',
    headers=headers,
    json={
        'model': 'nano-banana-2',
        'prompt': 'Same person holding product in different pose',
        'referenceImages': refs,
        'aspectRatio': '4:5'
    }
)
```

**Create AI influencer character sheet (10-image workflow):**
```python
def create_influencer_sheet(character_description):
    # Phase 1: Generate hero front portrait
    hero = requests.post(
        f'{BASE_URL}/v1/images/generate',
        headers=headers,
        json={
            'model': 'nano-banana-2',
            'prompt': f"""
            Professional front-facing portrait. {character_description}.
            Direct eye contact, neutral expression, even lighting, white background.
            High detail on facial features for reference consistency.
            """,
            'aspectRatio': '4:5'
        }
    ).json()
    
    hero_url = poll_image(hero['jobId'])
    
    # User approval gate
    print(f"Hero portrait: {hero_url}")
    if input("Approve hero? (yes/no): ").lower() != 'yes':
        return None
    
    # Download hero for references
    hero_b64 = download_as_base64(hero_url)
    
    # Phase 2: Generate 9 additional angles using hero as reference
    angles = [
        "3/4 view looking left, slight smile",
        "3/4 view looking right, neutral expression",
        "Profile view left side, serious expression",
        "Profile view right side, laughing",
        "Close-up of face, surprised expression",
        "Close-up of face, concentrated expression",
        "Full body shot, casual standing pose",
        "Candid expression, mid-conversation",
        "Looking over shoulder, playful expression"
    ]
    
    images = [hero_url]
    for angle_prompt in angles:
        response = requests.post(
            f'{BASE_URL}/v1/images/generate',
            headers=headers,
            json={
                'model': 'nano-banana-2',
                'prompt': f"{character_description}. {angle_prompt}",
                'referenceImages': [hero_b64],
                'aspectRatio': '4:5'
            }
        ).json()
        
        img_url = poll_image(response['jobId'])
        images.append(img_url)
    
    return images

# Usage
influencer_images = create_influencer_sheet(
    "Woman, 22 years old, college student, freckles across nose, "
    "wavy brown hair, green eyes, natural makeup"
)
```

### ChatGPT Image 2 (typography and UI-style ads)

**Best for:** Text-heavy designs, UI mockups, screenshot-style ads

```python
response = requests.post(
    f'{BASE_URL}/v1/images/generate',
    headers=headers,
    json={
        'model': 'gpt-image-2',
        'prompt': """
        Apple Notes app interface. Title: "Why I switched to [PRODUCT]"
        Bulleted list with checkmarks:
        ✓ Saves me 2 hours every day
        ✓ Cut costs by 40%
        ✓ Actually works (unlike [COMPETITOR])
        iOS design aesthetic, light mode, clean typography.
        """,
        'aspectRatio': '1:1'
    }
)
```

## Static Meta image ads (37 template library)

### Template categories

The project includes 37 validated ad templates across three generator skills:

**ChatGPT Image 2 templates (typography/UI-heavy):**
- Apple Notes list, Forbes editorial, fake Google search, comparison table
- Sticky-note flatlay, Slack thread, ChatGPT conversation, iMessage screenshot
- Magazine cover, billboard, weather forecast UI, scratch-off ticket

**Nano Banana templates (photoreal/lifestyle):**
- Product hero on table, hand holding product, bathroom counter
- Kitchen scene, desk workspace, car interior, gym environment

**Cross-compatible templates:**
- Before/after split, founder letter, dating-app card, museum exhibit

### Using the image-ad generators

```python
# Option 1: chatgpt-image-ad (stdlib-only, no pip installs)
import subprocess

result = subprocess.run([
    'python3',
    'shared/skills/chatgpt-image-ad/generator.py',
    '--template', 'apple-notes',
    '--product', 'TimeBlock Pro',
    '--hook', 'Why I stopped using Google Calendar',
    '--bullets', 'Saves 2 hrs/day|Built for ADHD brains|Actually syncs'
], capture_output=True, text=True)

print(result.stdout)  # Job ID and polling info
```

```python
# Option 2: nano-banana-image-ad (photoreal)
result = subprocess.run([
    'python3',
    'shared/skills/nano-banana-image-ad/generator.py',
    '--template', 'bathroom-counter',
    '--product-image', 'product_photos/serum.jpg',
    '--style', 'morning-light-marble'
], capture_output=True, text=True)
```

```python
# Option 3: image-ad-clone (reverse-engineer existing ad)
result = subprocess.run([
    'python3',
    'shared/skills/image-ad-clone/cloner.py',
    '--source-image', 'competitor_ads/example.jpg',
    '--backend', 'nano-banana-2',  # or 'gpt-image-2'
    '--save-template'  # Creates new reusable template
], capture_output=True, text=True)
```

### Publishing to Meta Marketing API

```python
# After generating image, publish as Meta ad (paused)
import sys
sys.path.append('shared/skills/meta-ad-builder/scripts')
from meta_publisher import publish_ad

ad_data = {
    'image_path': 'output/apple_notes_ad.jpg',
    'headline': 'Stop wasting time on [competitor]',
    'primary_text': 'TimeBlock Pro helps ADHD brains stay focused...',
    'link': 'https://example.com',
    'call_to_action': 'LEARN_MORE'
}

ad_id = publish_ad(
    ad_account_id=os.getenv('META_AD_ACCOUNT_ID'),
    access_token=os.getenv('META_ACCESS_TOKEN'),
    ad_data=ad_data,
    status='PAUSED'  # Always create paused for review
)

print(f"Ad created (paused): {ad_id}")
```

## Multi-step animated pipelines

### Pixar-style 3D animated ad

**Pipeline:** Cast lockdown → 8-beat storyboard → Seedance i2v per beat → ffmpeg stitch + captions

```python
import subprocess
import json

def generate_pixar_ad(product_name, mascot_description, story_beats):
    """
    story_beats: list of 8 scene descriptions
    """
    
    # Phase 1: Lock character design
    mascot_response = requests.post(
        f'{BASE_URL}/v1/images/generate',
        headers=headers,
        json={
            'model': 'gpt-image-2',
            'prompt': f"""
            Pixar-style 3D character design. {mascot_description}.
            Full body turnaround reference. Clean, appealing, anthropomorphized.
            Soft lighting, high-quality render.
            """,
            'aspectRatio': '16:9'
        }
    ).json()
    
    mascot_img = poll_image(mascot_response['jobId'])
    mascot_b64 = download_as_base64(mascot_img)
    
    # Phase 2: Generate 8 storyboard frames (sequential, each uses prior as ref)
    frames = []
    prev_frame = mascot_b64
    
    for i, beat in enumerate(story_beats):
        refs = [mascot_b64, prev_frame] if i > 0 else [mascot_b64]
        
        frame_response = requests.post(
            f'{BASE_URL}/v1/images/generate',
            headers=headers,
            json={
                'model': 'gpt-image-2',
                'prompt': f"Pixar-style 3D scene. Beat {i+1}: {beat}",
                'referenceImages': refs[:5],  # Max 5 refs per API
                'aspectRatio': '16:9'
            }
        ).json()
        
        frame_url = poll_image(frame_response['jobId'])
        frames.append(frame_url)
        prev_frame = download_as_base64(frame_url)
    
    # Phase 3: Animate each frame with Seedance 2.0 (image-to-video)
    videos = []
    for i, frame_url in enumerate(frames):
        frame_b64 = download_as_base64(frame_url)
        
        video_response = requests.post(
            f'{BASE_URL}/v1/videos/generate',
            headers=headers,
            json={
                'model': 'seedance-2',
                'startFrame': frame_b64,
                'duration': 6,
                'prompt': f"Subtle character animation matching beat {i+1} energy"
            }
        ).json()
        
        video_url = poll_video(video_response['jobId'])
        videos.append(video_url)
    
    # Phase 4: Stitch with ffmpeg
    concat_file = 'temp_concat.txt'
    with open(concat_file, 'w') as f:
        for v in videos:
            local_path = f"temp_beat_{videos.index(v)}.mp4"
            download_video(v, local_path)
            f.write(f"file '{local_path}'\n")
    
    subprocess.run([
        'ffmpeg', '-f', 'concat', '-safe', '0', '-i', concat_file,
        '-c', 'copy', 'output/pixar_ad_raw.mp4'
    ])
    
    # Phase 5: Burn captions (requires HyperFrames + Whisper)
    subprocess.run([
        'python3',
        'shared/skills/caption-video/burn_captions.py',
        '--input', 'output/pixar_ad_raw.mp4',
        '--output', 'output/pixar_ad_final.mp4'
    ])
    
    return 'output/pixar_ad_final.mp4'

# Usage
story = [
    "Mascot wakes up looking tired, alarm clock ringing",
    "Mascot struggles with messy morning routine",
    "Product appears with magical glow",
    "Mascot uses product, eyes light up",
    "Mascot's day transforms - organized and happy",
    "Mascot recommends product to friend",
    "Both mascots using product, high-five",
    "Product hero shot with brand logo reveal"
]

final_video = generate_pixar_ad(
    product_name="MorningFlow",
    mascot_description="Friendly blue fox character, expressive eyes, wears a scarf",
    story_beats=story
)
```

### Claymation ad

Similar 8-beat structure, uses clay textures and stop-motion aesthetic:

```python
def generate_claymation_ad(product_name, story_beats):
    # Same pipeline as Pixar but with claymation prompts
    for beat in story_beats:
        prompt = f"""
        Aardman-style claymation. Sculpted plasticine characters with visible
        fingerprint textures. {beat}. Stop-motion aesthetic, 12fps judder feel.
        Warm practical lighting, handcrafted props.
        """
        # ... same storyboard → Seedance → stitch flow
    
    # Apply stop-motion judder in ffmpeg
    subprocess.run([
        'ffmpeg', '-i', 'output/clay_raw.mp4',
        '-vf', 'fps=12,fps=24',  # Simulate stop-motion
        'output/clay_final.mp4'
    ])
```

### YouTube thumbnail generator

Specialized skill with 5 CTR formulas:

```python
import subprocess

# Generate 6 thumbnail variations
result = subprocess.run([
    'python3',
    'shared/skills/generate-youtube-thumbnail/generator.py',
    '--style', 'peace-sign-branding',  # or comparison, terminal, reaction, before-after
    '--face-refs', 'references/creator/face_*.jpg',
    '--product-image', 'product.jpg',
    '--text', 'I Tested 37 AI Tools',
    '--variations', '6'
], capture_output=True, text=True)

print(result.stdout)
```

## Common patterns

### Polling with exponential backoff

```python
import time

def poll_with_backoff(job_id, max_wait=600):
    """Poll with exponential backoff, max 10 minutes"""
    intervals = [5, 10, 15, 30, 30, 60, 60, 60]
    total_wait = 0
    
    for interval in intervals:
        if total_wait >= max_wait:
            raise TimeoutError(f"Job {job_id} exceeded max wait time")
        
        response = requests.get(
            f'{BASE_URL}/v1/videos/{job_id}',
            headers=headers
        )
        data = response.json()
        
        if data['status'] == 'completed':
            return data
        elif data['status'] == 'failed':
            raise Exception(f"Job failed: {data.get('error')}")
        
        print(f"Status: {data['status']}, waiting {interval}s...")
        time.sleep(interval)
        total_wait += interval
    
    raise TimeoutError(f"Job {job_id} still processing after {max_wait}s")
```

### Cost estimation gate

```python
def estimate_cost(model, duration=None, num_images=None):
    """Show cost estimate before generation"""
    pricing = {
        'seedance-2': 0.10,  # per second
        'sora-2': 0.15,
        'veo-3': 0.12,
        'nano-banana-2': 0.05,  # per image
        'gpt-image-2': 0.04
    }
    
    if duration:
        cost = pricing.get(model, 0.10) * duration
        print(f"Estimated cost: ${cost:.2f} for {duration}s {model} video")
    elif num_images:
        cost = pricing.get(model, 0.05) * num_images
        print(f"Estimated cost: ${cost:.2f} for {num_images} {model} images")
    
    confirm = input("Proceed with generation? (yes/no): ")
    return confirm.lower() == 'yes'
```

### Batch generation with parallel requests

```python
import concurrent.futures

def generate_batch_images(prompts, model='nano-banana-2'):
    """Generate multiple images in parallel"""
    
    def generate_one(prompt):
        response = requests.post(
            f'{BASE_URL}/v1/images/generate',
            headers=headers,
            json={'model': model, 'prompt': prompt, 'aspectRatio': '4:5'}
        )
        job_id = response.json()['jobId']
        return poll_image(job_id)
    
    with concurrent.futures.ThreadPoolExecutor(max_workers=5) as executor:
        futures = [executor.submit(generate_one, p) for p in prompts]
        results = [f.result() for f in concurrent.futures.as_completed(futures)]
    
    return results

# Usage: generate 6 thumbnail variations in parallel
prompts = [
    f"YouTube thumbnail variation {i+1}, same person, different expression"
    for i in range(6)
]
thumbnails = generate_batch_images(prompts)
```

### File organization pattern

```python
import os
from datetime import datetime

def organize_output(file_url, project_name, asset_type):
    """Download and organize with timestamp"""
    timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
    
    # Create project directory structure
    base_dir = f"output/{project_name}"
    os.makedirs(f"{base_dir}/{asset_type}", exist_ok=True)
    
    # Download with descriptive name
    ext = 'mp4' if asset_type == 'videos' else 'jpg'
    filename = f"{timestamp}_{asset_type}.{ext}"
    output_path = f"{base_dir}/{asset_type}/{filename}"
    
    response = requests.get(file_url)
    with open(output_path, 'wb') as f:
        f.write(response.content)
    
    print(f"Saved: {output_path}")
    return output_path

# Usage
video_url = poll_video(job_id)
path = organize_output(video_url, 'skincare_campaign_q2', 'videos')
```

## Troubleshooting

### API key issues

```python
def verify_api_key():
    """Test API connectivity"""
    try:
        response = requests.get(
            f'{BASE_URL}/v1/account',
            headers=headers
        )
        if response.status_code == 200:
            print("✓ API key valid")
            print(f"Account: {response.json()}")
            return True
        else:
            print(f"✗ API error: {response.status_code}")
            print(response.text)
            return False
    except Exception as e:
        print(f"✗ Connection error: {e}")
        return False

# Run at startup
if not verify_api_key():
    print("Check your ARCADS_API_KEY in .env")
    exit(1)
```

### Generation failures

Common failure reasons and fixes:

```python
def handle_generation_error(error_data):
    """Parse and suggest fixes for common errors"""
    error_msg = error_data.get('error', '')
    
    fixes = {
        'insufficient credits': 'Top up your Arcads account at app.arcads.ai/billing',
        'invalid reference image': 'Ensure images are <10MB, JPEG/PNG, and base64-encoded',
        'prompt too long': 'Shorten prompt to <2000 chars, focus on key visual details',
        'duration out of range': 'Seedance: 4-15s, Sora: 4-20s, Veo: 4-12s',
        'invalid aspect ratio': 'Use 16:9, 9:16, 4:5, 1:1, or 4:3',
        'rate limit exceeded': 'Wait 60s between batch requests'
    }
    
    for keyword, fix in fixes.items():
        if keyword in error_msg.lower():
            print(f"Error: {error_msg}")
            print(f"Fix: {fix}")
            return
    
    print(f"Unknown error: {
