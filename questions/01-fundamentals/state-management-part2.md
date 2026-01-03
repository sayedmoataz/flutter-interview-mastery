# State Management - Part 2 (Q6-Q10)

*This is a continuation of [state-management.md](state-management.md)*

---

## Q6: Consumer vs Selector

**Level:** Mid  
**Category:** State Management  
**Difficulty:** ⭐⭐⭐⭐

### The Question
What's the difference between Consumer and Selector in Provider? When should you use each?

### Answer

**Consumer** rebuilds whenever ANY change happens in the provider.

**Selector** rebuilds ONLY when a specific property/value changes. It's more performant for large models.

### Code Example

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

// Model with multiple properties
class UserModel extends ChangeNotifier {
  String _name = 'John';
  int _age = 25;
  String _email = 'john@example.com';
  int _loginCount = 0;

  String get name => _name;
  int get age => _age;
  String get email => _email;
  int get loginCount => _loginCount;

  void updateName(String newName) {
    _name = newName;
    notifyListeners();
  }

  void updateAge(int newAge) {
    _age = newAge;
    notifyListeners();
  }

  void updateEmail(String newEmail) {
    _email = newEmail;
    notifyListeners();
  }

  void incrementLoginCount() {
    _loginCount++;
    notifyListeners(); // All consumers rebuild!
  }
}

void main() {
  runApp(
    ChangeNotifierProvider(
      create: (_) => UserModel(),
      child: MyApp(),
    ),
  );
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: ComparisonScreen(),
    );
  }
}

class ComparisonScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Consumer vs Selector')),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // Using Consumer - Rebuilds on ANY change
            const Text('Using Consumer:', style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
            Consumer<UserModel>(
              builder: (context, user, child) {
                print('Consumer rebuilt for name');
                return Text('Name: ${user.name}', style: TextStyle(fontSize: 18));
              },
            ),
            const SizedBox(height: 20),
            
            // Using Selector - Rebuilds ONLY when name changes
            const Text('Using Selector:', style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
            Selector<UserModel, String>(
              selector: (context, user) => user.name,
              builder: (context, name, child) {
                print('Selector rebuilt for name');
                return Text('Name: $name', style: TextStyle(fontSize: 18));
              },
            ),
            const SizedBox(height: 40),
            
            // Buttons to test
            ElevatedButton(
              onPressed: () {
                context.read<UserModel>().updateName('Jane');
              },
              child: const Text('Change Name (Both rebuild)'),
            ),
            ElevatedButton(
              onPressed: () {
                context.read<UserModel>().updateAge(30);
              },
              child: const Text('Change Age (Only Consumer rebuilds)'),
            ),
            ElevatedButton(
              onPressed: () {
                context.read<UserModel>().incrementLoginCount();
              },
              child: const Text('Increment Login Count (Only Consumer rebuilds)'),
            ),
          ],
        ),
      ),
    );
  }
}
```

### Advanced Selector Examples

```dart
// Example 1: Selector with multiple properties
Selector<UserModel, ({String name, int age})]>(
  selector: (context, user) => (name: user.name, age: user.age),
  builder: (context, data, child) {
    return Text('${data.name} is ${data.age} years old');
  },
)

// Example 2: Selector with computation
Selector<UserModel, bool>(
  selector: (context, user) => user.age >= 18,
  builder: (context, isAdult, child) {
    return Text(isAdult ? 'Adult' : 'Minor');
  },
)

// Example 3: Selector with child optimization
Selector<UserModel, String>(
  selector: (context, user) => user.name,
  builder: (context, name, child) {
    return Column(
      children: [
        Text('Name: $name'),
        child!, // This never rebuilds
      ],
    );
  },
  child: const ExpensiveWidget(), // Built once
)

class ExpensiveWidget extends StatelessWidget {
  const ExpensiveWidget({super.key});

