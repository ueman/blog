---
layout: post
title: "Achieving Flutter Favorite Package Quality"
---

This article doesn’t show you how your package become a Flutter Favorite
package, it rather shows you how to meet and exceed the quality bar for
the Flutter Favorite program.

<figure id="b33a" class="graf graf--figure graf-after--p">
<img
src="https://cdn-images-1.medium.com/max/800/1*ME68YcayB3flWa8IvaA98g.png"
class="graf-image" data-image-id="1*ME68YcayB3flWa8IvaA98g.png"
data-width="3840" data-height="2160" data-is-featured="true" />
<figcaption>Dash, who is probably reading this article
right now</figcaption>
</figure>

> About the author:   
> Jonas is a Dart & Flutter GDE as well as a Flutter Favorite package
> author. His Flutter Favorite package is
> <a href="https://pub.dev/packages/feedback"
> class="markup--anchor markup--blockquote-anchor"
> data-href="https://pub.dev/packages/feedback" rel="noopener"
> target="_blank">feedback</a>.

But first, what is a Flutter Favorite?

> The aim of the **Flutter Favorite** program is to identify packages
> and plugins that you should first consider when building your app.
> This is not a guarantee of quality or suitability to your particular
> situation — you should always perform your own evaluation of packages
> and plugins for your project.   
> — <a href="https://docs.flutter.dev/packages-and-plugins/favorites"
> class="markup--anchor markup--blockquote-anchor"
> data-href="https://docs.flutter.dev/packages-and-plugins/favorites"
> rel="noopener" target="_blank">Flutter Favorite Page</a>

You can see the complete list of
<a href="https://pub.dev/flutter/favorites"
class="markup--anchor markup--p-anchor"
data-href="https://pub.dev/flutter/favorites" rel="noopener"
target="_blank">Flutter Favorite packages</a> on pub.dev.

Flutter Favorite packages have to meet a certain quality, this quality
is measure based on the following metrics

- <span id="8181"><a href="https://pub.dev/help" class="markup--anchor markup--li-anchor"
  data-href="https://pub.dev/help" rel="noopener" target="_blank">Overall
  package score</a></span>
- <span id="bc03">**Permissive license**, including (but not limited to)
  Apache, Artistic, BSD, CC BY, MIT, MS-PL and W3C</span>
- <span id="2e92">GitHub **version tag** matches the current version
  from pub.dev, so you can see exactly what source is in the
  package</span>
- <span id="2f59">Feature **completeness** — and not marked as
  incomplete (for example, with labels like “beta” or “under
  construction”)</span>
- <span id="8815"><a href="https://dart.dev/tools/pub/verified-publishers"
  class="markup--anchor markup--li-anchor"
  data-href="https://dart.dev/tools/pub/verified-publishers"
  rel="noopener" target="_blank">Verified publisher</a></span>
- <span id="31d3">General **usability** when it comes to the overview,
  docs, sample/example code, and API quality</span>
- <span id="08d3">Good **runtime behavior** in terms of CPU and memory
  usage</span>
- <span id="2d4a">High quality **dependencies**</span>

Let’s take a look at each individual metric and see how we can achieve
to meet, but, ideally, exceed the quality bar.

### Package Score

<figure id="66b8" class="graf graf--figure graf-after--h3">
<img
src="https://cdn-images-1.medium.com/max/800/1*SJgZjBOml5DXBBhKymSQxA.png"
class="graf-image" data-image-id="1*SJgZjBOml5DXBBhKymSQxA.png"
data-width="1610" data-height="1160" />
<figcaption>Example package score</figcaption>
</figure>

The package score is a metric which in turn measures various metrics on
its own.

