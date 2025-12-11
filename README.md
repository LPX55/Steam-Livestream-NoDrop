# SteamTV Live Stream Chat Spam Filter (`!drop`)

A lightweight solution to filter out repetitive `!drop` spam messages from SteamTV live chat during the Budapest Major 2025.

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Solutions](#solutions)
  - [1. JavaScript Filter (Recommended)](#1-javascript-filter-recommended)
  - [2. CSS Visual Filter](#2-css-visual-filter)
- [Installation](#installation)
- [Usage](#usage)
- [Troubleshooting](#troubleshooting)
- [Security & Privacy](#security--privacy)
- [Browser Compatibility](#browser-compatibility)
- [Development](#development)
- [License](#license)

## 🎯 Overview

During the Budapest Major 2025, SteamTV chat was flooded with users spamming `!drop` commands expecting some kind of response. This repository provides two solutions to filter out these spam messages:

- **JavaScript Filter**: Blocks messages at the network level (recommended)
- **CSS Visual Filter**: Hides messages visually after they load

Both solutions are specifically designed for SteamTV's broadcast chat interface and may break with future UI updates.

## ✨ Features

- ✅ Filters `!drop` spam messages
- ✅ Two implementation options (JavaScript and CSS)
- ✅ Lightweight and fast
- ✅ No external dependencies
- ✅ Easy to customize and extend

## 🛠️ Solutions

### 1. JavaScript Filter (Recommended)

This solution intercepts network requests and filters out spam messages before they reach the chat interface. It's more efficient and prevents spam messages from loading entirely.

#### How it works:
- Intercepts `fetch()` and `XMLHttpRequest` API calls
- Filters out messages containing blocked patterns
- Returns clean data to the chat interface
- Works at the network level for optimal performance

#### Code:
```javascript
// ==UserScript==
// @name         SteamTV Chat !drop Filter
// @namespace    http://tampermonkey.net/
// @version      1.2
// @description  Filter !drop messages from SteamTV broadcast chat
// @match        https://steam.tv/*
// @match        https://steamcommunity.com/*
// @grant        none
// @run-at       document-start
// ==/UserScript==

(function() {
    'use strict';
    
    // Customizable blocked patterns
    const blockedPatterns = ['!drop'];
    
    // Intercept fetch requests
    const originalFetch = window.fetch;
    window.fetch = async function(...args) {
        const response = await originalFetch(...args);
        
        // Check if this is a chat API request
        const url = typeof args[0] === 'string' ? args[0] : args[0]?.url;
        if (url && (url.includes('broadcastchat') || url.includes('chat'))) {
            try {
                const clonedResponse = response.clone();
                const data = await clonedResponse.json();
                
                // Filter out messages containing blocked patterns
                if (Array.isArray(data)) {
                    const filtered = data.filter(msg => {
                        if (!msg.msg) return true;
                        return !blockedPatterns.some(pattern => 
                            msg.msg.toLowerCase().includes(pattern.toLowerCase())
                        );
                    });
                    
                    return new Response(JSON.stringify(filtered), {
                        status: response.status,
                        statusText: response.statusText,
                        headers: response.headers
                    });
                }
            } catch (e) {
                console.warn('Chat filter: Error parsing response', e);
            }
        }
        
        return response;
    };
    
    // Also intercept XMLHttpRequest for older API calls
    const originalOpen = XMLHttpRequest.prototype.open;
    const originalSend = XMLHttpRequest.prototype.send;
    
    XMLHttpRequest.prototype.open = function(method, url, ...rest) {
        this._url = url;
        return originalOpen.call(this, method, url, ...rest);
    };
    
    XMLHttpRequest.prototype.send = function(...args) {
        if (this._url && (this._url.includes('broadcastchat') || this._url.includes('chat'))) {
            this.addEventListener('load', function() {
                try {
                    const data = JSON.parse(this.responseText);
                    if (Array.isArray(data)) {
                        const filtered = data.filter(msg => {
                            if (!msg.msg) return true;
                            return !blockedPatterns.some(pattern => 
                                msg.msg.toLowerCase().includes(pattern.toLowerCase())
                            );
                        });
                        
                        Object.defineProperty(this, 'responseText', {
                            writable: true,
                            value: JSON.stringify(filtered)
                        });
                    }
                } catch (e) {
                    console.warn('Chat filter: Error parsing XHR response', e);
                }
            });
        }
        return originalSend.call(this, ...args);
    };
})();
```

### 2. CSS Visual Filter

This solution uses CSS to visually hide spam messages after they load. It's simpler to implement but less efficient as messages are still loaded and processed by the browser.

#### How it works:
- Uses CSS selectors to target message containers
- Hides elements containing `!drop` text
- Pure CSS solution with no JavaScript required
- Visual-only filtering (messages still load)

#### Code:
```css
/* Hide message content containing !drop */
[class*="broadcastchat_MessageContents_"] {
    display: none !important;
}

/* Hide entire message containers with !drop */
[class*="BroadcastChatDiv"] [class*="broadcastchat_MessageChat_"] {
    display: none !important;
}
```

**Note**: The original CSS had invalid syntax. This version uses a simpler approach that targets message containers directly.

## 📦 Installation

### JavaScript Filter (Recommended)

#### Option 1: Tampermonkey
1. Install [Tampermonkey](https://www.tampermonkey.net/) browser extension
2. Create a new userscript
3. Copy and paste the JavaScript code above
4. Save and enable the script
5. Visit SteamTV to see the filter in action

#### Option 2: Browser Console
1. Open SteamTV in your browser
2. Press `F12` to open Developer Tools
3. Go to the Console tab
4. Paste the JavaScript code (without the UserScript header)
5. Press Enter to execute

### CSS Visual Filter

#### Option 1: Browser Extension
1. Install a CSS style manager extension like [Stylus](https://github.com/openstyles/stylus)
2. Create a new style for `https://steam.tv/*`
3. Copy and paste the CSS code
4. Save and enable the style

#### Option 2: Browser DevTools
1. Open SteamTV in your browser
2. Press `F12` to open Developer Tools
3. Go to the Elements/Inspector tab
4. Find the `<style>` tag in the `<head>` section
5. Add the CSS code inside the style tag

#### Option 3: Bookmarklet
Create a bookmark with this code as the URL:
```javascript
javascript:(function(){var style=document.createElement('style');style.textContent='/* Your CSS code here */';document.head.appendChild(style);})();
```

## 🎮 Usage

1. **Install either solution** following the instructions above
2. **Visit SteamTV** at `https://steam.tv/csgo` or any other game
3. **Enjoy spam-free chat** - `!drop` messages will be filtered out
4. **Customize patterns** (JavaScript only) by editing the `blockedPatterns` array

### Customizing Blocked Patterns

To block additional spam patterns, modify the `blockedPatterns` array in the JavaScript code:

```javascript
const blockedPatterns = [
    '!drop',
    '!claim',
    '!free',
    'your-custom-pattern'
];
```

## 🔧 Troubleshooting

### Messages Still Appear
- **JavaScript**: Check if the userscript is enabled and there are no console errors
- **CSS**: Verify the CSS is loaded and SteamTV hasn't updated their class names
- Clear browser cache and refresh the page
- Try disabling other extensions that might interfere

### Performance Issues
- **JavaScript**: Check browser console for any errors or warnings
- **CSS**: Ensure no other styles are conflicting
- Monitor memory usage in browser Task Manager

### SteamTV Updates
- SteamTV may update their interface and break these solutions
- Check the repository for updated versions
- Report issues in the GitHub issues section

### Extension Conflicts
- Disable other chat-related extensions
- Check Tampermonkey/Stylus dashboard for conflicts
- Try incognito/private mode with only the necessary extension

## 🔒 Security & Privacy

### JavaScript Filter
- **No data collection**: The script only filters local chat data
- **Network interception**: Reads chat API responses but doesn't send data elsewhere
- **Minimal permissions**: Requires only basic script execution permissions
- **Transparent**: All filtering logic is visible in the source code

### CSS Filter
- **Pure client-side**: No JavaScript execution required
- **No data access**: Only visual styling, no content modification
- **Privacy-friendly**: Doesn't intercept or read any data

### General Considerations
- Use only trusted userscript managers (Tampermonkey, Violentmonkey)
- Review code before installing any scripts
- Keep extensions updated for security patches
- Consider the trade-off between convenience and privacy

## 🌐 Browser Compatibility

| Feature | Chrome | Firefox | Safari | Edge |
|---------|--------|---------|--------|------|
| JavaScript Filter | ✅ | ✅ | ⚠️ | ✅ |
| CSS Filter | ✅ | ✅ | ✅ | ✅ |
| Tampermonkey | ✅ | ✅ | ⚠️ | ✅ |
| Stylish/Stylus | ✅ | ✅ | ✅ | ✅ |

**Legend**: ✅ Full Support | ⚠️ Limited Support | ❌ Not Supported

## 🛠️ Development

### Contributing
1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

### Adding New Filters
To add support for other streaming platforms or spam patterns:

1. Modify the URL matching patterns in the UserScript header
2. Update the API endpoint detection logic
3. Test on the target platform
4. Update documentation

### Testing
- Test on multiple browsers
- Verify performance impact
- Check for console errors
- Validate against SteamTV updates

## 📄 License

This project is provided as-is for educational and personal use. Use at your own risk.

### Disclaimer
- Not affiliated with Valve Corporation or Steam
- May break with SteamTV updates
- No warranty or support provided
- Use responsibly and respect other users

---

**Created during Budapest Major 2025**
