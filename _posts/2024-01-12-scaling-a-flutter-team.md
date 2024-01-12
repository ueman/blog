---
layout: post
title: "Scaling a Flutter team"
---

I'm working at my current employer for just over two years now. Since I started there, we grew from two to five teams and from a dozen or so people to now nearly 40 people in our domain. Of course not all are devs but the majority is.

Scaling a team is hard as it come with a lot of challenges. This article contains some tips which helps with the technical side to make growing the team less painful.

> Disclaimer: These are my thoughts and do not necessarily reflect those of my employer or my coworkers.

## Building a UI library on top of a framework

A lot of apps have their own design language which doesn't follow material design (Android) or Apple's Human Interface Guidelines. This is especially common with Flutter, which makes it really easy to share the same design across Android and iOS. This corporate branding requires us to build our own UI component library. 

So what are the requirements, apart from matching the corporate branding?

- We want an easy discoverability of UI components, since we're most likely have more than a couple of components.
- Components should feel as if they're part of the Flutter framework. This makes them feel familiar and intuitive to use.

Tools are the best way to make UI components discoverable. The first one is to build an app which acts like a book of UI components. That way, you can simply scroll through it to see which component you need. Ideally, it's also somewhat ordered, so you can find components even easier. [Widget Book](https://pub.dev/packages/widgetbook) is a tool which helps you do this.

