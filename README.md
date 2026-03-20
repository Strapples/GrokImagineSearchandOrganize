# Grok Imagine Search and Organize (GISaO) - Updates are located at (Http://github.com/Strapples/GrokImagineSearchandOrganize/)
Forked from IronSniper1 (here on Github) Big thanks for making a usable base code for this piece!

GISaO is a full Grok Imagine UI replacement system, it replaces the Grok Imagine interface with a more desktop friendly interface and also one that likely will function better with accessibility tools. Infinite scrolling is gone which pains many people with not being able to directly identify a page number a particular item is on. Now if that favorite render of yours is on page 38 then expect it to stay there when you view your entire library (yup, fixed it, this tool now loads your entire library at startup instead of requiring a query to be in the box. - If you want to keep the Grok UI and only use a tool for search then you'll want to head to IronSniper1's fork) This tool has an instand handoff system so when you click on any of your images or videos (yup, made sure this one indexes video files too!) it'll take you to the respective item page in the full GrokUI.

---

## Features

- 🔍 **Search by prompt** — instantly filter your entire saved library by any keyword or phrase
- ⚡ **Persistent IndexedDB index** — After the first index pass all photos and videos (videos will be represented as a single frame of image from the video, clicking one will take you to the item page)
- 🔄 **Incremental updates** — on each visit, only fetches images you've liked since the last run
- 🚀 **Fast indexing** — first-time index runs as fast as the API allows with no artificial delays, fetching 40 images per request
- 📄 **Paginated results** — 20 results per page with Prev / Next / First / Last navigation (will work on creating an easy config token within the .js that users can change)
- 🗓 **Sort by Newest or Oldest** (sort by random coming soon because who doesn't like to shuffle the deck?)
- 🖼 **Own results grid** — renders matching images directly, completely bypassing Grok's virtualised scroll
- 💬 **Prompt tooltip** — hover any result card to see the full prompt
- 🖱 **Click to open** — click any image to open the asset in the Grok system
- ⌨️ **Keyboard shortcuts** — `Ctrl/Cmd+F` to focus search, `←` `→` to page through results
-  Videos do not autoplay, (total protection from those weird moments when one of your adult renders plays in front of the librarian!) (THIS IS A FEATURE NOT A BUG)
- Empty bar shows all images (WORKING)
- Instant transport to YOUR post item with Video or Photo (WORKING) (the prior script only loads images, this one loads your entire xAI Imagine item!)
- persistent tags WIP
- download baked PNG WIP
- Custom Prompt Injector Box (Possibility of using cross API calls to get GPT or another asset to randomize keywords or entire prompts)
---

## Installation

### Step 1 — Install Tampermonkey

Install the [Tampermonkey](https://www.tampermonkey.net/) browser extension:

- [Chrome](https://chrome.google.com/webstore/detail/tampermonkey/dhdgffkkebhmkfjojejmpbldmpobfkfo)
- [Firefox](https://addons.mozilla.org/en-US/firefox/addon/tampermonkey/)
- [Edge](https://microsoftedge.microsoft.com/addons/detail/tampermonkey/iikmkjmpaadaobahmlepeloendndfphd)
- [Safari](https://apps.apple.com/app/tampermonkey/id1482490089)

---

### Step 2 — Configure Tampermonkey (Do This Before Installing the Script)

After installing Tampermonkey, you need to enable two settings or the script will not work.

**Allow User Scripts:**

1. Go to `chrome://extensions` in your address bar
2. Find Tampermonkey and click **Details**
3. Scroll down and turn on **Allow User Scripts**

This allows Tampermonkey to run scripts that have not been reviewed by Google. You should only enable this if you trust the script you are installing.

**Allow in Incognito** *(optional)*:

On the same Details page, also turn on **Allow in Incognito** if you want the script to work in private/incognito windows. Without this the script will only run in normal browser windows.

Once both settings are enabled, proceed to installation below.

---

### Step 3 — Install the Script

1. Click the Tampermonkey icon in your browser toolbar
2. Select **Create a new script**
3. Copy the script from this repository, use the most recent build or face the wrath of my buuuuugs!
4. Paste it in the new script on tampermonkeys editor
5. Save it
6. hard refresh Grok or relaunch your browser
7. Enjoy the cleaner UI 

---

## Usage

### First Run

On your first visit after installing, the script will index your entire library. A status indicator in the search bar shows progress:

```
indexing… 1,240
indexing… 2,480
...
6,112 indexed
```

This only happens once. Subsequent loads read from the local IndexedDB cache instantly.

> ⏳ **Heads up:** Indexing time depends on how many images you have. A few hundred takes seconds, but a large library can take significantly longer. The script has to paginate through the entire API one batch at a time. It's best to leave the tab open and let it run in the background — or just kick it off before you go to sleep and it'll be ready in the morning. You only ever have to do this once.

> ⚠️ **API limit:** Grok's API caps how far back the liked feed goes in a single indexing session at around 6,000–6,100 posts. This is a hard server-side limit — not a bug in the script. Your index will naturally grow over time as the incremental update system picks up new likes on each visit, eventually building up a larger total index across multiple sessions.

### Searching

Type any word or phrase into the search bar. Multi-word searches require **all terms** to match (AND logic).

| Query | Matches |
|-------|---------|
| `dragon` | All prompts containing "dragon" |
| `red dragon forest` | Prompts containing all three words |
| `anime portrait` | Prompts containing both "anime" and "portrait" |

### Navigation

| Control | Action |
|---------|--------|
| `⏮ First` button | Jump to page 1 |
| `◀ Prev` button | Previous page |
| `Next ▶` button | Next page |
| `Last ⏭` button | Jump to last page |
| `←` / `→` arrow keys | Page through results (when search box is not focused) |
| `Ctrl/Cmd+F` | Focus the search bar |
| `Esc` | Unfocus the search bar |

### Sorting

Use the **Newest / Oldest** dropdown in the search bar to change the sort order of results. Sorting applies instantly without re-fetching.

### Incremental Updates

Every time you visit `grok.com/imagine`, the script:

1. Loads your full index from IndexedDB (instant)
2. Fetches the latest page from the API
3. Stops as soon as it hits an image it already knows
4. Saves any new images to the index

The status bar briefly shows `+N new (X total)` or `up to date`.

---

## How It Works

Grok's saved images page uses **virtualised rendering** — only the images currently visible in the viewport exist in the DOM. This makes it impossible to search by scanning the page directly.

This script works around that by:

1. **Calling the Grok API directly** (`/rest/media/post/list`) to fetch all your liked posts, including their prompt text
2. **Storing everything in IndexedDB** — a browser-native database with no meaningful size limit, persisted across sessions
3. **Rendering its own results grid** when you search, hiding Grok's native grid and displaying matching images from the index

---

## Storage

The script uses the browser's built-in **IndexedDB** to store your index locally. No data is sent anywhere — everything stays in your browser.

- **Database name:** `GrokSearchIndex`
- **Stored per post:** `id`, `prompt`, `thumbnail URL`, `media URL`, `createTime`
- **Typical size:** ~4–6 MB for 6,000+ posts

To clear the index, run this in your browser console on `grok.com/imagine/saved`:

```js
indexedDB.deleteDatabase('GrokSearchIndex');
```

Then refresh the page and the script will re-index from scratch.

---

## Checking Your Index

To verify how many posts are indexed, run this in the console on `grok.com/imagine/saved`:

```js
const req = indexedDB.open('GrokSearchIndex');
req.onsuccess = e => {
  const db = e.target.result;
  const tx = db.transaction('posts', 'readonly');
  const store = tx.objectStore('posts');
  const count = store.count();
  count.onsuccess = () => console.log(`✅ IndexedDB has ${count.result.toLocaleString()} posts indexed`);
  const sample = store.getAll(null, 3);
  sample.onsuccess = () => {
    console.log('Sample posts:');
    sample.result.forEach(p => console.log(`  • [${p.createTime?.slice(0,10)}] ${p.prompt?.slice(0,80)}`));
  };
};
```

---

## Compatibility

| Browser | Status |
|---------|--------|
| Chrome / Chromium | ✅ Should Work (Edge is a 'chrome like' browser now) |
| Firefox | ✅ Not Tested but support should work? Open a ticket in issues if it doesn't and I'll fiddle with the code, and by fiddle with the code I mean break it 30 times with Grok and ChatGPT before I find which line fixes it and polish it off myself four hours after starting the code sesh. |
| Edge | ✅ Tested |
| Safari | ⚠️ Not Yet Tested |

Requires Tampermonkey v4.0 or later.

---

## Known Limitations

- **API pagination cap** — Grok limits the liked feed to roughly 6,000–6,100 posts per indexing session. Older posts beyond this are not accessible in a single pass, but the index grows over time through incremental updates on each visit.
- **Posts with no prompt** — some posts (particularly auto-generated videos) have no prompt text. These are still indexed and counted and will appear with a BLANK search box (wildcard only)
- **Unliked images** — removing a like removes it from future incremental updates but does not automatically remove it from the local index. (Will work on a pruning mechanism, eventually........... (If this line is still here and it's past the year of 2027, open a ticket so I can fix this bug and close the ticket, open tickets are straight HELL, I never go to bed with open tickets.)
- **API changes** — if Grok changes their internal API, the script may break, which means you the user should contact me with a service ticket so I can restore the script to functionality, or find out that Grok locked down client side modding. 

USERS: Please be active on X in supporting client side UI tweaks. Developers like us don't hurt Grok, we make the Grok experience more flexible without adding work to X Developers. These tools can make Grok more accessible for people with disabilities, safer for persons to use in public areas and unlock client side compute for image edits and other tweaks within the browser that aren't worth X's time to develop but *are* worth the time of us code tinkerers who love to fuck around with code and make new things exist. 

I'm really proud of this forks success because it actually fixes a real software bug, adds new features and actually *fully works* (as of me testing it on Edge, if it doesn't work in your browser I again stress *please open a ticket, I will fix it ASAP*

---

## Related Scripts

These Scripts are from IronSniper1 (The guy who I forked this off of, so he gets credit for making the base code that I was able to frankenstein into what we now know as GISaO

**[Grok Imagine – Bulk Favorites Downloader](https://github.com/ironsniper1/Grok-Imagine-Bulk-Favorites-Downloader)**  
Downloads all your saved images in chunks, remembers what's already been downloaded, and supports folder-based organisation.
**{Grok Imagine Favorites Image Search) The thing I forked to make this thing
https://github.com/ironsniper1/Grok-imagine-favorite-image-search
---

## Privacy

This script runs entirely inside your browser. It only communicates with `grok.com`, `assets.grok.com`, and `x.ai` to fetch your own media. No data is sent to any third-party server. You can look at the code yourself in the code editor to verify the security of it.

---

## Disclaimer

This is an unofficial tool not affiliated with Grok or X. It uses Grok's publicly available API which may change at any time. This software may cease functioning at any time or may bugger up. If that happens open a ticket on here and I will try to fix it.

---

## License

MIT — do whatever you want with it (Though do link back to my github)

A Tampermonkey userscript that adds a **full-text search interface** to your saved Grok Imagine images — bypassing Grok's virtual scroll to search your entire library, not just what's currently on screen.

Screen - This image shows it in action.
<img width="899" height="1451" alt="Screenshot 2026-03-20 at 18 12 05" src="https://github.com/user-attachments/assets/a4ba2e86-030f-4991-b0c3-7a43a0963560" />
(Yes I already know the prompt box gets in the way of the bottom images. I am working on fixing this, I also plan on increasing the max images to 24 to fill the bottom line on larger monitors.)

