# Widgets & UI - Part 3 (Q8-Q10)

*This is a continuation of [widgets-part2.md](widgets-part2.md)*

---

## Q8: Common Widgets

**Level:** Junior  
**Category:** Widgets  
**Difficulty:** ⭐⭐

### The Question
What are the most common Flutter widgets? Provide examples of each.

### Answer

Flutter has hundreds of widgets. Here are the most commonly used ones categorized by purpose.

### Layout Widgets

```dart
import 'package:flutter/material.dart';

// 1. Container - Box model widget
class ContainerExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Container(
      width: 200,
      height: 200,
      padding: const EdgeInsets.all(16),
      margin: const EdgeInsets.all(8),
      decoration: BoxDecoration(
        color: Colors.blue,
        borderRadius: BorderRadius.circular(12),
        boxShadow: [
          BoxShadow(
            color: Colors.black26,
            blurRadius: 10,
            offset: Offset(0, 5),
          ),
        ],
      ),
      child: const Text('Container'),
    );
  }
}

// 2. Row - Horizontal layout
class RowExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Row(
      mainAxisAlignment: MainAxisAlignment.spaceAround,
      crossAxisAlignment: CrossAxisAlignment.center,
      children: [
        Icon(Icons.star),
        Text('Rating'),
        Text('4.5'),
      ],
    );
  }
}

// 3. Column - Vertical layout
class ColumnExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      mainAxisAlignment: MainAxisAlignment.center,
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text('Title'),
        Text('Subtitle'),
        ElevatedButton(onPressed: () {}, child: Text('Action')),
      ],
    );
  }
}

// 4. Stack - Overlapping widgets
class StackExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Stack(
      children: [
        Container(width: 300, height: 300, color: Colors.blue),
        Positioned(
          top: 50,
          left: 50,
          child: Container(width: 100, height: 100, color: Colors.red),
        ),
        Positioned(
          bottom: 20,
          right: 20,
          child: Icon(Icons.favorite, color: Colors.white),
        ),
      ],
    );
  }
}

// 5. Padding - Add padding
class PaddingExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.all(16.0),
      child: Text('Padded text'),
    );
  }
}

// 6. Center - Center child
class CenterExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Center(
      child: Text('Centered'),
    );
  }
}

// 7. SizedBox - Fixed size or spacing
class SizedBoxExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('First'),
        SizedBox(height: 20), // Spacing
        Text('Second'),
        SizedBox(
          width: 100,
          height: 100,
          child: Container(color: Colors.blue),
        ),
      ],
    );
  }
}

// 8. Expanded - Fill available space
class ExpandedExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Row(
      children: [
        Container(width: 50, height: 50, color: Colors.red),
        Expanded(
          child: Container(color: Colors.blue), // Takes remaining space
        ),
        Container(width: 50, height: 50, color: Colors.green),
      ],
    );
  }
}

// 9. Flexible - Flexible space allocation
class FlexibleExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Row(
      children: [
        Flexible(
          flex: 1,
          child: Container(height: 50, color: Colors.red),
        ),
        Flexible(
          flex: 2, // Takes 2x space of flex: 1
          child: Container(height: 50, color: Colors.blue),
        ),
        Flexible(
          flex: 1,
          child: Container(height: 50, color: Colors.green),
        ),
      ],
    );
  }
}
```

### UI Widgets

```dart
// 10. Text - Display text
class TextExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Simple text'),
        Text(
          'Styled text',
          style: TextStyle(
            fontSize: 24,
            fontWeight: FontWeight.bold,
            color: Colors.blue,
          ),
        ),
        Text(
          'Very long text that will overflow if not handled properly...',
          maxLines: 2,
          overflow: TextOverflow.ellipsis,
        ),
      ],
    );
  }
}

// 11. Image - Display images
class ImageExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // Network image
        Image.network(
          'https://example.com/image.jpg',
          width: 200,
          height: 200,
          fit: BoxFit.cover,
        ),
        // Asset image
        Image.asset('assets/logo.png'),
        // Icon
        Icon(Icons.home, size: 50, color: Colors.blue),
      ],
    );
  }
}

// 12. Card - Material card
class CardExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Card(
      elevation: 4,
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(12),
      ),
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            Text('Card Title'),
            Text('Card content goes here'),
          ],
        ),
      ),
    );
  }
}

// 13. ListTile - List item
class ListTileExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ListTile(
      leading: CircleAvatar(
        child: Icon(Icons.person),
      ),
      title: Text('John Doe'),
      subtitle: Text('Software Engineer'),
      trailing: Icon(Icons.arrow_forward),
      onTap: () {},
    );
  }
}
```

