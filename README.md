<h1 align="center">dot.mq</h1>

A [Graphviz DOT](https://graphviz.org/doc/info/lang.html) language parser implemented as an [mq](https://github.com/harehare/mq) module.

## Features

- Parse `graph` and `digraph` definitions
- Node definitions with attribute dicts
- Edge definitions (`->` and `--`) with optional labels
- Graph-level, node-level, and edge-level `[attr]` blocks
- `strict` keyword support
- Source node detection for digraphs

## Installation

```sh
cp dot.mq ~/.local/mq/config/
```

### HTTP Import

```sh
mq -I raw 'import "github.com/harehare/dot.mq" | dot::dot_parse(.)' graph.dot
```

## API

| Function | Description |
|---|---|
| `dot_parse(input)` | Parse DOT source, return `{ type, name, strict, nodes, edges, attrs }` |
| `dot_is_directed(input)` | Returns `true` for `digraph` |
| `dot_sources(input)` | Returns nodes with no incoming edges |

## Example

Given `pipeline.dot`:

```dot
digraph pipeline {
  graph [rankdir=LR]
  checkout [label="Checkout" shape=box]
  build    [label="Build"    shape=box]
  test     [label="Test"     shape=diamond]
  deploy   [label="Deploy"   shape=box]

  checkout -> build
  build    -> test
  test     -> deploy [label="passed"]
}
```

```sh
# Get all node labels
mq -I raw 'import "dot" | dot::dot_parse(.) | ."nodes" | map(fn(n): n["attrs"]["label"];)' pipeline.dot
# => ["Checkout", "Build", "Test", "Deploy"]

# Get edges with labels
mq -I raw 'import "dot" | dot::dot_parse(.) | ."edges" | filter(fn(e): len(e["attrs"]["label"]) > 0;)' pipeline.dot
# => [{"from":"test","to":"deploy","attrs":{"label":"passed"}}]

# Find entry-point nodes
mq -I raw 'import "dot" | dot::dot_sources(.) | map(fn(n): n["id"];)' pipeline.dot
# => ["checkout"]

# Count nodes and edges
mq -I raw 'import "dot" | dot::dot_parse(.) | {"nodes": len(."nodes"), "edges": len(."edges")}' pipeline.dot
# => {"nodes":4,"edges":4}
```

## Compatibility

Requires [mq](https://github.com/harehare/mq) v0.5 or later.

## License

MIT
