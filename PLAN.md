# Widforss Shorts — Production Implementation Plan

## 1. Sanity Schema Design

### Core Document: `short`

```js
{
  name: 'short',
  title: 'Short',
  type: 'document',
  fields: [
    // Video asset (Mux)
    { name: 'video', type: 'mux.video', validation: Rule => Rule.required() },
    { name: 'thumbnail', type: 'image', options: { hotspot: true } },

    // Metadata
    { name: 'title', type: 'string', validation: Rule => Rule.required().max(60) },
    { name: 'caption', type: 'text', rows: 2, validation: Rule => Rule.max(200) },
    { name: 'slug', type: 'slug', options: { source: 'title' } },

    // Linked products (Centra)
    {
      name: 'products',
      title: 'Shop the Look',
      type: 'array',
      of: [{
        type: 'object',
        name: 'taggedProduct',
        fields: [
          { name: 'centraProductId', type: 'string' },
          { name: 'centraDisplayUri', type: 'string' },
          { name: 'label', type: 'string' },
          { name: 'timestampStart', type: 'number', description: 'Show from (seconds)' },
          { name: 'timestampEnd', type: 'number', description: 'Show until (seconds)' },
        ],
      }],
    },

    // Creator
    { name: 'creator', type: 'reference', to: [{ type: 'creator' }] },

    // Categorization
    { name: 'tags', type: 'array', of: [{ type: 'reference', to: [{ type: 'shortsTag' }] }] },
    { name: 'collection', type: 'reference', to: [{ type: 'shortsCollection' }] },
    {
      name: 'department',
      type: 'string',
      options: { list: ['herr', 'dame', 'barn', 'jakt', 'friluftsliv', 'hund'] },
    },

    // UGC / Moderation
    {
      name: 'source',
      type: 'string',
      options: { list: ['editorial', 'brand', 'ugc', 'influencer'] },
      initialValue: 'editorial',
    },
    {
      name: 'moderationStatus',
      type: 'string',
      options: { list: ['pending', 'approved', 'rejected'] },
      initialValue: 'approved',
    },
    { name: 'moderationNotes', type: 'text' },

    // Publishing
    { name: 'publishedAt', type: 'datetime' },
    { name: 'featured', type: 'boolean', initialValue: false },
    { name: 'sortOrder', type: 'number' },
  ],
}
```

### Supporting: `creator`

```js
{
  name: 'creator',
  type: 'document',
  fields: [
    { name: 'name', type: 'string' },
    { name: 'handle', type: 'slug' },
    { name: 'avatar', type: 'image', options: { hotspot: true } },
    { name: 'instagramUrl', type: 'url' },
    { name: 'bio', type: 'text', rows: 2 },
    { name: 'type', type: 'string', options: { list: ['staff', 'ambassador', 'community'] } },
    { name: 'affiliateCode', type: 'string' },
  ],
}
```

### Supporting: `shortsTag` + `shortsCollection`

Tags for hashtag-style filtering, collections for curated campaigns (e.g. "Vinterjakten 2026").

---

## 2. Video Hosting — Mux

**Why Mux:**
- First-party Sanity plugin (`sanity-plugin-mux-input`) — upload directly from Studio
- Adaptive bitrate streaming (HLS) out of the box
- Auto-generated thumbnails at any timestamp
- Mux Data gives engagement analytics per video (completion rate, drop-off points, rebuffer %)
- Content moderation built in
- Animated GIF generation for email campaigns

**Cost estimate:** ~100 shorts at 30s each, 50k views/month = ~$75/month.

**Delivery chain:**
```
Sanity Studio → Mux (transcode) → HLS stream → @mux/mux-player-react
```

---

## 3. Centra Product Linking

Sanity stores only `centraProductId` + `centraDisplayUri` — never product data.

**"Shop the Look" flow:**
1. User clicks CTA → slide-up drawer appears over video
2. Next.js API route calls Centra CheckoutAPI with product IDs
3. Returns live price, stock, images, variants (respects NO/SE market)
4. "Legg i handlekurv" adds to existing Centra cart session

**Deep links:** `/produkt/{slug}?utm_source=shorts&utm_content={short.slug}`

**Sanity product picker:** Custom input component that searches Centra's product catalog, so editors never mistype IDs.

---

## 4. User-Generated Content (UGC)

### Upload Flow (Phase 1)
- Dedicated page: `/shorts/last-opp`
- Requires Centra customer login
- Form: video, title, caption, tags, optional product tagging
- Validation: max 60s, max 200MB, 9:16 preferred
- Upload via Mux direct upload (signed URL, no server bottleneck)
- Creates Sanity doc with `source: 'ugc'`, `moderationStatus: 'pending'`

### Moderation
- Custom Sanity Studio desk structure: "UGC Moderation" filtered to pending
- Actions: Approve / Reject / Request changes
- Mux auto-flags explicit content via webhook
- Email notifications on status change