Another pretty helpful tool is [DCM](https://dcm.dev/) (Dart Code Metrics). It's a linter which provides additional very helpful features. In this case, it's the ban name rule. That can be used to prohibit the usage of Flutter's built-in UI components and guide the developer instead to use the correct component from the UI component library. This helps to make the development a guided experience, and PR reviews also get easier since the reviewer doesn't have to check whether the correct component was used.

The following is an example for this use case. It assumes you always want to use `FooButton` instead of the built-in buttons from Flutter.

```yaml
dart_code_metrics:
  rules:
    - ban-name:
        entries:
          - ident: MaterialButton # Should be repeated for all other buttons too
            description: Use `FooButton` instead.
```

You can further enhance things with [custom assists](https://dcm.dev/docs/teams/assists/#wrap-with-). 

The next point is to make your custom UI components feel right at home in the Flutter framework. Consider the following example

```dart
// Button from the Flutter framework
ElevatedButton(
  style: ElevatedButton.styleFrom(textStyle: const TextStyle(fontSize: 20)),
  onPressed: () {},
  child: const Text('Enabled'),
)

// bad example
BadCustomButton(
  theme: CustomButton.styleFrom(textStyle: const TextStyle(fontSize: 20)),
  onClick: () {},
  content: const Text('Enabled')
)

// good example
GoodCustomButton(
  style: ElevatedButton.styleFrom(textStyle: const TextStyle(fontSize: 20)),
  onPressed: () {},
  child: const Text('Enabled'),
)
```

The first widget is taken from the Flutter framework. The next one represents a bad example. While the button is fully functional, it's constructor parameter names deviate drastically from what a developer would expect, and thus it gets hard to use. The last example is a button which closely mimics the button from the framework. Due to this, it's much easier to use. Of course, this is just a simple example, but following this practice goes a long way in developing widgets which will feel like they're part of the Flutter framework. This could be enforced by writing custom lint rules with [`custom_lint`](https://pub.dev/packages/custom_lint).

Following these tips doesn't allow help you scale and grow the team, it also makes the onboarding of new developers easier. Since we've improved the discoverability and also made it hard to use wrong UI components, new hires should become familiar with the UI component. 

Lastly, the developers of the UI component library should continuously ask the users of it for feedback. Think of the UI component library as your product, and act accordingly. Continuous feedback helps you catch mistakes early on and mitigate them.

## Ecosystem & Dependency Management

As your project grows, so does your list of dependencies. And those get updated from time to time for various reasons. Security and bug fixes are probably the most important ones. So maintaining up-to-date dependencies gives us a better security posture. 

Though, dependencies aren't the only thing we want to update. We also want to use the latest Flutter version and target the latest versions of Android and iOS.

So, how do we solve those needs?

For regular dependency updates, we can make use of dependable and renovate. That way we can automate the tedious task somewhat and free the engineers for more important work. Smaller updates also result in smaller chances of breaking anything. Since there are no "big bang" migrations, it's easier to understand and dig into problems if there are incompatibilities.

If there are bugs found while upgrading dependencies, fixing and contributing back should be encouraged, as it strengthens the engineering capabilities and helps us deepen our expertise and understanding of our application and its dependencies.

----

Your application doesn't only depend on the dependencies specified in the `pubspec.yaml`. It also depends on the Flutter framework, as well as on Android and iOS. Targeting always the latest versions of Flutter, Android and iOS ensures forward compatibility, and we can make use of newer features as soon as they're available, resulting in potential competitive advantages. Furthermore, ensuring compatibility results in easier upgrade paths as well as higher resilience and stability. Thus, the underlying framework and operating systems are an extension of the application. Treating it as an unimportant part of your application is a mistake, and many leaders make this mistake.

Ensuring compatibility to newer versions of Flutter can be achieved by running CI against the beta channel once a month or against the master channel once a week. That way, possible incompatibilities are found early. Fixing those issues results in a smooth upgrade path with the potential for a day one upgrade to the new version. Android and iOS also provide early beta versions as well as announcements for major changes. Try those out and act accordingly.

## Application Architecture

The application architecture allows us to reason about the app, because it sets expectations and provides us with familiarity of unknown code. The architecture furthermore establishes a set of assumptions in which estimates for planning make sense. The architecture is therefore much more than just a way to structure code.

As the number of team grows, it makes sense to split code by team, and typically a team has the ownership of one feature. This has a couple upsides. The architectural boundaries are similar, or even identical, with team structure, which makes ownership simpler because boundaries are clear. Additionally, each team can adapt their own workflows and quality processes, without interfering with the processes of other teams.

## Pull Requests and Merge Requests

There are a couple of things you can improve to improve the process of code reviews in pull requests (as known from GitHub) or Merge Requests (same thing but GitLab uses this term).

- Try to do fast and early reviews. This unblocks your coworkers and makes sure that work which is basically considered done lands soon. That's more important than finishing your own in progress work. 
- Do small PRs. Smaller chunks of code are most of the time easier to review. It's harder to do a code review for changes which are spread over 40 files. 
- If there's a bug fix PR, jump on it. Make sure a bugfix PR always has a regression test. We don't want to fix the same bug twice.
- If you notice a bug while reading the code, fix it directly or at least write a new ticket, so that it isn't forgotten.

## Documentation and Code search

As the developer team grows, so does the code base. Make sure to always document important parts of the code, chances are that you need it later. Documenting why something was engineered a certain way helps when onboarding new people as well as with adaptions to that code later on.

As the code base grows, it gets harder to stay DRY, since no one knows the whole code base. Chances are that code which is very similar in the feature set and which could be unified slips in completly undetected. There are tools to detect and avoid code duplication, so that's not an issue. That's why code search becomes more important as the code base grows. Good documentation aids in the code search, as the documentation is probably a good source of keywords for a certain feature.

We also want to reuse code as much as possible. Often times I find myself search for a specific class or feature, without knowing how it's called. Code search is very important for that. Another very

## Linting

This one is a tip that many of you may have heard before. Us a linter. That saves time by flagging issues already during the development. PR reviews are thus much more focused on the things that matter, and you don't have to waste time by teaching or commenting on small issues like formatting and other easy to avoid issues.

Though there's not just the linter shipped with Dart itself. There's also DCM and "custom_lint" which enable you to build lint rules to match your team's development practices.

### We're also building for other devs

Even though we primarily build for our customers, we also build for our colleagues. We should therefore try to understand how they will use the code we write. We should make the code a pleasure to use.

## APM tools

APM (application performance monitoring) tools help with detecting bugs and performance issues. Fixing those helps improve the quality of your app. Though, they don't just help to fix newly found issues. See those tools also as a way to detect systematic issues. If every feature you're developing suffers from similar issues, there's something wrong with your requirement engineering. Try to figure out a way to incorporate the insights from the APM tools while writing tickets. That way it gets way more sustainable as a whole group of issues will be a thing of the past. Or to put it into other words, learn from what you're seeing in APM tools and let that be a guide for future developments. (I've written a bit more about APM tools in other articles on this blog.)

## Fixing things

I'll just leave [this](https://overreacted.io/fix-like-no-ones-watching/) here.

## Thoughts about remote work

While I greatly enjoy working from home, onboarding on a remote team is harder than to a local team. Being able to talk "offline" to your coworkers in small ways creates trust. Chats and video calls can't replace that, because it feels way more formal. Therefore, I recommend having regular in-person meetups. That creates bonding experiences and creates another layer of trust.