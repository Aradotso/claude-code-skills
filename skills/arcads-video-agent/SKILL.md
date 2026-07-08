---
name: arcads-video-agent
description: Generate AI marketing videos and images using Arcads creative stack (Seedance 2.0, Sora 2, Veo 3.1, Kling, Nano Banana, ChatGPT Image) from Claude Code or Cursor
triggers:
  - "generate an arcads video"
  - "create a seedance ugc ad"
  - "make a nano banana product image"
  - "build a pixar style animated ad"
  - "generate meta image ad creative"
  - "create ai influencer character sheet"
  - "animate this image with veo"
  - "make claymation ad campaign"
---

# Arcads Video Agent Skill

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

This skill provides AI agent capabilities for the **Arcads Claude Code** project — a comprehensive toolkit for generating AI marketing videos and images using your Arcads account. Supports the full creative stack: **Seedance 2.0** (flagship video), **Sora 2**, **Veo 3.1**, **Kling 3.0**, **Grok Video**, **Nano Banana 2/Pro/Edit**, **ChatGPT Image 2**, **OmniHuman**, and **Audio-driven** models, plus 37 static Meta image-ad templates and multi-step pipelines for Pixar-style and claymation animated ads.

## What This Project Does

Arcads Claude Code is an agent skill pack that transforms natural language requests into production-ready video and image ads through:

- **Video Generation**: UGC selfies, product reveals, b-roll, scene animations (4-20s clips)
- **Image Generation**: AI influencer creation, product showcases, UGC stills, photoreal composites
- **Static Ad Library**: 37 validated Meta image-ad templates (Apple Notes, Forbes editorial, comparison tables, fake UI screenshots)
- **Multi-Step Pipelines**: Pixar-style 3D animation, claymation ads, caption burn-in workflows
- **API Orchestration**: Automated polling, cost confirmation, file organization, prompt engineering

## Installation

### Prerequisites

```bash
# Required for everything
python3 --version  # Must be 3.10+

# Optional dependencies (install only if using specific workflows)
brew install ffmpeg  # For Pixar/claymation/caption workflows
brew install jq      # For bash pipeline scripts
brew install node    # For caption burn-in (hyperframes)
pip install openai-whisper  # For transcription
```

### Setup

```bash
# Clone the repository
git clone https://github.com/krusemediallc/arcads-claude-code.git
cd arcads-claude-code

# Run interactive setup
./scripts/setup.sh
```

