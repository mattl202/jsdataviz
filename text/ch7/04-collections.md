### Manipulating Collections

Another set of Underscore.js tools provide simple but powerful functions for manipulating collections. In Underscore.js both arrays and objects are _collections_. We’re most likely dealing with arrays of objects in the context of data visualization, so that’s what the following examples use. It’s worth remembering, though, that the Underscore.js functions work in a natural way with pure arrays or pure objects as well.

Here’s a small data set we can use for the examples below. It contains a few statistics from the 2012 Major League Baseball season.

```language-javascript
var national_league = [
    { name: "Arizona Diamondbacks",  wins: 81, losses:  81, division: "west"    },
    { name: "Atlanta Braves",        wins: 94, losses:  68, division: "east"    },
    { name: "Chicago Cubs",          wins: 61, losses: 101, division: "central" },
    { name: "Cincinnati Reds",       wins: 97, losses:  65, division: "central" },
    { name: "Colorado Rockies",      wins: 64, losses:  98, division: "west"    },
    { name: "Houston Astros",        wins: 55, losses: 107, division: "central" },
    { name: "Los Angeles Dodgers",   wins: 86, losses:  76, division: "west"    },
    { name: "Miami Marlins",         wins: 69, losses:  93, division: "east"    },
    { name: "Milwaukee Brewers",     wins: 83, losses:  79, division: "central" },
    { name: "New York Mets",         wins: 74, losses:  88, division: "east"    },
    { name: "Philadelphia Phillies", wins: 81, losses:  81, division: "east"    },
    { name: "Pittsburgh Pirates",    wins: 79, losses:  83, division: "central" },
    { name: "San Diego Padres",      wins: 76, losses:  86, division: "west"    },
    { name: "San Francisco Giants",  wins: 94, losses:  68, division: "west"    },
    { name: "St. Louis Cardinals",   wins: 88, losses:  74, division: "central" },
    { name: "Washington Nationals",  wins: 98, losses:  64, division: "east"    }
];
```

#### Iteration

In this chapter’s first section we saw some of the pitfalls of traditional Javascript iteration loops as well as the improvements that functional programming can provide. Our Fibonacci example eliminated iteration by using recursion, but many algorithms don’t lend themselves to a recursive implementation. In those cases we can still use a functional programming style, however, by taking advantage of the iteration utilities in Underscore.js

The most basic Underscore utility is `each()`. It executes an arbitrary function on every element in a collection and often serves as a direct functional replacement for the traditional `for (i=0; i<len; i++)` loop.

```language-javascript
> _(national_league).each(function(team) { console.log(team.name); })
  Arizona Diamondbacks
  Atlanta Braves
  ...
  Washington Nationals
```

> Note: If you’re familiar with the jQuery library, you may know that jQuery includes a similar `$.each()` utility. There are two important differences between the Underscore.js and jQuery versions, however. First, the parameters passed to the iterator function differ between the two. Underscore.js passes `(element, index, list)` for arrays and `(value, key, list)` for simple objects, while jQuery passes `(index, value)`. Secondly, at least as of this writing, the Underscore.js implementation can execute much faster than the jQuery version, depending on the browser. (jQuery also includes a `$.map()` function that’s similar to the Underscore.js method.)

The Underscore.js `map()` method iterates through a collection and transforms each element with an arbitrary function. It returns a new collection containing the transformed elements. Here, for example, is how to create an array of all the team’s winning percentages.

```language-javascript
> _(national_league).map(function(team) {
      return Math.round(100*team.wins/(team.wins + team.losses);
  })
  [50, 58, 38, 60, 40, 34, 53, 43, 51, 46, 50, 49, 47, 58, 54, 60]
```

The `reduce()` method converts a collection into a single value by iterating through the collection’s elements. Its parameters include a starting point for that value and an arbitrary function that updates the value for each element in the collection. We can use `reduce()` for example, to calculate how many teams have a winning percentage over 500.

```language-javascript
> _(national_league).reduce(
      function(count, team) { return count + (team.wins > team.losses); },
      0  // starting point for reduced value
  )
  7
```

> Note: If you’ve followed the development of “big data” implementations such as Hadoop or Google’s search, you may know that the fundamental algorithm behind those technologies is _mapReduce_. Although the context differs, the same concepts underlie the `map()` and `reduce()` utilities in Underscore.js.

#### Finding Elements in a Collection

Underscore.js has several methods to help us find elements or sets of elements in a collection. We can, for example, use `find()` to get a team with more than 90 wins.

```language-javascript
> _(national_league).find( function(team) { return team.wins > 90; })
  {name: "Atlanta Braves", wins: 94, losses: 68, division: "east"}
```

The `find()` function returns the first element in the array that meets the criteria.

If we want to find all elements that meet our criteria, the `filter()` function makes that easy.

```language-javascript
> _(national_league).filter( function(team) { return team.wins > 90; })
  [ {name: "Atlanta Braves", wins: 94, losses: 68, division: "east"},
    { name: "Cincinnati Reds", wins: 97, losses: 65, division: "central" },
    { name: "San Francisco Giants", wins: 94, losses: 68, division: "west" },
    { name: "Washington Nationals", wins: 98, losses: 64, division: "east" }
  ]
```

The opposite of the `filter()` function is `reject()`. It returns an array of elements that don’t meet the criteria.

