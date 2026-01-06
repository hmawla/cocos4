# COCOS 4 Core Components Guide

## Introduction

This document provides detailed information about the core components of the COCOS 4 engine, their responsibilities, and how they interact.

## Component Overview

```
┌─────────────────────────────────────────────────────────┐
│                      COCOS 4 Core                       │
├─────────────────────────────────────────────────────────┤
│ Root Manager  │  Controls entire engine lifecycle       │
│ GFX Layer     │  Graphics API abstraction               │
│ Render System │  Scene rendering and pipelines          │
│ Scene Graph   │  Hierarchical object organization       │
│ Asset System  │  Resource loading and management        │
│ Animation     │  Keyframe and procedural animation      │
│ Physics       │  Collision and rigid body simulation    │
│ Audio         │  Sound playback and effects             │
│ Input         │  User input handling                    │
│ UI System     │  User interface components              │
└─────────────────────────────────────────────────────────┘
```

---

## 1. Root Manager (`cocos/root.ts`)

### Purpose
The Root class is the central orchestrator of the engine, managing the entire lifecycle and coordinating all major systems.

### Key Responsibilities
- Initialize and manage the graphics device
- Create and manage render pipelines
- Manage render scenes and windows
- Coordinate frame updates
- Manage global resources and pools

### API Overview

```typescript
class Root {
    // Core Properties
    device: Device;                    // Graphics device
    mainWindow: RenderWindow;          // Primary render target
    pipeline: PipelineRuntime;         // Active render pipeline
    scenes: RenderScene[];             // Managed scenes
    
    // Initialization
    initialize(info: IRootInfo): boolean;
    
    // Frame Management
    frameMove(deltaTime: number): void;  // Update logic
    frameRender(): void;                  // Render frame
    
    // Scene Management
    createScene(info: ISceneInfo): RenderScene;
    destroyScene(scene: RenderScene): void;
    
    // Window Management
    createWindow(info: IRenderWindowInfo): RenderWindow | null;
    destroyWindow(window: RenderWindow): void;
    
    // Resource Management
    recyclePool<T>(Ctor: Constructor<T>): RecyclePool<T>;
    
    // Pipeline Management
    setRenderPipeline(pipeline: PipelineRuntime): boolean;
}
```

### Usage Example

```typescript
import { Root } from 'cc';

// Initialize the engine
const root = new Root(device);
const success = root.initialize({
    enableHDR: true
});

if (!success) {
    console.error('Failed to initialize Root');
}

// Create main window
const window = root.createWindow({
    title: 'My Game',
    width: 1920,
    height: 1080
});

// Game loop
function tick(deltaTime: number) {
    root.frameMove(deltaTime);
}
```

### Design Patterns
- **Singleton-like**: Usually one Root instance per application
- **Facade**: Simplifies access to complex subsystems
- **Mediator**: Coordinates between different engine systems

---

## 2. Graphics Abstraction (GFX) (`cocos/gfx`)

### Purpose
Provides a unified interface to multiple graphics APIs (Vulkan, Metal, WebGL, WebGPU).

### Architecture

```
┌──────────────────────────────────────┐
│      Application/Engine Code         │
├──────────────────────────────────────┤
│        GFX Abstract Interface        │
├──────────────────────────────────────┤
│  Vulkan │ Metal │ WebGL │ WebGPU    │
└──────────────────────────────────────┘
```

### Core Classes

#### Device
The main graphics device interface.

```typescript
class Device {
    // Device Information
    gfxAPI: API;                    // Current graphics API
    deviceName: string;             // Device name
    capabilities: DeviceCapabilities;
    
    // Resource Creation
    createBuffer(info: BufferInfo): Buffer;
    createTexture(info: TextureInfo): Texture;
    createShader(info: ShaderInfo): Shader;
    createSampler(info: SamplerInfo): Sampler;
    createRenderPass(info: RenderPassInfo): RenderPass;
    createFramebuffer(info: FramebufferInfo): Framebuffer;
    createPipelineState(info: PipelineStateInfo): PipelineState;
    
    // Command Submission
    createCommandBuffer(info: CommandBufferInfo): CommandBuffer;
    queue: Queue;
    
    // Frame Management
    acquire(swapchains: Swapchain[]): void;
    present(): void;
}
```

#### Buffer
GPU buffer for vertex data, indices, uniforms.

```typescript
class Buffer {
    usage: BufferUsage;      // VERTEX, INDEX, UNIFORM, etc.
    memoryFlags: MemoryFlags; // DEVICE, HOST, etc.
    size: number;
    stride: number;
    
    update(data: ArrayBuffer, size?: number): void;
    resize(size: number): void;
    destroy(): void;
}
```

