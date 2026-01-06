# Contributing to COCOS 4 Engine

Welcome to the COCOS 4 engine project! This guide will help you get started with contributing to the engine.

## Table of Contents

1. [Getting Started](#getting-started)
2. [Development Setup](#development-setup)
3. [Understanding the Codebase](#understanding-the-codebase)
4. [Making Changes](#making-changes)
5. [Testing](#testing)
6. [Submitting Changes](#submitting-changes)
7. [Code Review Process](#code-review-process)
8. [Resources](#resources)

## Getting Started

### Prerequisites

Before you begin, ensure you have:

- **Node.js** v18.0.0 or higher
- **npm** (comes with Node.js)
- **Git** for version control
- **Code Editor** (VSCode recommended)
- **C++ Compiler** (if working on native code)
  - Windows: Visual Studio 2019+
  - macOS: Xcode Command Line Tools
  - Linux: GCC or Clang

### First Steps

1. **Read the Documentation**
   - [Engine Architecture](./ENGINE_ARCHITECTURE.md)
   - [Core Components](./CORE_COMPONENTS.md)
   - [Best Practices](./BEST_PRACTICES.md)
   - [TypeScript Coding Style](./TS_CODING_STYLE.md)
   - [C++ Coding Style](./CPP_CODING_STYLE.md)

2. **Explore the Codebase**
   - Browse through the `cocos/` directory
   - Look at existing issues on GitHub
   - Check the project board for upcoming features

3. **Join the Community**
   - Discussion forum: [Cocos Forum](https://discuss.cocos2d-x.org/c/creator)
   - Discord: Search for Cocos in Discord Discover

## Development Setup

### Clone the Repository

```bash
# Fork the repository on GitHub first, then clone your fork
git clone https://github.com/YOUR_USERNAME/cocos4.git
cd cocos4

# Add upstream remote
git remote add upstream https://github.com/hmawla/cocos4.git
```

### Install Dependencies

```bash
# Install all dependencies
npm install

# This will:
# - Install Node.js packages
# - Build debug information
# - Generate type declarations
# - Build platform adapters
# - Build native pack tools
```

### Build the Engine

```bash
# Development build (faster, includes source maps)
npm run build:dev

# Production build (optimized)
npm run build

# Watch mode for development
npm run build:dev && npm run watch  # If available
```

### Set Up Your Editor

#### VSCode (Recommended)

1. Install recommended extensions:
   - ESLint
   - TypeScript and JavaScript Language Features
   - C/C++ (if working on native code)
   - Prettier (optional)

2. Configure workspace settings (`.vscode/settings.json`):
```json
{
  "editor.formatOnSave": false,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "typescript.tsdk": "node_modules/typescript/lib",
  "eslint.validate": ["typescript", "javascript"]
}
```

### Verify Your Setup

```bash
# Run type checking
npx tsc --noEmit

# Run linter
npx eslint cocos/**/*.ts

# Run tests (note: some may fail if dependencies aren't built)
npm test
```

## Understanding the Codebase

### Directory Structure

```
cocos4/
├── cocos/              # Main engine source code (TypeScript)
│   ├── core/          # Core utilities (math, events, etc.)
│   ├── gfx/           # Graphics abstraction layer
│   ├── rendering/     # Render pipeline
│   ├── scene-graph/   # Node hierarchy
│   ├── 2d/            # 2D rendering
│   ├── 3d/            # 3D rendering
│   ├── animation/     # Animation system
│   ├── physics/       # 3D physics
│   ├── physics-2d/    # 2D physics
│   └── ...
├── native/            # Native C++ code
│   ├── cocos/         # Core C++ implementation
│   └── tools/         # Native build tools
├── pal/               # Platform abstraction layer
├── tests/             # Test files
├── docs/              # Documentation
├── scripts/           # Build scripts
└── package.json       # Node.js dependencies and scripts
```

### Module Organization

Each module in `cocos/` typically has:
- `index.ts` - Public API exports
- `category.json` - Module metadata
- Implementation files (`.ts`)
- Types and interfaces
- Internal utilities

### Important Files

- `cocos/root.ts` - Engine root manager
- `cocos/core/index.ts` - Core module entry
- `cocos/gfx/base/device.ts` - Graphics device interface
- `tsconfig.json` - TypeScript configuration
- `.eslintrc.yaml` - ESLint rules
- `jest.config.js` - Test configuration

## Making Changes

### Finding Issues to Work On

1. **Good First Issues**: Look for issues labeled `good first issue`
2. **Help Wanted**: Issues labeled `help wanted` need contributors
3. **Feature Requests**: Check for approved feature requests
4. **Bug Reports**: Look for confirmed bugs

### Creating a Branch

```bash
# Update your fork
git checkout main
git pull upstream main

# Create a feature branch
git checkout -b feature/my-awesome-feature

# Or for bug fixes
git checkout -b fix/issue-123
```

### Coding Guidelines

#### TypeScript Code

```typescript
// ✓ Good: Follow naming conventions
class MyComponent extends Component {
    private _myPrivateVar: number = 0;
    public myPublicVar: string = '';
    
    public myMethod(): void {
        // Implementation
    }
}

// ✓ Good: Add JSDoc comments for public APIs
/**
 * @en Calculate the distance between two points
 * @zh 计算两点之间的距离
 * @param a First point
 * @param b Second point
 * @returns Distance between points
 */
function distance(a: Vec3, b: Vec3): number {
    return Vec3.distance(a, b);
}

// ✓ Good: Use type annotations
function processData(input: string, count: number): Result {
    // ...
}

// ✗ Avoid: Using any
function badFunction(data: any): any {
    // ...
}

// ✓ Good: Use specific types
function goodFunction(data: DataType): ResultType {
    // ...
}
```

#### C++ Code

```cpp
// ✓ Good: Follow naming conventions
class MyRenderer {
public:
    MyRenderer() = default;
    ~MyRenderer() = default;
    
    void render();
    
private:
    int _frameCount{0};
    Device* _device{nullptr};
};

// ✓ Good: Use pragma once
#pragma once

// ✓ Good: Member initializers
MyRenderer::MyRenderer()
: _frameCount(0),
  _device(nullptr) {
}

// ✓ Good: Const correctness
void processData(const Data& data) const;
```

### Making Incremental Changes

1. **Start Small**: Make focused, incremental changes
2. **One Issue Per PR**: Don't mix unrelated changes
3. **Commit Often**: Make small, logical commits
4. **Write Clear Commits**: Use descriptive commit messages

```bash
# Good commit message format
git commit -m "feat: Add new particle system feature

- Implement GPU-based particle simulation
- Add custom shader support
- Update documentation

Fixes #123"
```

### Commit Message Guidelines

Follow the Conventional Commits format:

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting, etc.)
- `refactor`: Code refactoring
- `perf`: Performance improvements
- `test`: Adding or updating tests
- `chore`: Build process or auxiliary tool changes

**Examples:**
```
feat(rendering): Add deferred shading pipeline

fix(physics): Correct collision detection for rotated boxes

docs: Update installation instructions

refactor(core): Simplify event system implementation

perf(2d): Optimize sprite batching algorithm
```

## Testing

### Running Tests

```bash
# Run all tests
npm test

# Run specific test file
npx jest tests/core/math.test.ts

# Run tests in watch mode
npx jest --watch

# Run with coverage
npx jest --coverage
```

### Writing Tests

```typescript
// tests/my-feature/my-component.test.ts
import { MyComponent } from '../../cocos/my-feature/my-component';

describe('MyComponent', () => {
    let component: MyComponent;
    
    beforeEach(() => {
        component = new MyComponent();
    });
    
    afterEach(() => {
        component.destroy();
    });
    
    test('should initialize correctly', () => {
        expect(component.isInitialized).toBe(true);
    });
    
    test('should handle invalid input', () => {
        expect(() => component.process(null)).toThrow();
    });
    
    test('should produce correct output', () => {
        const result = component.calculate(5);
        expect(result).toBeCloseTo(25, 2);
    });
});
```

### Testing Best Practices

1. **Test Public APIs**: Focus on testing public interfaces
2. **Test Edge Cases**: Include boundary conditions
3. **Test Error Handling**: Verify error cases work correctly
4. **Mock External Dependencies**: Use mocks for external systems
5. **Keep Tests Fast**: Tests should run quickly
6. **Make Tests Deterministic**: Tests should always produce the same result

### Manual Testing

For features that require visual verification:

1. Build the engine: `npm run build:dev`
2. Create a test project or use existing test cases
3. Test across multiple platforms if possible
4. Document your testing steps in the PR

## Submitting Changes

### Before Submitting

1. **Ensure Code Quality**
   ```bash
   # Run linter
   npx eslint cocos/**/*.ts --fix
   
   # Run type checker
   npx tsc --noEmit
   
   # Run tests
   npm test
   ```

2. **Update Documentation**
   - Update relevant `.md` files
   - Add JSDoc comments for new APIs
   - Update CHANGELOG if applicable

3. **Test Thoroughly**
   - Run manual tests
   - Ensure no regressions
   - Test on different platforms if possible

### Creating a Pull Request

1. **Push Your Branch**
   ```bash
   git push origin feature/my-awesome-feature
   ```

2. **Open a PR on GitHub**
   - Go to your fork on GitHub
   - Click "New Pull Request"
   - Select your branch
   - Fill out the PR template

3. **PR Description Should Include:**
   - **Summary**: What does this PR do?
   - **Motivation**: Why is this change needed?
   - **Implementation**: How is it implemented?
   - **Testing**: How was it tested?
   - **Screenshots**: For visual changes
   - **Related Issues**: Link to issues (e.g., "Fixes #123")
   - **Breaking Changes**: List any breaking changes

### PR Template Example

```markdown
## Description
Adds support for custom render passes in the forward pipeline.

## Motivation
Users need to inject custom rendering logic between standard passes.

## Changes
- Added `CustomRenderPass` class
- Modified `ForwardPipeline` to support custom passes
- Updated documentation

## Testing
- Added unit tests for `CustomRenderPass`
- Tested with sample project on Web and Native platforms
- Performance impact: <1% overhead

## Screenshots
[If applicable]

## Checklist
- [x] Tests pass
- [x] Documentation updated
- [x] No breaking changes
- [x] Follows coding style

Fixes #456
```

## Code Review Process

### What to Expect

1. **Initial Review**: A maintainer will review your PR within a few days
2. **Feedback**: You may receive comments and change requests
3. **Discussion**: Be open to discussion about implementation
4. **Iteration**: Make requested changes and push updates
5. **Approval**: Once approved, a maintainer will merge your PR

### Responding to Feedback

- **Be Respectful**: Maintainers are volunteers
- **Ask Questions**: If feedback is unclear, ask for clarification
- **Provide Context**: Explain your reasoning if needed
- **Be Open**: Be willing to change your approach
- **Update Quickly**: Address feedback promptly

### Common Review Comments

- **Code Style**: "Please follow the style guide"
- **Performance**: "This could be optimized"
- **Tests**: "Please add tests for this case"
- **Documentation**: "Please document this API"
- **Breaking Changes**: "This is a breaking change, can we avoid it?"

## Best Practices Summary

### Do's ✓

- Follow the coding style guides
- Write clear, descriptive commit messages
- Add tests for new features
- Document public APIs
- Keep changes focused and small
- Ask questions when unsure
- Be patient and respectful

### Don'ts ✗

- Don't make breaking changes without discussion
- Don't mix unrelated changes in one PR
- Don't ignore CI failures
- Don't take feedback personally
- Don't commit generated files (unless necessary)
- Don't use console.log (use the logging system)

## Resources

### Documentation

- [Engine Architecture](./ENGINE_ARCHITECTURE.md)
- [Core Components](./CORE_COMPONENTS.md)
- [Best Practices](./BEST_PRACTICES.md)
- [TypeScript Style Guide](./TS_CODING_STYLE.md)
- [C++ Style Guide](./CPP_CODING_STYLE.md)

### External Resources

- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [Jest Documentation](https://jestjs.io/docs/getting-started)
- [ESLint Rules](https://eslint.org/docs/rules/)
- [Conventional Commits](https://www.conventionalcommits.org/)

### Getting Help

- **GitHub Issues**: For bug reports and feature requests
- **GitHub Discussions**: For questions and discussions
- **Forum**: [Cocos Forum](https://discuss.cocos2d-x.org/c/creator)
- **Discord**: Join the Cocos Discord community

## License

By contributing to COCOS 4, you agree that your contributions will be licensed under the MIT License.

## Thank You!

Thank you for contributing to COCOS 4! Your contributions help make the engine better for everyone.

---

**Questions?** Feel free to open a discussion or reach out to the maintainers.
