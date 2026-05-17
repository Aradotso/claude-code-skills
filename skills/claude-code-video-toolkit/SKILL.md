---
name: claude-code-video-toolkit
description: AI-native video production toolkit for programmatic video creation with Remotion, ElevenLabs, and cloud AI tools
triggers:
  - create a video with remotion
  - generate ai voiceover for video
  - set up video production toolkit
  - render video with claude code
  - create video template
  - generate speech from script
  - record browser demo as video
  - create sprint review video
---

# claude-code-video-toolkit

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## Overview

The **claude-code-video-toolkit** is a complete AI-native video production workspace that integrates Remotion (React-based video), ElevenLabs/Qwen3-TTS (AI voiceover), FLUX.2/LTX-2 (AI image/video generation), ACE-Step (AI music), and cloud GPU deployment for creating professional explainer videos programmatically.

The toolkit provides:
- **Skills** for Remotion, ElevenLabs, FFmpeg, Playwright recording, AI image/video generation
- **Commands** (`/video`, `/setup`, `/template`, `/scene-review`) for guided workflows
- **Templates** (sprint-review, product-demo) with scene-based composition
- **Brand profiles** for visual identity consistency
- **Project management** system tracking scenes, audio, and workflow phases
- **Python tools** for voiceover generation, image editing, music composition

## Installation

```bash
git clone https://github.com/digitalsamba/claude-code-video-toolkit.git
cd claude-code-video-toolkit

# Install Python dependencies (optional, for AI tools)
python3 -m pip install -r tools/requirements.txt

# Open in Claude Code
claude
```

### Prerequisites

- **Node.js** 18+ (for Remotion)
- **Python** 3.9+ (for AI tools)
- **FFmpeg** (optional, for media processing)
- **Claude Code** or compatible AI agent

## Quick Start

```bash
# First-time setup (cloud GPU, voice, storage)
/setup

# Create your first video
/video

# Or render example immediately (no API keys needed)
cd examples/hello-world
npm install
npm run render
```

## Core Commands

### `/setup`
Interactive first-time configuration:
- Cloud GPU provider (Modal or RunPod)
- File transfer method (Cloudflare R2, SSH, local)
- Voice configuration (ElevenLabs or Qwen3-TTS)
- Dependency checks

### `/video`
Video project management:
- List existing projects
- Resume in-progress projects
- Create new project from template
- Guided workflow through phases: planning → assets → review → audio → editing → rendering

### `/template`
Template management:
- List available templates
- Create new template from scratch
- Templates define scene structure, components, and styling

### `/scene-review`
Launch Remotion Studio for scene-by-scene review and editing:
```bash
cd projects/my-project
npm run studio
```

### `/brand`
Brand profile management:
- List, edit, or create brand profiles
- Brands define colors, fonts, typography, voice settings

### `/generate-voiceover`
Generate AI voiceover from script:
```bash
/generate-voiceover
# Or directly:
python tools/voiceover.py --script script.md --output audio.mp3
```

### `/record-demo`
Record browser interactions with Playwright:
```bash
/record-demo
# Generates MP4 from automated browser session
```

## Project Structure

```
claude-code-video-toolkit/
├── .claude/               # Claude Code skills and commands
│   ├── skills/           # Skill definitions (remotion, elevenlabs, etc.)
│   └── commands/         # Command definitions (/video, /setup, etc.)
├── projects/             # Video projects (gitignored)
├── templates/            # Project templates
│   ├── sprint-review/
│   └── product-demo/
├── brands/               # Brand profiles
│   ├── default/
│   └── digital-samba/
├── lib/                  # Shared libraries
│   ├── transitions/      # Scene transition effects
│   ├── project/          # Project management system
│   └── components/       # Reusable Remotion components
├── tools/                # Python AI tools
│   ├── voiceover.py      # Generate AI speech
│   ├── qwen_edit.py      # AI image editing
│   ├── acestep.py        # AI music generation
│   └── ltx2.py           # AI video generation
└── examples/             # Example projects
```

## Creating a Video Project

### 1. Start a New Project

```bash
/video
# Select "Create new project"
# Choose template (e.g., sprint-review-v2)
# Specify brand profile
# Answer planning questions
```

### 2. Define Scenes

The toolkit uses a scene-based workflow. Each scene has:
- **Intent**: What to show (description)
- **Visual type**: demo-recording, stats-screen, title-screen, etc.
- **Duration**: Length in seconds
- **Assets**: Required files (videos, images, audio)

Example `project.json` scene:
```json
{
  "intent": "Show iOS app login screen with animation",
  "visualType": "demo-recording",
  "duration": 8,
  "assets": {
    "video": {
      "status": "needed",
      "path": null
    }
  }
}
```

### 3. Generate Assets

