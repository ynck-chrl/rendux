![rendux Logo](https://raw.githubusercontent.com/ynck-chrl/rendux/master/rendux-logo.jpg)

# rendux

## Introduction

`rendux` is a lightweight dependency-free templating engine designed for use within Custom Elements (Web Components) and plain HTML. It provides reactive templating features through directives, making it easy to create dynamic, state-driven components without a full framework.

### Size

- Minified: 13.2 KB (ESM), 13.3 KB (CJS)
- Gzipped: ~4.7 KB

## Installation

### Via NPM

```bash
npm install rendux
```

Then import in your project:

```js
// ES Module (recommended)
import { rendux } from '@ynck/rendux';

// CommonJS
const { rendux } = require('rendux');
```

### Via CDN (No Build Step)

#### unpkg

```html
<!-- ES Module -->
<script type="module">
  import { rendux } from 'https://unpkg.com/@ynck/rendux@latest/dist/rendux.min.js';
</script>

<!-- Or use specific version -->
<script type="module">
  import { rendux } from 'https://unpkg.com/@ynck/rendux@0.93.2/dist/rendux.min.js';
</script>
```

#### esm.sh

```html
<!-- Latest version -->
<script type="module">
  import { rendux } from 'https://esm.sh/@ynck/rendux@latest'
</script>

<!-- Or with specific version -->
<script type="module">
  import { rendux } from 'https://esm.sh/@ynck/rendux@0.93.2'
</script>
```

#### Skypack

```html
<script type="module">
  import { rendux } from 'https://cdn.skypack.dev/@ynck/rendux'
</script>

<!-- Or with specific version -->
<script type="module">
  import { rendux } from 'https://cdn.skypack.dev/@ynck/rendux@0.93.2';
</script>
```

### From Source (Local Development)

Clone the repository and use the source directly:

```bash
git clone https://github.com/ynck-chrl/rendux.git
cd rendux
```

Then import from source:

```html
<script type="module">
  import { rendux } from './src/rendux.js';
</script>
```

Or if using in a Node.js project:

```js
import { rendux } from './path/to/rendux/src/rendux.js';
```

### Build Outputs

When installed via NPM, rendux provides multiple build outputs:

- **ESM (ES Modules)**: `dist/rendux.min.js` - For modern browsers and bundlers
- **CommonJS**: `dist/rendux.cjs.min.js` - For Node.js and legacy tools
- **Source Maps**: Available for all builds (`.map` files)

The `package.json` exports field automatically selects the correct build:

```json
{
  "exports": {
    ".": {
      "import": "./dist/rendux.min.js",
      "require": "./dist/rendux.cjs.min.js"
    }
  }
}
```

### TypeScript Support

TypeScript definitions are not yet included. You can create a declaration file:

```typescript
// rendux.d.ts
declare module 'rendux' {
  export function rendux(state: any, element: HTMLElement): void;
  export function use(plugin: any): typeof rendux;
  // Add other exports as needed
}
```



## Core Concepts

`rendux` is a function that takes a state object and a DOM element, then processes templating directives within that element. It provides several directives to handle common UI patterns:

- `x-if` for conditional rendering
- `x-for` for list rendering
- `x-class` for conditional classes
- `x-*` for dynamic attributes
- `render` for text content interpolation
- `@event` for event handling
- `render.plugin` for custom plugin execution

## Logging System

rendux includes a flexible logging system controlled by the `logs` attribute. You can enable logging for specific features:

```html
<!-- Enable all logs -->
<my-component logs="all"></my-component>

<!-- Enable specific feature logs -->
<my-component logs="for, if, attr, class, render, plugins"></my-component>
```

Available log categories:
- `for`: x-for loop processing
- `if`: x-if conditional rendering
- `attr`: x-attr attribute processing
- `class`: x-class processing
- `render`: Text content rendering
- `plugins`: Plugin execution and results

## Basic Usage

### With Custom Elements (Web Components)

#### With Internal State and Shadow DOM

```js
import { rendux } from 'rendux';

class MyElement extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({ mode: 'open' });
    
    // Internal component state
    this.state = {
      title: 'User List',
      users: [
        { name: 'Alice', active: true },
        { name: 'Bob', active: false }
      ]
    };
  }

  connectedCallback() {
    this.shadowRoot.innerHTML = `
      <div>
        <h2 render="title"></h2>
        <template x-for="user of users">
          <div x-class="(active, user.active)">
            <span render="user.name"></span>
            <button @click="toggleUser(user)">Toggle</button>
          </div>
        </template>
      </div>
    `;
    // Pass state and element - renders into shadowRoot
    rendux(this.state, this);
  }

  toggleUser(user) {
    user.active = !user.active;
    // Re-render after state change
    rendux(this.state, this);
  }
}
```

#### With Internal State and Light DOM

```js
import { rendux } from 'rendux';

class MyElement extends HTMLElement {
  constructor() {
    super();
    
    // Internal component state
    this.state = {
      title: 'User List',
      users: [
        { name: 'Alice', active: true },
        { name: 'Bob', active: false }
      ]
    };
  }

  connectedCallback() {
    // Render directly into light DOM (no shadow root)
    this.innerHTML = `
      <div>
        <h2 render="title"></h2>
        <template x-for="user of users">
          <div x-class="(active, user.active)">
            <span render="user.name"></span>
            <button @click="toggleUser(user)">Toggle</button>
          </div>
        </template>
      </div>
    `;
    // Pass state and element - renders into element itself (light DOM)
    rendux(this.state, this);
  }

  toggleUser(user) {
    user.active = !user.active;
    // Re-render after state change
    rendux(this.state, this);
  }
}
```

#### With External/Global State

```js
import { rendux } from 'rendux';
import { state } from './state.js'; // External state (could be from a store)

class MyGlobalUsers extends HTMLElement {
  connectedCallback() {
    this.innerHTML = `
      <div>
        <h2 render="title"></h2>
        <template x-for="(user, userIndex) of users">
          <div x-class="(active, user.active) (non-active, !user.active)">
            <span render="user.name"></span>
            <span render="userIndex"></span>
            <button @click="toggleUser(user)">Toggle</button>
          </div>
        </template>
      </div>
    `;
    // Use external state with this element
    rendux(state, this);
  }

  toggleUser(user) {
    user.active = !user.active;
    // Re-render with external state
    rendux(state, this);
  }
}
```

**Key Points:**
- **First argument**: Always the state object (can be `this.state`, an external state, or any object/array)
- **Second argument**: Always the DOM element (`this` in Custom Elements)
- **Rendering target**: If element has `shadowRoot`, renders into it; otherwise renders into the element itself (light DOM)
- **Template expressions**: Reference properties directly from the state object (e.g., `title`, `users`, not `state.title`)


### With Plain HTML

You can also use rendux with plain HTML:

```html
<!DOCTYPE html>
<html>
<head>
  <script type="module">
    import { rendux } from './src/rendux.js';
    
    // Create a state object with your data
    const state = {
      title: 'User List',
      users: [
        { name: 'Alice', active: true },
        { name: 'Bob', active: false }
      ]
    };
    
    // Get the container element
    const container = document.querySelector('#app');
    
    // Initial render
    rendux(state, container);
    
    // To update after state changes:
    // state.users.push({ name: 'Charlie', active: true });
    // rendux(state, container);
  </script>
</head>
<body>
  <div id="app">
    <h2 render="title">User List</h2>
    <template x-for="user of users">
      <div x-class="(active, user.active)">
        <span render="user.name"></span>
      </div>
    </template>
  </div>
</body>
</html>
```

**Key Points for Plain HTML Usage:**
- Create a state object with your data
- Pass the state and container element to `rendux(state, element)`
- Call `rendux(state, element)` again after state changes to re-render
- Event handlers (like `@click`) can reference methods if you add them to the element's prototype or use inline expressions

### With CommonJS (Node.js)

```js
// Import using CommonJS
const { rendux } = require('rendux');

// Create a state object with your data
const state = {
  title: 'Server Context',
  users: [
    { name: 'Alice', active: true },
    { name: 'Bob', active: false }
  ]
};

// Note: rendux requires a DOM environment
// For Node.js, use with a DOM implementation like jsdom
const { JSDOM } = require('jsdom');
const dom = new JSDOM(`
  <div id="app">
    <h2 render="title">User List</h2>
    <template x-for="user of users">
      <div x-class="(active, user.active)">
        <span render="user.name"></span>
      </div>
    </template>
  </div>
`);

// Set up global document for rendux
global.document = dom.window.document;

// Get the container element
const container = dom.window.document.querySelector('#app');

// Call rendux with state and element
rendux(state, container);
```

**When to Use `rendux.cjs`:**

Use the CommonJS build (`dist/rendux.cjs`) in these scenarios:

1. **Node.js Build Tools**: When using bundlers or build systems that require CommonJS (older Webpack configs, some testing frameworks)
2. **Server-Side Rendering (SSR)**: When rendering components on the server with a DOM implementation like jsdom or happy-dom
3. **Legacy Node.js Projects**: Projects that haven't migrated to ES modules (`type: "module"` in package.json)
4. **Testing Environments**: Jest or other test runners configured for CommonJS (though modern Jest supports ESM)
5. **npm Package Distribution**: When publishing packages that need to support both module systems

**Note**: rendux requires a DOM environment. For Node.js/server usage, you must provide a DOM implementation (jsdom, happy-dom, linkedom) since rendux uses `document`, `Element`, `querySelectorAll`, etc.

For modern projects with native ESM support, prefer the ESM build (`dist/rendux.js`) instead.

### With Reactive State Libraries

rendux works seamlessly with reactive state management libraries. Here's an example using a Proxy-based reactive state:

```js
import { rendux } from 'rendux';
import { restate } from 'restate'; // Or any reactive state library

// Create reactive state
const state = restate({
  title: 'User List',
  users: [
    { name: 'Alice', active: true },
    { name: 'Bob', active: false }
  ]
});

class MyComponent extends HTMLElement {
  connectedCallback() {
    this.innerHTML = `
      <div>
        <h2 render="title"></h2>
        <template x-for="user of users">
          <div x-class="(active, user.active)">
            <span render="user.name"></span>
            <button @click="toggleUser(user)">Toggle</button>
          </div>
        </template>
      </div>
    `;
    
    // Initial render
    rendux(state, this);
    
    // Set up automatic re-rendering on state changes
    state.$onChange(() => {
      rendux(state, this);
    });
  }
  
  toggleUser(user) {
    // Just mutate - reactive state will trigger re-render
    user.active = !user.active;
  }
}
```

**Benefits of Reactive State:**
- Automatic re-rendering when state changes
- No need to manually call `rendux()` after every update
- Works with any Proxy-based reactive library
- rendux handles circular references and deep objects safely

## Directives

### 1. Conditional Rendering (`x-if`)

```html
<div x-if="someCondition">
  Only shown when condition is true
</div>
```

The element is completely removed from the DOM (not just hidden) when the condition is false.

### 2. List Rendering (`x-for`)

Supports two syntaxes:
```html
<!-- Simple iteration -->
<template x-for="item of items">
  <div render="item.name"></div>
</template>

<!-- With index -->
<template x-for="(item, index) of items">
  <div render="index + ': ' + item.name"></div>
</template>
```

Features:
- Uses `<template>` tag to define the repeatable content
- Maintains its own render cache per iteration
- Supports deep change detection for arrays
- Provides iteration context to nested elements

### 3. Class Binding (`x-class`)

Two syntaxes available:

```html
<!-- Single class with condition -->
<div x-class="highlight, isActive">
</div>

<!-- Multiple class/condition pairs -->
<div x-class="(selected, isSelected) (highlight, isHighlighted)">
</div>
```

### 4. Dynamic Attributes (`x-*`)

Any attribute prefixed with `x-` (except special directives) becomes dynamic:

```html
<input x-value="inputValue">
<img x-src="imageUrl">
<div x-data-id="getId()">
```

### 5. Text Content (`render`)

Renders dynamic text content:

```html
<!-- Simple value -->
<span render="message"></span>

<!-- With condition -->
<span render="message, isVisible">
  Default text when not visible
</span>
```

Features:
- Supports expressions
- Can access component methods and properties
- Handles objects (converts to JSON)
- Supports conditional rendering

### 6. Event Handling (`@event`)

```html
<!-- Click events -->
<button @click="handleClick()">Click me</button>

<!-- Form events -->
<input @input="updateValue(event.target.value)"
       @focus="handleFocus()"
       @blur="handleBlur()">

<!-- Keyboard events -->
<input @keyup="handleKeyUp(event)"
       @keydown="handleKeyDown(event)">
```

The event object is automatically available in handlers as `event`.

## Plugin System

rendux includes a powerful plugin system that allows you to extend templating functionality with custom directives.

### Using Plugins

Plugins are registered using the `use()` method, which is chainable:

```js
import { rendux } from 'rendux';
import { myCustomPlugin } from './plugins/my-plugin.js';

// Register plugin (chainable)
rendux.use(myCustomPlugin);

// Or chain multiple plugins
rendux
  .use(plugin1)
  .use(plugin2)
  .use(plugin3);
```

### Using Plugin Directives in Templates

Once registered, plugins can be used with the `render.pluginName` attribute:

```html
<!-- Using a custom formatter plugin -->
<span render.format="user.name">Default Name</span>

<!-- Using a custom i18n plugin -->
<p render.i18n="greeting">Hello!</p>

<!-- Generic plugin syntax -->
<span render.plugin="myPlugin('arg1', 'arg2')">Fallback</span>
```

### Creating Custom Plugins

A plugin is an object with specific properties and methods:

```js
export const myPlugin = {
  // Required: Plugin name (used in render.pluginName)
  name: 'myPlugin',
  
  // Required: Main execution function
  execute(value, ...args) {
    // Process the value and return result
    // The returned string becomes the element's textContent
    return processedValue;
  },
  
  // Optional: Called once before any plugin executes
  onBeforeRender(root, component) {
    // Initialize plugin state
    // Access to root element and component context
  },
  
  // Optional: Called before each element execution
  onBeforeExecute(element, rawValue, context) {
    // Setup for this specific element
    // Can modify element attributes, add classes, etc.
  },
  
  // Optional: Called after each element execution
  onAfterExecute(element, rawValue, context) {
    // Cleanup or post-processing for this element
  },
  
  // Optional: Called once after all plugins execute (can be async)
  async onAfterRender(root, context) {
    // Final cleanup or async operations
    // This is the only lifecycle hook that supports async
  }
};
```

### Plugin Example: Text Formatter

Here's a complete example of a text formatting plugin:

```js
export const formatPlugin = {
  name: 'format',
  
  execute(value, format = 'uppercase') {
    if (typeof value !== 'string') {
      value = String(value);
    }
    
    switch(format) {
      case 'uppercase':
        return value.toUpperCase();
      case 'lowercase':
        return value.toLowerCase();
      case 'capitalize':
        return value.charAt(0).toUpperCase() + value.slice(1).toLowerCase();
      case 'reverse':
        return value.split('').reverse().join('');
      default:
        return value;
    }
  }
};

// Register the plugin
rendux.use(formatPlugin);
```

Usage in template:

```html
<span render.format="user.name, 'capitalize'">John Doe</span>
<span render.format="title, 'uppercase'">Hello World</span>
```

### Plugin Example: Conditional Formatter

A more advanced plugin with lifecycle hooks:

```js
export const conditionalPlugin = {
  name: 'showIf',
  
  onBeforeRender(root, component) {
    // Store original content for elements using this plugin
    this.originalContent = new WeakMap();
  },
  
  execute(condition, trueValue, falseValue = '') {
    // Evaluate condition and return appropriate value
    return condition ? trueValue : falseValue;
  },
  
  onAfterExecute(element, rawValue, context) {
    // Add a custom class based on the condition
    const condition = rawValue.split(',')[0].trim();
    if (condition) {
      element.classList.add('visible');
    } else {
      element.classList.remove('visible');
    }
  }
};
```

### Plugin Best Practices

1. **Validate Inputs**: Always validate and sanitize plugin inputs
2. **Handle Errors Gracefully**: Use try-catch to prevent plugin errors from breaking rendering
3. **Keep Plugins Focused**: Each plugin should do one thing well
4. **Document Parameters**: Clearly document what arguments your plugin expects
5. **Return Strings**: The `execute()` method should return a string (or value that converts to string)
6. **Avoid Side Effects**: Minimize DOM manipulation outside of the element being processed

### Security Considerations for Plugins

When creating plugins, be mindful of security:

```js
export const safePlugin = {
  name: 'safe',
  
  execute(userInput) {
    // Validate input type
    if (typeof userInput !== 'string') {
      return '';
    }
    
    // Sanitize HTML entities
    return userInput.replace(/[<>&"']/g, char => ({
      '<': '&lt;',
      '>': '&gt;',
      '&': '&amp;',
      '"': '&quot;',
      "'": '&#39;'
    })[char]);
  }
};
```

## Security

rendux is designed with security as a core principle, implementing several protective measures to prevent common web vulnerabilities:

### XSS Prevention Through Safe Content Handling

**No innerHTML Usage for Dynamic Content**
- rendux never uses `innerHTML` for dynamic content rendering
- All user data is inserted using `textContent`, which automatically escapes HTML
- This prevents malicious script injection through user-controlled data

```js
// Safe: User input is escaped automatically
<span render="userInput"></span>  // Uses textContent internally

// Unsafe (what rendux avoids):
element.innerHTML = userInput;    // Could execute scripts
```

### Sandboxed Expression Evaluator

**Custom Mini-Evaluator Instead of eval()**
- rendux includes a custom expression parser that never uses `eval()` or `Function()`
- Only supports safe operations: property access, method calls, basic operators
- Prevents arbitrary code execution through template expressions

**Whitelisted Global Access**
- Only safe global objects are accessible: `Math`, `Date`, `Number`, `String`, `Boolean`, `JSON`, `Array`
- No access to dangerous globals like `window`, `document`, or `eval`
- Controlled execution environment limits potential attack vectors

```js
// Safe expressions supported:
render="user.name"                    ✓
render="Math.max(a, b)"              ✓
render="items.length > 0 ? 'Yes' : 'No'" ✓

// Dangerous expressions blocked:
render="eval('malicious code')"       ✗ (eval not accessible)
render="window.location = 'evil.com'" ✗ (window not accessible)
```

### Template Content Security

**Safe Template Processing**
- Template elements (`<template x-for>`) use `.content` property, not innerHTML
- Attributes are processed safely without executing embedded scripts
- Event handlers are properly scoped and don't use string-to-function conversion

### Best Practices for Secure Usage

1. **Validate Inputs**: Always validate and sanitize user inputs before adding them to state
2. **Context Isolation**: Each component maintains its own isolated context
3. **No Script Injection**: User data in expressions is evaluated safely, not executed as code
4. **Safe State Updates**: Mutate state objects directly and call `rendux()` to re-render, avoiding innerHTML manipulation

```js
// Safe state update example
class MyComponent extends HTMLElement {
  constructor() {
    super();
    this.state = {
      userInput: ''
    };
  }
  
  handleInput(event) {
    // Validate and sanitize
    const value = event.target.value;
    if (typeof value === 'string' && value.length < 100) {
      this.state.userInput = value.trim();
      rendux(this.state, this); // Safe re-render
    }
  }
}
```

This security model makes rendux suitable for applications handling user-generated content while maintaining the flexibility of a templating engine.

### Security Audit Summary

rendux has been designed and audited to ensure maximum security for web applications:

#### Core Engine Security ✓

**No Dangerous Functions**
- ✓ No `eval()` or `Function()` constructor used
- ✓ No `innerHTML` for dynamic content
- ✓ No `outerHTML` manipulation
- ✓ No `document.write()`

**Safe Content Rendering**
- ✓ All dynamic content uses `textContent` (auto-escapes HTML)
- ✓ User input like `<script>alert('XSS')</script>` is rendered as escaped text, never executed
- ✓ Plugin outputs are also rendered via `textContent`

**Sandboxed Expression Evaluator**
- ✓ Custom parser with whitelisted globals only
- ✓ Allowed: `Math`, `Date`, `Number`, `String`, `Boolean`, `JSON`, `Array`, `parseInt`, `parseFloat`, `isFinite`
- ✗ Blocked: `window`, `document`, `eval`, `fetch`, `XMLHttpRequest`, `localStorage`, `location`

**Safe Template Processing**
- ✓ Templates use `.content` property (no innerHTML)
- ✓ Event handlers use sandboxed evaluator (no string-to-function conversion)

#### Plugin System Security

**Framework Security ✓**
- ✓ Plugin output automatically escaped via `textContent`
- ✓ Plugin execution wrapped in try-catch for graceful error handling
- ✓ Plugins cannot access dangerous globals

**Developer Responsibility ⚠️**

Plugin developers **must validate and sanitize** all inputs in their `execute()` method:

```js
// ✓ SECURE: Properly validated plugin
export const securePlugin = {
  name: 'format',
  
  execute(value, options = {}) {
    // 1. Type validation
    if (typeof value !== 'string') {
      console.warn('Invalid input type, expected string');
      return '';
    }
    
    // 2. Length validation
    if (value.length > 10000) {
      console.warn('Input too long');
      return value.substring(0, 10000);
    }
    
    // 3. Sanitization (if needed for your use case)
    const sanitized = value.replace(/[<>&"']/g, char => ({
      '<': '&lt;', '>': '&gt;', '&': '&amp;',
      '"': '&quot;', "'": '&#39;'
    })[char]);
    
    // 4. Safe processing
    return sanitized.toUpperCase();
  }
};

// ✗ INSECURE: No validation
export const insecurePlugin = {
  name: 'unsafe',
  execute(value) {
    // Dangerous: No type checking, no validation
    return value.toUpperCase(); // Crashes if value is not a string
  }
};
```

**Plugin Security Checklist**

When creating plugins, ensure you:

1. ✓ **Validate input types** - Check `typeof` before processing
2. ✓ **Validate input length** - Prevent resource exhaustion
3. ✓ **Sanitize if needed** - Escape special characters for your use case
4. ✓ **Handle errors gracefully** - Use try-catch to prevent crashes
5. ✓ **Return safe values** - Always return strings or values that convert to string
6. ✓ **Document requirements** - Clearly document expected input types and formats
7. ✓ **Avoid DOM manipulation** - Minimize direct DOM changes outside the element
8. ✓ **Test with malicious input** - Test with XSS payloads, large inputs, invalid types

#### Security Guarantees

| Feature | Core Engine | Plugin System |
|---------|-------------|---------------|
| XSS Prevention | ✓ Guaranteed | ✓ Framework protected, ⚠️ validate inputs |
| Code Injection | ✓ Blocked | ✓ Blocked |
| HTML Escaping | ✓ Automatic | ✓ Automatic |
| Global Access | ✓ Whitelisted only | ✓ Whitelisted only |
| Expression Safety | ✓ Sandboxed | ✓ Sandboxed |
| Input Validation | ✓ Type-checked | ⚠️ Developer responsibility |

**Verdict**: rendux provides a **secure foundation** for web applications. The core engine is hardened against XSS and code injection attacks. Plugin developers must follow security best practices and validate all inputs to maintain the security guarantee.

## License

This project is available under two licenses:
- Non-commercial use: See LICENSE-NONCOMMERCIAL.md
- Commercial use: See EULA-COMMERCIAL.md