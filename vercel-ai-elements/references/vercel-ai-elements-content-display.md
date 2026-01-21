# Content Display

## Artifact

The Artifact component provides a polished, professional container for displaying generated content with built-in header management, action buttons, and content areas. Perfect for showing code snippets, documents, designs, or any output that users might want to copy or interact with.

### Installation

```sh
npx ai-elements@latest add artifact
```

### Basic Usage

```tsx
import {
  Artifact,
  ArtifactHeader,
  ArtifactTitle,
  ArtifactDescription,
  ArtifactContent,
  ArtifactActions,
  ArtifactAction,
} from '@/components/ai-elements/artifact';

<Artifact>
  <ArtifactHeader>
    <div>
      <ArtifactTitle>Fibonacci Function</ArtifactTitle>
      <ArtifactDescription>TypeScript • Updated 2 minutes ago</ArtifactDescription>
    </div>
    <ArtifactActions>
      <ArtifactAction
        icon={CopyIcon}
        label="Copy"
        tooltip="Copy to clipboard"
        onClick={() => copyToClipboard(code)}
      />
    </ArtifactActions>
  </ArtifactHeader>
  <ArtifactContent>
    <CodeBlock code={code} language="typescript" />
  </ArtifactContent>
</Artifact>
```

### Key Features

- Structured container for displaying generated content (code, documents, outputs)
- Built-in header with title and description areas
- Flexible action button system with tooltips
- Consistent styling with border, shadow, and rounded corners
- Integrates seamlessly with other AI Elements components
- Supports copy, download, share, and custom actions
- Accessible design with proper labels and keyboard navigation
- Responsive layout that adapts to different screen sizes

### Props Reference

#### Artifact

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| className | string | Additional CSS classes | Yes |
| [...props] | React.HTMLAttributes<HTMLDivElement> | Standard div attributes | Yes |

#### ArtifactHeader

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| className | string | Additional CSS classes | Yes |
| [...props] | React.HTMLAttributes<HTMLDivElement> | Standard div attributes | Yes |

#### ArtifactTitle

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| children | React.ReactNode | Title text | No |
| className | string | Additional CSS classes | Yes |

#### ArtifactDescription

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| children | React.ReactNode | Description/metadata text | No |
| className | string | Additional CSS classes | Yes |

#### ArtifactActions

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| children | React.ReactNode | Action buttons | No |
| className | string | Additional CSS classes | Yes |

#### ArtifactAction

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| icon | LucideIcon | Icon component to display | No |
| label | string | Screen reader label | No |
| tooltip | string | Hover tooltip text | No |
| onClick | function | Button click handler | No |
| variant | string | Button style variant | Yes |
| disabled | boolean | Disabled state | Yes |

#### ArtifactContent

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| children | React.ReactNode | Content to display | No |
| className | string | Additional CSS classes | Yes |

---

## CodeBlock

The CodeBlock component provides syntax highlighting, line numbers, and copy to clipboard functionality for code blocks.

### Installation

```sh
npx ai-elements@latest add code-block
```

### Basic Usage

```tsx
import { CodeBlock, CodeBlockCopyButton } from '@/components/ai-elements/code-block';

<CodeBlock data={"console.log('hello world')"} language="jsx">
  <CodeBlockCopyButton
    onCopy={() => console.log('Copied code to clipboard')}
    onError={() => console.error('Failed to copy code to clipboard')}
  />
</CodeBlock>
```

### Key Features

- Syntax highlighting with react-syntax-highlighter
- Line numbers (optional)
- Copy to clipboard functionality
- Automatic light/dark theme switching
- Customizable styles
- Accessible design

### Props Reference

#### CodeBlock

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| code | string | The code content to display | No |
| language | string | Programming language for syntax highlighting | No |
| showLineNumbers | boolean | Whether to show line numbers (default: false) | Yes |
| children | React.ReactNode | Child elements (like CodeBlockCopyButton) | Yes |
| className | string | Additional CSS classes | Yes |
| [...props] | React.HTMLAttributes<HTMLDivElement> | Standard div attributes | Yes |

#### CodeBlockCopyButton

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| onCopy | () => void | Callback fired after successful copy | Yes |
| onError | (error: Error) => void | Callback fired if copying fails | Yes |
| timeout | number | How long to show copied state in ms (default: 2000) | Yes |
| children | React.ReactNode | Custom content for button (defaults to copy/check icons) | Yes |
| className | string | Additional CSS classes | Yes |
| [...props] | React.ComponentProps<typeof Button> | Button component props | Yes |

---

## Image

The Image component displays AI-generated images from the AI SDK. It accepts an Experimental_GeneratedImage object from the AI SDK's generateImage function and automatically renders it as an image.

### Installation

```sh
npx ai-elements@latest add image
```

### Basic Usage

```tsx
import { Image } from '@/components/ai-elements/image';

<Image
  base64="valid base64 string"
  mediaType="image/jpeg"
  uint8Array={new Uint8Array([])}
  alt="Example generated image"
  className="h-[150px] aspect-square border"
/>
```

### Key Features

- Accepts Experimental_GeneratedImage objects directly from AI SDK
- Automatically creates proper data URLs from base64-encoded image data
- Supports all standard HTML image attributes
- Responsive by default with max-w-full h-auto styling
- Customizable with additional CSS classes
- Includes proper TypeScript types for AI SDK compatibility

### Props Reference

#### Image

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| alt | string | Alternative text for the image | Yes |
| className | string | Additional CSS classes | Yes |
| [...props] | Experimental_GeneratedImage | Image data from AI SDK | Yes |

---

## Response

The Response component renders a Markdown response from a large language model. It uses Streamdown under the hood to render the markdown.

### Installation

```sh
npx ai-elements@latest add response
```

### Important: CSS Configuration

After adding the component, add the following to your `globals.css` file:

```css
@source "../node_modules/streamdown/dist/index.js";
```

This is **required** for the Response component to work properly.

### Basic Usage

```tsx
import { Response } from '@/components/ai-elements/response';

<Response>**Hi there.** I am an AI model designed to help you.</Response>
```

### Key Features

- Renders markdown content with support for paragraphs, links, and code blocks
- Supports GFM features like tables, task lists, and strikethrough text
- Supports rendering Math Equations via rehype-katex
- Smart streaming support - automatically completes incomplete formatting during real-time text streaming
- Code blocks rendered with syntax highlighting for various programming languages
- Code blocks include a button to easily copy code to clipboard
- Adapts to different screen sizes while maintaining readability
- Seamlessly integrates with both light and dark themes
- Customizable appearance through className props and Tailwind CSS utilities
- Built with accessibility in mind for all users

### Props Reference

#### Response

| Prop | Type | Description | Default | Optional |
|------|------|-------------|---------|----------|
| children | string | The markdown content to render | | No |
| parseIncompleteMarkdown | boolean | Whether to parse and fix incomplete markdown syntax | true | Yes |
| className | string | CSS class names to apply to wrapper div | | Yes |
| components | object | Custom React components for rendering markdown elements | | Yes |
| allowedImagePrefixes | string[] | Array of allowed URL prefixes for images | ["*"] | Yes |
| allowedLinkPrefixes | string[] | Array of allowed URL prefixes for links | ["*"] | Yes |
| defaultOrigin | string | Default origin for relative URLs | | Yes |
| rehypePlugins | array | Array of rehype plugins for HTML processing | [rehypeKatex] | Yes |
| remarkPlugins | array | Array of remark plugins for markdown processing | [remarkGfm, remarkMath] | Yes |
| [...props] | React.HTMLAttributes<HTMLDivElement> | Standard div attributes | | Yes |
