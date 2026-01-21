# AI Processing UI

## Branch

The `Branch` component manages multiple versions of AI messages, allowing users to navigate between different response branches. It provides a clean, modern interface with customizable themes and keyboard-accessible navigation buttons.

### Installation

```sh
npx ai-elements@latest add branch
```

### Basic Usage

```tsx
import {
  Branch,
  BranchMessages,
  BranchNext,
  BranchPage,
  BranchPrevious,
  BranchSelector,
} from '@/components/ai-elements/branch';
```

```tsx
<Branch defaultBranch={0}>
  <BranchMessages>
    <Message from="user">
      <MessageContent>Hello</MessageContent>
    </Message>
    <Message from="user">
      <MessageContent>Hi!</MessageContent>
    </Message>
  </BranchMessages>
  <BranchSelector from="user">
    <BranchPrevious />
    <BranchPage />
    <BranchNext />
  </BranchSelector>
</Branch>
```

### Key Features

- Context-based state management for multiple message branches
- Navigation controls for moving between branches
- Uses CSS to prevent re-rendering when switching branches
- Branch counter showing current position (e.g., "1 of 3")
- Automatic branch tracking and synchronization
- Responsive design with mobile-friendly controls
- Keyboard-accessible navigation buttons

### Props Reference

#### `<Branch />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `defaultBranch` | `number` | The index of branch to show by default (default: 0) | Yes |
| `onBranchChange` | `(branchIndex: number) => void` | Callback fired when branch changes | Yes |
| `[...props]` | `React.HTMLAttributes<HTMLDivElement>` | Any other props spread to root div | No |

#### `<BranchMessages />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `[...props]` | `React.HTMLAttributes<HTMLDivElement>` | Any other props spread to root div | No |

#### `<BranchSelector />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `from` | `UIMessage["role"]` | Aligns selector for user, assistant or system messages | No |
| `[...props]` | `React.HTMLAttributes<HTMLDivElement>` | Any other props spread to selector container | No |

---

## Chain of Thought

The `ChainOfThought` component provides a visual representation of an AI's reasoning process, showing step-by-step thinking with support for search results, images, and progress indicators.

### Installation

```sh
npx ai-elements@latest add chain-of-thought
```

### Basic Usage

```tsx
import {
  ChainOfThought,
  ChainOfThoughtContent,
  ChainOfThoughtHeader,
  ChainOfThoughtImage,
  ChainOfThoughtSearchResult,
  ChainOfThoughtStep,
} from '@/components/ai-elements/chain-of-thought';
```

```tsx
<ChainOfThought defaultOpen>
  <ChainOfThoughtHeader />
  <ChainOfThoughtContent>
    <ChainOfThoughtStep
      icon={SearchIcon}
      label="Searching for information"
      status="complete"
    >
      <ChainOfThoughtSearchResult>
        Result 1
      </ChainOfThoughtSearchResult>
    </ChainOfThoughtStep>
  </ChainOfThoughtContent>
</ChainOfThought>
```

### Key Features

- Collapsible interface with smooth animations
- Step-by-step visualization of AI reasoning process
- Support for different step statuses (complete, active, pending)
- Built-in search results display with badge styling
- Image support with captions for visual content
- Custom icons for different step types
- Fully typed with TypeScript
- Accessible with keyboard navigation support
- Smooth fade and slide animations

### Props Reference

#### `<ChainOfThought />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `open` | `boolean` | Controlled open state of collapsible | Yes |
| `defaultOpen` | `boolean` | Default open state when uncontrolled | Yes |
| `onOpenChange` | `(open: boolean) => void` | Callback when open state changes | Yes |

#### `<ChainOfThoughtHeader />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `children` | `React.ReactNode` | Custom header text (defaults to "Chain of Thought") | Yes |

#### `<ChainOfThoughtStep />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `icon` | `LucideIcon` | Icon to display for step | Yes |
| `label` | `string` | The main text label for step | No |
| `description` | `string` | Optional description text shown below label | Yes |
| `status` | `"complete" \| "active" \| "pending"` | Visual status of step | Yes |

#### `<ChainOfThoughtImage />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `caption` | `string` | Optional caption text displayed below image | Yes |

---

## Loader

The `Loader` component provides a spinning animation to indicate loading states in your AI applications.

### Installation

```sh
npx ai-elements@latest add loader
```

### Basic Usage

```tsx
import { Loader } from '@/components/ai-elements/loader';
```

```tsx
<Loader />
```

### Key Features

