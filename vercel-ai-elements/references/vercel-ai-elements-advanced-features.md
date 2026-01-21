# Advanced Features

## Context

The Context component provides a comprehensive view of AI model usage through a compound component system. It displays context window utilization, token consumption breakdown, and cost estimation in an interactive hover card interface.

### Installation

```sh
npx ai-elements@latest add context
```

### Key Features

- Circular SVG progress ring showing context usage percentage
- Token breakdown: input, output, reasoning, and cached tokens
- Cost estimation using the tokenlens library
- Intelligent formatting: automatic token count formatting (K, M, B suffixes)
- Interactive hover card with detailed information
- Compound component architecture for flexible composition
- TypeScript support with full type definitions
- Theme integration with automatic color adaptation

### Props Reference

#### `<Context />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `maxTokens` | `number` | Total context window size in tokens | No |
| `usedTokens` | `number` | Number of tokens currently used | No |
| `usage` | `LanguageModelUsage` | Detailed token usage breakdown from AI SDK | Yes |
| `modelId` | `ModelId` | Model identifier for cost calculation | Yes |

#### `<ContextTrigger />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `children` | `React.ReactNode` | Custom trigger element | Yes |

#### `<ContextContent />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `className` | `string` | Additional CSS classes | Yes |

#### Usage Components

`<ContextInputUsage />`, `<ContextOutputUsage />`, `<ContextReasoningUsage />`, `<ContextCacheUsage />` all support:

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `children` | `React.ReactNode` | Custom content (default: token count and cost) | Yes |
| `className` | `string` | Additional CSS classes | Yes |

---

## Inline Citation

The InlineCitation component provides a way to display citations inline with text content, similar to academic papers. It shows detailed source information on hover with carousel navigation for multiple sources.

### Installation

```sh
npx ai-elements@latest add inline-citation
```

### Key Features

- Hover interaction to reveal detailed citation information
- Carousel navigation for multiple citations with prev/next controls
- Live index tracking showing current slide position
- Support for source titles, URLs, and descriptions
- Optional quote blocks for relevant excerpts
- Accessible design with keyboard navigation
- Smart badge display showing hostname and count
- Clean visual design that doesn't disrupt reading

### Props Reference

#### `<InlineCitation />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `[...props]` | `React.ComponentProps<"span">` | Standard span attributes | Yes |

#### `<InlineCitationCardTrigger />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `sources` | `string[]` | Array of source URLs (determines badge count) | No |

#### `<InlineCitationCarouselIndex />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `[...props]` | `React.ComponentProps<"div">` | Children override default index display | Yes |

#### `<InlineCitationSource />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `title` | `string` | Title of source | Yes |
| `url` | `string` | URL of source | Yes |
| `description` | `string` | Brief source description | Yes |

---

## Open In Chat

The OpenIn component provides a dropdown menu that allows users to open queries in different AI chat platforms with a single click.

### Installation

```sh
npx ai-elements@latest add open-in-chat
```

### Key Features

- Pre-configured links to popular AI chat platforms
- Context-based query passing
- Customizable dropdown trigger button
- Automatic URL parameter encoding
- Support for ChatGPT, Claude, T3 Chat, Scira AI, v0, and Cursor
- Branded icons for each platform
- Accessible dropdown menu with keyboard navigation

### Props Reference

#### `<OpenIn />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `query` | `string` | Query text sent to all AI platforms | No |
| `[...props]` | `React.ComponentProps<typeof DropdownMenu>` | Props spread to DropdownMenu | Yes |

#### `<OpenInTrigger />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `children` | `React.ReactNode` | Custom trigger button | Yes |

#### Platform Components

`<OpenInChatGPT />`, `<OpenInClaude />`, `<OpenInT3 />`, `<OpenInScira />`, `<OpenInv0 />`, `<OpenInCursor />`

Query is automatically provided via context from parent OpenIn component.

---

## Queue

The Queue component provides a flexible system for displaying lists of messages, todos, attachments, and collapsible sections. Perfect for showing AI workflow progress and pending tasks.

### Installation

```sh
npx ai-elements@latest add queue
```

### Key Features

- Flexible component system with composable parts
- Collapsible sections with smooth animations
- Completed/pending state indicators
- Built-in scroll area for long lists
- Attachment display with images and files
- Hover-revealed action buttons
- TypeScript support with comprehensive types
- Responsive design with mobile-friendly interactions
- Keyboard navigation and accessibility

