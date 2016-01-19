# [Core API](http://www.coreapi.org/)

**Hypermedia driven Web APIs.**

---

Core API is a specification for creating Hypermedia driven Web APIs, that allows you to build rich, expressive, and meaningful interfaces.

* **Robust** - Clients interacting with a Core API service always have the available interactions presented to them. This allows for generic client libraries that are always automatically up to date with the services they interact with.
* **Expressive** - Documents may be nested, and support in-place transitions, allowing you to express rich and complex interfaces without having to make multiple network calls.
* **Explorable** - Client libraries allow you to inspect and interact with a Core API interface, and the HTML encoding allows for fully web browsable APIs.
* **Flexible** - Rather than be coupled to a particular encoding, Core API separates out the document, encoding and transport concerns. This means that client libraries can communicate with a number of different hypermedia formats. Equally, Core API services can make themselves available over a number of different encodings.

#### Example services

You can interact with these example services either directly through your browser, by installing the command-line client, or by using one of the client libraries.

* **Notes** - Create, update and delete items from a list of notes. [http://notes.coreapi.org/](http://notes.coreapi.org/)
* **Game** - Find the treasure in 5 turns or less. [http://game.coreapi.org/](http://game.coreapi.org/)

For example, let's try interacting with the "Game" service using the command line client.

First make sure to [install Python](https://www.python.org/downloads/), then...

```bash
    $ pip install coreapi  # Use Python's package manager `pip` to install the command-line client.
    $ coreapi get http://game.coreapi.org/
    <Home "http://game.coreapi.org/">
        new_game()
    $ coreapi action new_game
    <Game "http://game.coreapi.org/dee07521-6862-4744-95ec-812db90135bd">
        board: "...a\n...b\n...c\n123"
        description: "5 turns remaining."
        new_game()
        play([position])
    $ coreapi action play --param position=b3
    <Game "http://game.coreapi.org/dee07521-6862-4744-95ec-812db90135bd">
        board: "...a\n..xb\n...c\n123"
        description: "4 turns remaining."
        new_game()
        play([position])
```

#### Tooling

* A [command line client][command-line-client] for interacting with services from the console.
* There is a complete [Python client library][python-client] for Core API.
* A [Javascript client library][javascript-client] is currently planned.
* We have an [example server implementation][example-server], for demonstration purposes.

#### Discussion

For news and updates follow [@core-api](https://twitter.com/core_api), or [@_tomchristie](https://twitter.com/_tomchristie).

For discussion of the tools and specification, use the [Hypermedia Web mailing list](https://groups.google.com/forum/#!forum/hypermedia-web).

#### What does it look like?

Core API has a lightweight JSON encoding called 'Core JSON'. For example:

    {
        "_type": "document",
        "_meta": {
            "url": "/",
            "title": "Notes"
        },
        "notes": [
            {
                "_type": "document",
                "_meta": {
                    "url": "/1de153fe-6747-41d3-bc0e-d9d7d87e448a",
                    "title": "Note"
                },
                "complete": false,
                "description": "Email venue about conference dates",
                "delete": {
                    "_type": "link",
                    "action": "delete"
                },
                "edit": {
                    "_type": "link",
                    "action": "put",
                    "fields": [
                        {
                            "name": "description"
                        }, {
                            "name": "complete"
                        }
                    ]
                }
            }
        ],
        "add_note": {
            "_type": "link",
            "action": "post",
            "fields": [
                {
                    "name": "description",
                    "required": true
                }
            ]
        }
    }

Additionally, an HTML based encoding is defined. This allows servers to present APIs that can be interacted with directly from a Web browser. The previous example rendered in HTML looks like this:

![HTML encoding example](http://www.coreapi.org/images/html-encoding.png)

---

## Overview

There are three layers to the Core API specification.

Name               |   Description
------------------- | ---------------------
[Document layer](http://www.coreapi.org/specification/document/) | The abstract object interface that clients interact with.
[Encoding layer](http://www.coreapi.org/specification/encoding/) | The mapping between a Document and a byte string.
[Transport layer](http://www.coreapi.org/specification/transport/) | How document interactions are mapped to network requests.

The following is an overview of the document layer, describing how the client interacts with a Core API interface.

#### Document

Documents are the basic building blocks of Core API.

Documents are key-value pairs that contain the data and actions presented by the interface. Documents always have an associated URL, and should also have a title.

The top level element in any Core API interface is always a Document.

Let's take a look at a Core API document by using the Python client library.

    $ pip install coreapi
    $ python
    >>> import coreapi
    >>> doc = coreapi.get('http://notes.coreapi.org/')
    >>> print(doc)
    <Notes "http://notes.coreapi.org/">
        notes: [
            <Note "http://notes.coreapi.org/1f1bc8d6-9411-48d9-a2fc-6d8b82a48888">
                complete: true
                description: "Write command line client"
                delete()
                edit([description], [complete]),
            <Note "http://notes.coreapi.org/123d4e35-cb09-40c3-98d3-d119e9079fca">
                complete: true
                description: "Example"
                delete()
                edit([description], [complete]),
            <Note "http://notes.coreapi.org/0ace0b3b-9db2-4c10-989e-76c5c61265e7">
                complete: true
                description: "Fix the kitchen door"
                delete()
                edit([description], [complete])
        ]
        add_note(description)

We've got a document here that contains a couple of other nested documents. We can also see the actions and data that the interface exposes.

#### Links

Links are the available points of interaction that the interface presents.

Links have an associated URL and action, and may accept a set of named parameters.

Links inside nested documents may optionally be marked as an "in-place" transition.
Links that are marked as in-place effect a partial transformation on the document,
modifying or removing the nested document from the document tree.

The 'put', 'patch' and 'delete' actions default to being in-place.

Let's return to the python client library, and take a look at calling some links.
We'll start by removing all the existing notes:

    >>> while doc["notes"]:
    >>>     doc = coreapi.action(doc, ['notes', 0, 'delete'])

Calling `.action()` effects a transition on the given link, and returns a
new document instance.

There should now be no notes remaining:

    >>> print(doc)
    <Notes "http://notes.coreapi.org/">
        notes: []
        add_note(description)

Okay, let's create a new note:

    >>> doc = coreapi.action(doc, 'add_note', params={'description': 'Email venue about conference dates'})
    >>> print(doc)
    <Notes "http://notes.coreapi.org/">
        notes: [
            <Note "http://notes.coreapi.org/e7785f34-2b74-41d2-ab3f-f754f688987c/">
                complete: false
                description: "Email venue about conference dates"
                delete()
                edit([description], [complete])
        ]
        add_note(description)

Finally we'll update the state of the note we've just created:

    >>> doc = coreapi.action(doc, ['notes', 0, 'edit'], params={'complete': True})
    >>> print(doc)
    <Notes "http://notes.coreapi.org/">
        notes: [
            <Note "http://notes.coreapi.org/e7785f34-2b74-41d2-ab3f-f754f688987c/">
                complete: true
                description: "Email venue about conference dates"
                delete()
                edit([description], [complete])
        ]
        add_note(description)

#### Data primitives

Data primitives are the set of basic datatypes that may be used to represent data in the interface.

Core API supports the same subset of data primitives as JSON. These are Object, Array, String, Integer, Number, `true`, `false`, and `null`.

    >>> doc['notes'][0]['description']
    'Email venue about conference dates'
    >>> doc['notes'][0]['complete']
    True

#### Errors

Following a link may result in an error. An error is a set of key-value pairs, which is used to represent any error information associated with the failed transition.

Encountering an error prevents any transition from taking place, and will normally be represented by an exception or other error status by the client library.

    >>> coreapi.action(doc, 'add_note', params={'description': 'x' * 999999})
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    coreapi.exceptions.ErrorMessage: {"description": ["Ensure this parameter has no more than 100 characters."]}

---

## Design considerations

Because Core API documents are composable there are a few design decisions you'll want to be aware of, that may not exist in other API systems.

#### Rich interfaces

The status quo with many Web APIs today is to have a close relationship between items in the storage backend and API endpoints. While Core API *does* allow you to build collection type APIs it can also express richer interfaces.

You probably want to think of a Core API interface as being similar to a web page, but without any layout or style information. You can embed multiple controls and related elements inside a single document.

#### Actions should effect child elements

When in-place link transitions are followed they will only ever update the part of the document that the link is contained by. You should make sure that links are always included at the topmost level of any elements they might effect.

Failing to follow this constraint will mean that clients may need to have some implicit knowledge about which other parts of a document to reload once a transition takes place. This introduces coupling, and increases the number of required network requests.

#### Interlinking vs nesting

When building a Core API interface you'll need to decide on when to link to associated document, and when to nest an associated document. The former implies a necessary network request to retrieve the document, while the later embeds the document directly.

Normally this decision should be self-evident, and will depend on the type of the relationship that the link expresses.


[command-line-client]: http://www.coreapi.org/tools-and-resources/command-line-client/
[python-client]: https://github.com/core-api/python-client
[javascript-client]: https://github.com/core-api/javascript-client
[example-server]: https://github.com/core-api/example-server