- Clean, modern spinning animation using CSS animations
- Configurable size with the `size` prop
- Built-in `animate-spin` animation with proper centering
- Uses `currentColor` for proper theme integration
- Responsive and accessible design

### Props Reference

#### `<Loader />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `size` | `number` | The size (width and height) of loader in pixels (default: 16) | Yes |
| `[...props]` | `React.HTMLAttributes<HTMLDivElement>` | Any other props spread to root div | Yes |

---

## Plan

The `Plan` component provides a flexible system for displaying AI-generated execution plans with collapsible content. Perfect for showing multi-step workflows and task breakdowns with support for streaming content.

### Installation

```sh
npx ai-elements@latest add plan
```

### Basic Usage

```tsx
import {
  Plan,
  PlanAction,
  PlanContent,
  PlanDescription,
  PlanFooter,
  PlanHeader,
  PlanTitle,
  PlanTrigger,
} from '@/components/ai-elements/plan';
```

```tsx
<Plan defaultOpen={false}>
  <PlanHeader>
    <div>
      <PlanTitle>Implement new feature</PlanTitle>
      <PlanDescription>
        Add authentication system with JWT tokens.
      </PlanDescription>
    </div>
    <PlanTrigger />
  </PlanHeader>
  <PlanContent>
    <div className="space-y-4 text-sm">
      <p>Implementation strategy details...</p>
    </div>
  </PlanContent>
  <PlanFooter>
    <Button>Execute Plan</Button>
  </PlanFooter>
</Plan>
```

### Key Features

- Collapsible content with smooth animations
- Streaming support with shimmer loading states
- Built on shadcn/ui Card and Collapsible components
- Customizable styling with Tailwind CSS
- Responsive design with mobile-friendly interactions
- Theme-aware with automatic dark mode support

### Props Reference

#### `<Plan />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `isStreaming` | `boolean` | Whether content is currently streaming (enables shimmer animations) | Yes |
| `defaultOpen` | `boolean` | Whether plan is expanded by default | Yes |
| `[...props]` | `React.ComponentProps<typeof Collapsible>` | Any other props spread to Collapsible component | Yes |

#### `<PlanTitle />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `children` | `string` | The title text (displays with shimmer when isStreaming) | No |

#### `<PlanDescription />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `children` | `string` | The description text (displays with shimmer when isStreaming) | No |

---

## Reasoning

The `Reasoning` component displays AI reasoning content, automatically opening during streaming and closing when finished.

### Installation

```sh
npx ai-elements@latest add reasoning
```

### Basic Usage

```tsx
import {
  Reasoning,
  ReasoningContent,
  ReasoningTrigger,
} from '@/components/ai-elements/reasoning';
```

```tsx
<Reasoning className="w-full" isStreaming={false}>
  <ReasoningTrigger />
  <ReasoningContent>I need to compute the square of 2.</ReasoningContent>
</Reasoning>
```

### Key Features

- Automatically opens when streaming and closes when finished
- Manual toggle control for user interaction
- Smooth animations powered by Radix UI
- Visual streaming indicator with pulsing animation
- Built with accessibility in mind
- Responsive design
- TypeScript support

### Props Reference

#### `<Reasoning />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `isStreaming` | `boolean` | Whether reasoning is currently streaming (auto-opens and closes) | Yes |
| `[...props]` | `React.ComponentProps<typeof Collapsible>` | Any other props spread to Collapsible component | Yes |

#### `<ReasoningTrigger />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `title` | `string` | Optional title to display (default: "Reasoning") | Yes |

---

## Shimmer

The `Shimmer` component provides an animated shimmer effect that sweeps across text, perfect for indicating loading states and progressive reveals.

### Installation

```sh
npx ai-elements@latest add shimmer
```

### Basic Usage

```tsx
import { Shimmer } from '@/components/ai-elements/shimmer';
```

```tsx
<Shimmer>Loading your response...</Shimmer>
```

### Key Features

- Smooth animated shimmer effect using CSS gradients and Framer Motion
- Customizable animation duration and spread
- Polymorphic component - render as any HTML element via `as` prop
- Automatic spread calculation based on text length
- Theme-aware styling using CSS custom properties
- Infinite looping animation
- Memoized for optimal performance
- TypeScript support

### Props Reference

#### `<Shimmer />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `children` | `string` | The text content to apply shimmer effect to | No |
| `as` | `ElementType` | The HTML element to render (default: "p") | Yes |
| `className` | `string` | Additional CSS classes | Yes |
| `duration` | `number` | Duration of shimmer animation in seconds (default: 2) | Yes |
| `spread` | `number` | Spread multiplier for shimmer gradient (default: 2) | Yes |
