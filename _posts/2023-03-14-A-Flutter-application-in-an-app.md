---
layout: post
title: "A Flutter application in an app icon"
---

In this blog post, I will explain to you how I created the following tech demo:

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Okay, hear me out! Rendering a Flutter application in the app icon on macOS is the next big thing! <a href="https://t.co/7mzzL83oDO">pic.twitter.com/7mzzL83oDO</a></p>&mdash; Jonas Uekötter (@ue_man) <a href="https://twitter.com/ue_man/status/1620146406307823616?ref_src=twsrc%5Etfw">January 30, 2023</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

I have to confess, though, that it’s actually not rendered in the app
icon. Instead, the view of the Flutter application is captured and
streamed to the app icon.

### The basics

This paragraph covers the basics. If you’re already familiar with
MethodChannels and RepaintBoundaries, and macOSs NSDock API, you can
skip this.

MethodChannels are the mechanism to communicate between Dart code and
the underlying native code. In this case, it’s Swift. I won’t cover how
exactly it works, because it’s already 
<a href="https://docs.flutter.dev/development/platform-integration/platform-channels?tab=type-mappings-swift-tab" target="_blank">
explained in depth on the Flutter docs</a>.
We need the MethodChannels to send the screen of the Flutter application to the native code.

Capturing the screen of the Flutter application is done via <a
href="https://api.flutter.dev/flutter/widgets/RepaintBoundary-class.html" target="_blank">RepaintBoundaries</a>.
This nifty widget allows you to capture its child widgets as an image. If you capture the
child widgets every frame, it looks like a video stream of the
application. The Flutter YouTube channel made a good video about it:

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/cVAGLDuc2xE?si=4uV6geO7hkGY-0DR" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

The last important part is the
<a href="https://developer.apple.com/documentation/appkit/nsdocktile" target="_blank">NSDockTile</a>
API from macOS. (It’s not from Flutter.) This API gives you access to various things around the
dock tile for the app. We’re just interested in setting the 
<a href="https://developer.apple.com/documentation/appkit/nsdocktile/1525995-contentview" target="_blank">content</a> of the dock tile.

Basically it boils down to the following Swift code

```swift
// Create an image view
let imageView = NSImageView() 
// set an image
imageView.image = NSImage(data: Data(byte)) 

// set the image view as the content of the dock tile
NSApp.dockTile.contentView = imageView 

// force the dock tile to update
NSApp.dockTile.display()
```

People do all kind of crazy stuff with it, I’m not the only one who’s “misusing” it.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">I can’t wait to show you Superlist. ⚡️<br><br>The team is on a mission to perfectly melt notes and to-do lists, and the progress in the last few weeks has been absolutely amazing! Exciting updates coming soon: <a href="https://twitter.com/SuperlistHQ?ref_src=twsrc%5Etfw">@SuperlistHQ</a> <a href="https://t.co/KandoY4yqO">pic.twitter.com/KandoY4yqO</a></p>&mdash; Christian Reber (@christianreber) <a href="https://twitter.com/christianreber/status/1438188068549312528?ref_src=twsrc%5Etfw">September 15, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Now that we’re covered all the APIs and how they’re working, we can
assemble them together.

### Putting it all together

First, we’ll need to take a look at the native macOS API, since that’s
where we can render something into the app icon in the dock. You can do
something like the following code snippet to render an image to the app
icon.

```swift
let imageView = NSImageView() 
imageView.image = NSImage(data: Data(byte)) 
NSApp.dockTile.contentView = imageView 
NSApp.dockTile.display() 
```

Okay, but the Twitter post showed a moving application. That’s because I
continuously captured the content of the Flutter application. For that,
we need to record the screen of the Flutter application.

First, we’ll need the good old Flutter counter example and adapt it to
our needs. We’re going to wrap the main content in a 
<a href="https://api.flutter.dev/flutter/widgets/RepaintBoundary-class.html" target="_blank">RepaintBoundary</a>,
because it can screenshot its child widgets, and we give it a global key. Later, we can
access the RepaintBoundary based on the global key.

