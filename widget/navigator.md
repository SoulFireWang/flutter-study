#navigator
###简介
A widget that manages a set of child widgets with a stack discipline.

Many apps have a navigator near the top of their widget hierarchy in order
to display their logical history using an [Overlay] with the most recently
visited pages visually on top of the older pages. Using this pattern lets
the navigator visually transition from one page to another by moving the widgets
around in the overlay. Similarly, the navigator can be used to show a dialog
by positioning the dialog widget above the current page.

```
1. 使用[Overlay]去显示页面的层级，
2. 通过这种方式，能展示页面切换的逻辑和过程。
3. 同时，也能在当前页面中展示对话框
```

#### Using the Navigator API（使用Navigator接口）

Mobile apps typically reveal their contents via full-screen elements
called "screens" or "pages". In Flutter these elements are called
routes and they're managed by a [Navigator] widget. The navigator
manages a stack of [Route] objects and provides two ways for managing
the stack, the declarative API [Navigator.pages] or imperative API
[Navigator.push] and [Navigator.pop].

```
1. routes 用于展示应用内容，等价于“screens” 或者 “pages”（全屏组件）
2. navigator管理routes
3. 两种管理方式
   声明式API Navigator.pages
   即时API Navigator.push 和 Navigator.pop
```

#### Using the Pages API（使用Pages接口）

The [Navigator] will convert its [Navigator.pages] into a stack of [Route]s
if it is provided. A change in [Navigator.pages] will trigger an update to
the stack of [Route]s. The [Navigator] will update its routes to match the
new configuration of its [Navigator.pages]. To use this API, one can create
a [Page] subclass and defines a list of [Page]s for [Navigator.pages]. A
[Navigator.onPopPage] callback is also required to properly clean up the
input pages in case of a pop.

```
1. 将Navigator.pages转变为Route栈
2. Navigator.pages的修改会触发Route的更新
3. 如何使用这个API
   a. 继承Page类
   b. 为Navigator.pages定义一个Page类型的列表
   c. 实现Navigator.onPopPage，在页面退出的时候正确删除Page
```

By Default, the [Navigator] will use [DefaultTransitionDelegate] to decide
how routes transition in or out of the screen. To customize it, define a
[TransitionDelegate] subclass and provide it to the
[Navigator.transitionDelegate].

```
1. 通过DefaultTransitionDelegate去决定页面切换动画
2. 可以设置Navigator.transitionDelegate来自定义页面切换动画
```

#### Displaying a full-screen route（显示一个全屏页面）

Although you can create a navigator directly, it's most common to use the
navigator created by the `Router` which itself is created and configured by
a [WidgetsApp] or a [MaterialApp] widget. You can refer to that navigator
with [Navigator.of].

```
1. 不推荐自己创建navigator，
   大部分情况下用Router（被WidgetsApp或MaterialApp所创建）创建的navigator。
2. 能通过Navigator.of访问
```

A [MaterialApp] is the simplest way to set things up. The [MaterialApp]'s
home becomes the route at the bottom of the [Navigator]'s stack. It is what
you see when the app is launched.
```
MaterialApp 是最简单的navigator实例化方法，MaterialApp的home就是Navigator的栈底
```

#### Using named navigator routes（使用名称进行路由）

Mobile apps often manage a large number of routes and it's often
easiest to refer to them by name. Route names, by convention,
use a path-like structure (for example, '/a/b/c').
The app's home page route is named '/' by default.

```
1. 使用名称去路由最方便
2. 用 path-like 结构作为路由名称（例如：'/a/b/c' ）
3. 回到APP主页可用默认用'/'
```

#####例如

```dart
void main() {
   runApp(MaterialApp(
     home: MyAppHome(), // becomes the route named '/'
     routes: <String, WidgetBuilder> {
       '/a': (BuildContext context) => MyPage(title: 'page A'),
       '/b': (BuildContext context) => MyPage(title: 'page B'),
       '/c': (BuildContext context) => MyPage(title: 'page C'),
     },
   ));
 }
```

#####使用名称进行路由
 ```dart
 Navigator.pushNamed(context, '/b');
 ```

#### Routes can return a value（路由返回值）

When a route is pushed to ask the user for a value, the value can be
returned via the [pop] method's result parameter.

Methods that push a route return a [Future]. The Future resolves when the
route is popped and the [Future]'s value is the [pop] method's `result`
parameter.

```
Navigator.push会返回Future，会在pop时调用Future的resolve，pop的result
可以作为返回参数
```

#####例如

```dart
 bool value = await Navigator.push(context, MaterialPageRoute<bool>(
   builder: (BuildContext context) {
     return Center(
       child: GestureDetector(
         child: Text('OK'),
         onTap: () { Navigator.pop(context, true); }
       ),
     );
   }
 ));
```


#### Popup routes（弹出路由）

 Routes don't have to obscure the entire screen. [PopupRoute]s cover the
 screen with a [ModalRoute.barrierColor] that can be only partially opaque to
 allow the current screen to show through. Popup routes are "modal" because
 they block input to the widgets below.

 ```
 1. PopupRoute 不比遮盖整个屏幕， 
 2. 可以通过ModalRoute.barrierColor遮罩弹框在屏幕中的其余部分，阻止页面中其余部分的交互响应
 ```


 
 There are functions which create and show popup routes. For
 example: [showDialog], [showMenu], and [showModalBottomSheet]. These
 functions return their pushed route's Future as described above.
 Callers can await the returned value to take an action when the
 route is popped, or to discover the route's value.