### Interactive Widgets

```dart
// 14. ElevatedButton - Material button
class ButtonExamples extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        ElevatedButton(
          onPressed: () {},
          child: Text('Elevated'),
        ),
        TextButton(
          onPressed: () {},
          child: Text('Text Button'),
        ),
        OutlinedButton(
          onPressed: () {},
          child: Text('Outlined'),
        ),
        IconButton(
          icon: Icon(Icons.favorite),
          onPressed: () {},
        ),
        FloatingActionButton(
          onPressed: () {},
          child: Icon(Icons.add),
        ),
      ],
    );
  }
}

// 15. TextField - Text input
class TextFieldExample extends StatefulWidget {
  @override
  State<TextFieldExample> createState() => _TextFieldExampleState();
}

class _TextFieldExampleState extends State<TextFieldExample> {
  final _controller = TextEditingController();
  
  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
  
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        TextField(
          controller: _controller,
          decoration: InputDecoration(
            labelText: 'Enter name',
            hintText: 'John Doe',
            prefixIcon: Icon(Icons.person),
            border: OutlineInputBorder(),
          ),
        ),
        TextField(
          obscureText: true,
          decoration: InputDecoration(
            labelText: 'Password',
            suffixIcon: Icon(Icons.visibility),
          ),
        ),
      ],
    );
  }
}

// 16. Checkbox, Radio, Switch
class SelectionWidgets extends StatefulWidget {
  @override
  State<SelectionWidgets> createState() => _SelectionWidgetsState();
}

class _SelectionWidgetsState extends State<SelectionWidgets> {
  bool _isChecked = false;
  int _radioValue = 1;
  bool _isSwitched = false;
  
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        CheckboxListTile(
          title: Text('Checkbox'),
          value: _isChecked,
          onChanged: (value) => setState(() => _isChecked = value ?? false),
        ),
        RadioListTile<int>(
          title: Text('Option 1'),
          value: 1,
          groupValue: _radioValue,
          onChanged: (value) => setState(() => _radioValue = value ?? 1),
        ),
        SwitchListTile(
          title: Text('Switch'),
          value: _isSwitched,
          onChanged: (value) => setState(() => _isSwitched = value),
        ),
      ],
    );
  }
}
```

### Scrollable Widgets

```dart
// 17. ListView - Scrollable list
class ListViewExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ListView(
      children: [
        ListTile(title: Text('Item 1')),
        ListTile(title: Text('Item 2')),
        ListTile(title: Text('Item 3')),
      ],
    );
  }
}

// 18. ListView.builder - Efficient list
class ListViewBuilderExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      itemCount: 100,
      itemBuilder: (context, index) {
        return ListTile(
          title: Text('Item $index'),
        );
      },
    );
  }
}

// 19. GridView - Grid layout
class GridViewExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return GridView.builder(
      gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: 2,
        crossAxisSpacing: 10,
        mainAxisSpacing: 10,
      ),
      itemCount: 20,
      itemBuilder: (context, index) {
        return Container(
          color: Colors.blue,
          child: Center(child: Text('Item $index')),
        );
      },
    );
  }
}

// 20. SingleChildScrollView - Scrollable single child
class SingleChildScrollViewExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return SingleChildScrollView(
      child: Column(
        children: List.generate(
          20,
          (index) => Container(
            height: 100,
            margin: EdgeInsets.all(8),
            color: Colors.blue,
            child: Center(child: Text('Item $index')),
          ),
        ),
      ),
    );
  }
}
```

### Structural Widgets

