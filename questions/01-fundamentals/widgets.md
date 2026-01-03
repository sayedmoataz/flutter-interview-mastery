# Widgets & UI - Interview Questions

## Table of Contents
1. [StatelessWidget vs StatefulWidget](#q1-statelesswidget-vs-statefulwidget)
2. [Widget Lifecycle](#q2-widget-lifecycle)
3. [BuildContext Explained](#q3-buildcontext-explained)
4. [Keys in Flutter](#q4-keys-in-flutter)
5. [const Constructor](#q5-const-constructor)
6. [Widget vs Element vs RenderObject](#q6-widget-vs-element-vs-renderobject)
7. [InheritedWidget](#q7-inheritedwidget)
8. [Widget Best Practices](#q8-widget-best-practices)
9. [Common Widget Mistakes](#q9-common-widget-mistakes)
10. [Custom Widgets](#q10-custom-widgets)

---

## Q1: StatelessWidget vs StatefulWidget

**Level:** Junior  
**Category:** Widgets  
**Difficulty:** ⭐⭐

### The Question
What's the difference between StatelessWidget and StatefulWidget? When should you use each?

### Answer

**StatelessWidget:**
- Immutable (doesn't change)
- No internal state
- Rebuilds only when parent rebuilds
- Simpler and more performant

**StatefulWidget:**
- Mutable (can change)
- Has internal state
- Can rebuild itself using setState()
- Used for interactive/dynamic UI

### Code Example

```dart
import 'package:flutter/material.dart';

// ✅ StatelessWidget - Display static data
class UserProfile extends StatelessWidget {
  final String name;
  final String email;

  const UserProfile({
    super.key,
    required this.name,
    required this.email,
  });

  @override
  Widget build(BuildContext context) {
    return Card(
      child: Column(
        children: [
          Text('Name: $name'),
          Text('Email: $email'),
        ],
      ),
    );
  }
}

// ✅ StatefulWidget - Interactive counter
class Counter extends StatefulWidget {
  const Counter({super.key});

  @override
  State<Counter> createState() => _CounterState();
}

class _CounterState extends State<Counter> {
  int _count = 0; // Mutable state

  void _increment() {
    setState(() {
      _count++; // Update state and rebuild
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

### When to Use Each

```dart
// ✅ Use StatelessWidget for:
// - Static text/images
// - Layout widgets (Container, Row, Column)
// - Widgets that only depend on constructor parameters

class WelcomeScreen extends StatelessWidget {
  const WelcomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Welcome')),
      body: const Center(
        child: Text('Hello, Flutter!'),
      ),
    );
  }
}

// ✅ Use StatefulWidget for:
// - Forms with user input
// - Animations
// - Data that changes over time
// - Interactive widgets (checkboxes, switches)

class LoginForm extends StatefulWidget {
  const LoginForm({super.key});

  @override
  State<LoginForm> createState() => _LoginFormState();
}

class _LoginFormState extends State<LoginForm> {
  final _formKey = GlobalKey<FormState>();
  String _email = '';
  String _password = '';
  bool _isLoading = false;

  Future<void> _submit() async {
    if (_formKey.currentState!.validate()) {
      setState(() => _isLoading = true);
      
      // Simulate API call
      await Future.delayed(const Duration(seconds: 2));
      
      setState(() => _isLoading = false);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Form(
      key: _formKey,
      child: Column(
        children: [
          TextFormField(
            decoration: const InputDecoration(labelText: 'Email'),
            onChanged: (value) => _email = value,
            validator: (value) {
              if (value == null || value.isEmpty) {
                return 'Please enter email';
              }
              return null;
            },
          ),
          TextFormField(
            decoration: const InputDecoration(labelText: 'Password'),
            obscureText: true,
            onChanged: (value) => _password = value,
            validator: (value) {
              if (value == null || value.length < 6) {
                return 'Password must be at least 6 characters';
              }
              return null;
            },
          ),
          const SizedBox(height: 20),
          _isLoading
              ? const CircularProgressIndicator()
              : ElevatedButton(
                  onPressed: _submit,
                  child: const Text('Login'),
                ),
        ],
      ),
    );
  }
}
```

### Performance Comparison

```dart
// ❌ BAD: Using StatefulWidget when StatelessWidget is enough
class BadWelcomeMessage extends StatefulWidget {
  final String name;
  const BadWelcomeMessage({super.key, required this.name});

  @override
  State<BadWelcomeMessage> createState() => _BadWelcomeMessageState();
}

class _BadWelcomeMessageState extends State<BadWelcomeMessage> {
  @override
  Widget build(BuildContext context) {
    return Text('Welcome, ${widget.name}!');
  }
}

// ✅ GOOD: Use StatelessWidget for static content
class GoodWelcomeMessage extends StatelessWidget {
  final String name;
  const GoodWelcomeMessage({super.key, required this.name});

  @override
  Widget build(BuildContext context) {
    return Text('Welcome, $name!');
  }
}
```

### Key Takeaways
- Default to StatelessWidget when possible (better performance)
- Use StatefulWidget only when you need to manage changing state
- StatefulWidget is split into widget (immutable) and state (mutable)
- Always use `const` constructors for StatelessWidget when possible

---

## Q2: Widget Lifecycle

**Level:** Junior  
**Category:** Widgets  
**Difficulty:** ⭐⭐⭐

### The Question
Explain the lifecycle of a StatefulWidget. What methods are called and in what order?

### Answer

### StatefulWidget Lifecycle Methods (in order):

1. **constructor** - Widget is created
2. **createState()** - Creates the State object
3. **initState()** - Called once when State is inserted into tree
4. **didChangeDependencies()** - Called after initState and when dependencies change
5. **build()** - Builds the widget tree
6. **didUpdateWidget()** - Called when parent rebuilds with new widget
7. **setState()** - Triggers rebuild
8. **deactivate()** - Called when widget is removed from tree (temporarily)
9. **dispose()** - Called when State is removed permanently

### Code Example

```dart
import 'package:flutter/material.dart';
import 'dart:async';

class LifecycleDemo extends StatefulWidget {
  final String title;

  const LifecycleDemo({super.key, required this.title});

  @override
  State<LifecycleDemo> createState() {
    print('1. createState() called');
    return _LifecycleDemoState();
  }
}

class _LifecycleDemoState extends State<LifecycleDemo> {
  int _counter = 0;
  late Timer _timer;

  // 2. Constructor (optional, not commonly used)
  _LifecycleDemoState() {
    print('2. State Constructor called');
  }

  // 3. Called once when widget is inserted into tree
  @override
  void initState() {
    super.initState();
    print('3. initState() called');
    
    // Good place for:
    // - Initialize state variables
    // - Start animations
    // - Subscribe to streams
    // - Setup controllers
    
    _timer = Timer.periodic(const Duration(seconds: 1), (timer) {
      if (mounted) {
        setState(() {
          _counter++;
        });
      }
    });
  }

  // 4. Called after initState and when InheritedWidget changes
  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    print('4. didChangeDependencies() called');
    
    // Good place for:
    // - Getting data from InheritedWidget
    // - Theme changes
    // - MediaQuery changes
  }

  // 5. Builds the widget tree (called many times)
  @override
  Widget build(BuildContext context) {
    print('5. build() called');
    
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title), // Access widget properties via widget.property
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Text('Counter:'),
            Text(
              '$_counter',
              style: Theme.of(context).textTheme.headlineMedium,
            ),
            ElevatedButton(
              onPressed: () {
                setState(() {
                  _counter++;
                });
              },
              child: const Text('Increment'),
            ),
          ],
        ),
      ),
    );
  }

  // 6. Called when parent widget changes and rebuilds this widget
  @override
  void didUpdateWidget(LifecycleDemo oldWidget) {
    super.didUpdateWidget(oldWidget);
    print('6. didUpdateWidget() called');
    print('   Old title: ${oldWidget.title}');
    print('   New title: ${widget.title}');
    
    // Compare old and new widget
    if (oldWidget.title != widget.title) {
      // Do something when title changes
      print('   Title changed!');
    }
  }

  // 7. Called when widget is removed from tree (but might be reinserted)
  @override
  void deactivate() {
    print('7. deactivate() called');
    super.deactivate();
  }

  // 8. Called when State is removed permanently
  @override
  void dispose() {
    print('8. dispose() called');
    
    // Clean up:
    // - Cancel timers
    // - Close streams
    // - Dispose controllers
    // - Unsubscribe from listeners
    
    _timer.cancel();
    super.dispose();
  }
}

// Example usage showing lifecycle
void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: const LifecycleParent(),
    );
  }
}

class LifecycleParent extends StatefulWidget {
  const LifecycleParent({super.key});

  @override
  State<LifecycleParent> createState() => _LifecycleParentState();
}

class _LifecycleParentState extends State<LifecycleParent> {
  String _title = 'Initial Title';
  bool _showWidget = true;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          ElevatedButton(
            onPressed: () {
              setState(() {
                _title = 'Updated Title ${DateTime.now().second}';
              });
            },
            child: const Text('Change Title (triggers didUpdateWidget)'),
          ),
          ElevatedButton(
            onPressed: () {
              setState(() {
                _showWidget = !_showWidget;
              });
            },
            child: Text(_showWidget ? 'Hide Widget' : 'Show Widget'),
          ),
          if (_showWidget)
            Expanded(
              child: LifecycleDemo(title: _title),
            ),
        ],
      ),
    );
  }
}
```

### Lifecycle Flow Diagram

```
Widget Created
      ↓
