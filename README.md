# objection-match

A lightweight search plugin built on top of [Objection.js](https://github.com/Vincit/objection.js).

**objection-match** was created for programs that support dynamic searches. In a distributed setting, a set of constraints for a search may be defined on a client machine and sent to a server for processing. However, the task of parsing the payload, loading relations, and building a query can be cumbersome. By providing a robust and portable query language that can be used to represent a complex search with various constraints, this problem can be abstracted over. Since Objection already has a powerful, expressive way for loading relations, this library simply takes any valid input query and builds an objection query.

## Grammar

The grammar can be found in [src/lib/parser/parser.pegjs](https://github.com/rkrishn7/objection-match/blob/master/src/lib/parser/parser.pegjs). It defines the structure of the query language.

Briefly, a query consists of _logical_ and _comparison_ nodes. The tables below describe the mapping from each node to their respective function.

| Logical Node | Corresponding Function |
| ------------ | ---------------------- |
| match_all    | AND                    |
| match_any    | OR                     |

<br/>

| Comparison Node | Corresponding Function |
| --------------- | ---------------------- |
| eq              | =                      |
| neq             | !=                     |
| geq             | >=                     |
| leq             | <=                     |
| lt              | <                      |
| gt              | >                      |
| like            | LIKE                   |
| in              | IN                     |

Logical nodes can contain children that include both node types while comparison nodes cannot contain any children. Here's an example:

```js
match_all: {
  eq: ["person.name", "Antonio"],
  neq: ["shirt.color", "white"],
  match_any: {
    eq: ["shirt.style", "polo"],
    eq: ["shirt.style", "dress"]
  }
}
```

A simple, browser-compatible package that constructs this syntax will be available as well.

## Usage

The `search()` method is invoked directly on a model class. In order to support this, the plugin mixin needs to be added to any models that wish to use it. Example usage:

```tsx
import Search from 'objection-match';

class Employee extends Search(Model) {
  ...
}
```

Now, you can call the `search()` method like so:

```tsx
const results = await Employee.search({
  predicate: `
    match_all: {
      geq: ["salary", 60000],
      geq: ["salary_start_date", "1986-06-26"],
      in: ["first_name", "Georgi, Bob"]
    }
  `,
  limit: 5,
  fields: ['salaries.salary', 'first_name'],
  aliases: {
    salary: 'salaries.salary',
    salary_start_date: 'salaries.from_date',
  },
  orderBy: ['salary', 'desc'],
});
```

WIP
