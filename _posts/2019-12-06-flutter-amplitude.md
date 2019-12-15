---
layout: post
title: Provider + Rebuild + Navigator make it hard to implement tracking 
categories: [All, Flutter]
tags: [flutter, amplitude, ios, mobile]
fullview: false
excerpt: Recently we ran into an issue of using Amplitude in Flutter. We had to jump through a few hoops to get Amplitude events working. Hopefully, by sharing our experience, the readers can avoid the mistakes we make.
comments: true
---

**Note: This article was written before we were aware of [Daivd's article](https://medium.com/flutter-community/how-to-track-screen-transitions-in-flutter-with-routeobserver-733984a90dea). For implementing page tracking, [RouteObserver class](https://api.flutter.dev/flutter/widgets/RouteObserver-class.html) is a better solution.** 

Recently we ran into an issue of using [Amplitude](https://github.com/amplitude/Amplitude-Flutter) in Flutter. We had to jump through many hoops to get Amplitude events working. Hopefully, by sharing our experience, the readers can avoid the mistakes we make.

The requirement is simple: When user goes to a page or opens a dialogue, we send an Amplitude event. This is helpful to understand user behaviour as well as makes it easier to know where the app crashes.

Following the tutorial, we have an Amplitude class like this:

```dart
class Amplitude {
  final AmplitudeFlutter client = AmplitudeFlutter('api-key'); 
  void dispose() {}
}
```

We think that, initializing an Amplitude object before sending each event could be expensive. Why don't we have just one client and share it among all components? 

We know that Provider package can help us share variables inside the widget tree, so we gave it a try. So in my `main.dart`, we have something like this:

```dart
runApp(
      MultiProvider(
        providers: [
          Provider<Amplitude>(
            builder: (context) => Amplitude(),
            dispose: (_, value) => value.dispose(),
          ),
          // other Providers
       ]
       child: MyApp()
   ));
```
With this, we continue to implement a page level tracking. Let's use a simplified example to illustrate what we did:

```dart
class FirstPage extends StatelessWidget {
    Widget build(BuildContext context) {
      Provider.of<Amplitude>(context).client.logEvent(name: 'first page');
      // lots of build logic
      return Button(
                  text: 'Turn a new leaf!',
                  onPressed: () => _open2ndPage(context),
                );

     }
     _open2ndPage(context) {
       Navigator.push(
         context,
          MaterialPageRoute(
          builder: (context) => SecondPage(),
        ),
    );
   }
}
```
The implementation should be straightforward: When building the 1st page, we should receive the a `first page` event. 

It turns out `first page` event was fired TWICE when we were on the first page. When we hit the button, the same event was fired THREE times!

What's going on?

After some investigations, we found that `Provider.of()` will trigger rebuild when the provided variable changes. In our case, this variable is the Amplitude client, so when firing an event, it must have changed the state and triggered build() to run again, so we saw the first duplicate events.

The solution was simple. According to [Provider documentation](https://pub.dev/packages/provider), `Provider.of` has a flag `listen: false` can let Provider to not trigger the rebuild even if the variable. So this solves the first problem.

However, the 2nd issue still exists. When we pressed the button, it still fired a `fist page` event. That means that, the build() function was run again. 

This issue puzzled us for a while, until we found the [GH issue](https://github.com/flutter/flutter/issues/18366):  Widgets get rebuilt after `Navigator.push()/pop()`! 

According to the discussion thread, this is actually expected. So `build()` should be written to be okay rendering many times, even when we push another page on top.

In our case, we could get away with using `Navigator.pushReplacement()` everywhere to build a new page to replace the current page. 

However, this solution means we won't be able to implement a temporary page that gets stacked on top of current one,  and gets throws away quickly. Also, all page transition has to be explicit.

Another idea was avoid sending Amplitude events inside build(). Instead, we send the event in the FirstPage constructor function. This way, we make sure the event only fires once when the page object is built. We haven't tried this way yet.

Do you have a better solution? Feel free to drop me a note!
