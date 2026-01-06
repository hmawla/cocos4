# COCOS 4 Engine Documentation Index

## Welcome to COCOS 4 Engine

COCOS 4 is a modern, high-performance, cross-platform game engine built on a hybrid C++/TypeScript architecture. This documentation will help you understand how the engine works, its core components, and best practices for development.

## Quick Links

### For Understanding the Engine
- **[Engine Architecture](./ENGINE_ARCHITECTURE.md)** - Comprehensive overview of the engine's architecture and design
- **[Core Components](./CORE_COMPONENTS.md)** - Detailed guide to the engine's core components
- **[Best Practices](./BEST_PRACTICES.md)** - Analysis of best practices used in the codebase

### For Contributors
- **[Contributing Guide](./CONTRIBUTING_GUIDE.md)** - Step-by-step guide for contributing to the engine
- **[TypeScript Coding Style](./TS_CODING_STYLE.md)** - TypeScript/JavaScript coding standards
- **[C++ Coding Style](./CPP_CODING_STYLE.md)** - C++ coding standards

### For Users
- **[README](../README.md)** - Project overview and quick start
- **[Main Website](https://www.cocos.com/)** - Official Cocos Creator website
- **[Documentation](https://docs.cocos.com/creator/manual/en/)** - User manual

## What is COCOS 4?

COCOS 4 is an evolution of the Cocos Creator engine, now fully open-source and AI-ready. Key features include:

### Modern Graphics
- **Multi-API Support**: Vulkan, Metal, WebGL 2.0, WebGPU
- **Customizable Pipelines**: Forward and deferred rendering
- **Advanced Features**: PBR, IBL, HDR, post-processing

### High Performance
- **Hybrid Architecture**: C++ for performance, TypeScript for productivity
- **Optimizations**: Batching, instancing, culling, LOD
- **Efficient**: Object pooling, memory management

### Cross-Platform
- **Web**: WebGL 2.0, WebGPU
- **Mobile**: iOS (Metal), Android (Vulkan)
- **Desktop**: Windows, macOS, Linux
- **Mini-Games**: WeChat, Alipay, ByteDance

### Developer-Friendly
- **TypeScript API**: Modern, type-safe scripting
- **VSCode Integration**: First-class editor support
- **Rich Ecosystem**: Extensive libraries and tools

## Engine Overview

### Architecture Layers

```
┌─────────────────────────────────────────┐
│     Game Application (TypeScript)       │
├─────────────────────────────────────────┤
│     Engine Core (TypeScript)            │
│  • Core Systems  • 2D/3D  • Animation   │
│  • Physics       • UI     • Audio       │
├─────────────────────────────────────────┤
│     JavaScript Bindings (JSB)           │
├─────────────────────────────────────────┤
│     Native Layer (C++)                  │
│  • GFX  • Renderer  • Scene Graph       │
├─────────────────────────────────────────┤
│     Graphics APIs                       │
│  Vulkan | Metal | WebGL | WebGPU        │
└─────────────────────────────────────────┘
```

### Core Modules

1. **Core** (`cocos/core`)
   - Math library (Vec3, Mat4, Quat)
   - Event system
   - Memory operations
   - Utilities

2. **GFX** (`cocos/gfx`)
   - Graphics API abstraction
   - Device management
   - Resource creation
   - Command buffers

3. **Rendering** (`cocos/rendering`)
   - Render pipelines
   - Culling and batching
   - Shadow mapping
   - Post-processing

4. **Scene Graph** (`cocos/scene-graph`)
   - Node hierarchy
   - Component system
   - Transform management

5. **Assets** (`cocos/asset`)
   - Resource loading
   - Asset management
   - Bundle system

6. **Animation** (`cocos/animation`)
   - Keyframe animation
   - Animation curves
   - Blending and transitions

7. **Physics** (`cocos/physics`, `cocos/physics-2d`)
   - 3D: Bullet, PhysX
   - 2D: Box2D
   - Collision detection
   - Rigid body dynamics

## Key Concepts

### Component-Based Architecture

COCOS 4 uses an Entity-Component System (ECS) variant:

```typescript
// Create a game object
const player = new Node('Player');

// Add visual component
const model = player.addComponent(ModelComponent);
model.mesh = playerMesh;
model.material = playerMaterial;

// Add physics
const rigidBody = player.addComponent(RigidBody);
rigidBody.mass = 1.0;

// Add custom logic
const controller = player.addComponent(PlayerController);
```

### Data-Driven Design

Assets and configurations are data-driven:

```typescript
// Load assets declaratively
assetManager.loadBundle('resources', (err, bundle) => {
    bundle.load('prefabs/enemy', Prefab, (err, prefab) => {
        const enemy = instantiate(prefab);
        scene.addChild(enemy);
    });
});
```

### Performance-First

The engine prioritizes performance:
- Object pooling for reduced allocations
- Automatic batching for draw call reduction
- Efficient data structures (TypedArrays)
- Optimized native code for hot paths

## Development Workflow

### 1. Setup Development Environment

```bash
# Clone repository
git clone https://github.com/hmawla/cocos4.git
cd cocos4

# Install dependencies
npm install

# Build engine
npm run build:dev
```

### 2. Make Changes

```bash
# Create feature branch
git checkout -b feature/my-feature

# Make changes to code
# Edit cocos/**/*.ts files

# Run linter
npx eslint cocos/**/*.ts --fix

# Run tests
npm test
```

### 3. Submit Changes

```bash
# Commit changes
git add .
git commit -m "feat: Add awesome feature"

# Push to GitHub
git push origin feature/my-feature

# Create Pull Request on GitHub
```

## Code Quality Standards

### TypeScript Standards

```typescript
// ✓ Good: Type-safe, well-documented
/**
 * @en Calculate damage with multiplier
 * @zh 计算伤害值（带倍率）
 */
function calculateDamage(base: number, multiplier: number): number {
    return base * multiplier;
}

// ✗ Bad: No types, no documentation
function calc(a, b) {
    return a * b;
}
```

### C++ Standards

```cpp
// ✓ Good: Clear naming, RAII
class Renderer {
public:
    Renderer() = default;
    ~Renderer() = default;
    
    void render(const Scene& scene);
    
private:
    Device* _device{nullptr};
    int _frameCount{0};
};

// ✗ Bad: Manual memory management, unclear names
class R {
    void* d;
    int fc;
};
```

## Testing Philosophy

### Test-Driven Development

```typescript
// Write tests first
describe('MathUtils', () => {
    test('lerp should interpolate correctly', () => {
        expect(lerp(0, 10, 0.5)).toBe(5);
        expect(lerp(0, 10, 0)).toBe(0);
        expect(lerp(0, 10, 1)).toBe(10);
    });
});

// Then implement
function lerp(a: number, b: number, t: number): number {
    return a + (b - a) * t;
}
```

### Test Coverage

- **Unit Tests**: Test individual functions and classes
- **Integration Tests**: Test component interactions
- **Performance Tests**: Benchmark critical paths
- **Platform Tests**: Verify cross-platform compatibility

## Performance Guidelines

### Memory Management

```typescript
// ✓ Good: Object pooling
class ParticleSystem {
    private _particlePool = new RecyclePool(() => new Particle());
    
    emit() {
        const particle = this._particlePool.alloc();
        // Use particle
    }
    
    recycle(particle: Particle) {
        this._particlePool.free(particle);
    }
}

// ✗ Bad: Allocating every frame
update() {
    const particle = new Particle();  // Don't do this in update()
}
```

### Rendering Optimization

```typescript
// ✓ Good: Batch rendering
class SpriteRenderer {
    batchSprites(sprites: Sprite[]) {
        // Group by material
        const batches = groupByMaterial(sprites);
        for (const batch of batches) {
            this.drawBatch(batch);  // One draw call per material
        }
    }
}

// ✗ Bad: Individual draw calls
for (const sprite of sprites) {
    this.drawSprite(sprite);  // Many draw calls
}
```

## Security Considerations

### Input Validation

```typescript
// ✓ Good: Validate all inputs
function loadAsset(url: string): void {
    if (!isValidURL(url)) {
        errorID(1001, 'Invalid URL');
        return;
    }
    // Proceed with loading
}

// ✗ Bad: No validation
function loadAsset(url: string): void {
    fetch(url);  // Potential security risk
}
```

### Resource Limits

```typescript
// ✓ Good: Enforce limits
const MAX_TEXTURES = 256;

function addTexture(texture: Texture): boolean {
    if (this._textures.length >= MAX_TEXTURES) {
        warnID(2001, 'Texture limit reached');
        return false;
    }
    this._textures.push(texture);
    return true;
}
```

## Community and Support

### Getting Help

- **GitHub Issues**: Bug reports and feature requests
- **Forum**: [Cocos Forum](https://discuss.cocos2d-x.org/c/creator)
- **Discord**: Join the Cocos community
- **Documentation**: [Official Docs](https://docs.cocos.com/creator/manual/en/)

### Contributing

We welcome contributions! Please see:
- [Contributing Guide](./CONTRIBUTING_GUIDE.md)
- [Code of Conduct](../CODE_OF_CONDUCT.md) (if available)
- [Pull Request Template](../.github/PULL_REQUEST_TEMPLATE.md)

### Example Projects

Learn from examples:
- [Mind Your Step 3D](https://github.com/cocos/cocos-tutorial-mind-your-step)
- [Test Cases](https://github.com/cocos/cocos-test-projects)
- [Example Cases](https://github.com/cocos/cocos-example-projects)
- [Awesome Cocos](https://github.com/cocos/awesome-cocos)

## Roadmap

COCOS 4 is actively developed with focus on:
- Enhanced AI integration
- WebGPU support
- Performance improvements
- Better tooling
- Extended platform support
- Community-driven features

Check the [GitHub Projects](https://github.com/orgs/cocos/projects) for the latest roadmap.

## License

COCOS 4 is licensed under the [MIT License](../LICENSE), which means:
- ✓ Commercial use
- ✓ Modification
- ✓ Distribution
- ✓ Private use

## Acknowledgments

COCOS 4 builds on the foundation of:
- Cocos Creator 1.x, 2.x, 3.x
- Cocos2d-x legacy
- Open-source community contributions
- Industry-standard libraries (Bullet, Box2D, etc.)

## Next Steps

1. **New to COCOS 4?**
   - Start with [README](../README.md)
   - Read [Engine Architecture](./ENGINE_ARCHITECTURE.md)
   - Explore example projects

2. **Want to Contribute?**
   - Read [Contributing Guide](./CONTRIBUTING_GUIDE.md)
   - Check [Good First Issues](https://github.com/hmawla/cocos4/labels/good%20first%20issue)
   - Join the community discussions

3. **Building a Game?**
   - Check [Official Documentation](https://docs.cocos.com/creator/manual/en/)
   - Explore [Core Components](./CORE_COMPONENTS.md)
   - Learn [Best Practices](./BEST_PRACTICES.md)

4. **Need Help?**
   - Search existing issues
   - Ask on the forum
   - Join Discord community

---

**Happy Coding!** 🎮

*This documentation was created to help understand the COCOS 4 engine architecture, core components, and development best practices.*
