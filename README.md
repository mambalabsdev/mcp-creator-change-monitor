# Influencer Change Monitor MCP Server

[![Smithery](https://smithery.ai/badge/mambabuilt/mcp-creator-change-monitor)](https://smithery.ai/servers/mambabuilt/mcp-creator-change-monitor) [![Glama score](https://glama.ai/mcp/servers/mambalabsdev/mcp-creator-change-monitor/badges/score.svg)](https://glama.ai/mcp/servers/mambalabsdev/mcp-creator-change-monitor) [![MCP Registry](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fregistry.modelcontextprotocol.io%2Fv0%2Fservers%3Fsearch%3Dcom.mambabuilt%252Fmcp-creator-change-monitor%26limit%3D1&query=%24.servers%5B0%5D._meta%5B%22io.modelcontextprotocol.registry%2Fofficial%22%5D.status&label=mcp%20registry&color=blue)](https://registry.modelcontextprotocol.io/v0/servers?search=com.mambabuilt/mcp-creator-change-monitor&limit=1) [![npm version](https://img.shields.io/npm/v/@mambalabsdev/mcp-creator-change-monitor)](https://www.npmjs.com/package/@mambalabsdev/mcp-creator-change-monitor) [![npm downloads](https://img.shields.io/npm/dm/@mambalabsdev/mcp-creator-change-monitor)](https://www.npmjs.com/package/@mambalabsdev/mcp-creator-change-monitor) [![license](https://img.shields.io/github/license/mambalabsdev/mcp-creator-change-monitor)](https://github.com/mambalabsdev/mcp-creator-change-monitor/blob/main/LICENSE) [![mcpservers.org](https://img.shields.io/badge/mcpservers.org-listed-blue)](https://mcpservers.org/servers/mambalabsdev/mcp-creator-change-monitor)

MCP server for the Mamba Labs [Influencer Change Monitor](https://apify.com/mambalabs/creator-change-monitor) actor on Apify.

A named list of creator handles in, one row per change since the previous run out.

## Install

```bash
npx -y @mambalabsdev/mcp-creator-change-monitor
```

### Claude Desktop

```json
{
  "mcpServers": {
    "mamba-creator-change-monitor": {
      "command": "npx",
      "args": ["-y", "@mambalabsdev/mcp-creator-change-monitor"],
      "env": { "APIFY_TOKEN": "your-apify-token" }
    }
  }
}
```

Get an Apify token at [console.apify.com/account/integrations](https://console.apify.com/account/integrations).

## Tool

### `monitor_influencer_changes`

Recheck a named influencer watchlist and return what moved since the run before it.

| Input | Type | Required | Notes |
| --- | --- | --- | --- |
| `list_name` | string | no | Names the saved state. The first run for a list returns one first_seen row per creator as the baseline; later runs with the same name return only changes. Letters, digits, dot, dash, underscore. Default `"default"`. |
| `handles` | array | no | One per line. A profile URL on any supported platform (https://www.tiktok.com/@name, https://www.instagram.com/name/, https://www.youtube.com/@name, a Pinterest, Twitch, or Threads profile, an Apple Podcasts show. |
| `platforms` | array | no | Which platforms a bare @handle is looked up on. A full profile URL carries its own platform and ignores this. This actor does not search; pass the creators you want read. Supported: TikTok, Instagram, YouTube,. |
| `check_links` | boolean | no | Rechecks newsletter status, products, and manager contact on every run. Uncheck for a follower and bio link only check at lower cost: the link columns then keep the values from the last run that read them, and no. Default `true`. |
| `match_agencies` | boolean | no | Matches the domain of a manager or business email against the bundled talent agency list and fills agency_name, agency_domain, and agency_match_method. Charged per matched row (event agency-match). Default `true`. |
| `escalate_on_block` | boolean | no | On by default. A profile fetch that comes back as a bot detection page is retried once over the residential proxy. On Instagram the bio, bio link, and following are read from the profile page over residential when. Default `true`. |
| `batch_size` | integer | no | Rows fetched at once. Leave empty for the measured per platform default; the measurement is in the README. Higher is faster and, above the measured point, loses rows. |
| `twitch_client_id` | string | no | Optional. Your own registered Twitch application client id. With `twitch_app_token` the Twitch reads use the official Helix API instead of the public web endpoint. Never a Mamba Labs credential. |
| `twitch_app_token` | string | no | Optional. An app access token for your Twitch client id (client credentials flow). Used only for Twitch reads, never stored or logged. |
| `contribute_to_shared_pool` | boolean | no | On by default. Contributes the public records this run finds to a shared creator and agency pool that all users of this actor read from. Only public data already in the returned rows is sent, and a contribution is not charged. Set false to read the pool and write nothing. Default `true`. |

Nothing is required. Influencer Change Monitor answers a run with no usable input with a row carrying `row_status` and `error_reason` rather than failing, and the tool mirrors that.

## Pricing

Influencer Change Monitor is pay per event on Apify. Every price below is flat across the FREE, BRONZE, SILVER, and GOLD tiers.

| Event | Charged for | Price | Fires when |
| --- | --- | ---: | --- |
| `actor-start` | Actor start | $0.002 | Once per run, on start. Covers the run overhead. |
| `creator-rechecked` | Creator rechecked | $0.008 | Once per creator whose profile (and, when check_links is on, link page) was read on a monitor run, whether or not anything changed. This is the base price per creator; a change row is charged on top of it. |
| `change-detected` | Change detected | $0.01 | Once per change row returned by the monitor, on top of the recheck. A first_seen baseline row is a change row. Unchanged creators return no row. |
| `instagram-bio-fetch` | Instagram bio fetch | $0.01 | Once per Instagram profile row when the bio, bio link, and following were not on the embed widget or the datacenter API and the profile page was read over the residential proxy and came back readable. Only when escalate_on_block is on. Never on the embed or datacenter reads, never on another platform, never on a blocked page, and never on an error row. |

## Reading the output

Every row carries `row_status` and `error_reason`. A creator the actor could not read comes back as a row saying why, not as a gap in the list, so an absence is readable rather than inferred. Filter on `row_status` before loading a table.

## What this actor shares

The run contributes the records it finds to a shared creator and agency pool that all users of this actor read from. What one run finds, the next run can read.

The toggle is `contribute_to_shared_pool`. It is on by default. Set it to false and the run still reads the pool and writes nothing to it.

**What this actor contributes.** The profile fields that changed since the last run, and any manager contact that is new.

**Only public data that is already in your own output.** Every field written to the pool is a field this run returned to you, read from a page the platform or the creator publishes to anyone without a login. Nothing from your Apify account, your input list, your API keys, or your own notes is sent. A contribution never deletes anything from the pool.

**What a contribution is labeled with.** The actor ID, the run ID, the pool key issued to the actor build, and a hash of the calling IP address, used for the rate limit and nothing else. Your Apify account and your user ID are not recorded.

**Contributing is free.** No event is charged for a write to the pool. If the pool is unreachable the run finishes as normal, the rows are dropped, and the run log says so.

## Actor

Actor ID `d2VVgahNL6UmcLkhg`. The wrapper calls the actor by that immutable ID rather than by its Store slug, so a Store rename never breaks it.

## Suite

| Actor | Actor ID | MCP server |
| --- | --- | --- |
| [Influencer Finder](https://apify.com/mambalabs/creator-finder) | `tpkedmloIIWtXx6sg` | [`@mambalabsdev/mcp-creator-finder`](https://www.npmjs.com/package/@mambalabsdev/mcp-creator-finder) |
| [Influencer Profile Scraper](https://apify.com/mambalabs/creator-profile-reader) | `EqEnklDVMIcB1V499` | [`@mambalabsdev/mcp-creator-profile-reader`](https://www.npmjs.com/package/@mambalabsdev/mcp-creator-profile-reader) |
| [Link in Bio Scraper and Newsletter Detector](https://apify.com/mambalabs/link-in-bio-newsletter-checker) | `OorucdheTIgu7RFzK` | [`@mambalabsdev/mcp-link-in-bio-newsletter-checker`](https://www.npmjs.com/package/@mambalabsdev/mcp-link-in-bio-newsletter-checker) |
| [Influencer Change Monitor](https://apify.com/mambalabs/creator-change-monitor) | `d2VVgahNL6UmcLkhg` | [`@mambalabsdev/mcp-creator-change-monitor`](https://www.npmjs.com/package/@mambalabsdev/mcp-creator-change-monitor) |
| [Influencer Lead List Builder](https://apify.com/mambalabs/creator-lead-list-all-in-one) | `KnmByszcv135yM30G` | [`@mambalabsdev/mcp-creator-lead-list-all-in-one`](https://www.npmjs.com/package/@mambalabsdev/mcp-creator-lead-list-all-in-one) |

Built by [Mamba Labs](https://mambabuilt.com).
