###如何获取statefulwidget的state
* findAncestorStateOfType （参考overlay state ）

```dart
static OverlayState? of(
    BuildContext context, {
    bool rootOverlay = false,
    Widget? debugRequiredFor,
  }) {
    final OverlayState? result = rootOverlay
        ? context.findRootAncestorStateOfType<OverlayState>()
        : context.findAncestorStateOfType<OverlayState>();
    return result;
  }

```
> context.findRootAncestorStateOfType<OverlayState>() 或者 context.findAncestorStateOfType<OverlayState>()进行查找

* 通过GlobalKey.currentState

assign a [GlobalKey] to the [Overlay] widget and obtain the [OverlayState] via [GlobalKey.currentState]

1. 给state设置GlobalKey
2. 通过GlobalKey.currentState获取state