  @override
  Widget build(BuildContext context) {
    print('ExpensiveWidget built');
    return Container(
      height: 200,
      color: Colors.blue,
      child: const Center(child: Text('I am expensive')),
    );
  }
}
```

### Performance Comparison

```dart
class PerformanceComparison extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // ❌ INEFFICIENT: Rebuilds on any change
        Consumer<UserModel>(
          builder: (context, user, _) => Text(user.name),
        ),
        Consumer<UserModel>(
          builder: (context, user, _) => Text('${user.age}'),
        ),
        Consumer<UserModel>(
          builder: (context, user, _) => Text(user.email),
        ),
        // If loginCount changes, ALL three rebuild!

        // ✅ EFFICIENT: Each rebuilds only when its data changes
        Selector<UserModel, String>(
          selector: (_, user) => user.name,
          builder: (_, name, __) => Text(name),
        ),
        Selector<UserModel, int>(
          selector: (_, user) => user.age,
          builder: (_, age, __) => Text('$age'),
        ),
        Selector<UserModel, String>(
          selector: (_, user) => user.email,
          builder: (_, email, __) => Text(email),
        ),
        // If loginCount changes, NONE rebuild!
      ],
    );
  }
}
```

### Comparison Table

| Feature | Consumer | Selector |
|---------|----------|----------|
| Rebuilds | On any change | Only when selected value changes |
| Performance | Good | Better |
| Complexity | Simple | Medium |
| Use case | Small models | Large models with many properties |
| Type safety | Runtime | Compile-time |

### Best Practices
- Use Consumer for simple cases
- Use Selector when model has many properties
- Use Selector.child for expensive static widgets
- Combine multiple properties in one Selector instead of multiple Consumers

### Common Mistakes to Avoid
- ❌ Using Consumer when only one property is needed
- ❌ Selector returning complex objects that always differ
- ❌ Not using shouldRebuild parameter when needed
- ❌ Creating new objects in selector function

---

## Q7: Multiple Providers

**Level:** Mid  
**Category:** State Management  
**Difficulty:** ⭐⭐⭐

### The Question
How do you provide multiple providers in a Flutter app? What are best practices?

### Answer

Use **MultiProvider** to provide multiple providers at once. It's cleaner and more maintainable than nesting multiple ChangeNotifierProviders.

### Code Example

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

// Multiple models
class AuthModel extends ChangeNotifier {
  User? _user;
  bool _isAuthenticated = false;

  User? get user => _user;
  bool get isAuthenticated => _isAuthenticated;

  void login(String email, String password) {
    _user = User(name: 'John', email: email);
    _isAuthenticated = true;
    notifyListeners();
  }

  void logout() {
    _user = null;
    _isAuthenticated = false;
    notifyListeners();
  }
}

class CartModel extends ChangeNotifier {
  final List<Product> _items = [];

  List<Product> get items => List.unmodifiable(_items);
  int get itemCount => _items.length;

  void addItem(Product product) {
    _items.add(product);
    notifyListeners();
  }

  void clear() {
    _items.clear();
    notifyListeners();
  }
}

class ThemeModel extends ChangeNotifier {
  bool _isDarkMode = false;

  bool get isDarkMode => _isDarkMode;

  void toggleTheme() {
    _isDarkMode = !_isDarkMode;
    notifyListeners();
  }
}

class User {
  final String name;
  final String email;
  User({required this.name, required this.email});
}

class Product {
  final String name;
  final double price;
  Product({required this.name, required this.price});
}

// ❌ BAD: Nested providers - hard to read
void mainBad() {
  runApp(
    ChangeNotifierProvider(
      create: (_) => AuthModel(),
      child: ChangeNotifierProvider(
        create: (_) => CartModel(),
        child: ChangeNotifierProvider(
          create: (_) => ThemeModel(),
          child: MyApp(),
        ),
      ),
    ),
  );
}

// ✅ GOOD: MultiProvider - clean and organized
void main() {
  runApp(
    MultiProvider(
      providers: [
        ChangeNotifierProvider(create: (_) => AuthModel()),
        ChangeNotifierProvider(create: (_) => CartModel()),
        ChangeNotifierProvider(create: (_) => ThemeModel()),
      ],
      child: MyApp(),
    ),
  );
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Consumer<ThemeModel>(
      builder: (context, themeModel, _) {
        return MaterialApp(
          theme: themeModel.isDarkMode ? ThemeData.dark() : ThemeData.light(),
          home: HomeScreen(),
        );
      },
    );
  }
}

class HomeScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Multiple Providers'),
        actions: [
          // Access CartModel
          Consumer<CartModel>(
            builder: (context, cart, _) {
              return IconButton(
                icon: Badge(
                  label: Text('${cart.itemCount}'),
                  child: const Icon(Icons.shopping_cart),
                ),
                onPressed: () {},
              );
            },
          ),
          // Access ThemeModel
          IconButton(
            icon: const Icon(Icons.brightness_6),
            onPressed: () {
              context.read<ThemeModel>().toggleTheme();
            },
          ),
        ],
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // Access AuthModel
            Consumer<AuthModel>(
              builder: (context, auth, _) {
                return Text(
                  auth.isAuthenticated
                      ? 'Welcome, ${auth.user?.name}'
                      : 'Not logged in',
                );
              },
            ),
            const SizedBox(height: 20),
            ElevatedButton(
              onPressed: () {
                final auth = context.read<AuthModel>();
                if (auth.isAuthenticated) {
                  auth.logout();
                } else {
                  auth.login('john@example.com', 'password');
                }
              },
              child: Consumer<AuthModel>(
                builder: (context, auth, _) {
                  return Text(auth.isAuthenticated ? 'Logout' : 'Login');
                },
              ),
            ),
            ElevatedButton(
              onPressed: () {
                context.read<CartModel>().addItem(
                      Product(name: 'Item', price: 9.99),
                    );
              },
              child: const Text('Add to Cart'),
            ),
          ],
        ),
      ),
    );
  }
}
```

