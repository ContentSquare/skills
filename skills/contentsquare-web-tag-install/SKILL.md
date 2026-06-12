---
name: contentsquare-web-tag-install
description: Install and verify the Contentsquare tracking tag in any web project. Use when asked to "add Contentsquare", "install CS tag", "set up Contentsquare analytics", or configure Contentsquare tracking for a web application.
metadata:
  author: Contentsquare
  version: "1.0.0"
---

# Install Contentsquare Tag

You are helping a developer install and verify the Contentsquare tracking tag in their web project. Follow this document step by step. Do not skip sections.

---

<rules>

## Rules (read first, apply throughout)

- The package is exactly `@contentsquare/tag-sdk`. The function is exactly `injectContentsquareScript`. The only required option is `clientId`. Do not substitute, rename, or invent alternatives.
- `injectContentsquareScript` **must run in a browser context only** — never during SSR. If you're placing it in a file that runs on the server (Next.js Server Component, Nuxt server route, SvelteKit `+page.server.ts`, etc.), it is wrong.
- The hashed project ID (a.k.a. tag ID) is a **lowercase hex string, ~13 characters** (e.g. `81c677ba742d8`). If what you have looks like a plain decimal number, an email, a URL, or anything else — stop and ask for the **hashed** project ID. **Never hash, transform, generate, or guess the ID yourself.**
- Base framework detection only on actual command output. If detection is ambiguous or `package.json` is missing, **ask the developer**.
- Never start, restart, or kill the dev server. Always ask the developer to do it.
- Never modify CSP unless a CSP violation is detected, or you find an existing CSP missing Contentsquare domains. Do not preventively add CSP headers to a project that has none.
- Run `npm install` (or equivalent) only after confirming the package manager and that you are in the project root.
- After **2 failed verification attempts**, stop modifying code and ask the developer for guidance.
- If the project has no `tsconfig.json`, use `.js`/`.jsx` extensions instead of `.ts`/`.tsx` in all file paths and templates below.

**Completion criteria — the task is complete only when ALL of these are true:**
1. A network request to `t.contentsquare.net/uxa/<HASHED_PROJECT_ID>.js` returns HTTP 200.
2. At least one *additional* network request to `t.contentsquare.net` (beyond the script itself) is sent after the script loads.
3. No CSP violations referencing `contentsquare.net` or `contentsquare.com` appear in the browser console.

Do not declare success based on "the code looks right." Only the three signals above count.

</rules>

---

## Before You Start

### Tag ID

Look for the tag ID in the user's original instruction (e.g. "Install Contentsquare (tag ID: 81c677ba742d8)"). If provided, use it directly — do not ask again.

If **not** provided, ask:
> "What is your Contentsquare tag ID? This is the hex string (e.g. `81c677ba742d8`) used as `clientId`. You can find it in your Contentsquare tag snippet or in Settings."

### Browser Automation

Check whether you have a browser automation MCP server. Look for tools matching any of these patterns:

- **Playwright MCP:** `browser_navigate`, `browser_network_requests`, `browser_console_messages`
- **Chrome DevTools MCP:** `chrome_navigate`, `chrome_network_requests`, `chrome_console_messages`, `cdp_navigate`, `mcp_chrome_devtoo_navigate_page`, `mcp_chrome_devtoo_list_network_requests`, `mcp_chrome_devtoo_list_console_messages`, or similar `chrome_*` / `cdp_*` / `mcp_chrome_devtoo_*` prefixed tools

If you find tools that can navigate to a URL, capture network requests, and read console output — regardless of prefix — you have browser automation.

- **If available:** Note which MCP you have (Playwright or Chrome DevTools). You can verify the tag automatically. Proceed.
- **If not available:** Tell the developer:
  > "I don't have browser automation tools. You'll need to check DevTools manually when I ask for verification. Alternatively, I can add the `@playwright/mcp@latest` MCP server config to your project — but you'll need to reload your editor, which ends this session. After reloading, paste your original prompt again to resume. Otherwise, we'll continue with manual verification"

---

## Flow

1. **Install** — detect framework, install SDK, add tag initialization code
2. **Verify** — confirm the tag loads and sends data in the browser
3. **CSP Check** — if violations detected, fix Content Security Policy
4. **Final Verification** — re-verify everything works end-to-end

---

## Step 1 — Install the Tag

### 1.1 Detect the framework

