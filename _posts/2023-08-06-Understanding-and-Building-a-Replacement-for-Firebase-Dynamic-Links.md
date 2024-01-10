---
layout: post
title: "Understanding and Building a Replacement for Firebase Dynamic Links"
---

As you might know, Firebase Dynamic Links got deprecated as stated in
<a href="https://firebase.google.com/support/dynamic-links-faq"
rel="noopener" target="_blank">an FAQ entry</a>:

> **Note:** Firebase Dynamic Links is no longer recommended for new
> projects. In the future, the Dynamic Links service will shut down, but
> you will have at least 12 months from the announcement date to
> migrate. We will announce more information in Q3 2023.

<img src="https://cdn-images-1.medium.com/max/800/1*GbScs59lOT2tjF7wEYdG5Q.jpeg" />
<figcaption>Connected chain links. Get it? Because links!</figcaption>

That’s why we’re going to build a replacement for Firebase Dynamic Links
(FDL) with Flutter in this article.

But first we’ll need to understand what FDL offers and how it works,
otherwise we’re not able to build a replacement. Furthermore, we want to
understand why it got deprecated, though, since there are no official
reasons mentioned, the reasons mentioned by me are just educated
guesses. With that being said, let’s jump straight into the teardown of
FDL.

### Firebase Dynamic Links: What is it and how does it work

<a href="https://firebase.google.com/docs/dynamic-links" target="_blank">FDLs value proposition</a>

> \[…\] are links that work the way you want, on multiple platforms, and
> whether or not your app is already installed.

At a first glance, this sounds a lot like
<a href="https://developer.android.com/training/app-links" target="_blank">
Android’s App Links</a> or
<a href="https://developer.apple.com/ios/universal-links/" target="_blank">
iOS’ Universal Links</a>. Both of those
technologies enable deeplinks into your app, just like FDL. Contrary to
FDL, those are two mechanisms are baked right into their respective
operating systems and SDKs. Interestingly enough, FDL actually predates
App Links and Universal Links. FDL is about 7 years old, whereas App
Links and Universal Links are younger technologies.

The biggest advantage of FDL over App Links and Universal Links is,
however, that FDL has a mechanism that covers deeplinks for the first
app start. It’s called deferred deeplinking. This is very nicely
explained by this image

<img src="https://cdn-images-1.medium.com/max/800/0*8ICKhwIQgMd7JiKi.png"/>
<figcaption>Taken from <a href="https://blog.pragmaticengineer.com/google-shutting-down-firebase-dynamic-links/" target="_blank">
https://blog.pragmaticengineer.com/google-shutting-down-firebase-dynamic-links/</a></figcaption>

On iOS, this first app start deeplink works by copying the deeplink
information to the users' clipboard on the webpage. After installing and
starting the app, the app reads the users’ clipboard, parses it and then
allows the developer to use that information to open a specific page in
the app.

On Android, I assume that
<a href="https://developer.android.com/google/play/installreferrer" target="_blank">
Google Play Install Referrer</a> are used
for deeplinks for the first app start. Though, the same clipboard
solution can be used where Google Play Services cannot be used.

Let’s take a closer look at the clipboard solution.

We’ll start with the part that’s happening in the browser. We can make
use of the `navigator.clipboard` APIs in order to write something to the
clipboard. Nice! Well, kinda. The <a
href="https://developer.mozilla.org/en-US/docs/Web/API/Navigator/clipboard" target="_blank">documentation</a> for it states that we
have to request the users’ permission first, otherwise we’re not able to
copy something to the clipboard.

In the app, reading from the clipboard causes a popup on iOS and
Android, notifying the user about it. This makes the app looks like it’s
maliciously reading or even spying on its users when it’s not properly
explained.

Other similar services, like branch.io, build their deferred deeplinking
solution by saving the destination on the backend for the IP of a
specific. This has the drawback, that you can only do it, if the user
allows it. Otherwise, you’re violating GDPR.

Additionally, FDL provides functionality for shortlinks and analytics.

### Guessing reasons for FDL being shut down

Now that we’ve understood what FDL offers and how it works, we’re able
to do some educated guess as to why it’s getting shut down.

1.  The deeplinking functionality is now baked right in
    into the mobile operating systems. There’s no need anymore for a
    third party service.
2.  The deferred deeplinks for the first app aren’t as
    seamless anymore as they’re now plagued (for a good reason) by
    permission requests for clipboard writing, and hints for clipboard
    access. This turned the whole first app start experience from
    magical experience to a somewhat nightmarish experience, as the app
    looks like it’s spying on you right on the first app start.  
    Solutions which work by saving a user's IP need the consent of the
    user. An IP is typically considered personal data. So this approach
    is also not really seamless.
3.  It’s easy enough to build the open app store
    functionality yourself, if you’re already using App Links and
    Universal links, so the shortlinks kinda loose some of its
    value.

### **Building a replacement for FDL**

We need to decided how important which parts of FDL are for our app.

- Do we value the deferred deeplinking functionality?
- Do we value the regular deeplinks?
- Do we value short links?

The important part can be any combination of the above questions.
However, I’m just explaining point 1 and 2, as there are already a lot
of tuturials out there explaining shortlinks, and this is a mobile
focused article.