#### Texture
GPU texture for images and render targets.

```typescript
class Texture {
    type: TextureType;       // 2D, CUBE, 3D, etc.
    usage: TextureUsage;     // SAMPLED, STORAGE, COLOR_ATTACHMENT
    format: Format;          // RGBA8, DEPTH24_STENCIL8, etc.
    width: number;
    height: number;
    depth: number;
    
    update(data: ArrayBuffer): void;
    resize(width: number, height: number): void;
    destroy(): void;
}
```

#### Shader
Compiled shader program.

```typescript
class Shader {
    name: string;
    stages: ShaderStage[];   // Vertex, Fragment, Compute
    attributes: Attribute[];  // Input attributes
    blocks: UniformBlock[];   // Uniform blocks
    samplers: UniformSampler[]; // Texture samplers
    
    destroy(): void;
}
```

#### CommandBuffer
Records GPU commands for execution.

```typescript
class CommandBuffer {
    type: CommandBufferType;  // PRIMARY, SECONDARY
    
    // State Management
    begin(): void;
    end(): void;
    beginRenderPass(renderPass: RenderPass, ...): void;
    endRenderPass(): void;
    
    // Pipeline Binding
    bindPipelineState(pipelineState: PipelineState): void;
    bindDescriptorSet(set: number, descriptorSet: DescriptorSet): void;
    
    // Resource Binding
    bindInputAssembler(inputAssembler: InputAssembler): void;
    
    // Drawing
    draw(inputAssembler: InputAssembler): void;
    drawInstanced(inputAssembler: InputAssembler, instanceCount: number): void;
    
    // Compute
    dispatch(workgroupX: number, workgroupY: number, workgroupZ: number): void;
    
    // Barriers
    pipelineBarrier(barriers: PipelineBarrier[]): void;
}
```

### Usage Example

```typescript
// Create a vertex buffer
const vertexBuffer = device.createBuffer({
    usage: BufferUsageBit.VERTEX,
    memUsage: MemoryUsageBit.DEVICE,
    size: vertices.byteLength,
    stride: Float32Array.BYTES_PER_ELEMENT * 3
});
vertexBuffer.update(vertices);

// Create a texture
const texture = device.createTexture({
    type: TextureType.TEX2D,
    usage: TextureUsageBit.SAMPLED,
    format: Format.RGBA8,
    width: 512,
    height: 512
});
texture.update(imageData);

// Record commands
const cmdBuff = device.createCommandBuffer({
    type: CommandBufferType.PRIMARY
});

cmdBuff.begin();
cmdBuff.beginRenderPass(renderPass, framebuffer, ...);
cmdBuff.bindPipelineState(pipelineState);
cmdBuff.bindDescriptorSet(0, descriptorSet);
cmdBuff.bindInputAssembler(inputAssembler);
cmdBuff.draw(inputAssembler);
cmdBuff.endRenderPass();
cmdBuff.end();

device.queue.submit([cmdBuff]);
```

---

## 3. Rendering System (`cocos/rendering`)

### Purpose
Manages the rendering pipeline, scene culling, and draw call batching.

### Key Components

#### Render Pipeline
Orchestrates the entire rendering process.

```typescript
abstract class RenderPipeline {
    flows: RenderFlow[];
    
    abstract initialize(info: IRenderPipelineInfo): boolean;
    abstract render(scenes: RenderScene[]): void;
    abstract destroy(): void;
}

// Built-in pipelines
class ForwardPipeline extends RenderPipeline { }
class DeferredPipeline extends RenderPipeline { }
```

#### Render Flow
Defines a sequence of render stages.

```typescript
class RenderFlow {
    name: string;
    priority: number;
    stages: RenderStage[];
    
    initialize(info: IRenderFlowInfo): boolean;
    render(view: RenderView): void;
    destroy(): void;
}
```

#### Render Stage
Individual rendering phase.

```typescript
class RenderStage {
    name: string;
    priority: number;
    
    initialize(info: IRenderStageInfo): boolean;
    render(view: RenderView): void;
    destroy(): void;
}
```

#### Render Queue
Batches and sorts render commands.

```typescript
class RenderQueue {
    queue: IRenderPass[];
    
    clear(): void;
    sort(): void;
    recordCommandBuffer(device: Device, renderPass: RenderPass, cmdBuff: CommandBuffer): void;
}
```

### Pipeline Flow

