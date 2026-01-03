# Widgets & UI - Part 2 (Q6-Q10)

*This is a continuation of [widgets.md](widgets.md)*

---

## Q6: Widget vs Element vs RenderObject

**Level:** Mid  
**Category:** Architecture  
**Difficulty:** ⭐⭐⭐⭐

### The Question
Explain the difference between Widget, Element, and RenderObject. How do they work together?

### Answer

Flutter has 3 trees:

1. **Widget Tree** (Configuration) - Immutable, describes how UI should look
2. **Element Tree** (Lifecycle) - Mutable, manages widget lifecycle and state
3. **RenderObject Tree** (Rendering) - Handles layout, painting, hit testing

```
Widget (Blueprint)
   ↓
Element (Manager)
   ↓
RenderObject (Painter)
```

### Code Example

```dart
import 'package:flutter/material.dart';
import 'package:flutter/rendering.dart';

// WIDGET - Configuration (Immutable)
class MyButton extends StatelessWidget {
  final String text;
  final VoidCallback onPressed;

  // Widget is just configuration
  // Created and destroyed frequently
  const MyButton({
    super.key,
    required this.text,
    required this.onPressed,
  });

  @override
  Widget build(BuildContext context) {
    print('Widget build called');
    return ElevatedButton(
      onPressed: onPressed,
      child: Text(text),
    );
  }

  // Widget creates Element
  @override
  StatelessElement createElement() {
    print('Element created');
    return StatelessElement(this);
  }
}

// ELEMENT - Lifecycle Manager
// Element is created once and reused
// Holds reference to both Widget and RenderObject
class CustomElement extends ComponentElement {
  CustomElement(Widget widget) : super(widget);

  @override
  void mount(Element? parent, dynamic newSlot) {
    print('Element mounted');
    super.mount(parent, newSlot);
  }

  @override
  void update(Widget newWidget) {
    print('Element updated with new widget');
    super.update(newWidget);
  }

  @override
  void unmount() {
    print('Element unmounted');
    super.unmount();
  }

  @override
  Widget build() {
    return Container();
  }
}

// RENDEROBJECT - Handles rendering
class CustomRenderBox extends RenderBox {
  Color _color;

  CustomRenderBox(this._color);

  Color get color => _color;
  set color(Color value) {
    if (_color == value) return;
    _color = value;
    markNeedsPaint(); // Request repaint
  }

  @override
  void performLayout() {
    // Calculate size
    size = constraints.constrain(const Size(200, 200));
  }

  @override
  void paint(PaintingContext context, Offset offset) {
    // Draw on canvas
    final paint = Paint()..color = _color;
    context.canvas.drawRect(
      offset & size,
      paint,
    );
  }
}

// Widget that uses custom RenderObject
class ColoredBox extends LeafRenderObjectWidget {
  final Color color;

  const ColoredBox({super.key, required this.color});

  @override
  RenderObject createRenderObject(BuildContext context) {
    return CustomRenderBox(color);
  }

  @override
  void updateRenderObject(BuildContext context, CustomRenderBox renderObject) {
    renderObject.color = color;
  }
}
```

### How They Work Together

```dart
class TreesDemo extends StatefulWidget {
  const TreesDemo({super.key});

  @override
  State<TreesDemo> createState() => _TreesDemoState();
}

class _TreesDemoState extends State<TreesDemo> {
  Color _color = Colors.blue;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('3 Trees Demo')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // When setState is called:
            // 1. Widget tree rebuilds (new widget created)
            // 2. Element tree compares old and new widget
            // 3. If widget type is same, Element is reused
            // 4. Element updates RenderObject if needed
            ColoredBox(color: _color),
            
            const SizedBox(height: 20),
            
            ElevatedButton(
              onPressed: () {
                setState(() {
                  _color = _color == Colors.blue ? Colors.red : Colors.blue;
                });
              },
              child: const Text('Change Color'),
            ),
          ],
        ),
      ),
    );
  }
}
```

### Widget Lifecycle with Trees