createState()
      ↓
initState() ← Called once
      ↓
didChangeDependencies()
      ↓
build() ← Can be called many times
      ↓
┌─────────────────┐
│  Widget Active  │
└─────────────────┘
      ↓
  setState() → build() (rebuild)
      ↓
  Parent rebuilds → didUpdateWidget() → build()
      ↓
  Dependencies change → didChangeDependencies() → build()
      ↓
deactivate() (removed from tree)
      ↓
dispose() ← Called once (cleanup)
```

### Common Use Cases

```dart
class CommonLifecyclePatterns extends StatefulWidget {
  const CommonLifecyclePatterns({super.key});

  @override
  State<CommonLifecyclePatterns> createState() => _CommonLifecyclePatternsState();
}

class _CommonLifecyclePatternsState extends State<CommonLifecyclePatterns> {
  late TextEditingController _controller;
  late StreamSubscription<int> _subscription;
  late AnimationController _animationController;

  @override
  void initState() {
    super.initState();
    
    // ✅ Initialize controllers
    _controller = TextEditingController();
    
    // ✅ Start listening to streams
    _subscription = Stream.periodic(const Duration(seconds: 1), (i) => i)
        .listen((value) {
      print('Stream value: $value');
    });
    
    // ✅ Setup animation controller
    _animationController = AnimationController(
      vsync: this as TickerProvider,
      duration: const Duration(seconds: 2),
    );
    
    // ✅ Fetch initial data
    _loadData();
  }