### Provider with Dependencies

```dart
// When one provider depends on another
class OrderModel extends ChangeNotifier {
  final AuthModel authModel;
  final CartModel cartModel;

  OrderModel({required this.authModel, required this.cartModel});

  Future<void> placeOrder() async {
    if (!authModel.isAuthenticated) {
      throw Exception('Must be logged in');
    }

    if (cartModel.itemCount == 0) {
      throw Exception('Cart is empty');
    }

    // Place order logic
    print('Order placed by ${authModel.user?.name}');
    print('Items: ${cartModel.itemCount}');
    cartModel.clear();
    notifyListeners();
  }
}

// Setup with dependencies
void mainWithDependencies() {
  runApp(
    MultiProvider(
      providers: [
        ChangeNotifierProvider(create: (_) => AuthModel()),
        ChangeNotifierProvider(create: (_) => CartModel()),
        ChangeNotifierProvider(create: (_) => ThemeModel()),
        // ProxyProvider - depends on other providers
        ProxyProvider2<AuthModel, CartModel, OrderModel>(
          update: (_, auth, cart, previous) => OrderModel(
            authModel: auth,
            cartModel: cart,
          ),
        ),
      ],
      child: MyApp(),
    ),
  );
}
```

### Different Provider Types in MultiProvider

```dart
void mainAllTypes() {
  runApp(
    MultiProvider(
      providers: [
        // 1. ChangeNotifierProvider
        ChangeNotifierProvider(create: (_) => AuthModel()),
        
        // 2. Provider (simple value)
        Provider<String>(create: (_) => 'App Version 1.0'),
        
        // 3. FutureProvider (async data)
        FutureProvider<UserSettings>(
          initialData: UserSettings.empty(),
          create: (_) => fetchUserSettings(),
        ),
        
        // 4. StreamProvider (stream data)
        StreamProvider<List<Notification>>(
          initialData: const [],
          create: (_) => notificationStream(),
        ),
        
        // 5. ProxyProvider (depends on others)
        ProxyProvider<AuthModel, ApiService>(
          update: (_, auth, __) => ApiService(authToken: auth.user?.token),
        ),
      ],
      child: MyApp(),
    ),
  );
}

class UserSettings {
  UserSettings.empty();
}

class Notification {}

class ApiService {
  final String? authToken;
  ApiService({this.authToken});
}

Future<UserSettings> fetchUserSettings() async {
  await Future.delayed(Duration(seconds: 1));
  return UserSettings.empty();
}

Stream<List<Notification>> notificationStream() {
  return Stream.periodic(Duration(seconds: 5), (_) => <Notification>[]);
}
```

### Best Practices
- Group related providers together in MultiProvider
- Place providers at the highest common ancestor
- Use ProxyProvider for dependencies
- Keep provider tree flat (not too many levels)
- Consider lazy loading providers

### Common Mistakes to Avoid
- ❌ Nesting providers instead of using MultiProvider
- ❌ Providing too many providers at root
- ❌ Not considering provider dependencies
- ❌ Creating providers too low in the tree

---

## Q8: State Management Mistakes

**Level:** Mid  
**Category:** Best Practices  
**Difficulty:** ⭐⭐⭐

### The Question
What are the most common state management mistakes in Flutter?

### Answer

### Top 10 State Management Mistakes

#### 1. Calling setState in dispose()

```dart
// ❌ WRONG: setState after dispose
class BadExample extends StatefulWidget {
  @override
  State<BadExample> createState() => _BadExampleState();
}

class _BadExampleState extends State<BadExample> {
  @override
  void dispose() {
    setState(() {}); // ERROR! Widget is being disposed
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => Container();
}

// ✅ CORRECT: No setState in dispose
class GoodExample extends StatefulWidget {
  @override
  State<GoodExample> createState() => _GoodExampleState();
}

class _GoodExampleState extends State<GoodExample> {
  @override
  void dispose() {
    // Just cleanup, no setState
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => Container();
}
```

#### 2. Not Checking mounted Before Async setState

