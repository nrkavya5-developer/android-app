# Shaale-Vikas | Rural School Bridge — Documentation

## Overview

Shaale-Vikas is a prototype for a transparent bridge between rural government schools in India and their global alumni network. The application enables headmasters to post infrastructure needs and alumni to donate directly to verified causes, with full visibility into fund utilization through impact reports.

The current codebase is a single-page HTML application that demonstrates the complete user flow and interface design using reactive, in-memory state management.

---

## Architecture

### Single-Page Application Pattern

The entire application lives in one `index.html` file. Screen transitions are handled entirely on the client side using Alpine.js, with no page reloads or server calls.

**Core State Object** (`function app()` in the inline `<script>`):

| Property | Type | Purpose |
|----------|------|---------|
| `currentScreen` | String | Controls which screen template is rendered — one of `Landing`, `Login`, `Dash`, `Needs`, `NeedDetail`, `Impact`, `Donors`, `Settings` |
| `currentRole` | String | Tracks authenticated user role — `Admin` or `Alumni` |
| `isDark` | Boolean | Dark mode toggle state |
| `needs` | Array | In-memory list of school needs/requests |
| `impacts` | Array | In-memory list of completed projects with before/after images |
| `donors` | Array | In-memory leaderboard of donors |
| `search` | String | Filter text for the needs search bar |
| `toast` | Object | Controls the temporary toast notification |

### Screen Rendering

Each screen is wrapped in a `<template x-if="currentScreen === 'ScreenName'">` block. Alpine.js's reactivity system hides and shows the relevant markup based on the `currentScreen` property. Navigation updates this property, which triggers Alpine.js to re-evaluate all `x-if` and `x-show` bindings.

### Navigation Bar

A fixed bottom navigation bar (visible when authenticated) provides access to:
- **Dash** (Admin only) — School Dashboard
- **Needs** — Browse active requests
- **Impact** — Completed project reports
- **Donors** — Hall of Fame leaderboard
- **Settings** — Account and preferences

A floating action button (FAB) appears for Admin users on the Needs screen to post new requests.

---

## User Roles

### Admin / Headmaster

- Login via "Login as Headmaster" button on the landing screen
- Sees the **Dashboard** as the home screen after login
- Can **post new school needs** via the FAB + bottom-sheet form
- Can **add impact reports** on the Impact screen
- Views **funds received** and **problems solved** metrics

### Alumni

- Login via "Login as Alumni" button on the landing screen
- Sees the **Needs Feed** as the home screen after login
- Can **search** and browse needs across all schools
- Can **view need details** with full funding progress
- Can **donate** via the bottom-sheet donation modal
- Appears in the **Hall of Fame** leaderboard

---

## Navigation and Screens

### Landing Screen

The entry point. Two role-based login buttons. The title "Shaale-Vikas" and tagline set the mission context. Selecting a role transitions to the Login screen.

### Login Screen

A context-aware registration form:
- **Admin (Headmaster):** Fields include Full Name, UDISE Code, Email, Password
- **Alumni:** Fields include Full Name, Passing Year, Email, Password

The "Join Network" button simulates authentication and routes the user to their respective home screen.

### Dashboard (Admin Only)

Displays key metrics in two cards:
- **Funds Received** — Total contributions (sample: ₹1.4L)
- **Problems Solved** — Completed initiatives count

Below the metrics, a **Recent Activity** feed shows individual donation entries with donor name, cause, and amount.

### Needs Feed

- **For Alumni:** A search bar filters needs by title or school name
- **For Admin:** Lists the school's own active requests

Each need card displays:
- Background image with gradient overlay
- School name badge and URGENT flag (if marked urgent)
- Title and truncated description
- Progress bar showing percentage funded vs target amount

### Need Detail Screen

Full-screen view of a selected need:
- Hero image with gradient overlay and back button
- Problem description
- "Contact School" shortcut card
- Funding progress card with raised amount, goal amount, and progress bar
- **Donate now** button (visible only to Alumni)