  Future<void> _loadData() async {
    // Fetch data from API
    await Future.delayed(const Duration(seconds: 1));
    if (mounted) {
      setState(() {
        // Update UI
      });
    }
  }

  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    
    // ✅ Access InheritedWidget or BuildContext dependent data
    final theme = Theme.of(context);
    final mediaQuery = MediaQuery.of(context);
  }

  @override
  void didUpdateWidget(CommonLifecyclePatterns oldWidget) {
    super.didUpdateWidget(oldWidget);
    
    // ✅ React to widget property changes
    // Compare oldWidget with widget
  }

  @override
  void dispose() {
    // ✅ Clean up everything
    _controller.dispose();
    _subscription.cancel();
    _animationController.dispose();
    
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Container();
  }
}
```

### Best Practices

✅ **DO:**
- Call `super.initState()` first in initState()
- Call `super.dispose()` last in dispose()
- Always dispose controllers, subscriptions, and timers
- Check `mounted` before calling setState() in async operations
- Use initState() for one-time setup

❌ **DON'T:**
- Don't call setState() in initState()
- Don't use BuildContext in initState() (use didChangeDependencies instead)
- Don't forget to dispose resources
- Don't call setState() after dispose()
- Don't perform expensive operations in build()

---

## Q3: BuildContext Explained

**Level:** Junior  
**Category:** Widgets  
**Difficulty:** ⭐⭐⭐

### The Question
What is BuildContext? Why do we need it? How does it work?

### Answer

**BuildContext** is a handle to the location of a widget in the widget tree. It's used to:
- Navigate to other screens
- Show dialogs/snackbars
- Access InheritedWidgets (Theme, MediaQuery, Provider)
- Get widget size/position

### Code Example

```dart
import 'package:flutter/material.dart';

class BuildContextDemo extends StatelessWidget {
  const BuildContextDemo({super.key});

