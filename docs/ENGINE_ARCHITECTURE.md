# COCOS 4 Engine Architecture Overview

## Introduction

COCOS 4 is a modern, high-performance, cross-platform game engine built on a hybrid C++/TypeScript architecture. This document provides a comprehensive overview of the engine's architecture, core components, and design principles.

## Engine Philosophy

COCOS 4 follows a "write once, run anywhere" philosophy with:
- **Hybrid Architecture**: Performance-critical code in C++, game logic in TypeScript
- **Modern Graphics**: GFX abstraction layer supporting Vulkan, Metal, WebGL, and WebGPU
- **Customizable Pipelines**: Fully customizable render pipelines
- **Open Source**: MIT licensed, community-driven development

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Game Application Layer                   │
│                    (TypeScript/JavaScript)                   │
├─────────────────────────────────────────────────────────────┤
│                    Engine Core (TypeScript)                  │
│  ┌──────────┬──────────┬───────────┬──────────┬──────────┐ │
│  │   Core   │   2D/3D  │ Animation │ Physics  │    UI    │ │
│  │  Systems │  Rendering│  System   │ Engines  │  System  │ │
│  └──────────┴──────────┴───────────┴──────────┴──────────┘ │
├─────────────────────────────────────────────────────────────┤
│              JavaScript Bindings (JSB Layer)                 │
├─────────────────────────────────────────────────────────────┤
│                   Native Layer (C++)                         │
│  ┌──────────┬──────────┬───────────┬──────────┬──────────┐ │
│  │   GFX    │  Render  │  Scene    │  Asset   │ Platform │ │
│  │  Device  │ Pipeline │   Graph   │  Manager │ Adapter  │ │
│  └──────────┴──────────┴───────────┴──────────┴──────────┘ │
├─────────────────────────────────────────────────────────────┤
│              Graphics API Abstraction (GFX)                  │
│         Vulkan | Metal | WebGL | WebGPU | GLES              │
└─────────────────────────────────────────────────────────────┘
```

## Core Components

### 1. Core Module (`cocos/core`)

The foundation of the engine providing essential utilities and systems.

**Key Features:**
- **Math Library**: Vectors, matrices, quaternions, and geometric primitives
- **Memory Operations**: Efficient memory pooling and management
- **Event System**: Centralized event dispatching and handling
- **Data Structures**: Optimized data structures (curves, algorithms)
- **Settings System**: Engine-wide configuration management
- **Scheduler**: Frame-based task scheduling
- **Platform Abstraction**: Cross-platform utilities

**Code Organization:**
```
cocos/core/
├── math/              # Mathematical utilities
├── value-types/       # Core value types (Vec3, Quat, etc.)
├── memop/             # Memory operations
├── event/             # Event system
├── data/              # Data decorators and serialization
├── utils/             # Utility functions
├── platform/          # Platform-specific code
├── curves/            # Animation curves
├── algorithm/         # Core algorithms
└── geometry/          # Geometric primitives
```

### 2. Graphics Abstraction Layer (GFX) (`cocos/gfx`)

The GFX module provides a unified interface to multiple graphics APIs.

**Supported Backends:**
- **Vulkan** (Windows, Android)
- **Metal** (macOS, iOS)
- **WebGL 2.0** (Web platforms)
- **WebGPU** (Modern web browsers)
- **OpenGL ES** (Legacy mobile)

**Key Classes:**
- `Device`: Main graphics device interface
- `CommandBuffer`: Records GPU commands
- `Buffer`: GPU buffer management
- `Texture`: Texture resource management
- `Shader`: Shader compilation and management
- `RenderPass`: Render pass configuration
- `Framebuffer`: Render target management
- `Pipeline`: Graphics/compute pipeline state

**Design Pattern:**
The GFX layer uses the **Abstract Factory** pattern, where each backend implements the same interface with platform-specific optimizations.

```typescript
// Abstract interface
interface Device {
    createBuffer(info: BufferInfo): Buffer;
    createTexture(info: TextureInfo): Texture;
    // ... other resource creation methods
}

