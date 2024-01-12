---
layout: post
title: "Shamelessly stealing web optimizations for Flutter"
---

Web has a very long history of optimization. There’s even a famous study
from Amazon, which indicates that slow performance results in less
sales. Most of those web optimizations also apply for apps, so I’ll
present some optimizations, which are shamelessly stolen from the web.

Well, actually they aren’t really stolen from web, as they’re well
understood. There’s for example a lot of information to be found at
<a href="https://web.dev/metrics/" target="_blank">https://web.dev/metrics/</a>.
We’ll now take a look at a couple of strategies, which have a positive impact on your app.

### Responsive image loading

At the start of the development of an app, you’re typically running a
“one size fits all” strategy. Though, this is probably not intentional,
but because optimizations aren’t yet as important as getting it working.

Serving desktop-sized images to mobile devices can use 2–4x more data
than needed. This results in slower loading times for images, increased
data usage, and slower image decoding times on lower end devices. The
web has <a href="https://web.dev/serve-responsive-images/" target="_blank">
special APIs for this</a>.

Flutter doesn’t have specific APIs for this, so I’m going to show you
how to do this. But first, we’ll need to talk about the backend.
Typically, you want to have the images in 3–5 different resolutions.
Those images should be pre-sized, since on demand resizing increases the
duration of the request.

Now back to implementing this on Flutter. For bundled images, this can
be done by following 
<a href="https://docs.flutter.dev/ui/assets/assets-and-images#resolution-aware" target="_blank">
this Flutter guide</a>. For images coming
from the web, this isn’t as easy. First, we’ll create a widget to get
the size of the available space for the image, as well as the current
screen density. With that ResponsiveImageBuilder we can then create a
URL to request the image in the correct size.

```dart
typedef ImageBuilder = Widget Function(BuildContext, double width, double height, double densitiy);

class ResponsiveImageBuilder extends StatelessWidget {
  const ResponsiveImageBuilder({super.key, required this.builder});
  
  final ImageBuilder builder;
  
  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
        builder: (BuildContext context, BoxConstraints constraints) {
          return builder(
            context,
            constraints.maxWidth,
            constraints.maxHeight,
            MediaQuery.of(context).devicePixelRatio,
          );
        },
      );
  }
}

// ...

ResponsiveImageBuilder(
  builder: (context, width, height, density) {
    // Build URL to request image based on the width, height and density
    final url = '';
    return Image.network(url);
  }
);
```

### Adaptive Loading

Adaptive loading is loading content depending on the network, like 2G,
3G, LTE.

- <span id="d8d3">Decide on whether to load high or low resolution
  content, like images, videos and so on. Typically, streaming services
  do this.</span>
- <span id="dcca">If you do background up- or downloads, pause and
  resume them depending on the network</span>
- <span id="dd8a">Check whether roaming is enabled and warn users, that
  additional cost may occur.</span>

There are <a href="https://pub.dev/packages?q=network+type"
class="markup--anchor markup--p-anchor"
data-href="https://pub.dev/packages?q=network+type" rel="noopener"
target="_blank">several libraries available on pub.dev</a> which provide
this kind of information. Choose one as you see fit.

### Incremental Loading

Let’s assume that the initial page of the app consists of multiple
different data sets, as it is often the case. Incremental loading means
that each of these data sets is individually loaded. This incremental
loading makes your app seem faster, since content can be presented
faster. It also helps you to be interactive faster. Make sure that you
don’t introduce annoying layout shifts, though, as that can break the
now faster experience again.

### Preloading content

Preloading content is the process of loading content for pages your user
might visit next. Flutter does already provide an
<a href="https://api.flutter.dev/flutter/widgets/precacheImage.html"
class="markup--anchor markup--p-anchor"
data-href="https://api.flutter.dev/flutter/widgets/precacheImage.html"
rel="noopener" target="_blank">API</a> for preloading images:

```dart
precacheImage(
  [NetworkImage(url1), NetworkImage(url2)],
  context,
  onError: (exception, stackTrace) {
    print('$exception $stackTrace')
  },
);
```

Other content has to be preloaded by yourself, though.

### Optimizing bundle size

On web, you often use JavaScript minification (and obfuscation) to
reduce the bundle size, in order to reduce the amount of code which
needs to be downloaded. This results in quicker page loading.
Additionally, you typically use a tool like
<a href="https://imageoptim.com/mac"
class="markup--anchor markup--p-anchor"
data-href="https://imageoptim.com/mac" rel="noopener"
target="_blank">ImageOptim</a> in order to reduce the size of images, so
that images load quicker. This is done additionally to the responsive
image loading.

In Flutter, you can also make use of code obfuscation (minification,
sometimes also called tree-shaking, is done by default in Dart) and also
of splitting debug symbols. Both can reduce the bundle size of your app.
Be aware, though, that you need to make your error monitoring service of
choice aware of the debug symbols.

Flutter has also tree-shaking for icon fonts and not just for code,
which helps to further reduce the bundle size.

Image optimization is possible for Flutter, just crunch the images
you’re bundling with a tool similarly to ImageOptim. Ideally, this is
also built-in into your build pipeline, so that it doesn’t matter if
you’re committing optimized images.

You can further reduce the bundle size of your app by employing native
strategies to reduce code size. Favor, for example, Android’s AABs over
APKs for distribution of your app.

Originally posted on [Medium](https://medium.com/@jonasuekoetter/shamelessly-stealing-web-optimizations-for-flutter-e0f89f56f604).
