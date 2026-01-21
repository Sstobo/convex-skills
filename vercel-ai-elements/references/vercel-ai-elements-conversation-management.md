# Conversation Management

## Conversation

The `Conversation` component wraps messages and automatically scrolls to the bottom. Also includes a scroll button that appears when not at the bottom.

### Installation

```sh
npx ai-elements@latest add conversation
```

### Basic Usage

```tsx
import {
  Conversation,
  ConversationContent,
  ConversationEmptyState,
  ConversationScrollButton,
} from '@/components/ai-elements/conversation';
```

```tsx
<Conversation className="relative w-full" style={{ height: '500px' }}>
  <ConversationContent>
    {messages.length === 0 ? (
      <ConversationEmptyState
        icon={<MessageSquare className="size-12" />}
        title="No messages yet"
        description="Start a conversation to see messages here"
      />
    ) : (
      messages.map((message) => (
        <Message from={message.from} key={message.id}>
          <MessageContent>{message.content}</MessageContent>
        </Message>
      ))
    )}
  </ConversationContent>
  <ConversationScrollButton />
</Conversation>
```

### Features

- Automatic scrolling to the bottom when new messages are added
- Smooth scrolling behavior with configurable animation
- Scroll button that appears when not at the bottom
- Responsive design with customizable padding and spacing
- Flexible content layout with consistent message spacing
- Accessible with proper ARIA roles for screen readers
- Customizable styling through className prop
- Support for any number of child message components

### Props

#### `<Conversation />`

| Prop | Type | Description |
|------|------|-------------|
| `contextRef` | `React.Ref<StickToBottomContext>` | Optional ref to access the StickToBottom context object |
| `instance` | `StickToBottomInstance` | Optional instance for controlling the StickToBottom component |
| `children` | `((context: StickToBottomContext) => ReactNode) \| ReactNode` | Render prop or ReactNode for custom rendering with context |
| `[...props]` | `Omit<React.HTMLAttributes<HTMLDivElement>, "children">` | Any other props are spread to the root div |

#### `<ConversationContent />`

| Prop | Type | Description |
|------|------|-------------|
| `children` | `((context: StickToBottomContext) => ReactNode) \| ReactNode` | Render prop or ReactNode for custom rendering with context |
| `[...props]` | `Omit<React.HTMLAttributes<HTMLDivElement>, "children">` | Any other props are spread to the root div |

#### `<ConversationEmptyState />`

| Prop | Type | Description |
|------|------|-------------|
| `title` | `string` | The title text to display. Defaults to "No messages yet" |
| `description` | `string` | The description text to display. Defaults to "Start a conversation to see messages here" |
| `icon` | `React.ReactNode` | Optional icon to display above the text |
| `children` | `React.ReactNode` | Optional additional content to render below the text |
| `[...props]` | `ComponentProps<"div">` | Any other props are spread to the root div |

#### `<ConversationScrollButton />`

| Prop | Type | Description |
|------|------|-------------|
| `[...props]` | `ComponentProps<typeof Button>` | Any other props are spread to the underlying shadcn/ui Button component |

### Integration with AI SDK

```tsx
import {
  Conversation,
  ConversationContent,
  ConversationEmptyState,
  ConversationScrollButton,
} from '@/components/ai-elements/conversation';
import { Message, MessageContent } from '@/components/ai-elements/message';
import {
  Input,
  PromptInputTextarea,
  PromptInputSubmit,
} from '@/components/ai-elements/prompt-input';
import { MessageSquare } from 'lucide-react';
import { useState } from 'react';
import { useChat } from '@ai-sdk/react';
import { Response } from '@/components/ai-elements/response';

const ConversationDemo = () => {
  const [input, setInput] = useState('');
  const { messages, sendMessage, status } = useChat();

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    if (input.trim()) {
      sendMessage({ text: input });
      setInput('');
    }
  };

  return (
    <div className="max-w-4xl mx-auto p-6 relative size-full rounded-lg border h-[600px]">
      <div className="flex flex-col h-full">
        <Conversation>
          <ConversationContent>
            {messages.length === 0 ? (
              <ConversationEmptyState
                icon={<MessageSquare className="size-12" />}
                title="Start a conversation"
                description="Type a message below to begin chatting"
              />
            ) : (
              messages.map((message) => (
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
              ))
            )}
          </ConversationContent>
          <ConversationScrollButton />
        </Conversation>

        <Input
          onSubmit={handleSubmit}
          className="mt-4 w-full max-w-2xl mx-auto relative"
        >
          <PromptInputTextarea
            value={input}
            placeholder="Say something..."
            onChange={(e) => setInput(e.currentTarget.value)}
            className="pr-12"
          />
          <PromptInputSubmit
            status={status === 'streaming' ? 'streaming' : 'ready'}
            disabled={!input.trim()}
            className="absolute bottom-1 right-1"
          />
        </Input>
      </div>
    </div>
  );
};

export default ConversationDemo;
```