```
┌─────────────────────────────────────────┐
│         Pipeline::render()              │
├─────────────────────────────────────────┤
│  1. Scene Culling                       │
│     - Frustum culling                   │
│     - Occlusion culling                 │
│     - Layer filtering                   │
├─────────────────────────────────────────┤
│  2. Shadow Map Pass (if enabled)        │
│     - Render from light's perspective   │
├─────────────────────────────────────────┤
│  3. Reflection Pass (if enabled)        │
│     - Render reflection probes          │
├─────────────────────────────────────────┤
│  4. Main Render Pass                    │
│     ├─ Opaque Queue                     │
│     ├─ Transparent Queue                │
│     └─ UI Queue                         │
├─────────────────────────────────────────┤
│  5. Post-Processing                     │
│     - Bloom, tone mapping, etc.         │
└─────────────────────────────────────────┘
```

---

## 4. Scene Graph (`cocos/scene-graph`)

### Purpose
Hierarchical organization of game objects with spatial relationships.

### Core Classes

#### Node
Basic scene graph element.

```typescript
class Node {
    // Hierarchy
    parent: Node | null;
    children: Node[];
    
    // Transform
    position: Vec3;
    rotation: Quat;
    scale: Vec3;
    worldMatrix: Mat4;
    
    // Components
    components: Component[];
    
    // Visibility
    active: boolean;
    layer: Layers;
    
    // Methods
    addChild(child: Node): void;
    removeChild(child: Node): void;
    getComponent<T extends Component>(type: Constructor<T>): T | null;
    addComponent<T extends Component>(type: Constructor<T>): T;
    setPosition(x: number, y: number, z: number): void;
    setRotation(x: number, y: number, z: number, w: number): void;
    setScale(x: number, y: number, z: number): void;
    lookAt(target: Vec3, up?: Vec3): void;
    translate(translation: Vec3, ns?: NodeSpace): void;
    rotate(rotation: Quat, ns?: NodeSpace): void;
}
```

#### Component
Modular functionality attached to nodes.

```typescript
abstract class Component {
    node: Node;
    enabled: boolean;
    
    // Lifecycle
    onLoad?(): void;
    onEnable?(): void;
    start?(): void;
    update?(dt: number): void;
    lateUpdate?(dt: number): void;
    onDisable?(): void;
    onDestroy?(): void;
}
```

#### Scene
Root container for game objects.

```typescript
class Scene {
    name: string;
    autoReleaseAssets: boolean;
    
    // Lifecycle
    load(): Promise<void>;
    unload(): void;
}
```

### Usage Example

```typescript
import { Node, Scene, Vec3 } from 'cc';

// Create scene
const scene = new Scene('MainScene');

// Create node hierarchy
const rootNode = new Node('Root');
const childNode = new Node('Child');
rootNode.addChild(childNode);

// Set transforms
rootNode.setPosition(0, 0, 0);
childNode.setPosition(1, 2, 3);

// Add components
class PlayerController extends Component {
    update(dt: number) {
        // Move logic
        const pos = this.node.position;
        pos.x += dt * 5;
        this.node.setPosition(pos.x, pos.y, pos.z);
    }
}

childNode.addComponent(PlayerController);
```

---

## 5. Asset Management (`cocos/asset`)

### Purpose
Handles loading, caching, and lifecycle management of game resources.

### Core Classes

#### Asset
Base class for all asset types.

```typescript
abstract class Asset {
    uuid: string;
    name: string;
    
    // Lifecycle
    addRef(): Asset;
    decRef(): Asset;
    destroy(): boolean;
}
```

#### AssetManager
Central asset loading and management.

```typescript
class AssetManager {
    // Loading
    load<T extends Asset>(bundle: string, path: string, type: Constructor<T>, callback: (err: Error | null, asset: T) => void): void;
    loadDir<T extends Asset>(bundle: string, path: string, type: Constructor<T>, callback: (err: Error | null, assets: T[]) => void): void;
    loadScene(name: string, callback: (err: Error | null, scene: Scene) => void): void;
    
    // Bundles
    loadBundle(name: string, callback: (err: Error | null, bundle: Bundle) => void): void;
    removeBundle(name: string): void;
    getBundle(name: string): Bundle | null;
    
    // Release
    releaseAsset(asset: Asset): void;
    releaseUnusedAssets(): void;
}
```

### Asset Types

- **Texture**: Image files (PNG, JPG, etc.)
- **Material**: Shader + parameters
- **Mesh**: 3D geometry
- **Animation**: Animation clips
- **Audio**: Sound files
- **Prefab**: Reusable node templates
- **Scene**: Complete game scenes
- **Script**: TypeScript/JavaScript code

### Usage Example

