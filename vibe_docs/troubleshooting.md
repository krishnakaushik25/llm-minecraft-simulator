# Troubleshooting Guide

## Error Log Format
For each error, document:

### [Error Title/Type]
**Date:** [YYYY-MM-DD]
**Error Message:**
```
[Exact error message]
```

**Context:** [What were you trying to do?]
**Root Cause:** [Why did this happen?]
**Solution:**
1. [Step-by-step solution]
2. [Include exact commands used]

**Prevention:** [How to avoid this in the future]
**Related Files:** [Which files were modified to fix this]

---

## Error: terser not found during build
**Date:** 2025-01-30
**Error Message:**
```
[vite:terser] terser not found. Since Vite v3, terser has become an optional dependency. You need to install it.
```

**Context:** Attempting to run `npm run build` or `npm run deploy`.
**Root Cause:** The `terser` package, required by Vite for minification during the build process, was not installed.
**Solution:**
1. Install terser as a dev dependency.
2. Run `npm install --save-dev terser`.

**Prevention:** Ensure `terser` is included in dev dependencies for Vite projects using terser minification.
**Related Files:** package.json, vite.config.js, vibe_docs/environment_setup.md

---

## Issue: Deployment to GitHub Pages is slow or interrupted
**Date:** 2025-01-30
**Error Message:** (No specific error message, command is interrupted)
```
^C
```

**Context:** Running `npm run deploy` or `npx gh-pages -d dist` to deploy the project to GitHub Pages.
**Root Cause:** Deploying the project, especially with large files like the LLM model, can take significant time depending on network speed. Interrupting the command (`^C`) stops the deployment process before it can complete.
**Solution:**
1. Ensure the command runs to completion without interruption.
2. Allow sufficient time for the files to be transferred and the gh-pages branch to be updated (this can take several minutes).

**Prevention:** Be aware that the first deployment or deployments with large changes may take time. Avoid interrupting the terminal command during the deployment process.
**Related Files:** package.json, vite.config.js, vibe_docs/development_log.md

---

## Issue: Keyboard input interfering with AI assistant input
**Date:** 2025-01-30
**Error Message:** (No specific error message, game controls trigger while typing)
```
(No specific error message)
```

**Context:** Typing into the AI assistant input field causes game actions to be triggered.
**Root Cause:** The game's keyboard input listeners were active even when the AI assistant input field was focused, capturing key presses intended for typing.
**Solution:**
1. Get a reference to the AI assistant input element.
2. Add `focus` and `blur` event listeners to the input element.
3. Use a flag (`inputFocused`) to track whether the input field is focused.
4. Modify the game's `keydown` and `keyup` listeners to only process game controls if the `inputFocused` flag is false.

**Prevention:** Always consider how different interactive UI elements might interfere with global input listeners in a web application.
**Related Files:** src/game.js, index.html, vibe_docs/troubleshooting.md

---

## Issue: Railway deployment failing with "terser not found" despite terser being in devDependencies
**Date:** 2025-01-30
**Error Message:**
```
[vite:terser] terser not found. Since Vite v3, terser has become an optional dependency. You need to install it.
```

**Context:** Deploying to Railway platform where build process requires terser for minification.
**Root Cause:** Railway runs `npm ci` in production mode, which skips `devDependencies`. But static site builds require build tools during the production build process.
**Solution:**
1. Move critical build dependencies from `devDependencies` to `dependencies`:
   - `terser`: needed for minification during build
   - `vite`: needed for the build process
   - `marked`: used at runtime in the browser
2. Run `npm install` to update `package-lock.json`
3. Commit and push the updated `package.json` and `package-lock.json`

**Prevention:** For static site deployments, consider whether build tools should be in `dependencies` rather than `devDependencies` if the platform builds in production mode.
**Related Files:** package.json, package-lock.json

---

## Issue: Railway using cached Docker layers preventing dependency updates
**Date:** 2025-01-30
**Error Message:**
```
[stage-0  6/10] RUN npm ci  ✔ 0ms – CACHED
[vite:terser] terser not found. Since Vite v3, terser has become an optional dependency.
```

**Context:** Railway continues to fail with "terser not found" even after moving terser to dependencies, due to Docker layer caching.
**Root Cause:** Railway's Docker build system caches the `npm ci` step, so even after updating `package.json`, the cached `node_modules` from the previous build is still being used.
**Solution:**
1. Force cache invalidation by making file changes that affect Docker context:
   - Bump the version number in `package.json`
   - Create a dummy file (e.g., `.railway-rebuild`) with timestamp
2. Run `npm install` to update `package-lock.json`
3. Commit and push changes to trigger fresh Railway deployment

**Prevention:** When updating dependencies for cloud deployments, consider that Docker layer caching may prevent updates from taking effect immediately.
**Related Files:** package.json, package-lock.json, .railway-rebuild

---

## Issue: Railway treating browser JavaScript as Node.js application
**Date:** 2025-01-30
**Error Message:**
```
ReferenceError: document is not defined
at file:///app/src/main.js:169:1
```

**Context:** Railway successfully builds the project but fails to start because it's trying to execute browser JavaScript with `node src/main.js`.
**Root Cause:** Railway/Nixpacks automatically detects this as a Node.js application and tries to run the main entry point with Node.js, but `src/main.js` contains browser-specific code that uses `document` and DOM APIs.
**Solution:**
1. Add a proper start script in `package.json`: `"start": "vite preview --host 0.0.0.0 --port $PORT"`
2. Create `railway.toml` configuration file to specify deployment settings
3. Configure Railway to serve static files instead of executing browser JavaScript

**Prevention:** For static web applications, ensure the deployment platform is configured to serve built files rather than execute source code.
**Related Files:** package.json, railway.toml, .railway-rebuild

---

## Issue: Railway health checks blocked by Vite preview allowedHosts
**Date:** 2025-01-30
**Error Message:**
```
Blocked request. This host ("healthcheck.railway.app") is not allowed.
To allow this host, add "healthcheck.railway.app" to `preview.allowedHosts` in vite.config.js.
```

**Context:** Railway deployment starts successfully but health checks fail repeatedly, preventing the service from becoming fully available.
**Root Cause:** Vite's preview server blocks requests from unknown hosts for security. Railway's health check system uses "healthcheck.railway.app" as the host, which isn't in the default allowlist.
**Solution:**
1. Add `preview` configuration to `vite.config.js`
2. Include `allowedHosts: ['healthcheck.railway.app']` in the preview settings
3. Deploy the updated configuration

**Prevention:** When using Vite preview server for production deployments, ensure all necessary health check and monitoring hosts are in the allowedHosts list.
**Related Files:** vite.config.js, .railway-rebuild

---