```bash
cat package.json | grep -E '"next"|"nuxt"|"vue"|"react"|"@angular/core"|"svelte"|"@sveltejs/kit"|"vite"|"@contentsquare/'
ls package-lock.json yarn.lock pnpm-lock.yaml bun.lockb 2>/dev/null
ls tsconfig.json 2>/dev/null
ls src/app app src/pages pages 2>/dev/null
ls src/main.ts src/main.tsx src/index.ts src/index.tsx app/layout.tsx app/layout.jsx src/app/layout.tsx src/app/layout.jsx pages/_app.tsx pages/_app.jsx 2>/dev/null
grep -r "contentsquare\|injectContentsquareScript\|_uxa" --include="*.ts" --include="*.tsx" --include="*.js" --include="*.jsx" --include="*.html" -l 2>/dev/null | grep -v node_modules
```

**Determine:**
- Framework (priority: Next.js > React, Nuxt > Vue, SvelteKit > Svelte, Angular, Vite)
- App Router or Pages Router (Next.js only) — check if `app/` or `src/app/` exists
- TypeScript (tsconfig.json present) or JavaScript
- Package manager (npm/yarn/pnpm/bun)
- Already installed? → Skip to Step 2

### 1.2 Confirm with the developer

> "I detected **[framework]** using **[TS/JS]** with **[package manager]**. The entry point is `[file]`. Correct?"

Do not proceed until confirmed. If you don't have the tag ID yet, ask now.

### 1.3 Install the tag

**Install the package:**
```bash
npm install @contentsquare/tag-sdk
# or: yarn add / pnpm add / bun add — match the detected package manager
```

**Add initialization code.** Replace `<TAG_ID>` with the developer's actual tag ID.

---

**Next.js — App Router** — create a client component at the path where `layout` lives (`app/contentsquare.tsx` or `src/app/contentsquare.tsx`):

```typescript
'use client';
import { useEffect } from 'react';
import { injectContentsquareScript } from '@contentsquare/tag-sdk';

export function Contentsquare() {
  useEffect(() => {
    injectContentsquareScript({ clientId: '<TAG_ID>' });
  }, []);
  return null;
}
```

Then add `<Contentsquare />` to the root layout (same directory):

```typescript
import { Contentsquare } from './contentsquare';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <Contentsquare />
        {children}
      </body>
    </html>
  );
}
```

---

**Next.js — Pages Router** — in `pages/_app.tsx`:

```typescript
import { useEffect } from 'react';
import { injectContentsquareScript } from '@contentsquare/tag-sdk';
import type { AppProps } from 'next/app';

export default function App({ Component, pageProps }: AppProps) {
  useEffect(() => {
    injectContentsquareScript({ clientId: '<TAG_ID>' });
  }, []);
  return <Component {...pageProps} />;
}
```

---

**React (Vite / CRA)** — in `src/main.tsx` (or `src/index.tsx`), before `ReactDOM.createRoot`:

```typescript
import { injectContentsquareScript } from '@contentsquare/tag-sdk';
injectContentsquareScript({ clientId: '<TAG_ID>' });
```

---

**Vue (Vite)** — in `src/main.ts`:

```typescript
import { injectContentsquareScript } from '@contentsquare/tag-sdk';
injectContentsquareScript({ clientId: '<TAG_ID>' });
```

---

**Nuxt 3** — create `plugins/contentsquare.client.ts`:

```typescript
import { injectContentsquareScript } from '@contentsquare/tag-sdk';

export default defineNuxtPlugin(() => {
  injectContentsquareScript({ clientId: '<TAG_ID>' });
});
```

---

**Angular** — in `src/main.ts`, before `platformBrowserDynamic().bootstrapModule(...)`:

```typescript
import { injectContentsquareScript } from '@contentsquare/tag-sdk';
injectContentsquareScript({ clientId: '<TAG_ID>' });
```

---

**SvelteKit** — in `src/routes/+layout.svelte`:

```svelte
<script lang="ts">
  import { onMount } from 'svelte';
  import { injectContentsquareScript } from '@contentsquare/tag-sdk';

  onMount(() => {
    injectContentsquareScript({ clientId: '<TAG_ID>' });
  });
</script>

<slot />
```

---

**Static HTML / Server-rendered** (Django, Rails, Laravel, PHP, WordPress, or no JS framework) — add inside `<head>`:

```html
<script src="https://t.contentsquare.net/uxa/<TAG_ID>.js" defer></script>
```

Common locations: Django → `templates/base.html`, Rails → `app/views/layouts/application.html.erb`, Laravel → `resources/views/layouts/app.blade.php`, WordPress → `header.php`.

> If you use the `_uxa` API for custom variables or virtual pageviews, use the IIFE loader pattern instead so commands are buffered before the script loads:

