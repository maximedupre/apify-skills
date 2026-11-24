# Apify Skills

Skills that let your AI agent get data from the web with Apify Actors.

## Skills

| Skill | What it does |
| --- | --- |
| [amazon-keywords-discovery-tool](skills/amazon-keywords-discovery-tool/SKILL.md) | Expand seeds into Amazon autocomplete keywords |
| [bilibili-video-downloader](skills/bilibili-video-downloader/SKILL.md) | Download Bilibili videos, audio and details |
| [handshake-jobs-scraper](skills/handshake-jobs-scraper/SKILL.md) | Collect public Handshake jobs, pay, and employers |
| [instagram-reels-search-scraper](skills/instagram-reels-search-scraper/SKILL.md) | Search public Instagram Reels by keyword or hashtag |
| [jefferson-parish-assessor](skills/jefferson-parish-assessor/SKILL.md) | Look up Jefferson Parish property assessor records |
| [kick-clips](skills/kick-clips/SKILL.md) | Find, rank, and transcribe public Kick clips |
| [reddit-scraper](skills/reddit-scraper/SKILL.md) | Scrape Reddit posts, comments, subreddits, users |
| [rottentomatoes](skills/rottentomatoes/SKILL.md) | Get Rotten Tomatoes scores, details, and reviews |
| [snapchat-ads](skills/snapchat-ads/SKILL.md) | Scrape Snapchat ads, political ads, creator content |
| [snapchat-video-downloader](skills/snapchat-video-downloader/SKILL.md) | Download public Snapchat Spotlight videos with metadata |
| [spotify-downloader](skills/spotify-downloader/SKILL.md) | Get Spotify track data and MP3 links from URLs or searches |
| [storygraph](skills/storygraph/SKILL.md) | Scrape StoryGraph ratings, moods and warnings |
| [tiktok-shop-shops](skills/tiktok-shop-shops/SKILL.md) | Rank TikTok Shop shops by sales or keyword |
| [twitter-lists](skills/twitter-lists/SKILL.md) | Scrape X lists: members, tweets, details |
| [untappd-scraper](skills/untappd-scraper/SKILL.md) | Collect public Untappd beers, breweries, and venues |
| [viewstats](skills/viewstats/SKILL.md) | Get Viewstats YouTube channel analytics via Apify |

## Installation

### npx skills

Install all skills:

```bash
npx skills add maximedupre/apify-skills
```

Install one skill:

```bash
npx skills add maximedupre/apify-skills --skill <skill-name>
```

### Claude Code

```
/plugin marketplace add maximedupre/apify-skills
/plugin install <skill-name>@apify-skills
```

## Prerequisites

- **Apify account**: [sign up here](https://apify.com/?fpr=qn4m5w). The free plan is enough to start.
- **APIFY_TOKEN**: get it from [console.apify.com/account/integrations](https://console.apify.com/account/integrations?fpr=qn4m5w)
- **curl** in your environment

```bash
export APIFY_TOKEN="apify_api_xxxxxxxxxxxx"
```