```dart
// ❌ WRONG: Async setState without mounted check
class AsyncBad extends StatefulWidget {
  @override
  State<AsyncBad> createState() => _AsyncBadState();
}

class _AsyncBadState extends State<AsyncBad> {
  String _data = '';

  @override
  void initState() {
    super.initState();
    loadData();
  }

  Future<void> loadData() async {
    await Future.delayed(Duration(seconds: 5));
    setState(() => _data = 'Loaded'); // Might crash if disposed!
  }

  @override
  Widget build(BuildContext context) => Text(_data);
}

// ✅ CORRECT: Check mounted before setState
class AsyncGood extends StatefulWidget {
  @override
  State<AsyncGood> createState() => _AsyncGoodState();
}

class _AsyncGoodState extends State<AsyncGood> {
  String _data = '';

  @override
  void initState() {
    super.initState();
    loadData();
  }

  Future<void> loadData() async {
    await Future.delayed(Duration(seconds: 5));
    if (mounted) { // Check if still mounted
      setState(() => _data = 'Loaded');
    }
  }

  @override
  Widget build(BuildContext context) => Text(_data);
}
```

#### 3. Using context.watch in Callbacks

```dart
// ❌ WRONG: watch in onPressed
class WatchInCallback extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () {
        final counter = context.watch<Counter>(); // ERROR!
        counter.increment();
      },
      child: const Text('Increment'),
    );
  }
}

// ✅ CORRECT: Use read in callbacks
class ReadInCallback extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () {
        context.read<Counter>().increment(); // Correct!
      },
      child: const Text('Increment'),
    );
  }
}
```

#### 4. Creating Provider in build()

```dart
// ❌ WRONG: Provider in build method
class ProviderInBuild extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (_) => Counter(), // Recreated on every build!
      child: CounterWidget(),
    );
  }
}

// ✅ CORRECT: Provider above build
class ProviderOutsideBuild extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return CounterWidget(); // Provider created higher up
  }
}

void main() {
  runApp(
    ChangeNotifierProvider(
      create: (_) => Counter(), // Created once
      child: MyApp(),
    ),
  );
}
```

#### 5. Putting Everything in Global State

```dart
// ❌ BAD: Everything in global state
class GlobalStateOveruse extends ChangeNotifier {
  String _buttonText = 'Click me'; // Local UI state!
  bool _isLoading = false; // Could be local!
  String _userName = 'John'; // Should be global
  List<Product> _cart = []; // Should be global

  // Everything calls notifyListeners, rebuilding all widgets!
}

// ✅ GOOD: Separate local and global state
class UserModel extends ChangeNotifier {
  String _userName = 'John';
  String get userName => _userName;
  
  void updateName(String name) {
    _userName = name;
    notifyListeners();
  }
}

// Button text stays local
class ButtonWidget extends StatefulWidget {
  @override
  State<ButtonWidget> createState() => _ButtonWidgetState();
}

class _ButtonWidgetState extends State<ButtonWidget> {
  String _buttonText = 'Click me'; // Local state

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () => setState(() => _buttonText = 'Clicked!'),
      child: Text(_buttonText),
    );
  }
}
```

#### 6. Not Disposing Resources

```dart
// ❌ MEMORY LEAK: Not disposing
class LeakyWidget extends StatefulWidget {
  @override
  State<LeakyWidget> createState() => _LeakyWidgetState();
}

class _LeakyWidgetState extends State<LeakyWidget> {
  final TextEditingController _controller = TextEditingController();
  final StreamSubscription _subscription = someStream.listen((data) {});
  
  @override
  Widget build(BuildContext context) {
    return TextField(controller: _controller);
  }
  // No dispose - MEMORY LEAK!
}

// ✅ CORRECT: Always dispose
class NoLeakWidget extends StatefulWidget {
  @override
  State<NoLeakWidget> createState() => _NoLeakWidgetState();
}

class _NoLeakWidgetState extends State<NoLeakWidget> {
  late final TextEditingController _controller;
  late final StreamSubscription _subscription;

  @override
  void initState() {
    super.initState();
    _controller = TextEditingController();
    _subscription = someStream.listen((data) {});
  }

  @override
  void dispose() {
    _controller.dispose();
    _subscription.cancel();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return TextField(controller: _controller);
  }
}

Stream<String> someStream = Stream.value('test');
```

#### 7. Forgetting notifyListeners()

```dart
// ❌ WRONG: State changes but UI doesn't update
class CounterBad extends ChangeNotifier {
  int _count = 0;
  int get count => _count;

  void increment() {
    _count++;
    // Forgot notifyListeners() - UI won't update!
  }
}

// ✅ CORRECT: Always call notifyListeners
class CounterGood extends ChangeNotifier {
  int _count = 0;
  int get count => _count;

  void increment() {
    _count++;
    notifyListeners(); // UI updates!
  }
}
```

#### 8. setState in build Method

