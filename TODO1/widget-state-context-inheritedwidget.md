> * 原文地址：[Widget - State - Context - InheritedWidget](https://www.didierboelens.com/2018/06/widget---state---context---inheritedwidget/)
> * 原文作者：[www.didierboelens.com](https://www.didierboelens.com)
> * 译文出自：[掘金翻译计划](https://github.com/xitu/gold-miner)
> * 本文永久链接：[https://github.com/xitu/gold-miner/blob/master/TODO1/widget-state-context-inheritedwidget.md](https://github.com/xitu/gold-miner/blob/master/TODO1/widget-state-context-inheritedwidget.md)
> * 译者：
> * 校对者：

# Widget - State - Context - InheritedWidget

This article covers the important notions of Widget, State, Context and InheritedWidget in Flutter Applications. Special attention is paid on the InheritedWidget which is one of the most important and less documented widgets.

Difficulty: _Beginner_

## Foreword

The notions of **Widget**, **State** and **Context** in Flutter are ones of the most important concepts that every Flutter developer needs to fully understand.

However, the documentation is huge and this concept is not always clearly explained.

I will explain these notions with my own words and shortcuts, knowing that this might risk to shock some purists, but the real objective of this article to try to clarify the following topics:

*   difference of Stateful and Stateless widgets
*   what is a Context
*   what is a State and how to use it
*   relationship between a context and its state object
*   InheritedWidget and the way to propagate the information inside a Widgets tree
*   notion of rebuild

This article is also available on [Medium - Flutter Community](https://medium.com/flutter-community/widget-state-buildcontext-inheritedwidget-898d671b7956).

## Part 1: Concepts

### Notion of Widget

In _Flutter_, almost everything is a **Widget**.

> Think of a _Widget_ as a visual component (or a component that interacts with the visual aspect of an application).

When you need to build anything that directly or indirectly is in relation with the layout, you are using **Widgets**.

### Notion of Widgets tree

**Widgets** are organized in tree structure(s).

A widget that contains other widgets is called **parent Widget** (or _Widget container_). Widgets which are contained in a _parent Widget_ are called **children Widgets**.

Let’s illustrate this with the base application which is automatically generated by _Flutter_. Here is the simplified code, limited to the **build** method:

```
@override
Widget build(BuildContext){
    return new Scaffold(
      appBar: new AppBar(
        title: new Text(widget.title),
      ),
      body: new Center(
        child: new Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            new Text(
              'You have pushed the button this many times:',
            ),
            new Text(
              '$_counter',
              style: Theme.of(context).textTheme.display1,
            ),
          ],
        ),
      ),
      floatingActionButton: new FloatingActionButton(
        onPressed: _incrementCounter,
        tooltip: 'Increment',
        child: new Icon(Icons.add),
      ),
    );
}
```

If we now consider this basic example, we obtain the following Widgets tree structure (_limited the list of Widgets present in the code_):

![state diagram basic](https://www.didierboelens.com/images/state_basic_tree.png)

### Notion of Context

Another important notion is the **Context**.

A _context_ is nothing else but a reference to the location of a Widget within the tree structure of all the Widgets which are built.

> In short, think of a _context_ as the part of Widgets tree where the Widget is attached to this tree.

A _context_ only belongs to **one** widget.

If a widget ‘A’ has children widgets, the _context_ of widget ‘A’ will become the _parent context_ of the direct _children contexts_.

Reading this, it is clear that **contexts are chained** and are composing a tree of contexts (parent-children relationship).

If we now try to illustrate the notion of **Context** in the previous diagram, we obtain (_still as a very simplified view_) where each color represents a **context** (_except the MyApp one, which is different_):

![state diagram basic context](https://www.didierboelens.com/images/state_basic_context_tree.png)

> **Context Visibility** (_Simplified statement_):
> 
> _Something_ is only visible within its own context or in the context of its parent(s) context.

From this statement we can derive that from a child context, it is easily possible to find an **ancestor** (= parent) Widget.

> An example is, considering the Scaffold > Center > Column > Text: context.ancestorWidgetOfExactType(Scaffold) => returns the first Scaffold by going up to tree structure from the Text context.

From a parent context, it is also possible to find a **descendant** (= child) Widget but it is not advised to do so (_we will discuss this later_).

### Types of Widgets

Widgets are of 2 types:

#### Stateless Widget

Some of this visual components do not depend on anything else but their own configuration information, which is provided **at time of building it** by its direct parent.

In other words, these Widgets will not have to care about any _variation_, once created.

These Widgets are called **Stateless Widgets**.

Typical examples of such Widgets could be Text, Row, Column, Container… where during the building time, we simply pass some parameters to them.

_Parameters_ might be anything from a decoration, dimensions, or even other widget(s). It does not matter. The only thing which is important is that this configuration, once applied, will not change before the next building process.

> A stateless widget can only be drawn only once when the Widget is loaded/built, which means that that Widget cannot be redrawn based on any events or user actions.

##### Stateless Widget lifecycle

Here is a typical structure of the code related to a _Stateless Widget_.

As you may see, we can pass some additional parameters to its constructor. However, bear in mind that these parameters will _NOT_ change (mutate) at a later stage and have to be only used _as is_.

```
class MyStatelessWidget extends StatelessWidget {

	MyStatelessWidget({
		Key key,
		this.parameter,
	}): super(key:key);

	final parameter;

	@override
	Widget build(BuildContext context){
		return new ...
	}
}
```

Even if there is another method that could be overridden (_createElement_), the latter is barely never overridden. The only one that **needs** to be overridden is **build**.

The lifecycle of such Stateless widget is straightforward:

*   initialization
*   rendering via build()

#### Stateful Widget

Some other Widgets will handle some _inner data_ that will change during the Widget’s lifetime. This _data_ hence becomes **dynamic**.

The set of _data_ held by this Widget and which may vary during the lifetime of this Widget is called a **State**.

These Widgets are called **Stateful Widgets**.

An example of such Widget might be a list of Checkboxes that the user can select or a Button which is disabled depending on a condition.

### Notion of State

A **State** defines the “_behavioural_” part of a _StatefulWidget_ instance.

It holds information aimed at **interacting / interferring** with the Widget in terms of:

*   behaviour
*   layout

> Any changes which is applied to a _State_ forces the Widget to **rebuild**.

## Relation between a State and a Context

For _Stateful widgets_, a _State_ is associated with a _Context_. This association is _permanent_ and the _State_ object will never change its _context_.

Even if the Widget Context can be moved around the tree structure, the _State_ will remain associated with that _context_.

When a _State_ is associated with a _Context_, the _State_ is considered as **mounted**.

> **HYPER IMPORTANT**:
> 
> As a _State object_ is associated with a _context_, this means that the _State object_ is **NOT** (directly) accessible through _another context_ ! (we will further discuss this in a few moment).

* * *

## Stateful Widget lifecycle

Now that the base concepts have been introduced, it is time to dive a bit deeper…

Here is a typical structure of the code related to a _Stateful Widget_.

As the main objective of this article is to explain the notion of _State_ in terms of “variable” data, I will intentionally skip any explanation related to some Stateful Widget _overridable_ methods, which do not specifically relate to this. These overridable methods are _didUpdateWidget, deactivate, reassemble_. These will be discussed in a next article.

```
class MyStatefulWidget extends StatefulWidget {

	MyStatefulWidget({
		Key key,
		this.parameter,
	}): super(key: key);
	
	final parameter;
	
	@override
	_MyStatefulWidgetState createState() => new _MyStatefulWidgetState();
}

class _MyStatefulWidgetState extends State<MyStatefulWidget> {

	@override
	void initState(){
		super.initState();
		
		// Additional initialization of the State
	}
	
	@override
	void didChangeDependencies(){
		super.didChangeDependencies();
		
		// Additional code
	}
	
	@override
	void dispose(){
		// Additional disposal code
		
		super.dispose();
	}
	
	@override
	Widget build(BuildContext context){
		return new ...
	}
}
```

The following diagram shows (_a simplified version of_) the sequence of actions/calls related to the creation of a Stateful Widget. At the right side of the diagram, you will notice the inner status of the _State_ object during the flow. You will also see the moment when the context is associated with the state, and thus becomes available (_mounted_).

![state diagram](https://www.didierboelens.com/images/state_diagram.png)

So let’s explain it with some additional details:

#### initState()

The _initState()_ method is the very first method (after the constructor) to be called once the State object has been created. This method is to be overridden when you need to perform additional initializations. Typical initializations are related to animations, controllers… If you override this method, you need to call the **super.initState()** method and normally at first place.

In this method, a _context_ is available but you **cannot** really use it yet since the framework has not yet fully associated the state with it.

Once the _initState()_ method is complete, the State object is now initialized and the context, available.

This method will not be invoked anymore during the lifetime of this State object.

#### didChangeDependencies()

The _didChangeDependencies()_ method is the second method to be invoked.

At this stage, as the _context_ is available, you may use it.

This method is usually overridden if your Widget is linked to an **InheritedWidget** and/or if you need to initialize some _listeners_ (based on the _context_).

Note that if your widget is linked to an _InheritedWidget_, this method will be called each time this Widget will be rebuilt.

If you override this method, you should invoke the _super.didChangeDependencies()_ at first place.

#### build()

The _build(BuildContext context)_ method is called after the _didChangeDependencies()_ (and _didUpdateWidget_).

This is the place where you build your widget (and potentially any sub-tree).

This method will be called **each time your State object changes** (or when an InheritedWidget needs to notify the “_registered_” widgets) !!

In order to force a rebuild, you may invoke _setState((){…})_ method.

#### dispose()

The _dispose()_ method is called when the widget is discarded.

Override this method if you need to perform some cleanup (e.g. listeners), then invoke the _super.dispose()_ right after.

## Stateless or Stateful Widget?

This is a question that many developers need to ask themselves: _do I need my Widget to be Stateless or Stateful?_

In order to answer this question, ask yourself:

> In the lifetime of my widget, do I need to consider a **variable** that will change and when changed, will force the widget to be **rebuilt**?

If the answer to the question is _yes_, then you need a _Stateful_ widget, otherwise, you need a _Stateless_ widget.

Some examples:

*   a widget to display a list of checkboxes. To display the checkboxes, you need to consider an array of items. Each item is an object with a title and a status. If you click on a checkbox, the corresponding item.status is toggled;
    
    In this case, you need to use a _Stateful_ widget to remember the status of the items to be able to redraw the checkboxes.
    
*   a screen with a Form. The screen allows the user to fill the Widgets of the Form and send the form to the server.
    
    In this case, _unless you need to validate the Form or do any other action before submitting it_, a _Stateless_ widget might be enough.

* * *

## Stateful Widget is made of 2 parts

Remember the structure of a **Stateful** widget? There are 2 parts:

### The Widget main definition

```
class MyStatefulWidget extends StatefulWidget {
    MyStatefulWidget({
		Key key,
		this.color,
	}): super(key: key);
	
	final Color color;

	@override
	_MyStatefulWidgetState createState() => new _MyStatefulWidgetState();
}
```

The first part “_MyStatefulWidget_” is _normally_ the **public** part of the Widget. You instantiate this part when you want to add it to a widget tree. This part does not vary during the lifetime of the Widget but may accept parameters that could be used by its corresponding _State_ instance.

> Note that any variable, defined at the level of this first part of the Widget will _normally_ **NOT** change during its lifetime.

### The Widget State definition

```
class _MyStatefulWidgetState extends State<MyStatefulWidget> {
    ...
	@override
	Widget build(BuildContext context){
	    ...
	}
}
```

The second part _“_MyStatefulWidgetState”_ is the part which **varies** during the lifetime of the Widget and forces this specific instance of the Widget to rebuild each time a modification is applied. The ‘**_**’ character in the beginning of the name makes the class **private** to the .dart file.

If you need to make a reference to this class outside the .dart file, do not use the ‘**_**’ prefix.

The _`_MyStatefulWidgetState`_ class can access any variable which is stored in the _MyStatefulWidget_, using **widget.{name of the variable}**. In this example: _widget.color_

* * *

## Widget unique identity - Key

In Flutter, each Widget is uniquely identified. This unique identity is defined by the framework **at build/rendering time**.

This unique identity corresponds to the optional **Key** parameter. If omitted, Flutter will generate one for you.

In some circumstances, you might need to force this **key**, so that you can access a widget by its key.

To do so, you can use one of the following helpers: _GlobalKey_, _LocalKey_, _UniqueKey_ or _ObjectKey_.

The _GlobalKey_ ensures that the key is unique across the whole application.

To force a unique identity of a Widget:

```
GlobalKey myKey = new GlobalKey();
...
@override
Widget build(BuildContext context){
    return new MyWidget(
        key: myKey
    );
}
```

* * *

## Part 2: How to access the State?

As previously explained, a **State** is linked to **one Context** and a **Context** is linked to an **instance** of a Widget.

### 1. The Widget itself

In theory, the only one which is able to access a _State_ is the **Widget State itself**.

In this case, there is no difficulty. The Widget State class accesses any of its variables.

### 2. A direct child Widget

Sometimes, a parent widget might need to get access to the State of one of its direct children to perform specific tasks.

In this case, to access these direct children _State_, you need to **know** them.

The easiest way to call somebody is via a _name_. In Flutter, each Widget has a unique identity, which is determined at **build/rendering time** by the framework. As shown earlier, you may force the identity of a Widget, using the **key** parameter.

```
...
GlobalKey<MyStatefulWidgetState> myWidgetStateKey = new GlobalKey<MyStatefulWidgetState>();
...
@override
Widget build(BuildContext context){
    return new MyStatefulWidget(
        key: myWidgetStateKey,
        color: Colors.blue,
    );
}
```

Once identified, a _parent_ Widget might access the _State_ of its child via:

> myWidgetStateKey.currentState

Let’s consider a basic example that shows a SnackBar when the user hits a button. As the SnackBar is a child Widget of the Scaffold it is not directly accessible to any other child of the body of the Scaffold (_remember the notion of context and its hierarchy/tree structure ?_). Therefore, the only way to access it, is via the _ScaffoldState_, which exposes a public method to show the SnackBar.

```
class _MyScreenState extends State<MyScreen> {
    /// the unique identity of the Scaffold
    final GlobalKey<ScaffoldState> _scaffoldKey = new GlobalKey<ScaffoldState>();

    @override
    Widget build(BuildContext context){
        return new Scaffold(
            key: _scaffoldKey,
            appBar: new AppBar(
                title: new Text('My Screen'),
            ),
            body: new Center(
                new RaiseButton(
                    child: new Text('Hit me'),
                    onPressed: (){
                        _scaffoldKey.currentState.showSnackBar(
                            new SnackBar(
                                content: new Text('This is the Snackbar...'),
                            )
                        );
                    }
                ),
            ),
        );
    }
}
```

### 3. Ancestor Widget

Suppose that you have a Widget that belongs to a sub-tree of another Widget as shown in the following diagram.

![state child get state](https://www.didierboelens.com/images/state_child_get_state.png)

3 conditions need to be met to make this possible:

#### 1. the “_Widget with State_” (in red) needs to expose its _State_

In order to _expose_ its _State_, the Widget needs to record it at time of creation, as follows:

```
class MyExposingWidget extends StatefulWidget {

   MyExposingWidgetState myState;
	
   @override
   MyExposingWidgetState createState(){
      myState = new MyExposingWidgetState();
      return myState;
   }
}
```

#### 2. the “_Widget State_” needs to expose some getters/setters

In order to let a “_stranger_” to set/get a property of the State, the _Widget State_ needs to authorize the access, through:

*   public property (not recommended)
*   getter / setter

Example:

```
class MyExposingWidgetState extends State<MyExposingWidget>{
   Color _color;
	
   Color get color => _color;
   ...
}
```

#### 3. the “_Widget interested in getting the State_” (in blue) needs to get a reference to the _State_

```
class MyChildWidget extends StatelessWidget {
   @override
   Widget build(BuildContext context){
      final MyExposingWidget widget = context.ancestorWidgetOfExactType(MyExposingWidget);
      final MyExposingWidgetState state = widget?.myState;
		
      return new Container(
         color: state == null ? Colors.blue : state.color,
      );
   }
}
```

This solution is easy to implement but how does the child widget know when it needs to rebuild?

With this solution, it **does not**. It will have to wait for a rebuild to happen to refresh its content, which is not very convenient.

The next section tackles the notion of **Inherited Widget** which gives a solution to this problem.

* * *

## InheritedWidget

In short and with simple words, the **InheritedWidget** allows to efficiently propagate (and share) information down a tree of _widgets_.

The **InheritedWidget** is a special Widget, that you put in the Widgets tree as a parent of another sub-tree. All widgets part of that sub-tree will have to ability to _interact_ with the data which is exposed by that **InheritedWidget**.

### Basics

In order to explain it, let’s consider the following piece of code:

```
class MyInheritedWidget extends InheritedWidget {
   MyInheritedWidget({
      Key key,
      @required Widget child,
      this.data,
   }): super(key: key, child: child);
	
   final data;
	
   static MyInheritedWidget of(BuildContext context) {
      return context.inheritFromWidgetOfExactType(MyInheritedWidget);
   }

   @override
   bool updateShouldNotify(MyInheritedWidget oldWidget) => data != oldWidget.data;
}
```

This code defines a Widget, named “_MyInheritedWidget_”, aimed at “_sharing_” some data across all widgets, part of the child sub-tree.

As mentioned earlier, an **InheritedWidget** needs to be positioned at the top of a widgets tree in order to be able to propagate/share some data, this explains the _@required Widget child_ which is passed to the InheritedWidget base constructor.

The _static MyInheritedWidget of(BuildContext context)_ method, allows all the children widgets to get the instance of the closest _MyInheritedWidget_ which encloses the context (see later).

Finally the _updateShouldNotify_ overridden method is used to tell the _InheritedWidget_ whether notifications will have to be passed to all the children widgets (that registered/subscribed) if a modification be applied to the _data_ (see later).

Therefore, we need to put it at a tree node level as follows:

```
class MyParentWidget... {
   ...
   @override
   Widget build(BuildContext context){
      return new MyInheritedWidget(
         data: counter,
         child: new Row(
            children: <Widget>[
               ...
            ],
         ),
      );
   }
}
```

### How does a child get access to the data of the InheritedWidget?

At time of building a child, the latter will get a reference to the InheritedWidget, as follows:

```
class MyChildWidget... {
   ...
	
   @override
   Widget build(BuildContext context){
      final MyInheritedWidget inheritedWidget = MyInheritedWidget.of(context);
		
      ///
      /// From this moment, the widget can use the data, exposed by the MyInheritedWidget
      /// by calling:  inheritedWidget.data
      ///
      return new Container(
         color: inheritedWidget.data.color,
      );
   }
}
```

### How to make interactions between Widgets?

Consider the following diagram that shows a widgets tree structure.

![inheritedwidget tree](https://www.didierboelens.com/images/inherited_widget_tree_1.png)

In order to illustrate a type of interaction, let’s suppose the following:

*   ‘Widget A’ is a button that adds an item to the shopping cart;
*   ‘Widget B’ is a Text that displays the number of items in the shopping cart;
*   ‘Widget C’ is next to Widget B and is a Text with any text inside;
*   We want the ‘Widget B’ to automatically display the right number of items in the shopping cart, as soon as the ‘Widget A’ is pressed but we do not want ‘Widget C’ to be rebuilt

The **InheritedWidget** is just the right Widget to use for that!

#### Example by the code

Let’s first write the code and explanations will follow:

```
class Item {
   String reference;

   Item(this.reference);
}

class _MyInherited extends InheritedWidget {
  _MyInherited({
    Key key,
    @required Widget child,
    @required this.data,
  }) : super(key: key, child: child);

  final MyInheritedWidgetState data;

  @override
  bool updateShouldNotify(_MyInherited oldWidget) {
    return true;
  }
}

class MyInheritedWidget extends StatefulWidget {
  MyInheritedWidget({
    Key key,
    this.child,
  }): super(key: key);

  final Widget child;

  @override
  MyInheritedWidgetState createState() => new MyInheritedWidgetState();

  static MyInheritedWidgetState of(BuildContext context){
    return (context.inheritFromWidgetOfExactType(_MyInherited) as _MyInherited).data;
  }
}

class MyInheritedWidgetState extends State<MyInheritedWidget>{
  /// List of Items
  List<Item> _items = <Item>[];

  /// Getter (number of items)
  int get itemsCount => _items.length;

  /// Helper method to add an Item
  void addItem(String reference){
    setState((){
      _items.add(new Item(reference));
    });
  }

  @override
  Widget build(BuildContext context){
    return new _MyInherited(
      data: this,
      child: widget.child,
    );
  }
}

class MyTree extends StatefulWidget {
  @override
  _MyTreeState createState() => new _MyTreeState();
}

class _MyTreeState extends State<MyTree> {
  @override
  Widget build(BuildContext context) {
    return new MyInheritedWidget(
      child: new Scaffold(
        appBar: new AppBar(
          title: new Text('Title'),
        ),
        body: new Column(
          children: <Widget>[
            new WidgetA(),
            new Container(
              child: new Row(
                children: <Widget>[
                  new Icon(Icons.shopping_cart),
                  new WidgetB(),
                  new WidgetC(),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
}

class WidgetA extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final MyInheritedWidgetState state = MyInheritedWidget.of(context);
    return new Container(
      child: new RaisedButton(
        child: new Text('Add Item'),
        onPressed: () {
          state.addItem('new item');
        },
      ),
    );
  }
}

class WidgetB extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final MyInheritedWidgetState state = MyInheritedWidget.of(context);
    return new Text('${state.itemsCount}');
  }
}

class WidgetC extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new Text('I am Widget C');
  }
}
```

### Explanations

In this very basic example,

*   _`_MyInherited`_ is an **InheritedWidget** that is recreated each time we add an Item via a click on the button of ‘Widget A’
*   _MyInheritedWidget_ is a Widget with a **State** that contains the list of Items. This _State_ is accessible via the _static MyInheritedWidgetState of(BuildContext context)_
*   _MyInheritedWidgetState_ exposes one getter (_itemsCount_) and one method (_addItem_) so that they will be usable by the widgets, part of the _child_ widgets tree
*   Each time we add an Item to the State, the _MyInheritedWidgetState_ rebuilds
*   _MyTree_ class simply builds a widgets tree, having the _MyInheritedWidget_ as parent of the tree
*   _WidgetA_ is a simple _RaisedButton_ which, when pressed, invokes the _addItem_ method from the **closest** _MyInheritedWidget_
*   _WidgetB_ is a simple _Text_ which displays the number of items, present at the level of the **closest** _MyInheritedWidget_

_How does all this work?_

#### Registration of a Widget for later notifications

When a child Widget invokes the _MyInheritedWidget.of(**context**)_, it makes a call to the following method of MyInheritedWidget, passing its own _context_.

```
static MyInheritedWidgetState of(BuildContext context) {
    return (context.inheritFromWidgetOfExactType(_MyInherited) as _MyInherited).data;
}
```

Internally, on top of simply returning the instance of _MyInheritedWidgetState_, it also subscribes the _consumer_ widget to the changes notifications.

Behind the scene, the simple call to this static method actually does 2 things:

*   the _consumer_ widget is automatically added to the list of **subscribers** that will be **rebuilt** when a modification is applied to the **InheritedWidget** (here _`_MyInherited`_)
*   the _data_ referenced in the _`_MyInherited`_ widget (aka _MyInheritedWidgetState_) is returned to the _consumer_

#### Flow

Since both ‘Widget A’ and ‘Widget B’ have subscribed with the **InheritedWidget** so that if a modification is applied to the _`_MyInherited`_, the flow of operations is the following (simplified version) when the _RaisedButton_ of Widget A is clicked:

1.  A call is made to the _addItem_ method of _MyInheritedWidgetState_
2.  _MyInheritedWidgetState.addItem_ method adds a new Item to the List
3.  _setState()_ is invoked in order to rebuild the _MyInheritedWidget_
4.  A new instance of _`_MyInherited`_ is created with the new content of the List
5.  _`_MyInherited`_ records the new _State_ which is passed in argument (_data_)
6.  As an _InheritedWidget_, it checks whether there is a need to _notify_ the _consumers_ (answer is true)
7.  It iterates the whole list of _consumers_ (here Widget A and Widget B) and requests them to rebuild
8.  As Wiget C is not a _consumer_, it is not rebuilt.

So it works !

However, both Widget A and Widget B are rebuilt while it is useless to rebuild Wiget A since nothing changed for it. How to prevent this from happening?

#### Prevent some Widgets from rebuilding while still accessing the Inherited Widget

The reason why Widget A was also rebuilt comes from the way it accesses the _MyInheritedWidgetState_.

As we saw earlier, the fact of invoking the _context.inheritFromWidgetOfExactType()_ method automatically subscribed the Widget to the list of _consumers_.

The solution to prevent this automatic subscription while still allowing the Widget A access the _MyInheritedWidgetState_ is to change the static method of _MyInheritedWidget_ as follows:

```
static MyInheritedWidgetState of([BuildContext context, bool rebuild = true]){
    return (rebuild ? context.inheritFromWidgetOfExactType(_MyInherited) as _MyInherited
                    : context.ancestorWidgetOfExactType(_MyInherited) as _MyInherited).data;
}
```

By adding a boolean extra parameter…

*   If the _rebuild_ parameter is true (by default), we use the normal approach (and the Widget will be added to the list of subscribers)
*   If the _rebuild_ parameter is false, we still get access to the data **but** without using the _internal implementation_ of the _InheritedWidget_

So, to complete the solution, we also need to slightly update the code of Widget A as follows (we add the false extra parameter):

```
class WidgetA extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final MyInheritedWidgetState state = MyInheritedWidget.of(context, false);
    return new Container(
      child: new RaisedButton(
        child: new Text('Add Item'),
        onPressed: () {
          state.addItem('new item');
        },
      ),
    );
  }
}
```

There it is, Widget A is no longer rebuilt when we press it.

## Special note for Routes, Dialogs…

> Routes, Dialogs contexts are tied to the **Application**.
> 
> This means that even if inside a Screen A you request to display another Screen B (on top of the current, for example), there is _no easy way_ from any of the 2 screens to relate their own contexts.
> 
> The only way for Screen B to know anything about the context of Screen A is to obtain it from Screen A as parameter of Navigator.of(context).push(….)

## Interesting links

*   [Maksim Ryzhikov](https://medium.com/@maksimrv/reactive-app-state-in-flutter-73f829bcf6a7)
*   [Chema Molins](https://medium.com/@chemamolins/is-flutters-inheritedwidget-a-good-fit-to-hold-app-state-2ec5b33d023e)
*   [Official documentation](https://docs.flutter.io/flutter/widgets/InheritedWidget-class.html)
*   [Video from Google I/O 2018](https://www.youtube.com/watch?reload=9&time_continue=7&v=RS36gBEp8OI)
*   [Scoped_Model](https://github.com/brianegan/scoped_model)

## Conclusions

There is still so much to say on these topics… especially on **InheritedWidget**.

In a next article I will introduce the notion of **Notifiers / Listeners** which is also very interesting to use in the context of **State** and the way of conveying data.

So, stay tuned and happy coding.

> 如果发现译文存在错误或其他需要改进的地方，欢迎到 [掘金翻译计划](https://github.com/xitu/gold-miner) 对译文进行修改并 PR，也可获得相应奖励积分。文章开头的 **本文永久链接** 即为本文在 GitHub 上的 MarkDown 链接。

---

> [掘金翻译计划](https://github.com/xitu/gold-miner) 是一个翻译优质互联网技术文章的社区，文章来源为 [掘金](https://juejin.im) 上的英文分享文章。内容覆盖 [Android](https://github.com/xitu/gold-miner#android)、[iOS](https://github.com/xitu/gold-miner#ios)、[前端](https://github.com/xitu/gold-miner#前端)、[后端](https://github.com/xitu/gold-miner#后端)、[区块链](https://github.com/xitu/gold-miner#区块链)、[产品](https://github.com/xitu/gold-miner#产品)、[设计](https://github.com/xitu/gold-miner#设计)、[人工智能](https://github.com/xitu/gold-miner#人工智能)等领域，想要查看更多优质译文请持续关注 [掘金翻译计划](https://github.com/xitu/gold-miner)、[官方微博](http://weibo.com/juejinfanyi)、[知乎专栏](https://zhuanlan.zhihu.com/juejinfanyi)。
