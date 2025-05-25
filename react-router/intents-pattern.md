# The Intents Pattern: A Server Action Architecture for React Router V7 / Remix

## Overview

The Intents Pattern is a server-side architecture for organizing and routing multiple related actions through a single Remix API route. Instead of creating separate routes for each action, this pattern uses an "intent" field to determine which specific handler function should process the request.

## Core Concepts

### What is an Intent?

An **intent** is a string identifier that specifies what action the client wants to perform. It acts as a routing mechanism within a single action function, allowing you to group related operations under a unified endpoint.

### Benefits

- **Centralized Action Management**: Group related actions in one place
- **Type Safety**: Leverage TypeScript for intent validation and handler mapping
- **Consistent Error Handling**: Apply uniform error handling across all intents
- **Scalable Architecture**: Easy to add new actions without creating new routes
- **Clear Separation of Concerns**: API routes handle routing, handlers contain business logic

## Architecture Components

### 1. API Route Structure

```typescript
// Basic API route template
import { data, type ActionFunctionArgs } from "react-router"

const intents = ["intent-one", "intent-two", "intent-three"] as const

export async function loader() {
    return data('Not Allowed', { status: 405 })
}

export async function action({ request }: ActionFunctionArgs) {
    const formData = await request.formData()
    const intent = formData.get('intent') as string

    // Intent validation
    if (!intent || !intents.includes(intent)) {
        return data({ success: false, message: 'Invalid form submission' }, { status: 400 })
    }

    try {
        // Handler mapping and execution
        const handlers = {
            'intent-one': handleIntentOne,
            'intent-two': handleIntentTwo,
            'intent-three': handleIntentThree,
        } as const

        const handler = handlers[intent as keyof typeof handlers]
        return handler(request, formData)
    } catch (error) {
        console.error('Action error:', error)
        return data({ error: 'An unexpected error occurred' }, { status: 500 })
    }
}
```

### 2. Handler Function Signature

All handler functions should follow a consistent signature:

```typescript
type IntentHandler = (
    request: Request, 
    formData: FormData
) => Promise<Response | TypedResponse>
```

### 3. Handler Implementation Pattern

```typescript
export async function handleSpecificAction(request: Request, formData: FormData) {
    // 1. Authentication/Authorization (if required)
    const { session } = await isAuthenticated(request)
    if (!session) {
        return redirect("/login")
    }

    // 2. Extract and validate data
    const actionData = {
        field1: formData.get("field1"),
        field2: formData.get("field2")
    }
    
    const validation = actionSchema.safeParse(actionData)
    if (!validation.success) {
        return data({
            success: false,
            message: 'Invalid Fields'
        }, { status: 400 })
    }

    // 3. Business logic
    try {
        // Perform the actual operation
        const result = await performOperation(validation.data)
        
        return data({
            success: true,
            message: "Operation completed successfully",
            data: result
        })
    } catch (error) {
        console.error('Handler error:', error)
        return data({
            success: false,
            message: error instanceof Error ? error.message : 'Operation failed'
        }, { status: 500 })
    }
}
```

## Implementation Guidelines

### 1. Intent Naming Conventions

- Use **kebab-case** for intent names: `sign-in`, `update-password`, `create-user`
- Be **descriptive and specific**: `add-instagram-credentials` vs `add-creds`
- Group related intents by **domain**: auth intents, user intents, etc.

### 2. File Organization

```
app/
├── routes/
│   └── api/
│       ├── auth.ts              # Auth-related intents
│       ├── user.ts              # User management intents
│       └── content.ts           # Content-related intents
├── lib/
│   └── actions/
│       ├── auth.server.ts        # Auth handler implementations
│       ├── user.server.ts        # User handler implementations
│       └── content.server.ts     # Content handler implementations
```

### 3. Type Safety Best Practices

```typescript
// Define intents as const assertions for type safety
const intents = ["sign-in", "sign-out", "update-profile"] as const
type Intent = typeof intents[number]

// Create typed handler map
const handlers: Record<Intent, IntentHandler> = {
    'sign-in': handleSignIn,
    'sign-out': handleSignOut,
    'update-profile': handleUpdateProfile,
} as const

// Type-safe intent checking
function isValidIntent(intent: string): intent is Intent {
    return intents.includes(intent as Intent)
}
```

### 4. Error Handling Strategy

Implement consistent error handling across all levels:

