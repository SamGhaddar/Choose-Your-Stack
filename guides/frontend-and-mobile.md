# Frontend and Mobile Guide

Last updated: 2026-11-17

Use a static site unless the pages need logged-in, changing data; then use React with Next.js. The front end is the dining room: everything the user sees and taps.

## How to use this doc

Attach it at the start of a Claude session, or reference it from a project's `CLAUDE.md`. Use the prompt in the [README](../README.md).

## Quick picker

| You are building | Pick | Why | Avoid when |
| --- | --- | --- | --- |
| Marketing site, portfolio, client site | Astro, or plain HTML, CSS and JS | Fast, cheap to host, almost no JavaScript shipped | Pages depend on per-user data |
| Web app with accounts and dashboards | React with Next.js | Largest ecosystem and hiring pool | The site is purely static |
| iOS and Android app, one team | React Native with Expo (or Flutter) | One codebase for both stores | You need heavy native features such as AR |
| Internal admin panel | A low-code tool or admin framework | Built in days, not weeks | It becomes customer-facing |

Vue is a gentler learning curve; Angular suits large enterprise teams. For styling, Tailwind is the common default and Bootstrap gives prebuilt components fast.
