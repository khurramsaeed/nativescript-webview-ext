# @nota/nativescript-webview-ext

Extended WebView for NativeScript which adds "x-local"-custom-scheme for loading local-files, handle events between WebView and NativeScript, JavaScript execution, injecting CSS and JS-files.
Supports Android 19+ and iOS9+.

## Features
* Adds a custom-scheme handler for x-local:// to the webview for loading of resources inside the webview.
    * Note: For iOS 11+ WKWebView is used, but for iOS <11 UIWebView is used
* Adds support for capturing URLs This allows the app to open external links in an external browser and handle tel-links
* Adds functions:
    - `executeJavaScript(code: string)` for executing JavaScript-code and getting result.
    - `executePromise(code: string)` for calling promises and getting the result.
    - `getTitle()` returns document.title.
* Listen for events inside the webview from nativescript
* Listen for events from the webview in nativescript.
* Adds functions to load `css`- and `javascript`-files in the running webpage.
    * Supports loading into the current page and setting up auto-loading for every loaded page.
* Supports:
    * Android 19+
    * iOS 9+

### Possible features to come:

* Cookie helpers
* Setting view-port metadata
* Share cache with native-layer?

### Android
* Setup remote debugging, e.g calling `android.webkit.WebView.setWebContentsDebuggingEnabled(true);`

#### iOS
* Optionas for native scrolllView.

## Installation

Describe your plugin installation steps. Ideally it would be something like:

```bash
tns plugin add @nota/nativescript-webview-ext
```

## Usage

## Limitations

In order to intercept requests for the custom scheme, we use `UIWebView` for iOS 9 and 10 and `WKWebView` for iOS 11+.

iOS 11 added support for setting a `WKURLSchemeHandler` on the `WKWebView`.
Prior to iOS 11 there isn't support for intercepting the URL with `WKWebView`, so we use a custom `NSURLProtocol` + `UIWebView`.

### Important:
The custom `NSURLProtocol` used with UIWebView is shared with all instances of the WebViewExt, so mapping `x-local://local-filename.js` => `file://app/full/path/local-filename.js` is shared between them.

## API

### NativeScript View

| Property | Value | Description |
| --- | --- | --- |
| isUIWebView | true if `iOS <11` | Is the native webview an UIWebView? |
| isWkWebView | true if `iOS >=11` | Is the native webview an WKWebView? |

| Function | Description |
| --- | --- |
| registerLocalResource(resourceName: string, path: string): void; | Map the "x-local://{resourceName}" => "{path}". |
| unregisterLocalResource(resourceName: string): void; | Removes the mapping from "x-local://{resourceName}" => "{path}" |
| getRegistretLocalResource(resourceName: string): void; | Get the mapping from "x-local://{resourceName}" => "{path}" |
| loadJavaScriptFile(scriptName: string, filepath: string) | Inject a javascript-file into the webview. Should be called after the `loadFinishedEvent` |
| loadStyleSheetFile(stylesheetName: string, filepath: string, insertBefore: boolean) | Loads a CSS-file into document.head. If before is true, it will be added to the top of document.head otherwise as the last element |
| loadJavaScriptFiles(files: {resourceName: string, filepath: string}[]) | Inject multiple javascript-files into the webview. Should be called after the `loadFinishedEvent` |
| loadStyleSheetFiles(files: {resourceName: string, filepath: string, insertBefore: boolean}[]) | Loads multiple CSS-files into the document.head. If before is true, it will be added to the top of document.head otherwise as the last element |
| autoLoadJavaScriptFile | Register a JavaScript-file to be injected on `loadFinishedEvent`. If a page is already loaded, the script will be injected into the current page. |
| autoLoadStyleSheetFile | Register a CSS-file to be injected on `loadFinishedEvent`. If a page is already loaded, the CSS-file will be injected into the current page. |
| executeJavaScript(scriptCode: string) | Execute JavaScript in the webpage. *Note:* scriptCode should be ES5 compatible, or it might not work on iOS < 11. |
| executePromise(scriptCode: string, timeout: number = 500) | Run a promise inside the webview. *Note:* Executing scriptCode must return a promise. |
| getTitle() | Returns a promise with the current document title. |

#### Examples

**Executing javascript**:
```typescript
const scriptCode = `document.title`;

webview.executeJavaScript(scriptCode).then((title) => {
    console.log(`document title is: ${title}`);
});
```

**Executing a promise:**
```typescript
const scriptCode = `new Promise(function(resolve) {
    resolve(document.title);
});`;

webview.executePromise(scriptCode).then((title) => {
    console.log(`document title is: ${title}`);
});
```

### WebView

| Function | Description |
| --- | --- |
| window.nsWebViewBridge.on(eventName: string, cb: (data: any) => void) | Registers handlers for events from the native layer. |
| window.nsWebViewBridge.off(eventName: string, cb?: (data: any) => void) | Deregister handlers for events from the native layer. |
| window.nsWebViewBridge.emit(eventName: string, data: any) | Emits event to NativeScript layer. Will be emitted on the WebViewExt as any other event, data will be a part of the WebViewEventData-object |

### HTML

These examples assumes that `locale-stylesheet.css` and `locale-javascript.js` have been mapped via `webview.registerLocalResource(name: string, path: string)`.

Loading CSS-file via the `x-local`-scheme:
```html
    <link rel="stylesheet" href="x-local://locale-stylesheet.css" />
```

Loading JavaScript-file via the `x-local`-scheme:
```html
    <script src="x-local://locale-javascript.js"></script>
```

## License

Apache License Version 2.0, January 2004

## About Nota

Nota is the Danish Library and Expertise Center for people with print disabilities.
To become a member of Nota you must be able to document that you cannot read ordinary printed text. Members of Nota are visually impaired, dyslexic or otherwise impaired.
Our purpose is to ensure equal access to knowledge, community participation and experiences for people who're unable to read ordinary printed text.
