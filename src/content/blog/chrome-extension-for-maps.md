---
title: Building a Chrome Extension to Restore the Google Maps Tab
description: A comprehensive guide on how to restore the Google Maps search button with your own Chrome extension
tags: [ chrome, extension, maps, google maps, how to create chrome extension ]
publishDate: 2024-11-26
---

# Building a Chrome Extension to restore the Google Maps Tab

You may have noticed that Google Search has recently removed the dedicated "Maps" tab from their
search results page. This was in my opinion one of the nicest features of Google search - I clicked that link every time
I looked for a location!

As a web developer, I thought:
> Why not help myself (and others) out?

In this article, we'll walk through the process of creating a Chrome extension that brings back the missing Maps tab,
so that you can easily access Google Maps directly from the search results!

The full code for this guide & a download link for the final extension is available on my **GitHub profile - check it
out [at the end of the page](#download-the-extension-to-bring-back-the-google-maps-tab-in-google-search)**.

## Understanding Chrome Extensions

You probably know this, but let's recap: Chrome extensions are (small) tools that users can install to add new
features or modify the behavior of their web browser. These extensions are built using standard web technologies like
HTML, CSS, and JavaScript.

A Chrome extension typically consists of the following key components:

- Manifest File: The manifest.json file is the core of the extension, defining its metadata, permissions, and other
  configuration settings.
- Content Scripts: These are JavaScript files that run in the context of web pages, allowing the extension to interact
  with the page's content and structure.
- Background Scripts: Background scripts handle extension events and manage the overall functionality of the extension.
- Popup/Options UI: Extensions can also include HTML/CSS/JS files that render a popup or options page, providing a user
  interface for the extension.
- Icons: Extensions require various icon sizes (16x16, 48x48, 128x128) to represent the extension in the browser.

We will actually only need the content.js! However, in my final version that you can download from my GitHub, I also
included a
small popup that links to my socials.

## Building the "Bring Back Maps" Extension

Let's dive into the step-by-step process of creating an extension to restore the Google Maps tab.

Here is what our final result will look like (yes, doesn't look like a lot - but trust me, it's gonna save you lots of
time ;) ).
The image of the map will also be clickable!
![Final result: Navbar in Google search displays a "Maps" tab again!](/images/google_search_for_location.png)

### 1. Setting up the Project for our Chrome Extension

Create a new directory for your extension project.
Within the directory, we are going to need the following files:

- `manifest.json`
- `content.js`

If you have icon files (16x16, 48x48, 128x128 pixels), add them to the project directory.

### 2. Configuring the Manifest JSON File

The `manifest.json` file is the entry point of your extension. Here's an example configuration:

```json
{
  "manifest_version": 3,
  "name": "Google Maps Tab Restorer",
  "version": "1.0",
  "description": "Restores the Maps tab in Google Search results for location searches",
  "permissions": [
    "activeTab"
  ],
  "content_scripts": [
    {
      "matches": [
        "*://www.google.com/*",
        "*://google.com/*"
      ],
      "js": [
        "content.js"
      ],
      "css": [
        "styles.css"
      ]
    }
  ],
  "icons": {
    "16": "icon16.png",
    "48": "icon48.png",
    "128": "icon128.png"
  }
}
```

This configuration specifies that the extension will run on all google.com pages, injecting the
content.js script and styles.css file. It also declares the required permissions, so we are allowed to access the active
tab, and includes the necessary icon sizes (this is of course optional).

### 3. Implementing the content.js Script

The content.js file is where the core functionality of the extension is implemented. It will detect location-related
searches, wait for the search results to load, and then inject the Maps tab into the page.

Let's start by outlining what needs to happen. This script first checks if the current search query appears to be
location-related, then waits for the search results page to load. Once the necessary DOM elements are available, it
clones an existing tab element, modifies the URL to point to Google Maps, and inserts the new Maps tab into the page.

```javascript
function init() {
    if (isLocationSearch()) { // check if the current search is a search about a location
        const observer = new MutationObserver((mutations, obs) => {
            // find the container that contains navigation tabs
            const tabsContainer = document.querySelector('div[role="navigation"] div[role="list"]');
            if (tabsContainer) {
                createMapsTab(); // create navigation tab with the link to Maps
                obs.disconnect(); // stop observing the dom
                makeImageClickable(); // make the map image clickable
            }
        });

        observer.observe(document.body, {
            childList: true,
            subtree: true
        });
    }
}

init();
const originalPushState = history.pushState;
history.pushState = function () {
    originalPushState.apply(this, arguments);
    setTimeout(init, 100);
};

```

