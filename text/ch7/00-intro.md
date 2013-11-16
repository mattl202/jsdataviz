## Chapter 7: Managing Data in the Browser

In the first six chapters we’ve looked at a lot of visualization tools and techniques, but we haven’t spent much time considering the _data_ part of data visualization. The emphasis on visualization is appropriate in many cases. Especially if the data is static, we can take all the time we need to clean and groom it before it’s even represented in Javascript. But what if we’re not so lucky. Our data may be dynamic, and we may have no choice but to retrieve the raw source directly into our Javascript application. We have much less control over data from third party REST APIs, Google Docs spreadsheets, or automatically generated CSV files. With those types of data sources, we often need to validate, reformat, recalculate, or otherwise manipulate the data in the browser.

This chapter considers a Javascript library that is particularly helpful for managing large data sets in the web browser--[Underscore.js](http://underscorejs.org). We‘ll cover several aspects of Underscore in the following sections:

* A programming style that Underscore.js encourages called “functional programming.”
* Working with simple arrays using Underscore.js utilities.
* Enhancing Javascript objects.
* Manipulating collections of objects.

The format of this chapter differs from the other chapters in the book. Instead of covering a few examples of moderate complexity, we’ll look a lot of simple, small examples. Each section collects several related examples together, but each of the small examples is independent. The first section differs even further. It’s a brief introduction to functional programming cast as a step-by-step migration from the more common programming style. Understanding functional programming is very helpful, however, as its philosophy underlies almost all of the Underscore.js utilities. This chapter serves as a tour of the Underscore.js library with a special focus on managing data. (As a concession to the book’s overall focus on data visualization, it also includes many illustrations.)