```dart
class LifecycleWithTrees extends StatefulWidget {
  const LifecycleWithTrees({super.key});

  @override
  State<LifecycleWithTrees> createState() => _LifecycleWithTreesState();
}

class _LifecycleWithTreesState extends State<LifecycleWithTrees> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // Step 1: Widget created
            const Text('Hello'),
            // Step 2: Element checks if it can reuse existing Element
            // Step 3: Element updates RenderObject if needed
            // Step 4: RenderObject paints on screen
          ],
        ),
      ),
    );
  }
}

// What happens during setState:
void explainSetState() {
  // 1. setState() called
  // 2. Element marked as dirty
  // 3. Build scheduled for next frame
  // 4. build() called → new Widget created
  // 5. Element compares old and new Widget
  // 6. If same type: Element reused, RenderObject updated
  // 7. If different type: Element and RenderObject recreated
  // 8. RenderObject marks itself for layout/paint
  // 9. Layout phase: RenderObject calculates size
  // 10. Paint phase: RenderObject draws on canvas
}
```

### Performance Implications

```dart
class PerformanceDemo extends StatefulWidget {
  const PerformanceDemo({super.key});

  @override
  State<PerformanceDemo> createState() => _PerformanceDemoState();
}

class _PerformanceDemoState extends State<PerformanceDemo> {
  int _counter = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          Text('Count: $_counter'),
          
          // ✅ EFFICIENT
          // Widget recreated (cheap)
          // Element reused (saves memory)
          // RenderObject updated (only if needed)
          const ExpensiveWidget(),
          
          // ❌ INEFFICIENT
          // Don't do expensive work in build()
          ExpensiveComputationWidget(),
          
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
  const ExpensiveWidget({super.key});

  @override
  Widget build(BuildContext context) {
    // Widget is cheap to create
    return const Text('I am const, so Element is reused');
  }
}

class ExpensiveComputationWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // ❌ BAD: Expensive computation in build
    final result = List.generate(1000, (i) => i * i).reduce((a, b) => a + b);
    return Text('Result: $result');
  }
}
```

### Key Takeaways

| Aspect | Widget | Element | RenderObject |
|--------|--------|---------|-------------|
| **Mutability** | Immutable | Mutable | Mutable |
| **Lifespan** | Short (recreated often) | Long (reused) | Long (reused) |
| **Purpose** | Configuration | Lifecycle | Rendering |
| **Creation** | Every build | Once | Once |
| **Cost** | Cheap | Medium | Expensive |

---

## Q7: InheritedWidget

**Level:** Mid  
**Category:** State Management  
**Difficulty:** ⭐⭐⭐⭐

### The Question
What is InheritedWidget? How does it work? When should you use it?

### Answer

**InheritedWidget** is a special widget that allows data to be efficiently propagated down the widget tree. It's the foundation of:
- Theme
- MediaQuery
- Provider
- Riverpod

### Code Example

```dart
import 'package:flutter/material.dart';

// Step 1: Create InheritedWidget
class UserData extends InheritedWidget {
  final String username;
  final String email;

  const UserData({
    super.key,
    required this.username,
    required this.email,
    required super.child,
  });

  // Access method - called from descendants
  static UserData? of(BuildContext context) {
    return context.dependOnInheritedWidgetOfExactType<UserData>();
  }

  // Determines when to notify descendants
  @override
  bool updateShouldNotify(UserData oldWidget) {
    return username != oldWidget.username || email != oldWidget.email;
  }
}

// Step 2: Provide data at top of tree
class App extends StatelessWidget {
  const App({super.key});

  @override
  Widget build(BuildContext context) {
    return const UserData(
      username: 'john_doe',
      email: 'john@example.com',
      child: MaterialApp(
        home: HomeScreen(),
      ),
    );
  }
}

// Step 3: Access data from anywhere below
class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    // Access user data
    final userData = UserData.of(context);

    return Scaffold(
      appBar: AppBar(title: const Text('InheritedWidget Demo')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Username: ${userData?.username}'),
            Text('Email: ${userData?.email}'),
            const SizedBox(height: 20),
            const DeepWidget(),
          ],
        ),
      ),
    );
  }
}

// Can access from any depth
class DeepWidget extends StatelessWidget {
  const DeepWidget({super.key});

  @override
  Widget build(BuildContext context) {
    final userData = UserData.of(context);
    
    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Text('Deep widget accessing: ${userData?.username}'),
      ),
    );
  }
}
```

