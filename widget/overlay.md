# overlay

###简介

A [Stack] of entries that can be managed independently.
Overlays let independent child widgets "float" visual elements on top of
other widgets by inserting them into the overlay's [Stack]. The overlay lets
each of these widgets manage their participation in the overlay using [OverlayEntry] objects.

```
1. 独立的栈式布局
2. 通过插入overlay布局栈，使独立的子组件的可视化元素__悬浮在其它组件上面__
3. 这些元素通过【OverlayEntry】进行各自管理
```

Although you can create an [Overlay] directly, it's most common to use the
overlay created by the [Navigator] in a [WidgetsApp] or a [MaterialApp]. The
navigator uses its overlay to manage the visual appearance of its routes.

```
1. 大部分情况通过Navigator创建Overlay
2. navigator用overlay去管理路由的可视化部分
```

###代码解析

```dart
@override
  Widget build(BuildContext context) {
    // This list is filled backwards and then reversed below before
    // it is added to the tree.
    final List<Widget> children = <Widget>[];
    bool onstage = true;
    int onstageCount = 0;
    for (int i = _entries.length - 1; i >= 0; i -= 1) {
      final OverlayEntry entry = _entries[i];
      if (onstage) {
        onstageCount += 1;
        children.add(_OverlayEntryWidget(
          key: entry._key,
          entry: entry,
        ));
        if (entry.opaque)
          onstage = false;
      } else if (entry.maintainState) {
        children.add(_OverlayEntryWidget(
          key: entry._key,
          entry: entry,
          tickerEnabled: false,
        ));
      }
    }
    /// 关于_Theatre的说明
    /// Special version of a [Stack], that doesn't layout and render the first
    /// [skipCount] children.
    ///
    /// The first [skipCount] children are considered "offstage".
    return _Theatre(
      skipCount: children.length - onstageCount,
      children: children.reversed.toList(growable: false),
      clipBehavior: widget.clipBehavior,
    );
  }
```
> overlay从逻辑上可以看做一个特殊的stack

###OverlayEntry
* 通过OverlayState.insert或者OverlayState.insertAll插入到Overlay中
* 通过Overlay.of找到相关联的通过OverlayState
* Positioned 和 AnimatedPositioned 是可用的，由此可实现拖动效果。
* 如果一个不透明的组件覆盖了了overlay,那么overlay将不会在widget树中继续存在，特别是在overlay中的stateful widgets，
假如想要继续保存实例，设置__maintainState__为true. 这回增大开销，所以要小心使用

###通过navigator创建overlay