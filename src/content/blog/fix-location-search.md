---
title: Fix location search - revive Google Maps button
description: A simple & 100% free chrome extension that restores the Maps button for location search
tags: [ maps, google maps, maps button, how to add google maps, fix google maps, location search ]
publishDate: 2024-12-02
---

# Fixing location search by restoring the Google Maps Tab: A Developer's Solution

When Google quietly removed the dedicated Maps tab from search results, something was lost. That convenient
click-through to location information disappeared, making local searches just a bit more cumbersome. As a web
developer, I saw an opportunity: why not build a solution?

## The Problem

Google Search used to have a beautiful, intuitive Maps tab. One click would transport you directly to map results for
your location search. Suddenly, that option vanished. Users were forced to take extra steps to find location
information – a small friction that adds up over countless searches.

## The Chrome Extension Solution - Fix location search!

This is where a lightweight Chrome extension comes into
play - [bringbackmaps](https://chromewebstore.google.com/detail/bringbackmaps/dhfofnchclaidhjihbikjjemdodddlji). By
leveraging standard web technologies – HTML, JavaScript, and Chrome's extension APIs – we can inject the Maps tab back
into search results.

![Final result: Navbar in Google search displays a "Maps" tab again!](/images/google_search_for_location.png)

## How It Works

The extension uses a few key technical strategies:

1. Location Detection: A smart algorithm identifies location-related searches by:

    - Scanning search keywords like "near", "location", "address"
    - Detecting map-related page layouts

2. Dynamic Tab Injection: Using a MutationObserver, the extension:

    - Waits for search results to load
    - Identifies the navigation tabs
    - Dynamically creates and inserts a new Maps tab

3. Intelligent Linking: The injected tab links directly to Google Maps with your current search query.

## Technical Highlights

The core of the extension is a content script that runs on Google search pages. Here's a simplified peek at the location
detection logic:

```javascript 
function isLocationSearch() {
    const searchQuery = getInputField()?.value.toLowerCase() || '';
    const locationKeywords = ['near', 'in', 'at', 'location', 'address', 'where', 'place', 'directions']; // extend for other languages

    const locationInfobox = getTravelLayout();

    return locationInfobox !== null || locationKeywords.some(keyword => searchQuery.includes(keyword));
}

function getTravelLayout() {
    return document.querySelector("div[data-ly='/travel_layout/map_2modules']");
}
```

Detecting location searches by analyzing search keywords & DOM elements.

## Getting the Extension

- [Available for free on the Chrome Web Store](https://chromewebstore.google.com/detail/bringbackmaps/dhfofnchclaidhjihbikjjemdodddlji)
- [Open-source code on GitHub](https://github.com/lukas-karsch/bringbackmaps)
- 100% open-source and transparent
