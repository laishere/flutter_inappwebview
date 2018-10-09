# Flutter InAppBrowser Plugin

[![Pub](https://img.shields.io/pub/v/flutter_inappbrowser.svg)](https://pub.dartlang.org/packages/flutter_inappbrowser)

A Flutter plugin that allows you to open an in-app browser window.
This plugin is inspired by the popular [cordova-plugin-inappbrowser](https://github.com/apache/cordova-plugin-inappbrowser)!

## Getting Started

For help getting started with Flutter, view our online
[documentation](https://flutter.io/).

For help on editing plugin code, view the [documentation](https://flutter.io/developing-packages/#edit-plugin-package).

## Installation
First, add `flutter_inappbrowser` as a [dependency in your pubspec.yaml file](https://flutter.io/using-packages/).

## Usage
Classes:
- [InAppBrowser](#inappbrowser): Native WebView.
- [ChromeSafariBrowser](#chromesafaribrowser): [Chrome Custom Tabs](https://developer.android.com/reference/android/support/customtabs/package-summary) on Android / [SFSafariViewController](https://developer.apple.com/documentation/safariservices/sfsafariviewcontroller) on iOS.

Screenshots [here](#screenshots).

### `InAppBrowser` class
Create a Class that extends the `InAppBrowser` Class in order to override the callbacks to manage the browser events.
Example:
```dart
import 'package:flutter/material.dart';
import 'package:flutter_inappbrowser/flutter_inappbrowser.dart';

class MyInAppBrowser extends InAppBrowser {

  @override
  void onLoadStart(String url) {
    print("\n\nStarted $url\n\n");
  }

  @override
  Future onLoadStop(String url) async {
    print("\n\nStopped $url\n\n");
    // print body html
    print(await this.injectScriptCode("document.body.innerHTML"));

    // console messages
    await this.injectScriptCode("console.log({'testObject': 5});"); // the message will be: [object Object]
    await this.injectScriptCode("console.log('testObjectStringify', JSON.stringify({'testObject': 5}));"); // the message will be: testObjectStringify {"testObject": 5}
    await this.injectScriptCode("console.error('testError', false);"); // the message will be: testError false
    
    // add jquery library and custom javascript
    await this.injectScriptFile("https://code.jquery.com/jquery-3.3.1.min.js");
    this.injectScriptCode("""
      \$( "body" ).html( "Next Step..." )
    """);
    
    // add custom css
    this.injectStyleCode("""
    body {
      background-color: #3c3c3c !important;
    }
    """);
    this.injectStyleFile("https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css");
  }

  @override
  void onLoadError(String url, int code, String message) {
    print("\n\nCan't load $url.. Error: $message\n\n");
  }

  @override
  void onExit() {
    print("\n\nBrowser closed!\n\n");
  }
  
  @override
  void shouldOverrideUrlLoading(String url) {
    print("\n\n override $url\n\n");
    this.loadUrl(url);
  }

  @override
  void onConsoleMessage(ConsoleMessage consoleMessage) {
    print("""
    console output:
      sourceURL: ${consoleMessage.sourceURL}
      lineNumber: ${consoleMessage.lineNumber}
      message: ${consoleMessage.message}
      messageLevel: ${consoleMessage.messageLevel}
    """);
  }

}

MyInAppBrowser inAppBrowser = new MyInAppBrowser();

void main() => runApp(new MyApp());

class MyApp extends StatefulWidget {
  @override
  _MyAppState createState() => new _MyAppState();
}

class _MyAppState extends State<MyApp> {

  @override
  void initState() {
    super.initState();
  }

  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
      home: new Scaffold(
        appBar: new AppBar(
          title: const Text('Flutter InAppBrowser Plugin example app'),
        ),
        body: new Center(
          child: new RaisedButton(onPressed: () {
            inAppBrowser.open("https://flutter.io/", options: {
               "useShouldOverrideUrlLoading": true
             });
          },
          child: Text("Open InAppBrowser")
          ),
        ),
      ),
    );
  }
}
```

#### Future\<void\> InAppBrowser.open

Opens a URL in a new InAppBrowser instance or the system browser.

```dart
inAppBrowser.open(String url, {Map<String, String> headers = const {}, String target = "_self", Map<String, dynamic> options = const {}});
```

Opens an `url` in a new `InAppBrowser` instance or the system browser.

- `url`: The `url` to load. Call `encodeUriComponent()` on this if the `url` contains Unicode characters.

- `headers`: The additional headers to be used in the HTTP request for this URL, specified as a map from name to value.

- `target`: The target in which to load the `url`, an optional parameter that defaults to `_self`.

  - `_self`: Opens in the `InAppBrowser`.
  - `_blank`: Opens in the `InAppBrowser`.
  - `_system`: Opens in the system's web browser.

- `options`: Options for the `InAppBrowser`.

  All platforms support:
  - __useShouldOverrideUrlLoading__: Set to `true` to be able to listen at the `shouldOverrideUrlLoading` event. The default value is `false`.
  - __clearCache__: Set to `true` to have all the browser's cache cleared before the new window is opened. The default value is `false`.
  - __userAgent___: Set the custom WebView's user-agent.
  - __javaScriptEnabled__: Set to `true` to enable JavaScript. The default value is `true`.
  - __javaScriptCanOpenWindowsAutomatically__: Set to `true` to allow JavaScript open windows without user interaction. The default value is `false`.
  - __hidden__: Set to `true` to create the browser and load the page, but not show it. The `onLoadStop` event fires when loading is complete. Omit or set to `false` (default) to have the browser open and load normally.
  - __toolbarTop__: Set to `false` to hide the toolbar at the top of the WebView. The default value is `true`.
  - __toolbarTopBackgroundColor__: Set the custom background color of the toolbat at the top.
  - __hideUrlBar__: Set to `true` to hide the url bar on the toolbar at the top. The default value is `false`.
  - __mediaPlaybackRequiresUserGesture__: Set to `true` to prevent HTML5 audio or video from autoplaying. The default value is `true`.
  
  **Android** supports these additional options:
  
  - __hideTitleBar__: Set to `true` if you want the title should be displayed. The default value is `false`.
  - __closeOnCannotGoBack__: Set to `false` to not close the InAppBrowser when the user click on the back button and the WebView cannot go back to the history. The default value is `true`.
  - __clearSessionCache__: Set to `true` to have the session cookie cache cleared before the new window is opened.
  - __builtInZoomControls__: Set to `true` if the WebView should use its built-in zoom mechanisms. The default value is `false`.
  - __supportZoom__: Set to `false` if the WebView should not support zooming using its on-screen zoom controls and gestures. The default value is `true`.
  - __databaseEnabled__: Set to `true` if you want the database storage API is enabled. The default value is `false`.
  - __domStorageEnabled__: Set to `true` if you want the DOM storage API is enabled. The default value is `false`.
  - __useWideViewPort__: Set to `true` if the WebView should enable support for the "viewport" HTML meta tag or should use a wide viewport. When the value of the setting is false, the layout width is always set to the width of the WebView control in device-independent (CSS) pixels. When the value is true and the page contains the viewport meta tag, the value of the width specified in the tag is used. If the page does not contain the tag or does not provide a width, then a wide viewport will be used. The default value is `true`.
  - __safeBrowsingEnabled__: Set to `true` if you want the Safe Browsing is enabled. Safe Browsing allows WebView to protect against malware and phishing attacks by verifying the links. The default value is `true`.
  - __progressBar__: Set to `false` to hide the progress bar at the bottom of the toolbar at the top. The default value is `true`.

  **iOS** supports these additional options:
 
  - __disallowOverScroll__: Set to `true` to disable the bouncing of the WebView when the scrolling has reached an edge of the content. The default value is `false`.
  - __toolbarBottom__: Set to `false` to hide the toolbar at the bottom of the WebView. The default value is `true`.
  - __toolbarBottomBackgroundColor__: Set the custom background color of the toolbat at the bottom.
  - __toolbarBottomTranslucent__: Set to `true` to set the toolbar at the bottom translucent. The default value is `true`.
  - __closeButtonCaption__: Set the custom text for the close button.
  - __closeButtonColor__: Set the custom color for the close button.
  - __presentationStyle__: Set the custom modal presentation style when presenting the WebView. The default value is `0 //fullscreen`. See [UIModalPresentationStyle](https://developer.apple.com/documentation/uikit/uimodalpresentationstyle) for all the available styles. 
  - __transitionStyle__: Set to the custom transition style when presenting the WebView. The default value is `0 //crossDissolve`. See [UIModalTransitionStyle](https://developer.apple.com/documentation/uikit/uimodaltransitionStyle) for all the available styles.
  - __enableViewportScale__: Set to `true` to allow a viewport meta tag to either disable or restrict the range of user scaling. The default value is `false`.
  - __keyboardDisplayRequiresUserAction__: Set to `true` if you want the user must explicitly tap the elements in the WebView to display the keyboard (or other relevant input view) for that element. When set to `false`, a focus event on an element causes the input view to be displayed and associated with that element automatically. The default value is `true`.
  - __suppressesIncrementalRendering__: Set to `true` if you want the WebView suppresses content rendering until it is fully loaded into memory.. The default value is `false`.
  - __allowsAirPlayForMediaPlayback__: Set to `true` to allow AirPlay. The default value is `true`.
  - __allowsBackForwardNavigationGestures__: Set to `true` to allow the horizontal swipe gestures trigger back-forward list navigations. The default value is `true`.
  - __allowsLinkPreview__: Set to `true` to allow that pressing on a link displays a preview of the destination for the link. The default value is `true`.
  - __ignoresViewportScaleLimits__: Set to `true` if you want that the WebView should always allow scaling of the webpage, regardless of the author's intent. The ignoresViewportScaleLimits property overrides the `user-scalable` HTML property in a webpage. The default value is `false`.
  - __allowsInlineMediaPlayback__: Set to `true` to allow HTML5 media playback to appear inline within the screen layout, using browser-supplied controls rather than native controls. For this to work, add the `webkit-playsinline` attribute to any `<video>` elements. The default value is `false`.
  - __allowsPictureInPictureMediaPlayback__: Set to `true` to allow HTML5 videos play picture-in-picture. The default value is `true`.
  - __spinner__: Set to `false` to hide the spinner when the WebView is loading a page. The default value is `true`.
  
Example:
```dart
inAppBrowser.open('https://flutter.io/', options: {
  "useShouldOverrideUrlLoading": true,
  "clearCache": true,
  "disallowOverScroll": true,
  "domStorageEnabled": true,
  "supportZoom": false,
  "toolbarBottomTranslucent": false,
  "allowsLinkPreview": false
});
``` 

#### Events

Event fires when the `InAppBrowser` starts to load an `url`.
```dart
  @override
  void onLoadStart(String url) {
  
  }
```

Event fires when the `InAppBrowser` finishes loading an `url`.
```dart
  @override
  void onLoadStop(String url) {
  
  }
```

Event fires when the `InAppBrowser` encounters an error loading an `url`.
```dart
  @override
  void onLoadError(String url, String code, String message) {
  
  }
```

Event fires when the `InAppBrowser` window is closed.
```dart
  @override
  void onExit() {
  
  }
```

Event fires when the `InAppBrowser` webview receives a `ConsoleMessage`.
```dart
  @override
  void onConsoleMessage(ConsoleMessage consoleMessage) {

  }
```

Give the host application a chance to take control when a URL is about to be loaded in the current WebView.
In order to be able to listen this event, you need to set `useShouldOverrideUrlLoading` option to `true`.
```dart
  @override
  void shouldOverrideUrlLoading(String url) {

  }
```

#### Future\<void\> InAppBrowser.loadUrl

Loads the given `url` with optional `headers` specified as a map from name to value.

```dart
inAppBrowser.loadUrl(String url, {Map<String, String> headers = const {}});
```

#### Future\<void\> InAppBrowser.show

Displays an `InAppBrowser` window that was opened hidden. Calling this has no effect if the `InAppBrowser` was already visible.

```dart
inAppBrowser.show();
``` 

#### Future\<void\> InAppBrowser.hide

Hides the `InAppBrowser` window. Calling this has no effect if the `InAppBrowser` was already hidden.

```dart
inAppBrowser.hide();
``` 

#### Future\<void\> InAppBrowser.close

Closes the `InAppBrowser` window.

```dart
inAppBrowser.close();
``` 

#### Future\<void\> InAppBrowser.reload

Reloads the `InAppBrowser` window.

```dart
inAppBrowser.reload();
``` 

#### Future\<void\> InAppBrowser.goBack

Goes back in the history of the `InAppBrowser` window.

```dart
inAppBrowser.goBack();
``` 

#### Future\<void\> InAppBrowser.goForward

Goes forward in the history of the `InAppBrowser` window.

```dart
inAppBrowser.goForward();
``` 

#### Future\<bool\> InAppBrowser.isLoading

Check if the Web View of the `InAppBrowser` instance is in a loading state.

```dart
inAppBrowser.isLoading();
``` 

#### Future\<void\> InAppBrowser.stopLoading

Stops the Web View of the `InAppBrowser` instance from loading.

```dart
inAppBrowser.stopLoading();
``` 

#### Future\<bool\> InAppBrowser.isHidden

Check if the Web View of the `InAppBrowser` instance is hidden.

```dart
inAppBrowser.isHidden();
``` 

#### Future\<String\> InAppBrowser.injectScriptCode

Injects JavaScript code into the `InAppBrowser` window and returns the result of the evaluation. (Only available when the target is set to `_blank` or to `_self`)

```dart
inAppBrowser.injectScriptCode(String source);
``` 

#### Future\<void\> InAppBrowser.injectScriptFile

Injects a JavaScript file into the `InAppBrowser` window. (Only available when the target is set to `_blank` or to `_self`)

```dart
inAppBrowser.injectScriptFile(String urlFile);
``` 

#### Future\<void\> InAppBrowser.injectStyleCode

Injects CSS into the `InAppBrowser` window. (Only available when the target is set to `_blank` or to `_self`)

```dart
inAppBrowser.injectStyleCode(String source);
``` 

#### Future\<void\> InAppBrowser.injectStyleFile

Injects a CSS file into the `InAppBrowser` window. (Only available when the target is set to `_blank` or to `_self`)

```dart
inAppBrowser.injectStyleFile(String urlFile);
``` 

### `ChromeSafariBrowser` class
Create a Class that extends the `ChromeSafariBrowser` Class in order to override the callbacks to manage the browser events. Example:
```dart
import 'package:flutter/material.dart';
import 'package:flutter_inappbrowser/flutter_inappbrowser.dart';

class MyInAppBrowser extends InAppBrowser {

  @override
  Future onLoadStart(String url) async {
    print("\n\nStarted $url\n\n");
  }

  @override
  Future onLoadStop(String url) async {
    print("\n\nStopped $url\n\n");
  }

  @override
  void onLoadError(String url, int code, String message) {
    print("\n\nCan't load $url.. Error: $message\n\n");
  }

  @override
  void onExit() {
    print("\n\nBrowser closed!\n\n");
  }
  
}

MyInAppBrowser inAppBrowserFallback = new MyInAppBrowser();

class MyChromeSafariBrowser extends ChromeSafariBrowser {
  
  MyChromeSafariBrowser(browserFallback) : super(browserFallback);

  @override
  void onOpened() {
    print("ChromeSafari browser opened");
  }

  @override
  void onLoaded() {
    print("ChromeSafari browser loaded");
  }

  @override
  void onClosed() {
    print("ChromeSafari browser closed");
  }
}

MyChromeSafariBrowser chromeSafariBrowser = new MyChromeSafariBrowser(inAppBrowserFallback);


void main() => runApp(new MyApp());

class MyApp extends StatefulWidget {
  @override
  _MyAppState createState() => new _MyAppState();
}

class _MyAppState extends State<MyApp> {

  @override
  void initState() {
    super.initState();
  }

  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
      home: new Scaffold(
        appBar: new AppBar(
          title: const Text('Flutter InAppBrowser Plugin example app'),
        ),
        body: new Center(
          child: new RaisedButton(onPressed: () {
            chromeSafariBrowser.open("https://flutter.io/", options: {
                  "addShareButton": false,
                  "toolbarBackgroundColor": "#000000",
                  "dismissButtonStyle": 1,
                  "preferredBarTintColor": "#000000",
                },
              optionsFallback: {
                "toolbarTopBackgroundColor": "#000000",
                "closeButtonCaption": "Close"
              });
          },
          child: Text("Open ChromeSafariBrowser")
          ),
        ),
      ),
    );
  }
}

```

#### Future\<void\> ChromeSafariBrowser.open
Opens an `url` in a new `ChromeSafariBrowser` instance or the system browser.

- `url`: The `url` to load. Call `encodeUriComponent()` on this if the `url` contains Unicode characters.

- `options`: Options for the `ChromeSafariBrowser`.

- `headersFallback`: The additional header of the `InAppBrowser` instance fallback to be used in the HTTP request for this URL, specified as a map from name to value.

- `optionsFallback`: Options used by the `InAppBrowser` instance fallback.

**Android** supports these options:

- __addShareButton__: Set to `false` if you don't want the default share button. The default value is `true`.
- __showTitle__: Set to `false` if the title shouldn't be shown in the custom tab. The default value is `true`.
- __toolbarBackgroundColor__: Set the custom background color of the toolbar.
- __enableUrlBarHiding__: Set to `true` to enable the url bar to hide as the user scrolls down on the page. The default value is `false`.
- __instantAppsEnabled__: Set to `true` to enable Instant Apps. The default value is `false`.

**iOS** supports these options:

- __entersReaderIfAvailable__: Set to `true` if Reader mode should be entered automatically when it is available for the webpage. The default value is `false`.
- __barCollapsingEnabled__: Set to `true` to enable bar collapsing. The default value is `false`.
- __dismissButtonStyle__: Set the custom style for the dismiss button. The default value is `0 //done`. See [SFSafariViewController.DismissButtonStyle](https://developer.apple.com/documentation/safariservices/sfsafariviewcontroller/dismissbuttonstyle) for all the available styles.
- __preferredBarTintColor__: Set the custom background color of the navigation bar and the toolbar.
- __preferredControlTintColor__: Set the custom color of the control buttons on the navigation bar and the toolbar.
- __presentationStyle__: Set the custom modal presentation style when presenting the WebView. The default value is `0 //fullscreen`. See [UIModalPresentationStyle](https://developer.apple.com/documentation/uikit/uimodalpresentationstyle) for all the available styles.
- __transitionStyle__: Set to the custom transition style when presenting the WebView. The default value is `0 //crossDissolve`. See [UIModalTransitionStyle](https://developer.apple.com/documentation/uikit/uimodaltransitionStyle) for all the available styles.

Example:
```dart
chromeSafariBrowser.open("https://flutter.io/", options: {
  "addShareButton": false,
  "toolbarBackgroundColor": "#000000",
  "dismissButtonStyle": 1,
  "preferredBarTintColor": "#000000",
});
```

#### Events

Event fires when the `ChromeSafariBrowser` is opened.
```dart
  @override
  void onOpened() {
  
  }
```

Event fires when the `ChromeSafariBrowser` is loaded.
```dart
  @override
  void onLoaded() {
  
  }
```

Event fires when the `ChromeSafariBrowser` is closed.
```dart
  @override
  void onClosed() {
  
  }
```

## Screenshots:

#### InAppBrowser
iOS:

![ios](https://user-images.githubusercontent.com/5956938/45934084-2a935400-bf99-11e8-9d71-9e1758b5b8c6.gif)

Android:

![android](https://user-images.githubusercontent.com/5956938/45934080-26ffcd00-bf99-11e8-8136-d39a81bd83e7.gif)

#### ChromeSafariBrowser
iOS:

![ios](https://user-images.githubusercontent.com/5956938/46532148-0c362e00-c8a0-11e8-9a0e-343e049dcf35.gif)

Android:

![android](https://user-images.githubusercontent.com/5956938/46532149-0c362e00-c8a0-11e8-8134-9af18f38a746.gif)