```dart
// 21. Scaffold - App structure
class ScaffoldExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Scaffold'),
        actions: [
          IconButton(icon: Icon(Icons.search), onPressed: () {}),
          IconButton(icon: Icon(Icons.more_vert), onPressed: () {}),
        ],
      ),
      body: Center(child: Text('Body')),
      floatingActionButton: FloatingActionButton(
        onPressed: () {},
        child: Icon(Icons.add),
      ),
      drawer: Drawer(
        child: ListView(
          children: [
            DrawerHeader(child: Text('Header')),
            ListTile(title: Text('Item 1')),
          ],
        ),
      ),
      bottomNavigationBar: BottomNavigationBar(
        items: [
          BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Home'),
          BottomNavigationBarItem(icon: Icon(Icons.person), label: 'Profile'),
        ],
      ),
    );
  }
}

// 22. AppBar - Top app bar
class AppBarExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('AppBar'),
        leading: IconButton(
          icon: Icon(Icons.menu),
          onPressed: () {},
        ),
        actions: [
          IconButton(icon: Icon(Icons.search), onPressed: () {}),
        ],
        backgroundColor: Colors.blue,
        elevation: 4,
      ),
    );
  }
}
```

### Dialog & Overlay Widgets

```dart
// 23. AlertDialog
class DialogExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () {
        showDialog(
          context: context,
          builder: (context) => AlertDialog(
            title: Text('Alert'),
            content: Text('This is an alert dialog'),
            actions: [
              TextButton(
                onPressed: () => Navigator.pop(context),
                child: Text('Cancel'),
              ),
              TextButton(
                onPressed: () => Navigator.pop(context),
                child: Text('OK'),
              ),
            ],
          ),
        );
      },
      child: Text('Show Dialog'),
    );
  }
}

// 24. SnackBar
class SnackBarExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(
            content: Text('This is a snackbar'),
            action: SnackBarAction(
              label: 'Undo',
              onPressed: () {},
            ),
          ),
        );
      },
      child: Text('Show SnackBar'),
    );
  }
}
```

### Widget Cheat Sheet

| Widget | Purpose | Example Use |
|--------|---------|-------------|
| Container | Box with styling | Cards, backgrounds |
| Row/Column | Linear layout | Forms, lists |
| Stack | Overlay widgets | Badges, image overlays |
| ListView | Scrollable list | Messages, feeds |
| GridView | Grid layout | Photo gallery |
| Text | Display text | Labels, paragraphs |
| Image | Display images | Photos, icons |
| TextField | Text input | Forms, search |
| ElevatedButton | Action button | Submit, save |
| Card | Material card | Product cards |
| Scaffold | App structure | Full screens |
| AppBar | Top bar | Navigation, title |

---

## Q9: Custom Widgets Best Practices

**Level:** Mid  
**Category:** Best Practices  
**Difficulty:** ⭐⭐⭐

### The Question
What are the best practices for creating custom widgets?

### Answer

### 1. Extract Widgets for Reusability

```dart
// ❌ BAD: Everything in one widget
class BadExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          // Product card 1
          Card(
            child: Column(
              children: [
                Image.network('url1'),
                Text('Product 1'),
                Text('\$99'),
                ElevatedButton(onPressed: () {}, child: Text('Buy')),
              ],
            ),
          ),
          // Product card 2 - Duplicated!
          Card(
            child: Column(
              children: [
                Image.network('url2'),
                Text('Product 2'),
                Text('\$149'),
                ElevatedButton(onPressed: () {}, child: Text('Buy')),
              ],
            ),
          ),
        ],
      ),
    );
  }
}

// ✅ GOOD: Extract reusable widget
class GoodExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          ProductCard(name: 'Product 1', price: 99, imageUrl: 'url1'),
          ProductCard(name: 'Product 2', price: 149, imageUrl: 'url2'),
        ],
      ),
    );
  }
}

class ProductCard extends StatelessWidget {
  final String name;
  final double price;
  final String imageUrl;
  
  const ProductCard({
    super.key,
    required this.name,
    required this.price,
    required this.imageUrl,
  });
  
  @override
  Widget build(BuildContext context) {
    return Card(
      child: Column(
        children: [
          Image.network(imageUrl),
          Text(name),
          Text('\$$price'),
          ElevatedButton(onPressed: () {}, child: const Text('Buy')),
        ],
      ),
    );
  }
}
```

### 2. Use const Constructors

```dart
// ✅ GOOD: const constructor
class GoodWidget extends StatelessWidget {
  final String title;
  final IconData icon;
  
  const GoodWidget({
    super.key,
    required this.title,
    required this.icon,
  });
  
  @override
  Widget build(BuildContext context) {
    return Row(
      children: [
        Icon(icon),
        const SizedBox(width: 8),
        Text(title),
      ],
    );
  }
}

// Usage with const
const GoodWidget(title: 'Home', icon: Icons.home)
```