```bash
# Record demo with Playwright
/record-demo

# Generate AI voiceover
python tools/voiceover.py \
  --script projects/my-project/audio/script.md \
  --output projects/my-project/audio/voiceover.mp3 \
  --voice $ELEVENLABS_VOICE_ID

# Generate AI image
python tools/qwen_edit.py \
  --prompt "futuristic tech background, dark blue" \
  --output projects/my-project/assets/background.png

# Generate AI music
python tools/acestep.py \
  --prompt "upbeat electronic, tech demo" \
  --duration 60 \
  --output projects/my-project/audio/music.mp3
```

### 4. Review in Remotion Studio

```bash
cd projects/my-project
npm run studio
# Opens http://localhost:3000 for live preview
```

### 5. Render Final Video

```bash
cd projects/my-project
npm run render
# Outputs to out/video.mp4
```

## Remotion Scene Composition

### Basic Scene Structure

```typescript
// src/scenes/IntroScene.tsx
import { AbsoluteFill, useCurrentFrame } from 'remotion';
import { interpolate } from 'remotion';

export const IntroScene: React.FC<{ title: string }> = ({ title }) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(frame, [0, 30], [0, 1]);
  
  return (
    <AbsoluteFill style={{ 
      backgroundColor: '#0a0e27',
      justifyContent: 'center',
      alignItems: 'center'
    }}>
      <h1 style={{ 
        opacity,
        fontSize: 72,
        color: '#00d4ff'
      }}>
        {title}
      </h1>
    </AbsoluteFill>
  );
};
```

### Using Transitions

```typescript
import { Series } from 'remotion';
import { glitch, fade } from '../../lib/transitions';
import { IntroScene } from './scenes/IntroScene';
import { DemoScene } from './scenes/DemoScene';

export const Video: React.FC = () => (
  <Series>
    <Series.Sequence durationInFrames={90}>
      <IntroScene title="My Product" />
    </Series.Sequence>
    <Series.Transition
      presentation={glitch()}
      timing={{ durationInFrames: 15 }}
    />
    <Series.Sequence durationInFrames={240}>
      <DemoScene videoSrc="assets/demo.mp4" />
    </Series.Sequence>
  </Series>
);
```

### Available Transitions

```typescript
import { 
  glitch,        // Digital distortion
  rgbSplit,      // Chromatic aberration
  zoomBlur,      // Radial motion blur
  lightLeak,     // Lens flare
  clockWipe,     // Radial sweep
  pixelate,      // Mosaic dissolution
  checkerboard,  // Grid reveal (9 patterns)
  slide,         // Official Remotion
  fade,          // Official Remotion
  wipe,          // Official Remotion
  flip           // Official Remotion
} from '../../lib/transitions';
```

## AI Voiceover Generation

### ElevenLabs (Cloud API)

```python
# tools/voiceover.py
import os
from elevenlabs.client import ElevenLabs

client = ElevenLabs(api_key=os.getenv("ELEVENLABS_API_KEY"))

audio = client.generate(
    text="Welcome to our product demo",
    voice="EXAVITQu4vr4xnSDxMaL",  # Voice ID
    model="eleven_multilingual_v2"
)

with open("voiceover.mp3", "wb") as f:
    for chunk in audio:
        f.write(chunk)
```

### Qwen3-TTS (Self-hosted, free)

```bash
# Deploy to Modal
modal deploy tools/qwen_tts_modal.py

# Generate speech
python tools/voiceover.py \
  --script script.md \
  --output audio.mp3 \
  --provider qwen
```

### Script Format

```markdown
# script.md

In this sprint, we shipped three major features.

[pause:1.0]

First, we rebuilt the authentication system with biometric support.

[emphasis]This was a big win for security.[/emphasis]

[pause:0.5]

Let's see it in action.
```

## AI Image Generation & Editing

### Generate with FLUX.2

```python
# tools/qwen_edit.py (uses FLUX.2 internally)
python tools/qwen_edit.py \
  --prompt "cyberpunk city at night, neon lights, rain" \
  --output background.png \
  --width 1920 \
  --height 1080
```

### Edit Existing Image

```python
python tools/qwen_edit.py \
  --input photo.jpg \
  --prompt "add sunglasses and a leather jacket" \
  --output edited.png
```

## AI Music Generation

```python
# tools/acestep.py
python tools/acestep.py \
  --prompt "upbeat electronic music for tech demo" \
  --duration 60 \
  --output music.mp3 \
  --lyrics "optional verse lyrics here"
```

## Brand Profiles

### Creating a Brand

```bash
/brand
# Select "Create new brand"
```

Brand structure:
```
brands/my-brand/
├── brand.json      # Visual identity
├── voice.json      # Voice settings
└── assets/
    ├── logo.png
    └── background.jpg
```

### brand.json Example

```json
{
  "name": "My Brand",
  "colors": {
    "primary": "#00d4ff",
    "secondary": "#ff006e",
    "background": "#0a0e27",
    "text": "#ffffff"
  },
  "fonts": {
    "heading": "Inter",
    "body": "Inter"
  },
  "typography": {
    "headingSize": 72,
    "bodySize": 24
  }
}
```

### voice.json Example

