---
layout: post
title: Semantic Web - OWL, RDF and related tech
comments: true
---

What is Semantic Web?

Idea by Tim Berners Lee (father of WWW): instead of making connections between
documents, lets do connections (hyperlinks) between data points.

Concepts:

* [Resource Description Framework (RDF)][1] - spec of an abstract data model.
  It is based on the idea of making statements about resources (in particular:
  web resources) in the form of triples: subject-predicate-object. 

  "The subject denotes the resource, and the predicate denotes traits or
  aspects of the resource, and expresses a relationship between the subject and
  the object."

  * Subjects, predicates and objects all have unique IDs (URIs).
  * RDF an be written in XML (there is a spec for that).
  * There is no domain knowledge - there are just triples.
  * Idea was that every institution would publish their own data in RDF and
    people could just link between them. The URI is always used to create the
    link. You can easily add a concept of "sameAs" - you can map the same
    properties/objects from different databases to each other.

* [Ontology][2] - Web Ontology Language. Allows to create a domain of knowledge
  and model concepts within the domain and relationships between the concepts.
  Key challenge: how to understand whether the same words mean the same things
  for different people.

  "An ontology is a formal, explicit specification of shared
  conceptualization."

  * Has classes - which are concepts
  * Has properties - which link those concepts
  * Has individuals - instances of classes
  * Has restrictions - e.g. cardinality

* How OWL is connected to RDF?
  * 

SPARQL - 

Links:
* http://owl.cs.manchester.ac.uk/tools/list-of-reasoners/ - list of reasoning
  software
* https://www.stardog.com/ - company once behind Pellet
* https://protege.stanford.edu/ - tool with a nice GUI for managing and
  developing ontologies

[1]: https://en.wikipedia.org/wiki/Resource_Description_Framework
[2]: https://en.wikipedia.org/wiki/Ontology_(information_science)
[3]: https://www.youtube.com/watch?v=zteyEk9LADs
