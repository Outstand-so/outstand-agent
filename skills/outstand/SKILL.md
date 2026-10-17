---
name: outstand
description: >-
  Publish, schedule and analyze social posts on 12 networks through the Outstand MCP
  server. Create, edit, repost and delete posts on X, LinkedIn, Instagram, TikTok,
  YouTube, Facebook, Threads, Bluesky, Pinterest, Google Business Profile, Vimeo and
  Reddit, upload media, read and reply to comments, manage connected social accounts,
  and read post and account analytics. Use whenever "Outstand", "outstand.so", or the user's Outstand-connected
  social accounts, scheduled posts or social analytics come up.
---

# Outstand

Outstand is one API for publishing to 12 social networks. The `outstand` MCP server
bundled with this plugin acts inside the single Outstand organization the user picked
when signing in, and only on the social accounts connected to that organization.

## Before you act

1. **List accounts first.** Call `list_social_accounts` before composing a post.
   `accounts` on `create_post` takes the returned account IDs or exact usernames, not
   network names or nicknames. If the account the user means is not in the list, it is
   not connected; see [Connecting accounts](#connecting-accounts).
2. **Never invent IDs.** Every ID the server returns is an opaque string. Pass it back
   exactly as received; never build one or treat it as a number.
3. **Confirm before anything goes public or gets deleted.** `create_post` always queues
   publishing, either now or at `scheduledAt`; it has no draft mode. Show the user the
   final text, media, target accounts and time, and get an explicit yes first. Do the
   same for `update_post`, `repost_post`, `create_reply`, `delete_post`,
   `delete_remote_post`, `delete_media`, `delete_social_account`, `connect_bluesky`,
   `update_social_network` and `delete_social_network`. Tool annotations
   (`destructiveHint`) mark these. Deleting a post later does not undo the fact that
   people saw it.
4. **Schedule in UTC.** `scheduledAt` is an ISO 8601 UTC timestamp such as
   `2026-10-20T14:00:00Z`, at most 30 days ahead; a later value is rejected and nothing
   is created. Ask for the user's time zone if you do not know it, convert before
   sending, and show times back in their zone. Omit `scheduledAt` to publish now.

## Choosing a tool

| You want to | Use |
|---|---|
| See connected accounts | `list_social_accounts`, `get_social_account`, or the `outstand://social-accounts` resource |
| Publish now or schedule | `create_post` |
| See what is scheduled or published | `list_posts` (filter by `social_account_id`, `scheduled_after`/`scheduled_before`, `created_after`/`created_before`; `limit` up to 100 with `offset`), `get_post` |
| Edit or reschedule a post that has not gone out | `update_post` |
| Hold a queued post back, or release it | `update_post` with `isDraft: true` or `false` |
| Cancel a scheduled post | `delete_post` |
| Take a published post down from the networks | `delete_remote_post` |
| Reshare or quote a published post | `repost_post` (X, Bluesky, LinkedIn, Threads) |
| Read and answer comments | `get_replies`, `create_reply` |
| Post performance | `get_post_analytics` |
| Follower and engagement trends | `get_account_metrics` (defaults to the last 30 days) |
| Upload a file | `request_media_upload_url`, then `confirm_media_upload` |
| Plan limits and usage this billing period | `get_account_usage` |
| Which networks support what | the `outstand://supported-networks` resource |

## Posts

- Use `content` for a text-only post. Use `containers` for media or threads: each
  container has optional `content` and `media` entries (`url`, `filename`, optional
  `altText`), and later containers become replies in a thread where the network
  supports it.
- `update_post` works only while the post has not gone out to any account. Sending
  `content` or `containers` replaces the whole body; platform settings merge per field.
  Target accounts cannot change, so delete the post and create a new one instead.
- `delete_post` removes the Outstand record and cancels queued publishing. It does not
  touch content that is already live; `delete_remote_post` does that. TikTok and
  Instagram do not support remote deletion, so those accounts fail while the others are
  still attempted.

## Media

1. `request_media_upload_url` with `filename` returns `media_id`, `upload_url` and
   `content_type`.
2. PUT the file bytes to `upload_url` with that `Content-Type` within one hour.
3. `confirm_media_upload` with `media_id` returns the public URL. Use it in a post only
   after this step.

A file that already has a public URL can go straight into `containers[].media[].url`.
Confirmed uploads are kept for 60 days. `processMedia: true` on `create_post` re-encodes
video for each network and is billed per rendition, so ask before turning it on.

## Network rules

- **TikTok:** defaults to `tiktok.postMode: MEDIA_UPLOAD`, which lands as a draft in the
  user's TikTok inbox. `DIRECT_POST` requires `tiktok.privacyLevel`.
- **Pinterest:** requires `pinterest.board_id` from `list_pinterest_boards`.
- **Reddit:** requires `reddit.subreddit` and `reddit.title`; set `reddit.url` for a
  link post.
- **Threads:** rejects any container over 500 characters, thread replies included,
  rather than truncating. Split long text before sending.
- **Facebook:** `facebook.publishAsStory` (one image or video, no caption, replies or
  mentions) and `facebook.publishAsReel` (one video) cannot be combined.
- **Instagram:** `instagram.publishAsStory`, `instagram.reelCoverUrl`,
  `instagram.isAiGenerated` and `instagram.trialReel`.
- **YouTube:** `youtube.containsSyntheticMedia` for altered or synthetic content, and
  `youtube.thumbnailUrl` for a custom thumbnail.

The tool input schemas carry the full set of per-network options.

## Analytics

`get_post_analytics` returns per-account metrics plus totals. Before reporting numbers,
check each account's `metrics_error` and `platform_specific.unavailable_fields`: a
value the network does not provide is `null`, a measured zero is `0`, and totals can be
partial. Instagram Story insights expire after about 24 hours and then come back with
`metrics_error.code: metrics_expired`.

## Connecting accounts

Accounts are usually connected in the Outstand dashboard at https://www.outstand.so/app.
From here:

- `get_auth_url` returns a link the user opens in the browser to connect an account on
  a configured network.
- `connect_bluesky` connects Bluesky with a handle and an app password. Only an app
  password works; never ask for the user's main Bluesky password.
- X, Google Business Profile, Vimeo and Reddit need the organization's own app
  credentials before an account can connect. `create_social_network` stores them, but
  suggest the Social Networks page of the dashboard first so the client secret is not
  pasted into the conversation.

## Not supported through this server

Paid ad campaigns, direct messages, hiding or deleting comments, importing past posts,
webhooks, and creating or revoking API keys. Say so plainly rather than reaching for a
different tool. Some of these are in the REST API.

## Building against Outstand in code

When the user is writing an integration rather than managing their own accounts, the
MCP tools are still the quickest way to inspect real data, but the code should call the
REST API:

- Base URL `https://api.outstand.so`, authenticated with `Authorization: Bearer <key>`.
  API keys start with `ost_` and are created in the dashboard.
- Docs: https://www.outstand.so/docs, with the full text for agents at
  https://www.outstand.so/docs/llms-full.txt.
