# AI Elements Workflow Components Reference

Detailed API documentation for all workflow visualization components.

## Canvas

A React Flow-based canvas component for building interactive node-based interfaces.

### Features

- Pre-configured React Flow canvas with AI-optimized defaults
- Pan on scroll enabled for intuitive navigation
- Selection on drag for multi-node operations
- Customizable background color using CSS variables
- Delete key support (Backspace and Delete keys)
- Auto-fit view to show all nodes
- Disabled double-click zoom for better UX
- Disabled pan on drag to prevent accidental canvas movement
- Fully compatible with React Flow props and API

### Props

| Prop | Type | Description |
|------|------|-------------|
| children | ReactNode | Child components (Controls, Panel, etc.) |
| ...props | ReactFlowProps | All React Flow props supported |

### Installation

```bash
npx ai-elements@latest add canvas
```

---

## Connection

A custom connection line component with animated bezier curve styling.

### Features

- Smooth bezier curve animation for connection lines
- Visual indicator circle at the target position
- Theme-aware styling using CSS variables
- Cubic bezier curve calculation for natural flow
- Lightweight implementation with minimal props
- Full TypeScript support with React Flow types

### Props

| Prop | Type | Description |
|------|------|-------------|
| fromX | number | Starting X coordinate |
| fromY | number | Starting Y coordinate |
| toX | number | Ending X coordinate |
| toY | number | Ending Y coordinate |

### Installation

```bash
npx ai-elements@latest add connection
```

---

## Controls

A styled controls component with zoom and fit view functionality.

### Features

- Zoom in/out controls
- Fit view button to center and scale content
- Rounded pill design with backdrop blur
- Theme-aware card background
- Subtle drop shadow for depth
- Full TypeScript support
- Compatible with all React Flow control features

### Props

| Prop | Type | Description |
|------|------|-------------|
| className | string | Optional CSS class |
| ...props | ComponentProps<typeof Controls> | React Flow Controls props |

### Installation

```bash
npx ai-elements@latest add controls
```

---

## Edge

Customizable edge components with animated and temporary states.

### Edge Types

#### Edge.Temporary

A dashed edge style for temporary or preview connections. Uses a simple Bezier path with a dashed stroke pattern.

#### Edge.Animated

A solid edge with an animated circle that moves along the path. The animation repeats indefinitely with a 2-second duration, providing visual feedback for active connections.

### Features

- Two distinct edge types: Temporary and Animated
- Temporary edges use dashed lines with ring color
- Animated edges include a moving circle indicator
- Automatic handle position calculation
- Smart offset calculation based on handle type and position
- Uses Bezier curves for smooth, natural-looking connections
- Fully compatible with React Flow's edge system

### Props

| Prop | Type | Description |
|------|------|-------------|
| id | string | Edge identifier |
| source | string | Source node ID |
| target | string | Target node ID |
| sourceX | number | Source X coordinate |
| sourceY | number | Source Y coordinate |
| targetX | number | Target X coordinate |
| targetY | number | Target Y coordinate |
| sourcePosition | Position | Source handle position |
| targetPosition | Position | Target handle position |
| markerEnd | string | Optional end marker |
| style | React.CSSProperties | Custom styles |

### Installation

```bash
npx ai-elements@latest add edge
```

---

## Node

A composable node component with Card-based styling.

### Sub-components

- `Node` - Main container with handle support
- `NodeHeader` - Header section
- `NodeTitle` - Title text
- `NodeDescription` - Description text
- `NodeAction` - Action area
- `NodeContent` - Main content area
- `NodeFooter` - Footer section

### Features

- Built on shadcn/ui Card components for consistent styling
- Automatic handle placement (left for target, right for source)
- Composable sub-components
- Semantic structure for organizing node information
- Pre-styled sections with borders and backgrounds
- Responsive sizing with fixed small width
- Full TypeScript support

### Props

#### `<Node />`

| Prop | Type | Description |
|------|------|-------------|
| handles | `{ target: boolean; source: boolean }` | Configure connection handles |
| className | string | Optional CSS class |
| ...props | ComponentProps<typeof Card> | Card props |

#### `<NodeHeader />`

| Prop | Type | Description |
|------|------|-------------|
| className | string | Optional CSS class |
| ...props | ComponentProps<typeof CardHeader> | CardHeader props |

#### `<NodeTitle />`

| Prop | Type | Description |
|------|------|-------------|
| ...props | ComponentProps<typeof CardTitle> | CardTitle props |

#### `<NodeDescription />`

| Prop | Type | Description |
|------|------|-------------|
| ...props | ComponentProps<typeof CardDescription> | CardDescription props |

#### `<NodeAction />`

| Prop | Type | Description |
|------|------|-------------|
| ...props | ComponentProps<typeof CardAction> | CardAction props |

#### `<NodeContent />`

| Prop | Type | Description |
|------|------|-------------|
| className | string | Optional CSS class |
| ...props | ComponentProps<typeof CardContent> | CardContent props |

#### `<NodeFooter />`

| Prop | Type | Description |
|------|------|-------------|
| className | string | Optional CSS class |
| ...props | ComponentProps<typeof CardFooter> | CardFooter props |

### Installation

```bash
npx ai-elements@latest add node
```

---

## Panel

A styled panel component for positioning custom UI elements.

### Features

- Flexible positioning (top-left, top-right, bottom-left, bottom-right, top-center, bottom-center)
- Rounded pill design with backdrop blur
- Theme-aware card background
- Flexbox layout for easy content alignment
- Subtle drop shadow for depth
- Full TypeScript support
- Compatible with React Flow's panel system

### Props

| Prop | Type | Description |
|------|------|-------------|
| position | `'top-left' \| 'top-center' \| 'top-right' \| 'bottom-left' \| 'bottom-center' \| 'bottom-right'` | Panel position |
| className | string | Optional CSS class |
| ...props | ComponentProps<typeof Panel> | React Flow Panel props |

### Installation

```bash
npx ai-elements@latest add panel
```

---

## Toolbar

A styled toolbar component for React Flow nodes with flexible positioning.

### Features

- Attaches to any React Flow node
- Bottom positioning by default
- Rounded card design with border
- Theme-aware background styling
- Flexbox layout with gap spacing
- Full TypeScript support
- Compatible with all React Flow NodeToolbar features

### Props

| Prop | Type | Description |
|------|------|-------------|
| className | string | Optional CSS class |
| ...props | ComponentProps<typeof NodeToolbar> | React Flow NodeToolbar props |

### Installation

```bash
npx ai-elements@latest add toolbar
```
