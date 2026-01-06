# COCOS 4 Quick Reference Guide

A quick reference for common tasks, patterns, and APIs in COCOS 4.

## Table of Contents

- [Node & Scene Graph](#node--scene-graph)
- [Components](#components)
- [Transform Operations](#transform-operations)
- [Asset Loading](#asset-loading)
- [Rendering](#rendering)
- [Animation](#animation)
- [Physics](#physics)
- [Input Handling](#input-handling)
- [Common Patterns](#common-patterns)
- [Performance Tips](#performance-tips)

---

## Node & Scene Graph

### Creating Nodes

```typescript
import { Node, Scene } from 'cc';

// Create a node
const node = new Node('MyNode');

// Create with hierarchy
const parent = new Node('Parent');
const child = new Node('Child');
parent.addChild(child);

// Remove from parent
parent.removeChild(child);

// Destroy node
node.destroy();
```

### Finding Nodes

```typescript
// Get child by name
const child = parent.getChildByName('ChildName');

// Get child by path
const nested = parent.getChildByPath('Child/GrandChild');

// Find in hierarchy
const found = parent.findChild((node) => node.name === 'Target');
```

### Node Properties

```typescript
// Active state
node.active = true;
node.activeInHierarchy;  // Read-only: considers parent

// Layer
node.layer = Layers.Enum.UI_2D;

// Transform (local space)
node.position;
node.rotation;
node.scale;

// Transform (world space)
node.worldPosition;
node.worldRotation;
node.worldScale;
```

---

## Components

### Adding Components

```typescript
import { Component } from 'cc';

// Define custom component
class MyComponent extends Component {
    onLoad() {
        // Called when component is first loaded
    }
    
    start() {
        // Called before first frame
    }
    
    update(deltaTime: number) {
        // Called every frame
    }
    
    onDestroy() {
        // Called when component is destroyed
    }
}

// Add to node
const comp = node.addComponent(MyComponent);
```

### Getting Components

```typescript
// Get component on same node
const comp = node.getComponent(MyComponent);

// Get component in children
const comp = node.getComponentInChildren(MyComponent);

// Get all components of type
const comps = node.getComponents(MyComponent);
```

### Component Lifecycle

```typescript
class LifecycleExample extends Component {
    onLoad() {
        // Initialize component
        // Called once when loaded
    }
    
    onEnable() {
        // Called when component is enabled
        // Can be called multiple times
    }
    
    start() {
        // Called before first update
        // All components are loaded
    }
    
    update(dt: number) {
        // Called every frame
    }
    
    lateUpdate(dt: number) {
        // Called after all updates
    }
    
    onDisable() {
        // Called when component is disabled
    }
    
    onDestroy() {
        // Called when component is destroyed
        // Clean up resources here
    }
}
```

---

## Transform Operations

### Position

```typescript
import { Vec3 } from 'cc';

// Set position
node.setPosition(10, 20, 30);
node.position = new Vec3(10, 20, 30);

// Get position
const pos = node.position;
const worldPos = node.worldPosition;

// Translate
node.translate(new Vec3(1, 0, 0));
```

### Rotation

```typescript
import { Quat, Vec3 } from 'cc';

// Set rotation (quaternion)
node.setRotation(0, 0, 0, 1);
node.rotation = Quat.IDENTITY;

// Rotate by euler angles
node.setRotationFromEuler(0, 90, 0);

// Look at target
node.lookAt(target.worldPosition);

// Rotate around axis
node.rotate(Quat.fromAxisAngle(new Quat(), Vec3.UP, Math.PI / 4));
```

### Scale

```typescript
import { Vec3 } from 'cc';

// Set scale
node.setScale(2, 2, 2);
node.scale = new Vec3(2, 2, 2);

// Uniform scale
node.setScale(2, 2, 2);
```

---

## Asset Loading

### Loading Single Asset

```typescript
import { assetManager, Texture2D, Material, Prefab } from 'cc';

// Load from resources
assetManager.resources.load('textures/player', Texture2D, (err, texture) => {
    if (err) {
        console.error(err);
        return;
    }
    // Use texture
});

// Load with type inference
assetManager.resources.load<Material>('materials/pbr', Material, (err, material) => {
    // Use material
});
```

### Loading Multiple Assets

```typescript
// Load directory
assetManager.resources.loadDir('prefabs', Prefab, (err, assets) => {
    if (err) {
        console.error(err);
        return;
    }
    // assets is an array of Prefab
});

// Load specific assets
const paths = ['path1', 'path2', 'path3'];
assetManager.resources.load(paths, Texture2D, (err, textures) => {
    // textures is an array
});
```

### Asset Bundles

```typescript
// Load bundle
assetManager.loadBundle('myBundle', (err, bundle) => {
    if (err) {
        console.error(err);
        return;
    }
    
    // Load from bundle
    bundle.load('assets/texture', Texture2D, (err, texture) => {
        // Use texture
    });
});

// Get loaded bundle
const bundle = assetManager.getBundle('myBundle');
```

### Remote Assets

```typescript
// Load remote asset
assetManager.loadRemote<Texture2D>('http://example.com/image.png', (err, texture) => {
    if (err) {
        console.error(err);
        return;
    }
    // Use texture
});
```

### Releasing Assets

```typescript
// Release single asset
assetManager.releaseAsset(texture);

// Release unused assets
assetManager.releaseUnusedAssets();
```

---

## Rendering

### Sprites (2D)

```typescript
import { Sprite, SpriteFrame } from 'cc';

// Add sprite component
const sprite = node.addComponent(Sprite);

// Set sprite frame
sprite.spriteFrame = spriteFrame;

// Set color
sprite.color = new Color(255, 0, 0, 255);

// Set size mode
sprite.sizeMode = Sprite.SizeMode.CUSTOM;
sprite.customSize = new Size(100, 100);
```

### Models (3D)

```typescript
import { ModelComponent, Material, Mesh } from 'cc';

// Add model component
const model = node.addComponent(ModelComponent);

// Set mesh
model.mesh = mesh;

// Set material
model.material = material;

// Set materials (multiple)
model.setMaterial(material, 0);  // Index 0
```

### Cameras

```typescript
import { Camera } from 'cc';

// Add camera component
const camera = node.addComponent(Camera);

// Set projection
camera.projection = Camera.ProjectionType.PERSPECTIVE;
camera.fov = 60;
camera.near = 0.1;
camera.far = 1000;

// Orthographic
camera.projection = Camera.ProjectionType.ORTHO;
camera.orthoHeight = 10;

// Clear flags
camera.clearFlags = CameraClearFlags.COLOR | CameraClearFlags.DEPTH;
camera.clearColor = new Color(0, 0, 0, 255);

// Visibility layers
camera.visibility = Layers.Enum.DEFAULT;
```

### Lights

```typescript
import { DirectionalLight, PointLight, SpotLight } from 'cc';

// Directional light
const dirLight = node.addComponent(DirectionalLight);
dirLight.color = new Color(255, 255, 255, 255);
dirLight.illuminance = 65000;

// Point light
const pointLight = node.addComponent(PointLight);
pointLight.range = 10;
pointLight.luminousFlux = 1000;

// Spot light
const spotLight = node.addComponent(SpotLight);
spotLight.range = 10;
spotLight.spotAngle = 45;
```

---

## Animation

### Animation Component

```typescript
import { Animation, AnimationClip } from 'cc';

// Add animation component
const anim = node.addComponent(Animation);

// Add clip
anim.clips = [walkClip, runClip];
anim.defaultClip = walkClip;

// Play animation
anim.play('walk');

// Control playback
anim.pause('walk');
anim.resume('walk');
anim.stop('walk');

// Animation events
anim.on(Animation.EventType.PLAY, () => {
    console.log('Animation started');
});

anim.on(Animation.EventType.FINISHED, () => {
    console.log('Animation finished');
});
```

### Tweening

```typescript
import { tween, Vec3 } from 'cc';

// Tween position
tween(node)
    .to(1, { position: new Vec3(10, 0, 0) })
    .start();

// Chain tweens
tween(node)
    .to(1, { position: new Vec3(10, 0, 0) })
    .to(1, { position: new Vec3(0, 10, 0) })
    .start();

// Repeat
tween(node)
    .to(1, { scale: new Vec3(2, 2, 2) })
    .to(1, { scale: new Vec3(1, 1, 1) })
    .repeatForever()
    .start();

// Callbacks
tween(node)
    .to(1, { position: new Vec3(10, 0, 0) })
    .call(() => {
        console.log('Tween finished');
    })
    .start();

// Easing
import { EasingType } from 'cc';

tween(node)
    .to(1, { position: new Vec3(10, 0, 0) }, { easing: 'sineInOut' })
    .start();
```

---

## Physics

### 3D Physics

```typescript
import { RigidBody, BoxCollider } from 'cc';

// Add rigid body
const rb = node.addComponent(RigidBody);
rb.mass = 1;
rb.linearDamping = 0.1;
rb.angularDamping = 0.1;
rb.useGravity = true;

// Add collider
const collider = node.addComponent(BoxCollider);
collider.size = new Vec3(1, 1, 1);
collider.center = new Vec3(0, 0, 0);

// Apply forces
rb.applyForce(new Vec3(0, 100, 0));
rb.applyImpulse(new Vec3(0, 10, 0));
rb.applyTorque(new Vec3(0, 1, 0));

// Collision callbacks
import { ICollisionEvent } from 'cc';

const collider = node.getComponent(Collider);
collider.on('onCollisionEnter', (event: ICollisionEvent) => {
    console.log('Collision with', event.otherCollider.node.name);
});
```

### 2D Physics

```typescript
import { RigidBody2D, BoxCollider2D } from 'cc';

// Add rigid body
const rb = node.addComponent(RigidBody2D);
rb.type = ERigidBody2DType.Dynamic;
rb.linearVelocity = new Vec2(0, 0);
rb.gravityScale = 1;

// Add collider
const collider = node.addComponent(BoxCollider2D);
collider.size = new Size(100, 100);
collider.offset = new Vec2(0, 0);

// Apply forces
rb.applyForceToCenter(new Vec2(100, 0));
rb.applyLinearImpulse(new Vec2(10, 0), new Vec2(0, 0));
```

---

## Input Handling

### Keyboard

```typescript
import { Input, input, KeyCode } from 'cc';

// Listen for key down
input.on(Input.EventType.KEY_DOWN, (event) => {
    if (event.keyCode === KeyCode.SPACE) {
        console.log('Space pressed');
    }
});

// Listen for key up
input.on(Input.EventType.KEY_UP, (event) => {
    console.log('Key released:', event.keyCode);
});
```

### Mouse

```typescript
import { Input, input, EventMouse } from 'cc';

// Mouse move
input.on(Input.EventType.MOUSE_MOVE, (event: EventMouse) => {
    const delta = event.getDelta();
    console.log('Mouse moved:', delta);
});

// Mouse button
input.on(Input.EventType.MOUSE_DOWN, (event: EventMouse) => {
    const button = event.getButton();
    console.log('Mouse button pressed:', button);
});

// Mouse wheel
input.on(Input.EventType.MOUSE_WHEEL, (event: EventMouse) => {
    const scrollY = event.getScrollY();
    console.log('Mouse wheel:', scrollY);
});
```

### Touch

```typescript
import { Input, input, EventTouch } from 'cc';

// Touch start
input.on(Input.EventType.TOUCH_START, (event: EventTouch) => {
    const touch = event.touch;
    const location = touch.getLocation();
    console.log('Touch at:', location);
});

// Touch move
input.on(Input.EventType.TOUCH_MOVE, (event: EventTouch) => {
    const touch = event.touch;
    const delta = touch.getDelta();
});

// Touch end
input.on(Input.EventType.TOUCH_END, (event: EventTouch) => {
    console.log('Touch ended');
});
```

---

## Common Patterns

### Singleton Component

```typescript
class GameManager extends Component {
    private static _instance: GameManager | null = null;
    
    static get instance(): GameManager {
        return this._instance!;
    }
    
    onLoad() {
        if (GameManager._instance) {
            this.destroy();
            return;
        }
        GameManager._instance = this;
    }
    
    onDestroy() {
        if (GameManager._instance === this) {
            GameManager._instance = null;
        }
    }
}
```

### Object Pool

```typescript
import { NodePool, Prefab, instantiate } from 'cc';

class BulletPool {
    private _pool: NodePool;
    
    constructor(prefab: Prefab, initialSize: number) {
        this._pool = new NodePool();
        for (let i = 0; i < initialSize; i++) {
            const node = instantiate(prefab);
            this._pool.put(node);
        }
    }
    
    get(): Node {
        if (this._pool.size() > 0) {
            return this._pool.get();
        }
        return instantiate(this._prefab);
    }
    
    put(node: Node): void {
        this._pool.put(node);
    }
}
```

### Event System

```typescript
import { EventTarget } from 'cc';

class GameEvents {
    static readonly SCORE_CHANGED = 'score-changed';
    static readonly GAME_OVER = 'game-over';
    
    private static _eventTarget = new EventTarget();
    
    static on(type: string, callback: Function, target?: any): void {
        this._eventTarget.on(type, callback, target);
    }
    
    static emit(type: string, ...args: any[]): void {
        this._eventTarget.emit(type, ...args);
    }
    
    static off(type: string, callback?: Function, target?: any): void {
        this._eventTarget.off(type, callback, target);
    }
}

// Usage
GameEvents.on(GameEvents.SCORE_CHANGED, (score: number) => {
    console.log('Score:', score);
});

GameEvents.emit(GameEvents.SCORE_CHANGED, 100);
```

---

## Performance Tips

### Avoid Allocations in Update

```typescript
// ✗ Bad: Creates new object every frame
update() {
    const pos = new Vec3(0, 0, 0);
    this.node.getPosition(pos);
}

// ✓ Good: Reuse object
private _tempPos = new Vec3();

update() {
    this.node.getPosition(this._tempPos);
}
```

### Cache Component References

```typescript
// ✗ Bad: Looks up component every frame
update() {
    const sprite = this.node.getComponent(Sprite);
    sprite.color = Color.RED;
}

// ✓ Good: Cache reference
private _sprite: Sprite = null!;

onLoad() {
    this._sprite = this.node.getComponent(Sprite)!;
}

update() {
    this._sprite.color = Color.RED;
}
```

### Use Object Pools

```typescript
// ✓ Good: Pool frequently created objects
class BulletManager {
    private _pool = new NodePool();
    
    spawnBullet(): Node {
        let bullet = this._pool.get();
        if (!bullet) {
            bullet = instantiate(this.bulletPrefab);
        }
        return bullet;
    }
    
    recycleBullet(bullet: Node): void {
        this._pool.put(bullet);
    }
}
```

### Batch Rendering

```typescript
// ✓ Good: Sprites with same texture are batched
// Make sure sprites use the same material and texture
sprite1.spriteFrame = frame;
sprite2.spriteFrame = frame;
sprite3.spriteFrame = frame;
```

---

This quick reference covers the most common operations in COCOS 4. For more detailed information, see the [Core Components Guide](./CORE_COMPONENTS.md) and [Official Documentation](https://docs.cocos.com/creator/manual/en/).
