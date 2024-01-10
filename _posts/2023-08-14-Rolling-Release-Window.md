---
layout: post
title: "Rolling Release Window"
---

A rolling release window is a strategy to be able to safely deprecate
old app versions as well as old APIs your app uses. In this article,
I’ll explain what it is and how you can implement it.

But first, let me explain the problems it can solve. Mobile apps have a
couple of problems, based on their very own nature.

While most users will update to new app versions in a matter of days,
there will be a long tail of users who are several versions behind. This
is due to user that don’t have automatic updates enabled. The automatic
updates are also not updating an app instantaneously. Therefore, you can
not assume that all users will get an update, ever.

At the same time, old app versions are unlikely to be regularly tested
by the mobile team because it is a lot of effort, with little payoff.

This long tail of old app versions makes it basically impossible to
deprecate old APIs used to communicate with the backend. This is
pictured in the image below. The versions v1 and v2 of the app use the
API v1 of the backend. As the times goes on, you see the API no longer
makes sense for the app and v2 API is established on the backend. The
app implements it and no longer uses v1 of the backend in v3 of the app.
As a result, the backend teams want to remove v1 of their API. Since old
app versions basically never die, they, however, can’t get rid of v1 of
the backend API.

<img src="https://cdn-images-1.medium.com/max/800/1*v7ioj_CEc71od8f0EKvS5w@2x.jpeg" />
<figcaption>A sad story of not being able to deprecate old backend APIs</figcaption>

Now we see that we have additional testing effort for old app versions
and old backend APIs. Most likely, the deprecated backend APIs are also
neglected and their reliability and security deteriorates.

Backend APIs are not the only things, that can’t be removed. The same
problem applies to feature flags, remote translations, third party APIs,
and the list goes on.

Another important point is, that security updates to apps can’t be
rolled out to older app versions, so only updated app versions are
getting security updates.

### The Solution — A Rolling Release Window

The solution to all the above-mentioned problems are rolling release
windows. A rolling release window is a basically a set of the latest
supported app versions. Let’s say we want to support the latest 7 app
versions. After an app version is no longer in the set of the latest 7
app versions, we do a force update to the oldest app version which is
still in the set of the supported app versions. This is also pictured in
the image below.

<img src="https://cdn-images-1.medium.com/max/800/1*rF9fPcv7sG0DGzjg5QuWSg@2x.jpeg" />
<figcaption>Rolling Release Window in practice</figcaption>

This strategy allows you to reduce testing efforts, get rid of old app
versions, get rid of old backend APIs and so on. It solves all the
above-mentioned problems.

A common issue with force updates is that users of an app can get quite
annoyed by them. So you have to make sure to support enough versions, so
that ideally all users updated from a version which drops out of the
supported versions, or that at least only a small amount of users is
affected.

Another benefit of rolling release windows is that it provides an
easy-to-understand framework to communicate to stakeholders when a
specific app version is no longer supported. This is especially helpful
for the backend teams.

Make sure that you test the force update mechanism in the app in
production. This mechanism should also be implemented for the first
public release of an app. Often enough, the force update mechanism is
never actually tested in production. This is quite similar to backups
which are never tried to be restored.

If you’re following a specific versioning scheme, you can even
completely automate this strategy.

Originally published on [Medium](https://medium.com/@jonasuekoetter/rolling-release-window-802d3d8472c4).