The first one is ‘Follow Dart file conventions’. In order to achieve all
available points, you have to use a permissive package license. This
means to use an OSI-approved license. To learn more about licenses, you
can take a <a href="https://www.tldrlegal.com/"
class="markup--anchor markup--p-anchor"
data-href="https://www.tldrlegal.com/" rel="noopener"
target="_blank">TLDRLegal</a> which explains licenses in plain English.
A list of OSI-approved licenses can be found at
<a href="https://opensource.org/licenses/"
class="markup--anchor markup--p-anchor"
data-href="https://opensource.org/licenses/" rel="nofollow noopener"
target="_blank">https://opensource.org/licenses/</a>. Additionally, you
have to provide a valid README.md, CHANGELOG.md and pubspec.yaml file.
This can be checked via `dart pub publish --dry-run`.

The next category is ‘Provide documentation’. To get all available
points, you have to provide extensive documentation, including an
example. To make sure that all your APIs are documented, you can turn on
the following links:

- <span id="35ea"><a href="https://dart.dev/tools/linter-rules/public_member_api_docs"
  class="markup--anchor markup--li-anchor"
  data-href="https://dart.dev/tools/linter-rules/public_member_api_docs"
  rel="noopener" target="_blank"><code
  class="markup--code markup--li-code">public_member_api_docs</code></a></span>
- <span id="4276"><a href="https://dart.dev/tools/linter-rules/package_api_docs"
  class="markup--anchor markup--li-anchor"
  data-href="https://dart.dev/tools/linter-rules/package_api_docs"
  rel="noopener" target="_blank"><code
  class="markup--code markup--li-code">package_api_docs</code></a></span>
- <span id="ebd1"><a
  href="https://dart.dev/tools/linter-rules/dangling_library_doc_comments"
  class="markup--anchor markup--li-anchor"
  data-href="https://dart.dev/tools/linter-rules/dangling_library_doc_comments"
  rel="noopener" target="_blank"><code
  class="markup--code markup--li-code">dangling_library_doc_comments</code></a></span>
- <span id="0bf2"><a href="https://dart.dev/tools/linter-rules/comment_references"
  class="markup--anchor markup--li-anchor"
  data-href="https://dart.dev/tools/linter-rules/comment_references"
  rel="noopener" target="_blank"><code
  class="markup--code markup--li-code">comment_references</code></a></span>

<a href="https://dart.dev/tools/pub/package-layout#examples"
class="markup--anchor markup--p-anchor"
data-href="https://dart.dev/tools/pub/package-layout#examples"
rel="noopener" target="_blank">The Dart doc side tells you how you
should structure your example.</a>

The ‘platform support’ metrics grants you full points, when you support
all platforms that Flutter supports. This is only important, if your
package is platform specific.

‘Pass static analysis’ means that you should have no warnings or errors
when running `dart analyze .` or `dart format .`. Ideally, you should
also use <a href="https://pub.dev/packages/lints"
class="markup--anchor markup--p-anchor"
data-href="https://pub.dev/packages/lints" rel="noopener"
target="_blank">lints</a> or
<a href="https://pub.dev/packages/flutter_lints"
class="markup--anchor markup--p-anchor"
data-href="https://pub.dev/packages/flutter_lints" rel="noopener"
target="_blank">flutter_lints</a>.

‘Support up-to-date dependencies’ grants all points, if your code works
when using the newest available dependencies listed in your pubspec.yaml
file, and also supports the latest Dart or Flutter version.

### GitHub version tag matches Pub version

To be clear, this doesn’t mean that you have to use GitHub, GitLab and
other git hosts are also fine, but your git tags should still match the
versions on pub.dev. The aim of this rule is that people can easily see
what code is in which version. This aims to raise confidence against
supply chain attacks.

Ideally, you automate this by deploying your code to pub.dev, when you
create a new version tag. If you’re indeed using GitHub, you can use
<a href="https://dart.dev/tools/pub/automated-publishing"
class="markup--anchor markup--p-anchor"
data-href="https://dart.dev/tools/pub/automated-publishing"
rel="noopener" target="_blank">the guide from the Dart team</a> to
automate publishing.

