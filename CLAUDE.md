# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Devin Burstein's personal career site: plain static HTML/CSS/JS, no build step, no package manager, no framework. Deployed on Vercel with the "Other" framework preset (no build command, no output directory — Vercel just serves the files as-is).

## Commands

There is no build, lint, or test tooling in this repo — it's hand-written static files.

- **Preview locally**: `python3 -m http.server 8000` from the repo root, then open `http://localhost:8000/index.html`. (Opening the file directly via `file://` also works for most content, but the contact form's `fetch()` POST and some relative paths behave better served over HTTP.)
- **Deploy**: push to `main` on GitHub. Vercel is connected via Git integration and auto-deploys on every push — there is no `vercel` CLI usage or manual deploy step in this workflow.
- **Committing**: the user pushes via GitHub Desktop rather than `git push` from the terminal. It's fine to `git add`/`git commit` locally, but don't assume a commit is live until the user has pushed it.

## Architecture

**Two pages, one stylesheet.** `index.html` (the main single-page site) and `whatilike.html` (a secondary page) both link the same `style.css`. All design tokens (colors, fonts, spacing) live in the `:root` custom properties at the top of `style.css` — that's the single source of truth for the visual system. There is no dark-mode variant; the palette is intentionally fixed.

**Content patterns repeat two layouts, both borderless-divider lists rather than boxed cards:**
- `.timeline > .timeline-item` — used for Experience and Part-Time & Fractional roles (icon + date/location, role/company/description).
- `.content-list > .content-row` — used for Skills, and for Hobbies/Events/People on `whatilike.html` (category label + description). Cards are used sparingly by design; prefer this divided-list pattern over introducing new bordered boxes.

**Section rhythm**: sections alternate between the primary background and `--bg-soft` by targeting specific `#id` selectors in CSS (`#experience, #skills, #hobbies, #people`), not a nth-child pattern — adding a new section to the rhythm means adding its id to that selector list.

**Internationalization (index.html only)**: a `translations` object in the inline `<script>` at the bottom of `index.html` holds `en`/`es`/`pt`/`fr` copy, keyed by string id. Elements opt in via `data-i18n="key"` (text content) or `data-i18n-placeholder="key"` (input placeholder); `applyLang()` swaps them and persists the chosen language to `localStorage`. By convention, only section labels/headings/UI chrome are translated — the actual Experience/Fractional role entries (dates, titles, descriptions) are English-only and carry no `data-i18n` attribute. `whatilike.html` has no i18n at all.

**Nav active-state** is a scroll-spy: an `IntersectionObserver` in `index.html`'s script watches each section and toggles `.active` on the matching `.nav-links a[href="#id"]`. On `whatilike.html`, the "What I Like" nav link is just statically marked `class="active"` in the markup since it's a separate page, not an anchor.

**Contact form** submits via `fetch()` directly to a hardcoded Formspree endpoint (`https://formspree.io/f/mwlkprab`, set as the form's `action`) — there is no serverless function or backend. On success/failure it shows a toast using the `toast`/`toast_error` i18n strings.

**Motion**: reveal-on-scroll (`.reveal` class) and the nav scroll-spy both use plain `IntersectionObserver`, no animation library. A global `prefers-reduced-motion` media query in `style.css` collapses all transitions/animations for users who request it — respect this when adding new motion rather than adding one-off exceptions.
