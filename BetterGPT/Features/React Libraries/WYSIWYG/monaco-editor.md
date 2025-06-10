# Monaco Editor in BetterGPT

## Why Both Monaco and TipTap?

While TipTap excels at rich text editing and document collaboration, Monaco Editor serves a different, complementary purpose in our application:

| Feature | TipTap | Monaco |
|---------|---------|---------|
| Rich Text Editing | ✅ Primary focus | ❌ Limited |
| Code Editing | ⚠️ Basic | ✅ Advanced |
| Collaboration | ✅ Built-in | ⚠️ Requires setup |
| IntelliSense | ❌ No | ✅ Full support |
| Language Services | ❌ No | ✅ Extensive |

### Key Reasons for Including Monaco

1. **Advanced Code Editing**
   - Language-aware autocompletion
   - Real-time error detection
   - Type information display
   - Code folding and navigation
   - Multi-cursor editing

2. **Development Experience**
   - VS Code-like experience
   - Integrated terminal support
   - Debugging capabilities
   - Git diff viewing

3. **Language Services**
   ```typescript
   import * as monaco from 'monaco-editor';
   
   // Example: TypeScript language features
   monaco.languages.typescript.typescriptDefaults.setCompilerOptions({
     target: monaco.languages.typescript.ScriptTarget.ES2015,
     allowNonTsExtensions: true,
     moduleResolution: monaco.languages.typescript.ModuleResolutionKind.NodeJs,
     module: monaco.languages.typescript.ModuleKind.CommonJS,
     noEmit: true,
     typeRoots: ["node_modules/@types"]
   });
   ```

## Integration with TipTap

```typescript
import { Extension } from '@tiptap/core';
import { MonacoEditorView } from '@monaco-editor/react';

const CodeBlockWithMonaco = Extension.create({
  name: 'codeBlockMonaco',

  addOptions() {
    return {
      languageDetection: true,
      defaultLanguage: 'typescript'
    };
  },

  addNodeView() {
    return ({ node, getPos }) => {
      const dom = document.createElement('div');
      
      const editor = <MonacoEditorView
        height="200px"
        language={node.attrs.language}
        value={node.textContent}
        onChange={(value) => {
          // Update TipTap content
          const pos = getPos();
          const transaction = this.editor.state.tr.setNodeMarkup(
            pos,
            undefined,
            { ...node.attrs, content: value }
          );
          this.editor.view.dispatch(transaction);
        }}
        options={{
          minimap: { enabled: false },
          lineNumbers: 'on',
          scrollBeyondLastLine: false,
          automaticLayout: true
        }}
      />;

      return {
        dom,
        contentDOM: dom,
        ignoreMutation: () => true,
        update: (node) => {
          // Handle updates from TipTap
          if (node.type.name !== 'codeBlockMonaco') return false;
          editor.setValue(node.textContent);
          return true;
        }
      };
    };
  }
});
```

## Usage Scenarios

1. **Document Code Blocks**
   - TipTap handles the document structure
   - Monaco provides advanced editing for code blocks

2. **Dedicated Code Files**
   - Full Monaco instance for .ts, .js, etc.
   - Language services and IDE features

3. **Code Review**
   - Monaco's diff viewer for changes
   - Inline commenting via TipTap

## Required Dependencies

```json
{
  "dependencies": {
    "@monaco-editor/react": "^4.6.0",
    "monaco-editor": "^0.45.0",
    "monaco-editor-webpack-plugin": "^7.1.0"
  }
}
```

## Configuration

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import monacoEditorPlugin from 'vite-plugin-monaco-editor';

export default defineConfig({
  plugins: [
    monacoEditorPlugin({
      languageWorkers: ['typescript', 'javascript', 'html', 'css']
    })
  ]
});
```

## Performance Considerations

1. **Lazy Loading**
   - Load Monaco only when needed
   - Use dynamic imports for code blocks

2. **Worker Management**
   - Configure specific language workers
   - Control memory usage

3. **Editor Instance Lifecycle**
   - Proper disposal of editors
   - Memory leak prevention

## Integration Best Practices

1. Use Monaco for:
   - Dedicated code files
   - Complex code blocks
   - Diff viewing
   - Debug sessions

2. Use TipTap for:
   - Document structure
   - Rich text content
   - Collaboration
   - Simple code snippets

3. Hybrid Approach:
   - TipTap manages document flow
   - Monaco handles code-specific editing
   - Seamless switching between modes 