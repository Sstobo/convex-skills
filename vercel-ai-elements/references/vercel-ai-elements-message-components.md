# Message Components

## Message

The `Message` component displays a chat interface message from either a user or an AI. It includes an avatar, a name, and a message content.

### Installation

```sh
npx ai-elements@latest add message
```

### Basic Usage

```tsx
import { Message, MessageContent } from '@/components/ai-elements/message';
```

```tsx
// Default contained variant
<Message from="user">
  <MessageContent>Hi there!</MessageContent>
</Message>

// Flat variant for a minimalist look
<Message from="assistant">
  <MessageContent variant="flat">Hello! How can I help you today?</MessageContent>
</Message>
```

### Features

- Displays messages from both the user and AI assistant with distinct styling
- Two visual variants: **contained** (default) and **flat** for different design preferences
- Includes avatar images for message senders with fallback initials
- Automatically aligns user and assistant messages on opposite sides
- Uses different background colors for user and assistant messages
- Accepts any React node as message content

### Variants

#### Contained (default)

The **contained** variant provides distinct visual separation with colored backgrounds:
- User messages appear with primary background color and are right-aligned
- Assistant messages have secondary background color and are left-aligned
- Both message types have padding and rounded corners

#### Flat

The **flat** variant offers a minimalist design that matches modern AI interfaces like ChatGPT and Gemini:
- User messages use softer secondary colors with subtle borders
- Assistant messages display full-width without background or padding
- Creates a cleaner, more streamlined conversation appearance

### Props

#### `<Message />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `from` | `UIMessage["role"]` | The role of the message sender ("user", "assistant", or "system") | No |
| `[...props]` | `React.HTMLAttributes<HTMLDivElement>` | Any other props are spread to the root div | Yes |

#### `<MessageContent />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `variant` | `"contained" \| "flat"` | Visual style variant. "contained" (default) shows colored backgrounds, "flat" provides a minimalist design | Yes |
| `[...props]` | `React.HTMLAttributes<HTMLDivElement>` | Any other props are spread to the content div | Yes |

#### `<MessageAvatar />`

| Prop | Type | Description | Optional |
|------|------|-------------|----------|
| `src` | `string` | The URL of the avatar image | No |
| `name` | `string` | The name to use for the avatar fallback (first 2 letters shown if image is missing) | Yes |
| `[...props]` | `React.ComponentProps<typeof Avatar>` | Any other props are spread to the underlying Avatar component | Yes |

### Example: Render Markdown

Use the Response component to render markdown content:

```tsx
<Message from="assistant">
  <MessageContent>
    <Response>{markdownContent}</Response>
  </MessageContent>
</Message>
```

### Integration with AI SDK

```tsx
import { Message, MessageContent } from '@/components/ai-elements/message';
import { useChat } from '@ai-sdk/react';
import { Response } from '@/components/ai-elements/response';

const MessageDemo = () => {
  const { messages } = useChat();

  return (
    <div className="max-w-4xl mx-auto p-6">
      {messages.map((message) => (
        <Message from={message.role} key={message.id}>
          <MessageContent>
            {message.parts.map((part, i) => {
              switch (part.type) {
                case 'text':
                  return (
                    <Response key={`${message.id}-${i}`}>
                      {part.text}
                    </Response>
                  );
                default:
                  return null;
              }
            })}
          </MessageContent>
        </Message>
      ))}
    </div>
  );
};

export default MessageDemo;
```