### Props Reference

#### `<Queue />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `[...props]` | `React.ComponentProps<"div">` | Standard div attributes | Yes |

#### `<QueueSection />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `defaultOpen` | `boolean` | Whether section is open by default (default: true) | Yes |

#### `<QueueSectionLabel />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `label` | `string` | Label text to display | No |
| `count` | `number` | Count to display before label | Yes |
| `icon` | `React.ReactNode` | Optional icon before count | Yes |

#### `<QueueItemIndicator />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `completed` | `boolean` | Whether item is completed (default: false) | Yes |

#### `<QueueItemContent />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `completed` | `boolean` | Whether item is completed (affects text styling) | Yes |

#### `<QueueItemDescription />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `completed` | `boolean` | Whether item is completed | Yes |

---

## Sources

The Sources component allows users to view the sources or citations used to generate a response.

### Installation

```sh
npx ai-elements@latest add sources
```

### Key Features

- Collapsible component for viewing sources/citations
- Customizable trigger and content components
- Support for custom sources or citations
- Responsive design with mobile-friendly controls
- Clean, modern styling

### Props Reference

#### `<Sources />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `[...props]` | `React.HTMLAttributes<HTMLDivElement>` | Standard div attributes | Yes |

#### `<SourcesTrigger />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `count` | `number` | Number of sources to display | Yes |

#### `<Source />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `[...props]` | `React.AnchorHTMLAttributes<HTMLAnchorElement>` | Standard anchor attributes | Yes |

---

## Tool

The Tool component displays a collapsible interface for showing tool details, designed to work with the ToolUIPart type from the AI SDK.

### Installation

```sh
npx ai-elements@latest add tool
```

### Key Features

- Collapsible interface for showing/hiding tool details
- Visual status indicators with icons and badges
- Multiple tool execution states (pending, running, completed, error)
- Formatted parameter display with JSON syntax highlighting
- Result and error handling with appropriate styling
- Auto-opens completed tools by default
- Accessible keyboard navigation

### Props Reference

#### `<Tool />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `[...props]` | `React.ComponentProps<typeof Collapsible>` | Props spread to Collapsible | Yes |

#### `<ToolHeader />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `type` | `ToolUIPart["type"]` | The type/name of the tool | No |
| `state` | `ToolUIPart["state"]` | Current state (input-streaming, input-available, output-available, output-error) | No |
| `className` | `string` | Additional CSS classes | Yes |

#### `<ToolInput />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `input` | `ToolUIPart["input"]` | Input parameters passed to tool (displayed as JSON) | No |

#### `<ToolOutput />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `output` | `React.ReactNode` | Output/result of tool execution | No |
| `errorText` | `ToolUIPart["errorText"]` | Error message if execution failed | No |

---

## Web Preview

The WebPreview component provides a full-featured preview environment for displaying generated UI components and web content. Ideal for UI generators, documentation with interactive examples, and design system showcases.

### Installation

```sh
npx ai-elements@latest add web-preview
```

### Key Features

- Full-featured preview environment for web content
- Navigation controls with back/forward/reload buttons
- URL address bar for navigation
- Iframe-based content display
- Console logging support
- Responsive design modes
- Full-screen mode support
- Live documentation support
- Design system showcase capabilities

### Props Reference

#### `<WebPreview />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `defaultUrl` | `string` | Initial URL to load | Yes |
| `onUrlChange` | `function` | Callback when URL changes | Yes |
| `style` | `React.CSSProperties` | CSS styles (use for height control) | Yes |
| `className` | `string` | Additional CSS classes | Yes |

#### `<WebPreviewNavigation />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `children` | `React.ReactNode` | Navigation controls | Yes |

#### `<WebPreviewNavigationButton />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `icon` | `LucideIcon` | Icon component to display | No |
| `tooltip` | `string` | Hover tooltip text | Yes |
| `onClick` | `function` | Click handler | Yes |

#### `<WebPreviewBody />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `loading` | `React.ReactNode` | Custom loading indicator | Yes |
| `src` | `string` | Optional URL to load | Yes |
| `onLoad` | `function` | Callback when iframe loads | Yes |

#### `<WebPreviewConsole />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `logs` | `Array<{level, message, timestamp}>` | Console log entries to display | No |