// Platform-specific implementations
class VulkanDevice implements Device { /* ... */ }
class MetalDevice implements Device { /* ... */ }
class WebGLDevice implements Device { /* ... */ }
```

### 3. Rendering System (`cocos/rendering`)

The rendering system implements customizable render pipelines.

**Architecture:**
- **Render Pipeline**: Orchestrates the entire rendering process
- **Render Flow**: Defines rendering stages
- **Render Stage**: Individual rendering phases
- **Render Queue**: Batches and sorts render commands
- **Custom Pipeline**: User-defined rendering pipelines

**Built-in Pipelines:**
1. **Forward Pipeline**: Traditional forward rendering
2. **Deferred Pipeline**: Deferred shading for complex lighting
3. **Post-Process**: Post-processing effects

**Key Features:**
- Customizable render passes
- Automatic batching and instancing
- Culling and visibility determination
- Shadow mapping
- Reflection probes
- Light probes and global illumination

### 4. Scene Graph (`cocos/scene-graph`)

Hierarchical scene organization using a node-based system.

**Key Concepts:**
- **Node**: Basic scene graph element with transform
- **Component**: Modular functionality attached to nodes
- **Scene**: Root container for game objects
- **Layers**: Visibility and rendering layers

**Design Pattern:** **Entity-Component System (ECS)** variant where nodes act as entities and components provide functionality.

### 5. Render Scene (`cocos/render-scene`)

High-level rendering abstractions.

**Key Classes:**
- `Model`: Renderable mesh with materials
- `Camera`: View and projection configuration
- `Light`: Various light types (directional, point, spot)
- `Material`: Shader + parameters
- `Pass`: Rendering technique

### 6. 2D Rendering (`cocos/2d`)

Optimized 2D rendering system with batching.

**Features:**
- Sprite rendering with atlases
- UI components (labels, buttons, layouts)
- Particle systems
- Tilemaps
- Spine animations
- DragonBones support

**Optimization:**
- Automatic sprite batching
- Draw call minimization
- Dynamic atlasing

### 7. 3D Rendering (`cocos/3d`)

Advanced 3D features and components.

**Features:**
- Skeletal animation with skinning
- Morph target animations
- LOD (Level of Detail) system
- Terrain rendering
- Particle systems (3D)

### 8. Animation System (`cocos/animation`)

Flexible animation framework.

**Features:**
- Keyframe animations
- Animation curves (Bezier, linear, step)
- Animation clips and states
- Animation blending
- Animation events

### 9. Physics Engines (`cocos/physics`, `cocos/physics-2d`)

Integrated physics simulations.

**2D Physics:**
- Box2D integration
- Rigid bodies, colliders
- Joints and constraints

**3D Physics:**
- Bullet (built-in)
- PhysX (optional)
- Cannon.js (web fallback)

### 10. Audio System (`cocos/audio`)

Cross-platform audio playback.

**Features:**
- Multiple audio sources
- 3D spatial audio
- Audio mixing
- Platform-specific backends

### 11. Asset Management (`cocos/asset`)

Resource loading and management.

**Key Classes:**
- `Asset`: Base asset class
- `AssetManager`: Central asset loading
- `Bundle`: Asset packaging
- `Cache`: Resource caching

**Asset Types:**
- Textures
- Meshes
- Materials
- Animations
- Audio clips
- Prefabs
- Scenes

### 12. Platform Abstraction Layer (PAL) (`pal/`)

Platform-specific implementations.

**Abstractions:**
- File system
- Input handling
- Screen management
- System information
- Audio backend
- Network

## Root Manager

The `Root` class (`cocos/root.ts`) is the central manager:

```typescript
class Root {
    device: Device;              // GFX device
    mainWindow: RenderWindow;    // Main render target
    scenes: RenderScene[];       // Active scenes
    pipeline: PipelineRuntime;   // Render pipeline
    dataPoolManager: DataPoolManager; // Resource pools
    
    initialize(info: IRootInfo): boolean;
    tick(deltaTime: number): void;
    frameMove(deltaTime: number): void;
}
```

**Responsibilities:**
- Initialize graphics device
- Manage render pipeline
- Coordinate scene updates
- Handle render windows
- Manage global resources

## Data Flow

### 1. Initialization Flow
```
Application Start
    ↓
Initialize Root
    ↓
Create Device (GFX)
    ↓
Create Render Pipeline
    ↓
Load Initial Scene
    ↓
Start Game Loop
```

### 2. Frame Update Flow
```
Frame Start
    ↓
Input Processing
    ↓
Game Logic Update (TypeScript)
    ↓
