# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**PMHVibes Dashboard** — a single-file internal business dashboard for an e-commerce/POD operation. There is no build system; open `index.html` directly in a browser.

## Architecture

Everything lives in one file: `index.html`. It contains inline CSS (`<style>`), inline HTML structure, and inline JavaScript (`<script>`). There are no external JS files, no npm packages, and no server-side code. The only external dependency is Google Fonts (loaded via CDN).

### Data Layer

- **Source of truth**: Google Sheets (read via the Sheets REST API using a user-supplied API key)
- **Write-backs**: Either via a Google Apps Script web app URL (preferred) or directly via the Sheets API
- **Local storage keys**:
  - `pod_settings` — `{ sheetId, apiKey, scriptUrl, scriptToken }`
  - `pod_accounts` — array of user account objects (username, password, role, perms)
  - `pod_session` — currently logged-in username
  - `pod_templates` — saved email reply templates

### Expected Google Sheet columns (Sheet1, row 1 = headers)
`id`, `from_name`, `from_email`, `account`, `subject`, `body`, `type`, `draft_reply`, `status`, `thread_id`, `timestamp`

Status column is column J (row index used for write-backs). Draft column is column I.

### Auth

Client-side only. Credentials are compared in plaintext from localStorage. Default account: `admin` / `pod2024!`. Roles are `admin` (full access) or `custom` (per-section permissions stored in the account object's `perms` array).

### Key JS globals

- `emails` — array of all loaded email objects (reversed from sheet order)
- `currentFilter` — active sidebar folder/filter (`all`, `unread`, `tracking`, `sent`, `trash`, `cancel`, `update`, `refund`, `general`)
- `currentPage` — pagination page number
- `selectedIds` — `Set` of indices into `emails` for bulk operations

### Pages

Active pages: **Email Support**, **Email Templates**. Pages for Facebook Support, Orders & Profit, Ad Campaigns, and Product Spy are stubbed with "coming soon" placeholders.

### Email flow

1. Sheets → `loadFromSheets()` → populates `emails[]` → `renderEmails()`
2. Opening an email: `openModalByIndex(idx)` strips HTML from the stored draft for plain-text editing
3. Saving draft: converts plain-text back to `<p><br>` HTML, calls `updateSheetDraft()`
4. Sending: sets `status = 'sent'`, calls `updateSheetStatus()`, re-renders
5. Thread grouping: emails sharing `threadId` are collapsed to one row; clicking opens `openThread()`

### Apps Script integration

The Apps Script URL is called with GET params `?row=N&status=X&token=T` (for status) or `?row=N&draft=X&token=T` (for draft). It must match `SECRET_TOKEN` in the script.

## Development Rules

1. **Balanced braces**: After every edit to the `<script>` block, verify `{` and `}` counts are equal before finishing.
2. **Preserve existing functionality**: When adding new features, test that existing email loading, filtering, sending, draft saving, auth, and thread grouping still work as before.
3. **Change log**: At the end of every editing session, note what was changed (which function or HTML section, and what was added/modified) as a comment or in this file under a `## Changelog` section.
4. **Single file**: Keep all CSS, HTML, and JavaScript in `index.html`. Do not create separate `.js`, `.css`, or template files.