```dart
// ❌ INFINITE LOOP: setState in build
class SetStateInBuild extends StatefulWidget {
  @override
  State<SetStateInBuild> createState() => _SetStateInBuildState();
}

class _SetStateInBuildState extends State<SetStateInBuild> {
  int _count = 0;

  @override
  Widget build(BuildContext context) {
    setState(() => _count++); // INFINITE LOOP!
    return Text('Count: $_count');
  }
}

// ✅ CORRECT: setState in callbacks or lifecycle methods
class SetStateCorrect extends StatefulWidget {
  @override
  State<SetStateCorrect> createState() => _SetStateCorrectState();
}

class _SetStateCorrectState extends State<SetStateCorrect> {
  int _count = 0;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Count: $_count'),
        ElevatedButton(
          onPressed: () => setState(() => _count++),
          child: const Text('Increment'),
        ),
      ],
    );
  }
}
```

#### 9. Mixing Business Logic with UI

```dart
// ❌ BAD: Business logic in widget
class MixedLogic extends StatefulWidget {
  @override
  State<MixedLogic> createState() => _MixedLogicState();
}

class _MixedLogicState extends State<MixedLogic> {
  List<Product> _products = [];

  Future<void> loadProducts() async {
    // API call
    final response = await http.get(Uri.parse('https://api.example.com/products'));
    final json = jsonDecode(response.body);
    
    // Business logic
    final products = (json as List)
        .map((item) => Product.fromJson(item))
        .where((p) => p.price > 10)
        .toList();
    
    setState(() => _products = products);
  }

  @override
  Widget build(BuildContext context) {
    // UI code mixed with business logic above
    return ListView.builder(
      itemCount: _products.length,
      itemBuilder: (context, index) => ListTile(
        title: Text(_products[index].name),
      ),
    );
  }
}

// ✅ GOOD: Separate business logic
class ProductModel extends ChangeNotifier {
  List<Product> _products = [];
  List<Product> get products => List.unmodifiable(_products);

  Future<void> loadProducts() async {
    // API call
    final response = await http.get(Uri.parse('https://api.example.com/products'));
    final json = jsonDecode(response.body);
    
    // Business logic in model
    _products = (json as List)
        .map((item) => Product.fromJson(item))
        .where((p) => p.price > 10)
        .toList();
    
    notifyListeners();
  }
}

// Clean UI widget
class ProductList extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Consumer<ProductModel>(
      builder: (context, model, _) {
        return ListView.builder(
          itemCount: model.products.length,
          itemBuilder: (context, index) => ListTile(
            title: Text(model.products[index].name),
          ),
        );
      },
    );
  }
}
```

#### 10. Unnecessary Rebuilds

```dart
// ❌ INEFFICIENT: Everything rebuilds
class UnnecessaryRebuilds extends StatefulWidget {
  @override
  State<UnnecessaryRebuilds> createState() => _UnnecessaryRebuildsState();
}

class _UnnecessaryRebuildsState extends State<UnnecessaryRebuilds> {
  int _counter = 0;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        ExpensiveWidget(), // Rebuilds on every setState!
        Text('Counter: $_counter'),
        ElevatedButton(
          onPressed: () => setState(() => _counter++),
          child: const Text('Increment'),
        ),
      ],
    );
  }
}

// ✅ EFFICIENT: Extract and use const
class EfficientRebuilds extends StatefulWidget {
  @override
  State<EfficientRebuilds> createState() => _EfficientRebuildsState();
}

class _EfficientRebuildsState extends State<EfficientRebuilds> {
  int _counter = 0;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        const ExpensiveWidget(), // Never rebuilds!
        Text('Counter: $_counter'),
        ElevatedButton(
          onPressed: () => setState(() => _counter++),
          child: const Text('Increment'),
        ),
      ],
    );
  }
}

class Counter {
  void increment() {}
}
```

### Prevention Checklist

**Before committing:**
- [ ] All controllers/streams disposed?
- [ ] Checked `mounted` before async setState?
- [ ] Using `read` (not `watch`) in callbacks?
- [ ] Provider created outside build method?
- [ ] notifyListeners() called after state changes?
- [ ] No setState in build() or dispose()?
- [ ] Business logic separated from UI?
- [ ] Using const where possible?
- [ ] Local state kept local?
- [ ] No unnecessary rebuilds?

---

## Q9: When to Use setState vs Provider

**Level:** Mid  
**Category:** Decision Making  
**Difficulty:** ⭐⭐⭐

### The Question
How do you decide between setState and Provider? When should you use each?

### Answer

### Decision Tree

```
Does state need to be shared?
│
├─ NO → Use setState
│   └─ Simple, local widget state
│
└─ YES → How many widgets?
    │
    ├─ 2-3 widgets (parent-child) → Lift state up + setState
    │   └─ Pass data via constructors
    │
    └─ Many widgets / Deep tree → Use Provider
        └─ Global or complex shared state
```