### InheritedWidget with State Management

```dart
// Create a stateful InheritedWidget
class CounterProvider extends StatefulWidget {
  final Widget child;

  const CounterProvider({super.key, required this.child});

  @override
  State<CounterProvider> createState() => CounterProviderState();

  // Helper to access state from descendants
  static CounterProviderState of(BuildContext context) {
    return context.dependOnInheritedWidgetOfExactType<_InheritedCounter>()!.state;
  }
}

class CounterProviderState extends State<CounterProvider> {
  int _counter = 0;

  int get counter => _counter;

  void increment() {
    setState(() {
      _counter++;
    });
  }

  void decrement() {
    setState(() {
      _counter--;
    });
  }

  @override
  Widget build(BuildContext context) {
    return _InheritedCounter(
      state: this,
      child: widget.child,
    );
  }
}

// Internal InheritedWidget
class _InheritedCounter extends InheritedWidget {
  final CounterProviderState state;

  const _InheritedCounter({
    required this.state,
    required super.child,
  });

  @override
  bool updateShouldNotify(_InheritedCounter oldWidget) {
    return state._counter != oldWidget.state._counter;
  }
}

// Usage
class CounterApp extends StatelessWidget {
  const CounterApp({super.key});

  @override
  Widget build(BuildContext context) {
    return CounterProvider(
      child: MaterialApp(
        home: const CounterScreen(),
      ),
    );
  }
}

class CounterScreen extends StatelessWidget {
  const CounterScreen({super.key});

  @override
  Widget build(BuildContext context) {
    // Access state
    final counterState = CounterProvider.of(context);

    return Scaffold(
      appBar: AppBar(title: const Text('Counter with InheritedWidget')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(
              '${counterState.counter}',
              style: const TextStyle(fontSize: 48),
            ),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                ElevatedButton(
                  onPressed: counterState.decrement,
                  child: const Icon(Icons.remove),
                ),
                const SizedBox(width: 20),
                ElevatedButton(
                  onPressed: counterState.increment,
                  child: const Icon(Icons.add),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}
```

### How InheritedWidget Works

```dart
// Behind the scenes:
class InheritedWidgetDemo {
  void explain() {
    // 1. InheritedWidget stores data
    // 2. Descendants call dependOnInheritedWidgetOfExactType()
    // 3. Flutter registers dependency
    // 4. When data changes, updateShouldNotify() is called
    // 5. If returns true, all dependent widgets rebuild
    // 6. Flutter efficiently updates only affected widgets
  }
}

// Dependency tracking
class DependencyDemo extends StatelessWidget {
  const DependencyDemo({super.key});

  @override
  Widget build(BuildContext context) {
    // This creates a dependency
    final userData = UserData.of(context);
    // Now this widget rebuilds when UserData changes

    return Text(userData?.username ?? '');
  }
}

// No dependency
class NoDependencyDemo extends StatelessWidget {
  const NoDependencyDemo({super.key});

  @override
  Widget build(BuildContext context) {
    // Using findAncestorWidgetOfExactType doesn't create dependency
    final userData = context.findAncestorWidgetOfExactType<UserData>();
    // This widget WON'T rebuild when UserData changes

    return Text(userData?.username ?? '');
  }
}
```

### Real-World Example: Theme

