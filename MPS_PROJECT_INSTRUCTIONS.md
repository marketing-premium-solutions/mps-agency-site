# MPS Agency Site — Project Instructions v1.0
**Owner:** Marketing Premium Solutions  
**Live Site:** marketingpremiumsolutions.com  
**Dev Environment:** LocalWP → marketingpremiumsolutions.com  
**Stack:** WordPress + Elementor (free) + ElementsKit Lite + Forminator  
**Last Updated:** August 2026

---

## CRITICAL: Read This Before Doing Anything

This document is the single source of truth for the MPS agency website build.
Read it fully before writing, editing, or building anything.
Every page, section, widget, and line of copy must conform to these rules.

---

## 1. Business Details (Use These Everywhere — No Tokens Left Unreplaced)

```
Business Name:    Marketing Premium Solutions
Phone Display:    065 221 9944
Phone Link:       tel:+27652219944
WhatsApp:         https://wa.me/27652219944?text=Hi%20Marketing%20Premium%20Solutions%2C%20I%27d%20like%20to%20discuss%20growing%20my%20business
Email:            office@marketingpremiumsolutions.com
Address:          300 Witch-Hazel Ave, Eco-Park Estate, Centurion, 0144
Hours:            Mon–Thu 08:00–17:00 | Visits by appointment only
Website:          https://marketingpremiumsolutions.com
Founder:          Prayer Adrian Makanda
```

**Social Media (update when confirmed):**
```
Instagram:        {{INSTAGRAM}}
Facebook:         {{FACEBOOK}}
LinkedIn:         {{LINKEDIN}}
TikTok:           {{TIKTOK}}
```

---

## 2. Brand Design System (Locked — Never Deviate)

### Colours (global.json slots — never hardcode hexes)
```
primary:        #10112A  — navy — headers, footer, dark CTA bands
secondary:      #0046D6  — blue — buttons, links, icons
accent2:        #1B4FE0  — gradient partner only
accent_gold:    #F4B021  — eyebrows, stats, micro-accents ONLY
text:           #1B1F28  — headings and body copy
surface:        #F7F8FA  — alternate sections (use sparingly)
light:          #FFFFFF  — main background, cards (preferred)
secondary_dark: #003BB8  — hover states only
```

### The 80/15/4/1 Ratio (Sacred)
- **White 80%** — primary background always
- Navy 15% — footer, CTA band only
- Blue 4% — buttons, links, icons
- Gold 1% — eyebrows, stat accents, stars only
- **White backgrounds are strongly preferred throughout the site**
- Never use gold as a large background
- Never use all three accent colours in the same section

### Brand Gradient
```
linear-gradient(135deg, #10112A, #0046D6)
```
- Max 1 gradient surface per viewport
- Used on: featured stat cards, CTA band accent, hero accent shapes only

### Warm Gradient (micro-use only)
```
linear-gradient(60deg, #F4B021, #F12D1A)
```
- Used ONLY on the header phone button hover
- Never anywhere else on the site

### Typography
```
Headings:  Montserrat 600/700/800
Body:      DM Sans 400/500/600
H1:        Montserrat 800, 60px desktop / 36px mobile
H2:        Montserrat 700, 44px desktop / 30px mobile
H3:        Montserrat 600, 26px desktop / 21px mobile
H4:        Montserrat 600, 20px
Body:      DM Sans 400, 17px desktop / 16px mobile, lh 1.7
Eyebrow:   DM Sans 600, 13px UPPERCASE, letter-spacing 2.5px, gold colour
Button:    Montserrat 600, 15px
Stats:     Montserrat 800, 64px desktop / 44px mobile, gradient text fill
```

### Layout
```
Container max-width:  1280px
Content width:        1140px
Section padding:      120px top/bottom desktop, 72px mobile
Mobile gutters:       24px minimum — every section, non-negotiable
Column gap:           40px default, 60px on hero sections
Card radius:          16px
Card border:          1px solid #E8EAF0
Card hover:           border shifts to secondary, no shadow
Shadows:              NONE — borders only throughout the site
```

