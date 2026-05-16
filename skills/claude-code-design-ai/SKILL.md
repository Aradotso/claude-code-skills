---
name: claude-code-design-ai
description: AI-powered UI/UX design framework for converting screenshots to React, generating Tailwind components, and creating design systems with Claude AI.
triggers:
  - convert this design to react code
  - generate tailwind components from screenshot
  - create a design system with claude
  - build ui components from figma
  - setup claude design framework
  - generate responsive layouts with tailwind
  - create react components from wireframes
  - export design to shadcn ui components
---

# Claude Code Design AI

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## Overview

Claude Design is a TypeScript-based UI/UX framework that leverages Claude AI to transform visual designs into production-ready React components. It specializes in screenshot-to-code conversion, Figma component generation, Tailwind CSS styling, and design system creation. The framework integrates with shadcn/ui and provides tools for prototyping, dark mode, responsive layouts, and vector asset management.

## Installation

```bash
# Clone the repository
git clone https://github.com/mikesheehan54/Claude-Code-Design-AI.git
cd Claude-Code-Design-AI

# Install dependencies
npm install

# Set up environment variables
cp .env.example .env
```

## Configuration

Create a `.env` file in the project root:

```env
ANTHROPIC_API_KEY=your_api_key_here
VITE_API_BASE_URL=https://api.anthropic.com
VITE_MODEL=claude-3-5-sonnet-20241022
```

For TypeScript configuration, ensure your `tsconfig.json` includes:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

## Core Features

### Screenshot to React Component

Convert design screenshots into React components with Tailwind CSS:

```typescript
import { DesignConverter } from '@/lib/design-converter';
import { ImageAnalyzer } from '@/lib/image-analyzer';

async function convertScreenshotToComponent(imageUrl: string) {
  const analyzer = new ImageAnalyzer(process.env.ANTHROPIC_API_KEY!);
  
  // Analyze the screenshot
  const designAnalysis = await analyzer.analyzeDesign(imageUrl);
  
  // Generate React component
  const converter = new DesignConverter();
  const component = await converter.toReactComponent(designAnalysis, {
    framework: 'react',
    styling: 'tailwind',
    typescript: true,
    componentName: 'GeneratedComponent'
  });
  
  return component;
}

// Usage
const reactCode = await convertScreenshotToComponent('/path/to/screenshot.png');
console.log(reactCode);
```

### Design System Generator

Create comprehensive design systems from visual inputs:

```typescript
import { DesignSystemBuilder } from '@/lib/design-system';
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY!
});

async function generateDesignSystem(brandColors: string[], typography: any) {
  const builder = new DesignSystemBuilder(client);
  
  const designSystem = await builder.create({
    colors: {
      primary: brandColors[0],
      secondary: brandColors[1],
      accent: brandColors[2]
    },
    typography: {
      fontFamily: typography.primary,
      scale: 'modular'
    },
    spacing: 'tailwind',
    components: ['button', 'input', 'card', 'modal']
  });
  
  // Export to Tailwind config
  const tailwindConfig = builder.exportToTailwind(designSystem);
  
  return { designSystem, tailwindConfig };
}
```

### Tailwind CSS Generator

Generate utility-first CSS from design tokens:

```typescript
import { TailwindGenerator } from '@/lib/tailwind-generator';

const generator = new TailwindGenerator();

// Generate Tailwind classes from design tokens
const styles = generator.fromTokens({
  backgroundColor: '#1a202c',
  textColor: '#ffffff',
  padding: { top: 16, right: 24, bottom: 16, left: 24 },
  borderRadius: 8,
  boxShadow: 'md'
});

console.log(styles);
// Output: "bg-gray-900 text-white px-6 py-4 rounded-lg shadow-md"

// Generate complete component styles
const buttonStyles = generator.generateComponent('button', {
  variant: 'primary',
  size: 'lg',
  withIcon: true
});
```

### Figma to React Conversion

Import Figma components and convert to React:

```typescript
import { FigmaImporter } from '@/lib/figma-importer';

async function importFromFigma(fileKey: string, nodeId: string) {
  const importer = new FigmaImporter({
    accessToken: process.env.FIGMA_ACCESS_TOKEN!,
    claudeApiKey: process.env.ANTHROPIC_API_KEY!
  });
  
  // Fetch Figma component
  const figmaNode = await importer.fetchNode(fileKey, nodeId);
  
  // Convert to React with shadcn/ui
  const component = await importer.toReact(figmaNode, {
    useShadcn: true,
    includeVariants: true,
    exportFormat: 'tsx'
  });
  
  return component;
}
```

### Responsive Layout Generation

Create responsive layouts with AI assistance:

```typescript
import { LayoutGenerator } from '@/lib/layout-generator';

async function createResponsiveLayout(layoutType: string) {
  const generator = new LayoutGenerator(process.env.ANTHROPIC_API_KEY!);
  
  const layout = await generator.generate({
    type: layoutType, // 'dashboard', 'landing', 'blog', etc.
    breakpoints: ['mobile', 'tablet', 'desktop'],
    gridSystem: 'tailwind',
    sections: ['header', 'main', 'sidebar', 'footer']
  });
  
  return layout.code;
}

// Usage
const dashboardLayout = await createResponsiveLayout('dashboard');
```

### Dark Mode Implementation

Toggle and generate dark mode variants:

```typescript
import { DarkModeGenerator } from '@/lib/dark-mode';

const darkMode = new DarkModeGenerator();

// Generate dark mode variant
const component = `
<div className="bg-white text-gray-900 p-4">
  <h1 className="text-2xl font-bold">Title</h1>
  <p className="text-gray-600">Description</p>
</div>
`;

const withDarkMode = darkMode.addDarkMode(component);
console.log(withDarkMode);
// Adds dark: variants to all color classes

// Setup dark mode toggle hook
import { useDarkMode } from '@/hooks/use-dark-mode';

function App() {
  const { theme, toggleTheme } = useDarkMode();
  
  return (
    <div className={theme === 'dark' ? 'dark' : ''}>
      <button onClick={toggleTheme}>Toggle Theme</button>
    </div>
  );
}
```

### SVG Icon Creator

Generate custom SVG icons with AI:

```typescript
import { IconGenerator } from '@/lib/icon-generator';

async function createCustomIcon(description: string) {
  const generator = new IconGenerator(process.env.ANTHROPIC_API_KEY!);
  
  const icon = await generator.create({
    description,
    size: 24,
    strokeWidth: 2,
    style: 'outline', // or 'solid', 'duotone'
    format: 'react-component'
  });
  
  return icon;
}

// Usage
const uploadIcon = await createCustomIcon('upload arrow pointing upward with cloud');

// Export as React component
const IconComponent = ({ className }: { className?: string }) => (
  <svg className={className} width="24" height="24" viewBox="0 0 24 24">
    {/* Generated SVG paths */}
  </svg>
);
```

### Streaming UI Components

Handle Claude's streaming responses in UI:

```typescript
import { useClaudeStream } from '@/hooks/use-claude-stream';
import { StreamingText } from '@/components/streaming-text';

function ChatInterface() {
  const { stream, isStreaming, response } = useClaudeStream({
    model: 'claude-3-5-sonnet-20241022',
    apiKey: process.env.ANTHROPIC_API_KEY!
  });
  
  const handleGenerate = async (prompt: string) => {
    await stream({
      messages: [
        { role: 'user', content: prompt }
      ],
      max_tokens: 4096
    });
  };
  
  return (
    <div className="chat-container">
      <StreamingText 
        content={response}
        isStreaming={isStreaming}
        className="prose dark:prose-invert"
      />
    </div>
  );
}
```

### shadcn/ui Integration

Export components using shadcn/ui conventions:

```typescript
import { ShadcnExporter } from '@/lib/shadcn-exporter';

const exporter = new ShadcnExporter();

// Convert to shadcn/ui component
const shadcnComponent = exporter.toShadcn({
  name: 'CustomButton',
  baseComponent: 'button',
  variants: {
    variant: {
      default: 'bg-primary text-primary-foreground hover:bg-primary/90',
      destructive: 'bg-destructive text-destructive-foreground hover:bg-destructive/90',
      outline: 'border border-input bg-background hover:bg-accent'
    },
    size: {
      default: 'h-10 px-4 py-2',
      sm: 'h-9 rounded-md px-3',
      lg: 'h-11 rounded-md px-8'
    }
  }
});

// Export to file
await exporter.writeToFile(shadcnComponent, './components/ui/custom-button.tsx');
```