```json
{
  "provider": "elevenlabs",
  "voiceId": "EXAVITQu4vr4xnSDxMaL",
  "stability": 0.5,
  "similarityBoost": 0.75,
  "model": "eleven_multilingual_v2"
}
```

## Cloud GPU Deployment

### Modal (Recommended)

```bash
# Install Modal
pip install modal

# Set up token
modal token new

# Deploy AI tools
modal deploy tools/qwen_tts_modal.py
modal deploy tools/qwen_edit_modal.py
modal deploy tools/acestep_modal.py
```

### RunPod (Alternative)

```bash
# Configure RunPod endpoint
export RUNPOD_API_KEY=your_key
export RUNPOD_ENDPOINT_ID=your_endpoint

# Use with tools
python tools/voiceover.py --provider qwen --runpod
```

## Project Lifecycle

Projects follow this phase progression:

1. **planning** - Define scenes, assets, voiceover script
2. **assets** - Generate/record videos, images, audio
3. **review** - Preview in Remotion Studio, refine timing
4. **audio** - Generate final voiceover and music
5. **editing** - Final polish, transitions, effects
6. **rendering** - Export final video
7. **complete** - Archived

Check project status:
```bash
cat projects/my-project/project.json | grep phase
```

## Common Patterns

### Overlay Text on Video

```typescript
import { Video, AbsoluteFill } from 'remotion';

export const OverlayScene: React.FC = () => (
  <AbsoluteFill>
    <Video src="demo.mp4" />
    <AbsoluteFill style={{ 
      justifyContent: 'flex-end',
      padding: 40 
    }}>
      <h2 style={{ color: 'white', fontSize: 48 }}>
        Feature Demo
      </h2>
    </AbsoluteFill>
  </AbsoluteFill>
);
```

### Animated Stats

```typescript
import { useCurrentFrame } from 'remotion';
import { interpolate } from 'remotion';

export const StatsScene: React.FC = () => {
  const frame = useCurrentFrame();
  const value = Math.floor(interpolate(frame, [0, 60], [0, 1247]));
  
  return (
    <AbsoluteFill style={{ 
      justifyContent: 'center',
      alignItems: 'center'
    }}>
      <div style={{ fontSize: 120, fontWeight: 'bold' }}>
        {value.toLocaleString()}
      </div>
      <div style={{ fontSize: 36, marginTop: 20 }}>
        GitHub Stars
      </div>
    </AbsoluteFill>
  );
};
```

### Audio Sync

```typescript
import { Audio, useCurrentFrame } from 'remotion';

export const NarratedScene: React.FC = () => {
  const frame = useCurrentFrame();
  const fps = 30;
  
  // Show text at specific timestamps
  const showText = frame > fps * 2.5 && frame < fps * 5;
  
  return (
    <AbsoluteFill>
      <Audio src="voiceover.mp3" />
      {showText && <h2>This appears during narration</h2>}
    </AbsoluteFill>
  );
};
```

## Environment Variables

```bash
# ElevenLabs (optional)
export ELEVENLABS_API_KEY=your_key
export ELEVENLABS_VOICE_ID=voice_id

# Modal (recommended for AI tools)
export MODAL_TOKEN_ID=your_token
export MODAL_TOKEN_SECRET=your_secret

# RunPod (alternative)
export RUNPOD_API_KEY=your_key
export RUNPOD_ENDPOINT_ID=endpoint_id

# Cloudflare R2 (optional, for file transfer)
export R2_ACCOUNT_ID=account_id
export R2_ACCESS_KEY_ID=access_key
export R2_SECRET_ACCESS_KEY=secret_key
export R2_BUCKET_NAME=bucket_name
```

## Troubleshooting

### Remotion render fails with memory error
```bash
# Increase Node memory
export NODE_OPTIONS="--max-old-space-size=8192"
npm run render
```

### ElevenLabs voice sounds robotic
- Reduce `stability` (0.3-0.5 for more natural)
- Increase `similarityBoost` (0.7-0.9)
- Use `eleven_multilingual_v2` model

### Modal deployment fails
```bash
# Reinstall Modal
pip install --upgrade modal

# Check authentication
modal token new

# Verify deployment
modal app list
```

### Video preview is choppy in Remotion Studio
- Reduce preview quality in Studio settings
- Use proxy videos (lower resolution) during editing
- Increase RAM available to Node.js

### Transitions not working
```bash
# Ensure @remotion/transitions is installed
cd projects/my-project
npm install @remotion/transitions
```

### Voice clone quality is poor
- Use at least 1 minute of clear audio
- Remove background noise
- Speak at consistent volume and pace
- Use ElevenLabs professional voice cloning (paid tier)

## Further Resources

- [Remotion Documentation](https://www.remotion.dev/docs)
- [ElevenLabs API Reference](https://elevenlabs.io/docs)
- [Modal Documentation](https://modal.com/docs)
- [Toolkit Examples](https://github.com/digitalsamba/claude-code-video-toolkit/tree/main/examples)
- [Transitions Preview](https://github.com/digitalsamba/claude-code-video-toolkit/tree/main/showcase/transitions)
