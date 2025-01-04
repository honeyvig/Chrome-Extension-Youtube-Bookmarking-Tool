# Chrome-Extension-Youtube-Bookmarking-Tool
The Chrome extension will help users bookmark and organize YouTube videos with specific metadata. Here’s the functionality I need:

1. Tag & Save Button:
Add a button on YouTube videos to capture:
Video Title.
Video URL.
Current Timestamp.
Allow users to assign tags and choose a folder when saving the video.

2. Search Functionality:
A search bar where users can search by:
Video Title.
Tags.
Folders.
Clicking a search result:
Opens the video on YouTube at the saved timestamp (for videos).
Filters results by the tag or folder (for tags or folders).

3. Library:
A pop-up library interface accessible via the Chrome extension icon.
Features include:
Folder organization (create/edit/delete folders).
Viewing and managing bookmarks within folders.
Filtering by tags within folders.

4. Bookmark Management:
Save data for each bookmark:
Title, URL, Timestamp, Tag, Folder.
Edit/Delete bookmarks as needed.

5. Optional Features for Future Updates:
Export Bookmarks: Save bookmarks as a CSV file (with Title, URL, Timestamp, Tags, and Folder).

Drag-and-Drop Functionality:
Allow users to reorder videos within folders or move them between folders.

This is not essential for the initial version but would be a great addition later. Let me know if you have experience implementing this feature and how much additional time it might require.
-----------
To build the Chrome extension that allows users to bookmark and organize YouTube videos with specific metadata, we need to break the project down into several core components. These components will include adding a button to YouTube videos, storing bookmarks with metadata, implementing a search functionality, and creating a library interface for managing bookmarks.
Core Features:

    Tag & Save Button: The extension will capture video metadata (Title, URL, Timestamp) and allow users to tag and organize the video into folders.
    Search Functionality: Users will be able to search by title, tags, or folders, and click a search result to jump to the saved timestamp.
    Library: A pop-up library will display saved bookmarks and allow users to organize, filter, and manage them.
    Bookmark Management: Users can save, edit, and delete bookmarks.

Manifest File (manifest.json):

The manifest file defines the extension's permissions, content scripts, popup, and background scripts.

{
  "manifest_version": 3,
  "name": "YouTube Video Bookmark Manager",
  "version": "1.0",
  "description": "Bookmark and organize YouTube videos with metadata.",
  "permissions": [
    "storage",
    "activeTab"
  ],
  "background": {
    "service_worker": "background.js"
  },
  "content_scripts": [
    {
      "matches": ["https://www.youtube.com/*"],
      "js": ["content.js"]
    }
  ],
  "action": {
    "default_popup": "popup.html",
    "default_icon": {
      "16": "icons/icon16.png",
      "48": "icons/icon48.png",
      "128": "icons/icon128.png"
    }
  },
  "icons": {
    "16": "icons/icon16.png",
    "48": "icons/icon48.png",
    "128": "icons/icon128.png"
  }
}

Background Script (background.js):

Handles storing and managing the bookmarks in localStorage or chrome.storage.

// Fetch stored bookmarks or initialize if empty
chrome.storage.local.get(['bookmarks'], function(result) {
  if (!result.bookmarks) {
    chrome.storage.local.set({ bookmarks: [] });
  }
});

// Save a new bookmark
function saveBookmark(bookmark) {
  chrome.storage.local.get(['bookmarks'], function(result) {
    const bookmarks = result.bookmarks || [];
    bookmarks.push(bookmark);
    chrome.storage.local.set({ bookmarks: bookmarks });
  });
}

// Get all bookmarks
function getBookmarks(callback) {
  chrome.storage.local.get(['bookmarks'], function(result) {
    callback(result.bookmarks || []);
  });
}

// Delete a bookmark by index
function deleteBookmark(index) {
  chrome.storage.local.get(['bookmarks'], function(result) {
    const bookmarks = result.bookmarks || [];
    bookmarks.splice(index, 1);
    chrome.storage.local.set({ bookmarks: bookmarks });
  });
}

chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.action === 'saveBookmark') {
    saveBookmark(message.bookmark);
  } else if (message.action === 'getBookmarks') {
    getBookmarks(sendResponse);
    return true; // To respond asynchronously
  } else if (message.action === 'deleteBookmark') {
    deleteBookmark(message.index);
  }
});

Content Script (content.js):

Adds the "Tag & Save" button to the YouTube page and captures metadata such as video title, URL, and timestamp.

