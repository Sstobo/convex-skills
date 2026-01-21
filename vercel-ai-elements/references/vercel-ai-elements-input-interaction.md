# Input & Interaction

## Actions

The Actions component provides a flexible row of action buttons for AI responses with common actions like retry, like, dislike, copy, and share.

### Installation

```sh
npx ai-elements@latest add actions
```

### Basic Usage

```tsx
import { Actions, Action } from '@/components/ai-elements/actions';
import { ThumbsUpIcon } from 'lucide-react';
```

```tsx
<Actions className="mt-2">
  <Action label="Like">
    <ThumbsUpIcon className="size-4" />
  </Action>
</Actions>
```

### Key Features

- Row of composable action buttons with consistent styling
- Support for custom actions with tooltips
- State management for toggle actions (like, dislike, favorite)
- Keyboard accessible with proper ARIA labels
- Clipboard and Web Share API integration
- TypeScript support with proper type definitions
- Consistent with design system styling

### Props

#### `<Actions />`

| Prop | Type | Description |
|------|------|-------------|
| `[...props]` | `React.HTMLAttributes<HTMLDivElement>` | HTML attributes to spread to the root div |

#### `<Action />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `tooltip` | `string` | Optional tooltip text shown on hover | Yes |
| `label` | `string` | Accessible label for screen readers | Yes |
| `[...props]` | `React.ComponentProps<typeof Button>` | Any other props spread to shadcn/ui Button | No |

---

## Prompt Input

The PromptInput component allows a user to send a message with file attachments to a large language model. It includes a textarea, file upload capabilities, a submit button, and a dropdown for selecting the model.

### Installation

```sh
npx ai-elements@latest add prompt-input
```

### Key Features

- Auto-resizing textarea that adjusts height based on content
- File attachment support with drag-and-drop
- Image preview for image attachments
- Configurable file constraints (max files, max size, accepted types)
- Automatic submit button icons based on status
- Support for keyboard shortcuts (Enter to submit, Shift+Enter for new line)
- Built-in model selection dropdown
- Built-in native speech recognition button (Web Speech API)
- Optional provider for lifted state management
- Form automatically resets on submit
- Responsive design with mobile-friendly controls
- Clean, modern styling with customizable themes
- Global document drop support (opt-in)

### Props

#### `<PromptInput />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `onSubmit` | `(message: PromptInputMessage, event: FormEvent) => void` | Handler called when form is submitted with message text and files | No |
| `accept` | `string` | File types to accept (e.g., "image/*"). Leave undefined for any | Yes |
| `multiple` | `boolean` | Whether to allow multiple file selection | Yes |
| `globalDrop` | `boolean` | When true, accepts file drops anywhere on document | Yes |
| `maxFiles` | `number` | Maximum number of files allowed | Yes |
| `maxFileSize` | `number` | Maximum file size in bytes | Yes |

#### `<PromptInputTextarea />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `[...props]` | `React.ComponentProps<typeof Textarea>` | Any other props spread to underlying Textarea component | Yes |

#### `<PromptInputSubmit />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `status` | `ChatStatus` | Current chat status (submitted, streaming, error) | Yes |
| `[...props]` | `React.ComponentProps<typeof Button>` | Any other props spread to underlying Button | Yes |

#### `<PromptInputSpeechButton />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `textareaRef` | `RefObject<HTMLTextAreaElement>` | Reference to textarea element to insert transcribed text | Yes |
| `onTranscriptionChange` | `(text: string) => void` | Callback fired when transcription text changes | Yes |

---

## Suggestion

The Suggestion component displays a horizontal row of clickable suggestions for user interaction.

### Installation

```sh
npx ai-elements@latest add suggestion
```

### Key Features

- Horizontal row of clickable suggestion buttons
- Flexible layout that wraps suggestions on smaller screens
- onClick callback that emits the selected suggestion string
- Clean, modern styling with hover effects
- Responsive design with mobile-friendly touch targets
- TypeScript support with proper type definitions

### Props

#### `<Suggestions />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `[...props]` | `React.ComponentProps<typeof ScrollArea>` | Any other props spread to underlying ScrollArea | Yes |

#### `<Suggestion />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `suggestion` | `string` | The suggestion string to display and emit on click | No |
| `onClick` | `(suggestion: string) => void` | Callback fired when suggestion is clicked | Yes |
| `[...props]` | `React.ComponentProps<typeof Button>` | Any other props spread to underlying Button | Yes |

---

## Task

The Task component provides a structured way to display task lists or workflow progress with collapsible details, status indicators, and progress tracking.

### Installation

```sh
npx ai-elements@latest add task
```

### Key Features

- Visual icons for pending, in-progress, completed, and error states
- Expandable content for task descriptions and additional information
- Built-in progress counter showing completed vs total tasks
- Optional progressive reveal of tasks with customizable timing
- Support for custom content within task items
- Full type safety with proper TypeScript definitions
- Keyboard navigation and screen reader support

### Props

#### `<Task />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `[...props]` | `React.ComponentProps<typeof Collapsible>` | Any other props spread to root Collapsible component | Yes |

#### `<TaskTrigger />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `title` | `string` | The title of the task to display in trigger | No |
| `[...props]` | `React.ComponentProps<typeof CollapsibleTrigger>` | Any other props spread to CollapsibleTrigger | Yes |

#### `<TaskContent />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `[...props]` | `React.ComponentProps<typeof CollapsibleContent>` | Any other props spread to CollapsibleContent | Yes |

#### `<TaskItem />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `[...props]` | `React.ComponentProps<"div">` | Any other props spread to underlying div | Yes |

#### `<TaskItemFile />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `[...props]` | `React.ComponentProps<"div">` | Any other props spread to underlying div | Yes |