### Code Examples

#### Use Case 1: Local State → setState

```dart
// ✅ CORRECT: setState for local state
class ExpandableCard extends StatefulWidget {
  final String title;
  final String content;

  const ExpandableCard({
    super.key,
    required this.title,
    required this.content,
  });

  @override
  State<ExpandableCard> createState() => _ExpandableCardState();
}

class _ExpandableCardState extends State<ExpandableCard> {
  bool _isExpanded = false; // Local UI state

  @override
  Widget build(BuildContext context) {
    return Card(
      child: Column(
        children: [
          ListTile(
            title: Text(widget.title),
            trailing: IconButton(
              icon: Icon(_isExpanded ? Icons.expand_less : Icons.expand_more),
              onPressed: () => setState(() => _isExpanded = !_isExpanded),
            ),
          ),
          if (_isExpanded)
            Padding(
              padding: const EdgeInsets.all(16.0),
              child: Text(widget.content),
            ),
        ],
      ),
    );
  }
}

// Why setState? 
// - State only matters to this widget
// - No other widget needs this state
// - Simple toggle behavior
```

#### Use Case 2: Parent-Child → Lift State + setState

```dart
// ✅ CORRECT: Lift state for parent-child communication
class ColorPicker extends StatefulWidget {
  @override
  State<ColorPicker> createState() => _ColorPickerState();
}

class _ColorPickerState extends State<ColorPicker> {
  Color _selectedColor = Colors.red;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // Child 1: Shows selected color
        ColorDisplay(color: _selectedColor),
        
        // Child 2: Buttons to change color
        ColorButtons(
          onColorSelected: (color) {
            setState(() => _selectedColor = color);
          },
        ),
      ],
    );
  }
}

class ColorDisplay extends StatelessWidget {
  final Color color;
  
  const ColorDisplay({super.key, required this.color});

  @override
  Widget build(BuildContext context) {
    return Container(
      width: 200,
      height: 200,
      color: color,
    );
  }
}

class ColorButtons extends StatelessWidget {
  final Function(Color) onColorSelected;
  
  const ColorButtons({super.key, required this.onColorSelected});

  @override
  Widget build(BuildContext context) {
    return Row(
      children: [
        ElevatedButton(
          onPressed: () => onColorSelected(Colors.red),
          child: const Text('Red'),
        ),
        ElevatedButton(
          onPressed: () => onColorSelected(Colors.blue),
          child: const Text('Blue'),
        ),
      ],
    );
  }
}

// Why not Provider?
// - Only 2-3 related widgets
// - State hierarchy is clear
// - No deep nesting
```

#### Use Case 3: Many Widgets / Deep Tree → Provider

```dart
// ✅ CORRECT: Provider for shared global state
class ThemeModel extends ChangeNotifier {
  bool _isDarkMode = false;
  
  bool get isDarkMode => _isDarkMode;
  
  void toggleTheme() {
    _isDarkMode = !_isDarkMode;
    notifyListeners();
  }
}

void main() {
  runApp(
    ChangeNotifierProvider(
      create: (_) => ThemeModel(),
      child: MyApp(),
    ),
  );
}

// Can be accessed anywhere in the app
class HomeScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final isDark = context.watch<ThemeModel>().isDarkMode;
    return Scaffold(
      backgroundColor: isDark ? Colors.black : Colors.white,
      body: Column(
        children: [
          SettingsButton(), // Deep in tree
          ProfileScreen(),  // Different branch
          DashboardWidget(), // Another branch
        ],
      ),
    );
  }
}

class SettingsButton extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return IconButton(
      icon: const Icon(Icons.settings),
      onPressed: () {
        context.read<ThemeModel>().toggleTheme();
      },
    );
  }
}

// Why Provider?
// - Many widgets need this state
// - Widgets are in different parts of tree
// - Would require passing through many layers
// - State is app-level, not local
```

### Comparison Matrix

| Criteria | setState | Lift State | Provider |
|----------|----------|------------|----------|
| **Widgets affected** | 1 | 2-3 (related) | Many (anywhere) |
| **Tree depth** | N/A | 1-2 levels | Any depth |
| **State scope** | Local | Parent-child | Global/app |
| **Complexity** | Simple | Medium | Medium-High |
| **Boilerplate** | None | Callbacks | Setup |
| **Testability** | Hard | Medium | Easy |
| **Examples** | Toggle, form input | Color picker | Auth, cart, theme |

### Real-World Decision Examples

