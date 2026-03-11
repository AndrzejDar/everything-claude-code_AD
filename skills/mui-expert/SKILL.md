---
name: mui-expert
description: "Expert guide for building React UIs with Material UI (MUI). Triggers when user asks to: (1) create or build UI components with MUI/Material UI, (2) customize MUI themes, (3) style MUI components, (4) debug MUI issues, (5) implement responsive layouts with MUI Grid, (6) use MUI data components like DataGrid or Tables, (7) follow MUI best practices. Keywords: MUI, Material UI, Material-UI, @mui/material, React components, theming, sx prop, styled API, Grid, DataGrid, ThemeProvider."
---

# MUI Expert

Expert guide for building high-quality React user interfaces with Material UI (MUI).

## MCP Integration

MUI provides an official Model Context Protocol (MCP) server for accessing up-to-date documentation and code examples. When available, use the MUI MCP tools:

1. Call `useMuiDocs` to fetch docs for the relevant MUI package
2. Call `fetchDocs` to retrieve additional documentation using URLs from returned content
3. Repeat until all relevant docs are gathered
4. Use fetched content to provide accurate, up-to-date answers

### MCP Setup (for reference)

```bash
# Claude Code
claude mcp add mui-mcp -- npx -y @mui/mcp@latest

# VS Code / Cursor / Windsurf (settings.json)
"mcpServers": {
  "mui-mcp": {
    "type": "stdio",
    "command": "npx",
    "args": ["-y", "@mui/mcp@latest"]
  }
}
```

## Installation

```bash
npm install @mui/material @emotion/react @emotion/styled
# or
yarn add @mui/material @emotion/react @emotion/styled
```

For icons: `npm install @mui/icons-material`

## Core Principles

### 1. Theme-First Development

Always wrap application with `ThemeProvider` and define a custom theme:

```tsx
import { createTheme, ThemeProvider } from '@mui/material/styles';
import CssBaseline from '@mui/material/CssBaseline';

const theme = createTheme({
  palette: {
    primary: { main: '#1976d2' },
    secondary: { main: '#dc004e' },
  },
  typography: {
    fontFamily: '"Inter", "Roboto", sans-serif',
  },
});

function App() {
  return (
    <ThemeProvider theme={theme}>
      <CssBaseline />
      {/* App content */}
    </ThemeProvider>
  );
}
```

### 2. Styling Hierarchy

Choose the right approach based on use case (narrowest to broadest):

| Approach | When to Use |
|----------|-------------|
| `sx` prop | One-off, inline customizations |
| `styled()` API | Reusable styled components |
| Theme `styleOverrides` | Global component customizations |
| CSS/Emotion | Complex conditional styles |

### 3. sx Prop Best Practices

```tsx
// ✅ Good: Access theme values
<Box sx={{ p: 2, bgcolor: 'primary.main', color: 'primary.contrastText' }} />

// ✅ Good: Responsive values
<Box sx={{ width: { xs: '100%', sm: '50%', md: '33%' } }} />

// ✅ Good: Pseudo-selectors
<Button sx={{ '&:hover': { transform: 'scale(1.05)' } }} />

// ❌ Avoid: Large style objects (use styled() instead)
// ❌ Avoid: Duplicating sx across multiple components
```

### 4. styled() API for Reusability

```tsx
import { styled } from '@mui/material/styles';
import Button from '@mui/material/Button';

const PrimaryButton = styled(Button)(({ theme }) => ({
  backgroundColor: theme.palette.primary.main,
  padding: theme.spacing(1, 3),
  '&:hover': {
    backgroundColor: theme.palette.primary.dark,
  },
}));

// With custom props
interface CustomProps { success?: boolean; }

const StatusButton = styled(Button, {
  shouldForwardProp: (prop) => prop !== 'success',
})<CustomProps>(({ theme, success }) => ({
  ...(success && {
    backgroundColor: theme.palette.success.main,
  }),
}));
```

## Component Patterns

### Responsive Grid Layout

```tsx
import Grid from '@mui/material/Grid';

<Grid container spacing={2}>
  <Grid size={{ xs: 12, sm: 6, md: 4 }}>
    {/* Content */}
  </Grid>
</Grid>
```

### Component Composition

Use the `component` prop to change root elements:

```tsx
import { Link } from 'react-router-dom';

<Button component={Link} to="/dashboard">
  Dashboard
</Button>
```

### Forward Refs for Custom Components

```tsx
const CustomInput = React.forwardRef<HTMLInputElement, Props>((props, ref) => (
  <input {...props} ref={ref} />
));

<TextField InputProps={{ inputComponent: CustomInput }} />
```

## Performance Optimization

### 1. Code Splitting

```tsx
const Dialog = React.lazy(() => import('@mui/material/Dialog'));

<Suspense fallback={<CircularProgress />}>
  <Dialog open={open}>{/* content */}</Dialog>
</Suspense>
```

### 2. Memoization

```tsx
const MemoizedList = React.memo(({ items }) => (
  <List>
    {items.map(item => <ListItem key={item.id}>{item.name}</ListItem>)}
  </List>
));
```

### 3. Static GlobalStyles

```tsx
// ✅ Hoist to prevent re-renders
const globalStyles = <GlobalStyles styles={{ body: { margin: 0 } }} />;

function App() {
  return <>{globalStyles}<Main /></>;
}
```

## Anti-Patterns to Avoid

| Anti-Pattern | Solution |
|--------------|----------|
| Inline styles | Use `sx` prop or `styled()` |
| Direct DOM manipulation | Use React state/refs |
| Deep component nesting | Refactor into smaller components |
| Mutating theme directly | Use `createTheme()` and `ThemeProvider` |
| Ignoring accessibility | Use proper ARIA attributes |
| Over-using `any` in TypeScript | Define accurate types |

## Accessibility Checklist

- Use semantic HTML elements via `component` prop
- Provide `aria-label` for icon-only buttons
- Ensure sufficient color contrast (use theme tokens)
- Support keyboard navigation
- Test with screen readers

## TypeScript Integration

```tsx
import { TypographyProps } from '@mui/material/Typography';

// Extend component props
interface CustomTypographyProps extends TypographyProps<'span'> {
  highlight?: boolean;
}

const HighlightText: React.FC<CustomTypographyProps> = ({ highlight, ...props }) => (
  <Typography
    component="span"
    sx={{ bgcolor: highlight ? 'warning.light' : 'transparent' }}
    {...props}
  />
);
```

## SSR Considerations

For Next.js or other SSR frameworks:

1. Configure Emotion cache for SSR
2. Use `CssBaseline` for consistent cross-browser styles
3. Handle hydration mismatches with `useEffect` for client-only features

## Debugging Tips

1. Use React DevTools to inspect component props
2. Use browser DevTools to identify MUI class names (format: `Mui[Component]-[slot]`)
3. Check console for MUI-specific warnings
4. Use MCP inspector for connection issues: `npx @modelcontextprotocol/inspector`

## Additional References

For more detailed patterns, see:

- **[references/theming.md](references/theming.md)**: Advanced theme customization, dark mode, custom breakpoints, typography variants, custom colors with TypeScript
- **[references/component-patterns.md](references/component-patterns.md)**: Form components, data tables, navigation patterns, feedback components, layout patterns

## Quick Reference

```tsx
// Theme spacing: theme.spacing(1) = 8px
// Breakpoints: xs (0), sm (600), md (900), lg (1200), xl (1536)
// Colors: primary, secondary, error, warning, info, success
// Variants: text, contained, outlined (Button), standard, filled, outlined (TextField)
```
