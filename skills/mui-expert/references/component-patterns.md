# MUI Component Patterns

Common patterns for Material UI components.

## Table of Contents

1. [Form Components](#form-components)
2. [Data Display](#data-display)
3. [Navigation](#navigation)
4. [Feedback](#feedback)
5. [Layout Patterns](#layout-patterns)

## Form Components

### Controlled TextField

```tsx
import { useState } from 'react';
import TextField from '@mui/material/TextField';

function ControlledInput() {
  const [value, setValue] = useState('');
  const [error, setError] = useState('');

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const newValue = e.target.value;
    setValue(newValue);
    setError(newValue.length < 3 ? 'Minimum 3 characters' : '');
  };

  return (
    <TextField
      label="Username"
      value={value}
      onChange={handleChange}
      error={!!error}
      helperText={error}
      fullWidth
    />
  );
}
```

### Select with Options

```tsx
import { FormControl, InputLabel, Select, MenuItem, FormHelperText } from '@mui/material';

<FormControl fullWidth error={hasError}>
  <InputLabel id="category-label">Category</InputLabel>
  <Select
    labelId="category-label"
    value={category}
    label="Category"
    onChange={(e) => setCategory(e.target.value)}
  >
    <MenuItem value="">
      <em>None</em>
    </MenuItem>
    <MenuItem value="tech">Technology</MenuItem>
    <MenuItem value="health">Health</MenuItem>
  </Select>
  {hasError && <FormHelperText>Required field</FormHelperText>}
</FormControl>
```

### Form with Validation (react-hook-form)

```tsx
import { useForm, Controller } from 'react-hook-form';
import { TextField, Button, Stack } from '@mui/material';

function Form() {
  const { control, handleSubmit, formState: { errors } } = useForm();

  return (
    <Stack component="form" spacing={2} onSubmit={handleSubmit(onSubmit)}>
      <Controller
        name="email"
        control={control}
        rules={{ required: 'Email required', pattern: { value: /^\S+@\S+$/, message: 'Invalid email' } }}
        render={({ field }) => (
          <TextField
            {...field}
            label="Email"
            error={!!errors.email}
            helperText={errors.email?.message}
          />
        )}
      />
      <Button type="submit" variant="contained">Submit</Button>
    </Stack>
  );
}
```

## Data Display

### Data Table with Sorting

```tsx
import { Table, TableBody, TableCell, TableContainer, TableHead, TableRow, TableSortLabel, Paper } from '@mui/material';

function SortableTable({ rows }) {
  const [orderBy, setOrderBy] = useState('name');
  const [order, setOrder] = useState<'asc' | 'desc'>('asc');

  const handleSort = (property: string) => {
    const isAsc = orderBy === property && order === 'asc';
    setOrder(isAsc ? 'desc' : 'asc');
    setOrderBy(property);
  };

  const sortedRows = [...rows].sort((a, b) => {
    const cmp = a[orderBy] < b[orderBy] ? -1 : 1;
    return order === 'asc' ? cmp : -cmp;
  });

  return (
    <TableContainer component={Paper}>
      <Table>
        <TableHead>
          <TableRow>
            <TableCell>
              <TableSortLabel
                active={orderBy === 'name'}
                direction={orderBy === 'name' ? order : 'asc'}
                onClick={() => handleSort('name')}
              >
                Name
              </TableSortLabel>
            </TableCell>
          </TableRow>
        </TableHead>
        <TableBody>
          {sortedRows.map((row) => (
            <TableRow key={row.id}>
              <TableCell>{row.name}</TableCell>
            </TableRow>
          ))}
        </TableBody>
      </Table>
    </TableContainer>
  );
}
```

### Card with Actions

```tsx
import { Card, CardHeader, CardContent, CardActions, Avatar, IconButton, Typography, Button } from '@mui/material';
import { MoreVert, Favorite, Share } from '@mui/icons-material';

<Card>
  <CardHeader
    avatar={<Avatar>R</Avatar>}
    action={<IconButton aria-label="settings"><MoreVert /></IconButton>}
    title="Card Title"
    subheader="September 14, 2024"
  />
  <CardContent>
    <Typography variant="body2" color="text.secondary">
      Card content goes here.
    </Typography>
  </CardContent>
  <CardActions disableSpacing>
    <IconButton aria-label="add to favorites"><Favorite /></IconButton>
    <IconButton aria-label="share"><Share /></IconButton>
    <Button size="small" sx={{ ml: 'auto' }}>Learn More</Button>
  </CardActions>
</Card>
```

## Navigation

### Responsive App Bar

```tsx
import { AppBar, Toolbar, Typography, IconButton, Drawer, List, ListItem, ListItemButton, ListItemText, Box, useMediaQuery, useTheme } from '@mui/material';
import { Menu as MenuIcon } from '@mui/icons-material';

function ResponsiveAppBar() {
  const [drawerOpen, setDrawerOpen] = useState(false);
  const theme = useTheme();
  const isMobile = useMediaQuery(theme.breakpoints.down('md'));

  const navItems = ['Home', 'About', 'Contact'];

  return (
    <>
      <AppBar position="static">
        <Toolbar>
          {isMobile && (
            <IconButton color="inherit" edge="start" onClick={() => setDrawerOpen(true)}>
              <MenuIcon />
            </IconButton>
          )}
          <Typography variant="h6" sx={{ flexGrow: 1 }}>Logo</Typography>
          {!isMobile && navItems.map((item) => (
            <Button key={item} color="inherit">{item}</Button>
          ))}
        </Toolbar>
      </AppBar>
      <Drawer open={drawerOpen} onClose={() => setDrawerOpen(false)}>
        <List>
          {navItems.map((item) => (
            <ListItem key={item} disablePadding>
              <ListItemButton onClick={() => setDrawerOpen(false)}>
                <ListItemText primary={item} />
              </ListItemButton>
            </ListItem>
          ))}
        </List>
      </Drawer>
    </>
  );
}
```

### Tabs with Panels

```tsx
import { Tabs, Tab, Box } from '@mui/material';

interface TabPanelProps {
  children?: React.ReactNode;
  index: number;
  value: number;
}

function TabPanel({ children, value, index }: TabPanelProps) {
  return (
    <div role="tabpanel" hidden={value !== index} id={`tabpanel-${index}`}>
      {value === index && <Box sx={{ p: 3 }}>{children}</Box>}
    </div>
  );
}

function TabsExample() {
  const [value, setValue] = useState(0);

  return (
    <Box>
      <Tabs value={value} onChange={(_, newValue) => setValue(newValue)} aria-label="tabs">
        <Tab label="Tab One" id="tab-0" aria-controls="tabpanel-0" />
        <Tab label="Tab Two" id="tab-1" aria-controls="tabpanel-1" />
      </Tabs>
      <TabPanel value={value} index={0}>Content One</TabPanel>
      <TabPanel value={value} index={1}>Content Two</TabPanel>
    </Box>
  );
}
```

## Feedback

### Snackbar with Alert

```tsx
import { Snackbar, Alert, AlertProps } from '@mui/material';

type SnackbarState = {
  open: boolean;
  message: string;
  severity: AlertProps['severity'];
};

function useSnackbar() {
  const [state, setState] = useState<SnackbarState>({ open: false, message: '', severity: 'info' });

  const showSnackbar = (message: string, severity: AlertProps['severity'] = 'info') => {
    setState({ open: true, message, severity });
  };

  const closeSnackbar = () => setState((prev) => ({ ...prev, open: false }));

  const SnackbarComponent = (
    <Snackbar open={state.open} autoHideDuration={6000} onClose={closeSnackbar}>
      <Alert onClose={closeSnackbar} severity={state.severity} variant="filled">
        {state.message}
      </Alert>
    </Snackbar>
  );

  return { showSnackbar, SnackbarComponent };
}
```

### Confirmation Dialog

```tsx
import { Dialog, DialogTitle, DialogContent, DialogContentText, DialogActions, Button } from '@mui/material';

function ConfirmDialog({ open, title, message, onConfirm, onCancel }) {
  return (
    <Dialog open={open} onClose={onCancel}>
      <DialogTitle>{title}</DialogTitle>
      <DialogContent>
        <DialogContentText>{message}</DialogContentText>
      </DialogContent>
      <DialogActions>
        <Button onClick={onCancel}>Cancel</Button>
        <Button onClick={onConfirm} variant="contained" color="error">Confirm</Button>
      </DialogActions>
    </Dialog>
  );
}
```

## Layout Patterns

### Centered Container

```tsx
import { Container, Box } from '@mui/material';

<Container maxWidth="sm">
  <Box sx={{ display: 'flex', flexDirection: 'column', alignItems: 'center', minHeight: '100vh', justifyContent: 'center' }}>
    {/* Centered content */}
  </Box>
</Container>
```

### Sticky Footer

```tsx
<Box sx={{ display: 'flex', flexDirection: 'column', minHeight: '100vh' }}>
  <AppBar />
  <Box component="main" sx={{ flex: 1, py: 3 }}>
    {/* Main content */}
  </Box>
  <Box component="footer" sx={{ py: 2, bgcolor: 'grey.200' }}>
    Footer
  </Box>
</Box>
```

### Dashboard Layout

```tsx
const drawerWidth = 240;

<Box sx={{ display: 'flex' }}>
  <AppBar position="fixed" sx={{ zIndex: (theme) => theme.zIndex.drawer + 1 }}>
    <Toolbar>
      <Typography variant="h6">Dashboard</Typography>
    </Toolbar>
  </AppBar>
  <Drawer
    variant="permanent"
    sx={{
      width: drawerWidth,
      '& .MuiDrawer-paper': { width: drawerWidth, boxSizing: 'border-box' },
    }}
  >
    <Toolbar />
    <List>{/* Nav items */}</List>
  </Drawer>
  <Box component="main" sx={{ flexGrow: 1, p: 3 }}>
    <Toolbar />
    {/* Page content */}
  </Box>
</Box>
```
