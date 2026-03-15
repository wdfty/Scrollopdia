# Scrollopedia

A TikTok-style vertical feed for browsing Wikipedia articles. Instead of doomscrolling social media, you doomscroll through random (and eventually personalized) encyclopedia entries. Built as a single-page HTML/JS app with no frameworks or build tools.

## What It Does

Scrollopedia pulls article summaries from the Wikipedia REST API and presents them as full-screen cards you swipe through vertically. Each card shows a hero image (when available), the article title, a short extract, and a link to the full Wikipedia page.

The app starts by showing completely random articles, but as you like things, it starts learning what you're into. Over time the feed becomes a mix of content related to your interests and random discoveries so it doesn't turn into a total echo chamber.

## Features

**Vertical scroll feed** -- navigate by scrolling, swiping on mobile, or using arrow keys / j and k. Each article takes up the full viewport.

**Like system** -- hit the heart button (or press L) to save articles you find interesting. Liked articles are stored in a side panel you can open from the header. Likes also feed into the recommendation engine.

**Recommendation engine** -- the app extracts keywords from articles you like and builds a lightweight interest profile. It uses that profile to search Wikipedia for related content and pull articles linked from things you've already enjoyed. The feed targets roughly 60% related content and 40% random stuff to keep things fresh.

**Interest decay** -- older interests gradually lose weight so the feed evolves with you rather than getting stuck on whatever you liked first.

**Click tracking** -- opening the full Wikipedia article gives a smaller interest signal (about 40% of a like), so even browsing behavior shapes future recommendations.

**Sharing** -- uses the native Web Share API where available, falls back to copying the URL to clipboard.

**Image attribution** -- fetches license and artist info from Wikimedia Commons in the background and displays it on cards.

**Persistence** -- likes, interest profile, and seen-article history are saved to localStorage so everything carries over between sessions.

## How It Works

### Data Flow

1. On load, the app checks localStorage for any saved state (previous likes, interests, seen articles).
2. It calls `loadSmartArticles()` which decides how to source the next batch based on whether the user has liked anything yet.
3. If there are no likes, everything comes from Wikipedia's random article endpoint.
4. If there are likes, it builds search queries from top interest keywords, fetches linked articles from recently liked pages, and mixes those with random articles.
5. Articles are scored by how well their keywords match the user's interest profile, sorted, then interleaved with random articles in a 2:1 pattern.
6. As the user scrolls, new batches are prefetched when the buffer drops below 6 articles.

### Recommendation Scoring

When you like an article, the engine extracts its top 15 keywords (filtering out stop words). Keywords that appear in the title get a 4x weight boost, description keywords get 2.5x. Every time you like something, all existing interest weights decay by 5% to keep the profile from getting stale.

When scoring a candidate article, it checks keyword overlap with the interest profile and applies bonuses for multiple matches (topic density), having an image, and penalties for stub articles with very short extracts.

### Deduplication

The app tracks seen article IDs and titles across sessions (up to 500 IDs persisted) to avoid showing the same thing twice. Each batch is also deduplicated internally before rendering.

## Tech Stack

This is a single HTML file. No React, no bundler, no dependencies beyond two Google Fonts (Linux Libertine and Instrument Sans). All the logic lives in vanilla JavaScript at the bottom of the file.

External APIs used:
- Wikipedia REST API (`en.wikipedia.org/api/rest_v1/`) for article summaries and random articles
- Wikipedia Action API (`en.wikipedia.org/w/api.php`) for search, article links, and image metadata

## Running It

Open the HTML file in a browser. That's it. No server, no build step, no install. You do need an internet connection since it fetches everything from Wikipedia on the fly.

## Known Limitations

- localStorage has a size cap (usually 5-10 MB depending on the browser), and the app stores seen article IDs indefinitely. It caps the persisted set at 500 IDs, but heavy users could still run into issues over a very long period.
- The recommendation engine is keyword-based, which means it can latch onto common words that happen to appear across unrelated articles. It works well enough for casual browsing but it's not doing any real NLP.
- Wikipedia's random article endpoint sometimes returns disambiguation pages or very short stubs. The app filters these out, but occasionally one slips through.
- No offline support. Every article is fetched live.
- Image loading can be slow for large Wikimedia files since there's no thumbnail size negotiation -- it just takes whatever Wikipedia's API gives it.

## Other
Wikipedia content fetched by the app is available under the Creative Commons Attribution-ShareAlike License as provided by the Wikimedia Foundation.