## Common Patterns

### Complete Design-to-Code Workflow

```typescript
import { ClaudeDesignWorkflow } from '@/lib/workflow';

async function completeWorkflow(designInput: string) {
  const workflow = new ClaudeDesignWorkflow({
    anthropicApiKey: process.env.ANTHROPIC_API_KEY!,
    figmaToken: process.env.FIGMA_ACCESS_TOKEN
  });
  
  // 1. Analyze design
  const analysis = await workflow.analyzeDesign(designInput);
  
  // 2. Extract design tokens
  const tokens = await workflow.extractTokens(analysis);
  
  // 3. Generate components
  const components = await workflow.generateComponents(tokens, {
    framework: 'react',
    styling: 'tailwind',
    includeTests: true
  });
  
  // 4. Create design system
  const designSystem = await workflow.createDesignSystem(tokens);
  
  // 5. Export everything
  await workflow.export('./output', {
    components,
    designSystem,
    format: 'typescript'
  });
  
  return { components, designSystem };
}
```

### Building Reusable Component Library

```typescript
import { ComponentLibraryBuilder } from '@/lib/library-builder';

const builder = new ComponentLibraryBuilder({
  name: 'my-design-system',
  prefix: 'mds',
  baseTheme: 'light'
});

// Add components
builder.addComponent('Button', buttonConfig);
builder.addComponent('Input', inputConfig);
builder.addComponent('Card', cardConfig);

// Generate package
await builder.build({
  outputDir: './lib',
  includeStorybook: true,
  includeDocs: true
});
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 50,
  windowMs: 60000 // 1 minute
});

async function makeClaudeRequest(prompt: string) {
  await limiter.wait();
  
  try {
    // Make your API call
    return await client.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      messages: [{ role: 'user', content: prompt }],
      max_tokens: 1024
    });
  } catch (error) {
    if (error.status === 429) {
      console.log('Rate limited, retrying...');
      await new Promise(resolve => setTimeout(resolve, 5000));
      return makeClaudeRequest(prompt);
    }
    throw error;
  }
}
```

### Image Processing Issues

```typescript
// Ensure images are properly encoded for Claude
import { ImageProcessor } from '@/lib/image-processor';

const processor = new ImageProcessor();

async function processImage(imagePath: string) {
  // Optimize image size
  const optimized = await processor.optimize(imagePath, {
    maxWidth: 1568,
    maxHeight: 1568,
    quality: 85
  });
  
  // Convert to base64
  const base64 = await processor.toBase64(optimized);
  
  return {
    type: 'image',
    source: {
      type: 'base64',
      media_type: 'image/png',
      data: base64
    }
  };
}
```

### Component Generation Errors

```typescript
// Add validation and error handling
import { ComponentValidator } from '@/lib/validator';

const validator = new ComponentValidator();

async function generateAndValidate(design: any) {
  const component = await generateComponent(design);
  
  const validation = validator.validate(component, {
    checkSyntax: true,
    checkAccessibility: true,
    checkResponsiveness: true
  });
  
  if (!validation.isValid) {
    console.error('Validation errors:', validation.errors);
    // Auto-fix common issues
    return validator.autoFix(component, validation.errors);
  }
  
  return component;
}
```

## CLI Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Run type checking
npm run type-check

# Generate component from design file
npx claude-design convert ./design.png --output ./components

# Create design system
npx claude-design system --colors primary:#3b82f6 secondary:#8b5cf6

# Export to shadcn
npx claude-design export --format shadcn --output ./components/ui
```

## Best Practices

1. **Always validate generated code** before using in production
2. **Cache Claude responses** to avoid redundant API calls
3. **Use environment variables** for all API keys and secrets
4. **Test components** across different screen sizes
5. **Optimize images** before sending to Claude API (< 5MB recommended)
6. **Version control** your design tokens and component configurations
7. **Use TypeScript** for type safety in generated components
