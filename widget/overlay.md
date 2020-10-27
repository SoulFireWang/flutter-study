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
    // 这里是找出了几个要显示的页面（onstage，因为路由站中有些页面可能是部分显示的弹出框，或者背景透明的）,
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

观察navigator中的build方法，可以看到将Overlay作为布局方法。
_allRouteOverlayEntries返回了OverlayEntry列表
```dart
    @override
  Widget build(BuildContext context) {
    assert(!_debugLocked);
    assert(_history.isNotEmpty);
    // Hides the HeroControllerScope for the widget subtree so that the other
    // nested navigator underneath will not pick up the hero controller above
    // this level.
    return HeroControllerScope.none(
      child: Listener(
        onPointerDown: _handlePointerDown,
        onPointerUp: _handlePointerUpOrCancel,
        onPointerCancel: _handlePointerUpOrCancel,
        child: AbsorbPointer(
          absorbing: false, // it's mutated directly by _cancelActivePointers above
          child: FocusScope(
            node: focusScopeNode,
            autofocus: true,
            child: Overlay(
              key: _overlayKey,
              initialEntries: overlay == null ?  _allRouteOverlayEntries.toList(growable: false) : const <OverlayEntry>[],
            ),
          ),
        ),
      ),
    );
  }
```
#####什么是_allRouteOverlayEntries
```dart
 Iterable<OverlayEntry> get _allRouteOverlayEntries sync* {
    for (final _RouteEntry entry in _history)
      yield* entry.route.overlayEntries;
  }
```
还是调用到NavigatorState的_history
```dart
List<_RouteEntry> _history = <_RouteEntry>[];
```
那么_history中的_RouteEntry从何而来

```dart
Future<T> push<T>(Route<T> route) {
    assert(!_debugLocked);
    assert(() {
      _debugLocked = true;
      return true;
    }());
    assert(route != null);
    assert(route._navigator == null);
    /// 在这里加入到_history中
    _history.add(_RouteEntry(route, initialState: _RouteLifecycle.push));
    _flushHistoryUpdates();
    assert(() {
      _debugLocked = false;
      return true;
    }());
    _afterNavigation(route);
    return route.popped;
  }
```

##### 什么时候生成的overlayEntries
我们注意到overlayEntries在实例化的时候为空
```dart
  /// Called when the route is inserted into the navigator.
  ///
  /// Uses this to populate [overlayEntries]. There must be at least one entry in
  /// this list after [install] has been invoked. The [Navigator] will be in charge
  /// to add them to the [Overlay] or remove them from it by calling
  /// [OverlayEntry.remove].
  @protected
  @mustCallSuper
  void install() { }
```
这里表述在调用install的时候会生成overlayEntries
有以下两个问题
1. __install里面又干了什么？__
2. __那么install在哪里调用？__

####先来探究__问题1__，我们看到install作为抽象类Route的方法，其实现一定在Route的子类中
####调用逻辑
__abstract class ModalRoute<T> extends TransitionRoute<T>__
```dart
@override
  void install() {
    super.install();
    _animationProxy = ProxyAnimation(super.animation);
    _secondaryAnimationProxy = ProxyAnimation(super.secondaryAnimation);
  }

  ///在父类OverlayRoute中被调用
  @override
  Iterable<OverlayEntry> createOverlayEntries() sync* {
    /// 生成蒙版overlay
    yield _modalBarrier = OverlayEntry(builder: _buildModalBarrier);

    /// 生成啥也不知道了
    yield _modalScope = OverlayEntry(builder: _buildModalScope, maintainState: maintainState);
  }
```

__abstract class TransitionRoute<T> extends OverlayRoute<T>__

实例化了动画
```dart
@override
  void install() {
    assert(!_transitionCompleter.isCompleted, 'Cannot install a $runtimeType after disposing it.');
    _controller = createAnimationController();
    assert(_controller != null, '$runtimeType.createAnimationController() returned null.');
    _animation = createAnimation()
      ..addStatusListener(_handleStatusChanged);
    assert(_animation != null, '$runtimeType.createAnimation() returned null.');
    super.install();
    if (_animation!.isCompleted && overlayEntries.isNotEmpty) {
      overlayEntries.first.opaque = opaque;
    }
  }
```

__abstract class OverlayRoute<T> extends Route<T>__
```dart
@override
  void install() {
    assert(_overlayEntries.isEmpty);
    /// 这里
    _overlayEntries.addAll(createOverlayEntries());
    super.install();
  }
```
下面要探究createOverlayEntries干了什么，那么跳回回到之前的modalroute可以就看到生成了2个OverlayEntry
```dart
/// Subclasses should override this getter to return the builders for the overlay.
@factory
Iterable<OverlayEntry> createOverlayEntries();
```

>这里做一下补充：我们查看MaterialPageRoute 或者 PageRouteBuilder（两个常用的路由实例对象）都可以看到其父类都是ModalRoute

####问题2：那么install在什么时候被调用呢（这里盲猜：肯定是在NavigatorState的push方法中）
```dart
@optionalTypeArgs
  Future<T> push<T>(Route<T> route) {
    assert(!_debugLocked);
    assert(() {
      _debugLocked = true;
      return true;
    }());
    assert(route != null);
    assert(route._navigator == null);
    _history.add(_RouteEntry(route, initialState: _RouteLifecycle.push));
    _flushHistoryUpdates();
    assert(() {
      _debugLocked = false;
      return true;
    }());
    _afterNavigation(route);
    return route.popped;
  }


void _flushHistoryUpdates({bool rearrangeOverlay = true}) {
    assert(_debugLocked && !_debugUpdatingPage);
    // Clean up the list, sending updates to the routes that changed. Notably,
    // we don't send the didChangePrevious/didChangeNext updates to those that
    // did not change at this point, because we're not yet sure exactly what the
    // routes will be at the end of the day (some might get disposed).
    int index = _history.length - 1;
    _RouteEntry? next;
    _RouteEntry? entry = _history[index];
    _RouteEntry? previous = index > 0 ? _history[index - 1] : null;
    bool canRemoveOrAdd =
        false; // Whether there is a fully opaque route on top to silently remove or add route underneath.
    Route<dynamic>?
        poppedRoute; // The route that should trigger didPopNext on the top active route.
    bool seenTopActiveRoute =
        false; // Whether we've seen the route that would get didPopNext.
    final List<_RouteEntry> toBeDisposed = <_RouteEntry>[];
    while (index >= 0) {
      switch (entry!.currentState) {
        case _RouteLifecycle.add:
          assert(rearrangeOverlay);
          entry.handleAdd(
            navigator: this,
            previousPresent:
                _getRouteBefore(index - 1, _RouteEntry.isPresentPredicate)
                    ?.route,
          );
...
      }
    }
}

class _RouteEntry extends RouteTransitionRecord{
    void handleAdd(
      {required NavigatorState navigator,
      required Route<dynamic>? previousPresent}) {
    assert(currentState == _RouteLifecycle.add);
    assert(navigator != null);
    assert(navigator._debugLocked);
    assert(route._navigator == null);
    route._navigator = navigator;
    route.install();
    assert(route.overlayEntries.isNotEmpty);
    currentState = _RouteLifecycle.adding;
    navigator._observedRouteAdditions
        .add(_NavigatorPushObservation(route, previousPresent));
  }
}


```