### Phase 2: In-app recording
- Mobile PWA with `MediaRecorder` API — capture from camera directly
- Removes "upload a file" friction entirely

---

## 5. Next.js Implementation

### Route Structure
```
/shorts                     → SSR feed (first 5 videos)
/shorts/[slug]              → deep link to single short (OG meta for sharing)
/shorts/samling/[collection]→ collection feed
/shorts/tagg/[tag]          → tag feed
/shorts/last-opp            → UGC upload (auth-gated)
/api/shorts/feed            → cursor-paginated feed
/api/shorts/products        → Centra product lookup
/api/shorts/upload          → Mux upload URL generator
```

### Video Preloading Strategy
```
Slide N-1:  preload="none"     (watched, release memory)
Slide N:    preload="auto"     (playing)
Slide N+1:  preload="metadata" (next up)
Slide N+2+: preload="none"     (not yet needed)
```

Unmount `<MuxPlayer>` components more than 2 slides from current to prevent memory leaks on infinite scroll.

### Sharing / SEO
Server-render OG tags on `/shorts/[slug]`:
```html
<meta property="og:type" content="video.other" />
<meta property="og:video" content="https://stream.mux.com/{id}.m3u8" />
<meta property="og:image" content="https://image.mux.com/{id}/thumbnail.jpg" />
```
Shared links render rich previews on social platforms — driving traffic back to Widforss instead of Instagram.

---

## 6. Features Beyond the Prototype

### Timeline Product Tags
Products appear as floating pills on the video at specific timestamps. When creator holds up a jacket at second 12, the tag appears and links to that jacket.

### Shorts Widget on PLPs/PDPs
Small carousel on product pages: "Se denna i aktion" — query all shorts that reference that product. Creates a feedback loop between shorts and product discovery.

### Engagement Analytics
Mux Data per-video: views, completion rate, drop-off heatmap. Expose as read-only tab in Sanity Studio so content team knows what works.

### Affiliate Tracking for Creators
Creator `affiliateCode` is attached to cart sessions when viewers buy through their shorts. Enables commission reporting and incentivizes UGC.

### AI Auto-Tagging
On video upload → extract keyframes → vision model suggests tags and department. Reduces editorial effort, especially for UGC.

### Animated GIFs for Email
Mux generates animated GIFs from any playback ID:
```
https://image.mux.com/{id}/animated.gif?start=2&end=5&width=300
```
Use in Klaviyo campaigns: "Se veckans populara shorts" — much higher click-through than static images.

---

## 7. Mobile Experience

### Layout Shift (< 768px)
- Video goes full viewport (100vw, 100dvh)
- Info overlays bottom of video (like Reels)
- Thumbnails sidebar disappears — swipe only
- "Shop the look" becomes bottom sheet
- Mute button top-right

### Touch Gestures
- Swipe up/down: next/prev (CSS scroll-snap handles natively)
- Tap: pause/play
- Long press: product tags + share
- Swipe left: open shop drawer

### Performance
- Cap at 720p on mobile: `stream.mux.com/{id}.m3u8?max_resolution=720p`
- IntersectionObserver with rootMargin to preload next video 200px early
- Service worker caches shell, fonts, branding

### PWA
Basic manifest enables "Add to Home Screen" — repeat visitors get app-like access. Push notifications in Phase 3.

---

## 8. Architecture

```
Sanity Studio (content + moderation)
        │                    │
    GROQ API            Mux plugin
        ▼                    ▼
   Sanity CDN              Mux
   (metadata)         (video stream)
        │                    │
        └────────┬───────────┘
                 ▼
          Next.js App
         /shorts pages
         /api routes
                 │
                 ▼
              Centra
        (products, cart,
         customer auth)
```

---

## 9. Phases

| Phase | Scope | Est. |
|-------|-------|------|
| **1: Core Feed** | Sanity schemas, Mux, Next.js /shorts, Shop the look drawer, responsive | 2-3 weeks |
| **2: Discovery** | Timeline product tags, tag/collection feeds, shorts on PLPs/PDPs, share | 1-2 weeks |
| **3: UGC** | Upload form, Mux direct upload, moderation dashboard, creator profiles | 2-3 weeks |
| **4: Growth** | Affiliate tracking, analytics in Studio, AI tagging, PWA, email GIFs | Ongoing |

---

## 10. Key Decisions

| Decision | Recommendation | Why |
|----------|---------------|-----|
| Video hosting | Mux | Sanity plugin, HLS, analytics, moderation |
| Player | @mux/mux-player-react | Handles HLS, preload, analytics |
| Product data source | Centra CheckoutAPI at render time | Fresh price/stock, market-aware |
| Product data in Sanity | ID + slug only | No sync needed, Centra is truth |
| Feed pagination | Cursor-based (publishedAt) | Stable under new content |
| Mobile scroll | CSS scroll-snap | Native performance, no JS library |
| UGC uploads | Mux direct uploads | No server bottleneck |
| Moderation | Sanity Studio | Content team's existing tool |
