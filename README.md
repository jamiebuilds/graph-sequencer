# graph-sequencer

> Sort items in a graph using a topological sort while resolving cycles with
> priority groups.

Say you have some sort of graph of dependencies: (using an adjacency list)

```js
let graph = {
  "task-a": ["task-d"], // task-a depends on task-d
  "task-b": ["task-d", "task-a"],
  "task-c": ["task-d"],
  "task-d": ["task-a"],
};
```

You could run a topological sort on these items, but you'd still end up with
cycles:

```
task-a -> task-d -> task-a
```

To resolve this you pass "priority groups" to `graph-sequencer`:

```js
let groups = [
  ["task-d"], // higher priority
  ["task-a", "task-b", "task-c"], // lower priority
];
```

The result will be a chunked list of items sorted topologically and by the
priority groups:

```js
let chunks = [
  ["task-d"],
  ["task-a", "task-c"],
  ["task-b"]
];
```

You can then run all these items in order with maximum concurrency:

```js
for (let chunk of chunks) {
  await Promise.all(chunk.map(task => exec(task)));
}
```

However, even with priority groups you can still accidentally create cycles of
dependencies in your graph.

`graph-sequencer` will return a list of the unresolved cycles:

```js
let cycles = [
  ["task-a", "task-b"] // task-a depends on task-b which depends on task-a
];
```

However, `graph-sequencer` will still return an "unsafe" set of chunks. When it
comes across a cycle, it will add another chunk with the item with the fewest
dependencies remaining which will often break cycles.


All together that looks like this:

```js
const graphSequencer = require('graph-sequencer');

graphSequencer({
  graph: {
    "task-a": ["task-d"], // task-a depends on task-d
    "task-b": ["task-d", "task-a"],
    "task-c": ["task-d"],
    "task-d": ["task-a"],
  },
  groups: [
    ["task-d"], // higher priority
    ["task-a", "task-b", "task-c"], // lower priority
  ],
})
// {
//   safe: true,
//   chunks: [["task-d"], ["task-a", "task-c"], ["task-b"]],
//   cycles: [],
// }
```