### 3. Keep Widgets Small and Focused

```dart
// ❌ BAD: God widget doing everything
class BadGodWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // Header (50 lines)
        Container(
          /* ... */
        ),
        // Body (100 lines)
        ListView(
          /* ... */
        ),
        // Footer (50 lines)
        Container(
          /* ... */
        ),
      ],
    );
  }
}

// ✅ GOOD: Split into focused widgets
class GoodScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        const ScreenHeader(),
        const Expanded(child: ScreenBody()),
        const ScreenFooter(),
      ],
    );
  }
}

class ScreenHeader extends StatelessWidget {
  const ScreenHeader({super.key});
  @override
  Widget build(BuildContext context) => Container(/* ... */);
}

class ScreenBody extends StatelessWidget {
  const ScreenBody({super.key});
  @override
  Widget build(BuildContext context) => ListView(/* ... */);
}

class ScreenFooter extends StatelessWidget {
  const ScreenFooter({super.key});
  @override
  Widget build(BuildContext context) => Container(/* ... */);
}
```

### 4. Proper Naming Conventions

```dart
// ✅ GOOD: Descriptive names
class UserProfileCard extends StatelessWidget { }
class ProductListItem extends StatelessWidget { }
class PrimaryActionButton extends StatelessWidget { }

// ❌ BAD: Generic names
class MyWidget extends StatelessWidget { }
class Widget1 extends StatelessWidget { }
class Thing extends StatelessWidget { }
```

### 5. Handle Edge Cases

```dart
class RobustWidget extends StatelessWidget {
  final String? title;
  final List<String>? items;
  
  const RobustWidget({super.key, this.title, this.items});
  
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // Handle null
        if (title != null) Text(title!),
        
        // Handle empty list
        if (items == null || items!.isEmpty)
          const Text('No items')
        else
          ...items!.map((item) => Text(item)),
      ],
    );
  }
}
```

### 6. Use Composition Over Inheritance

```dart
// ❌ BAD: Inheritance (anti-pattern in Flutter)
class BaseButton extends StatelessWidget {
  final VoidCallback onPressed;
  const BaseButton({super.key, required this.onPressed});
  
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(onPressed: onPressed, child: Text('Button'));
  }
}

class PrimaryButton extends BaseButton {
  const PrimaryButton({super.key, required super.onPressed});
  // Trying to customize through inheritance - hard to maintain
}

// ✅ GOOD: Composition
class CustomButton extends StatelessWidget {
  final String label;
  final VoidCallback onPressed;
  final Color? color;
  final IconData? icon;
  
  const CustomButton({
    super.key,
    required this.label,
    required this.onPressed,
    this.color,
    this.icon,
  });
  
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      style: ElevatedButton.styleFrom(
        backgroundColor: color,
      ),
      onPressed: onPressed,
      child: Row(
        mainAxisSize: MainAxisSize.min,
        children: [
          if (icon != null) ..[
            Icon(icon),
            const SizedBox(width: 8),
          ],
          Text(label),
        ],
      ),
    );
  }
}
```

### 7. Document Complex Widgets

```dart
/// A custom card widget that displays user information.
/// 
/// This widget shows a user's avatar, name, email, and an optional
/// action button. It handles loading states and error cases.
/// 
/// Example:
/// ```dart
/// UserInfoCard(
///   name: 'John Doe',
///   email: 'john@example.com',
///   avatarUrl: 'https://...',
///   onTap: () => print('Tapped'),
/// )
/// ```
class UserInfoCard extends StatelessWidget {
  /// The user's display name
  final String name;
  
  /// The user's email address
  final String email;
  
  /// URL to the user's avatar image
  final String? avatarUrl;
  
  /// Callback when card is tapped
  final VoidCallback? onTap;
  
  const UserInfoCard({
    super.key,
    required this.name,
    required this.email,
    this.avatarUrl,
    this.onTap,
  });
  