The donate button opens a bottom-sheet modal where the user enters an amount. On confirmation, the collected amount updates in real time.

### Impact Screen

Lists completed projects, each with:
- Project title and school name
- **Before** and **After** photo comparison (two-image grid)
- Description or testimonial

Admin users see an "+ Add Impact" button to submit new completions.

### Donors Screen (Hall of Fame)

A hero banner with the "Hall of Fame" title. Below it, a ranked list of donors showing:
- Rank number (1, 2, 3...)
- User avatar placeholder
- Donor name and supported cause
- Contribution amount badge

### Settings Screen

Two sections:
1. **Profile Information** — Avatar, name, role description, and edit profile action
2. **App Preferences** — Dark Mode toggle (switch component) and Notifications toggle

A **Secure Logout** button at the bottom resets the session and returns to the Landing screen.

### Dark Mode

Toggled from the Settings screen. The `isDark` property applies conditional classes to the `<body>` element and all child components:
- Background: `bg-[#0F172A]` (dark) / `bg-[#F8FAFC]` (light)
- Cards: `bg-[#1E293B]` (dark) / `bg-white` (light)
- Navigation bar: translucent backdrop blur with role-specific tinting

---

## Data Flow

### In-Memory Stores

All application data is stored in plain JavaScript arrays within the Alpine.js component state. There is no backend or persistent storage in the prototype.

**Needs Store** (`app().needs`):
```javascript
[
  { id, school, title, desc, target, collected, urgent, image }
]
```

**Impacts Store** (`app().impacts`):
```javascript
[
  { id, title, school, desc, before, after }
]
```

**Donors Store** (`app().donors`):
```javascript
[
  { name, supported, amount }
]
```

### User Flows

#### Post a Need (Admin)
1. Admin presses the FAB (+) on the Needs screen
2. A bottom-sheet modal appears with form fields: photo upload, title, description, target amount
3. On submit, the new need is prepended to the `needs` array with an auto-generated ID
4. A toast notification confirms: "Problem Published Globally!"

#### Donate (Alumni)
1. Alumni taps a need card on the Needs Feed
2. The Need Detail screen shows full information
3. Alumni taps "Donate to this Cause"
4. A bottom-sheet modal opens with an amount input
5. On confirm, the `activeNeed.collected` value is updated instantly
6. A toast notification confirms: "Awesome! You donated ₹X"

#### Search Needs (Alumni)
The `filteredNeeds` computed property filters the `needs` array:
```javascript
get filteredNeeds() {
  if (!this.search) return this.needs;
  return this.needs.filter(n =>
    n.title.toLowerCase().includes(this.search.toLowerCase()) ||
    n.school.toLowerCase().includes(this.search.toLowerCase())
  );
}
```

#### Logout
The `logout()` method resets `currentRole` to `'None'` and `currentScreen` to `'Landing'`, returning the user to the initial state. A toast confirms the action.

---

## Customization Guide

### Changing Branding

- **Title and tagline:** Update the text inside the Landing screen's `<h1>` and `<p>` tags (lines 34-35)
- **Favicon:** Add a `<link rel="icon">` in the `<head>`
- **Color scheme:** Tailwind utility classes are used throughout. Replace `blue-600`, `emerald-500`, `amber-400`, etc., with preferred color tokens

### Adding Screens

1. Add a new `<template x-if="currentScreen === 'NewScreen'">` block inside the main container
2. Add a corresponding entry in the `app()` return object's `currentScreen` default (not strictly required, but good practice)
3. Add a navigation button in the bottom nav bar referencing the new screen name

### Modifying Sample Data

Edit the `needs`, `impacts`, and `donors` arrays in the `app()` function (lines 359-370). Replace image URLs, text values, and amounts with real or updated placeholder data.

### Disabling Authentication

To bypass the login flow for testing, set `currentRole` to `'Admin'` or `'Alumni'` directly in the `app()` return object, and change `currentScreen` to `'Dash'` or `'Needs'` as desired.

---

## License

This project is open source and available under the [MIT License](../LICENSE).
