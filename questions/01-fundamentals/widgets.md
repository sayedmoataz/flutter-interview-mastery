# Widgets & UI - Interview Questions

## Table of Contents
1. [What is a Widget?](#q1-what-is-a-widget)
2. [StatelessWidget vs StatefulWidget](#q2-statelesswidget-vs-statefulwidget)
3. [Widget Lifecycle](#q3-widget-lifecycle)
4. [BuildContext Explained](#q4-buildcontext-explained)
5. [Keys in Flutter](#q5-keys-in-flutter)
6. [const Constructors](#q6-const-constructors)
7. [Widget Tree vs Element Tree vs RenderObject Tree](#q7-widget-tree-vs-element-tree-vs-renderobject-tree)
8. [Common Widgets](#q8-common-widgets)
9. [Custom Widgets Best Practices](#q9-custom-widgets-best-practices)
10. [Widget Performance](#q10-widget-performance)

---

## Q1: What is a Widget?

**Level:** Junior  
**Category:** Widgets  
**Difficulty:** ⭐

### The Question
What is a widget in Flutter? What's the difference between widgets and views in other frameworks?

### Answer

**Widget** is the basic building block of Flutter UI. Everything in Flutter is a widget - from a button to padding to the entire app.

**Key Concept:** Widgets are **immutable** (can't be changed). When you want to change something, Flutter creates a new widget.

### Code Example

```dart
import 'package:flutter/material.dart';

// Everything is a widget!
void main() {
  runApp(MyApp()); // App is a widget
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp( // MaterialApp is a widget
      home: Scaffold( // Scaffold is a widget
        appBar: AppBar( // AppBar is a widget
          title: Text('Everything is a Widget'), // Text is a widget
        ),
        body: Center( // Center is a widget
          child: Container( // Container is a widget
            padding: EdgeInsets.all(16), // Padding is a widget (EdgeInsets is not)
            child: Column( // Column is a widget
              children: [
                Icon(Icons.star), // Icon is a widget
                Text('Hello Flutter'), // Text is a widget
                ElevatedButton( // Button is a widget
                  onPressed: () {},
                  child: Text('Press Me'),
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }
}
```

### Widget Categories

```dart
// 1. Structural Widgets - Layout and structure
Scaffold(
  appBar: AppBar(),
  body: Container(),
  bottomNavigationBar: BottomNavigationBar(),
)

// 2. Layout Widgets - Arrange children
Column(
  children: [
    Row(children: []),
    Stack(children: []),
  ],
)

// 3. UI Widgets - Visual elements
Text('Hello')
Icon(Icons.home)
Image.network('url')

// 4. Interactive Widgets - User interaction
ElevatedButton(onPressed: () {}, child: Text('Click'))
TextField()
GestureDetector(onTap: () {})

// 5. Styling Widgets - Appearance
Container(
  decoration: BoxDecoration(color: Colors.blue),
  padding: EdgeInsets.all(8),
)
```

### Widgets vs Other Frameworks

| Flutter Widgets | Android Views | iOS UIKit | React Components |
|----------------|---------------|-----------|------------------|
| Immutable | Mutable | Mutable | Can be both |
| Declarative | Imperative | Imperative | Declarative |
| Lightweight | Heavy | Heavy | Lightweight |
| Rebuilt often | Updated in place | Updated in place | Virtual DOM |

### Why Widgets are Immutable

```dart
// ❌ This doesn't work - widgets are immutable
class WrongExample extends StatelessWidget {
  final Text myText = Text('Hello');
  
  void changeText() {
    // myText.data = 'World'; // ERROR! Can't modify widget
  }
  
  @override
  Widget build(BuildContext context) => myText;
}

// ✅ Create new widget instead
class CorrectExample extends StatefulWidget {
  @override
  State<CorrectExample> createState() => _CorrectExampleState();
}

class _CorrectExampleState extends State<CorrectExample> {
  String text = 'Hello';
  
  void changeText() {
    setState(() {
      text = 'World'; // Change state, Flutter creates new Text widget
    });
  }
  
  @override
  Widget build(BuildContext context) {
    return Text(text); // New widget created with new text
  }
}
```

### Best Practices
- Keep widgets small and focused
- Extract reusable widgets
- Use const constructors when possible
- Compose widgets instead of inheriting

---

## Q2: StatelessWidget vs StatefulWidget

**Level:** Junior  
**Category:** Widgets  
**Difficulty:** ⭐⭐

### The Question
What's the difference between StatelessWidget and StatefulWidget? When should you use each?

### Answer

**StatelessWidget** - Doesn't change over time. Built once with given parameters.

**StatefulWidget** - Can change over time. Has mutable state managed by State object.

### Code Example

```dart
// StatelessWidget - Never changes
class Greeting extends StatelessWidget {
  final String name;
  
  const Greeting({super.key, required this.name});
  
  @override
  Widget build(BuildContext context) {
    // Built once with the name, never changes
    return Text('Hello, $name!');
  }
}

// Usage
Greeting(name: 'John') // Always shows "Hello, John!"
```

```dart
// StatefulWidget - Can change
class Counter extends StatefulWidget {
  const Counter({super.key});
  
  @override
  State<Counter> createState() => _CounterState();
}

class _CounterState extends State<Counter> {
  int _count = 0; // Mutable state
  
  void _increment() {
    setState(() {
      _count++; // State changes, widget rebuilds
    });
  }
  
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Count: $_count'),
        ElevatedButton(
          onPressed: _increment,
          child: const Text('Increment'),
        ),
      ],
    );
  }
}
```

### Detailed Comparison

```dart
// StatelessWidget structure
class MyStatelessWidget extends StatelessWidget {
  // 1. Properties are final (immutable)
  final String title;
  final Color color;
  
  // 2. Constructor
  const MyStatelessWidget({
    super.key,
    required this.title,
    this.color = Colors.blue,
  });
  
  // 3. Only build method
  @override
  Widget build(BuildContext context) {
    return Container(
      color: color,
      child: Text(title),
    );
  }
}

// StatefulWidget structure
class MyStatefulWidget extends StatefulWidget {
  // 1. Properties are also final
  final String initialTitle;
  
  // 2. Constructor
  const MyStatefulWidget({super.key, required this.initialTitle});
  
  // 3. Create state
  @override
  State<MyStatefulWidget> createState() => _MyStatefulWidgetState();
}

// Separate State class
class _MyStatefulWidgetState extends State<MyStatefulWidget> {
  // 1. Mutable state variables
  late String _currentTitle;
  bool _isPressed = false;
  
  // 2. Lifecycle methods
  @override
  void initState() {
    super.initState();
    _currentTitle = widget.initialTitle; // Access widget properties
  }
  
  // 3. State modification methods
  void _changeTitle(String newTitle) {
    setState(() {
      _currentTitle = newTitle;
    });
  }
  
  // 4. Build method
  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: () => setState(() => _isPressed = !_isPressed),
      child: Container(
        color: _isPressed ? Colors.blue : Colors.grey,
        child: Text(_currentTitle),
      ),
    );
  }
}
```

### When to Use Each

```dart
// ✅ Use StatelessWidget when:

// 1. Displaying static data
class UserProfile extends StatelessWidget {
  final String name;
  final String email;
  
  const UserProfile({super.key, required this.name, required this.email});
  
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text(name),
        Text(email),
      ],
    );
  }
}

// 2. Pure UI components
class Logo extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Image.asset('assets/logo.png');
  }
}

// 3. Layout widgets
class CenteredCard extends StatelessWidget {
  final Widget child;
  
  const CenteredCard({super.key, required this.child});
  
  @override
  Widget build(BuildContext context) {
    return Center(
      child: Card(child: child),
    );
  }
}

// ✅ Use StatefulWidget when:

// 1. User input changes state
class LoginForm extends StatefulWidget {
  @override
  State<LoginForm> createState() => _LoginFormState();
}

class _LoginFormState extends State<LoginForm> {
  String _email = '';
  String _password = '';
  
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        TextField(
          onChanged: (value) => setState(() => _email = value),
        ),
        TextField(
          onChanged: (value) => setState(() => _password = value),
          obscureText: true,
        ),
      ],
    );
  }
}

// 2. Animations
class FadeInWidget extends StatefulWidget {
  final Widget child;
  
  const FadeInWidget({super.key, required this.child});
  
  @override
  State<FadeInWidget> createState() => _FadeInWidgetState();
}

class _FadeInWidgetState extends State<FadeInWidget>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _animation;
  
  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(seconds: 2),
      vsync: this,
    );
    _animation = Tween<double>(begin: 0, end: 1).animate(_controller);
    _controller.forward();
  }
  
  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
  
  @override
  Widget build(BuildContext context) {
    return FadeTransition(
      opacity: _animation,
      child: widget.child,
    );
  }
}

// 3. Timers and async operations
class CountdownTimer extends StatefulWidget {
  final int seconds;
  
  const CountdownTimer({super.key, required this.seconds});
  
  @override
  State<CountdownTimer> createState() => _CountdownTimerState();
}

class _CountdownTimerState extends State<CountdownTimer> {
  late int _remaining;
  Timer? _timer;
  
  @override
  void initState() {
    super.initState();
    _remaining = widget.seconds;
    _startTimer();
  }
  
  void _startTimer() {
    _timer = Timer.periodic(const Duration(seconds: 1), (timer) {
      if (_remaining > 0) {
        setState(() => _remaining--);
      } else {
        timer.cancel();
      }
    });
  }
  
  @override
  void dispose() {
    _timer?.cancel();
    super.dispose();
  }
  
  @override
  Widget build(BuildContext context) {
    return Text('Time remaining: $_remaining seconds');
  }
}
```

### Performance Tip

```dart
// ❌ Unnecessary StatefulWidget
class BadExample extends StatefulWidget {
  final String title;
  
  const BadExample({super.key, required this.title});
  
  @override
  State<BadExample> createState() => _BadExampleState();
}

class _BadExampleState extends State<BadExample> {
  @override
  Widget build(BuildContext context) {
    // No state changes, should be StatelessWidget!
    return Text(widget.title);
  }
}

// ✅ Use StatelessWidget instead
class GoodExample extends StatelessWidget {
  final String title;
  
  const GoodExample({super.key, required this.title});
  
  @override
  Widget build(BuildContext context) {
    return Text(title);
  }
}
```

### Comparison Table

| Feature | StatelessWidget | StatefulWidget |
|---------|----------------|----------------|
| State | Immutable | Mutable |
| Rebuilds | When parent rebuilds | When setState called |
| Memory | Less | More (State object) |
| Use case | Static UI | Dynamic UI |
| Performance | Faster | Slightly slower |
| Lifecycle methods | Only build | initState, dispose, etc. |

---

## Q3: Widget Lifecycle

**Level:** Junior-Mid  
**Category:** Widgets  
**Difficulty:** ⭐⭐⭐

### The Question
What is the lifecycle of a StatefulWidget? Explain each lifecycle method.

### Answer

StatefulWidget has several lifecycle methods called in specific order. Understanding these is crucial for proper resource management.

### Lifecycle Flow Diagram

```
[Widget Created]
      |
      v
  createState() ← Called once
      |
      v
  initState() ← Called once, initialize data
      |
      v
  didChangeDependencies() ← Called after initState and when dependencies change
      |
      v
    build() ← Called every time widget needs to rebuild
      |
      |--→ setState() ← Triggers rebuild
      |       |
      |←------|
      |
      v
  didUpdateWidget() ← Called when widget configuration changes
      |
      v
    build()
      |
      v
  deactivate() ← Widget removed from tree (temporarily)
      |
      v
   dispose() ← Widget removed permanently, cleanup
      |
      v
  [Widget Destroyed]
```

### Code Example

```dart
import 'package:flutter/material.dart';
import 'dart:async';

class LifecycleDemo extends StatefulWidget {
  final String title;
  
  const LifecycleDemo({super.key, required this.title});
  
  @override
  State<LifecycleDemo> createState() {
    print('1. createState()');
    return _LifecycleDemoState();
  }
}

class _LifecycleDemoState extends State<LifecycleDemo> {
  int _counter = 0;
  Timer? _timer;
  
  // 1️⃣ INITIALIZATION
  @override
  void initState() {
    super.initState();
    print('2. initState() - Initialize data here');
    
    // ✅ Do: Initialize variables, start animations, subscribe to streams
    _counter = 0;
    _timer = Timer.periodic(Duration(seconds: 1), (timer) {
      print('Timer tick: ${timer.tick}');
    });
    
    // ❌ Don't: Use BuildContext here (not ready yet)
    // Don't: Call setState (widget not built yet)
  }
  
  // 2️⃣ DEPENDENCIES
  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    print('3. didChangeDependencies() - Dependencies changed');
    
    // ✅ Do: Access InheritedWidget, MediaQuery, Theme
    final theme = Theme.of(context);
    final mediaQuery = MediaQuery.of(context);
    print('Theme brightness: ${theme.brightness}');
    print('Screen width: ${mediaQuery.size.width}');
    
    // Called:
    // - After initState (first time)
    // - When InheritedWidget changes
  }
  
  // 3️⃣ BUILD
  @override
  Widget build(BuildContext context) {
    print('4. build() - Building widget tree');
    
    return Scaffold(
      appBar: AppBar(title: Text(widget.title)),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Counter: $_counter'),
            Text('Title: ${widget.title}'),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          setState(() {
            _counter++;
            print('setState() called - Will trigger build()');
          });
        },
        child: const Icon(Icons.add),
      ),
    );
  }
  
  // 4️⃣ UPDATE
  @override
  void didUpdateWidget(LifecycleDemo oldWidget) {
    super.didUpdateWidget(oldWidget);
    print('5. didUpdateWidget() - Widget config changed');
    print('   Old title: ${oldWidget.title}');
    print('   New title: ${widget.title}');
    
    // ✅ Do: Compare old and new widget properties
    if (oldWidget.title != widget.title) {
      print('   Title changed! React to it.');
      // Update state based on new properties
    }
  }
  
  // 5️⃣ DEACTIVATION
  @override
  void deactivate() {
    print('6. deactivate() - Widget removed from tree');
    super.deactivate();
    
    // Called when:
    // - Widget moved in the tree
    // - Widget being removed
    // Might be reinserted (don't cleanup yet)
  }
  
  // 6️⃣ CLEANUP
  @override
  void dispose() {
    print('7. dispose() - Widget destroyed, cleanup');
    
    // ✅ Do: Cancel timers, close streams, dispose controllers
    _timer?.cancel();
    
    // ❌ Don't: Call setState (widget being destroyed)
    
    super.dispose();
  }
}
```

### Real-World Example: User Profile Screen

```dart
class UserProfileScreen extends StatefulWidget {
  final String userId;
  
  const UserProfileScreen({super.key, required this.userId});
  
  @override
  State<UserProfileScreen> createState() => _UserProfileScreenState();
}

class _UserProfileScreenState extends State<UserProfileScreen> {
  User? _user;
  bool _isLoading = true;
  StreamSubscription? _userSubscription;
  
  @override
  void initState() {
    super.initState();
    print('[INIT] Fetching user data for ${widget.userId}');
    _loadUserData();
  }
  
  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    // Check if user is online/offline
    final connectivityStatus = ConnectivityProvider.of(context);
    print('[DEPS] Connectivity: $connectivityStatus');
  }
  
  Future<void> _loadUserData() async {
    // Start listening to user updates
    _userSubscription = FirebaseFirestore.instance
        .collection('users')
        .doc(widget.userId)
        .snapshots()
        .listen((snapshot) {
      if (mounted) {
        setState(() {
          _user = User.fromJson(snapshot.data()!);
          _isLoading = false;
        });
      }
    });
  }
  
  @override
  void didUpdateWidget(UserProfileScreen oldWidget) {
    super.didUpdateWidget(oldWidget);
    // User ID changed - reload data
    if (oldWidget.userId != widget.userId) {
      print('[UPDATE] User ID changed: ${oldWidget.userId} → ${widget.userId}');
      setState(() => _isLoading = true);
      _loadUserData();
    }
  }
  
  @override
  void deactivate() {
    print('[DEACTIVATE] User profile being removed');
    super.deactivate();
  }
  
  @override
  void dispose() {
    print('[DISPOSE] Cleaning up user profile');
    _userSubscription?.cancel();
    super.dispose();
  }
  
  @override
  Widget build(BuildContext context) {
    if (_isLoading) {
      return const Center(child: CircularProgressIndicator());
    }
    
    return Scaffold(
      appBar: AppBar(title: Text(_user?.name ?? 'Profile')),
      body: Column(
        children: [
          Text('Email: ${_user?.email}'),
          Text('Phone: ${_user?.phone}'),
        ],
      ),
    );
  }
}

class User {
  final String name;
  final String email;
  final String phone;
  
  User({required this.name, required this.email, required this.phone});
  
  factory User.fromJson(Map<String, dynamic> json) {
    return User(
      name: json['name'],
      email: json['email'],
      phone: json['phone'],
    );
  }
}

class ConnectivityProvider extends InheritedWidget {
  final String status;
  
  const ConnectivityProvider({
    super.key,
    required this.status,
    required super.child,
  });
  
  static String of(BuildContext context) {
    return context.dependOnInheritedWidgetOfExactType<ConnectivityProvider>()!.status;
  }
  
  @override
  bool updateShouldNotify(ConnectivityProvider oldWidget) {
    return status != oldWidget.status;
  }
}
```

### Common Pitfalls

```dart
// ❌ WRONG: Using context in initState
class WrongInitState extends StatefulWidget {
  @override
  State<WrongInitState> createState() => _WrongInitStateState();
}

class _WrongInitStateState extends State<WrongInitState> {
  @override
  void initState() {
    super.initState();
    // ERROR! BuildContext not ready yet
    final theme = Theme.of(context);
  }
  
  @override
  Widget build(BuildContext context) => Container();
}

// ✅ CORRECT: Use didChangeDependencies
class CorrectInitState extends StatefulWidget {
  @override
  State<CorrectInitState> createState() => _CorrectInitStateState();
}

class _CorrectInitStateState extends State<CorrectInitState> {
  late ThemeData _theme;
  
  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    _theme = Theme.of(context); // ✅ Context ready
  }
  
  @override
  Widget build(BuildContext context) => Container();
}

// ❌ WRONG: Calling setState in dispose
class WrongDispose extends StatefulWidget {
  @override
  State<WrongDispose> createState() => _WrongDisposeState();
}

class _WrongDisposeState extends State<WrongDispose> {
  @override
  void dispose() {
    setState(() {}); // ERROR! Widget being destroyed
    super.dispose();
  }
  
  @override
  Widget build(BuildContext context) => Container();
}

// ✅ CORRECT: Check mounted before async setState
class CorrectAsync extends StatefulWidget {
  @override
  State<CorrectAsync> createState() => _CorrectAsyncState();
}

class _CorrectAsyncState extends State<CorrectAsync> {
  String _data = '';
  
  @override
  void initState() {
    super.initState();
    _loadData();
  }
  
  Future<void> _loadData() async {
    await Future.delayed(Duration(seconds: 2));
    
    if (mounted) { // ✅ Check if widget still in tree
      setState(() => _data = 'Loaded');
    }
  }
  
  @override
  Widget build(BuildContext context) => Text(_data);
}
```

### Lifecycle Cheat Sheet

| Method | Called When | Use For | Can Use Context |
|--------|-------------|---------|----------------|
| `createState()` | Once, when widget created | Create State object | ❌ No |
| `initState()` | Once, after createState | Initialize data, start timers | ❌ No |
| `didChangeDependencies()` | After initState & when deps change | Access InheritedWidget | ✅ Yes |
| `build()` | Every rebuild | Build UI | ✅ Yes |
| `didUpdateWidget()` | When widget config changes | React to prop changes | ✅ Yes |
| `setState()` | When you call it | Trigger rebuild | ✅ Yes |
| `deactivate()` | Widget removed (temp) | Rare usage | ✅ Yes |
| `dispose()` | Widget destroyed | Cleanup resources | ❌ No (don't use) |

### Best Practices
- Always call `super` first (except dispose - call super last)
- Check `mounted` before async setState
- Dispose all resources (timers, controllers, subscriptions)
- Don't use BuildContext in initState
- Use didChangeDependencies for context-dependent initialization

---

## Q4: BuildContext Explained

**Level:** Mid  
**Category:** Widgets  
**Difficulty:** ⭐⭐⭐

### The Question
What is BuildContext? How does it work and why is it important?

### Answer

**BuildContext** is a handle to the location of a widget in the widget tree. It's used to access inherited widgets, theme data, and navigate between screens.

**Key Point:** Every widget has its own BuildContext.

### Code Example

```dart
import 'package:flutter/material.dart';

class BuildContextDemo extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // 'context' refers to BuildContextDemo's position in the tree
    
    // ✅ Accessing theme
    final theme = Theme.of(context);
    
    // ✅ Accessing media query
    final mediaQuery = MediaQuery.of(context);
    
    // ✅ Accessing navigator
    Navigator.of(context);
    
    // ✅ Accessing scaffold
    Scaffold.of(context);
    
    return Scaffold(
      appBar: AppBar(
        title: Text(
          'Context Demo',
          style: theme.textTheme.titleLarge, // Using theme from context
        ),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Screen width: ${mediaQuery.size.width}'),
            ElevatedButton(
              onPressed: () {
                Navigator.of(context).push(
                  MaterialPageRoute(builder: (_) => SecondScreen()),
                );
              },
              child: const Text('Navigate'),
            ),
          ],
        ),
      ),
    );
  }
}

class SecondScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) => Scaffold(
    appBar: AppBar(title: const Text('Second Screen')),
    body: Center(
      child: ElevatedButton(
        onPressed: () => Navigator.of(context).pop(),
        child: const Text('Go Back'),
      ),
    ),
  );
}
```

### Understanding Context Scope

```dart
class ContextScopeDemo extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // context here = ContextScopeDemo's context
    
    return Scaffold(
      appBar: AppBar(title: const Text('Context Scope')),
      body: Builder(
        // New context created by Builder
        builder: (BuildContext newContext) {
          // newContext here = Builder's context
          // This is BELOW Scaffold in the tree
          
          return Center(
            child: ElevatedButton(
              onPressed: () {
                // ✅ Works! Builder's context can find Scaffold above it
                ScaffoldMessenger.of(newContext).showSnackBar(
                  const SnackBar(content: Text('Hello!')),
                );
              },
              child: const Text('Show Snackbar'),
            ),
          );
        },
      ),
    );
  }
}
```

### Common Context Problem

```dart
// ❌ PROBLEM: Scaffold not in context
class WrongContextExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // context here DOESN'T have Scaffold above it
    
    return Scaffold(
      appBar: AppBar(title: const Text('Wrong Context')),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            // ERROR! Scaffold.of() called with context that doesn't have Scaffold
            Scaffold.of(context).showBottomSheet(
              (context) => Container(height: 200),
            );
          },
          child: const Text('Show Bottom Sheet'),
        ),
      ),
    );
  }
}

// ✅ SOLUTION 1: Use Builder
class CorrectWithBuilder extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Correct with Builder')),
      body: Builder(
        builder: (BuildContext scaffoldContext) {
          // scaffoldContext has Scaffold above it
          return Center(
            child: ElevatedButton(
              onPressed: () {
                Scaffold.of(scaffoldContext).showBottomSheet(
                  (context) => Container(height: 200, color: Colors.blue),
                );
              },
              child: const Text('Show Bottom Sheet'),
            ),
          );
        },
      ),
    );
  }
}

// ✅ SOLUTION 2: Extract to separate widget
class CorrectWithExtraction extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Correct with Extraction')),
      body: const ShowBottomSheetButton(),
    );
  }
}

class ShowBottomSheetButton extends StatelessWidget {
  const ShowBottomSheetButton({super.key});
  
  @override
  Widget build(BuildContext context) {
    // context here has Scaffold above it
    return Center(
      child: ElevatedButton(
        onPressed: () {
          Scaffold.of(context).showBottomSheet(
            (context) => Container(height: 200, color: Colors.green),
          );
        },
        child: const Text('Show Bottom Sheet'),
      ),
    );
  }
}

// ✅ SOLUTION 3: Use GlobalKey (not recommended)
class CorrectWithGlobalKey extends StatelessWidget {
  final GlobalKey<ScaffoldState> _scaffoldKey = GlobalKey<ScaffoldState>();
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      key: _scaffoldKey,
      appBar: AppBar(title: const Text('Correct with GlobalKey')),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            _scaffoldKey.currentState?.showBottomSheet(
              (context) => Container(height: 200, color: Colors.red),
            );
          },
          child: const Text('Show Bottom Sheet'),
        ),
      ),
    );
  }
}
```

### Context and InheritedWidget

```dart
// InheritedWidget example
class UserData extends InheritedWidget {
  final String username;
  final String email;
  
  const UserData({
    super.key,
    required this.username,
    required this.email,
    required super.child,
  });
  
  // Access method
  static UserData? of(BuildContext context) {
    return context.dependOnInheritedWidgetOfExactType<UserData>();
  }
  
  @override
  bool updateShouldNotify(UserData oldWidget) {
    return username != oldWidget.username || email != oldWidget.email;
  }
}

// Usage
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: UserData(
        username: 'john_doe',
        email: 'john@example.com',
        child: HomeScreen(),
      ),
    );
  }
}

class HomeScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // Access UserData using context
    final userData = UserData.of(context);
    
    return Scaffold(
      appBar: AppBar(title: const Text('Home')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Username: ${userData?.username}'),
            Text('Email: ${userData?.email}'),
            const ProfileWidget(),
          ],
        ),
      ),
    );
  }
}

class ProfileWidget extends StatelessWidget {
  const ProfileWidget({super.key});
  
  @override
  Widget build(BuildContext context) {
    // Can access UserData from any level deep
    final userData = UserData.of(context);
    
    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Text('Profile: ${userData?.username}'),
      ),
    );
  }
}
```

### Context vs State

```dart
class ContextVsState extends StatefulWidget {
  @override
  State<ContextVsState> createState() => _ContextVsStateState();
}

class _ContextVsStateState extends State<ContextVsState> {
  int _counter = 0;
  
  @override
  Widget build(BuildContext context) {
    // context: Position in tree, access inherited data
    // state: Widget's mutable data
    
    return Scaffold(
      appBar: AppBar(
        // Using context to get theme
        backgroundColor: Theme.of(context).primaryColor,
        title: const Text('Context vs State'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // Using state
            Text('Counter: $_counter'),
            
            // Using context
            Text(
              'Screen width: ${MediaQuery.of(context).size.width}',
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        // Both context and state available
        onPressed: () => setState(() => _counter++),
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

### Advanced: Multiple Contexts

```dart
class MultipleContextsDemo extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    print('Context 1: ${context.hashCode}');
    
    return Scaffold(
      appBar: AppBar(title: const Text('Multiple Contexts')),
      body: Builder(
        builder: (context2) {
          print('Context 2: ${context2.hashCode}');
          
          return Column(
            children: [
              Builder(
                builder: (context3) {
                  print('Context 3: ${context3.hashCode}');
                  
                  // All different contexts!
                  // context  = MultipleContextsDemo
                  // context2 = First Builder
                  // context3 = Second Builder
                  
                  return Text('Three different contexts');
                },
              ),
            ],
          );
        },
      ),
    );
  }
}
```

### Best Practices
- Use Builder when you need a new context below Scaffold
- Extract widgets instead of using Builder everywhere
- Don't store context in variables (it might become invalid)
- Use context only in build method or callbacks from build
- Understand the widget tree to know which context to use

### Common Mistakes
- ❌ Using wrong context (before widget is added to tree)
- ❌ Storing context in State class variables
- ❌ Using context after widget is disposed
- ❌ Not understanding context scope in the tree

---

*Continued with Q5-Q10 in next message...*