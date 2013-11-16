### Enhancing Objects

Although the previous section’s examples show numeric arrays, often our visualization data consists of Javascript objects instead of simple numbers. That’s especially likely if we get the data via a REST interface, as such interfaces almost always deliver data in Javascript Object Notation (JSON). If we need to enhance or transform objects, Underscore.js has another set of utilities that can help. For the following examples, we can use a simple `pizza` object.

```language-javascript
var pizza = { size: 10, crust: "thin", cheese: true, toppings: ["pepperoni","sausage"]}
```

![](img/obj.png)

#### Keys and Values

Underscore.js includes several methods to work with the keys and values that make up objects. For example, the `keys()` function creates an array consisting solely of an object’s keys.

```language-javascript
> _(pizza).keys()
  ["size", "crust", "cheese", "toppings"]
```

![](img/obj.keys.png)

Similarly, the `values()` function creates an array consisting solely of an object’s values.

```language-javascript
> _(pizza).values()
  [10, "thin", true, ["pepperoni","sausage"]]
```

![](img/obj.values.png)

The `pairs()` function creates a two-dimensional array. Each element of the outer array is itself an array which contains an object’s key and its corresponding value.

```language-javascript
> _(pizza).pairs()
 [ ["size",10], ["crust","thin"], ["cheese",true], ["toppings",["pepperoni","sausage"]] ]
```

![](img/obj.pairs.png)

To reverse this transformation and convert an array into an object, there is the `object()` method.

```language-javascript
> var arr = [ ["size",10], ["crust","thin"], ["cheese",true], ["toppings",["pepperoni","sausage"]] ]
> _(arr).object()
  { size: 10, crust: "thin", cheese: true, toppings: ["pepperoni","sausage"]}
```

Finally, we can swap the roles of keys and values in an object with the `invert()` function.

```language-javascript
> _(pizza).invert()
  {10: "size", thin: "crust", true: "cheese", "pepperoni,sausage": "toppings"}
```

![](img/obj.invert.png)

As the example shows, Underscore.js can even invert an object if the value isn’t a simple type. In this case it takes an array, `["pepperoni","sausage”]` and converts it to a value by joining the individual array elements with commas, creating the key `"pepperoni,sausage"`.

Note also that Javascript requires that all of an object’s keys are unique. That’s not necessarily the case for values. If you have an object in which multiple keys have the same value, then `invert()` only keeps the last of those keys in the inverted object. For example, `_({key1: value, key2: value}).invert()` returns `{value: key2}`.

#### Object Subsets

If the objects that contain our data include attributes that we don’t need, it may be helpful to transform the objects by eliminating the unnecessary attributes. The `pick()` function is Underscore.js’s most straightforward tools for that transformation. We simply pass it a list of attributes that we want to retain.

```language-javascript
> _(pizza).pick("size","crust")
  {size: 10, crust: "thin"}
```

![](img/obj.pick.png)

We can also do the opposite of `pick()` by listing the attributes that we want to delete. Underscore.js keeps all the other attributes in the object.

```language-javascript
> _(pizza).omit("size","crust")
 {cheese: true, toppings: ["pepperoni","sausage"]}
```

![](img/obj.omit.png)

#### Updating Attributes

In addition to extracting parts of an object, we often also need to intelligently update an object’s attributes. An especially common requirement is ensuring that an object includes certain attributes and those attributes have appropriate default values. Underscore.js includes two utilities for this function.

The two utilities, `extend()` and `defaults()` both start with one object it and adjust its properties based on those of other objects. In both cases, if the secondary objects include attributes that the original object lacks, the utility adds those properties to the original. The utilities differ in how they handle properties that are already present in the original. The `extend()` function overrides the original properties with new values, while `defaults()` leaves the original properties unchanged.

```language-javascript
> var standard = { size: 12, crust: "regular", cheese: true }
> var order = { size: 10, crust: "thin", toppings: ["pepperoni","sausage"] };
> _({}).extend(standard, order)
  { size: 10, crust: "thin", cheese: true, toppings: ["pepperoni","sausage"] };
```

![](img/obj.extend.png)

```language-javascript
> var order = { size: 10, crust: "thin", toppings: ["pepperoni","sausage"] };
> var standard = { size: 12, crust: "regular", cheese: true }
> _({}).defaults(order, standard)
  { size: 10, crust: "thin", toppings ["pepperoni","sausage"], cheese: true };
```

![](img/obj.defaults.png)

It’s important to note that both `extend()` and `defaults()` modify the original object directly; they do not make a copy of that object and return the copy. Consider, for example, the following

```language-javascript
> var order = { size: 10, crust: "thin", toppings: ["pepperoni","sausage"] };
> var standard = { size: 12, crust: "regular", cheese: true }
> var pizza = _.extend(standard, order)
  { size: 10, crust: "thin", cheese: true, toppings: ["pepperoni","sausage"] };
```

That code sets the `pizza` variable as you would expect, _but it also sets the `standard` variable to that same object_. More specifically, the code modifies `standard` with the properties from `order`, and then it sets a new variable `pizza` equal to `standard`. The modification of `standard` is probably not intended. If you need to use either `extend()` or `defaults()` in a way that does not modify input parameters, start with an empty object. We can rewrite the code above to avoid modifying `standard`.

```language-javascript
> var order = { size: 10, crust: "thin", toppings: ["pepperoni","sausage"] };
> var standard = { size: 12, crust: "regular", cheese: true }
> var pizza = _.extend({}, standard, order)
  { size: 10, crust: "thin", cheese: true, toppings: ["pepperoni","sausage"] };
```
