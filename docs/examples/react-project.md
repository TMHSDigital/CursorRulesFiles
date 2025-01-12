# React Project Example

This example shows how to use CursorRules in a React project.

## Project Setup

1. Create project directory structure:
```
/my-react-project
  /.cursorrules           # Directory for rule files
    project.cursorrules   # Core project rules
    frontend.cursorrules  # React-specific rules
  /src
    /components
    /pages
    /hooks
    /utils
  /tests
  package.json
  tsconfig.json
```

2. Copy relevant rules:
```bash
# Copy core rules
cp rules/core/project.cursorrules .cursorrules/
cp rules/core/code_quality.cursorrules .cursorrules/

# Copy frontend rules
cp rules/frontend/frontend.cursorrules .cursorrules/

# Copy testing rules
cp rules/testing/testing.cursorrules .cursorrules/
```

## Rule Customization

### Project Rules
```
// project.cursorrules
// Customize for React project

// Technology Stack
- Primary language: TypeScript
- Frontend: React 18 with Next.js 13
- State: React Query + Context
- UI: Tailwind CSS + Headless UI

// Project Structure
/src
  /components          # Reusable components
    /ui               # UI components
    /forms           # Form components
    /layout          # Layout components
  /pages             # Next.js pages
  /hooks             # Custom hooks
  /utils             # Utilities
  /types             # TypeScript types
  /styles            # Global styles
```

### Frontend Rules
```
// frontend.cursorrules
// React-specific patterns

// Component Structure
export const ExampleComponent: React.FC<Props> = ({
  // Props with defaults
  title = 'Default Title',
  children
}) => {
  // State hooks
  const [isOpen, setIsOpen] = useState(false)
  
  // Query hooks
  const { data, isLoading } = useQuery(...)
  
  // Effect hooks
  useEffect(() => {
    // Component logic
  }, [dependencies])
  
  // Render
  return (
    <div>
      <h1>{title}</h1>
      {children}
    </div>
  )
}
```

## Implementation Examples

### Component Example
```tsx
// src/components/ui/Button.tsx
import { forwardRef } from 'react'
import { cva, type VariantProps } from 'class-variance-authority'

const buttonVariants = cva(
  'inline-flex items-center justify-center rounded-md text-sm font-medium',
  {
    variants: {
      variant: {
        default: 'bg-primary text-white hover:bg-primary/90',
        secondary: 'bg-secondary text-white hover:bg-secondary/90',
        outline: 'border border-input bg-transparent hover:bg-accent',
      },
      size: {
        default: 'h-10 px-4 py-2',
        sm: 'h-9 px-3',
        lg: 'h-11 px-8',
      },
    },
    defaultVariants: {
      variant: 'default',
      size: 'default',
    },
  }
)

interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  asChild?: boolean
}

const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, asChild = false, ...props }, ref) => {
    return (
      <button
        className={buttonVariants({ variant, size, className })}
        ref={ref}
        {...props}
      />
    )
  }
)
Button.displayName = 'Button'

export { Button, buttonVariants }
```

### Hook Example
```tsx
// src/hooks/useDebounce.ts
import { useState, useEffect } from 'react'

export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value)

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value)
    }, delay)

    return () => {
      clearTimeout(timer)
    }
  }, [value, delay])

  return debouncedValue
}
```

## Testing Example
```tsx
// src/components/ui/Button.test.tsx
import { render, screen, fireEvent } from '@testing-library/react'
import { Button } from './Button'

describe('Button', () => {
  it('should_render_button_with_text', () => {
    // Arrange
    render(<Button>Click me</Button>)
    
    // Act
    const button = screen.getByText('Click me')
    
    // Assert
    expect(button).toBeInTheDocument()
  })

  it('should_call_onClick_when_clicked', () => {
    // Arrange
    const handleClick = jest.fn()
    render(<Button onClick={handleClick}>Click me</Button>)
    
    // Act
    fireEvent.click(screen.getByText('Click me'))
    
    // Assert
    expect(handleClick).toHaveBeenCalledTimes(1)
  })
})
```

## Best Practices

1. Follow component organization:
   - Group related components
   - Keep components focused
   - Use proper TypeScript types

2. Implement proper testing:
   - Write unit tests for components
   - Test user interactions
   - Test accessibility

3. Follow performance guidelines:
   - Use proper memoization
   - Implement code splitting
   - Optimize bundle size

4. Maintain consistency:
   - Follow naming conventions
   - Use consistent styling
   - Document components properly 