```dart
// This is how Theme works internally (simplified)
class MyTheme extends InheritedWidget {
  final Color primaryColor;
  final TextStyle textStyle;

  const MyTheme({
    super.key,
    required this.primaryColor,
    required this.textStyle,
    required super.child,
  });

  static MyTheme of(BuildContext context) {
    return context.dependOnInheritedWidgetOfExactType<MyTheme>()!;
  }

  @override
  bool updateShouldNotify(MyTheme oldWidget) {
    return primaryColor != oldWidget.primaryColor ||
        textStyle != oldWidget.textStyle;
  }
}

// Usage like Theme.of(context)
class ThemedWidget extends StatelessWidget {
  const ThemedWidget({super.key});

  @override
  Widget build(BuildContext context) {
    final theme = MyTheme.of(context);

    return Container(
      color: theme.primaryColor,
      child: Text(
        'Themed Text',
        style: theme.textStyle,
      ),
    );
  }
}
```

### Best Practices

✅ **DO:**
- Use InheritedWidget for data that many widgets need
- Implement efficient updateShouldNotify()
- Provide null-safe access methods
- Use for app-wide settings (theme, locale, user data)

❌ **DON'T:**
- Don't use for local widget state
- Don't put mutable objects without state management
- Don't forget updateShouldNotify() optimization
- Don't overuse (causes rebuilds)

---

## Q8: Widget Best Practices

**Level:** Mid  
**Category:** Best Practices  
**Difficulty:** ⭐⭐⭐

### The Question
What are the best practices for building efficient Flutter widgets?

### Answer

### 1. Use const Constructors

```dart
// ❌ BAD
class BadWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Title'),
        Icon(Icons.star),
      ],
    );
  }
}

// ✅ GOOD
class GoodWidget extends StatelessWidget {
  const GoodWidget({super.key});

  @override
  Widget build(BuildContext context) {
    return const Column(
      children: [
        Text('Title'),
        Icon(Icons.star),
      ],
    );
  }
}
```

### 2. Extract Widgets

```dart
// ❌ BAD: Everything in one build method
class BadScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          Container(
            padding: const EdgeInsets.all(16),
            child: Row(
              children: [
                const Icon(Icons.person),
                const SizedBox(width: 8),
                Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    const Text('John Doe'),
                    Text(
                      'john@example.com',
                      style: TextStyle(color: Colors.grey[600]),
                    ),
                  ],
                ),
              ],
            ),
          ),
          // 100 more lines...
        ],
      ),
    );
  }
}

// ✅ GOOD: Extract to separate widgets
class GoodScreen extends StatelessWidget {
  const GoodScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return const Scaffold(
      body: Column(
        children: [
          UserProfileTile(),
          // Other widgets...
        ],
      ),
    );
  }
}

class UserProfileTile extends StatelessWidget {
  const UserProfileTile({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.all(16),
      child: Row(
        children: [
          const Icon(Icons.person),
          const SizedBox(width: 8),
          Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              const Text('John Doe'),
              Text(
                'john@example.com',
                style: TextStyle(color: Colors.grey[600]),
              ),
            ],
          ),
        ],
      ),
    );
  }
}
```

### 3. Use ListView.builder for Long Lists

```dart
// ❌ BAD: Creates all items at once
class BadList extends StatelessWidget {
  final List<String> items = List.generate(1000, (i) => 'Item $i');

  @override
  Widget build(BuildContext context) {
    return ListView(
      children: items.map((item) => ListTile(title: Text(item))).toList(),
    );
  }
}

// ✅ GOOD: Lazy loading
class GoodList extends StatelessWidget {
  final List<String> items = List.generate(1000, (i) => 'Item $i');

  const GoodList({super.key});

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      itemCount: items.length,
      itemBuilder: (context, index) {
        return ListTile(title: Text(items[index]));
      },
    );
  }
}
```

### 4. Minimize Widget Rebuilds

```dart
class RebuildOptimization extends StatefulWidget {
  const RebuildOptimization({super.key});

  @override
  State<RebuildOptimization> createState() => _RebuildOptimizationState();
}

class _RebuildOptimizationState extends State<RebuildOptimization> {
  int _counter = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          // ❌ BAD: Rebuilds on every setState
          ExpensiveWidget(),
          
          // ✅ GOOD: Never rebuilds
          const ExpensiveWidget(),
          
          Text('Counter: $_counter'),
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
  const ExpensiveWidget({super.key});

  @override
  Widget build(BuildContext context) {
    // Expensive operation
    return Container(
      height: 200,
      color: Colors.blue,
    );
  }
}
```

