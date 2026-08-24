---
name: frontend-patterns
description: "Build React components and pages in the App Router. Use when creating or changing a component, a page, a form, or a data fetch, when deciding what runs on the server versus the client, when handling loading and error states, or when something re-renders wrongly or state is out of sync. Triggers include 'build the UI', 'component', 'form', 'data fetching', 'use client', 'this re-renders', 'state', 'loading state', 'error boundary'. Covers component structure, the server and client boundary, forms with actions, and the four async states. Do NOT use for colors, spacing, or variants; that is design-system."
---

# Frontend patterns

## The server and client boundary

Server Components by default. `"use client"` only where interactivity or a browser API demands it, and as deep in the tree as possible.

The common mistake is marking a page client because one button needs a handler. Extract the button. Everything above it stays on the server, which means it stays out of the bundle.

What crosses the boundary is data, not capability. A client component receives a DTO. It does not receive a database handle, a session object, or a function that closes over a secret.

## Data fetching

Fetch in the Server Component that renders the data. Not in a parent that drills it down four levels, and not in an effect.

- Parallel fetches with `Promise.all` when independent. Sequential awaits are a waterfall nobody asked for.
- `Suspense` boundaries around slow sections so the rest of the page is not held hostage.
- Never fetch your own API route from your own server. Call the function.

Client-side fetching is for data that changes after render in response to user action. If it is available at render time, it belongs on the server.

## Forms

Server Actions with progressive enhancement. The form works before the JavaScript loads, then improves.

- `useActionState` for pending and error states.
- Client validation mirrors server validation and never replaces it. It is a courtesy, not a check.
- Field-level errors returned from the action, associated with the field via `aria-describedby`.
- The submit control is disabled while pending, and it says what it is doing.
- After success, revalidate the affected path. Do not manually patch client state to match what you assume the server now holds.

## The four states

Every async surface handles loading, empty, error, and success. Missing the empty state is the default bug, and it is the first thing a new user sees.

- `loading.tsx` per route for route-level loading.
- `error.tsx` per route, not one at the root. A root-level boundary turns a small failure into a blank application.
- Skeletons match the shape they replace, which prevents layout shift.

## State

- Derive, do not duplicate. If it can be computed from props, compute it. Copying props into state creates two sources of truth and one of them goes stale.
- URL search params for state that should survive reload, sharing, and the back button. Filters, tabs, pagination. This is the most underused option available.
- `useEffect` synchronizes with something outside React. It is not for transforming data and not for fetching in a server-rendered app. An effect that sets state from props is almost always a mistake.
- Keys are stable identifiers from the data. Never the array index for anything that reorders, filters, or deletes.

## Component structure

- Props are a typed interface, no `any`, no overly broad object types.
- A component doing two unrelated things is two components.
- Colocate: the component, its test, and its story live together.
- Composition over configuration. A component with nine boolean props is several components hiding.

## Optimistic updates

Only where the failure path is genuinely handled. An optimistic update that silently reverts is worse than a spinner, because the user believes it worked.

## Review checks

- `"use client"` higher in the tree than necessary
- Data fetched in an effect that could be fetched on the server
- Missing empty state
- Root-level error boundary as the only one
- Props copied into state
- Index used as a key on a mutable list
- Client validation with no server counterpart
