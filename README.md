# Memento Mundi — Phygital Travel Scrapbook PWA  https://memento-mundi-nine.vercel.app/

> Bridging physical NFC collectibles with collaborative, infinite-canvas digital travel journals.

[![Next.js](https://img.shields.io/badge/Next.js-App_Router-0B2130?style=flat-square&logo=next.js)](https://nextjs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-Strict-0B2130?style=flat-square&logo=typescript)](https://www.typescriptlang.org/)
[![Supabase](https://img.shields.io/badge/Supabase-PostgreSQL_%2B_RLS-0B2130?style=flat-square&logo=supabase)](https://supabase.com/)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-Soft_Brutalism-0B2130?style=flat-square&logo=tailwindcss)](https://tailwindcss.com/)
[![Framer Motion](https://img.shields.io/badge/Framer_Motion-Physics_%26_Gestures-0B2130?style=flat-square&logo=framer)](https://www.framer.com/motion/)

---

## Overview

**Memento Mundi** is a full-stack **Progressive Web Application (PWA)** that connects physical hardware—custom die-cut stickers, keychains, and lighters embedded with **NTAG213 NFC chips**—to shared digital travel scrapbooks. Scanning a physical item with a smartphone bypasses App Store friction and deep-links directly into a collaborative canvas where travelers co-curate photos, notes, and location memories.

**[Live Demo](https://your-demo-url.vercel.app)** · **[Report Bug](https://github.com/your-username/memento-mundi/issues)**

---

## Key Features

* **Hardware-to-Cloud NFC Routing:** Maps unique NTAG213 chip token hashes (`token_hash`) to specific travel scrapbooks, unlocking instant mobile access via background NFC reading on iOS and Android.
* **Infinite-Canvas Collaborative Editor:** Free-form panning, zooming, and tactile drag-and-drop placement of photos, text notes, and custom frames, persisted in real time with debounced state updates.
* **Multi-User Permissions & Privacy:** Supports shared trips (`owner`, `editor`, `viewer` roles) alongside private/public visibility controls enforced at the database layer.
* **Memory-Safe Social Feed:** Progressive on-demand pagination (`IntersectionObserver` / cursor-based fetching) to prevent DOM bloat and high memory footprint on mobile devices, paired with user-scoped private/public profile filtering.
* **Native-Feel Standalone PWA:** Configured with `display: "standalone"`, custom Apple Web App metadata, safe-area viewport insets, and touch-action CSS locks (`-webkit-touch-callout`, `overscroll-behavior`) to replicate native iOS/Android ergonomics without wrapper overhead.
* **Integrated Collectible Store & UGC Engine:** Built-in e-commerce catalog (`price_cents`, SKU stock tracking) and a User-Generated Content (UGC) reward verification pipeline.

---

## Architecture & Engineering Decisions

### 1. Hybrid Relational + JSONB Canvas Persistence
Rather than normalizing every visual property of a free-form canvas into rigid columns, `scrapbook_elements` stores spatial coordinates (`x`, `y`), rotation angles, z-index layering (`sort_order`), and frame styles inside a structured **`JSONB` payload**, while keeping core entities (`profiles`, `scrapbooks`, `scrapbook_members`, `stickers`, `products`) strictly relational. This keeps schema migrations clean while supporting arbitrary future canvas widgets.

### 2. Zero-Trust Database Security with PostgreSQL RLS
All client-side queries run directly against Supabase using strict **Row-Level Security (RLS)** policies:
* Users can only read or mutate `scrapbook_photos` and `scrapbook_elements` if they are the `owner_id` or hold an active `editor` record in `scrapbook_members`.
* Profile feeds isolate private uploads so each user exclusively accesses their own private memories while exposing public entries to the social stream.
* Automated PostgreSQL triggers (`handle_new_user`) provision profile records atomically upon Supabase Auth registration.

### 3. Tactile Motion Physics & Asset Optimization
Built with a **Soft Brutalism** design system (`#EAE7DA` matte paper surface, `#0B2130` ink borders, hard offset shadows) powered by `framer-motion` spring physics (`stiffness: 300, damping: 24`) and full-viewport media intro sequences with scroll-lock callbacks.

---

## Tech Stack

| Layer | Technologies |
| :--- | :--- |
| **Framework** | Next.js (App Router), React, TypeScript |
| **Styling & Motion** | Tailwind CSS, Framer Motion |
| **Backend & Auth** | Supabase (PostgreSQL, Auth, Storage, Row-Level Security) |
| **Hardware & PWA** | NTAG213 NFC Deep-Linking, Web App Manifest (`standalone`), iOS Safe-Area APIs |
| **Deployment** | Vercel |

---

## Database Schema Overview

```text
auth.users ──▶ profiles (handle_new_user trigger)
                  ├──▶ scrapbooks ──▶ scrapbook_members (owner / editor / viewer)
                  │         ├──▶ scrapbook_photos (geolocation + storage path)
                  │         └──▶ scrapbook_elements (JSONB canvas state)
                  ├──▶ stickers (NTAG213 token_hash ──▶ scrapbook_id)
                  └──▶ ugc_claims (social proof verification & rewards)
products (standalone active catalog with SKU & cent-based pricing)
