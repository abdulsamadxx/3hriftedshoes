# AI_CONTEXT

## Project
3hriftedshoes — static, single-page storefront for thrifted/pre-loved sneakers.

## Architecture
- Static `index.html` containing HTML, CSS, and vanilla JavaScript.
- Product/site content in `data/products.json` and `data/site.json`.
- Vercel deployment configuration in `vercel.json`.
- No backend server, database, package manager, or framework is present in the supplied project.
- Admin publishing uses the GitHub Contents API directly from browser JavaScript.

## Audit scope
- Focus: P0/P1 and genuinely important P2 functional/security/production issues.
- Preserve existing UI, routes, data filenames, and functionality.
- Avoid unnecessary refactoring or dependencies.

## Admin image workflow (gallery upload)
- Admin panel no longer asks for image URLs to paste. Each product card shows a thumbnail grid; a "+" tile opens the phone's gallery/camera picker (`<input type="file" accept="image/*">`), and each existing photo has a small replace (↻) and delete (✕) control.
- On selection, a photo is resized client-side (canvas, max 1600px edge, JPEG quality 0.82) and committed directly to the GitHub repo via the same Contents API the admin panel already uses for `products.json`/`site.json` (`ghPutBinaryFile`, `resizeImageFile`, `uploadGalleryImage` in index.html). Product photos land in `images/products/`, the banner photo in `images/site/`.
- The resulting `https://raw.githubusercontent.com/...` URL is pushed into `p.images` (or `SITE.banner`) immediately, so the thumbnail shows right away — but it isn't live for visitors until "Publish Changes" commits the updated `products.json`/`site.json` pointing at it.
- Banner image gained the same "Gallery se Upload" button alongside its existing URL field (URL field kept as a manual fallback for both banner and, historically, products).

## Fixes made in current audit
- Escaped product-controlled values in compare/recently-viewed HTML rendering to prevent stored HTML/script injection through admin-editable product data.
- Made account greeting rendering DOM-safe instead of inserting user-supplied name through `innerHTML`.
- Added defensive filtering of remotely loaded product records so malformed entries do not replace valid built-in data and crash rendering.
- Made product star rendering and price formatting tolerant of malformed numeric values.
- Kept `filterBrand` synchronized with admin-edited brand values for Nike/Adidas/Puma/other filters.

## Known architectural risk not silently redesigned
The admin panel accepts a GitHub personal access token in the browser and stores it in `localStorage`. A secure server-side admin/authentication boundary would require an architectural change and cannot be safely implemented as a tiny static-file patch.