The setup script will:
1. Prompt for Arcads API key (get from [app.arcads.ai/settings/api](https://app.arcads.ai/settings/api))
2. Create `.env` file with credentials
3. Create `MASTER_CONTEXT.md` workspace file
4. Verify API connection

### Configuration

Create or verify `.env` in project root:

```bash
ARCADS_API_KEY=your_api_key_here
```

## Core API Usage

### Python API Client Pattern

```python
import os
import requests
import json
import time

# Load API key from environment
API_KEY = os.getenv('ARCADS_API_KEY')
BASE_URL = 'https://api.arcads.ai'

def make_request(endpoint, method='POST', payload=None):
    """Standard API request with error handling."""
    headers = {
        'Authorization': f'Bearer {API_KEY}',
        'Content-Type': 'application/json'
    }
    
    url = f'{BASE_URL}{endpoint}'
    
    if method == 'POST':
        response = requests.post(url, headers=headers, json=payload)
    elif method == 'GET':
        response = requests.get(url, headers=headers)
    
    response.raise_for_status()
    return response.json()

def poll_until_complete(job_id, endpoint='/v1/jobs'):
    """Poll job status until completion."""
    while True:
        status = make_request(f'{endpoint}/{job_id}', method='GET')
        
        if status['status'] in ['completed', 'failed']:
            return status
        
        print(f"Status: {status['status']} - {status.get('progress', 0)}%")
        time.sleep(5)
```

## Video Generation Workflows

### Seedance 2.0 — UGC Selfie-Style Product Review

```python
def generate_seedance_ugc(product_name, competitor_name, duration=12):
    """
    Generate UGC selfie-style video using 9-layer Seedance prompt formula.
    See: skills/arcads-external-api/prompting/prompt-library/seedance-2-ugc.md
    """
    
    payload = {
        "model": "seedance-2",
        "duration": duration,
        "prompt": f"""
        iPhone selfie POV. Woman in modern kitchen, natural window light.
        
        She looks at camera, holds up {product_name} product.
        
        "I used to buy {competitor_name} until I found this."
        
        She examines the product, reads label, nods approvingly.
        
        Direct eye contact: "It's actually cheaper and works better."
        
        She sets product on counter, natural smile.
        
        Shot style: Handheld iPhone 15 Pro, 0.5x selfie lens. 
        Authentic home lighting, slight motion blur.
        Real person aesthetic — pores, natural makeup, casual delivery.
        """,
        "aspectRatio": "9:16"
    }
    
    # Submit job
    response = make_request('/v1/seedance2/video', payload=payload)
    job_id = response['jobId']
    
    print(f"Seedance job submitted: {job_id}")
    print(f"Estimated cost: ${response.get('estimatedCost', 0)}")
    
    # Poll until complete
    result = poll_until_complete(job_id)
    
    if result['status'] == 'completed':
        video_url = result['videoUrl']
        print(f"Video ready: {video_url}")
        return video_url
    else:
        raise Exception(f"Generation failed: {result.get('error')}")
```

### Veo 3.1 — Image to Video with Dialogue

```python
def animate_still_with_veo(image_path, dialogue_script, duration=8):
    """
    Animate a Nano Banana still into video with embedded dialogue.
    Uses startFrame for exact image match + natural motion.
    """
    
    import base64
    
    # Load and encode image
    with open(image_path, 'rb') as f:
        image_b64 = base64.b64encode(f.read()).decode('utf-8')
    
    payload = {
        "model": "veo-3.1",
        "duration": duration,
        "startFrame": image_b64,
        "prompt": f"""
        Natural human motion and expression from starting frame.
        Person speaks: "{dialogue_script}"
        
        Subtle head movement, natural blinking, lip sync to dialogue.
        Hands stay in frame, minimal camera shake.
        Indoor lighting maintains original atmosphere.
        """,
        "dialogue": dialogue_script,  # MANDATORY for Veo with speech
        "aspectRatio": "9:16"
    }
    
    response = make_request('/v1/veo3.1/video', payload=payload)
    job_id = response['jobId']
    
    result = poll_until_complete(job_id)
    return result['videoUrl']
```

### Sora 2 — Text to Video (Longer Durations)

```python
def generate_sora_scene(scene_description, duration=16):
    """
    Generate text-to-video with Sora 2 (up to 20s).
    Auto-selects duration from script word count (~2.5 words/sec).
    """
    
    payload = {
        "model": "sora-2",
        "duration": duration,
        "prompt": scene_description,
        "aspectRatio": "16:9"  # Sora works well for landscape
    }
    
    response = make_request('/v1/sora2/video', payload=payload)
    return poll_until_complete(response['jobId'])
```

### Kling 3.0 — B-Roll and Scene Generation

```python
def generate_broll(scene_description, duration=5):
    """B-roll clip generation via dedicated endpoint."""
    
    payload = {
        "prompt": scene_description,
        "duration": duration,
        "aspectRatio": "16:9"
    }
    
    response = make_request('/v1/b-roll', payload=payload)
    return poll_until_complete(response['jobId'])

def generate_scene(environment_description, duration=8):
    """Scene generation via dedicated endpoint."""
    
    payload = {
        "prompt": environment_description,
        "duration": duration,
        "aspectRatio": "16:9"
    }
    
    response = make_request('/v1/scene', payload=payload)
    return poll_until_complete(response['jobId'])
```

## Image Generation Workflows

### Create AI Influencer Character Sheet (10 Images)

```python
import os

def create_ai_influencer(description, name, output_dir='references/influencers'):
    """
    Two-pass workflow:
    1. Generate hero front portrait
    2. Generate 9 additional angles using hero as reference
    """
    
    os.makedirs(f'{output_dir}/{name}', exist_ok=True)
    
    # Phase 1: Hero portrait
    hero_payload = {
        "model": "nano-banana-2",
        "prompt": f"""
        Front-facing portrait: {description}
        
        Direct eye contact, neutral expression, even lighting.
        Sharp focus, professional photo quality.
        """,
        "aspectRatio": "1:1"
    }
    
    hero_response = make_request('/v1/nano-banana/image', payload=hero_payload)
    hero_result = poll_until_complete(hero_response['jobId'])
    hero_url = hero_result['imageUrl']
    
    print(f"Hero portrait generated: {hero_url}")
    print("Review and approve before generating additional angles? (y/n)")
    
    # Download hero
    import requests
    hero_img = requests.get(hero_url).content
    with open(f'{output_dir}/{name}/00_hero.png', 'wb') as f:
        f.write(hero_img)
    
    # Phase 2: Generate 9 additional angles
    import base64
    hero_b64 = base64.b64encode(hero_img).decode('utf-8')
    
    angles = [
        "3/4 profile, looking slightly left",
        "3/4 profile, looking slightly right",
        "Full side profile, looking left",
        "Close-up, slight smile",
        "Close-up, neutral expression",
        "Torso shot, arms crossed",
        "Full body, standing casual",
        "Candid laugh, natural",
        "Thoughtful expression, hand on chin"
    ]
    
    for idx, angle_desc in enumerate(angles, start=1):
        payload = {
            "model": "nano-banana-2",
            "prompt": f"{description}. Angle: {angle_desc}",
            "referenceImages": [hero_b64],
            "aspectRatio": "1:1"
        }
        
        response = make_request('/v1/nano-banana/image', payload=payload)
        result = poll_until_complete(response['jobId'])
        
        # Download and save
        img_data = requests.get(result['imageUrl']).content
        with open(f'{output_dir}/{name}/{idx:02d}_{angle_desc[:20]}.png', 'wb') as f:
            f.write(img_data)
        
        print(f"Generated angle {idx}/9: {angle_desc}")
    
    print(f"\n✓ Character sheet complete: {output_dir}/{name}/")
```

### UGC Product Selfie Still

```python
def generate_ugc_selfie(character_ref_path, product_ref_path, scene_desc):
    """
    Combine character + product + UGC aesthetic refs into authentic selfie.
    Includes skin realism and camera imperfections.
    """
    
    import base64
    
    # Load references
    with open(character_ref_path, 'rb') as f:
        char_b64 = base64.b64encode(f.read()).decode('utf-8')
    
    with open(product_ref_path, 'rb') as f:
        prod_b64 = base64.b64encode(f.read()).decode('utf-8')
    
    # Load UGC aesthetic references (up to 5 total)
    ugc_refs = []
    for ref_file in ['ugc-lighting-1.jpg', 'ugc-composition-1.jpg']:
        ref_path = f'references/aesthetics/ugc-selfie/{ref_file}'
        if os.path.exists(ref_path):
            with open(ref_path, 'rb') as f:
                ugc_refs.append(base64.b64encode(f.read()).decode('utf-8'))
    
    all_refs = [char_b64, prod_b64] + ugc_refs[:3]  # Max 5 refs
    
    payload = {
        "model": "nano-banana-2",
        "prompt": f"""
        iPhone 15 Pro selfie, 0.5x front camera. {scene_desc}
        
        Person holds product naturally, slight motion blur on hand.
        Window light from left, natural shadows on face.
        
        Realistic skin texture: pores visible, slight blemishes, natural makeup.
        Camera imperfections: slight lens distortion, auto-focus hunting.
        
        Authentic home environment, visible mess in background.
        Casual clothing, natural expression (not model pose).
        """,
        "referenceImages": all_refs,
        "aspectRatio": "9:16"
    }
    
    response = make_request('/v1/nano-banana/image', payload=payload)
    result = poll_until_complete(response['jobId'])
    return result['imageUrl']
```

### Nano Banana Model Selection

```python
def generate_nano_banana(prompt, model="nano-banana-2", reference_images=None):
    """
    Generate image with Nano Banana model selection:
    - nano-banana-2: Default, balanced quality/speed
    - nano-banana: Nano Banana Pro (Gemini 3 Pro Image) — higher fidelity
    - nano-banana-edit: Inpainting/editing workflow
    """
    
    payload = {
        "model": model,
        "prompt": prompt,
        "aspectRatio": "1:1"
    }
    
    if reference_images:
        payload["referenceImages"] = reference_images
    
    response = make_request('/v1/nano-banana/image', payload=payload)
    return poll_until_complete(response['jobId'])
```

## Static Meta Image Ad Creative (37-Template Library)

### ChatGPT Image Ad — Typography/UI Mimicry

```python
def generate_chatgpt_image_ad(template_name, product_data):
    """
    Generate static Meta ad using ChatGPT Image 2.
    Best for: Apple Notes lists, fake UI screenshots, comparison tables.
    
    See: shared/skills/image-ad-prompting/templates/{template_name}.md
    """
    
    # Load template prompt from library
    template_path = f'shared/skills/image-ad-prompting/templates/{template_name}.md'
    with open(template_path, 'r') as f:
        template_prompt = f.read()
    
    # Inject product data
    prompt = template_prompt.format(**product_data)
    
    payload = {
        "model": "gpt-image-2",
        "prompt": prompt,
        "aspectRatio": "1:1"  # Meta feed standard
    }
    
    response = make_request('/v1/chatgpt-image/ad', payload=payload)
    result = poll_until_complete(response['jobId'])
    
    return result['imageUrl']

# Example usage
product = {
    "name": "SleepGuard Pro",
    "features": ["Blocks 99% blue light", "Adjustable fit", "Lifetime warranty"],
    "price": "$29.99",
    "competitor": "SleepEasy"
}

ad_url = generate_chatgpt_image_ad('apple-notes-list', product)
```

### Nano Banana Image Ad — Photoreal/Lifestyle

```python
def generate_nano_banana_image_ad(template_name, product_data, reference_images=None):
    """
    Generate static Meta ad using Nano Banana 2.
    Best for: Lifestyle shots, product showcases, multi-reference composites.
    """
    
    template_path = f'shared/skills/image-ad-prompting/templates/{template_name}.md'
    with open(template_path, 'r') as f:
        template_prompt = f.read()
    
    prompt = template_prompt.format(**product_data)
    
    payload = {
        "model": "nano-banana-2",
        "prompt": prompt,
        "aspectRatio": "4:5"  # Vertical feed optimal
    }
    
    if reference_images:
        payload["referenceImages"] = reference_images
    
    response = make_request('/v1/nano-banana/image', payload=payload)
    return poll_until_complete(response['jobId'])
```

### Image Ad Clone — Reverse Engineer Existing Ad

```python
def clone_image_ad(source_image_path, backend='auto'):
    """
    Reverse-engineer existing ad into reusable template.
    
    Phase 1: Analyze structure
    Phase 8: Cross-validate against other backend (optional)
    
    See: shared/skills/image-ad-prompting/OVERVIEW.md
    """
    
    import base64
    
    with open(source_image_path, 'rb') as f:
        image_b64 = base64.b64encode(f.read()).decode('utf-8')
    
    # Phase 1: Analyze (vision model extracts structure)
    analysis_payload = {
        "image": image_b64,
        "task": "extract_template_structure"
    }
    
    analysis = make_request('/v1/image-ad/analyze', payload=analysis_payload)
    
    print(f"Detected template type: {analysis['template_type']}")
    print(f"Recommended backend: {analysis['backend']}")
    print(f"Elements: {', '.join(analysis['elements'])}")
    
    # Generate new version with backend
    backend = analysis['backend'] if backend == 'auto' else backend
    
    generate_payload = {
        "model": backend,
        "template": analysis['template_structure'],
        "referenceImage": image_b64,
        "aspectRatio": analysis['aspect_ratio']
    }
    
    response = make_request(f'/v1/{backend}/image', payload=generate_payload)
    return poll_until_complete(response['jobId'])
```

## Multi-Step Animated Ad Pipelines

### Pixar-Style 3D Animated Ad

```bash
#!/bin/bash
# Pixar-style pipeline: Cast sheet → Storyboard → Seedance i2v → Stitch + Captions

PRODUCT="SleepGuard Pro"
MASCOT="anthropomorphized pillow with sleepy eyes"
STORY_BEATS=8

# Phase 1: Lock cast sheet (ChatGPT Image 2 character turnaround)
python3 shared/skills/pixar-style-ad/scripts/generate_cast_sheet.py \
  --mascot "$MASCOT" \
  --output cast_sheet/

# Phase 2: Generate storyboard stills (sequential, prior frame as ref)
python3 shared/skills/pixar-style-ad/scripts/generate_storyboard.py \
  --cast-sheet cast_sheet/ \
  --story "$PRODUCT animation 8-beat arc" \
  --beats $STORY_BEATS \
  --output storyboard/

# Phase 3: Seedance 2.0 image-to-video per beat
for i in $(seq 1 $STORY_BEATS); do
  python3 shared/skills/pixar-style-ad/scripts/animate_beat.py \
    --frame storyboard/beat_${i}.png \
    --duration 8 \
    --output animated/beat_${i}.mp4
done

# Phase 4: Stitch with ffmpeg + burn captions
python3 shared/skills/pixar-style-ad/scripts/stitch_and_caption.py \
  --input animated/ \
  --output final_pixar_ad.mp4 \
  --captions script.srt
```

### Claymation / Aardman-Style Ad

```bash
#!/bin/bash
# Claymation pipeline: 8-beat narrator-driven + clay textures

python3 shared/skills/claymation-ad/scripts/generate_claymation.py \
  --product "SleepGuard Pro" \
  --style "Aardman plasticine texture" \
  --duration 90 \
  --narrator-script "Meet Bob. Bob can't sleep..." \
  --output claymation_ad.mp4

# Post-process: Add stop-motion judder (optional)
ffmpeg -i claymation_ad.mp4 \
  -vf "fps=12,fps=24" \
  claymation_ad_judder.mp4
```

### Caption Burn-In Workflow

```python
def burn_captions(video_path, output_path):
    """
    Add captions to finished video using Whisper + HyperFrames.
    Works on any source (Pixar, claymation, UGC, b-roll).
    """
    
    import subprocess
    
    # Step 1: Transcribe with Whisper
    subprocess.run([
        'whisper', video_path,
        '--model', 'medium.en',
        '--output_format', 'srt',
        '--output_dir', 'temp/'
    ])
    
    srt_path = video_path.replace('.mp4', '.srt')
    
    # Step 2: Group word-level transcript into reading phrases
    subprocess.run([
        'python3', 'shared/skills/caption-video/scripts/group_phrases.py',
        '--input', f'temp/{srt_path}',
        '--output', 'temp/grouped.srt'
    ])
    
    # Step 3: Render captions with HyperFrames
    subprocess.run([
        'npx', 'hyperframes',
        '--video', video_path,
        '--subtitles', 'temp/grouped.srt',
        '--style', 'mr-beast',  # Bold yellow + black stroke
        '--output', output_path
    ])
    
    print(f"✓ Captioned video saved: {output_path}")
```

## Common Patterns

### Cost Confirmation Before Generation

```python
def confirm_cost(estimated_cost, action_desc):
    """Always confirm costs before expensive operations."""
    
    print(f"\n⚠️  {action_desc}")
    print(f"Estimated cost: ${estimated_cost:.2f}")
    
    response = input("Proceed? (y/n): ")
    if response.lower() != 'y':
        raise Exception("Operation cancelled by user")
```

### Batch Generation with Error Recovery

```python
def batch_generate_videos(prompts, model='seedance-2'):
    """Generate multiple videos with error recovery."""
    
    results = []
    failed = []
    
    for idx, prompt in enumerate(prompts, start=1):
        try:
            payload = {
                "model": model,
                "prompt": prompt,
                "duration": 12,
                "aspectRatio": "9:16"
            }
            
            response = make_request('/v1/seedance2/video', payload=payload)
            job_id = response['jobId']
            
            print(f"[{idx}/{len(prompts)}] Submitted: {job_id}")
            
            result = poll_until_complete(job_id)
            results.append({
                'prompt': prompt,
                'video_url': result['videoUrl'],
                'job_id': job_id
            })
            
        except Exception as e:
            print(f"[{idx}/{len(prompts)}] Failed: {e}")
            failed.append({'prompt': prompt, 'error': str(e)})
    
    return results, failed
```

### File Organization Strategy

```python
import os
from datetime import datetime

def organize_output(file_url, project_name, file_type='video'):
    """Organize generated assets by project and date."""
    
    timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
    output_dir = f'output/{project_name}/{file_type}s/{timestamp}'
    
    os.makedirs(output_dir, exist_ok=True)
    
    # Download file
    import requests
    file_data = requests.get(file_url).content
    
    # Determine extension
    ext = 'mp4' if file_type == 'video' else 'png'
    filename = f'{timestamp}.{ext}'
    
    output_path = os.path.join(output_dir, filename)
    
    with open(output_path, 'wb') as f:
        f.write(file_data)
    
    print(f"✓ Saved: {output_path}")
    return output_path
```

## Troubleshooting

### API Key Issues

```python
# Check if API key is loaded
import os

api_key = os.getenv('ARCADS_API_KEY')
if not api_key:
    print("❌ ARCADS_API_KEY not found in environment")
    print("Run: ./scripts/setup.sh")
else:
    print(f"✓ API key loaded: {api_key[:10]}...")
```

### Job Polling Timeout

```python
def poll_with_timeout(job_id, timeout_minutes=10):
    """Poll with timeout to prevent infinite loops."""
    
    import time
    
    start_time = time.time()
    timeout_seconds = timeout_minutes * 60
    
    while True:
        elapsed = time.time() - start_time
        if elapsed > timeout_seconds:
            raise TimeoutError(f"Job {job_id} exceeded {timeout_minutes}min timeout")
        
        status = make_request(f'/v1/jobs/{job_id}', method='GET')
        
        if status['status'] in ['completed', 'failed']:
            return status
        
        time.sleep(5)
```

### Video Generation Failed

```python
def diagnose_failure(job_id):
    """Retrieve detailed error information."""
    
    status = make_request(f'/v1/jobs/{job_id}', method='GET')
    
    print(f"Status: {status['status']}")
    print(f"Error: {status.get('error', 'No error message')}")
    print(f"Error code: {status.get('errorCode', 'N/A')}")
    
    # Common issues
    if 'quota' in status.get('error', '').lower():
        print("\n💡 Likely cause: API quota exceeded")
        print("Solution: Check account credits at app.arcads.ai/settings/billing")
    
    elif 'prompt' in status.get('error', '').lower():
        print("\n💡 Likely cause: Prompt violation")
        print("Solution: Review content policy and rephrase prompt")
    
    elif 'reference' in status.get('error', '').lower():
        print("\n💡 Likely cause: Invalid reference image")
        print("Solution: Check image format (PNG/JPG) and base64 encoding")
```

### Missing Dependencies

```bash
# Check required CLI tools
command -v ffmpeg >/dev/null 2>&1 || echo "❌ ffmpeg not installed"
command -v jq >/dev/null 2>&1 || echo "❌ jq not installed"
command -v node >/dev/null 2>&1 || echo "❌ node not installed"

# Install missing tools
brew install ffmpeg jq node  # macOS
# or
apt install ffmpeg jq nodejs  # Linux
```

### Reference Image Limit

```python
def validate_references(reference_paths):
    """Ensure reference count doesn't exceed API limit."""
    
    MAX_REFS = 5  # Seedance 2.0 and ChatGPT Image 2 limit
    
    if len(reference_paths) > MAX_REFS:
        print(f"⚠️  Too many references: {len(reference_paths)}/{MAX_REFS}")
        print("Using first 5 only")
        return reference_paths[:MAX_REFS]
    
    return reference_paths
```

## Key Endpoints Reference

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/v1/seedance2/video` | POST | Generate Seedance 2.0 video |
| `/v1/sora2/video` | POST | Generate Sora 2 video |
| `/v1/veo3.1/video` | POST | Generate Veo 3.1 video (image-to-video) |
| `/v1/b-roll` | POST | Generate Kling b-roll clip |
| `/v1/scene` | POST | Generate Kling scene |
| `/v1/nano-banana/image` | POST | Generate Nano Banana image |
| `/v1/chatgpt-image/ad` | POST | Generate ChatGPT Image 2 ad |
| `/v1/jobs/{jobId}` | GET | Poll job status |
| `/v2/videos/generate` | POST | Multi-model video endpoint (Grok, etc.) |
| `/v1/omnihuman` | POST | OmniHuman avatar generation |
| `/v1/audio-driven` | POST | Audio-driven lip-sync video |

## Agent Workflow Tips

1. **Always confirm costs** before expensive batch operations
2. **Save reference images** to `references/` for reuse across projects
3. **Use MASTER_CONTEXT.md** to track project state, influencer personas, brand voice
4. **Check aspect ratio compatibility** per backend (see `image-ad-prompting/OVERVIEW.md`)
5. **Validate prompts** against content policy before submission
6. **Poll with timeouts** to prevent infinite loops on stuck jobs
7. **Organize output** by project name and timestamp for client delivery
8. **Test single generations** before running batch jobs
9. **Load aesthetic references** from `references/aesthetics/` for style consistency
10. **Read template docs** in `prompting/prompt-library/` before customizing formulas

## Additional Resources

- **Prompting Library**: `skills/arcads-external-api/prompting/prompt-library/`
- **Image Ad Templates**: `shared/skills/image-ad-prompting/templates/`
- **Video Tutorial**: [YouTube walkthrough](https://youtu.be/HHGQN9Zqaxo) by Mr. Paid Social
- **API Documentation**: [app.arcads.ai/docs](https://app.arcads.ai/docs)
- **Community**: [The AI Ad Alchemists Skool](https://skool.com