### 5. Use Keys Appropriately

```dart
class KeyUsage extends StatefulWidget {
  const KeyUsage({super.key});

  @override
  State<KeyUsage> createState() => _KeyUsageState();
}

class _KeyUsageState extends State<KeyUsage> {
  List<Todo> todos = [
    Todo(id: 1, title: 'Task 1'),
    Todo(id: 2, title: 'Task 2'),
  ];

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // ✅ GOOD: Keys for reorderable lists
        ...todos.map(
          (todo) => TodoItem(
            key: ValueKey(todo.id),
            todo: todo,
          ),
        ),
      ],
    );
  }
}

class Todo {
  final int id;
  final String title;
  Todo({required this.id, required this.title});
}

class TodoItem extends StatefulWidget {
  final Todo todo;

  const TodoItem({super.key, required this.todo});

  @override
  State<TodoItem> createState() => _TodoItemState();
}

class _TodoItemState extends State<TodoItem> {
  bool isChecked = false;

  @override
  Widget build(BuildContext context) {
    return CheckboxListTile(
      title: Text(widget.todo.title),
      value: isChecked,
      onChanged: (value) => setState(() => isChecked = value ?? false),
    );
  }
}
```

### 6. Dispose Resources

```dart
class ResourceManagement extends StatefulWidget {
  const ResourceManagement({super.key});

  @override
  State<ResourceManagement> createState() => _ResourceManagementState();
}

class _ResourceManagementState extends State<ResourceManagement> {
  late TextEditingController _controller;
  late AnimationController _animationController;
  late StreamSubscription _subscription;

  @override
  void initState() {
    super.initState();
    _controller = TextEditingController();
    _animationController = AnimationController(
      vsync: this as TickerProvider,
      duration: const Duration(seconds: 1),
    );
    _subscription = Stream.periodic(const Duration(seconds: 1))
        .listen((event) {});
  }

  @override
  void dispose() {
    // ✅ ALWAYS dispose
    _controller.dispose();
    _animationController.dispose();
    _subscription.cancel();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return TextField(controller: _controller);
  }
}
```

### 7. Use StatelessWidget When Possible

```dart
// ❌ BAD: StatefulWidget for static content
class BadStaticWidget extends StatefulWidget {
  final String title;
  const BadStaticWidget({super.key, required this.title});

  @override
  State<BadStaticWidget> createState() => _BadStaticWidgetState();
}

class _BadStaticWidgetState extends State<BadStaticWidget> {
  @override
  Widget build(BuildContext context) {
    return Text(widget.title);
  }
}

// ✅ GOOD: StatelessWidget
class GoodStaticWidget extends StatelessWidget {
  final String title;
  const GoodStaticWidget({super.key, required this.title});

  @override
  Widget build(BuildContext context) {
    return Text(title);
  }
}
```

### Best Practices Summary

✅ **DO:**
- Use const constructors everywhere possible
- Extract complex widgets to separate classes
- Use ListView.builder for long lists
- Dispose controllers and subscriptions
- Use StatelessWidget when no state is needed
- Use Keys for lists that reorder
- Keep build methods simple and fast

❌ **DON'T:**
- Don't put expensive operations in build()
- Don't create unnecessary StatefulWidgets
- Don't forget to dispose resources
- Don't nest widgets too deeply
- Don't use ListView() for large lists

---

## Q9: Common Widget Mistakes

**Level:** Mid  
**Category:** Debugging  
**Difficulty:** ⭐⭐⭐

### The Question
What are the most common mistakes developers make with widgets?

### Answer

### 1. Context Misuse