  @override
  Widget build(BuildContext context) {
    // context represents THIS widget's location in the tree
    
    return Scaffold(
      appBar: AppBar(
        title: const Text('BuildContext Demo'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // 1. Access Theme
            Text(
              'Themed Text',
              style: Theme.of(context).textTheme.headlineMedium,
            ),
            
            // 2. Access MediaQuery
            Text(
              'Screen width: ${MediaQuery.of(context).size.width}',
            ),
            
            // 3. Navigation
            ElevatedButton(
              onPressed: () {
                Navigator.of(context).push(
                  MaterialPageRoute(
                    builder: (context) => const SecondScreen(),
                  ),
                );
              },
              child: const Text('Navigate'),
            ),
            
            // 4. Show SnackBar
            ElevatedButton(
              onPressed: () {
                ScaffoldMessenger.of(context).showSnackBar(
                  const SnackBar(content: Text('Hello from SnackBar!')),
                );
              },
              child: const Text('Show SnackBar'),
            ),
            
            // 5. Show Dialog
            ElevatedButton(
              onPressed: () {
                showDialog(
                  context: context,
                  builder: (context) => AlertDialog(
                    title: const Text('Alert'),
                    content: const Text('This is a dialog'),
                    actions: [
                      TextButton(
                        onPressed: () => Navigator.pop(context),
                        child: const Text('OK'),
                      ),
                    ],
                  ),
                );
              },
              child: const Text('Show Dialog'),
            ),
          ],
        ),
      ),
    );
  }
}

class SecondScreen extends StatelessWidget {
  const SecondScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Second Screen')),
      body: Center(
        child: ElevatedButton(
          onPressed: () => Navigator.pop(context),
          child: const Text('Go Back'),
        ),
      ),
    );
  }
}
```

### Common BuildContext Mistakes

```dart
// ❌ WRONG: Using wrong context
class WrongContextExample extends StatelessWidget {
  const WrongContextExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Wrong Context')),
      body: ElevatedButton(
        onPressed: () {
          // ERROR: Scaffold.of() called with context that doesn't contain Scaffold
          Scaffold.of(context).showSnackBar(
            const SnackBar(content: Text('Error!')),
          );
        },
        child: const Text('Show SnackBar'),
      ),
    );
  }
}

// ✅ CORRECT: Using Builder to get correct context
class CorrectContextExample extends StatelessWidget {
  const CorrectContextExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Correct Context')),
      body: Builder(
        builder: (BuildContext scaffoldContext) {
          // scaffoldContext is below Scaffold in the tree
          return ElevatedButton(
            onPressed: () {
              ScaffoldMessenger.of(scaffoldContext).showSnackBar(
                const SnackBar(content: Text('Success!')),
              );
            },
            child: const Text('Show SnackBar'),
          );
        },
      ),
    );
  }
}

// ✅ ALTERNATIVE: Extract to separate widget
class AlternativeContextExample extends StatelessWidget {
  const AlternativeContextExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Alternative')),
      body: const SnackBarButton(), // Separate widget
    );
  }
}

class SnackBarButton extends StatelessWidget {
  const SnackBarButton({super.key});

  @override
  Widget build(BuildContext context) {
    // This context is below Scaffold
    return ElevatedButton(
      onPressed: () {
        ScaffoldMessenger.of(context).showSnackBar(
          const SnackBar(content: Text('Works!')),
        );
      },
      child: const Text('Show SnackBar'),
    );
  }
}
```

### Understanding Context Hierarchy

```dart
class ContextHierarchyDemo extends StatelessWidget {
  const ContextHierarchyDemo({super.key});