```dart
// ❌ OVERKILL: Provider for simple checkbox
class CheckboxBad extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (_) => CheckboxModel(),
      child: Consumer<CheckboxModel>(
        builder: (context, model, _) {
          return Checkbox(
            value: model.isChecked,
            onChanged: (_) => model.toggle(),
          );
        },
      ),
    );
  }
}

// ✅ CORRECT: setState for simple checkbox
class CheckboxGood extends StatefulWidget {
  @override
  State<CheckboxGood> createState() => _CheckboxGoodState();
}

class _CheckboxGoodState extends State<CheckboxGood> {
  bool _isChecked = false;

  @override
  Widget build(BuildContext context) {
    return Checkbox(
      value: _isChecked,
      onChanged: (value) => setState(() => _isChecked = value ?? false),
    );
  }
}

// ❌ WRONG: setState for shopping cart
class CartBad extends StatefulWidget {
  @override
  State<CartBad> createState() => _CartBadState();
}

class _CartBadState extends State<CartBad> {
  List<Product> _items = [];

  @override
  Widget build(BuildContext context) {
    // Problem: How do you access _items from product screen?
    // How do you show cart count in AppBar?
    // Would need to pass through many constructors!
    return Container();
  }
}

// ✅ CORRECT: Provider for shopping cart
class CartGood {
  // Use Provider (shown in previous examples)
  // Can access from anywhere: ProductScreen, AppBar, CheckoutScreen
}
```

### Best Practices

**Use setState when:**
- State is truly local (form input, toggle, animation)
- Only one widget cares about the state
- Simple UI state (not business logic)
- You're building a reusable widget

**Use Provider when:**
- State needs to be shared across multiple screens
- Deep widget tree (avoiding prop drilling)
- Business logic needs to be tested
- State persists across navigation
- Multiple unrelated widgets need same state

### Common Mistakes
- ❌ Using Provider for every piece of state
- ❌ Using setState for app-wide state
- ❌ Over-engineering simple widgets
- ❌ Not considering testability

---

## Q10: StreamController and Streams

**Level:** Mid  
**Category:** Reactive State  
**Difficulty:** ⭐⭐⭐⭐

### The Question
What are Streams and StreamController? How do you use them for state management?

### Answer

**Streams** are sequences of asynchronous events. They're useful for:
- Real-time data (chat messages, notifications)
- User input events
- Sensor data
- WebSocket connections

**StreamController** creates and manages streams.

### Code Example

```dart
import 'dart:async';
import 'package:flutter/material.dart';

// Example 1: Basic StreamController
class StreamExample extends StatefulWidget {
  @override
  State<StreamExample> createState() => _StreamExampleState();
}

class _StreamExampleState extends State<StreamExample> {
  final StreamController<int> _controller = StreamController<int>();
  int _counter = 0;

  @override
  void initState() {
    super.initState();
    // Add initial value
    _controller.add(0);
  }

  void _increment() {
    _counter++;
    _controller.add(_counter); // Add to stream
  }

  @override
  void dispose() {
    _controller.close(); // Always close streams!
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: StreamBuilder<int>(
          stream: _controller.stream,
          initialData: 0,
          builder: (context, snapshot) {
            if (snapshot.hasError) {
              return Text('Error: ${snapshot.error}');
            }
            
            if (!snapshot.hasData) {
              return const CircularProgressIndicator();
            }

            return Text(
              'Count: ${snapshot.data}',
              style: const TextStyle(fontSize: 48),
            );
          },
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _increment,
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

### Stream Types

```dart
// Single-subscription stream (one listener)
final controller = StreamController<int>();
controller.stream.listen((data) => print(data)); // Only one listener allowed

// Broadcast stream (multiple listeners)
final broadcastController = StreamController<int>.broadcast();
broadcastController.stream.listen((data) => print('Listener 1: $data'));
broadcastController.stream.listen((data) => print('Listener 2: $data'));
```

### Real-World Example: Chat Application

```dart
class Message {
  final String text;
  final String sender;
  final DateTime timestamp;

  Message({
    required this.text,
    required this.sender,
    required this.timestamp,
  });
}

class ChatService {
  final StreamController<List<Message>> _messagesController =
      StreamController<List<Message>>.broadcast();

  Stream<List<Message>> get messages => _messagesController.stream;

  final List<Message> _messageList = [];

  void sendMessage(String text, String sender) {
    final message = Message(
      text: text,
      sender: sender,
      timestamp: DateTime.now(),
    );

    _messageList.add(message);
    _messagesController.add(List.from(_messageList)); // Emit updated list
  }

  void dispose() {
    _messagesController.close();
  }
}

class ChatScreen extends StatefulWidget {
  @override
  State<ChatScreen> createState() => _ChatScreenState();
}

class _ChatScreenState extends State<ChatScreen> {
  final ChatService _chatService = ChatService();
  final TextEditingController _textController = TextEditingController();

  @override
  void dispose() {
    _chatService.dispose();
    _textController.dispose();
    super.dispose();
  }