```typescript
// API route level - catches handler errors
try {
    const handler = handlers[intent as keyof typeof handlers]
    return handler(request, formData)
} catch (error) {
    console.error('Action error:', error)
    return data({ error: 'An unexpected error occurred' }, { status: 500 })
}

// Handler level - catches business logic errors
try {
    // Business logic here
    return data({ success: true, message: "Success" })
} catch (error) {
    return data({ 
        success: false, 
        message: error instanceof Error ? error.message : 'Operation failed' 
    }, { status: 500 })
}
```

## Client-Side Integration

### Form Implementation

```tsx
import { Form } from "react-router"

function SignInForm() {
    return (
        <Form method="post" action="/api/auth">
            <input type="hidden" name="intent" value="sign-in" />
            <input name="emailOrUsername" type="text" required />
            <input name="password" type="password" required />
            <button type="submit">Sign In</button>
        </Form>
    )
}
```

### Programmatic Submission

```tsx
function handleSignOut() {
    const formData = new FormData()
    formData.set('intent', 'sign-out')
    
    submit(formData, {
        method: 'post',
        action: '/api/auth'
    })
}
```

## Advanced Patterns

### 1. Conditional Intent Lists

```typescript
// Different intents based on environment or user role
const baseIntents = ["sign-in", "sign-out"] as const
const adminIntents = [...baseIntents, "create-user", "delete-user"] as const

const intents = isProduction ? baseIntents : adminIntents
```

### 2. Middleware Pattern

```typescript
export async function action({ request }: ActionFunctionArgs) {
    // Apply middleware before intent processing
    const rateLimitResult = await applyRateLimit(request)
    if (!rateLimitResult.success) {
        return data({ error: 'Rate limit exceeded' }, { status: 429 })
    }

    // Continue with intent processing...
}
```

### 3. Intent Composition

```typescript
// Compose complex operations from simpler intents
async function handleComplexOperation(request: Request, formData: FormData) {
    // This handler might call other handlers internally
    const step1 = await handleValidateData(request, formData)
    if (!step1.success) return step1
    
    const step2 = await handleProcessData(request, formData)
    if (!step2.success) return step2
    
    return data({ success: true, message: "Complex operation completed" })
}
```

## Testing Strategies

### 1. Unit Testing Handlers

```typescript
describe('handleSignIn', () => {
    it('should authenticate valid credentials', async () => {
        const formData = new FormData()
        formData.set('emailOrUsername', 'user@example.com')
        formData.set('password', 'validpassword')
        
        const result = await handleSignIn(mockRequest, formData)
        expect(result.success).toBe(true)
    })
})
```

### 2. Integration Testing API Routes

```typescript
describe('/api/auth', () => {
    it('should handle sign-in intent', async () => {
        const response = await request(app)
            .post('/api/auth')
            .send('intent=sign-in&emailOrUsername=user@example.com&password=test')
            
        expect(response.status).toBe(200)
        expect(response.body.success).toBe(true)
    })
})
```

## Migration Guide

### From Individual Routes to Intents

1. **Identify Related Actions**: Group actions that belong to the same domain
2. **Create API Route**: Set up the intent-based action handler
3. **Extract Handler Logic**: Move existing route logic to handler functions
4. **Update Client Code**: Modify forms to use intent field
5. **Test Thoroughly**: Ensure all functionality still works

### Gradual Adoption

You can adopt this pattern gradually:
- Start with new features using the intents pattern
- Migrate existing routes when they need updates
- Maintain both patterns during transition

## Common Pitfalls and Solutions

### 1. Intent Validation
**Problem**: Missing or incorrect intent validation
**Solution**: Always validate intents against a predefined list

### 2. Handler Signature Consistency
**Problem**: Inconsistent handler function signatures
**Solution**: Define and enforce a standard handler interface

### 3. Error Boundary Overlap
**Problem**: Errors caught at multiple levels causing confusion
**Solution**: Establish clear error handling responsibilities

### 4. Type Safety Erosion
**Problem**: Losing type safety with string-based intent matching
**Solution**: Use const assertions and proper TypeScript patterns

## Conclusion

The Intents Pattern provides a clean, scalable architecture for organizing server actions in Remix applications. By centralizing related operations and maintaining consistent patterns, it improves code organization, type safety, and developer experience while keeping the flexibility to handle complex business logic.

This pattern works particularly well for:
- Authentication and authorization flows
- User management operations
- Content management systems
- API-like routes and endpoints
- Multi-step form processing

Remember to maintain consistency in your implementation and always prioritize type safety and error handling to get the most benefit from this architectural pattern.