  @override
  Widget build(BuildContext context) {
    // context here is ABOVE MaterialApp
    
    return MaterialApp(
      home: Builder(
        builder: (context) {
          // context here is BELOW MaterialApp
          // Can access Theme, Navigator, etc.
          
          return Scaffold(
            appBar: AppBar(
              title: const Text('Context Hierarchy'),
            ),
            body: Builder(
              builder: (context) {
                // context here is BELOW Scaffold
                // Can access Scaffold
                
                return Center(
                  child: Column(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      // Can access Theme (from MaterialApp)
                      Text(
                        'Themed Text',
                        style: Theme.of(context).textTheme.headlineMedium,
                      ),
                      
                      // Can access Scaffold
                      ElevatedButton(
                        onPressed: () {
                          ScaffoldMessenger.of(context).showSnackBar(
                            const SnackBar(content: Text('Hello!')),
                          );
                        },
                        child: const Text('Show SnackBar'),
                      ),
                    ],
                  ),
                );
              },
            ),
          );
        },
      ),
    );
  }
}
```

### Best Practices

✅ **DO:**
- Use Builder widget when you need context below current widget
- Extract widgets to separate classes for cleaner context
- Use `of(context)` methods to access InheritedWidgets
- Understand the widget tree hierarchy

❌ **DON'T:**
- Don't use context before MaterialApp is created
- Don't store context in variables (it can become invalid)
- Don't use context after widget is disposed

---

## Q4: Keys in Flutter

**Level:** Mid  
**Category:** Widgets  
**Difficulty:** ⭐⭐⭐⭐

### The Question
What are Keys in Flutter? When and why should you use them?

### Answer

**Keys** help Flutter identify widgets uniquely. They're used to:
- Preserve state when widgets move in a list
- Force widgets to rebuild
- Access widgets from outside

**Types of Keys:**
- **ValueKey** - Based on a value
- **ObjectKey** - Based on an object
- **UniqueKey** - Always unique
- **GlobalKey** - Access widget from anywhere
- **PageStorageKey** - Preserve scroll position

### Code Example

```dart
import 'package:flutter/material.dart';

// Example 1: Without Keys (State Lost)
class WithoutKeysDemo extends StatefulWidget {
  const WithoutKeysDemo({super.key});

  @override
  State<WithoutKeysDemo> createState() => _WithoutKeysDemoState();
}

class _WithoutKeysDemoState extends State<WithoutKeysDemo> {
  List<String> items = ['Item 1', 'Item 2', 'Item 3'];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Without Keys')),
      body: Column(
        children: [
          ...items.map((item) => StatefulTile(title: item)),
          ElevatedButton(
            onPressed: () {
              setState(() {
                items.removeAt(0); // Remove first item
              });
            },
            child: const Text('Remove First Item'),
          ),
        ],
      ),
    );
  }
}

class StatefulTile extends StatefulWidget {
  final String title;

  const StatefulTile({super.key, required this.title});

  @override
  State<StatefulTile> createState() => _StatefulTileState();
}

class _StatefulTileState extends State<StatefulTile> {
  Color? _color;

  @override
  void initState() {
    super.initState();
    _color = Colors.primaries[widget.title.hashCode % Colors.primaries.length];
  }

  @override
  Widget build(BuildContext context) {
    return Container(
      color: _color,
      padding: const EdgeInsets.all(16),
      margin: const EdgeInsets.all(8),
      child: Text(widget.title),
    );
  }
}

// Example 2: With Keys (State Preserved)
class WithKeysDemo extends StatefulWidget {
  const WithKeysDemo({super.key});

  @override
  State<WithKeysDemo> createState() => _WithKeysDemoState();
}

class _WithKeysDemoState extends State<WithKeysDemo> {
  List<String> items = ['Item 1', 'Item 2', 'Item 3'];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('With Keys')),
      body: Column(
        children: [
          ...items.map(
            (item) => StatefulTile(
              key: ValueKey(item), // ✅ Key preserves state
              title: item,
            ),
          ),
          ElevatedButton(
            onPressed: () {
              setState(() {
                items.removeAt(0);
              });
            },
            child: const Text('Remove First Item'),
          ),
        ],
      ),
    );
  }
}
```

### Types of Keys Examples

```dart
class KeyTypesDemo extends StatefulWidget {
  const KeyTypesDemo({super.key});

  @override
  State<KeyTypesDemo> createState() => _KeyTypesDemoState();
}

class _KeyTypesDemoState extends State<KeyTypesDemo> {
  final List<TodoItem> todos = [
    TodoItem(id: 1, title: 'Buy milk'),
    TodoItem(id: 2, title: 'Walk dog'),
    TodoItem(id: 3, title: 'Write code'),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Key Types')),
      body: ListView(
        children: [
          // 1. ValueKey - Based on value
          ...todos.map(
            (todo) => CheckboxListTile(
              key: ValueKey(todo.id), // ✅ Uses id as key
              title: Text(todo.title),
              value: todo.done,
              onChanged: (value) {
                setState(() {
                  todo.done = value ?? false;
                });
              },
            ),
          ),
          
          // 2. ObjectKey - Based on object
          TodoTile(
            key: ObjectKey(todos[0]), // ✅ Uses entire object
            todo: todos[0],
          ),
          
          // 3. UniqueKey - Always unique
          Container(
            key: UniqueKey(), // ✅ Forces rebuild every time
            child: const Text('Always rebuilds'),
          ),
          
          ElevatedButton(
            onPressed: () {
              setState(() {
                todos.shuffle(); // Reorder items
              });
            },
            child: const Text('Shuffle'),
          ),
        ],
      ),
    );
  }
}

