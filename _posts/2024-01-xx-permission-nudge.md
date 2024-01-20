---
layout: post
title: "Building a permission nudge to improve permission conversion rate"
---

This brief article explains how you can build a simple permission nudge widget, to improve the conversion rate of permission grants.

TODO Image here

There's a couple of problems we're trying to solve here:
- We've already asked the user for a permission, but the user declined and now it's permantly denied.
- A new permission is being requested during an onboarding flow, but already existing users don't go through it
- We want to make users aware of what they're missing out on
- We want to provide a guided experience in case they change their mind

Building a widget that reacts to permission changes is quite tricky, since there's no way to listen for permission status changes.
That means there's no easy way to react to permission changes that happen anywhere. Another problem is, that even when there's such
a thing, you probably could not react to permission changes that happen while your app is in the background.
However, an attentive might already sense where I'm heading, the solution to all those problems is to listen to the application lifecycle
and request the current status when the application gets resumed. Anytime a permission is requested the OS shows an dialog, thus putting the 
app into the paused state. Furthermore, the permission can be changed in the settings, where the app is also not in foreground. 
Therefore making a permission check on the resumed event an elegant solution.
The following code snippet shows how to do it.

```dart
// Tested with Flutter 3.16
import 'package:flutter/material.dart';
import 'package:permission_handler/permission_handler.dart';

typedef NudgeBuilder = Widget Function(
    BuildContext context, PermissionStatus? status);

class PermissionNudge extends StatefulWidget {
  const PermissionNudge({
    super.key,
    required this.permission,
    required this.builder,
  });

  final Permission permission;
  final NudgeBuilder builder;

  @override
  State<PermissionNudge> createState() => _PermissionNudgeState();
}

class _PermissionNudgeState extends State<PermissionNudge> {
  late AppLifecycleListener _listener;
  PermissionStatus? status;

  @override
  void initState() {
    _listener = AppLifecycleListener(onResume: _checkPermission);
    super.initState();
    _checkPermission();
  }

  @override
  void didUpdateWidget(covariant PermissionNudge oldWidget) {
    super.didUpdateWidget(oldWidget);
    if (oldWidget.permission != widget.permission) {
      _checkPermission();
    }
  }

  @override
  void dispose() {
    _listener.dispose();
    super.dispose();
  }

  Future<void> _checkPermission() async {
    final status = await widget.permission.status;
    setState(() {
      this.status = status;
    });
  }

  @override
  Widget build(BuildContext context) {
    return widget.builder(context, status);
  }
}
```
