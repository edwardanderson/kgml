---
title: Knowledge Graph Markup Language (KGML) Specification
author: Edward Anderson
---

# Knowledge Graph Markup Language

- [Knowledge Graph Markup Language](#knowledge-graph-markup-language)
  - [Document](#document)
    - [Body](#body)
      - [Structure](#structure)
        - [Graph](#graph)
          - [Unordered](#unordered)
          - [Ordered](#ordered)
        - [Comment](#comment)
      - [Resources](#resources)
        - [Hyperlink](#hyperlink)
          - [Label](#label)
          - [Class](#class)
        - [Plain Text](#plain-text)
        - [Blockquote](#blockquote)
          - [Identified](#identified)
          - [Reified](#reified)
          - [Language](#language)
          - [Style](#style)
          - [Reference](#reference)
          - [Datatype](#datatype)
        - [Image](#image)
        - [Code Block](#code-block)
      - [Table](#table)
    - [Frontmatter](#frontmatter)
      - [Base](#base)
      - [Document Language](#document-language)
      - [Import](#import)

> [!Note]
> v0.1

> [!Warning]
> Releases prior to v1.0 may unexpectedly break.

This document specifies the Knowledge Graph Markup Language (KGML). It includes language-agnostic [go-testmark](https://github.com/warpfork/go-testmark) integration test fixtures. Each input scenario is accompanied by an expected graph shape, described in Turtle, and an expected JSON-LD serialisation.

## Document

A KGML document is a plain-text Markdown document that complies with the following conventions for structure format and syntax.

### Body

#### Structure

##### Graph

Graphs of triples are composed of nests of either [unordered](#unordered) or [ordered](#ordered) lists of [resources](#resources).

###### Unordered

[testmark]:# (1-input)
```markdown
- John
  - knows
    - Paul
```

<details><summary><code>text/turtle</code></summary>

[testmark]:# (1-expected-isomorphic-ttl)
```turtle
@prefix : <http://example.org/> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

[] rdfs:label "John" ;
    :knows [ rdfs:label "Paul" ] .
```

</details>

<details><summary><code>application/ld+json</code></summary>

[testmark]:# (1-expected-string-json)
```json
{
  "@context": [
    {
      "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
      "_label": "rdfs:label"
    },
    {
      "@vocab": "http://example.org/"
    }
  ],
  "@graph": [
    {
      "_label": "John",
      "knows": {
        "_label": "Paul"
      }
    }
  ]
}
```

</details>

###### Ordered

Objects may be listed in sequence.

[testmark]:# (2-input)
```markdown
- John
  - spouse
    1. Cynthia
    2. Yoko
```

<details><summary><code>text/turtle</code></summary>

[testmark]:# (2-expected-isomorphic-ttl)
```turtle
@prefix : <http://example.org/> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

[] rdfs:label "John" ;
    :spouse (
        [ rdfs:label "Cynthia" ]
        [ rdfs:label "Yoko" ]
    ) .
```

</details>

<details><summary><code>application/ld+json</code></summary>

[testmark]:# (2-expected-string-json)
```json
{
  "@context": [
    {
      "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
      "_label": "rdfs:label"
    },
    {
       "@vocab": "http://example.org/"
    }
  ],
  "@graph": [
    {
      "_label": "John",
      "spouse": {
        "@list": [
           {
             "_label": "Cynthia"
           },
           {
             "_label": "Yoko"
           }
         ]
       }
    }
  ]
}
```

</details>

##### Comment

Comments not intended to be represented in data may be given in HTML.

[testmark]:# (5-input)
```markdown
<!-- Content inside HTML comment tags is ignored. -->

- John
  <!-- Consider using foaf:knows -->
  - knows
    - Paul
```

<details><summary><code>text/turtle</code></summary>

[testmark]:# (5-isomorphic-ttl)
```turtle
@prefix : <http://example.org/> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

[] rdfs:label "John" ;
  :knows [ rdfs:label "Paul" ] .
```

</details>

<details><summary><code>application/ld+json</code></summary>

[testmark]:# (5-expected-string-json)
```json
{
  "@context": [
    {
      "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
      "_label": "rdfs:label"
    },
    {
      "@vocab": "http://example.org/"
    }
  ],
  "@graph": [
    {
      "_label": "John",
      "knows": {
        "_label": "Paul"
      }
    }
  ]
}
```

</details>

#### Resources

Resources may be [hyperlinks](#hyperlink), [plain-text](#plain-text), [blockquotes](#blockquote) or [images](#image).

##### Hyperlink

Hyperlinks are composed of an IRI and, optionally, a [label](#label) and/or a [class](#class). If no label is provided, the resource is labelled with the last segment of the path.

IRIs may be absolute or relative. Resources are resolved relative to the [base](#base), which can be set in the document frontmatter.

[testmark]:# (6-input)
```markdown
- [John](http://example.org/john)
- <http://example.org/paul>
- [](http://example.org/george)
- [Ringo][1]
- [Yoko](people/yoko)

[1]: http://example.com/ringo
```

<details><summary><code>text/turtle</code></summary>

[testmark]:# (6-expected-isomorphic-ttl)
```turtle
@base <http://example.org> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

<http://example.org/john> rdfs:label "John" .

<http://example.org/paul> rdfs:label "paul" .

<http://example.org/george> rdfs:label "george" .

<http://example.org/ringo> rdfs:label "Ringo" .

<people/yoko> rdfs:label "Yoko" .
```

</details>

<details><summary><code>application/ld+json</code></summary>

[testmark]:# (6-expected-string-json)
```json
{
  "@context": [
    {
      "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
      "_label": "rdfs:label"
    },
    {
      "@base": "http://example.org/"
    }
  ],
  "@graph": [
    {
      "@id": "http://example.org/john",
      "_label": "John",
    },
    {
      "@id": "http://example.org/paul",
      "_label": "paul"
    },
    {
      "@id": "http://example.org/george",
      "_label": "george"
    },
    {
      "@id": "http://example.org/ringo",
      "_label": "Ringo"
    },
    {
      "@id": "people/yoko",
      "_label": "Yoko"
    }
  ]
}
```

</details>

###### Label

The language of the label may be specified.

[testmark]:# (7-input)
```markdown
- [John `en`](http://example.org/john)
```

<details><summary><code>text/turtle</code></summary>

[testmark]:# (7-expected-isomorphic-ttl)
```turtle
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

<http://example.org/john> rdfs:label "John"@en .
```

</details>

<details><summary><code>application/ld+json</code></summary>

[testmark]:# (7-expected-string-json)
```json
{
  "@context": [
    {
      "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
      "_label": "rdfs:label"
    }
  ],
  "@graph": [
    {
      "_label": {
        "@language": "en",
        "@value":  "John"
      }
    }
  ]
}
```

</details>

Trailing spaces are not preserved. To preserve the trailing space use a [whitespace character](https://en.wikipedia.org/wiki/Whitespace_character).

The label may be [styled](#style).

[testmark]:# (8-input)
```markdown
- [**John**](http://example.org/john)
```

<details><summary><code>text/turtle</code></summary>

[testmark]:# (8-expected-isomorphic-ttl)
```turtle
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

<http://example.org/john> rdfs:label "<p><strong>John</strong></p>"^^rdf:HTML .
```

</details>

</details>

<details><summary><code>application/ld+json</code></summary>

[testmark]:# (8-expected-string-json)
```json
{
  "@context": [
    {
      "@vocab": "http://example.org/",
      "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
      "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
      "_label": "rdfs:label",
      "_HTML": "rdf:HTML"
    }
  ],
  "@graph": [
    {
      "@id": "http://example.org/john",
      "_label": {
        "@type": "_HTML",
        "@value":  "<p><strong>John</strong></p>"
      }
    }
  ]
}
```

</details>

###### Class

The predicate keyword `a` can be provided to specify the `rdf:type` of a resource.

[testmark]:# (9-input)
```markdown
- John
  - a
    - Person
```

<details><summary><code>text/turtle</code></summary>

[testmark]:# (9-expected-isomorphic-ttl)
```turtle
@prefix : <http://example.org/> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

[] rdfs:label "John" ;
    a :Person .
```

</details>

<details><summary><code>application/ld+json</code></summary>

[testmark]:# (9-expected-string-json)
```json
{
  "@context": [
    {
      "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
      "_label": "rdfs:label"
    },
    {
      "@vocab": "http://example.org/"
    }
  ],
  "@graph": [
    {
      "_label": "John",
      "@type": "Person"
    }
  ]
}
```

</details><br/>

The tokens `â` or `^a` may be used to inverse the direction of the class assignment.

[testmark]:# (10-input)
```markdown
- Person
  - â
    - John
    - Paul
    - George
```

<details><summary><code>text/turtle</code></summary>

[testmark]:# (10-expected-isomorphic-ttl)
```turtle
@prefix : <http://example.org/> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

[] rdfs:label "John" ;
    a :Person .

[] rdfs:label "Paul" ;
    a :Person .

[] rdfs:label "George" ;
    a :Person .
```

</details>

<details><summary><code>application/ld+json</code></summary>

[testmark]:# (10-expected-string-json)
```json
{
  "@context": [
    {
      "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
      "_label": "rdfs:label"
    },
    {
      "@vocab": "http://example.org/"
    }
  ],
  "@graph": [
    {
      "_label": "John",
      "@type": "Person"
    },
    {
      "_label": "Paul",
      "@type": "Person"
    },
    {
      "_label": "George",
      "@type": "Person"
    }
  ]
}
```

</details>

Hyperlinks may include a class by overloading the hyperlink's `title` attribute with an IRI or a defined term.

[testmark]:# (11-input)
```markdown
- [John](http://example.org/john "http://example.org/Person")
  - [knows](http://xmlns.com/foaf/0.1/knows "http://www.w3.org/2002/07/owl#SymmetricProperty")
    - [Paul](http://example.org/paul "Person")

Person
: <http://schema.org/Person>
```

<details><summary><code>text/turtle</code></summary>

[testmark]:# (11-expected-isomorphic-ttl)
```turtle
@prefix : <http://example.org/> .
@prefix foaf: <http://xmlns.com/foaf/0.1/> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

<http://example.org/john> rdfs:label "John" ;
    a <http://example.org/Person> ;
    foaf:knows <http://example.org/paul> .

<http://example.org/paul> rdfs:label "Paul" ;
    a <http://example.org/Person> .

foaf:knows a owl:SymmetricProperty .
```

</details>

<details><summary><code>application/ld+json</code></summary>

[testmark]:# (11-expected-string-json)
```json
{
  "@context": [
    {
      "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
      "_label": "rdfs:label"
    },
    {
      "knows": {
        "@id": "http://xmlns.com/foaf/0.1/knows"
      },
      "Person": {
        "@id": "http://schema.org/Person"
      }
    }
  ],
  "@graph": [
    {
      "@id": "http://example.org/john",
      "@type": "Person",
      "_label": "John",
      "knows": {
        "@id": "http://example.org/paul",
        "@type": "Person",
        "_label": "Paul"
      }
    },
    {
      "@id": "http://xmlns.com/foaf/0.1/knows",
      "@type": "http://www.w3.org/2002/07/owl#SymmetricProperty"
    }
  ]
}
```

</details>

##### Plain Text

Subject and object resources in plain text are blank nodes with labels. The can be referred to by their label within the document.

[testmark]:# (12-input)
```markdown
- John
  - knows
    - Paul
      - knows
        - John
- Paul
  - birth place
    - Liverpool
```

<details><summary><code>text/turtle</code></summary>

[testmark]:# (12-expected-isomorphic-ttl)
```turtle
@prefix : <http://example.org/> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

_:John rdfs:label "John" ;
  :knows _:Paul .

_:Paul rdfs:label "Paul" ;
  :knows _:John ;
  :birth_place [ rdfs:label "Liverpool" ] .
```

</details>

<details><summary><code>application/ld+json</code></summary>

[testmark]:# (12-expected-string-json)
```json
{
  "@context": [
    {
      "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
      "_label": "rdfs:label"
    },
    {
      "@vocab": "http://example.org/"
    }
  ],
  "@graph": [
    {
      "@id": "_:John",
      "_label": "John",
      "knows": {
        "@id": "_:Paul",
        "_label": "Paul",
        "knows": {
          "@id": "_:John"
        },
        "birth_place": {
          "_label": "Liverpool"
        }
      }
    }
  ]
}
```

</details>

Plain text resources may be identified with definition lists.

[testmark]:# (13-input)
```markdown
- John
  - knows
    - Paul

John
: <http://www.wikidata.org/entity/Q1203>

knows
: <http://xmlns.com/foaf/0.1/knows>
```

<details><summary><code>text/turtle</code></summary>

[testmark]:# (13-expected-isomorphic-ttl)
```turtle
@prefix foaf: <http://xmlns.com/foaf/0.1/> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

<http://www.wikidata.org/entity/Q1203> rdfs:label "John" ;
    foaf:knows [ rdfs:label "Paul" ] .
```

</details>

<details><summary><code>application/ld+json</code></summary>

[testmark]:# (13-expected-string-json)
```json
{
  "@context": [
    {
      "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
      "_label": "rdfs:label"
    },
    {
      "knows": "http://xmlns.com/foaf/0.1/knows"
    }
  ],
  "@graph": [
    {
      "@id": "http://www.wikidata.org/entity/Q1203",
      "_label": "John",
      "knows": {
        "_label": "Paul"
      }
    }
  ]
}
```

</details>

Resources with multiple definitions have the same identity.

[testmark]:# (14-input)
```
- John

John
: <http://www.wikidata.org/entity/Q1203>
: <https://vocab.getty.edu/ulan/500106615>
```

<details><summary><code>text/turtle</code></summary>

[testmark]:# (14-expected-isomorphic-ttl)
```turtle
@prefix owl: <http://www.w3.org/2002/07/owl#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

<http://www.wikidata.org/entity/Q1203> rdfs:label "John" ;
    owl:sameAs <https://vocab.getty.edu/ulan/500106615> .
```

</details>

<details><summary><code>application/ld+json</code></summary>

[testmark]:# (14-expected-string-json)
```json
{
  "@context": [
    {
      "owl": "http://www.w3.org/2002/07/owl#",
      "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
      "_label": "rdfs:label",
      "_same_as": {
        "@id": "owl:sameAs",
        "@container": "@set"
      }
    }
  ],
  "@graph": [
    {
      "@id": "http://www.wikidata.org/entity/Q1203",
      "_label": "John",
      "_same_as": [
        {
          "@id": "https://vocab.getty.edu/ulan/500106615"
        }
      ]
    }
  ]
}
```

</details>

##### Blockquote

Blockquotes contain literal text.

[testmark]:# (15-input)
```markdown
- John
  - name
    - > John Winston Lennon
```

<details><summary><code>text/turtle</code></summary>

[testmark]:# (15-expected-isomorphic-ttl)
```turtle
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

[] rdfs:label "John" ;
    :name "John Winston Lennon" .
```

</details>

<details><summary><code>application/ld+json</code></summary>

[testmark]:# (15-expected-string-json)
```json
{
  "@context": [
    {
      "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
      "_label": "rdfs:label"
    },
    {
      "@vocab": "http://example.org/"
    }
  ],
  "@graph": [
    {
      "_label": "John",
      "name": "John Winston Lennon"
    }
  ]
}
```

</details>

###### Identified

A preceding sibling hyperlink identifies the blockquote.

[testmark]:# (16-input)
```markdown
- John
  - description
    - [bio](http://example.org/biography/1)
      > Born in Liverpool, Lennon became involved in the skiffle craze as a teenager.
```

<details><summary><code>text/turtle</code></summary>

[testmark]:# (16-expected-isomorphic-ttl)
```turtle
@prefix : <http://example.org/> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

[] rdfs:label "John" ;
    :description <http://example.org/biography/1> .

<http://example.org/biography/1> rdfs:label "bio" ;
    rdf:value "Born in Liverpool, Lennon became involved in the skiffle craze as a teenager." .
```

</details>

<details><summary><code>application/ld+json</code></summary>

[testmark]:# (16-expected-string-json)
```json
{
  "@context": [
    {
      "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
      "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
      "_label": "rdfs:label",
      "_content": "rdf:value"
    },
    {
      "@vocab": "http://example.org/"
    }
  ],
  "@graph": [
    {
      "_label": "John",
      "description": {
        "@id": "http://example.org/biography/1",
        "_label": "bio",
        "_content": "Born in Liverpool, Lennon became involved in the skiffle craze as a teenager."
      }
    }
  ]
}
```

</details>

A plain-text resource may also identify the blockquote in the local scope.

[testmark]:# (17-input)
```markdown
- John
  - name
    - a
      > John Winston Lennon
    - b
      > John Winston Ono Lennon
```

<details><summary><code>text/turtle</code></summary>

[testmark]:# (17-expected-isomorphic-ttl)
```turtle
@prefix : <http://example.org/> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

[] rdfs:label "John" ;
    :name _:a , _:b .

_:a rdfs:label "a" ;
    rdf:value "John winston Lennon" .

_:b rdfs:label "b" ;
    rdf:value "John Winston Ono Lennon" .
```

</details>

<details><summary><code>application/ld+json</code></summary>

[testmark]:# (17-expected-string-json)
```json
{
  "@context": [
    {
      "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
      "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
      "_label": "rdfs:label",
      "_content": "rdf:value"
    },
    {
      "@vocab": "http://example.org/"
    }
  ],
  "@graph": [
    {
      "_label": "John",
      "name": [
        {
          "@id": "_:a",
          "_label": "a",
          "_content": "John Winston Lennon"
        },
        {
          "@id": "_:b",
          "_label": "b",
          "_content": "John Winston Ono Lennon"
        }
      ]
    }
  ]
}
```

</details>


###### Reified

Making a statement about a blockquote reifies it.

[testmark]:# (18-input)
```markdown
- John
  - description
    - > He gained worldwide fame as the founder, co-lead vocalist and rhythm guitarist of the Beatles.
      - source
        - [Wikipedia](http://www.wikidata.org/entity/Q52)
```

<details><summary><code>text/turtle</code></summary>

[testmark]:# (18-expected-isomorphic-ttl)
```turtle
@prefix : <http://example.org/> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

[] rdfs:label "John" ;
    :description [
        rdf:value "He gained worldwide fame as the founder, co-lead vocalist and rhythm guitarist of the Beatles." ;
        :source <http://www.wikidata.org/entity/Q52>
    ] .

<http://www.wikidata.org/entity/Q52> rdfs:label "Wikipedia" .
```

</details>

<details><summary><code>application/ld+json</code></summary>

[testmark]:# (18-expected-string-json)
```json
{
  "@context": [
    {
      "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
      "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
      "_label": "rdfs:label",
      "_content": "rdf:value"
    },
    {
      "@vocab": "http://example.org/"
    }
  ],
  "@graph": [
    {
      "_label": "John",
      "description": {
        "_content": "He gained worldwide fame as the founder, co-lead vocalist and rhythm guitarist of the Beatles.",
        "source": {
          "@id": "http://www.wikidata.org/entity/Q52",
          "_label": "Wikipedia"
        }
      }
    }
  ]
}
```

</details>

###### Language

A language may be specified in code fences at the end of the line.

[testmark]:# (19-input)
```markdown
- John
  - said
    - > You may say I'm a dreamer `en`
```

<details><summary><code>text/turtle</code></summary>

[testmark]:# (19-expected-isomorphic-ttl)
```turtle
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

[] rdfs:label "John" ;
    :said "You may say I'm a dreamer"@en .
```

</details>

<details><summary><code>application/ld+json</code></summary>

[testmark]:# (19-expected-string-json)
```json
{
  "@context": [
    {
      "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
      "_label": "rdfs:label"
    },
    {
      "@vocab": "http://example.org/"
    }
  ],
  "@graph": [
    {
      "_label": "John",
      "said": {
        "@language": "en",
        "@value": "You may say I'm a dreamer"
      }
    }
  ]
}
```

</details>

###### Style

Styled text is serialised as HTML.

[testmark]:# (20-input)
```markdown
- John
  - note
    - > **John Winston Ono Lennon** was an English singer, songwriter and musician. 
```

<details><summary><code>text/turtle</code></summary>

[testmark]:# (20-expected-isomorphic-ttl)
```turtle
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

[] rdfs:label "John" ;
    :note "<p><strong>John Winston Ono Lennon</strong> was an English singer, songwriter and musician</p>"^^rdf:HTML .
```

</details>

<details><summary><code>application/ld+json</code></summary>

[testmark]:# (20-expected-string-json)
```json
{
  "@context": [
    {
      "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
      "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
      "_label": "rdfs:label",
      "_HTML": "rdf:HTML"
    },
    {
      "@vocab": "http://example.org/"
    }
  ],
  "@graph": [
    {
      "_label": "John",
      "note": {
        "@type": "_HTML",
        "@value": "<p><strong>John Winston Ono Lennon</strong> was an English singer, songwriter and musician</p>"
      }
    }
  ]
}
```

</details>

Specifying the language of styled text attributes the paragraph `p`.

[testmark]:# (21-input)
```markdown
- Paul
  - note
    - > **Sir James Paul McCartney** CH MBE (born 18 June 1942) is an English singer `en`
```

<details><summary><code>text/turtle</code></summary>

[testmark]:# (21-expected-isomorphic-ttl)
```turtle
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

[] rdfs:label "Paul" ;
    :note "<p lang=\"en\"><strong>Sir James Paul McCartney</strong> CH MBE (born 18 June 1942) is an English singer</p>"^^rdf:HTML .
```

</details>

<details><summary><code>application/ld+json</code></summary>

[testmark]:# (21-expected-string-json)
```json
{
  "@context": [
    {
      "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
      "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
      "_content": "rdf:value",
      "_label": "rdfs:label",
      "_HTML": "rdf:HTML"
    },
    {
      "@vocab": "http://example.org/"
    }
  ],
  "@graph": [
    {
      "_label": "Paul",
      "note": {
        "@type": "_HTML",
        "_content": "<p lang=\"en\"><strong>Sir James Paul McCartney</strong> CH MBE (born 18 June 1942) is an English singer</p>"
      }
    }
  ]
}
```

</details>

###### Reference

[Hyperlinks](#hyperlink) and [images](#image) inside blockquotes are materialised as references.

[testmark]:# (22-input)
```markdown
- John
  - description
    - > In 1956, he formed the [Quarrymen](https://en.wikipedia.org/wiki/The_Quarrymen)
```

<details><summary><code>text/turtle</code></summary>

[testmark]:# (22-expected-isomorphic-ttl)
```turtle
@prefix : <http://example.org/> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

[] rdfs:label "John" ;
    :description [
        rdf:value "<p>In 1956, he formed the <a href=\"https://en.wikipedia.org/wiki/The_Quarrymen\">Quarrymen</a></p>"^^rdf:HTML ;
        rdfs:seeAlso <https://en.wikipedia.org/wiki/The_Quarrymen>
    ] .
```

</details>

<details><summary><code>application/ld+json</code></summary>

[testmark]:# (22-expected-string-json)
```json
{
  "@context": [
    {
      "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
      "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
      "_content": "rdf:value",
      "_label": "rdfs:label",
      "_see_also": "rdfs:seeAlso",
      "_HTML": "rdf:HTML"
    },
    {
      "@vocab": "http://example.org/"
    }
  ],
  "@graph": [
    {
      "_label": "John",
      "description": {
        "@type": "_HTML",
        "_content": "<p>In 1956, he formed the <a href=\"https://en.wikipedia.org/wiki/The_Quarrymen\">Quarrymen</a></p>",
        "_see_also": [
          {
            "@id": "https://en.wikipedia.org/wiki/The_Quarrymen",
            "_label": "The Quarrymen"
          }
        ]
      }
    }
  ]
}
```

</details>

###### Datatype

A datatype may be given as a code-fenced reference to a defined term.

[testmark]:# (23-input)
```markdown
- John
  - date of birth
    - > 1940-10-09 `date`

date
: <http://www.w3.org/2001/XMLSchema#date>
```

<details><summary><code>text/turtle</code></summary>

[testmark]:# (23-expected-isomorphic-ttl)
```turtle
@prefix : <http://example.org/> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .

[] rdfs:label "John" ;
    :date-of-birth "1940-10-09"^^xsd:date .
```

</details>

<details><summary><code>application/ld+json</code></summary>

[testmark]:# (23-expected-string-json)
```json
{
  "@context": [
    {
      "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
      "_label": "rdfs:label",
    },
    {
      "@vocab": "http://example.org/",
      "date": "http://www.w3.org/2001/XMLSchema#date"
    }
  ],
  "@graph": [
    {
      "_label": "John",
      "date_of_birth": {
        "@type": "date",
        "@value": "1940-10-09"
      }
    }  
  ]
}
```

</details>

To escape a code-fenced term from being processed as a [language](#language) or [datatype](#datatype), add a line-feed character `&#xA;`.

[testmark]:# (24-input)
```markdown
- John
  - said
    - > It's been too long since we took the `time`&#xA;
```

<details><summary><code>text/turtle</code></summary>

[testmark]:# (24-expected-isomorphic-ttl)
```turtle
@prefix : <http://example.org/> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

[] rdfs:label "John" ;
    :said "It's been too long since we took the <code>time</code>"^^rdf:HTML .
```

</details>

<details><summary><code>application/ld+json</code></summary>

[testmark]:# (24-expected-string-json)
```json
{
  "@context": [
    {
      "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
      "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
      "_label": "rdfs:label",
      "_HTML": "rdf:HTML"
    },
    {
      "@vocab": "http://example.org/"
    }
  ],
  "@graph": [
    {
      "_label": "John",
      "said": {
        "@type": "_HTML",
        "@value": "It's been too long since we took the <code>time</code>"
      }
    }  
  ]
}
```

</details>

##### Image

[testmark]:# (25-input)
```markdown
- ![John Lennon, 1974 (restored cropped)](https://shorturl.at/SacIh)
```

<details><summary><code>text/turtle</code></summary>

[testmark]:# (25-expected-isomorphic-ttl)
```turtle
@prefix dcmitype: <http://purl.org/dc/dcmitype/> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

<https://shorturl.at/SacIh> a dcmitype:Image ;
  rdfs:label "John Lennon, 1974 (restored cropped)" .
```

</details>

<details><summary><code>application/ld+json</code></summary>

[testmark]:# (25-expected-string-json)
```json
{
  "@context": [
    {
      "dcmitype": "http://purl.org/dc/dcmitype/",
      "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
      "_label": "rdfs:label",
      "_Image": "dcmitype:Image"
    }
  ],
  "@graph": [
    {
      "@id": "https://shorturl.at/SacIh",
      "_label": "John Lennon, 1974 (restored cropped)",
      "@type": "_Image"
    }  
  ]
}
```

</details>

Image classes may be specified in the same way as for [hyperlink classes](#class).

##### Code Block

[testmark]:# (26-input)
````markdown
- Yesterday
  - lyrics
    - ```text
      There are places I remember
      All my life, though some have changed
      ```
````

<details><summary><code>text/turtle</code></summary>

[testmark]:# (26-expected-isomorphic-ttl)
```turtle
@prefix : <http://example.org/> .
@prefix dcterms: <http://purl.org/dc/terms/> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

[] rdfs:label "Yesterday" ;
  :lyrics [
    dcterms:format "text" ;
    rdf:value "There are places I remember\nAll my life, though some have changed"
  ] .
```

</details>

<details><summary><code>application/ld+json</code></summary>

[testmark]:# (26-expected-string-json)
```json
{
  "@context": [
    {
      "dcterms": "http://purl.org/dc/terms/",
      "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
      "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
      "_content": "rdf:value",
      "_label": "rdfs:label",
      "_format": "dcterms:format"
    },
    {
      "@vocab": "http://example.org/"
    }
  ],
  "@graph": [
    {
      "_label": "Yesterday",
      "lyrics": {
        "_format": "text",
        "_content": "There are places I remember\nAll my life, though some have changed"
      }
    }
  ]
}
```

</details>

Another example.

[testmark]:# (27-input)
````markdown
- ```text
  . 　　 　　　　　
  　·   ·   　    　
  　 ·  ✦ * 
   ✵  . 　　　　·  ·  ⋆  　 
     ✫  ✵  ·　　✵   　　 ˚ 
  · 　  ✵ 　　 　 .  ·
  ```
  - source
    - <https://x.com/tiny_star_field/status/1681381641753640960>
````

<details><summary><code>text/turtle</code></summary>

[testmark]:# (27-expected-isomorphic-ttl)
```turtle
@prefix : <http://example.org/> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .

[] dcterms:format "text" ; 
  rdf:value """  . 　　 　　　　　
  　·   ·   　    　
  　 ·  ✦ * 
   ✵  . 　　　　·  ·  ⋆  　 
     ✫  ✵  ·　　✵   　　 ˚ 
  · 　  ✵ 　　 　 .  ·""" ;
  :source <https://x.com/tiny_star_field/status/1681381641753640960> .
```

</details>

<details><summary><code>application/ld+json</code></summary>

[testmark]:# (27-expected-string-json)
```json
{
  "@context": [
    {
      "dcterms": "http://purl.org/dc/terms/",
      "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
      "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
      "_content": "rdf:value",
      "_format": "dcterms:format"
    },
    {
      "@vocab": "http://example.org/"
    }
  ],
  "@graph": [
    {
      "_format": "text",
      "_content": "\n  . \u3000\u3000 \u3000\u3000\u3000\u3000\u3000\n  \u3000\u00b7   \u00b7   \u3000    \u3000\n  \u3000 \u00b7  \u2726 * \n   \u2735  . \u3000\u3000\u3000\u3000\u00b7  \u00b7  \u22c6  \u3000 \n     \u272b  \u2735  \u00b7\u3000\u3000\u2735   \u3000\u3000 \u02da \n  \u00b7 \u3000  \u2735 \u3000\u3000 \u3000 .  \u00b7\n",
      "source": {
        "@id": "https://x.com/tiny_star_field/status/1681381641753640960"
      }
    }
  ]
}
```

</details>

#### Table

Tables are handled as HTML. It may be preferable to represent information semantically instead or as well.

[testmark]:# (28-input)
```markdown
- The Beatles
  - albums
    - | Title              | Year |
      |-                   |-     |
      | Please Please Me   | 1963 |
      | With the Beatles   | 1963 |
      | A Hard Day's Night | 1964 |
```

<details><summary><code>text/turtle</code></summary>

[testmark]:# (28-expected-isomorphic-ttl)
```turtle
@prefix : <http://example.org/> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

[] rdfs:label "The Beatles" ;
  :albums "<table><thead><tr><th>Title<th>Year<tbody><tr><td>Please Please Me<td>1963<tr><td>With the Beatles<td>1963<tr><td>A Hard Day's Night<td>1964</table>"^^rdf:HTML .
```

</details>

<details><summary><code>application/ld+json</code></summary>

[testmark]:# (28-expected-string-json)
```json
{
  "@context": [
    {
      "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
      "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
      "_content": "rdf:value",
      "_HTML": "rdf:HTML"
    },
    {
      "@vocab": "http://example.org/"
    }
  ],
  "@graph": [
    {
      "_label": "The Beatles",
      "albums": {
        "@type": "_HTML",
        "_content": "<table><thead><tr><th>Title<th>Year<tbody><tr><td>Please Please Me<td>1963<tr><td>With the Beatles<td>1963<tr><td>A Hard Day's Night<td>1964</table>"
      }
    }
  ]
}
```

</details>

Inserting hyperlinks or images into cells will reify the table as a blank node and generate [references](#reference).

### Frontmatter

#### Base

Resolve relative IRIs against the specified base.

[testmark]:# (29-input)
```markdown
---
base: http://example.org/
---

- [John](people/1 "Person")
- [Paul](people/2 "Person")
```

<details><summary><code>text/turtle</code></summary>

[testmark]:# (29-expected-isomorphic-ttl)
```turtle
@base <http://example.org/> .
@prefix : <http://example.org/> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

<people/1> rdfs:label "John" ;
  a :Person .

<people/2> rdfs:label "Paul" ;
  a :Person .
```

</details>

<details><summary><code>application/ld+json</code></summary>

[testmark]:# (29-expected-string-json)
```json
{
  "@context": [
    {
      "rdfs": "http://www.w3.org/2000/01/rdf-schema#"
    },
    {
      "@base": "http://example.org/",
      "@vocab": "http://example.org/"
    }
  ],
  "@graph": [
    {
      "@id": "people/1",
      "@type": "Person",
      "_label": "John"
    },
    {
      "@id": "people/2",
      "@type": "Person",
      "_label": "Person"
    }
  ]
}
```

</details>

#### Document Language

Set a default language for all strings, unless overridden.

[testmark]:# (30-input)
```markdown
---
language: en
---

- John
  - said
    - > I believe in everything until it's disproved
- Yoko
  - name
    - > 小野 洋子 `jp`
```

<details><summary><code>text/turtle</code></summary>

[testmark]:# (30-expected-isomorphic-ttl)
```turtle
@prefix : <http://example.org/> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

[] rdfs:label "John"@en ;
  :said "I believe in everything until it's disproved"@en .

[] rdfs:label "Yoko"@en ;
  :name "小野 洋子"@jp .
```

</details>

<details><summary><code>application/ld+json</code></summary>

[testmark]:# (30-expected-string-json)
```json
{
  "@context": [
    {
      "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
      "_label": "rdfs:label"
    },
    {
      "@base": "http://example.org/",
      "@language": "en",
      "@vocab": "http://example.org/"
    }
  ],
  "@graph": [
    {
      "_label": "John",
      "said": "I believe in everything until it's disproved"
    },
    {
      "_label": "Yoko",
      "name": {
          "@language": "jp",
          "@value": "小野 洋子"
        }
    }
  ]
}
```

</details>

#### Import

Import data from another file.

> ```markdown
> John
> : <http://www.wikidata.org/entity/Q1203>
>
> date
> : <http://www.w3.org/2001/XMLSchema#date>
>
> date of birth
> : <https://schema.org/birthDate>
> ```
>
> -- <http://example.org/terms.md>

[testmark]:# (31-input)
```markdown
---
import: http://example.org/terms.md
---

- John
  - date of birth
    - > 1940-10-09 `date`
```

<details><summary><code>text/turtle</code></summary>

[testmark]:# (31-expected-isomorphic)
```turtle
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix schema: <https://schema.org/> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .

<http://www.wikidata.org/entity/Q1203> rdfs:label "John" ;
    schema:birthDate "1940-10-09"^^xsd:date .
```

</details>

<details><summary><code>application/ld+json</code></summary>

[testmark]:# (30-expected-string-json)
```json
{
  "@context": [
    {
      "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
      "_label": "rdfs:label"
    },
    {
      "date": "http://www.w3.org/2001/XMLSchema#date",
      "date_of_birth": "https://schema.org/birthDate"
    }
  ],
  "@graph": [
    {
      "@id": "http://www.wikidata.org/entity/Q1203",
      "_label": "John",
      "date_of_birth": {
        "@type": "date",
        "@value": "1940-10-09"
      }
    }
  ]
}
```

</details>
