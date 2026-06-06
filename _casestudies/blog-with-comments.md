---
layout: page
title: Building a Blog with Comments and Multiple Authors
orderfield: 1
---

## Context

A freelance writer wanted a personal blog to publish long-form articles. After six months, readers started emailing replies. She wanted comments directly on the page — no social login, no tracking, no noise. Later, she invited two friends to co-publish.

**Constraints:**
- Small traffic (~200 visitors/day)
- No existing backend
- Budget: zero
- Must avoid spam without a full moderation dashboard

---

## Requirements

1. Publish articles (authors only)
2. Readers can leave comments (name + text, no account required)
3. Comments are visible to all readers after approval
4. Each author receives an email notification for comments on their own articles

---

## Data Model

```
Author
  id            integer, primary key
  name          text
  email         text, unique
  password_hash text

Article
  id            integer, primary key
  author_id     integer, foreign key → Author
  slug          text, unique
  title         text
  body          text
  published_at  datetime

Comment
  id            integer, primary key
  article_id    integer, foreign key → Article
  author_name   text
  body          text
  approved      boolean, default false
  created_at    datetime
```

A boolean `approved` flag on the comment row is the minimum solution. A separate moderation queue table would be premature at this scale.

---

## Approval Flow

No dashboard was built. The article's author receives an email with two links:

- `Approve: /admin/comments/42/approve?token=<hmac>`
- `Delete: /admin/comments/42/delete?token=<hmac>`

The token is an HMAC scoped to the comment, article, and author:

```
HMAC(comment_id + article_id + author_id, secret)
```

The server verifies that the `author_id` encoded in the token matches the article's `author_id` before taking any action. One click, no login required. One extra check, no new infrastructure.

---

## Authentication

A simple session-based login: email and password. On login, `author_id` is stored in the session. Every admin action verifies that the acting author owns the target article.

No roles table. No permissions matrix. Three authors, one level of access. A role system would be premature.

---

## Spam Prevention

A honeypot field — a hidden input that humans never fill but bots do:

```html
<input name="website" style="display:none">
```

If the field arrives non-empty, the comment is silently discarded. No CAPTCHA, no rate limiting. Revisit if spam volume grows.

---

## What Was Not Built

| Idea | Reason skipped |
|---|---|
| Moderation dashboard | Email link flow is sufficient |
| Nested replies | Not requested |
| Edit/delete by commenter | Not requested |
| Author profile pages | Not requested |
| Co-authorship on a single article | Not requested |
| Admin super-user role | All three authors are equal |
| Rich text in comments | Security risk, not requested |

---

## Outcome

The system went live in two weeks. Authors approve comments via email. No spam has passed the honeypot. No dashboard has ever been needed.

Adding multiple authors required one new table, one foreign key, and one scoped token check. The rest of the system was untouched — the expected outcome when the original model was not over-designed.

---

## Lesson

The temptation was to build a proper CMS with a full moderation UI, user accounts, and notifications. The actual need was: let readers say something, let the right author decide if it stays. The minimum solution fit the real need exactly, and extending it cost almost nothing.
