# [Feature name] — setup guide

<!-- Copy to docs/<feature>-setup.md. Audience: the merchant. Plain language, admin paths in bold, exact keys in code. Write it from the setup that worked on the client dev store, then follow it on prod to confirm it (docs/development-workflow.md, Stores and environments). Delete this comment. -->

One or two sentences: what the feature does for customers, where it shows up, and what the merchant controls.

## 1. Create the metaobject definition

<!-- Skip or remove sections that don't apply. Definitions are store data: the theme can't create them. -->

In **Settings → Custom data → Metaobjects → Add definition**, create **[Name]** with type **`[type]`**. Set **Storefront access** to **Read**.

| Field name | Key | Type | Notes |
|---|---|---|---|
| | `key` | | Required? Fallback when blank? Accepted values? |

The keys must match exactly. Set each entry's status to **Active**, because draft entries can't be read on the storefront.

## 2. Create the metafield

In **Settings → Custom data → [Products/Collections/…] → Add definition**, create **[Name]** with namespace and key **`namespace.key`**. Shopify pre-fills the namespace as `custom`; replace it if this guide uses a different one. Enable storefront read access.

## 3. Add content

Where to create entries and how to assign them. Say what happens when nothing is assigned (usually: the page renders exactly as before).

## 4. Theme controls

Where the settings live (**Theme settings → …** or the section in the theme editor), what each one does, and its default.

## Verification

A short checklist the merchant or reviewer can follow on the preview theme: which pages to open, at which widths, and what to expect, including the "nothing assigned" case.