  @override
  Widget build(BuildContext context) {
    return Card(
      child: ListTile(
        leading: CircleAvatar(
          backgroundImage: avatarUrl != null ? NetworkImage(avatarUrl!) : null,
          child: avatarUrl == null ? const Icon(Icons.person) : null,
        ),
        title: Text(name),
        subtitle: Text(email),
        onTap: onTap,
      ),
    );
  }
}
```

### 8. Avoid Heavy Computations in build()

```dart
// ❌ BAD: Heavy computation in build
class BadWidget extends StatelessWidget {
  final List<int> numbers;
  
  const BadWidget({super.key, required this.numbers});
  
  @override
  Widget build(BuildContext context) {
    // ❌ Computed every rebuild!
    final sum = numbers.reduce((a, b) => a + b);
    final average = sum / numbers.length;
    
    return Text('Average: $average');
  }
}

// ✅ GOOD: Pre-compute or use computed properties
class GoodWidget extends StatelessWidget {
  final double average; // Pre-computed
  
  const GoodWidget({super.key, required this.average});
  
  @override
  Widget build(BuildContext context) {
    return Text('Average: $average');
  }
}
```

### 9. Use Callbacks for Communication

```dart
class CustomListItem extends StatelessWidget {
  final String title;
  final VoidCallback onDelete;
  final VoidCallback onEdit;
  final ValueChanged<bool> onToggle;
  
  const CustomListItem({
    super.key,
    required this.title,
    required this.onDelete,
    required this.onEdit,
    required this.onToggle,
  });
  
  @override
  Widget build(BuildContext context) {
    return ListTile(
      title: Text(title),
      trailing: Row(
        mainAxisSize: MainAxisSize.min,
        children: [
          IconButton(
            icon: const Icon(Icons.edit),
            onPressed: onEdit,
          ),
          IconButton(
            icon: const Icon(Icons.delete),
            onPressed: onDelete,
          ),
          Switch(
            value: true,
            onChanged: onToggle,
          ),
        ],
      ),
    );
  }
}
```

### 10. Create Widget Libraries

```dart
// lib/widgets/buttons.dart
class PrimaryButton extends StatelessWidget {
  final String label;
  final VoidCallback onPressed;
  
  const PrimaryButton({
    super.key,
    required this.label,
    required this.onPressed,
  });
  
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      style: ElevatedButton.styleFrom(
        backgroundColor: Colors.blue,
        padding: const EdgeInsets.symmetric(horizontal: 32, vertical: 16),
      ),
      onPressed: onPressed,
      child: Text(label),
    );
  }
}

class SecondaryButton extends StatelessWidget {
  // ...
}

// lib/widgets/cards.dart
class ProductCard extends StatelessWidget {
  // ...
}

class UserCard extends StatelessWidget {
  // ...
}
```

### Best Practices Checklist

- [ ] Widget has a single responsibility
- [ ] Properties are final and use const constructor
- [ ] Widget name is descriptive
- [ ] No heavy computations in build()
- [ ] Handles null and edge cases
- [ ] Uses callbacks for communication
- [ ] Complex widgets are documented
- [ ] Reusable widgets are extracted
- [ ] Proper widget size (not too big)
- [ ] Follows Flutter style guide

---

## Q10: Widget Performance

**Level:** Mid-Senior  
**Category:** Performance  
**Difficulty:** ⭐⭐⭐⭐

### The Question
How do you optimize widget performance? What are common performance issues?

### Answer

### Performance Principles

1. **Minimize rebuilds** - Only rebuild what changed
2. **Use const** - Skip unnecessary rebuilds
3. **Optimize lists** - Use builder patterns
4. **Cache expensive operations** - Don't recompute

### 1. Use const Everywhere Possible

```dart
class ConstOptimization extends StatefulWidget {
  @override
  State<ConstOptimization> createState() => _ConstOptimizationState();
}

class _ConstOptimizationState extends State<ConstOptimization> {
  int _counter = 0;
  
  @override
  Widget build(BuildContext context) {
    print('Parent rebuilt');
    
    return Column(
      children: [
        // ✅ These won't rebuild (const)
        const Text('Static Header'),
        const Icon(Icons.star),
        const Divider(),
        
        // ❌ This rebuilds
        Text('Counter: $_counter'),
        
        // ✅ This won't rebuild (const)
        const Padding(
          padding: EdgeInsets.all(16),
          child: Text('Static Footer'),
        ),
        
        ElevatedButton(
          onPressed: () => setState(() => _counter++),
          child: const Text('Increment'),
        ),
      ],
    );
  }
}
```

### 2. Extract Widgets to Limit Rebuild Scope

```dart
// ❌ BAD: Entire screen rebuilds
class BadPerformance extends StatefulWidget {
  @override
  State<BadPerformance> createState() => _BadPerformanceState();
}

