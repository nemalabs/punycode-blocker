# Punycode Blocker

English | [日本語](README.ja.md)

A Chrome extension (Manifest V3) that blocks every request to a domain containing an internationalized (Punycode, `xn--`) label.

## Why

Internationalized domain names can contain characters that look almost identical to ASCII letters. This is used in phishing, where a domain is made to look like a well-known site (an IDN homograph attack). This extension takes a simple approach: it blocks all IDN domains, whether legitimate or not.

## How it works

When Chrome handles a URL, it converts a domain name containing non-ASCII characters into its Punycode form, where each such label starts with `xn--`. This extension uses the `declarativeNetRequest` API, and Chrome blocks any request whose host has a label starting with `xn--`.

The rule is defined in [`rules.json`](rules.json):

```
^[a-z][a-z0-9+.-]*://([^/?#@]*@)?([^/?#:@]*\.)?xn--
```

| URL | Result |
| --- | --- |
| `https://xn--abc.example/` | Blocked |
| `https://www.xn--abc.example/page` | Blocked (subdomain) |
| `https://shop.example.xn--abc:8443/` | Blocked (top-level domain, with port) |
| `wss://user@xn--abc.example/socket` | Blocked (other schemes, with user info) |
| `https://example.com/xn--abc` | Not blocked (`xn--` in the path) |
| `https://example.com/?q=xn--abc` | Not blocked (`xn--` in the query) |
| `https://axn--b.com/` | Not blocked (label does not start with `xn--`) |

The rule applies to page navigation, frames, scripts, stylesheets, images, fonts, XHR/fetch, WebSocket, media, and other resource types.

## Limitations

- Legitimate websites with internationalized domain names are also blocked. There is no allowlist.
- Lookalike domains made only of ASCII characters (for example, `rn` in place of `m`) are not blocked.
- Only Chrome has been considered. Behavior in other browsers has not been tested.

## Installation

1. Clone or download this repository.
2. Open `chrome://extensions` in Chrome.
3. Turn on **Developer mode** in the top-right corner.
4. Click **Load unpacked** and select the folder of this repository.

To visit an IDN site temporarily, turn the extension off on `chrome://extensions`.

## Permissions

The extension requests only the `declarativeNetRequest` permission. It contains no JavaScript; Chrome itself applies the rule in `rules.json`. The extension does not read page content or browsing history.

## Files

| File | Description |
| --- | --- |
| `manifest.json` | Extension manifest |
| `rules.json` | Block rule for `declarativeNetRequest` |
