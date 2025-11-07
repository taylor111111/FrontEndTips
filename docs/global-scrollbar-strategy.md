# 🧭 Global Scrollbar Strategy

This document defines the **global scrollbar behavior** and styling policy for the project.

---

## 🎯 Goals

- **Default:** Hide all scrollbars globally while preserving scroll functionality for a cleaner UI.  
- **Opt‑in display:** Only show scrollbars in specific areas where scroll feedback is necessary.  
- **Cross‑browser compatibility:** Support Chrome, Safari, Firefox, and Edge.

---

## 🧩 1. Global Hidden Scrollbars (Default Policy)

Add this snippet to `index.ejs` or a global CSS file:

```html
<style>
  /* Hide all scrollbars globally but preserve scroll behavior */
  * {
    scrollbar-width: none;       /* Firefox */
    -ms-overflow-style: none;    /* IE/old Edge */
  }
  *::-webkit-scrollbar {         /* Chrome / Safari / new Edge */
    width: 0;
    height: 0;
    background: transparent;
  }

  html, body {
    height: 100%;
    overscroll-behavior: contain; /* Prevent scroll chaining */
  }
</style>
```

✅ **Effect:**  
All elements with `overflow: auto/scroll` will remain scrollable, but scrollbars are invisible.

---

## 🧩 2. Show Scrollbars in Specific Areas

### ✅ Using a Utility Class (Recommended)

```css
/* Container with visible scrollbars */
.scrollbar-visible {
  overflow: auto !important;
  -webkit-overflow-scrolling: touch; /* Smooth iOS scrolling */
}

/* Firefox scrollbar */
.scrollbar-visible {
  scrollbar-width: thin;
  scrollbar-color: #c8ccd4 transparent;
}

/* Chrome / Safari / Edge scrollbar */
.scrollbar-visible::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}
.scrollbar-visible::-webkit-scrollbar-track {
  background: transparent;
}
.scrollbar-visible::-webkit-scrollbar-thumb {
  background: #c8ccd4;
  border-radius: 8px;
}
.scrollbar-visible::-webkit-scrollbar-thumb:hover {
  background: #aeb4bf;
}
```

**Usage Example:**

```tsx
<Box
  className="scrollbar-visible"
  height="100vh"
  bg="white"
  overflowY="auto"
  overflowX="hidden"
>
  {children}
</Box>
```

---

## 🧩 3. Chakra UI Inline `sx` Version

If you prefer Chakra inline styling without a CSS class:

```tsx
<Box
  overflow="auto"
  sx={{
    WebkitOverflowScrolling: 'touch',
    scrollbarWidth: 'thin',
    scrollbarColor: '#c8ccd4 transparent',
    '::-webkit-scrollbar': { width: '8px', height: '8px' },
    '::-webkit-scrollbar-thumb': {
      background: '#c8ccd4',
      borderRadius: '8px',
    },
    '::-webkit-scrollbar-thumb:hover': { background: '#aeb4bf' },
  }}
>
  ...
</Box>
```

---

## 🧩 4. Common Issues and Recommendations

| Scenario | Recommendation |
|-----------|----------------|
| Double scrollbars appear | Allow only one scroll container (usually content area) |
| iOS scroll feels laggy | Add `-webkit-overflow-scrolling: touch;` |
| Mobile 100vh instability | Use `minHeight="100dvh"` instead |
| Disable scroll completely | `overflow: hidden;` |
| Only vertical scroll | `overflowY: auto; overflowX: hidden;` |

---

## 🧱 Recommended Default Layout

```tsx
<Box
  className="scrollbar-visible"
  height="100vh"
  width="98%"
  bg="white"
  m={3}
  p={3}
  borderRadius="12px"
  boxShadow="sm"
  overflowY="auto"
  overflowX="hidden"
>
  <Tabs.Root>
    <Box p={4} flex={1} overflow="visible">
      {/* Page content */}
    </Box>
  </Tabs.Root>
</Box>
```