```html
 <script>
 (function(c,s,q,u,a,r,e){
   c[a]=c[a]||function(){(c[a].q=c[a].q||[]).push(arguments)};
   e=s.createElement(q);e.async=1;e.src=u;
   r=s.getElementsByTagName(q)[0];r.parentNode.insertBefore(e,r);
 })(window,document,'script','https://t.contentsquare.net/uxa/<TAG_ID>.js','_uxa');
 </script>
```

---

### 1.4 Prepare for verification

Ask the developer to **start their dev server**. Never start it yourself. Then proceed to Step 2.

---

## Step 2 — Verify the Installation

### 2.1 Get the URL

Suggest the default: Next.js/Nuxt/CRA → `localhost:3000`, Vite → `localhost:5173`, Angular → `localhost:4200`.

### 2.2 Check the tag

**With browser automation MCP (Playwright or Chrome DevTools):**
1. Navigate to the URL using the available navigation tool.
2. Wait at least 3 seconds after page load (the CS tag is async).
3. Capture network requests and check for `t.contentsquare.net/uxa/<TAG_ID>.js` returning HTTP 200.
4. Check for at least one additional request to `t.contentsquare.net` beyond the script load.
5. Read console output and check for CSP violations (`Refused to`, `Content-Security-Policy`).

**Without Playwright MCP — guide the developer:**
> Open DevTools (F12) → Network tab → reload → filter `contentsquare`. Look for:
> 1. A request to `t.contentsquare.net/uxa/<TAG_ID>.js` with status 200
> 2. Additional requests to `t.contentsquare.net` after it loads
>
> Also check Console for any `Refused to` or `Content-Security-Policy` errors.

### 2.3 Route based on results

| Result | Action |
|--------|--------|
| ✅ Script loads + data requests sent, no CSP errors | Proceed to Step 3 |
| ⚠️ CSP violations detected | Proceed to Step 3 |
| ❌ No request to `t.contentsquare.net` at all | Fix: ensure `injectContentsquareScript` runs client-side (useEffect/onMount), package is installed. Re-run Step 2. |
| ⚠️ Script loads but no data requests | Fix: verify the tag ID is correct, project is active. Re-run Step 2. |

---

## Step 3 — Check and Fix CSP

**Skip this step entirely** if Step 2 passed with no CSP errors AND no existing CSP is found.

### 3.1 Detect existing CSP

```bash
grep -r "Content-Security-Policy\|contentSecurityPolicy" next.config.* middleware.* 2>/dev/null
grep -r "Content-Security-Policy" --include="*.html" --include="*.ejs" --include="*.hbs" --include="*.blade.php" --include="*.erb" -l 2>/dev/null | grep -v node_modules
cat package.json | grep -E '"helmet"|"csp-header"|"content-security-policy"' 2>/dev/null
```

If **no CSP found** → skip to Step 4.

### 3.2 Required CSP additions

Add these domains to the project's **existing** CSP configuration. Do not replace the existing policy — append to each directive:

```
script-src:  'unsafe-inline' *.contentsquare.net app.contentsquare.com
connect-src: *.contentsquare.net *.contentsquare.com
img-src:     *.contentsquare.net
child-src:   blob:
worker-src:  blob:
```

### 3.3 Apply the fix

Locate the file where CSP is configured (from 3.1) and add the Contentsquare domains to each relevant directive. Preserve all existing values — only append.

**Example** (Next.js `next.config.ts` headers):
```typescript
"script-src 'self' 'unsafe-inline' *.contentsquare.net app.contentsquare.com",
"connect-src 'self' *.contentsquare.net *.contentsquare.com",
"img-src 'self' *.contentsquare.net",
"child-src 'self' blob:",
"worker-src 'self' blob:",
```

For other formats (middleware, meta tags, nginx, helmet), apply the same domains in the format the project already uses.

After fixing, ask the developer to **restart the dev server**.

---

## Step 4 — Final Verification

Re-run the same checks from Step 2. If Step 2 already passed and Step 3 found no CSP issues, skip this step.

**Pass:**
```
✅ Contentsquare setup complete!

- Tag installed: <framework> via @contentsquare/tag-sdk
- Tag script loading: ✓
- Data requests sending: ✓
- CSP: <no issues / fixed>
```

**Fail:** Report what's broken. Common issue: dev server not restarted after CSP changes.

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Tag not loading | Ensure `injectContentsquareScript` runs client-side only (`useEffect`, `onMount`, `.client.ts` plugin) |
| `injectContentsquareScript is not a function` | Check `@contentsquare/tag-sdk` is installed and imported correctly |
| CSP errors persist after fix | Restart the dev server so new headers are served |
| `eval` violation | Add `'unsafe-eval'` to `script-src` only if you see this specific error |
| Violations from `.contentsquare.com` | Add both `*.contentsquare.net` and `*.contentsquare.com` to `connect-src` |