```dart
final _screenshareKey = GlobalKey();

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return RepaintBoundary(
      key: _screenshareKey,
      child: MaterialApp(
        title: 'Flutter Demo',
        theme: ThemeData(
          primarySwatch: Colors.blue,
        ),
        home: const MyHomePage(title: 'Flutter Demo Home Page'),
      ),
    );
  }
}
```

Next, we’ll need to continuously need to take screenshots, in order to
send them to the app icon. The continuous capturing is done by using a
<a href="https://api.flutter.dev/flutter/scheduler/SchedulerBinding/addPersistentFrameCallback.html" target="_blank">persistent frame callback</a>
which is called once for each frame. In its callback we’re now actually taking a
screenshot of the application.

```dart
WidgetsBinding.instance.addPersistentFrameCallback((timeStamp) {
  _syncToDockIcon();
});

Future<void> _syncToDockIcon() async {
  final renderObject = _screenshareKey.currentContext!.findRenderObject()
      as RenderRepaintBoundary;

  final image = await renderObject.toImage();
  // ...
}
```

Next, we somehow need to get the images to the native side of the
Flutter application. For that, we’re going to use a 
<a href="https://api.flutter.dev/flutter/services/MethodChannel-class.html" target="_blank">MethodChannel</a>.
We now need to put all the code together as seen below. Additionally, we need to convert the
screenshot into an image format, which is understood by the native side.

```dart
const _channel = MethodChannel('sync');

Future<void> _syncToDockIcon() async {
  final renderObject = _screenshareKey.currentContext!.findRenderObject()
      as RenderRepaintBoundary;

  final image = await renderObject.toImage();
  final bytedata = await image.toByteData(format: ImageByteFormat.png);
  final imageBytes = bytedata!.buffer.asUint8List();
  await _channel.invokeMethod('sync', imageBytes);
}
```

That’s it for the Flutter side. Next stop, the native side.

We’ll need to adapt the MainFlutterWindow class to receive the
MehtodChannel messages from the Flutter side and to present them in the
app icon. We register for the MethodChannel and get the image bytes when
the MethodChannel is called. The image bytes we’re getting from it are
used to create an NSImageView. The image view is then attached to the
content of the app icon in the dock.

```swift
import Cocoa
import FlutterMacOS

class MainFlutterWindow: NSWindow {
  override func awakeFromNib() {
    // The usual init code is left out, and only the important parts for this blog post are left

    // Register at the MethodChannel
    let channel = FlutterMethodChannel(name: "sync", binaryMessenger: controller.engine.binaryMessenger)
    channel.setMethodCallHandler(handleMessage)

    super.awakeFromNib()
  }

  // This method is called when a method is called on the Flutter side
  private func handleMessage(call: FlutterMethodCall, result: FlutterResult) {
    if call.method != "sync" {
      fatalError("fatalError")
    }
    let imageBytes =  call.arguments as! FlutterStandardTypedData
    let byte = [UInt8](imageBytes.data)
      let imageView = NSImageView()
      imageView.image = NSImage(data: Data(byte))
      NSApp.dockTile.contentView = imageView
      NSApp.dockTile.display()
  }
}
```

That’s it. As you can see, there’s no magic involved. Just some clever
combination of various APIs.

You can find the whole source code at <a href="https://github.com/ueman/flutter_in_dock" target="_blank">https://github.com/ueman/flutter_in_dock</a>.

You can follow me on the socials below.

Originally published on [Medium](https://medium.com/p/16e32ed1f91f).

<a
href="https://medium.com/@jonasuekoetter/a-flutter-application-in-an-app-icon-16e32ed1f91f"
class="p-canonical">Canonical link</a>

Exported from [Medium](https://medium.com) on January 10, 2024.
