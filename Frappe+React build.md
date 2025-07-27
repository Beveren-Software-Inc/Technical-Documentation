
# React Integration with Frappe Apps – Automatic Build Setup

This guide walks you through setting up a **React app** inside a **Frappe app** (e.g., `docproc`) using **Vite**, in a way that integrates seamlessly with Frappe’s `bench build` process, inspired by the official [Frappe LMS](https://github.com/frappe/lms) setup.

----------

## ✅ Problems This Solves

-   React changes not reflecting after `bench build`
-   Manual `npm run build` and file copying
    
-   Static file caching issues (e.g., `index.js` cached forever)
    

----------

## 🎯 The Solution

Lets follow Frappe’s modern pattern that ensures:

1.  `bench build` auto-triggers your React build
    
2.  Hashed asset filenames for cache busting
    
3.  Auto-copying of `index.html` for web route rendering
    
4.  Clean asset linking via Frappe's public and www folders
    

----------

## 🔧 Setup Instructions

### 1. Configure `vite.config.js`

**File:** `apps/docproc/docproc-ui/vite.config.js`
```
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import path from "path";
import proxyOptions from "./proxyOptions.js";

export default defineConfig(({ mode }) => {
  const isProd = mode === "production";

  return {
    plugins: [react()],
    server: {
      port: 3000,
      proxy: proxyOptions,
    },
    resolve: {
      alias: {
        "@": path.resolve(__dirname, "src"),
      },
    },
    build: {
      outDir: "../docproc/public/fixdoc",
      emptyOutDir: true,
      target: "es2015",
      rollupOptions: {
        output: {
          entryFileNames: 'assets/index-[hash].js',
          chunkFileNames: 'assets/[name]-[hash].js',
          assetFileNames: 'assets/index-[hash].[ext]',
        },
      },
    },
    base: isProd ? "/assets/docproc/fixdoc/" : "/",
  };
});

```
✅ **Highlights:**

-   Hashed filenames (e.g. `index-abcd1234.js`)
    
-   Output to `public/fixdoc/` for Frappe asset management
    
-   Uses `/assets/docproc/fixdoc/` base in production
    

----------

### 2. Create Root `package.json`

**File:** `apps/docproc/package.json`
```
{
  "name": "docproc",
  "version": "1.0.0",
  "description": "Document Processing System",
  "scripts": {
    "dev": "cd docproc-ui && npm run dev",
    "build": "cd docproc-ui && npm run build && cd .. && npm run copy-html-entry",
    "copy-html-entry": "cp docproc/public/fixdoc/index.html docproc/www/fixdoc.html"
  },
  "private": true,
  "author": "Simon Wanyama",
  "license": "MIT",
  "dependencies": {
    "@radix-ui/react-select": "^2.2.5"
  }
}
```

✅ **Purpose:**

-   Central script hub for dev/build commands
    
-   Automates HTML copy to `www/fixdoc.html`
    
-   Matches the LMS-style integration
    

----------

## ⚙️ How It Works

### Build Flow

nginx

`bench build ➜ runs "npm run build" ➜ React build ➜ HTML copied ➜ Frappe serves assets` 

### Output Structure

```apps/docproc/
├── package.json
├── docproc-ui/
│   ├── vite.config.js
│   └── src/
├── docproc/
│   ├── public/fixdoc/
│   │   ├── index.html
│   │   └── assets/
│   │       ├── index-[hash].js
│   │       └── index-[hash].css
│   └── www/
│       └── fixdoc.html` 
```

## 🧪 Usage

### Development

`# Start React dev server "npm run dev"` 

### Production Build

`# Build React + copy HTML  cd apps/docproc && npm run build # Or just use: bench build` 

----------

## 🚀 Benefits

-   ✅ **Automatic Build:** No manual `npm run build`
    
-   ✅ **Cache Busting:** Thanks to hashed filenames
    
-   ✅ **Production-Ready:** Uses Frappe's asset pipeline
    
-   ✅ **Clean Dev Workflow:** Vite’s fast dev server for frontend
    
-   ✅ **Standardized:** Follows official Frappe app patterns