```dart
// ❌ WRONG: Using wrong context
class WrongContext extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: ElevatedButton(
        onPressed: () {
          // ERROR: Scaffold.of() called with context that doesn't have Scaffold ancestor
          Scaffold.of(context).showSnackBar(
            const SnackBar(content: Text('Error')),
          );
        },
        child: const Text('Show'),
      ),
    );
  }
}

// ✅ CORRECT: Use Builder or extract widget
class CorrectContext extends StatelessWidget {
  const CorrectContext({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Builder(
        builder: (context) {
          return ElevatedButton(
            onPressed: () {
              ScaffoldMessenger.of(context).showSnackBar(
                const SnackBar(content: Text('Success')),
              );
            },
            child: const Text('Show'),
          );
        },
      ),
    );
  }
}
```

### 2. Not Using const

```dart
// ❌ INEFFICIENT
class WithoutConst extends StatefulWidget {
  @override
  State<WithoutConst> createState() => _WithoutConstState();
}

class _WithoutConstState extends State<WithoutConst> {
  int _count = 0;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Count: $_count'),
        Icon(Icons.star), // Rebuilt every time
        ElevatedButton(
          onPressed: () => setState(() => _count++),
          child: Text('Increment'),
        ),
      ],
    );
  }
}

// ✅ EFFICIENT
class WithConst extends StatefulWidget {
  const WithConst({super.key});

  @override
  State<WithConst> createState() => _WithConstState();
}

class _WithConstState extends State<WithConst> {
  int _count = 0;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Count: $_count'),
        const Icon(Icons.star), // Never rebuilt
        ElevatedButton(
          onPressed: () => setState(() => _count++),
          child: const Text('Increment'),
        ),
      ],
    );
  }
}
```

### 3. setState in Wrong Place

```dart
// ❌ WRONG: setState in initState
class SetStateInInitState extends StatefulWidget {
  @override
  State<SetStateInInitState> createState() => _SetStateInInitStateState();
}

class _SetStateInInitStateState extends State<SetStateInInitState> {
  @override
  void initState() {
    super.initState();
    setState(() {}); // ERROR: Called during build
  }

  @override
  Widget build(BuildContext context) => Container();
}

// ✅ CORRECT: Initialize directly
class CorrectInit extends StatefulWidget {
  @override
  State<CorrectInit> createState() => _CorrectInitState();
}

class _CorrectInitState extends State<CorrectInit> {
  int _value = 0;

  @override
  void initState() {
    super.initState();
    _value = 10; // Direct assignment
  }

  @override
  Widget build(BuildContext context) => Text('$_value');
}
```

### 4. Memory Leaks

```dart
// ❌ MEMORY LEAK: Not disposing
class MemoryLeak extends StatefulWidget {
  @override
  State<MemoryLeak> createState() => _MemoryLeakState();
}

class _MemoryLeakState extends State<MemoryLeak> {
  late Timer _timer;
  late TextEditingController _controller;

  @override
  void initState() {
    super.initState();
    _timer = Timer.periodic(Duration(seconds: 1), (_) {});
    _controller = TextEditingController();
  }

  // Missing dispose - MEMORY LEAK!

  @override
  Widget build(BuildContext context) => Container();
}

// ✅ CORRECT: Always dispose
class NoMemoryLeak extends StatefulWidget {
  const NoMemoryLeak({super.key});

  @override
  State<NoMemoryLeak> createState() => _NoMemoryLeakState();
}

class _NoMemoryLeakState extends State<NoMemoryLeak> {
  late Timer _timer;
  late TextEditingController _controller;

  @override
  void initState() {
    super.initState();
    _timer = Timer.periodic(const Duration(seconds: 1), (_) {});
    _controller = TextEditingController();
  }

  @override
  void dispose() {
    _timer.cancel();
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => Container();
}
```

### 5. Async setState Without mounted Check