```language-javascript
> _(national_league).reject( function(team) { return team.wins > 90; })
  [ { name: "Arizona Diamondbacks", wins: 81, losses:  81, division: "west" },
    { name: "Chicago Cubs", wins: 61, losses: 101, division: "central" },
    ...
    { name: "St. Louis Cardinals", wins: 88, losses: 74, division: "central" }
  ]
```

If your criteria can be described as a property value, you can use a simpler version of `filter()`, the `where()` function. Instead of an arbitrary function to check for a match, `where()` takes for its parameter a set of properties that must match. We can use it to extract all the teams in the Eastern division.

```language-javascript
> _(national_league).where({division: "east"})
   [ { name: "Atlanta Braves", wins: 94, losses: 68, division: "east" },
     { name: "Miami Marlins", wins: 69, losses: 93, division: "east" },
     { name: "New York Mets", wins: 74, losses: 88, division: "east" },
     { name: "Philadelphia Phillies", wins: 81, losses: 81, division: "east" },
     { name: "Washington Nationals", wins: 98, losses: 64, division: "east" }
  ]
```

The `findWhere()` method combines the functionality of `find()` with the simplicity of `where()`. It returns the first element in a collection with properties that match specific values.

```language-javascript
> _(national_league).where({name: "Atlanta Braves"})
  {name: "Atlanta Braves", wins: 94, losses: 68, division: "east"}
```

Another Underscore.js utility that’s especially handy is `pluck()`. This function creates an array by extracting only the specified property from a collection. We could use it to extract an array of nothing but team names, for example.

```language-javascript
> _(national_league).pluck("team")
  ["Arizona Diamondbacks", "Atlanta Braves", ..., "Washington Nationals"]
```

#### Testing a Collection

Sometimes we don’t necessarily need to transform a collection; we simply want to check some aspect of it. Underscore.js provides several utilities to help with these tests.

The `every()` function tells us whether or not all elements in a collection pass an arbitrary test. We could use it to check if every team in our data set had at least 70 wins.

```language-javascript
> _(national_league).every(function(team) { return team.wins >= 70; })
  false
```

Perhaps we’d like to know if _any_ team had at least 70 wins. In that case the `any()` function provides an answer.

```language-javascript
> _(national_league).any(function(team) { return team.wins >= 70; })
  true
```

Underscore.js also lets us use arbitrary functions to find the maximum and minimum elements in a collection. If our criteria is number of wins, we use `max()` to find the “maximum” team.

```language-javascript
> _(national_league).max(function(team) { return team.wins; })
  {name: "Washington Nationals", wins: 98, losses: 64, division: "east"}
```

Not surprisingly, the `min()` function works the same way.

```language-javascript
> _(national_league).min(function(team) { return team.wins; })
  {name: "Houston Astros", wins: 55, losses: 107, division: "central"}
```

#### Rearranging Collections

To sort a collection, we can use the `sortBy()` method and supply an arbitrary function to provide sortable values. Here’s how to reorder our collection in order of increasing wins.

```language-javascript
> _(national_league).sortBy(function(team) { return team.wins; })
  [ {name: "Houston Astros", wins: 55, losses: 107, division: "central"}
    { name: "Chicago Cubs", wins: 61, losses: 101, division: "central" },
    ...
    {name: "Washington Nationals", wins: 98, losses: 64, division: "east"}
```

We could also reorganize our collection by grouping its elements according to a property. The Underscore.js function that helps in this case is `groupBy()`. One possibility is reorganizing the teams according to their division.

```language-javascript
> _(national_league).groupBy("division")
  {
    { west: 
      { name: "Arizona Diamondbacks", wins: 81, losses: 81, division: "west"},
      { name: "Colorado Rockies", wins: 64, losses: 98, division: "west" },
      { name: "Los Angeles Dodgers", wins: 86, losses: 76, division: "west" },
      { name: "San Diego Padres", wins: 76, losses: 86, division: "west" },
      { name: "San Francisco Giants", wins: 94, losses: 68, division: "west" },
    },
    { east:
      { name: "Atlanta Braves", wins: 94, losses: 68, division: "east" },
      { name: "Miami Marlins", wins: 69, losses: 93, division: "east" },
      { name: "New York Mets", wins: 74, losses: 88, division: "east" },
      { name: "Philadelphia Phillies", wins: 81, losses: 81, division: "east" },
      { name: "Washington Nationals", wins: 98, losses: 64, division: "east"    }
    },
    { central:
      { name: "Chicago Cubs", wins: 61, losses: 101, division: "central" },
      { name: "Cincinnati Reds", wins: 97, losses: 65, division: "central" },
      { name: "Houston Astros", wins: 55, losses: 107, division: "central" },
      { name: "Milwaukee Brewers", wins: 83, losses: 79, division: "central" },
      { name: "Pittsburgh Pirates", wins: 79, losses:  83, division: "central" },
      { name: "St. Louis Cardinals",  wins: 88, losses: 74, division: "central" },
    }
  }
```

We can also use the `countBy()` function to simply count the number of elements in each group.

```language-javascript
> _(national_league).countBy("division")
  {west: 5, east: 5, central: 6}
```

> Note: Although we’ve used a property value (`"division"`) for `groupBy()` and `countBy()`, both methods also accept an arbitrary function if the criteria for grouping isn’t a simple property.

As a final trick, Underscore.js let’s us randomly reorder a collection using the `shuffle()` function.

```language-javascript
_(national_league).shuffle()
```