class _BadPerformanceState extends State<BadPerformance> {
  int _counter = 0;
  
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        ExpensiveHeader(), // Rebuilds!
        Text('Counter: $_counter'),
        ExpensiveFooter(), // Rebuilds!
        ElevatedButton(
          onPressed: () => setState(() => _counter++),
          child: Text('Increment'),
        ),
      ],
    );
  }
}

// ✅ GOOD: Only counter widget rebuilds
class GoodPerformance extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        const ExpensiveHeader(), // Never rebuilds
        const CounterWidget(), // Only this rebuilds
        const ExpensiveFooter(), // Never rebuilds
      ],
    );
  }
}

class CounterWidget extends StatefulWidget {
  const CounterWidget({super.key});
  
  @override
  State<CounterWidget> createState() => _CounterWidgetState();
}

class _CounterWidgetState extends State<CounterWidget> {
  int _counter = 0;
  
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Counter: $_counter'),
        ElevatedButton(
          onPressed: () => setState(() => _counter++),
          child: const Text('Increment'),
        ),
      ],
    );
  }
}

class ExpensiveHeader extends StatelessWidget {
  const ExpensiveHeader({super.key});
  
  @override
  Widget build(BuildContext context) {
    print('ExpensiveHeader built');
    // Expensive widget
    return Container(height: 200, color: Colors.blue);
  }
}

class ExpensiveFooter extends StatelessWidget {
  const ExpensiveFooter({super.key});
  
  @override
  Widget build(BuildContext context) {
    print('ExpensiveFooter built');
    return Container(height: 100, color: Colors.grey);
  }
}
```

### 3. Use ListView.builder for Long Lists

```dart
// ❌ BAD: All items created at once
class BadList extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ListView(
      children: List.generate(
        10000, // 10,000 widgets created immediately!
        (index) => ListTile(title: Text('Item $index')),
      ),
    );
  }
}

// ✅ GOOD: Items created on demand
class GoodList extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      itemCount: 10000,
      itemBuilder: (context, index) {
        // Only visible items created
        return ListTile(title: Text('Item $index'));
      },
    );
  }
}
```

### 4. RepaintBoundary for Complex Animations

```dart
class RepaintBoundaryExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // ✅ Animated widget isolated
        RepaintBoundary(
          child: AnimatedWidget(),
        ),
        // Static content doesn't repaint
        const StaticContent(),
      ],
    );
  }
}

class AnimatedWidget extends StatefulWidget {
  @override
  State<AnimatedWidget> createState() => _AnimatedWidgetState();
}

class _AnimatedWidgetState extends State<AnimatedWidget>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  
  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(seconds: 2),
      vsync: this,
    )..repeat();
  }
  
  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
  
  @override
  Widget build(BuildContext context) {
    return AnimatedBuilder(
      animation: _controller,
      builder: (context, child) {
        return Transform.rotate(
          angle: _controller.value * 2 * 3.14159,
          child: Container(
            width: 100,
            height: 100,
            color: Colors.blue,
          ),
        );
      },
    );
  }
}

class StaticContent extends StatelessWidget {
  const StaticContent({super.key});
  
  @override
  Widget build(BuildContext context) {
    return Column(
      children: List.generate(
        100,
        (index) => Text('Static item $index'),
      ),
    );
  }
}
```

### 5. Cache Expensive Computations

```dart
class CachingExample extends StatefulWidget {
  final List<int> numbers;
  
  const CachingExample({super.key, required this.numbers});
  
  @override
  State<CachingExample> createState() => _CachingExampleState();
}

class _CachingExampleState extends State<CachingExample> {
  late final double _average; // Cached
  int _counter = 0;
  
  @override
  void initState() {
    super.initState();
    // ✅ Compute once
    _average = widget.numbers.reduce((a, b) => a + b) / widget.numbers.length;
  }
  