```dart
// ❌ WRONG: setState after widget disposed
class AsyncSetStateWrong extends StatefulWidget {
  @override
  State<AsyncSetStateWrong> createState() => _AsyncSetStateWrongState();
}

class _AsyncSetStateWrongState extends State<AsyncSetStateWrong> {
  String _data = '';

  @override
  void initState() {
    super.initState();
    loadData();
  }

  Future<void> loadData() async {
    await Future.delayed(Duration(seconds: 5));
    setState(() => _data = 'Loaded'); // Might crash!
  }

  @override
  Widget build(BuildContext context) => Text(_data);
}

// ✅ CORRECT: Check mounted
class AsyncSetStateCorrect extends StatefulWidget {
  const AsyncSetStateCorrect({super.key});

  @override
  State<AsyncSetStateCorrect> createState() => _AsyncSetStateCorrectState();
}

class _AsyncSetStateCorrectState extends State<AsyncSetStateCorrect> {
  String _data = '';

  @override
  void initState() {
    super.initState();
    loadData();
  }

  Future<void> loadData() async {
    await Future.delayed(const Duration(seconds: 5));
    if (mounted) {
      setState(() => _data = 'Loaded');
    }
  }

  @override
  Widget build(BuildContext context) => Text(_data);
}
```

---

## Q10: Custom Widgets

**Level:** Mid  
**Category:** Advanced  
**Difficulty:** ⭐⭐⭐⭐

### The Question
How do you create reusable custom widgets? What are the best practices?

### Answer

### Simple Custom Widget

```dart
import 'package:flutter/material.dart';

// Basic custom widget
class CustomButton extends StatelessWidget {
  final String text;
  final VoidCallback onPressed;
  final Color? color;
  final IconData? icon;

  const CustomButton({
    super.key,
    required this.text,
    required this.onPressed,
    this.color,
    this.icon,
  });

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: onPressed,
      style: ElevatedButton.styleFrom(
        backgroundColor: color ?? Theme.of(context).primaryColor,
        padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 12),
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(8),
        ),
      ),
      child: Row(
        mainAxisSize: MainAxisSize.min,
        children: [
          if (icon != null) ..[
            Icon(icon),
            const SizedBox(width: 8),
          ],
          Text(text),
        ],
      ),
    );
  }
}

// Usage
class CustomButtonDemo extends StatelessWidget {
  const CustomButtonDemo({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        CustomButton(
          text: 'Save',
          onPressed: () {},
          icon: Icons.save,
        ),
        CustomButton(
          text: 'Delete',
          onPressed: () {},
          color: Colors.red,
          icon: Icons.delete,
        ),
      ],
    );
  }
}
```

### Custom Widget with Configuration

```dart
// Configuration class
class CardConfig {
  final Color backgroundColor;
  final double elevation;
  final BorderRadius borderRadius;
  final EdgeInsets padding;

  const CardConfig({
    this.backgroundColor = Colors.white,
    this.elevation = 2.0,
    this.borderRadius = const BorderRadius.all(Radius.circular(8)),
    this.padding = const EdgeInsets.all(16),
  });
}

// Custom card widget
class CustomCard extends StatelessWidget {
  final Widget child;
  final CardConfig? config;
  final VoidCallback? onTap;

  const CustomCard({
    super.key,
    required this.child,
    this.config,
    this.onTap,
  });

  @override
  Widget build(BuildContext context) {
    final cardConfig = config ?? const CardConfig();

    return Card(
      color: cardConfig.backgroundColor,
      elevation: cardConfig.elevation,
      shape: RoundedRectangleBorder(
        borderRadius: cardConfig.borderRadius,
      ),
      child: InkWell(
        onTap: onTap,
        borderRadius: cardConfig.borderRadius,
        child: Padding(
          padding: cardConfig.padding,
          child: child,
        ),
      ),
    );
  }
}
```

### Custom Stateful Widget

