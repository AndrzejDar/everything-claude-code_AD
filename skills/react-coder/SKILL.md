---
name: react-coder
description: React component development skill for frontend coder agents. Use when implementing UI components, pages, forms, hooks, or any React-based frontend features. Triggers on tasks prefixed with FE: or frontend-related coding assignments. Provides structured workflow for writing production-quality React code with proper patterns, accessibility, and performance considerations.
---

# React Coder Skill

Skill for implementing React frontend features in multi-agent workflow pipeline.

## Pre-requisites

Before coding, always:
1. Read project conventions from `./CLAUDE.md`
2. Read `/.claude/skills/react-best-practices/SKILL.md` for React patterns
3. Analyze existing codebase structure in `frontend_path`

## Development Workflow

### 1. Task Analysis

Parse the task to identify:
- **Component type**: Page, feature component, UI primitive, form, layout
- **Data requirements**: Props, state, API calls, context
- **User interactions**: Events, validations, side effects
- **Dependencies**: Existing components to reuse, libraries needed

### 2. Implementation Strategy

**For new components:**
```
1. Define TypeScript interface for props
2. Create component file with proper naming
3. Implement core logic with hooks
4. Add error boundaries if needed
5. Export from index file
```

**For modifications:**
```
1. Locate existing component
2. Understand current implementation
3. Apply minimal changes
4. Preserve existing patterns
```

### 3. Code Quality Checklist

Before completing task, verify:

- [ ] TypeScript types are explicit (no `any`)
- [ ] Props have default values where appropriate
- [ ] Side effects are in `useEffect` with proper deps
- [ ] Memoization applied where beneficial (`useMemo`, `useCallback`)
- [ ] Error states handled gracefully
- [ ] Loading states implemented
- [ ] Accessibility attributes present (aria-*, role, semantic HTML)
- [ ] No hardcoded strings (use constants or i18n)
- [ ] Component is testable (logic extractable, props injectable)

## Component Patterns

### Functional Component Template

```tsx
interface ComponentNameProps {
  required: string;
  optional?: number;
  onAction: (value: string) => void;
}

export const ComponentName: React.FC<ComponentNameProps> = ({
  required,
  optional = 10,
  onAction,
}) => {
  // hooks first
  const [state, setState] = useState<StateType>(initialState);
  
  // derived state
  const computed = useMemo(() => /* ... */, [deps]);
  
  // callbacks
  const handleEvent = useCallback(() => /* ... */, [deps]);
  
  // effects last
  useEffect(() => { /* ... */ }, [deps]);
  
  // early returns for edge cases
  if (!required) return null;
  
  return (
    <div role="region" aria-label="descriptive label">
      {/* JSX */}
    </div>
  );
};
```

### Custom Hook Template

```tsx
interface UseHookNameOptions {
  option?: string;
}

interface UseHookNameReturn {
  data: DataType | null;
  isLoading: boolean;
  error: Error | null;
  actions: { refetch: () => void };
}

export const useHookName = (
  options: UseHookNameOptions = {}
): UseHookNameReturn => {
  // implementation
};
```

### Form Pattern

```tsx
interface FormData {
  field: string;
}

const schema = z.object({
  field: z.string().min(1, 'Required'),
});

export const FormComponent: React.FC = () => {
  const { register, handleSubmit, formState: { errors } } = useForm<FormData>({
    resolver: zodResolver(schema),
  });
  
  const onSubmit = async (data: FormData) => {
    // handle submission
  };
  
  return (
    <form onSubmit={handleSubmit(onSubmit)} noValidate>
      <label htmlFor="field">Label</label>
      <input id="field" {...register('field')} aria-invalid={!!errors.field} />
      {errors.field && <span role="alert">{errors.field.message}</span>}
      <button type="submit">Submit</button>
    </form>
  );
};
```

## File Organization

Follow project structure. Default pattern:

```
src/
├── components/
│   ├── ui/           # Primitives (Button, Input, Modal)
│   ├── features/     # Feature-specific components
│   └── layout/       # Layout components (Header, Sidebar)
├── hooks/            # Custom hooks
├── pages/            # Route pages
├── services/         # API calls
├── stores/           # State management
├── types/            # Shared TypeScript types
└── utils/            # Helper functions
```

## Common Tasks Reference

| Task | Key Considerations |
|------|-------------------|
| Create form | Validation library, error display, submit handling, a11y |
| Add API call | Loading/error states, caching strategy, error boundaries |
| Create modal | Focus trap, escape key, backdrop click, portal |
| Add routing | Lazy loading, route guards, URL params |
| State management | Local vs global, persistence needs, sync requirements |
| Add animation | Performance (CSS vs JS), reduced motion preference |

## Output Format

When completing a task, provide:

1. **Files created/modified** - List with brief description
2. **Key decisions** - Why specific patterns were chosen
3. **Testing notes** - What should be tested
4. **Dependencies** - Any new packages needed (with install command)

## Error Recovery

If tests fail after your implementation:
1. Read test output carefully
2. Identify root cause (logic, types, missing mock)
3. Fix minimally - don't over-engineer
4. Verify fix doesn't break other tests