class TodoItem {
  final int id;
  final String title;
  bool done;

  TodoItem({required this.id, required this.title, this.done = false});
}

class TodoTile extends StatelessWidget {
  final TodoItem todo;

  const TodoTile({super.key, required this.todo});

  @override
  Widget build(BuildContext context) {
    return ListTile(
      title: Text(todo.title),
      trailing: Checkbox(
        value: todo.done,
        onChanged: (value) {
          todo.done = value ?? false;
        },
      ),
    );
  }
}
```

### GlobalKey Example

```dart
class GlobalKeyDemo extends StatefulWidget {
  const GlobalKeyDemo({super.key});

  @override
  State<GlobalKeyDemo> createState() => _GlobalKeyDemoState();
}

class _GlobalKeyDemoState extends State<GlobalKeyDemo> {
  // GlobalKey to access FormState
  final _formKey = GlobalKey<FormState>();
  
  // GlobalKey to access custom widget state
  final _counterKey = GlobalKey<_CounterWidgetState>();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('GlobalKey Demo')),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          children: [
            // Form with GlobalKey
            Form(
              key: _formKey,
              child: Column(
                children: [
                  TextFormField(
                    decoration: const InputDecoration(labelText: 'Email'),
                    validator: (value) {
                      if (value == null || !value.contains('@')) {
                        return 'Invalid email';
                      }
                      return null;
                    },
                  ),
                  ElevatedButton(
                    onPressed: () {
                      // Access form state via GlobalKey
                      if (_formKey.currentState!.validate()) {
                        ScaffoldMessenger.of(context).showSnackBar(
                          const SnackBar(content: Text('Valid!')),
                        );
                      }
                    },
                    child: const Text('Validate'),
                  ),
                ],
              ),
            ),
            const SizedBox(height: 40),
            
            // Custom widget with GlobalKey
            CounterWidget(key: _counterKey),
            
            ElevatedButton(
              onPressed: () {
                // Access widget state from outside
                _counterKey.currentState?.increment();
              },
              child: const Text('Increment from Parent'),
            ),
            
            ElevatedButton(
              onPressed: () {
                // Read state value
                final count = _counterKey.currentState?.count ?? 0;
                ScaffoldMessenger.of(context).showSnackBar(
                  SnackBar(content: Text('Count is $count')),
                );
              },
              child: const Text('Read Count'),
            ),
          ],
        ),
      ),
    );
  }
}

class CounterWidget extends StatefulWidget {
  const CounterWidget({super.key});

  @override
  State<CounterWidget> createState() => _CounterWidgetState();
}

class _CounterWidgetState extends State<CounterWidget> {
  int count = 0;

  void increment() {
    setState(() {
      count++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.all(16),
      color: Colors.blue.shade100,
      child: Text(
        'Count: $count',
        style: const TextStyle(fontSize: 24),
      ),
    );
  }
}
```

### PageStorageKey Example

```dart
class PageStorageKeyDemo extends StatefulWidget {
  const PageStorageKeyDemo({super.key});

  @override
  State<PageStorageKeyDemo> createState() => _PageStorageKeyDemoState();
}

class _PageStorageKeyDemoState extends State<PageStorageKeyDemo> {
  int _currentIndex = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('PageStorageKey Demo')),
      body: IndexedStack(
        index: _currentIndex,
        children: [
          // Scroll position is preserved when switching tabs
          ListView.builder(
            key: const PageStorageKey('list1'), // ✅ Preserves scroll
            itemCount: 50,
            itemBuilder: (context, index) => ListTile(
              title: Text('List 1 - Item $index'),
            ),
          ),
          ListView.builder(
            key: const PageStorageKey('list2'), // ✅ Preserves scroll
            itemCount: 50,
            itemBuilder: (context, index) => ListTile(
              title: Text('List 2 - Item $index'),
            ),
          ),
        ],
      ),
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: _currentIndex,
        onTap: (index) => setState(() => _currentIndex = index),
        items: const [
          BottomNavigationBarItem(icon: Icon(Icons.home), label: 'List 1'),
          BottomNavigationBarItem(icon: Icon(Icons.business), label: 'List 2'),
        ],
      ),
    );
  }
}
```

### When to Use Keys

```dart
// ✅ USE KEYS when:

// 1. Reordering lists
ReorderableListView(
  children: items.map((item) => ListTile(
    key: ValueKey(item.id),
    title: Text(item.name),
  )).toList(),
);

// 2. Removing/Adding items in lists
Column(
  children: items.map((item) => Widget(
    key: ValueKey(item.id),
  )).toList(),
);

// 3. Preserving scroll position
ListView(
  key: const PageStorageKey('myList'),
  children: [...],
);

// 4. Accessing widget state from outside
final formKey = GlobalKey<FormState>();
Form(key: formKey, child: ...);

// ❌ DON'T USE KEYS when:
// - Static lists that never change
// - Simple StatelessWidgets without state
// - Performance-critical scrolling (overhead)
```

### Best Practices

✅ **DO:**
- Use ValueKey for simple values (id, string)
- Use ObjectKey for complex objects
- Use GlobalKey sparingly (performance overhead)
- Use PageStorageKey for preserving scroll position

❌ **DON'T:**
- Don't use keys everywhere (adds overhead)
- Don't use random keys (defeats the purpose)
- Don't use index as key in dynamic lists
- Don't create new GlobalKey in build method

---

## Q5: const Constructor

**Level:** Junior  
**Category:** Performance  
**Difficulty:** ⭐⭐⭐

### The Question
What is `const` constructor? Why is it important for performance?

### Answer

**const** creates compile-time constants. Benefits:
- Widget is created once and reused
- Prevents unnecessary rebuilds
- Reduces memory usage
- Improves performance

### Code Example

```dart
import 'package:flutter/material.dart';

// Example 1: Without const (rebuilds every time)
class WithoutConstDemo extends StatefulWidget {
  const WithoutConstDemo({super.key});

  @override
  State<WithoutConstDemo> createState() => _WithoutConstDemoState();
}

class _WithoutConstDemoState extends State<WithoutConstDemo> {
  int _counter = 0;

  @override
  Widget build(BuildContext context) {
    print('Parent build called');
    
    return Scaffold(
      appBar: AppBar(title: const Text('Without const')),
      body: Column(
        children: [
          Text('Counter: $_counter'),
          
          // ❌ Without const - rebuilds every time
          ExpensiveWidget(),
          
          ElevatedButton(
            onPressed: () => setState(() => _counter++),
            child: const Text('Increment'),
          ),
        ],
      ),
    );
  }
}

class ExpensiveWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    print('ExpensiveWidget rebuilt'); // Called every parent rebuild!
    
    return Container(
      height: 200,
      color: Colors.blue,
      child: const Center(
        child: Text('Expensive to rebuild'),
      ),
    );
  }
}

// Example 2: With const (never rebuilds)
class WithConstDemo extends StatefulWidget {
  const WithConstDemo({super.key});

  @override
  State<WithConstDemo> createState() => _WithConstDemoState();
}

class _WithConstDemoState extends State<WithConstDemo> {
  int _counter = 0;

  @override
  Widget build(BuildContext context) {
    print('Parent build called');
    
    return Scaffold(
      appBar: AppBar(title: const Text('With const')),
      body: Column(
        children: [
          Text('Counter: $_counter'),
          
          // ✅ With const - never rebuilds
          const ExpensiveWidget(),
          
          ElevatedButton(
            onPressed: () => setState(() => _counter++),
            child: const Text('Increment'),
          ),
        ],
      ),
    );
  }
}
```

### const vs final

```dart
class ConstVsFinalDemo {
  // const - Compile-time constant (value known at compile time)
  static const int maxUsers = 100;
  static const String appName = 'MyApp';
  static const List<int> numbers = [1, 2, 3];
  
  // final - Runtime constant (value known at runtime)
  final DateTime currentTime = DateTime.now();
  final String uuid = generateUuid();
  
  String generateUuid() => 'uuid-123';
}

// const in widgets
class ConstWidgetExample extends StatelessWidget {
  const ConstWidgetExample({super.key});

  @override
  Widget build(BuildContext context) {
    return const Column(
      children: [
        // ✅ const - compiled once
        Text('Static text'),
        Icon(Icons.home),
        SizedBox(height: 20),
      ],
    );
  }
}

// When you can't use const
class NonConstExample extends StatelessWidget {
  const NonConstExample({super.key});