### Verified Publisher

By being a verified publisher, you’re letting your consumers know, that
you’ve verified the identity. For example,
<a href="https://pub.dev/publishers/dart.dev/"
class="markup--anchor markup--p-anchor"
data-href="https://pub.dev/publishers/dart.dev/" rel="noopener"
target="_blank">dart.dev</a> is the verified publisher for packages that
Google’s Dart team supports. Again, this aims to raise the confidence in
you being a trustworthy publisher and thus supply chain attacks are less
likely.

To become a verified publisher, you have to follow <a
href="https://dart.dev/tools/pub/publishing#create-verified-publisher"
class="markup--anchor markup--p-anchor"
data-href="https://dart.dev/tools/pub/publishing#create-verified-publisher"
rel="noopener" target="_blank">the guide</a> from the Dart team.

### Usability of docs

Additionally, to the tips for docs further above, you should put a lot
of care into your docs, so that they are helpful. This means you should
give code snippets which show the consumer how to use your library.
These code snippets should have an explanation telling the consumer why
it works that way.

Packages that set a good example are

- <span id="e23a"><a href="https://pub.dev/packages/drift"
  class="markup--anchor markup--li-anchor"
  data-href="https://pub.dev/packages/drift" rel="nofollow noopener"
  target="_blank">https://pub.dev/packages/drift</a></span>
- <span id="2cce"><a href="https://pub.dev/packages/riverpod"
  class="markup--anchor markup--li-anchor"
  data-href="https://pub.dev/packages/riverpod" rel="nofollow noopener"
  target="_blank">https://pub.dev/packages/riverpod</a></span>

### Runtime behavior and dependencies

Further requirements are

- <span id="c244">Good **runtime behavior** in terms of CPU and memory
  usage</span>
- <span id="384a">High quality **dependencies**</span>

Usually, if you don’t do anything stupid, your code will not behave bad
and thus has a good runtime behavior and a good performance. If you’re
in the unfortunate situation where your package does a poor job, you can
use the Dart Dev Tools to debug that problem. Good resources for that
are the pages about the
<a href="https://docs.flutter.dev/tools/devtools/performance"
class="markup--anchor markup--p-anchor"
data-href="https://docs.flutter.dev/tools/devtools/performance"
rel="noopener" target="_blank">performance tools</a>, the
<a href="https://docs.flutter.dev/tools/devtools/cpu-profiler"
class="markup--anchor markup--p-anchor"
data-href="https://docs.flutter.dev/tools/devtools/cpu-profiler"
rel="noopener" target="_blank">CPU profiler</a> and the
<a href="https://docs.flutter.dev/tools/devtools/memory"
class="markup--anchor markup--p-anchor"
data-href="https://docs.flutter.dev/tools/devtools/memory"
rel="noopener" target="_blank">memory profiler</a>.

Ideally, your package doesn’t have any dependencies, as
<a href="https://develop.sentry.dev/sdk/philosophy/#dependencies-cost"
class="markup--anchor markup--p-anchor"
data-href="https://develop.sentry.dev/sdk/philosophy/#dependencies-cost"
rel="noopener" target="_blank">dependencies are costly</a>. Of course,
there are cases where it’s necessary. In those, you should make sure to
use dependencies which use a compatible license (as already discussed
above) which have a high number of pub points, a high popularity and
tests. Typically, those dependencies are already battle tested and thus
of a higher quality.

### Implied package aspects

There are some other aspects you need to consider next to the explicitly
stated quality properties.

Those are for example that your package should already have a high
popularity, since that is already an indicator that you match at least a
few of the above stated properties. Furthermore, your code should be
well tested in a sensible way, so like widget and golden tests for
widgets, unit test for non-UI code or integration test for native
platform code.

Originally published on [Medium](https://medium.com/@jonasuekoetter/achieving-flutter-favorite-package-quality-22433aa792ab).
