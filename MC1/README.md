# KGBuilder-MC1
Speed Presentation

## Table of Contents

1. [Problem & Use Cases](#1-problem--use-cases)
2. [Proposed Schema Elements](#2-proposed-schema-elements)
   - [Conceptual Model](#conceptual-model)
   - [Ontology](#ontology) ([ttl](ontology.ttl))
3. [Identifier Strategy](#3-identifier-strategy)
   - [Unique Identifiers](#unique-identifiers)
   - [Namespaces](#namespaces)
4. [Graph Representation Choice](#4-graph-representation-choice)
5. [Querying the KG](#5-querying-the-kg)
   - [Top 2 Contributors](#top-2-contributors)
   - [Dissenters of the DCA](#dissenters-of-the-dca)

----

## 1-Problem & Use Cases

### Problem 
Existing bookmarking tools like Pinterest and browser favorites save web links but fail to organize them around the ideas they support—whether insights, decisions, or hypotheses. Over time, I remember my conclusions but cannot easily trace which sources informed multiple ideas, identify authors, or recall dissenting perspectives, making it difficult to reconstruct my evidence-based thinking or recognize cross-referenced content.

I want to create a knowledge graph that helps me accomplish two main objectives:

1) organize & annotate information in a way that builds up to how I view it supporting an idea, hypothesis or decision;
2) be able to start by formulating an idea, hypothesis or decision and build the supporting data as I progress.

#### Scenario/Industry

I've built a couple of application ontologies in the past, and I wanted to take a stab at something higher-level. I am looking to develop some pattern that can be applied across multiple industries. For this exercise, the scenarios I've chosen are: 

1. Decision-making - in software development, there is a strategy called Architectural Decision Records (ADRs) that are typically used in Git repositories that summarize decisions made by a team  that are hard to decipher by reading an issue - especially lengthy issues. As a new person to the repository, it's hard to understand who the voices of authority and influence are, what pieces of content can be ignored, etc. An ADR is a summary of all this content (example: https://github.com/ESIPFed/science-on-schema.org/tree/main/decisions)

2. Idea Generation - I've been interested in the Financial Independence, Retire Early movement. Over the past 10 years, I've listened to a ton of podcasts and read numerous blog articles. To form a set of beliefs about what I'm going to do and why. As times change, I wonder what are the impact to these beliefs as data changes (tax codes, investing options, etc). I envision A graph where for a given Idea, I can link back to the sources of data for that idea, who said them, etc. These ideas could be built on top of other ideas (Hence, my CognitiveEntity class subclassing from InformationObject). How do ideas or insights change over time?

### Use Cases

1. Who are the top 2 people that contributed to my belief in that the "4% rule" is an effective retirement withdrawal strategy?
2. What are all the decisions asserted by myself having any dissenters that supported the decision to invest in "data-centric architecture" that are stored in Github?

## 2-Proposed Schema Elements

### Conceptual Model
![Conceptual Model](KGC-MC1-conceptual-model.png)

----

#### Graphical Notation
![Graphical Notation](KGC-MC1-graphical-notation.png)

### Ontology

[Ontology (ttl)](ontology.ttl) - here is a Turtle file built using Protege. 

![Protege](protege.png)


## 3-Identifier Strategy

I envision the use of this ontology for applications that are quick and easy (similar to Pinterest). Required fields are minimal. You want to be able to save some URL to a podcast, write a quick note, and go. Further annotation can happen afterward. This direction helps inform identifier strategy.

### Unique Identifiers 

**Ontology** - I chose URLs to identify classes, properties and named individuals. I want to publish this ontology at a website so that it can be shared and re-used. Becuase of this, I chose **slash termination** so that I have the freedom to use the hash terminator in URLs to be able to direct users to certain aspects of a webpage for the element being addressed at its ontology URI. This gives me downstream flexibility in case the website has individual webpages for its ontological element. For example, the `https://schema.harborlight.tech/ontology/dikw/v0.1/Dissenter` webpage may have certain elements with named anchor tags for: description, conceptual drawing, examples of usage, etc. the hash will let me direct a browser to these individual elements in web space while the slash terminator is used in ontology space.

**Entities/Data** - Because I'm thinking data will be entered through some application, and the data seems to not be something publicly available in all cases, I think I'd lean towards using a URN structure like: `urn:harborlighttech:dikw:{uuid}`. With the `:url` property, web-accessible data can be accessed through this and then URL management (ex: link rot) is up to the user. For web addressability of these URNs, I could stand up an identifying resolver service that given a URN, redirects to a webpage for that resource.

### Namespaces 

I found a lot of similarity with PROV-O and could model most of what I had here with PROV, so I decided to use it as an upper-ontology for this work and extend my classes off of it. I need to investigate more the if some of my properties can be `rdfs:subPropertyOf` some of the relationships in PROV-O, but I feel like that decision can be made downstream, if appropriate to avoid causing any entanglements at the start.

One of my classes, `AgentRole`, had a time component, and I noticed that `prov:Activity` had properties `startedAtTime` & `endedAtTime`, but I was hesitant to commit `AgentRole` to a `prov:Activity`. This is becuase PROV-O seems to describe _the past_, and I didn't want to make that assumption about my `AgentRole` just yet. For example, I might not know when an `AgentRole` had ended. A user might not ever supply that information. So, I left `AgentRole` as a standalone and then re-used the Time Ontology's `time:Instant` class for my start and end date. 

## 4-Graph-Representation-Choice

I selected RDF so that it was easy to share a machine-readable version of my schema (ontology). One of the disadvantages is that if I need to refactor my schema because a class needs to be inserted in between existing classes, this seems easier in LPG space where I can add a property off of an existing property. In RDF, that requires reification. 

The tooling and visualization are easier in LPG space, but I feel more comfortable in RDF space for the ontology and data where the tooling isn't a deterrent. 

Now, becuase my domain is about ideas and hypothesis and decisions, there is some notion that cross-linking between individual user graphs might be useful, specifically for Dissenter class where it can specify a CognitiveEntity that `contraposes` an existing one. I would think, most often, the contraposition would be in another graph entirely. 

## 5-Querying-the-KG

Given my [Use Cases](#use-cases), here are SPARQL queries that answer these questions. For simplicity, I ignore my use of UUIDs for the URNs to make the queries more readable. I created sample data in this repository and tested at: [SPARQL Playground](https://atomgraph.github.io/SPARQL-Playground/)

### Top 2 contributors 
CQ: Who are the top 5 people that contributed to my belief in that the "4% rule" is an effective retirement withdrawal strategy?

Given, that `<urn:harborlighttech:dikw:four-percent-rule-is-effective>` is the subject URI for for my belief that the "4% rule" is effective,
and the topic of "Retirement Withdrawal Strategy" is represented by the URI `<urn:harborlighttech:dikw:retirement-withdrawal-strategy>`:

* [Sample data](top-2-people_sample-data.ttl)
  
```
PREFIX dikw: <https://schema.harborlight.tech/ontology/dikw/v0.1/>
PREFIX prov: <http://www.w3.org/ns/prov#>
PREFIX time: <http://www.w3.org/2006/time#>
SELECT DISTINCT ?person ?name (COUNT(?data) as ?num)
WHERE {
   VALUES ?idea { <urn:harborlighttech:dikw:four-percent-rule-is-effective> }
   ?idea a dikw:Idea .
   ?idea dikw:about <urn:harborlighttech:dikw:retirement-withdrawal-strategy> .
   ?idea dikw:supportedBy ?data .
   ?data a dikw:SupportingData .
   ?data dikw:attributedTo ?person .
   ?person a prov:Person .
   ?person dikw:name ?name .
}
GROUP BY ?person ?name
ORDER BY DESC(COUNT(?data))
LIMIT 2
```

### Dissenters of the DCA
CQ: What are all the decisions that are supported by the decision to invest in "data-centric architecture", that were asserted by myself, having any dissenters, whose supporting data is stored in Github?

Given, that `<urn:harborlighttech:dikw:adam-shepherd>` is the subject URI for myself,
   and `<urn:harborlighttech:dikw:invest-in-data-centricity>` is the subject URI for the decision to invest in the data-centric architecture, 
   and `<urn:harborlighttech:dikw:github>` is the subject URI for Github:

* [Sample data](dissenters-of-the-dca.ttl)

```
PREFIX dikw: <https://schema.harborlight.tech/ontology/dikw/v0.1/>
PREFIX prov: <http://www.w3.org/ns/prov#>
PREFIX time: <http://www.w3.org/2006/time#>
SELECT DISTINCT ?decision ?name ?dissenter_name ?differing_idea_name
WHERE {
   ?decision a dikw:Decision .
   ?decision dikw:assertedBy <urn:harborlighttech:dikw:adam-shepherd> .
   ?decision dikw:supportedBy <urn:harborlighttech:dikw:invest-in-data-centricity> .
   ?decision ^dikw:dissentsTo ?dissenter .
   ?decision dikw:supportedBy ?data .
   FILTER (?data != <urn:harborlighttech:dikw:invest-in-data-centricity>)
   ?data dikw:provider <urn:harborlighttech:dikw:github> .
   OPTIONAL { ?decision dikw:name ?name }
   ?dissenter dikw:performedBy/dikw:name ?dissenter_name .
   OPTIONAL {
    ?dissenter dikw:contraposes ?differing_idea .
    OPTIONAL { ?differing_idea dikw:name ?differing_idea_name }
   }
}
```


   