Of course, you can also just use any available third party service like
branch.io, Adjust or whatever. But where would be the fun it?

#### Regular deeplinks replace FDLs deeplinks

In order to be forward compatible and not rely on a third party service
which could be shut down, we’re using App Links and Universal Links. An
additional benefit from that is, that Flutter already has built in
support for those kinds of deeplinks. This already explained well better
than I could on the Flutter website, so I recommend checking their
documentation:

- <span id="4ceb">Deeplinks with Flutter:
  <a href="https://docs.flutter.dev/ui/navigation/deep-linking"
  class="markup--anchor markup--li-anchor"
  data-href="https://docs.flutter.dev/ui/navigation/deep-linking"
  rel="nofollow noopener"
  target="_blank">https://docs.flutter.dev/ui/navigation/deep-linking</a></span>
- <span id="d820">Android Setup:
  <a href="https://docs.flutter.dev/cookbook/navigation/set-up-app-links"
  class="markup--anchor markup--li-anchor"
  data-href="https://docs.flutter.dev/cookbook/navigation/set-up-app-links"
  rel="nofollow noopener"
  target="_blank">https://docs.flutter.dev/cookbook/navigation/set-up-app-links</a></span>
- <span id="db6d">iOS Setup: <a
  href="https://docs.flutter.dev/cookbook/navigation/set-up-universal-links"
  class="markup--anchor markup--li-anchor"
  data-href="https://docs.flutter.dev/cookbook/navigation/set-up-universal-links"
  rel="nofollow noopener"
  target="_blank">https://docs.flutter.dev/cookbook/navigation/set-up-universal-links</a></span>

### Building deferred deeplinks ourselves

Now, moving on to the fascinating part of FDL, deferred deeplinks. When
I investigated into this topic, I was really fascinated by the approach,
it’s genius.

On the website, we assume that App Links and Universal Links are in use.
That means anytime the user is actually landing on the webpage, it means
he’s either not using a mobile phone, is on a mobile but hasn’t
installed the app, or is on a mobile phone and disabled deeplinking. To
discern between a mobile phone and desktop browser, we can check the
UserAgent of the browser. That also helps us in to know whether we
should link to the App Store or to Google Play.

```js
const getMobileOS = () => {
  const ua = navigator.userAgent
  if (/android/i.test(ua)) {
    return "Android"
  }
  else if (/iPad|iPhone|iPod/.test(ua)){
    return "iOS"
  }
  return "Other"
}

// ...

const os = getMobileOS()
if (os === 'Android') {
  window.location.href= "market://details?id=<packagename>";
} else if (os === 'iOS') {
  window.location.href= "https://apps.apple.com/de/app/<app-id>"
}
```

Okay, now we also need to copy the current URL to the clipboard, in
order to be able to read it in the app.

```js
const getMobileOS = () => {
  const ua = navigator.userAgent
  if (/android/i.test(ua)) {
    return "Android"
  }
  else if (/iPad|iPhone|iPod/.test(ua)){
    return "iOS"
  }
  return "Other"
}

// ...

const os = getMobileOS()
// The following line got added. You may or may not neet to request a permission to do this.
navigator.clipboard.writeText(window.location.href);
if (os === 'Android') {
  window.location.href= "market://details?id=<packagename>";
} else if (os === 'iOS') {
  window.location.href= "https://apps.apple.com/de/app/<app-id>"
}
```

Moving on to the Flutter part:

```dart
Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  final deferredDeeplink = await _getDeferredDeeplink();
  runApp(MaterialApp(
    initialRoute: deferredDeeplink,
  ));
}

Future<String?> _getDeferredDeeplink() async {
  if (await Clipboard.hasStrings() == false) {
    return null;
  }
  final content = await Clipboard.getData('text/plain');
  if (content == null) {
    return null;
  }
  // now you need to check whether the content matches
  // the format of your associated website.
  if (content.data.startsWith('https://example.org') {
    // The initial route property of MaterialApp expects the deeplink 
    // to start with a '/'
    return content.data.replaceFirst('https://example.org', '');
  }
  return null;
}
```

Additionally, the deferred deeplink check should only happen on the
first app start, not on any of the consecutive app starts. It’s not
shown here, but it can be done based on a persistent data store of your
choice.

I want to note, that this is a very naive approach as is, as I want to
show how it works. In a real world application, you should put emphasis
on explaining the users why you’re using the clipboard. Otherwise, you
might look like you’re spying on your users.

This can be done by either explaining this to your users, or by
explicitly asking your users if you’re allowed to do so. This however
makes the whole experience not as seamless anymore. Which, as I
explained above, is also why I believe FDL got deprecated.

------------------------------------------------------------------------

Additional resources, which we’re quite helpful during my research for
this article, were the following:

- <a href="https://blog.pragmaticengineer.com/google-shutting-down-firebase-dynamic-links/" target="_blank">https://blog.pragmaticengineer.com/google-shutting-down-firebase-dynamic-links/</a>
- <a href="https://firebase.google.com/support/dynamic-links-faq" target="_blank">https://firebase.google.com/support/dynamic-links-faq</a>

Originally published on [Medium](https://medium.com/@jonasuekoetter/understanding-and-building-a-replacement-for-firebase-dynamic-links-2dedd4ea5401).