```dart
// Custom expandable widget
class ExpandableCard extends StatefulWidget {
  final String title;
  final Widget child;
  final bool initiallyExpanded;

  const ExpandableCard({
    super.key,
    required this.title,
    required this.child,
    this.initiallyExpanded = false,
  });

  @override
  State<ExpandableCard> createState() => _ExpandableCardState();
}

class _ExpandableCardState extends State<ExpandableCard>
    with SingleTickerProviderStateMixin {
  late bool _isExpanded;
  late AnimationController _controller;
  late Animation<double> _animation;

  @override
  void initState() {
    super.initState();
    _isExpanded = widget.initiallyExpanded;
    _controller = AnimationController(
      duration: const Duration(milliseconds: 300),
      vsync: this,
    );
    _animation = CurvedAnimation(
      parent: _controller,
      curve: Curves.easeInOut,
    );

    if (_isExpanded) {
      _controller.value = 1.0;
    }
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  void _toggle() {
    setState(() {
      _isExpanded = !_isExpanded;
      if (_isExpanded) {
        _controller.forward();
      } else {
        _controller.reverse();
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    return Card(
      child: Column(
        children: [
          ListTile(
            title: Text(widget.title),
            trailing: RotationTransition(
              turns: Tween(begin: 0.0, end: 0.5).animate(_animation),
              child: const Icon(Icons.expand_more),
            ),
            onTap: _toggle,
          ),
          SizeTransition(
            sizeFactor: _animation,
            child: Padding(
              padding: const EdgeInsets.all(16.0),
              child: widget.child,
            ),
          ),
        ],
      ),
    );
  }
}
```

### Custom Widget with Builder Pattern

```dart
// Builder pattern for complex widgets
class CustomDialog {
  final BuildContext context;
  String? _title;
  String? _message;
  String? _positiveButton;
  String? _negativeButton;
  VoidCallback? _onPositive;
  VoidCallback? _onNegative;

  CustomDialog(this.context);

  CustomDialog setTitle(String title) {
    _title = title;
    return this;
  }

  CustomDialog setMessage(String message) {
    _message = message;
    return this;
  }

  CustomDialog setPositiveButton(String text, VoidCallback onPressed) {
    _positiveButton = text;
    _onPositive = onPressed;
    return this;
  }

  CustomDialog setNegativeButton(String text, VoidCallback onPressed) {
    _negativeButton = text;
    _onNegative = onPressed;
    return this;
  }

  void show() {
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: _title != null ? Text(_title!) : null,
        content: _message != null ? Text(_message!) : null,
        actions: [
          if (_negativeButton != null)
            TextButton(
              onPressed: () {
                Navigator.pop(context);
                _onNegative?.call();
              },
              child: Text(_negativeButton!),
            ),
          if (_positiveButton != null)
            ElevatedButton(
              onPressed: () {
                Navigator.pop(context);
                _onPositive?.call();
              },
              child: Text(_positiveButton!),
            ),
        ],
      ),
    );
  }
}

// Usage
class DialogDemo extends StatelessWidget {
  const DialogDemo({super.key});

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () {
        CustomDialog(context)
            .setTitle('Confirm Delete')
            .setMessage('Are you sure you want to delete this item?')
            .setPositiveButton('Delete', () {
              print('Deleted');
            })
            .setNegativeButton('Cancel', () {
              print('Cancelled');
            })
            .show();
      },
      child: const Text('Show Dialog'),
    );
  }
}
```

### Best Practices for Custom Widgets

✅ **DO:**
- Make widgets reusable with parameters
- Use const constructors when possible
- Provide sensible defaults
- Document parameters with comments
- Follow Flutter naming conventions
- Dispose resources in StatefulWidgets

❌ **DON'T:**
- Don't hardcode values
- Don't make widgets too specific
- Don't forget to handle null values
- Don't create widgets that do too much

---

## Summary

We've covered essential widget concepts:

6. **Widget Trees** - Understanding Widget, Element, RenderObject
7. **InheritedWidget** - Efficient data propagation
8. **Best Practices** - Writing efficient widgets
9. **Common Mistakes** - Pitfalls to avoid
10. **Custom Widgets** - Creating reusable components

### Next Steps
- Practice building custom widgets
- Study Flutter's built-in widgets source code
- Learn advanced animation widgets
- Explore performance profiling tools

---

*Return to [Part 1](widgets.md) for Q1-Q5*