  @override
  Widget build(BuildContext context) {
    final dynamicValue = DateTime.now().toString();
    
    return Column(
      children: [
        // ❌ Can't use const - value is dynamic
        Text(dynamicValue),
        
        // ✅ Can use const - static value
        const Text('Static'),
        
        // ❌ Can't use const - uses BuildContext
        Text(
          'Themed',
          style: Theme.of(context).textTheme.bodyMedium,
        ),
      ],
    );
  }
}
```

### Performance Comparison

```dart
class PerformanceDemo extends StatefulWidget {
  const PerformanceDemo({super.key});

  @override
  State<PerformanceDemo> createState() => _PerformanceDemoState();
}

class _PerformanceDemoState extends State<PerformanceDemo> {
  int _buildCount = 0;

  @override
  Widget build(BuildContext context) {
    _buildCount++;
    print('Build count: $_buildCount');
    
    return Scaffold(
      appBar: AppBar(title: const Text('Performance Demo')),
      body: Column(
        children: [
          Text('Builds: $_buildCount'),
          
          // ❌ BAD: Creates new widget every build (1000 builds = 1000 widgets)
          Container(
            height: 100,
            color: Colors.red,
            child: const Center(child: Text('Red')),
          ),
          
          // ✅ GOOD: Reuses same widget (1000 builds = 1 widget)
          const RedBox(),
          
          // ❌ BAD: New list every build
          Column(
            children: [
              Container(height: 50, color: Colors.blue),
              Container(height: 50, color: Colors.green),
            ],
          ),
          
          // ✅ GOOD: const list
          const Column(
            children: [
              SizedBox(height: 50, child: ColoredBox(color: Colors.blue)),
              SizedBox(height: 50, child: ColoredBox(color: Colors.green)),
            ],
          ),
          
          ElevatedButton(
            onPressed: () => setState(() {}),
            child: const Text('Rebuild'),
          ),
        ],
      ),
    );
  }
}

class RedBox extends StatelessWidget {
  const RedBox({super.key});

  @override
  Widget build(BuildContext context) {
    print('RedBox built'); // Only printed once with const!
    
    return Container(
      height: 100,
      color: Colors.red,
      child: const Center(child: Text('Red')),
    );
  }
}
```

### When to Use const

```dart
class ConstGuide extends StatelessWidget {
  final String name;
  
  const ConstGuide({super.key, required this.name});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      // ✅ Static AppBar
      appBar: AppBar(
        title: const Text('My App'),
      ),
      
      body: Column(
        children: [
          // ❌ Can't use const - uses constructor parameter
          Text('Hello, $name'),
          
          // ✅ Static padding
          const SizedBox(height: 20),
          
          // ✅ Static icon
          const Icon(Icons.home, size: 50),
          
          // ❌ Can't use const - uses Theme
          Text(
            'Themed',
            style: Theme.of(context).textTheme.headlineMedium,
          ),
          
          // ✅ Static container with const children
          const Padding(
            padding: EdgeInsets.all(16.0),
            child: Text('Static text'),
          ),
          
          // ✅ Extract const widgets
          const _StaticSection(),
        ],
      ),
    );
  }
}

class _StaticSection extends StatelessWidget {
  const _StaticSection();

  @override
  Widget build(BuildContext context) {
    return const Column(
      children: [
        Text('Line 1'),
        Text('Line 2'),
        Text('Line 3'),
      ],
    );
  }
}
```

### Best Practices

✅ **DO:**
- Use const for widgets that never change
- Use const constructors in custom widgets
- Extract static widgets to const widgets
- Use const for padding, icons, static text

❌ **DON'T:**
- Don't forget const when possible
- Don't use const with dynamic values
- Don't use const with BuildContext dependent widgets

### Performance Tips

```dart
// Tip 1: Use const for child parameter
AnimatedContainer(
  duration: const Duration(seconds: 1),
  child: const Icon(Icons.star), // Child doesn't rebuild
);

// Tip 2: Const lists
const List<Widget> items = [
  Text('Item 1'),
  Text('Item 2'),
];

// Tip 3: Const in loops (if possible)
Column(
  children: List.generate(
    5,
    (index) => const Padding(
      padding: EdgeInsets.all(8.0),
      child: Icon(Icons.star),
    ),
  ),
);
```

---

*Continued in next sections: Q6-Q10 covering Widget vs Element, InheritedWidget, Best Practices, Common Mistakes, and Custom Widgets...*