// Add a "Tag & Save" button to the YouTube page
function addSaveButton() {
  const button = document.createElement('button');
  button.innerText = 'Tag & Save';
  button.style.position = 'absolute';
  button.style.bottom = '10px';
  button.style.right = '10px';
  button.style.backgroundColor = '#FF4500';
  button.style.color = 'white';
  button.style.border = 'none';
  button.style.padding = '10px 20px';
  button.style.fontSize = '14px';
  button.style.borderRadius = '5px';

  button.addEventListener('click', () => {
    const videoTitle = document.title;
    const videoUrl = window.location.href;
    const timestamp = getTimestamp();

    const tag = prompt('Enter tags for this video (comma separated):');
    const folder = prompt('Enter a folder name for this video:');

    const bookmark = { title: videoTitle, url: videoUrl, timestamp: timestamp, tags: tag, folder: folder };

    chrome.runtime.sendMessage({ action: 'saveBookmark', bookmark: bookmark });
  });

  document.body.appendChild(button);
}

// Get current timestamp in the video
function getTimestamp() {
  const videoElement = document.querySelector('video');
  const currentTime = videoElement ? videoElement.currentTime : 0;
  return Math.floor(currentTime);
}

// Only add the button after the page is fully loaded
window.addEventListener('load', addSaveButton);

Popup HTML (popup.html):

A simple popup interface to view, search, and manage saved bookmarks.

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>YouTube Video Bookmark Manager</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      width: 300px;
      padding: 10px;
    }
    .bookmark {
      margin-bottom: 10px;
      padding: 5px;
      border: 1px solid #ddd;
      border-radius: 5px;
    }
    .bookmark a {
      text-decoration: none;
      color: #000;
    }
  </style>
</head>
<body>
  <h3>Your Bookmarks</h3>
  <input type="text" id="search" placeholder="Search by Title/Tag/Folder" />
  <div id="bookmark-list"></div>

  <script src="popup.js"></script>
</body>
</html>

Popup JS (popup.js):

Handles displaying bookmarks, searching, and interacting with the background script to retrieve saved bookmarks.

// Fetch bookmarks and display them in the popup
function displayBookmarks(bookmarks) {
  const bookmarkList = document.getElementById('bookmark-list');
  bookmarkList.innerHTML = '';

  bookmarks.forEach((bookmark, index) => {
    const div = document.createElement('div');
    div.classList.add('bookmark');
    
    const link = document.createElement('a');
    link.href = `${bookmark.url}&t=${bookmark.timestamp}s`; // Jump to timestamp
    link.innerText = `${bookmark.title} (${bookmark.timestamp}s)`;
    link.target = '_blank';

    div.appendChild(link);
    bookmarkList.appendChild(div);
  });
}

// Search for bookmarks based on user input
function searchBookmarks() {
  const searchText = document.getElementById('search').value.toLowerCase();

  chrome.runtime.sendMessage({ action: 'getBookmarks' }, function(bookmarks) {
    const filteredBookmarks = bookmarks.filter(bookmark => {
      return bookmark.title.toLowerCase().includes(searchText) ||
             bookmark.tags.toLowerCase().includes(searchText) ||
             bookmark.folder.toLowerCase().includes(searchText);
    });

    displayBookmarks(filteredBookmarks);
  });
}

// Event listener for searching
document.getElementById('search').addEventListener('input', searchBookmarks);

// Fetch and display bookmarks when the popup is opened
chrome.runtime.sendMessage({ action: 'getBookmarks' }, function(bookmarks) {
  displayBookmarks(bookmarks);
});

Styling:

The style is kept minimal, but you can enhance it based on your preferences or integrate a CSS framework.

/* styles.css - Simple styling */
body {
  font-family: Arial, sans-serif;
  width: 300px;
  padding: 10px;
}

#bookmark-list {
  margin-top: 10px;
}

.bookmark {
  margin-bottom: 10px;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
}

.bookmark a {
  color: #000;
  text-decoration: none;
}

Key Features:

    Tag & Save Button: Injected into YouTube pages, it allows users to save the video with metadata such as tags, title, URL, and timestamp.
    Popup Interface: Users can view, search, and manage their saved bookmarks in the popup.
    Bookmark Storage: Bookmarks are stored using Chrome’s chrome.storage.local API.
    Search Functionality: Search by video title, tags, or folders, with the ability to click on a result and jump to the saved timestamp.

Optional Future Features:

    Export Bookmarks: Adding functionality to export the saved bookmarks to a CSV file.
    Drag-and-Drop: Allowing users to reorder or move bookmarks between folders.

Testing:

    Testing Search: Ensure the search is filtering results correctly.
    Testing Bookmark Storage: Verify bookmarks are correctly stored and retrieved from Chrome storage.
    Testing Timestamp: Ensure that clicking on a bookmark correctly jumps to the saved timestamp.

Conclusion:

This Chrome extension allows users to organize YouTube videos with tags, titles, URLs, and timestamps, and offers features like searching and managing bookmarks in a simple and efficient way. It’s extensible and can be improved with additional features such as CSV export or drag-and-drop support later on.