  @override
  Widget build(BuildContext context) {
    // ✅ Uses cached value
    return Column(
      children: [
        Text('Average: $_average'),
        Text('Counter: $_counter'),
        ElevatedButton(
          onPressed: () => setState(() => _counter++),
          child: const Text('Increment'),
        ),
      ],
    );
  }
}
```

### 6. Optimize Images

```dart
class OptimizedImages extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // ✅ Cached network image
        Image.network(
          'https://example.com/image.jpg',
          cacheWidth: 200, // Resize for efficiency
          cacheHeight: 200,
          loadingBuilder: (context, child, progress) {
            if (progress == null) return child;
            return CircularProgressIndicator();
          },
        ),
        
        // ✅ Optimized asset
        Image.asset(
          'assets/logo.png',
          width: 100,
          height: 100,
          fit: BoxFit.cover,
        ),
      ],
    );
  }
}
```

### 7. Use Keys Wisely

```dart
class KeyOptimization extends StatefulWidget {
  @override
  State<KeyOptimization> createState() => _KeyOptimizationState();
}

class _KeyOptimizationState extends State<KeyOptimization> {
  List<Widget> items = [
    StatefulItem(key: ValueKey(1), color: Colors.red),
    StatefulItem(key: ValueKey(2), color: Colors.blue),
    StatefulItem(key: ValueKey(3), color: Colors.green),
  ];
  
  void _reorder() {
    setState(() {
      // ✅ Keys preserve state during reorder
      final item = items.removeAt(0);
      items.add(item);
    });
  }
  
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        ...items,
        ElevatedButton(
          onPressed: _reorder,
          child: const Text('Reorder'),
        ),
      ],
    );
  }
}

class StatefulItem extends StatefulWidget {
  final Color color;
  
  const StatefulItem({super.key, required this.color});
  
  @override
  State<StatefulItem> createState() => _StatefulItemState();
}

class _StatefulItemState extends State<StatefulItem> {
  int _counter = 0;
  
  @override
  Widget build(BuildContext context) {
    return Container(
      height: 100,
      color: widget.color,
      child: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Count: $_counter', style: TextStyle(color: Colors.white)),
            ElevatedButton(
              onPressed: () => setState(() => _counter++),
              child: const Text('Increment'),
            ),
          ],
        ),
      ),
    );
  }
}
```

### Performance Monitoring

```dart
import 'package:flutter/scheduler.dart';

void monitorPerformance() {
  // Track frame rendering time
  SchedulerBinding.instance.addTimingsCallback((timings) {
    for (final timing in timings) {
      print('Frame took: ${timing.totalSpan.inMilliseconds}ms');
      if (timing.totalSpan.inMilliseconds > 16) {
        print('SLOW FRAME DETECTED!');
      }
    }
  });
}
```

### Performance Checklist

- [ ] Using const constructors everywhere possible
- [ ] Widgets extracted to limit rebuild scope
- [ ] ListView.builder for long lists
- [ ] RepaintBoundary for complex animations
- [ ] Expensive computations cached
- [ ] Images optimized (size, caching)
- [ ] Keys used for stateful widget collections
- [ ] No heavy operations in build()
- [ ] DevTools used to identify performance issues

### Common Performance Killers

1. ❌ Not using const
2. ❌ setState() in parent rebuilds entire tree
3. ❌ ListView without builder for long lists
4. ❌ Large unoptimized images
5. ❌ Heavy computations in build()
6. ❌ No keys in dynamic lists
7. ❌ Unnecessary rebuilds

---

## Summary

You've completed the **Widgets & UI** section! Key takeaways:

1. **Widgets** are the building blocks (immutable configuration)
2. **StatelessWidget** for static, **StatefulWidget** for dynamic
3. **Lifecycle** methods manage widget behavior
4. **BuildContext** locates widgets in the tree
5. **Keys** help Flutter identify widgets during changes
6. **const** improves performance significantly
7. **Three trees** (Widget/Element/RenderObject) work together
8. **Common widgets** cover most UI needs
9. **Custom widgets** should be small, focused, reusable
10. **Performance** optimization is crucial for smooth UIs

**Next:** Move on to [Dart Basics](../01-fundamentals/dart-basics.md) or [Navigation](../01-fundamentals/navigation.md)!

---

*Return to [Part 1](widgets.md) • [Part 2](widgets-part2.md)*