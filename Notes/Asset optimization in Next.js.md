---
created: 2026-04-02
tags: [concept, next-js]
related: [ [[Performance optimization MOC]], [[Performance requirements]], [[CDN usage in frontend]], [[Frontend system design MOC]] ]
status: active
---

# Asset optimization in Next.js

## Overview

High-quality images, custom fonts, and third-party scripts are essential for a good user experience, but they are also the most common causes of high **LCP (Largest Contentful Paint)** and **CLS (Cumulative Layout Shift)** scores.

Next.js provides several built-in components and strategies to handle asset optimization automatically and efficiently.

---

## 1. Image Optimization (`next/image`)

The Next.js `<Image>` component is a significant improvement over the standard `<img>` tag.

### Benefits
- **Size Optimization**: Automatically serves correctly sized images for each device, using modern formats like **WebP** and **AVIF**.
- **Visual Stability**: Prevents layout shift (CLS) by requiring dimensions (`width` and `height`).
- **Faster Page Loads**: Uses native browser lazy loading by default. Images only load when they enter the viewport.
- **Asset Flexibility**: On-demand image resizing, even for images stored on remote servers.

### Basic Usage

```typescript
import Image from 'next/image'

export default function Profile() {
  return (
    <Image
      src="/me.png"
      alt="User Profile"
      width={150} // Required for sizing
      height={150} // Required for sizing
    />
  )
}
```

### Important Attributes

| Attribute | What it does | When to use? |
| :--- | :--- | :--- |
| **`priority`** | High priority loading. Disables lazy loading. | For hero images or anything "above the fold" to improve LCP. |
| **`placeholder="blur"`** | Shows a blurred version of the image during loading. | For static images or when using `blurDataURL` for dynamic ones. |
| **`fill`** | Parent container must have `position: relative`. | When you don't know the exact dimensions (e.g., responsive grid). |
| **`sizes`** | Tells the browser which image size to download at specific breakpoints. | Always use with `fill` for responsiveness. |

```typescript
// Responsive Hero Image
<div className="relative h-64 w-full">
  <Image
    src="/hero.jpg"
    alt="Hero Title"
    fill
    priority
    className="object-cover"
    sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
  />
</div>
```

---

## 2. Font Optimization (`next/font`)

Fonts are often large blocking resources. Custom fonts can also cause **FOUT** (Flash of Unstyled Text) or **FOIT** (Flash of Invisible Text), both of which are bad for UX.

### Benefits
- **Self-Hosting**: Next.js automatically downloads and hosts the fonts locally during build time. No requests to Google for Google Fonts.
- **Visual Stability**: Built-in `size-adjust` to reduce the layout shift when switching from fallback to custom font.
- **No Latency**: Zero external network requests to third-party font providers.

### Usage (Google Fonts)

```typescript
// app/layout.tsx
import { Inter } from 'next/font/google'

const inter = Inter({
  subsets: ['latin'],
  display: 'swap', // Fallback font shown immediately
})

export default function RootLayout({ children }) {
  return <body className={inter.className}>{children}</body>
}
```

---

## 3. Script Optimization (`next/script`)

Third-party scripts (analytics, ads, chat widgets) can drastically slow down your application and block the main thread.

### Loading Strategies (`strategy` prop)

| Strategy | When does it load? | Best For... |
| :--- | :--- | :--- |
| **`beforeInteractive`** | Before Next.js and hydration starts. | Critical security or bot-protection scripts. (Rare) |
| **`afterInteractive`** | (Default) After the page is hydrated. | Analytics, tag managers (Google Tag Manager). |
| **`lazyOnload`** | During browser idle time. | Non-critical widgets (Chat support, social embeds). |
| **`worker`** | (Experimental) Runs script in a **Web Worker**. | Offloading heavy scripts from the main thread entirely. |

### Usage

```typescript
import Script from 'next/script'

export default function Page() {
  return (
    <>
      <Script
        src="https://example.com/analytics.js"
        strategy="afterInteractive"
      />
    </main>
  )
}
```

---

## 4. Video Optimization

- **Host on Platforms**: Use YouTube, Vimeo, or Mux instead of hosting large files on your own server.
- **Use `poster` Attribute**: Provide an optimized image to show while the video loads.
- **Lazy Load Videos**: Don't load high-res videos until they are near the viewport.

---

## Related Notes
- [[Performance optimization MOC]]
- [[Performance requirements]]
- [[CDN usage in frontend]]
- [[Frontend system design MOC]]
- [[Code and bundle optimization]]