```typescript
import { assetManager, Texture2D, Material } from 'cc';

// Load a single asset
assetManager.loadRemote<Texture2D>('http://example.com/texture.png', (err, texture) => {
    if (err) {
        console.error(err);
        return;
    }
    // Use texture
});

// Load from bundle
assetManager.loadBundle('resources', (err, bundle) => {
    if (err) {
        console.error(err);
        return;
    }
    
    bundle.load('materials/player', Material, (err, material) => {
        if (!err) {
            // Use material
        }
    });
});

// Release when done
assetManager.releaseAsset(texture);
```

---

## 6. Animation System (`cocos/animation`)

### Purpose
Provides keyframe and procedural animation capabilities.

### Core Classes

#### AnimationClip
Container for animation data.

```typescript
class AnimationClip extends Asset {
    duration: number;
    sample: number;
    speed: number;
    wrapMode: WrapMode;
    
    curves: AnimationCurve[];
    events: AnimationEvent[];
}
```

#### AnimationState
Runtime state of an animation.

```typescript
class AnimationState {
    clip: AnimationClip;
    name: string;
    speed: number;
    time: number;
    weight: number;
    
    play(): void;
    pause(): void;
    stop(): void;
    setTime(time: number): void;
}
```

#### Animation Component
Manages animation playback on a node.

```typescript
class Animation extends Component {
    clips: AnimationClip[];
    defaultClip: AnimationClip | null;
    playOnLoad: boolean;
    
    play(name?: string): AnimationState | null;
    pause(name?: string): void;
    stop(name?: string): void;
    getState(name: string): AnimationState | null;
}
```

---

## 7. Physics System

### 2D Physics (`cocos/physics-2d`)

#### RigidBody2D
2D physics body.

```typescript
class RigidBody2D extends Component {
    type: ERigidBody2DType;  // Static, Kinematic, Dynamic
    linearVelocity: Vec2;
    angularVelocity: number;
    gravityScale: number;
    
    applyForce(force: Vec2, point: Vec2): void;
    applyLinearImpulse(impulse: Vec2, point: Vec2): void;
    applyTorque(torque: number): void;
}
```

#### Collider2D
2D collision shape.

```typescript
abstract class Collider2D extends Component {
    sensor: boolean;
    density: number;
    friction: number;
    restitution: number;
}

class BoxCollider2D extends Collider2D { }
class CircleCollider2D extends Collider2D { }
class PolygonCollider2D extends Collider2D { }
```

### 3D Physics (`cocos/physics`)

#### RigidBody
3D physics body.

```typescript
class RigidBody extends Component {
    type: ERigidBodyType;
    mass: number;
    linearVelocity: Vec3;
    angularVelocity: Vec3;
    useGravity: boolean;
    
    applyForce(force: Vec3, relativePoint?: Vec3): void;
    applyImpulse(impulse: Vec3, relativePoint?: Vec3): void;
    applyTorque(torque: Vec3): void;
}
```

---

## 8. Input System (`cocos/input`)

### Purpose
Handles user input from keyboard, mouse, touch, and gamepad.

### Core Classes

```typescript
class Input {
    // Keyboard
    on(type: InputEventType.KEY_DOWN, callback: (event: EventKeyboard) => void): void;
    
    // Mouse
    on(type: InputEventType.MOUSE_MOVE, callback: (event: EventMouse) => void): void;
    
    // Touch
    on(type: InputEventType.TOUCH_START, callback: (event: EventTouch) => void): void;
    
    // Gamepad
    on(type: InputEventType.GAMEPAD_INPUT, callback: (event: EventGamepad) => void): void;
}
```

---

## Component Interaction

### Example: Complete Game Object

```typescript
// Create a player entity with multiple components
const player = new Node('Player');

// Add visual representation
const modelComponent = player.addComponent(ModelComponent);
modelComponent.mesh = playerMesh;
modelComponent.material = playerMaterial;

// Add physics
const rigidBody = player.addComponent(RigidBody);
rigidBody.mass = 1.0;

const collider = player.addComponent(BoxCollider);
collider.size = new Vec3(1, 2, 1);

// Add animation
const animation = player.addComponent(Animation);
animation.defaultClip = walkClip;

// Add custom logic
const controller = player.addComponent(PlayerController);
```

This demonstrates the modular, component-based architecture that makes COCOS 4 flexible and extensible.

## Summary

COCOS 4's core components work together to provide:
- **Efficient rendering** through the GFX abstraction
- **Flexible scene organization** with the scene graph
- **Easy asset management** with the asset system
- **Smooth animations** with the animation system
- **Realistic physics** with integrated physics engines
- **Responsive input** handling

Each component is designed to be modular, testable, and extensible, following solid software engineering principles.
