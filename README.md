# Apify Skills

Skills that let your AI agent get data from the web with Apify Actors.

## Skills

| Skill | What it does |
| --- | --- |
| [bilibili-video-downloader](skills/bilibili-video-downloader/SKILL.md) | Download Bilibili videos, audio and details |
| [snapchat-video-downloader](skills/snapchat-video-downloader/SKILL.md) | Download public Snapchat Spotlight videos with metadata |
| [storygraph](skills/storygraph/SKILL.md) | Scrape StoryGraph ratings, moods and warnings |
| [twitter-lists](skills/twitter-lists/SKILL.md) | Scrape X lists: members, tweets, details |
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

- **Apify account**: [sign up here](https://console.apify.com/sign-up). The free plan is enough to start.
- **APIFY_TOKEN**: get it from [console.apify.com/account/integrations](https://console.apify.com/account/integrations)
- **curl** in your environment

```bash
export APIFY_TOKEN="apify_api_xxxxxxxxxxxx"
```