There are also widgets which create popup routes, like [PopupMenuButton] and
 [DropdownButton]. These widgets create internal subclasses of PopupRoute
 and use the Navigator's push and pop methods to show and dismiss them.
 ```
 1. 弹出型路由方法：
    a. showDialog
    b. showMenu
    c. showModalBottomSheet
 2. 同时也有自己创建弹出路由的组件，像 PopupMenuButton 和 DropdownButton。
    这些组件在内部创建PopupRoute的子类，并用Navigator's push and pop去显示或者隐藏他们
 ```

#### Custom routes（自定义路由）

 You can create your own subclass of one of the widget library route classes
 like [PopupRoute], [ModalRoute], or [PageRoute], to control the animated
 transition employed to show the route, the color and behavior of the route's
 modal barrier, and other aspects of the route.

```
1. 你能自己创建子类来控制路由动画的变化，路由蒙版的颜色和行为，还有路由的其它方面
```

 The [PageRouteBuilder] class makes it possible to define a custom route
 in terms of callbacks. Here's an example that rotates and fades its child
 when the route appears or disappears. This route does not obscure the entire
 screen because it specifies `opaque: false`, just as a popup route does.

```
PageRouteBuilder可以通过回调的方式自定义路由
```

```dart
Navigator.push(context, PageRouteBuilder(
  opaque: false,
  pageBuilder: (BuildContext context, _, __) {

    /// 居中显示
    return Center(child: Text('My PageRoute'));
  },
  // 定义了路由变换
  transitionsBuilder: (___, Animation<double> animation, ____, Widget child) {
    return FadeTransition(
      opacity: animation,
      child: RotationTransition(
        turns: Tween<double>(begin: 0.5, end: 1.0).animate(animation),
        child: child,
      ),
    );
  }
));
```

#### Nesting Navigators (嵌套路由)

An app can use more than one [Navigator]. Nesting one [Navigator] below
another [Navigator] can be used to create an "inner journey" such as tabbed
navigation, user registration, store checkout, or other independent journeys
that represent a subsection of your overall application.

```
路由嵌套的用法：tabbed navigation、用户注册、结账
```

例如对于iOS来说，tab条的每一个分项都有自己的navigator，称之为并行导航管理。
另外对于并行导航管理，仍然需要完全覆盖tab页的全屏路由。例如一个登陆流程，或者一个弹框。因此必须
存在一个根路由，tab路由作为该根路由的子路由。

#####示例
```dart
 class MyApp extends StatelessWidget {
   @override
   Widget build(BuildContext context) {
     return MaterialApp(
       title: 'Flutter Code Sample for Navigator',
       // MaterialApp contains our top-level Navigator
       initialRoute: '/',
       routes: {
         '/': (BuildContext context) => HomePage(),
         '/signup': (BuildContext context) => SignUpPage(),
       },
     );
   }
 }

 class HomePage extends StatelessWidget {
   @override
   Widget build(BuildContext context) {
     return DefaultTextStyle(
       style: Theme.of(context).textTheme.headline4,
       child: Container(
         color: Colors.white,
         alignment: Alignment.center,
         child: Text('Home Page'),
       ),
     );
   }
 }

 class CollectPersonalInfoPage extends StatelessWidget {
   @override
   Widget build(BuildContext context) {
     return DefaultTextStyle(
       style: Theme.of(context).textTheme.headline4,
       child: GestureDetector(
         onTap: () {
           // This moves from the personal info page to the credentials page,
           // replacing this page with that one.
           Navigator.of(context)
             .pushReplacementNamed('signup/choose_credentials');
         },
         child: Container(
           color: Colors.lightBlue,
           alignment: Alignment.center,
           child: Text('Collect Personal Info Page'),
         ),
       ),
     );
   }
 }

 class ChooseCredentialsPage extends StatelessWidget {
   const ChooseCredentialsPage({
     this.onSignupComplete,
   });

   final VoidCallback onSignupComplete;

   @override
   Widget build(BuildContext context) {
     return GestureDetector(
       onTap: onSignupComplete,
       child: DefaultTextStyle(
         style: Theme.of(context).textTheme.headline4,
         child: Container(
           color: Colors.pinkAccent,
           alignment: Alignment.center,
           child: Text('Choose Credentials Page'),
         ),
       ),
     );
   }
 }

 class SignUpPage extends StatelessWidget {
   @override
   Widget build(BuildContext context) {
     // SignUpPage builds its own Navigator which ends up being a nested
     // Navigator in our app.
     return Navigator(
       initialRoute: 'signup/personal_info',
       onGenerateRoute: (RouteSettings settings) {
         WidgetBuilder builder;
         switch (settings.name) {
           case 'signup/personal_info':
           // Assume CollectPersonalInfoPage collects personal info and then
           // navigates to 'signup/choose_credentials'.
             builder = (BuildContext _) => CollectPersonalInfoPage();
             break;
           case 'signup/choose_credentials':
           // Assume ChooseCredentialsPage collects new credentials and then
           // invokes 'onSignupComplete()'.
             builder = (BuildContext _) => ChooseCredentialsPage(
               onSignupComplete: () {
                 // Referencing Navigator.of(context) from here refers to the
                 // top level Navigator because SignUpPage is above the
                 // nested Navigator that it created. Therefore, this pop()
                 // will pop the entire "sign up" journey and return to the
                 // "/" route, AKA HomePage.
                 Navigator.of(context).pop();
               },
             );
             break;
           default:
             throw Exception('Invalid route: ${settings.name}');
         }
         return MaterialPageRoute(builder: builder, settings: settings);
       },
     );
   }
 }
```