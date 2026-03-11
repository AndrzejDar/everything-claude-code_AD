# MUI Theming Reference

Advanced theming patterns for Material UI.

## Table of Contents

1. [Custom Theme Structure](#custom-theme-structure)
2. [Component Style Overrides](#component-style-overrides)
3. [Dark Mode](#dark-mode)
4. [Custom Breakpoints](#custom-breakpoints)
5. [Typography Variants](#typography-variants)

## Custom Theme Structure

```tsx
import { createTheme, responsiveFontSizes } from '@mui/material/styles';

let theme = createTheme({
  palette: {
    mode: 'light',
    primary: {
      main: '#1976d2',
      light: '#42a5f5',
      dark: '#1565c0',
      contrastText: '#fff',
    },
    secondary: {
      main: '#9c27b0',
    },
    background: {
      default: '#f5f5f5',
      paper: '#ffffff',
    },
  },
  typography: {
    fontFamily: '"Inter", "Roboto", "Helvetica", sans-serif',
    h1: { fontWeight: 700 },
    button: { textTransform: 'none' },
  },
  shape: {
    borderRadius: 8,
  },
  spacing: 8, // base unit, theme.spacing(2) = 16px
});

theme = responsiveFontSizes(theme);
```

## Component Style Overrides

Global overrides in theme:

```tsx
const theme = createTheme({
  components: {
    MuiButton: {
      defaultProps: {
        disableElevation: true,
      },
      styleOverrides: {
        root: {
          borderRadius: 8,
          padding: '8px 16px',
        },
        containedPrimary: ({ theme }) => ({
          '&:hover': {
            backgroundColor: theme.palette.primary.dark,
          },
        }),
      },
      variants: [
        {
          props: { variant: 'dashed' },
          style: {
            border: '2px dashed currentColor',
          },
        },
      ],
    },
    MuiTextField: {
      defaultProps: {
        variant: 'outlined',
        size: 'small',
      },
    },
    MuiCard: {
      styleOverrides: {
        root: {
          boxShadow: '0 2px 8px rgba(0,0,0,0.08)',
        },
      },
    },
  },
});
```

### TypeScript: Extend Variants

```tsx
declare module '@mui/material/Button' {
  interface ButtonPropsVariantOverrides {
    dashed: true;
  }
}
```

## Dark Mode

### Toggle Implementation

```tsx
import { useMemo, useState, createContext, useContext } from 'react';
import { ThemeProvider, createTheme } from '@mui/material/styles';
import CssBaseline from '@mui/material/CssBaseline';

const ColorModeContext = createContext({ toggleColorMode: () => {} });

export function ThemeWrapper({ children }) {
  const [mode, setMode] = useState<'light' | 'dark'>('light');
  
  const colorMode = useMemo(
    () => ({
      toggleColorMode: () => setMode((prev) => (prev === 'light' ? 'dark' : 'light')),
    }),
    []
  );

  const theme = useMemo(
    () => createTheme({
      palette: { mode },
    }),
    [mode]
  );

  return (
    <ColorModeContext.Provider value={colorMode}>
      <ThemeProvider theme={theme}>
        <CssBaseline />
        {children}
      </ThemeProvider>
    </ColorModeContext.Provider>
  );
}

export const useColorMode = () => useContext(ColorModeContext);
```

### System Preference Detection

```tsx
const prefersDarkMode = useMediaQuery('(prefers-color-scheme: dark)');

const theme = useMemo(
  () => createTheme({ palette: { mode: prefersDarkMode ? 'dark' : 'light' } }),
  [prefersDarkMode]
);
```

## Custom Breakpoints

```tsx
const theme = createTheme({
  breakpoints: {
    values: {
      xs: 0,
      sm: 600,
      md: 900,
      lg: 1200,
      xl: 1536,
      // Custom breakpoints
      mobile: 0,
      tablet: 640,
      laptop: 1024,
      desktop: 1280,
    },
  },
});

// TypeScript declaration
declare module '@mui/material/styles' {
  interface BreakpointOverrides {
    xs: true;
    sm: true;
    md: true;
    lg: true;
    xl: true;
    mobile: true;
    tablet: true;
    laptop: true;
    desktop: true;
  }
}
```

## Typography Variants

### Custom Variants

```tsx
const theme = createTheme({
  typography: {
    poster: {
      fontSize: '4rem',
      fontWeight: 700,
      lineHeight: 1.1,
    },
    label: {
      fontSize: '0.75rem',
      fontWeight: 500,
      letterSpacing: '0.1em',
      textTransform: 'uppercase',
    },
  },
});

// TypeScript declaration
declare module '@mui/material/styles' {
  interface TypographyVariants {
    poster: React.CSSProperties;
    label: React.CSSProperties;
  }
  interface TypographyVariantsOptions {
    poster?: React.CSSProperties;
    label?: React.CSSProperties;
  }
}

declare module '@mui/material/Typography' {
  interface TypographyPropsVariantOverrides {
    poster: true;
    label: true;
  }
}
```

## Custom Colors

```tsx
const theme = createTheme({
  palette: {
    brand: {
      main: '#5048e5',
      light: '#828df8',
      dark: '#3832a0',
      contrastText: '#fff',
    },
  },
});

// TypeScript
declare module '@mui/material/styles' {
  interface Palette {
    brand: Palette['primary'];
  }
  interface PaletteOptions {
    brand?: PaletteOptions['primary'];
  }
}

declare module '@mui/material/Button' {
  interface ButtonPropsColorOverrides {
    brand: true;
  }
}

// Usage
<Button color="brand">Brand Button</Button>
```