---

## 3. CSS Class Naming Conventions (Exact — Never Change)

```
mps-svc-dash   — hero stat dashboard card
mps-prob-grid  — problem section 3-card grid
mps-inc        — what's included icon grid
mps-scs        — featured case study card
mps-proc       — process steps row
mps-price      — pricing tiers grid
mps-uni        — universal inclusions grid
mps-add        — add-ons section
```

All colours via `var(--e-global-color-*)` — never hardcode hexes except:
- `#FFFFFF` — white
- `#25D366` — WhatsApp green
- `#E8EAF0` — card border
- `rgba()` inside the Brand Gradient only

---

## 4. Page Structure Rules

### Eyebrow chips
- Always `<div>` — never `<span>`
- Format: `• LABEL TEXT` (bullet + space + uppercase)
- Colour: accent_gold
- Font: DM Sans 600, 13px, letter-spacing 2.5px

### FAQ Accordion
- Widget: `elementskit-accordion`
- Field keys: `ekit_accordion_items` → `acc_title` + `acc_content`
- Always populated with real questions — never lorem ipsum

### Every page must have:
- Max 1 dark (navy) CTA band per page
- Max 1 gradient surface per viewport
- Floating WhatsApp button (fixed bottom-right, #25D366)
- Real business details — no unreplaced tokens

### Mobile
- 24px side padding on every section — enforced via stamper
- All sections stack cleanly at 375px
- Images: WebP, quality 75, descriptive alt text
- Hero images: max 200KB, fetchpriority="high", NOT lazy-loaded
- Card images: max 100KB

---

## 5. Existing Pages (Phase 1 — Complete)

| Page | URL | Status |
|---|---|---|
| Homepage | / | ✅ Live |
| About Us | /about-us | ✅ Live |
| Contact | /contact | ✅ Live |
| Web Design | /web-design | ✅ Live |
| Google Ads Management | /google-ads-management | ✅ Live |
| SEO Services | /seo-services | ✅ Live |

---

## 6. Confirmed Pricing (Live on Site — Never Change Without Instruction)

### Web Design (once-off)
```
Starter:          R4,500   — 3 pages, basic SEO, 14-day turnaround
Business:         R8,500   — 6 pages, full SEO, schema, GA4 ⭐ Most Popular
Growth:           R12,000  — 10 pages, advanced SEO, blog, integrations 🏆 Best Value
Custom/E-Commerce: from R20,000 — bespoke, e-commerce, custom integrations
```

### Google Ads Management (monthly)
```
Starter:    R2,500/month  — 1 campaign, min R3,000 ad spend
Business:   R4,000/month  — 3 campaigns, min R5,000 ad spend ⭐ Most Popular
Growth:     R6,500/month  — unlimited campaigns, min R10,000 ad spend 🏆 Best Value
Enterprise: from R10,000/month — bespoke, R30k+ ad spend
```

### SEO Services (monthly)
```
Local SEO Starter: R2,500/month — 1 location, GBP, 5-10 keywords
Business SEO:      R4,500/month — 15-25 keywords, blog, schema ⭐ Most Popular
Growth SEO:        R7,500/month — 30-50 keywords, backlinks, location pages 🏆 Best Value
Enterprise:        from R12,000/month — bespoke, national/e-commerce
```

### Local SEO (monthly) — NEW PAGE
```
Starter:    R2,500/month  — 1 location, GBP + local pack
Business:   R3,500/month  — up to 3 locations + location pages ⭐ Most Popular
Growth:     R5,500/month  — up to 10 locations, full local domination 🏆 Best Value
Enterprise: from R8,500/month — multi-city, franchise, national local
```

### Universal Add-Ons (across all pages)
```
Hosting & Maintenance:        R450/month
Local SEO Pages Bundle:       R3,500 once-off (15 location pages)
SEO Audit of Existing Site:   R750 once-off (credited back if you sign)
GBP Setup & Optimisation:     R1,500 once-off
Schema Markup Implementation: R1,500 once-off
Technical SEO Audit:          R2,500 once-off
Single Blog Post:             R750 each
Content Refresh Package:      R2,500 once-off
AI Search Optimisation:       R1,500/month add-on
Photography:                  R3,500 half-day / R6,500 full-day
Drone Photography & Video:    R4,500 half-day
Logo Design:                  R2,500 once-off
Brand Guidelines Document:    R3,500 once-off
Additional Revisions:         R750/round
Extra Pages:                  R750/page
Rush Turnaround:              +30% on package price
```

---

## 7. Service Page Template Structure (10 Sections)

Every service page follows this exact structure:

```
Section 1:  Hero — split layout, stat card right, copy + CTAs left
Section 2:  Problem — 3 pain cards + "That's not what we do" closer
Section 3:  What's Included — 6-icon grid (mps-inc)
Section 4:  Featured Case Study (mps-scs)
Section 5:  Process — 5 numbered steps (mps-proc)
Section 6:  Pricing — tiers + banner (mps-price)
Section 7:  Universal Inclusions (mps-uni)
Section 8:  Add-Ons (mps-add)
Section 9:  FAQ — 6 questions, elementskit-accordion, real content
Section 10: CTA Band — navy, Brand Gradient radial, dual CTA
```

---

## 8. Phase 2 Pages (To Build)

| Page | URL | Status |
|---|---|---|
| Local SEO | /local-seo | 🔲 Next |
| TikTok Management | /tiktok-management | 🔲 Planned |
| Photography & Drone | /photography-drone | 🔲 Planned |
| Case Studies Index | /case-studies | 🔲 Planned |
| Industry: Dentists | /dentist-marketing | 🔲 Planned |
| Industry: Law Firms | /law-firm-marketing | 🔲 Planned |
| Industry: Greenhouse | /greenhouse-marketing | 🔲 Planned |
| Industry: Travel | /travel-agency-marketing | 🔲 Planned |
| Blog | /blog | 🔲 Planned |

---

## 9. Deployment Process

1. Build and test in LocalWP first
2. Claude Code writes/edits files in `C:\Users\User\Studio\mps-agency-site\`
3. LocalWP path: `C:\Users\User\Local Sites\marketing-premium-solutions\app\public\`
4. Export page as JSON from Elementor → import on live site
5. Or zip plugin/kit and upload via Plugins → Add New → Upload Plugin

---

## 10. SEO Requirements (Every Page)

- Rank Math configured: meta title + description
- One H1 per page, strict H2 → H3 → H4 hierarchy
- Schema: Service schema on every service page
- Internal links: every service page links to 2 other service pages + relevant case studies
- All images: WebP, descriptive alt text, never empty
- No lorem ipsum, no unreplaced tokens anywhere

---

## 11. Quality Checklist (Before Any Page Goes Live)

- [ ] All business details real — no {{TOKENS}} remaining
- [ ] Renders correctly at 1440px desktop, 768px tablet, 375px mobile
- [ ] 24px mobile gutters on every section
- [ ] No shadows — borders only
- [ ] Max 1 dark band per page
- [ ] Max 1 gradient per viewport
- [ ] Gold used at 1% only — no gold backgrounds
- [ ] All eyebrows are `<div>` not `<span>`
- [ ] FAQ accordion has real questions — no lorem
- [ ] All buttons link to correct URLs
- [ ] WhatsApp link uses pre-filled message
- [ ] Images WebP, alt text present
- [ ] PageSpeed mobile 90+ target

---

## 12. Real Stats (Use These Exactly)

```
Websites built:         122
Active businesses:      100+
Revenue generated:      R50M+
Ad spend managed:       R1M+
Google rating:          5.0
Reviews:                100+
```

---

*Update this document whenever pricing, pages, or business details change.  
Always paste this into Claude Code at the start of every new session.*
