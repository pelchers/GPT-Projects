# Vitest Integration in BetterGPT

## Overview

Vitest serves as our primary testing framework, chosen for its Vite integration, excellent TypeScript support, and Jest-compatible API. It provides fast, parallel test execution and built-in coverage reporting.

## Core Features

| Feature | Description | Priority |
|---------|-------------|----------|
| Unit Testing | Component and utility testing | P0 |
| Integration Testing | API and service testing | P0 |
| Coverage Reports | Code coverage analysis | P1 |
| Snapshot Testing | UI component snapshots | P1 |
| Performance Testing | Timing and optimization | P2 |

## Setup and Configuration

### 1. Base Configuration

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';
import tsconfigPaths from 'vite-tsconfig-paths';

export default defineConfig({
  plugins: [react(), tsconfigPaths()],
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: ['./src/test/setup.ts'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/',
        'src/test/',
        '**/*.d.ts',
      ]
    },
    include: ['src/**/*.{test,spec}.{ts,tsx}']
  }
});
```

### 2. Test Setup

```typescript
// src/test/setup.ts
import '@testing-library/jest-dom';
import { expect, afterEach } from 'vitest';
import { cleanup } from '@testing-library/react';
import matchers from '@testing-library/jest-dom/matchers';

// Extend matchers
expect.extend(matchers);

// Cleanup after each test
afterEach(() => {
  cleanup();
});
```

## Testing Patterns

### 1. Component Testing

```typescript
// src/components/Editor/__tests__/CodeBlock.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { describe, it, expect, vi } from 'vitest';
import { CodeBlock } from '../CodeBlock';

describe('CodeBlock', () => {
  it('renders with syntax highlighting', () => {
    const code = 'const x = 42;';
    render(<CodeBlock language="typescript" code={code} />);
    
    expect(screen.getByText('const')).toHaveClass('token keyword');
    expect(screen.getByText('42')).toHaveClass('token number');
  });

  it('handles language switching', async () => {
    const code = 'function test() {}';
    const { rerender } = render(
      <CodeBlock language="typescript" code={code} />
    );
    
    rerender(<CodeBlock language="python" code="def test():" />);
    expect(screen.getByText('def')).toHaveClass('token keyword');
  });
});
```

### 2. Hook Testing

```typescript
// src/hooks/__tests__/useCodeGeneration.test.ts
import { renderHook, act } from '@testing-library/react';
import { describe, it, expect, vi } from 'vitest';
import { useCodeGeneration } from '../useCodeGeneration';

describe('useCodeGeneration', () => {
  it('generates code successfully', async () => {
    const { result } = renderHook(() => useCodeGeneration());
    
    await act(async () => {
      const code = await result.current.generateCode(
        'Create a React component',
        { language: 'typescript' }
      );
      
      expect(code).toContain('React.FC');
      expect(result.current.isGenerating).toBe(false);
    });
  });

  it('handles errors gracefully', async () => {
    const { result } = renderHook(() => useCodeGeneration());
    
    await act(async () => {
      await expect(
        result.current.generateCode('', {})
      ).rejects.toThrow();
      
      expect(result.current.isGenerating).toBe(false);
    });
  });
});
```

### 3. Integration Testing

```typescript
// src/services/__tests__/OpenAIService.test.ts
import { describe, it, expect, vi } from 'vitest';
import { OpenAIService } from '../OpenAIService';

describe('OpenAIService', () => {
  it('generates code with proper context', async () => {
    const service = new OpenAIService();
    const mockCompletion = vi.spyOn(service['api'], 'createChatCompletion');
    
    await service.generateCode('Create a button', {
      language: 'typescript',
      framework: 'react'
    });
    
    expect(mockCompletion).toHaveBeenCalledWith(
      expect.objectContaining({
        messages: expect.arrayContaining([
          expect.objectContaining({
            content: expect.stringContaining('typescript')
          })
        ])
      })
    );
  });
});
```

## Test Coverage Requirements

```typescript
// .vitestrc.js
export default {
  coverageThreshold: {
    global: {
      statements: 80,
      branches: 75,
      functions: 80,
      lines: 80
    },
    './src/components/': {
      statements: 90,
      branches: 85,
      functions: 90,
      lines: 90
    }
  }
};
```

## Required Dependencies

```json
{
  "devDependencies": {
    "vitest": "^0.34.0",
    "@vitest/coverage-v8": "^0.34.0",
    "@testing-library/react": "^14.0.0",
    "@testing-library/jest-dom": "^6.0.0",
    "@testing-library/user-event": "^14.0.0",
    "jsdom": "^22.0.0"
  }
}
```

## Testing Scripts

```json
{
  "scripts": {
    "test": "vitest",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest run --coverage",
    "test:watch": "vitest watch",
    "test:ci": "vitest run --coverage --reporter=json --reporter=junit"
  }
}
```

## Best Practices

1. **Test Organization**
   - One test file per component/module
   - Clear test descriptions
   - Proper test isolation
   - Mock external dependencies

2. **Testing Hierarchy**
   ```
   src/
   ├── components/
   │   └── __tests__/
   │       ├── unit/
   │       └── integration/
   ├── hooks/
   │   └── __tests__/
   ├── services/
   │   └── __tests__/
   └── utils/
       └── __tests__/
   ```

3. **Performance**
   - Use test.concurrent for parallel execution
   - Mock heavy computations
   - Clear mocks between tests
   - Optimize test setup/teardown

4. **Coverage**
   - Track coverage trends
   - Focus on critical paths
   - Balance coverage vs maintenance
   - Document coverage exceptions

## CI/CD Integration

```yaml
# .github/workflows/test.yml
name: Tests
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npm run test:ci
      - uses: actions/upload-artifact@v3
        with:
          name: coverage
          path: coverage/
```

## Future Improvements

1. **Test Generation**
   - AI-assisted test generation
   - Automatic test maintenance
   - Smart test suggestions

2. **Performance Testing**
   - Component render benchmarks
   - API response timing tests
   - Memory leak detection

3. **Visual Testing**
   - Screenshot comparisons
   - Visual regression tests
   - Accessibility testing 