---
name: magento-frontend-development
description: Implement or fix Magento storefront layouts, templates, styling, JavaScript, and customer interactions. Use for frontend changes in the existing theme or storefront stack.
---

# Magento frontend development

## Locate the rendering path

Identify the active storefront stack and actual template/component before editing. For theme-based storefronts, trace the parent theme and layout handle. Follow the installed Luma, Hyvä, headless, or custom implementation; a feature request does not imply a stack migration.

## Implementation decisions

- Prefer a targeted style or layout change over copying a whole template. Keep `vendor/` unchanged and respect theme fallback. Use a mixin or supported component extension when it avoids maintaining a full JavaScript override.
- Keep data fetching and business logic out of templates. Reuse existing blocks or introduce a ViewModel where needed, passing it through layout arguments. Do not add a custom block or DI configuration without a reason.
- Escape dynamic output at the rendering boundary for its actual HTML, attribute, URL, or JavaScript context. Keep data reusable upstream; avoid double escaping trusted rendered HTML. Use supported CSP mechanisms for inline code.
- Preserve full-page caching and cache invalidation. A `cacheable="false"` block makes its entire page uncacheable. Use the storefront's private-content mechanism for customer-specific data; on Luma, check section sources and `etc/frontend/sections.xml` invalidation. Do not place sensitive values in browser storage just to make them dynamic.
- Follow existing styles, design tokens, and asset tooling. Reserve image space, avoid unnecessary asset loading, and preload only a measured critical resource. Keep interactions usable by keyboard with semantic controls and visible focus.

## Verify and deliver

Build affected assets with the project's commands. Check relevant viewport sizes, interaction states, and browser errors; for personalized content, include guest/customer and warm-cache behavior. State when browser verification is unavailable. Update existing documentation only for new configuration, styling conventions, or behavior that needs explanation.

Use Adobe's [XSS guidance](https://developer.adobe.com/commerce/php/development/security/cross-site-scripting) and [private-content documentation](https://developer.adobe.com/commerce/php/development/cache/page/private-content) when those paths change.