Great! Time to implement the functions. Let's start by determining whether the search is location-related.

```javascript
function isLocationSearch() {
    const searchQuery = getInputField()?.value.toLowerCase() || '';
    const locationKeywords = ['near', 'in', 'at', 'location', 'address', 'where', 'place', 'directions'];

    const locationInfobox = getTravelLayout();

    return locationInfobox !== null || locationKeywords.some(keyword => searchQuery.includes(keyword));
}

function getTravelLayout() {
    return document.querySelector("div[data-ly='/travel_layout/map_2modules']");
}

function getInputField() {
    return document.querySelector('input[name="q"]');
}
```

The query selectors might be quite hard to get right if you decide to do something similar on another page. I spent **a
lot of time** inside of Chrome devtools. Once you've identified where your desired element is placed, you can also ask
your favorite LLM (I personally use Claude by Anthropic for coding) to write the querySelector for you.

Alright, Now we know that the search is location-related! Let's actually create the tab in the search navigation bar.

```javascript
function getMapsUrl() {
    const searchTerm = encodeURIComponent(getInputField().value);
    return `https://www.google.com/maps/search/${searchTerm}`;
}

function createMapsTab() {
    const tabsContainer = document.querySelector('div[role="navigation"] div[role="list"]');
    console.log({tabsContainer})
    if (!tabsContainer || document.querySelector('.maps-tab-restored')) return;

    const mapsUrl = getMapsUrl()

    // Find an existing tab to clone its structure
    const existingTab = tabsContainer.querySelector('div[role="navigation"] div[role="listitem"]:has(> a:not([aria-disabled="true"]))');
    console.log({existingTab})
    if (!existingTab) return;

    const mapsTab = existingTab.cloneNode(true);

    const linkToMaps = mapsTab.querySelector('a');
    if (linkToMaps) {
        linkToMaps.href = mapsUrl;
        linkToMaps.setAttribute("data-ved", undefined) // unset some google attributes
        linkToMaps.setAttribute("data-hveid", undefined)
    }

    const linkContent = linkToMaps.querySelector("div")
    linkContent.textContent = "Maps"

    // Insert after Images tab (estimation)
    const imagesListItem = Array.from(tabsContainer.querySelectorAll('div[role="listitem"]'))[1]

    if (imagesListItem) {
        imagesListItem.after(mapsTab);
    } else {
        tabsContainer.appendChild(mapsTab); // or just append the tab at the end if we can't find the "images" tab
    }
}
```

Wow, we are almost done!! Now we just need to make that map image clickable.

```javascript
function getTravelLayout() {
    return document.querySelector("div[data-ly='/travel_layout/map_2modules']");
}

function makeImageClickable() {
    const travelLayout = getTravelLayout()
    const mapsImage = travelLayout.querySelector("img:not(ol img)");
    const parent = mapsImage.parentElement;

    const a = document.createElement("a")
    a.href = getMapsUrl()

    const removed = parent.removeChild(mapsImage)
    a.appendChild(removed)
    parent.appendChild(a)
}
```

### 4. Testing the Extension Locally

We implemented our functionality - time to test it out!

- Open the Chrome browser and navigate to chrome://extensions/
- Enable "Developer mode" by toggling the switch in the top right corner.
- Click the "Load unpacked" button and select your extension directory.
- Your extension should now appear in the list of installed extensions.
- Navigate to Google.com and perform a location-related search. The Maps tab should appear in the search results.

## Download the extension to bring back the Google Maps tab in Google Search

As promised, you can get the code for this extension here:
[GitHub](https://github.com/lukas-karsch/bringbackmaps)

Or add the extension directly to chrome by downloading it from
the [Chrome Web store](https://chromewebstore.google.com/detail/bringbackmaps/dhfofnchclaidhjihbikjjemdodddlji)

If anything breaks or you find a bug - let me know! Submit a Pull Request or send me a message
at [x.com/builtbylukas](https://x.com/builtbylukas)
