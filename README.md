# Outstand plugin for Grok Build

Connect Grok Build to [Outstand](https://www.outstand.so), one API for publishing to
X, LinkedIn, Instagram, TikTok, YouTube, Facebook, Threads, Bluesky, Pinterest, Google
Business Profile, Vimeo and Reddit. Publish and schedule posts, reply to comments,
manage connected accounts and media, and read post and account analytics through the
hosted Outstand MCP server.

## Installation

In Grok Build, open `/plugin`, search for **Outstand**, and install.

On first connection, Grok opens Outstand sign-in in your browser. Sign in (or create an
account), choose the organization Grok should act on, and approve. No API key is needed
and nothing should be pasted into chat.

You need an Outstand account with at least one connected social account. Usage is
subject to your [Outstand plan](https://www.outstand.so/#pricing).

## What you get

- **MCP server** `outstand` at `https://mcp.outstand.so/mcp` (Streamable HTTP, OAuth 2.1).
- **Skill** `outstand`: how to use the tools well (list accounts before posting, confirm
  before publishing or deleting, schedule in UTC within 30 days, the two-step media
  upload, per-network requirements) plus pointers to the REST API docs.

### Tools

| Area | Tools |
|---|---|
| Posts | `create_post`, `update_post`, `list_posts`, `get_post`, `delete_post`, `delete_remote_post`, `repost_post` |
| Comments | `get_replies`, `create_reply` |
| Analytics | `get_post_analytics`, `get_account_metrics`, `get_account_usage` |
| Social accounts | `list_social_accounts`, `get_social_account`, `delete_social_account`, `connect_bluesky`, `list_pinterest_boards` |
| Network credentials | `create_social_network`, `list_social_networks`, `get_social_network`, `update_social_network`, `delete_social_network`, `get_auth_url` |
| Media | `request_media_upload_url`, `confirm_media_upload`, `list_media`, `get_media`, `delete_media` |

Resources: `outstand://social-accounts` and `outstand://supported-networks`.

Every tool carries MCP annotations (`readOnlyHint`, `destructiveHint`, `openWorldHint`),
so Grok can tell a lookup from a publish or a deletion before calling it.

## Example prompts

- "Show my connected social accounts."
- "What posts do I have scheduled for next week?"
- "Schedule this announcement to LinkedIn and X for Tuesday at 9am my time."
- "Summarize how my last five posts performed."
- "Reply to the latest comments on yesterday's Instagram post."

## Authentication and network

Authentication is OAuth 2.1 authorization code with PKCE and dynamic client
registration; Grok handles the flow. Access is limited to the one organization chosen
on the consent screen and can be revoked from the Outstand dashboard.

Network endpoints:

- `https://mcp.outstand.so/mcp`: hosted MCP (Streamable HTTP)
- `https://mcp.outstand.so/.well-known/oauth-protected-resource/mcp`: protected-resource metadata (RFC 9728)
- `https://www.outstand.so/app/.well-known/oauth-authorization-server`: authorization server metadata
- `https://www.outstand.so/app/oauth/authorize`, `/app/api/auth/mcp/token`, `/app/api/auth/mcp/register`: OAuth 2.1 + DCR
- `https://www.outstand.so/app`: human sign-in, organization selection and consent
- `https://*.r2.cloudflarestorage.com`: presigned media upload URLs returned by `request_media_upload_url`, used only when uploading a file
- `https://media.outstand.so`: public URLs of confirmed media

Credentials: an Outstand account. The MCP server issues an access token scoped to the
chosen organization and Grok sends it as `Authorization: Bearer` on `/mcp`. The plugin
stores no API key and requests no filesystem or shell access.

## Headless use with an API key

For clients without OAuth support, the same server accepts an Outstand API key (`ost_`
prefix) as a Bearer token. Keys are created in the
[dashboard](https://www.outstand.so/app) and can use every tool in their organization,
so create a separate key per client. See the
[MCP setup docs](https://www.outstand.so/docs/mcp/setup).

## License

MIT for the files in this plugin; see [LICENSE](LICENSE). Use of the hosted MCP server is governed by the
[Outstand terms](https://www.outstand.so/terms) and
[privacy policy](https://www.outstand.so/privacy).