  void _sendMessage() {
    if (_textController.text.isNotEmpty) {
      _chatService.sendMessage(_textController.text, 'User');
      _textController.clear();
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Chat')),
      body: Column(
        children: [
          Expanded(
            child: StreamBuilder<List<Message>>(
              stream: _chatService.messages,
              initialData: const [],
              builder: (context, snapshot) {
                if (!snapshot.hasData || snapshot.data!.isEmpty) {
                  return const Center(child: Text('No messages yet'));
                }

                final messages = snapshot.data!;
                return ListView.builder(
                  itemCount: messages.length,
                  itemBuilder: (context, index) {
                    final message = messages[index];
                    return ListTile(
                      title: Text(message.text),
                      subtitle: Text(
                        '${message.sender} - ${message.timestamp.toString()}',
                      ),
                    );
                  },
                );
              },
            ),
          ),
          Padding(
            padding: const EdgeInsets.all(8.0),
            child: Row(
              children: [
                Expanded(
                  child: TextField(
                    controller: _textController,
                    decoration: const InputDecoration(
                      hintText: 'Type a message...',
                    ),
                  ),
                ),
                IconButton(
                  icon: const Icon(Icons.send),
                  onPressed: _sendMessage,
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```

### StreamProvider with Provider Package

```dart
import 'package:provider/provider.dart';

// Timer stream example
Stream<int> timerStream() {
  return Stream.periodic(
    const Duration(seconds: 1),
    (count) => count,
  );
}

void main() {
  runApp(
    StreamProvider<int>.value(
      value: timerStream(),
      initialData: 0,
      child: MyApp(),
    ),
  );
}

class TimerWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final count = context.watch<int>();
    
    return Scaffold(
      body: Center(
        child: Text(
          'Timer: $count seconds',
          style: const TextStyle(fontSize: 32),
        ),
      ),
    );
  }
}
```

### Advanced: Transform Streams

```dart
class AdvancedStreamExample extends StatefulWidget {
  @override
  State<AdvancedStreamExample> createState() => _AdvancedStreamExampleState();
}

class _AdvancedStreamExampleState extends State<AdvancedStreamExample> {
  final StreamController<String> _inputController = StreamController<String>();
  late Stream<String> _outputStream;

  @override
  void initState() {
    super.initState();
    
    // Transform stream
    _outputStream = _inputController.stream
        .where((text) => text.isNotEmpty) // Filter empty
        .map((text) => text.toUpperCase()) // Transform
        .debounce(Duration(milliseconds: 500)); // Debounce
  }

  @override
  void dispose() {
    _inputController.close();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          TextField(
            onChanged: (text) => _inputController.add(text),
            decoration: const InputDecoration(
              hintText: 'Type something...',
            ),
          ),
          StreamBuilder<String>(
            stream: _outputStream,
            builder: (context, snapshot) {
              return Text(
                snapshot.hasData ? snapshot.data! : 'Waiting...',
                style: const TextStyle(fontSize: 24),
              );
            },
          ),
        ],
      ),
    );
  }
}

// Extension for debounce
extension StreamExtensions<T> on Stream<T> {
  Stream<T> debounce(Duration duration) {
    final controller = StreamController<T>();
    Timer? timer;

    listen(
      (data) {
        timer?.cancel();
        timer = Timer(duration, () => controller.add(data));
      },
      onError: controller.addError,
      onDone: controller.close,
    );

    return controller.stream;
  }
}
```

### When to Use Streams

**Use Streams when:**
- Real-time data updates (chat, notifications)
- Server-sent events / WebSockets
- Sensor data / location updates
- User input that needs debouncing
- Event sequences over time

**Don't use Streams when:**
- Simple state that changes occasionally
- One-time async operations (use Future)
- Local UI state (use setState)

### Best Practices
- Always close StreamControllers in dispose()
- Use broadcast streams for multiple listeners
- Provide initialData to StreamBuilder
- Handle errors in StreamBuilder
- Consider using StreamProvider for app-wide streams

### Common Mistakes
- ❌ Not closing StreamController
- ❌ Using single-subscription stream for multiple listeners
- ❌ Not handling errors
- ❌ Overusing streams for simple state
- ❌ Creating new stream in build method

---

## Summary

We've covered advanced state management topics:

6. **Consumer vs Selector** - Performance optimization
7. **Multiple Providers** - Managing multiple state sources
8. **Common Mistakes** - Pitfalls to avoid
9. **setState vs Provider** - Decision making
10. **Streams** - Reactive programming

### Next Steps
- Practice with real projects
- Explore Riverpod for modern state management
- Learn BLoC pattern for large apps
- Study reactive programming concepts

---

*Return to [Part 1](state-management.md) for Q1-Q5*