Animation Update
    ↓
Physics Simulation
    ↓
Scene Culling
    ↓
Render Queue Building
    ↓
Command Buffer Recording
    ↓
GPU Submission
    ↓
Present Frame
```

### 3. Render Pipeline Flow
```
Begin Frame
    ↓
Update UBOs (Uniforms)
    ↓
Shadow Pass (if needed)
    ↓
Reflection Pass (if needed)
    ↓
Main Render Pass
    ├─ Opaque Objects
    ├─ Transparent Objects
    └─ UI Elements
    ↓
Post-Processing
    ↓
Present
```

## Module Organization

COCOS 4 uses a modular architecture with clear separation:

```
cocos/
├── core/           # Foundation (math, events, utils)
├── gfx/            # Graphics abstraction
├── rendering/      # Render pipeline
├── scene-graph/    # Node hierarchy
├── render-scene/   # High-level rendering
├── 2d/             # 2D specific
├── 3d/             # 3D specific
├── animation/      # Animation system
├── physics/        # 3D physics
├── physics-2d/     # 2D physics
├── audio/          # Audio system
├── ui/             # UI framework
├── video/          # Video playback
├── asset/          # Asset management
├── tween/          # Tweening
├── particle/       # Particle systems
├── terrain/        # Terrain rendering
├── xr/             # XR/VR support
└── game/           # Game orchestration
```

Each module has a `category.json` for metadata and an `index.ts` for exports.

## Build System

### TypeScript Compilation
- **Compiler**: TypeScript 4.9.5
- **Target**: ES6
- **Module**: CommonJS
- **Transpiler**: Babel for additional transformations
- **Bundler**: Browserify for web builds

### Build Scripts
```bash
npm run build           # Production build
npm run build:dev       # Development build
npm run build:min       # Minified build
npm run build:declaration # Type declarations
```

### Native Builds
Native platforms use CMake build system with platform-specific tools:
- **iOS/macOS**: Xcode
- **Android**: Gradle + NDK
- **Windows**: Visual Studio / MinGW

## Testing Infrastructure

### Test Framework
- **Runner**: Jest
- **Environment**: jsdom for DOM testing
- **Coverage**: Built-in Jest coverage
- **TypeScript**: ts-jest integration

### Test Organization
```
tests/
├── core/              # Core module tests
├── curves/            # Curve tests
├── asset-manager/     # Asset loading tests
├── rendering/         # Rendering tests
└── utils/             # Test utilities
```

### Running Tests
```bash
npm test               # Run all tests
npm run test:dts       # Test type declarations
```

## Performance Considerations

### 1. Memory Management
- Object pooling for frequently created/destroyed objects
- Efficient TypedArray usage for numeric data
- Manual memory management in C++ layer

### 2. Rendering Optimization
- Automatic batching and instancing
- Frustum culling
- Occlusion culling
- LOD system
- Texture atlasing

### 3. Code Optimization
- Hot path optimization in C++
- Minimal JavaScript allocations in game loop
- Efficient data structures
- SIMD support where available

## Cross-Platform Strategy

### Platform Targets
1. **Web**: WebGL 2.0, WebGPU
2. **Mobile**: iOS (Metal), Android (Vulkan/GLES)
3. **Desktop**: Windows (Vulkan/D3D), macOS (Metal), Linux (Vulkan)
4. **Mini-Games**: WeChat, Alipay, ByteDance platforms

### Platform Adaptation
- **PAL Layer**: Platform-specific implementations
- **Conditional Compilation**: Platform-specific code paths
- **Feature Detection**: Runtime capability detection
- **Graceful Degradation**: Fallbacks for missing features

## Extension Points

COCOS 4 provides multiple extension points:

1. **Custom Components**: User-defined components
2. **Custom Materials**: Shader-based materials
3. **Custom Render Pipeline**: Full pipeline customization
4. **Custom Post-Processing**: Post-process effects
5. **Custom Physics**: Physics engine integration
6. **Plugins**: Editor and runtime extensions

## Summary

COCOS 4 is a well-architected engine with:
- Clear separation of concerns
- Modular design
- Platform abstraction
- Performance-focused hybrid architecture
- Extensibility at multiple levels
- Modern graphics support

The hybrid C++/TypeScript approach provides both performance and developer productivity, while the GFX abstraction enables true cross-platform rendering.
