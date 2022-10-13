---
title: "shell_route"
emoji: "🐬"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

https://codewithandrea.com/articles/flutter-bottom-navigation-bar-nested-routes-gorouter-beamer/

```dart

import 'package:flutter/material.dart';

class DetailsScreen extends StatelessWidget {
  const DetailsScreen({
    super.key,
    required this.icon,
  });

  final Icon icon;

  @override
  Widget build(BuildContext context) {
    return Center(
      child: icon,
    );
  }
}




import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';
import 'package:shell_route/details_screen.dart';
import 'package:shell_route/root_screen.dart';

import 'scaffold_with_shell_nav_bar.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    // private navigators
    final rootNavigatorKey = GlobalKey<NavigatorState>();
    final shellNavigatorKey = GlobalKey<NavigatorState>();

    final router = GoRouter(
      initialLocation: '/b',
      navigatorKey: rootNavigatorKey,
      routes: [
        ShellRoute(
          navigatorKey: shellNavigatorKey,
          builder: (context, state, child) {
            return ScaffoldWithShellNavBar(child: child);
          },
          routes: [
            GoRoute(
              path: '/a',
              pageBuilder: (context, state) => NoTransitionPage<dynamic>(
                child: RootScreen(label: 'a'),
              ),
              routes: [
                GoRoute(
                  path: 'details',
                  builder: (context, state) => DetailsScreen(
                    icon: const Icon(Icons.ramen_dining),
                  ),
                ),
              ],
            ),
            GoRoute(
              path: '/b',
              pageBuilder: (context, state) => NoTransitionPage<dynamic>(
                child: RootScreen(label: 'b'),
              ),
              routes: [
                GoRoute(
                  path: 'details',
                  builder: (context, state) => DetailsScreen(
                    icon: const Icon(Icons.ramen_dining_outlined),
                  ),
                ),
              ],
            ),
          ],
        ),
      ],
    );

    return MaterialApp.router(
      routeInformationProvider: router.routeInformationProvider,
      routeInformationParser: router.routeInformationParser,
      routerDelegate: router.routerDelegate,
      title: 'shell route sample',
    );
  }
}



import 'package:flutter/material.dart';

class RootScreen extends StatelessWidget {
  const RootScreen({
    super.key,
    required this.label,
  });

  final String label;

  @override
  Widget build(BuildContext context) {
    return Column(
      mainAxisAlignment: MainAxisAlignment.spaceAround,
      children: <Widget>[
        Text(label),
      ],
    );
  }
}



import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

class ScaffoldWithShellNavBar extends StatefulWidget {
  const ScaffoldWithShellNavBar({super.key, required this.child});
  final Widget child;

  @override
  State<ScaffoldWithShellNavBar> createState() => _ScaffoldWithNavBarState();
}

class _ScaffoldWithNavBarState extends State<ScaffoldWithShellNavBar> {
  int get _currentIndex => _locationToTabIndex(GoRouter.of(context).location);

  int _locationToTabIndex(String location) {
    final index =
        tabs.indexWhere((t) => location.startsWith(t.initialLocation));
    return index < 0 ? 0 : index;
  }

  void _onItemTapped(BuildContext context, int tabIndex) {
    if (tabIndex != _currentIndex) {
      context.go(tabs[tabIndex].initialLocation);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: widget.child,
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: _currentIndex,
        items: tabs,
        onTap: (index) => _onItemTapped(context, index),
      ),
    );
  }
}

class ScaffoldWithShellNavBarTabItem extends BottomNavigationBarItem {
  const ScaffoldWithShellNavBarTabItem({
    required this.initialLocation,
    required super.icon,
    super.label,
  });
  final String initialLocation;
}

const tabs = [
  ScaffoldWithShellNavBarTabItem(
    initialLocation: '/a',
    icon: Icon(Icons.home),
    label: 'Section A',
  ),
  ScaffoldWithShellNavBarTabItem(
    initialLocation: '/b',
    icon: Icon(Icons.settings),
    label: 'Section B',
  ),
];


```