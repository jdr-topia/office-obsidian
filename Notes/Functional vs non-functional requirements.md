---
created: 2026-04-01
tags: [concept]
related: [ [[Requirement analysis in frontend design]], [[Performance requirements]], [[Scalability in frontend systems]], [[Frontend system design MOC]] ]
status: active
---

# Functional vs non-functional requirements

## Overview

Every system has two fundamental types of requirements. Understanding the difference is the foundation of all system design interviews and real-world architecture decisions.

---

## Functional Requirements

**What the system does.** These describe specific behaviours, features, and outputs of the system.

Think of functional requirements as: *"The system shall do X."*

### Examples in a frontend context

- The user can log in using email and password.
- The dashboard displays a table of orders with pagination.
- Users can filter orders by date range, status, and product.
- An admin can export records to Excel.
- A notification bell shows unread alerts in real-time.

### How to extract them

1. **Map user journeys**: Walk through every path a user can take in the app.
2. **Identify entities**: What data objects exist? (User, Order, Product).
3. **Identify CRUD operations**: What can be created, read, updated, deleted?
4. **Identify roles**: What can an Admin do vs. a regular User?

---

## Non-Functional Requirements (NFRs)

**How well the system does it.** These describe qualities, constraints, and standards the system must meet. They are often called "quality attributes."

Think of non-functional requirements as: *"The system shall do X **within constraint Y**."*

---

### The 7 Core NFRs for Frontend

#### 1. Performance
The system must respond quickly and not block the UI.

> **Target Example**: Page must achieve LCP < 2.5s on a 4G mobile connection.

→ See: [[Performance requirements]]

#### 2. Scalability
The system must handle growth in users, data, or team size.

> **Target Example**: The UI must remain stable with 10,000 rows in a table without lagging.

→ See: [[Scalability in frontend systems]]

#### 3. Availability
The system must be reliably accessible.

> **Target Example**: 99.9% uptime. The UI should show a fallback state if the API is down, not a blank screen.

#### 4. Accessibility (a11y)
The system must be usable by people with disabilities.

> **Target Example**: Must comply with WCAG 2.1 AA. All interactive elements must be keyboard-navigable.

→ See: [[Accessibility requirements]]

#### 5. Security
The system must protect user data.

> **Target Example**: All user input must be sanitized. Auth tokens must be stored in HttpOnly cookies.

#### 6. Internationalization (i18n)
The system must support multiple languages and locales.

> **Target Example**: The app must support English, Arabic (RTL), and French within 3 months of launch.

→ See: [[Internationalization (i18n)]]

#### 7. SEO
The system must be discoverable by search engines.

> **Target Example**: All public-facing pages must have server-rendered HTML for crawlers.

→ See: [[SEO considerations in frontend design]]

---

## Functional vs Non-Functional: Side-by-Side

| Dimension | Functional | Non-Functional |
| :--- | :--- | :--- |
| **Question answered** | What does it do? | How well does it do it? |
| **Testing** | Feature tests, E2E tests | Load tests, Lighthouse audits |
| **Examples** | Login form, Order table | LCP < 2.5s, WCAG AA |
| **Changes when?** | Product requirements change | Quality bar changes |
| **Drives decisions on** | Features, UI, APIs | Architecture, stack, infrastructure |

---

## Why NFRs Are Often Neglected and Why That's Dangerous

NFRs are easy to deprioritize because they don't show up in a standard sprint backlog. Functional features have visible demos; NFRs do not.

**Real-world consequences of ignoring NFRs:**
- Ignoring **performance** → Users experience sluggish UI, high bounce rate.
- Ignoring **accessibility** → Legal liability (ADA compliance lawsuits are real).
- Ignoring **i18n** upfront → Complete rewrite of string management when expanding globally.
- Ignoring **scalability** → App works in staging with 100 rows but crashes in production with 100,000.

> **Rule of thumb**: NFRs are constraints that determine your architecture. Identify them in the first design session, not as an afterthought.
