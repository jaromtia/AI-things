# SaaS UI Playbook: Creating and Improving UI Pages & Codebases

**Source:** [How I Design SaaS That Looks EXPENSIVE](https://www.youtube.com/watch?v=9d5fRVDkzRI) by Oliver (formerly Response AI)  
**Extended reference:** [roswell.dev](https://roswell.dev) — design rules, UI prompts, and full playbook

---

## Executive Summary

The gap between a "vibe-coded" app and a product people pay for is rarely the code. AI tools (Cursor, Lovable, Bolt, etc.) can produce functional software quickly, but they apply the same visual defaults every time. Users who look at software all day recognize those defaults instantly.

**Core thesis:** You do not need design taste. You need a checklist, written rules, and a process for auditing every screen against user intent.

Around **88% of users abandon an app after one poor experience**. Every vague icon, inconsistent label, and decorative element is friction that compounds into churn you cannot easily diagnose.

---

## Table of Contents

1. [Mindset: Users Are Not Tourists](#1-mindset-users-are-not-tourists)
2. [Part 1: Undo What AI Gave You](#2-part-1-undo-what-ai-gave-you)
3. [Part 2: Laws That Do Not Change](#3-part-2-laws-that-do-not-change)
4. [Onboarding & First-Run Experience](#4-onboarding--first-run-experience)
5. [Speed as an Aesthetic](#5-speed-as-an-aesthetic)
6. [Destructive Actions & Completion States](#6-destructive-actions--completion-states)
7. [Landing Pages](#7-landing-pages)
8. [How to Get Great UI Without Designing From Scratch](#8-how-to-get-great-ui-without-designing-from-scratch)
9. [Codebase Implementation Guide](#9-codebase-implementation-guide)
10. [Page Audit Checklist](#10-page-audit-checklist)
11. [AI Workflow for Existing Codebases](#11-ai-workflow-for-existing-codebases)

---

## 1. Mindset: Users Are Not Tourists

### You built the maze; they have never seen it

You spend months inside your app and know exactly where everything lives. New users do not. They bought your product to **get a job done**, not to explore your interface.

| Wrong assumption | Correct assumption |
|------------------|-------------------|
| "They'll figure it out" | Hold their hand more than feels necessary |
| "They care about our cool features" | They care about the **result** |
| "The UI is intuitive because I built it" | Every screen must answer one clear question |

### Instrument your product

Use session replay and analytics to watch real behavior:

- **Hotjar** — heatmaps, recordings
- **PostHog** — product analytics, funnels, session replay

You will discover users getting lost in places that feel obvious to you.

### The one-question rule

For every page, ask:

> **What is the one thing someone came here to do?**

Delete anything that is not that. If someone is on the Teams page, they do not need analytics widgets. If they want analytics, they go to the Analytics page.

---

## 2. Part 1: Undo What AI Gave You

AI optimizes for a screen that **looks finished**. You need a screen that **helps someone finish something**.

Ask of every element:

> **Does this help the user complete the job they came here for?**

If it is decoration, remove it.

Most fixes in this section take **under one hour** and deliver the highest ROI.

---

### 2.1 Kill Emojis; Use Real Icon Sets

**Problem:** Emojis as UI icons (🗑️ for delete, 🚀 for features) look amateur and clash with typography and color.

**Fix:** Install a professional icon library:

| Library | Notes |
|---------|-------|
| [Lucide](https://lucide.dev) | Clean, widely used, great React/Vue/SVG support |
| [Phosphor](https://phosphoricons.com) | Flexible weights, large set |
| Feather | Minimal, lightweight |

**Implementation in codebases:**

```bash
# React example
npm install lucide-react
```

```tsx
// Before (bad)
<button>🗑️ Delete</button>

// After (good)
import { Trash2 } from 'lucide-react';
<button><Trash2 size={16} /> Delete</button>
```

**Rules:**
- One icon set across the entire product
- Consistent size (typically 16px inline, 20–24px in nav)
- Same stroke weight everywhere
- Use icons for meaning, not decoration

---

### 2.2 Never Let AI Choose Your Colors

**Problem:** AI defaults to saturated blue or purple, then adds a second bright accent that fights the first. It reads as a template because it is one.

**Principle:** Pick a **restrained base** and **one accent**. Muted works. Trust comes from simplicity.

**Reference aesthetic:** Stripe, Attio, Airtable — mostly grayscale with color reserved for data and status.

| Use color for | Do NOT use color for |
|---------------|---------------------|
| Charts and data visualization | Every button |
| Status indicators (success, error, warning) | Decorative gradients |
| Primary call-to-action (one per view) | Background fills on cards |
| Priority / severity signals | Hero section "energy" |

**Dashboard rule:** A dashboard full of colored buttons looks cheap. The same dashboard with muted chrome and color only on charts/statuses looks expensive.

**Token structure (CSS variables):**

```css
:root {
  /* Base — near-white backgrounds, thin borders, no heavy shadows */
  --background: 0 0% 99%;
  --foreground: 0 0% 9%;
  --muted: 0 0% 96%;
  --muted-foreground: 0 0% 45%;
  --border: 0 0% 90%;

  /* One accent */
  --primary: 220 70% 50%;
  --primary-foreground: 0 0% 100%;

  /* Semantic only */
  --destructive: 0 72% 51%;
  --success: 142 60% 40%;
  --warning: 38 92% 50%;
}
```

**Button hierarchy example (Attio-style):**
- **Primary action** (Send Email) — filled, accent color
- **Secondary action** (Save Draft) — white/outline, visible but not dominant
- **Tertiary / destructive-adjacent** (Discard) — muted text, least visual weight

This guides the user toward completing the task without shouting.

---

### 2.3 Fix Consistency: Radius, Typography, Alignment

**Problem signs from vibe-coded UIs:**
- Mixed border radii (some buttons circular, some square)
- Inconsistent font sizes on the same hierarchy level
- Text not centered in buttons
- Feature cards with different dimensions and corner styles
- Clashing gradients and glow shadows

**Fix:** Define and enforce a design token scale.

| Token | Example values |
|-------|----------------|
| Border radius | `sm: 4px`, `md: 8px`, `lg: 12px` — pick ONE for buttons |
| Font sizes | `xs: 12`, `sm: 14`, `base: 16`, `lg: 18`, `xl: 24`, `2xl: 30` |
| Spacing | 4px base unit: 4, 8, 12, 16, 24, 32, 48, 64 |
| Button height | 32px (sm), 36px (md), 40px (lg) — one default |

**Before shipping any page, scan for:**
- [ ] All buttons use the same radius
- [ ] All buttons use the same height within a context
- [ ] Label text is vertically and horizontally centered
- [ ] Card components share one border/shadow/radius spec

---

### 2.4 Never Let AI Choose Your Layout

**Problem:** AI repeats the same patterns everywhere — four KPI cards on the dashboard, analytics page, and billing page — because it has no memory of what it already built.

**Fix:**
1. Audit every page for **repetition**
2. Assign **one purpose** per page
3. Remove cross-page widget duplication

**Card compression (highest ROI visual fix):**

AI is worst at dense, repeated components. Lists of cards accumulate buttons, chips, timestamps, and badges.

| Before (noisy) | After (quiet) |
|----------------|---------------|
| Row of 4 action buttons | Single `⋯` overflow menu |
| Text chips ("High Priority", "In Progress") | Icon + dot, or one colored priority indicator |
| Metadata scattered left | Key metric pushed right; secondary info collapsed |

**Example — task list row:**

```
Before:  Sarah Chen | Product Designer | In Progress | High Priority | Frontend | Bug | [Edit] [Delete] [Archive] [Share]

After:   Sarah Chen — Product Designer                    [⋯]
         ● High priority · In progress
```

Same information. One-third the noise. Color used once, meaningfully.

**Panel vs. modal rule:**
- 4 fields + empty slide-out panel → use a **centered modal** instead
- Match the container to the user's intent for that tab

---

### 2.5 AI Decorates; You Inform

Every bad pattern shares one trait: **visual material where information should be**.

| AI adds | You should add |
|---------|----------------|
| Emojis | Icons with meaning |
| Bright gradients | Muted surfaces |
| Repeated KPI cards | Purpose-specific content |
| Overloaded cards | Compressed, scannable rows |
| Empty side panels | Modals or removed chrome |
| Random colors | Semantic color only |

---

## 3. Part 2: Laws That Do Not Change

Interface trends rotate (skeuomorphic → flat → neumorphic → glass → whitespace/table-heavy). Underneath fashion sit principles that make software look **competent and trustworthy**.

---

### 3.1 Subtraction Over Addition

> When there is nothing left to remove, the design is perfect.

**Wrong instinct:** How much can I fit on this screen?  
**Right question:** How little can I get away with while still helping the user complete the job?

**Reference:** Basecamp — bold project titles, muted secondary text. That is the entire system. Reading it feels like reading a document.

**Process:**
1. Build the minimum viable screen
2. Delete elements one by one
3. Stop when removing something breaks the user's task

---

### 3.2 Write the Rules Down (Or You Will Break Them)

Inconsistency creates silent friction. Users will not report "delete vs. remove vs. trash" — they will just find your app annoying and leave.

**Create a `DESIGN_RULES.md` (or add to `CLAUDE.md` / `.cursor/rules/`):**

```markdown
# Design Rules

## Actions
- Destructive actions always use the verb "Delete" (never "Remove", "Trash", "Clear")
- Primary CTA label sitewide: "Start for free" (never mix with "Get started", "Try demo")

## Typography
- Page title: text-2xl font-semibold
- Section title: text-lg font-medium
- Body: text-sm text-muted-foreground
- Never use ALL CAPS for labels except overlines (e.g., "RECENT ACTIVITY")

## Spacing
- Section gap: 32px (space-8)
- Card padding: 16px (space-4)
- Inline element gap: 8px (space-2)

## Components
- Button radius: 8px (rounded-md)
- Card radius: 12px (rounded-lg)
- Border: 1px solid var(--border), no drop shadows on cards

## Color
- Color is semantic only: status, charts, one primary CTA per view
- No gradients on backgrounds
- No glow effects
```

**Paste this into your AI tool at the start of every UI session:**

> "These are the design rules. Follow them exactly. Do not invent new patterns."

AI forgets between sessions. Written rules are the fix.

---

### 3.3 Visual Hierarchy: Every Screen Is a Sentence

Something must be the **subject**. If everything is the same size and weight, the user must read everything to find what they need.

**Typical hierarchy:**

```
Welcome back                          ← greeting (medium, not bold)
Here's what's happening today         ← subtitle (muted)

RECENT ACTIVITY                       ← section overline (small caps, muted)
3 tasks completed this week           ← data point (default weight)
Updated 2 minutes ago                 ← metadata (smallest, most muted)
```

**Hierarchy tools:** size, weight, color, spacing.

**Practical rule:** Pick the **one element per screen that matters** and turn the volume down on everything else.

**Reference:** Airtable made spreadsheets scannable with color-coded statuses — color carries meaning, not decoration.

---

### 3.4 Start With Intent, Not Layout

**Wrong starting point:** "Where does the sidebar go? How many cards?"  
**Right starting point:** "What did the person arrive to do?"

**Example — CapForge login:**
- User intent: find and export leads
- Screen shows: database table, filters, export action, credits remaining
- Sidebar exists but the **primary surface** is the job

**Expansion rule:**

> Functionality expands when **intent** expands — never because you had empty space.

Do not add features because "it could be cool." Add them when you observe users needing to do more.

**Browsing vs. searching:**
- User knows what they want → search, direct navigation
- User is exploring → filters, categories, progressive disclosure

Design the container for the intent.

---

### 3.5 Design for Ugly Data, Not Demo Data

Your app looks great with tidy demo names and round numbers. Real data breaks it.

**Before launch, test with:**
- Long company names (50+ characters)
- Missing avatars / null states
- Zero results
- 10,000-row tables
- Slow network (3G throttling)

**Rules to decide now:**

| Scenario | Rule |
|----------|------|
| Long strings | Truncate with ellipsis after N characters (e.g., 24), show full on hover |
| Avatars | Icon on solid circle background so it survives any image |
| Empty lists | Deliberate empty state with one clear next action |
| Loading | Skeleton placeholders, not blank white |
| Errors | Human-readable message + recovery action |

---

## 4. Onboarding & First-Run Experience

The moment after trial start is when users are **least patient and most critical**. You have minutes, sometimes seconds.

### Do not force a tour

Forced tutorials feel like lectures. Users click through without reading to reach the app.

### Progressive onboarding wins

1. One obvious action that completes a real job
2. Reveal the next step when they finish the first
3. Celebrate completion

**First step design:**
- Primary card with subtle elevation (one shadow, not everywhere)
- Slightly more color than surroundings
- Label: "Start here"
- Progress bar showing finish line

**Celebration:**
- Confetti on first meaningful milestone (e.g., connected first account)
- Follow-up email: "Well done — here's the next step"

This works. It is not cheesy when tied to real accomplishment.

---

## 5. Speed as an Aesthetic

A 1-second delay can cost ~7% of conversions. Users do not always need faster code — they need **evidence that something is happening**.

### Perceived performance tactics

| Technique | When to use |
|-----------|-------------|
| Skeleton screens | Any data fetch > 200ms |
| Instant spinner (even native/OS) | Immediate feedback beats blank screen |
| Progress bar | Jobs > 1 second |
| Playful loader | Genuinely long jobs (PostHog-style mascot animations) |

**Key insight:** People tolerate 2 seconds of waiting. They do not tolerate 2 seconds of **blank screen**.

**iOS trick:** Some apps show Apple's system spinner immediately so users blame the device, not the app.

### Show value in numbers (retention feature)

B2B buyers must justify subscriptions to a boss or themselves. If the app does not show achievement, they assume they achieved nothing.

**Impact tracker example:**

```
Your impact this month
├── 30 hours saved
├── 100 tasks automated
├── 26 leads found
└── $18k revenue generated
```

Simple tallies work: "You've posted 25 times and saved 10 hours." People look at these. Milestone confetti reinforces the loop.

---

## 6. Destructive Actions & Completion States

### Ethical friction (not bad friction)

If delete instantly removes a row with no confirmation:
1. User cannot undo the mistake
2. User is uncertain it worked → refreshes → panic

For destructive, expensive, or final actions: **add confirmation**.

**Severity tiers:**

| Action severity | Pattern |
|-----------------|---------|
| Low (reversible) | Toast with 5-second undo |
| Medium | Modal: title + subtitle explaining consequences + Confirm/Cancel |
| High (irreversible) | Type-to-confirm (e.g., type server name to delete) |

**Example — good delete flow:**

```
1. User clicks Delete
2. Modal: "Delete Q3 Launch Plan"
   Subtitle: "This removes the project and all tasks. This can't be undone."
   Input: Type "delete" to confirm
3. On success: "Project deleted" + 5-second Undo button
```

Supabase requires typing the database name to delete. Tedious by design — it is the danger zone.

### Zeigarnik effect

Unfinished actions nag at people. If a task completes silently, users wonder "what just happened?"

**Completion states:**
- Checkmark animation after save
- Toast with clear outcome text
- Confetti for milestones (sparingly)

### Animation rules

> No animation unless it **tells the user something**.

| Allow | Avoid |
|-------|-------|
| Skeleton fade-in | Scroll-jacking |
| Confetti on milestone | Parallax |
| Confirm → success transition | Elements flying in from edges |
| Menu slide (stays open, moves sideways) | Decorative blur transitions |

### Load more vs. infinite scroll

Prefer **Load more** when it gives user control and lets them reach the footer. Infinite scroll hides footer content, feels endless, and can hurt performance.

---

## 7. Landing Pages

Landing pages are where vibe-coded products lose the most customers. The jump from generic to professional follows a known path.

### Escape template territory

| Kill | Replace with |
|------|----------------|
| Alternating text-left / image-right sections repeating down the page | Stacked hero with breathing room |
| Stock photos | Screenshots of your actual product |
| Mixed CTA copy ("Get started" / "Try demo" / "Free trial") | One label everywhere |
| Full dashboard screenshots | Cropped detail showing one proof point |
| Four identical feature cards | Bento grid ( varied cell sizes ) |

### Structure that converts

```
1. Hero — outcome headline + soft subtitle + single CTA pair
2. Three core features (not twenty)
3. Product in motion (short animation of real workflow)
4. Social proof (reviews, logos)
5. How it works (3 steps max)
6. Pricing (simple — one primary plan is fine)
7. FAQ
8. Final CTA
```

### Copy shift: feature → outcome

| Weak (descriptive) | Strong (promises outcome) |
|--------------------|---------------------------|
| Collect and analyze your data quickly | Turn your data into decisions |
| AI-powered content scheduling | From content to qualified pipeline |

This rewrite is often the **biggest single jump** on the page.

### Signals of depth (without complexity)

- Customer logo row
- Small badge ("New", "Beta")
- Mega menu in nav (quietly signals product depth)
- Cropped product screenshots with annotations

### Motion on landing pages

Precise motion only:
- Panel transitions that do not blur everything
- Menus that slide sideways instead of closing/reopening
- Feature animations showing real workflow

No custom illustration or 3D required — same components as your app.

---

## 8. How to Get Great UI Without Designing From Scratch

You do not need to invent aesthetics.

**Process:**
1. Screenshot apps and sites you admire (Attio, Stripe, Basecamp, Airtable, etc.)
2. Browse [Mobbin](https://mobbin.com) and [Dribbble](https://dribbble.com) for patterns
3. Keep a reference folder organized by pattern (dashboard, settings, empty state, landing hero)
4. Upload screenshots to Cursor/Lovable with prompt: **"Copy this design exactly. Match spacing, typography, and color restraint."**

Reference beats description every time.

---

## 9. Codebase Implementation Guide

This section maps video principles to concrete engineering work for **new pages** and **improving existing codebases**.

### 9.1 Foundation: Design Tokens First

Before touching individual pages, centralize tokens.

**Recommended stack:**
- CSS custom properties as source of truth
- Tailwind (or similar) mapped to those variables
- [shadcn/ui](https://ui.shadcn.com) for accessible primitives you own (not a black-box npm package)

```css
/* globals.css */
@layer base {
  :root {
    --radius: 0.5rem;
    /* ... color tokens ... */
  }
}
```

```js
// tailwind.config.js
theme: {
  extend: {
    borderRadius: {
      lg: 'var(--radius)',
      md: 'calc(var(--radius) - 2px)',
      sm: 'calc(var(--radius) - 4px)',
    },
    colors: {
      background: 'hsl(var(--background))',
      primary: 'hsl(var(--primary))',
      // ...
    },
  },
}
```

**Do not:**
- Hardcode hex colors in components
- Use JavaScript token objects that do not integrate with Tailwind
- Let each page define its own spacing

### 9.2 Component Library Structure

```
src/
├── components/
│   ├── ui/           # Primitives (Button, Card, Dialog, Input)
│   ├── patterns/     # Composed patterns (DataTable, PageHeader, EmptyState)
│   └── layout/       # AppShell, Sidebar, PageContainer
├── styles/
│   └── tokens.css
└── DESIGN_RULES.md
```

**Rules:**
- Pages compose from `patterns/` and `ui/` — never one-off buttons
- Business logic stays out of UI components
- One `Button` component with variants, not `PrimaryButton`, `BlueButton`, `ActionBtn`

### 9.3 Refactoring an Existing Codebase (Incremental)

Do not rewrite everything. Migrate page by page.

**Phase 1 — Stop the bleeding (Week 1)**
- [ ] Add `DESIGN_RULES.md` and AI context file
- [ ] Install icon library; find/replace emoji usage
- [ ] Define CSS token file; map Tailwind
- [ ] Add ESLint rule or code review checklist blocking hardcoded colors

**Phase 2 — Core primitives (Week 2–3)**
- [ ] Standardize Button, Input, Card, Dialog, Badge
- [ ] Delete duplicate components (grep for similar names)
- [ ] Add Storybook stories for stable components only

**Phase 3 — Page migration (Ongoing)**
- [ ] Pick highest-traffic page first
- [ ] Run page audit checklist (Section 10)
- [ ] One page per PR; visual diff review

**Phase 4 — Polish**
- [ ] Skeleton loaders on all async views
- [ ] Empty states on all lists
- [ ] Delete confirmations with undo
- [ ] Impact/ value metrics on dashboard

### 9.4 Page Template (New Screens)

Every new page should follow this structure:

```tsx
export function TeamsPage() {
  return (
    <PageContainer>
      <PageHeader
        title="Teams"
        description="Manage members and permissions"
        action={<Button>Add member</Button>}
      />
      <PageContent>
        {/* ONE primary intent — list, form, or detail */}
        <TeamsTable />
      </PageContent>
    </PageContainer>
  );
}
```

**Anti-patterns to reject in code review:**
- KPI cards on non-dashboard pages
- More than one primary button per view
- Inline styles for spacing/color
- `className="rounded-full"` on one button and `rounded-md` on another

### 9.5 Storybook (When APIs Stabilize)

Add Storybook **after** component APIs stabilize — not while structure changes daily.

Stories document:
- All button variants
- Empty / loading / error states
- Long text overflow behavior
- Mobile viewport

Optional: Chromatic for visual regression on PRs.

---

## 10. Page Audit Checklist

Run this on every page before merge.

### Intent
- [ ] Page answers exactly one user question
- [ ] Primary action is obvious within 3 seconds
- [ ] No duplicated widgets from other pages

### Visual cleanup
- [ ] No emojis as icons
- [ ] No glow shadows or decorative gradients
- [ ] One border radius system applied
- [ ] Typography follows hierarchy scale
- [ ] Color is semantic only

### Consistency
- [ ] Action verbs match `DESIGN_RULES.md`
- [ ] CTA labels match sitewide standard
- [ ] Icon set and sizes consistent
- [ ] Button heights consistent in context

### Data reality
- [ ] Tested with long strings
- [ ] Empty state exists
- [ ] Loading skeleton exists
- [ ] Error state with recovery action

### Interaction
- [ ] Destructive actions have confirmation
- [ ] Success states are visible (toast/tick)
- [ ] No animation without informational purpose

### Performance feel
- [ ] No blank screen during fetch
- [ ] Progress indicator for jobs > 1s

---

## 11. AI Workflow for Existing Codebases

When using Cursor, Claude Code, or similar tools on UI work:

### Before prompting

1. Attach `DESIGN_RULES.md`
2. Attach 2–3 reference screenshots
3. State the **one intent** for the page
4. Explicitly ban: emojis, gradients, glow, mixed radii, saturated purple/blue defaults

### Prompt template

```
Redesign [PageName] following DESIGN_RULES.md exactly.

User intent: [one sentence]

Reference: [attach screenshots]

Requirements:
- Muted palette, one accent, semantic color only
- Lucide icons, no emojis
- Single primary CTA: "[label]"
- Compress card rows: overflow menu instead of button rows
- Add skeleton loader and empty state
- Delete confirmation with undo toast

Do not add KPI cards, decorative gradients, or features not listed.
```

### Close the visual loop

AI cannot see what it builds. You must:

- Screenshot the result and paste back for correction
- Use browser MCP / Playwright for automated layout review
- Compare against reference screenshots side by side

### Prevent session drift

Without enforced boundaries, UI diverges every new chat. The written rules file is non-negotiable context for every session.

---

## Quick Reference: Bad → Good

| Signal | Vibe-coded | Professional |
|--------|------------|--------------|
| Icons | Emojis | Lucide / Phosphor |
| Color | Everywhere, saturated | Muted chrome, semantic accents |
| Layout | Same KPI cards on every page | One purpose per page |
| Cards | 6 buttons + 4 chips | Compressed row + overflow menu |
| Labels | Delete / Remove / Trash | One verb everywhere |
| Hierarchy | Everything same weight | One subject per screen |
| Landing | Alternating stock photo rows | Hero + cropped product shots |
| Loading | Blank white | Skeleton immediately |
| Delete | Instant vanish | Confirm → complete → undo |
| Onboarding | Forced 12-step tour | One action → next revealed |

---

## Further Resources

- **Video:** [How I Design SaaS That Looks EXPENSIVE](https://www.youtube.com/watch?v=9d5fRVDkzRI)
- **Playbook & prompts:** [roswell.dev](https://roswell.dev)
- **Reference products:** Attio, Stripe, Basecamp, Airtable, Paper Schedule (Oliver's prior startup)
- **Analytics:** Hotjar, PostHog
- **Inspiration:** Mobbin, Dribbble
- **Icons:** Lucide, Phosphor
- **Components:** shadcn/ui + Tailwind

---

*Document generated from video transcript and expanded with codebase implementation guidance.*
