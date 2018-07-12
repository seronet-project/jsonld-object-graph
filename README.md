# graphql-jsonld-utils
Using JSON-LD with GraphQL.

The function `jsonld2obj` constructs an object graph in memory by resolving the `@id` properties recursively.
The graph can contain cycles. It also constructs a `@id -> object` map for fast access to objects.

# Add dependency to your project
```console
$ yarn add https://github.com/vsimko/graphql-jsonld-utils.git
```

# Example
```js
const {jsonld2obj} = require('graphql-jsonld-utils')
const data = [
  {
    "@context": "https://json-ld.org/contexts/person.jsonld",
    "@id": "http://my/person/Gordon",
    "name": "Gordon Freeman",
    "knows": "http://my/person/Alyx"
  },
  {
    "@context": "https://json-ld.org/contexts/person.jsonld",
    "@id": "http://my/person/Alyx",
    "name": "Alyx Vence",
    "knows": "http://my/person/Gordon"
  }
]

const contexts = {
  '@base': 'http://my/person/',
  'foaf': 'http://xmlns.com/foaf/0.1/'
}
    
const {graph, id2obj} = await jsonld2obj(data, contexts)
console.log(graph)
```

Generates the following output:
```
[ { '@id': 'Gordon',
    'foaf:knows':
     { '@id': 'Alyx',
       'foaf:knows': [Circular],
       'foaf:name': 'Alyx Vence' },
    'foaf:name': 'Gordon Freeman' },
  { '@id': 'Alyx',
    'foaf:knows':
     { '@id': 'Gordon',
       'foaf:knows': [Circular],
       'foaf:name': 'Gordon Freeman' },
    'foaf:name': 'Alyx Vence' } ]
```

Now it is possible to navigate the graph as follows:
```js
id2obj.Gordon['foaf:knows']['foaf:name'] // -> "Alyx Vence"
id2obj.Gordon['foaf:knows']['foaf:knows']['foaf:name'] // -> "Gordon Freeman"
```
