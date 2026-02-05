# SPARQL with RDFS Entailment - Reasoning Exercises

## Introduction

In previous exercises, we wrote SPARQL queries against explicit RDF triples. But one of the most powerful features of semantic technologies is **reasoning** — the ability to infer new knowledge from existing data using logical rules.

**RDFS (RDF Schema)** provides basic reasoning capabilities through:
- **Class hierarchies** (`rdfs:subClassOf`) 
- **Property domains and ranges** (`rdfs:domain`, `rdfs:range`)
- **Property hierarchies** (`rdfs:subPropertyOf`)

This exercise demonstrates how RDFS reasoning makes queries simpler and more powerful.

---

## Setup: The Ontology

First, we need to define our ontology with RDFS statements that describe the structure of our domain.

### ontology.ttl

```turtle
@prefix example: <http://www.example.com/> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .

# ============================================
# Class Hierarchy
# ============================================

# Base classes
example:Room a rdfs:Class, owl:Class ;
    rdfs:label "Room" ;
    rdfs:comment "A hospital room" .

example:Patient a rdfs:Class, owl:Class ;
    rdfs:label "Patient" ;
    rdfs:comment "A hospital patient" .

example:Nurse a rdfs:Class, owl:Class ;
    rdfs:label "Nurse" ;
    rdfs:comment "A healthcare nurse" .

# Room type hierarchy
example:IsolationRoom a rdfs:Class, owl:Class ;
    rdfs:label "Isolation Room" ;
    rdfs:subClassOf example:Room ;
    rdfs:comment "A room for infection control isolation" .

example:AirborneIsolationRoom a rdfs:Class, owl:Class ;
    rdfs:label "Airborne Isolation Room" ;
    rdfs:subClassOf example:IsolationRoom ;
    rdfs:comment "Negative pressure room for airborne infections" .

example:ContactIsolationRoom a rdfs:Class, owl:Class ;
    rdfs:label "Contact Isolation Room" ;
    rdfs:subClassOf example:IsolationRoom ;
    rdfs:comment "Room for contact-transmitted infections" .

example:GeneralRoom a rdfs:Class, owl:Class ;
    rdfs:label "General Room" ;
    rdfs:subClassOf example:Room ;
    rdfs:comment "Standard patient room" .

# Patient risk hierarchy
example:HighRiskPatient a rdfs:Class, owl:Class ;
    rdfs:label "High Risk Patient" ;
    rdfs:subClassOf example:Patient ;
    rdfs:comment "Patient with elevated risk factors" .

# ============================================
# Property Domains and Ranges
# ============================================

# Patient properties
example:hasFallRiskScore a rdf:Property ;
    rdfs:label "has fall risk score" ;
    rdfs:domain example:Patient ;
    rdfs:range rdfs:Literal ;
    rdfs:comment "Patient's fall risk score (0-100)" .

example:hasName a rdf:Property ;
    rdfs:label "has name" ;
    rdfs:domain example:Patient ;
    rdfs:range rdfs:Literal .

example:hasID a rdf:Property ;
    rdfs:label "has ID" ;
    rdfs:domain example:Patient ;
    rdfs:range rdfs:Literal .

# Room properties
example:hasHumidity a rdf:Property ;
    rdfs:label "has humidity" ;
    rdfs:domain example:Room ;
    rdfs:range rdfs:Literal ;
    rdfs:comment "Room humidity percentage" .

example:hasTemperature a rdf:Property ;
    rdfs:label "has temperature" ;
    rdfs:domain example:Room ;
    rdfs:range rdfs:Literal ;
    rdfs:comment "Room temperature in Celsius" .

example:hasCapacity a rdf:Property ;
    rdfs:label "has capacity" ;
    rdfs:domain example:Room ;
    rdfs:range rdfs:Literal .

example:hasRoomID a rdf:Property ;
    rdfs:label "has room ID" ;
    rdfs:domain example:Room ;
    rdfs:range rdfs:Literal .

# Relationships
example:locatedIn a rdf:Property ;
    rdfs:label "located in" ;
    rdfs:domain example:Patient ;
    rdfs:range example:Room ;
    rdfs:comment "Patient is located in a room" .

example:assignedTo a rdf:Property ;
    rdfs:label "assigned to" ;
    rdfs:domain example:Patient ;
    rdfs:range example:Nurse ;
    rdfs:comment "Patient is assigned to a nurse" .
```

---

## The Challenge: Querying with Class Hierarchies

Consider our room type hierarchy:

```
Room
├── IsolationRoom
│   ├── AirborneIsolationRoom (e.g., ICU-3W-217)
│   └── ContactIsolationRoom (e.g., ICU-3W-216)
└── GeneralRoom (e.g., 3-WEST-214)
```

**Question:** "Find all patients in isolation rooms"

This should include patients in:
- Airborne isolation rooms (Emma)
- Contact isolation rooms (Sophie)

---

## Exercise 1: Query Without Reasoning

### The Problem

Without reasoning, the RDF data contains ONLY explicit statements:
- `example:room_ICU-3W-217 a example:AirborneIsolationRoom`
- `example:room_ICU-3W-216 a example:ContactIsolationRoom`

There are NO explicit triples saying:
- `example:room_ICU-3W-217 a example:IsolationRoom`
- `example:room_ICU-3W-216 a example:IsolationRoom`

### Your Task

Write a SPARQL query to find all patients in isolation rooms **WITHOUT using reasoning**.

### Starter Template

```sparql
PREFIX example: <http://www.example.com/>

SELECT ?patientName ?roomID ?roomType
WHERE {
  ?patient a example:Patient ;
           example:hasName ?patientName ;
           example:locatedIn ?room .
  
  ?room example:hasRoomID ?roomID ;
        a ?roomType .
  
  # TODO: How do you check if the room is an IsolationRoom
  #       when you only have explicit AirborneIsolationRoom 
  #       and ContactIsolationRoom types?
}
```

### Hint

You must explicitly list BOTH subclass types using `FILTER` or `VALUES`.

---

## Exercise 2: Query With Reasoning

### What the Reasoner Infers

When RDFS reasoning is enabled, the reasoner automatically infers additional triples based on `rdfs:subClassOf` statements:

**From the ontology:**
```turtle
example:AirborneIsolationRoom rdfs:subClassOf example:IsolationRoom .
example:ContactIsolationRoom rdfs:subClassOf example:IsolationRoom .
example:IsolationRoom rdfs:subClassOf example:Room .
```

**Reasoner infers:**
```turtle
# Since room_ICU-3W-217 is an AirborneIsolationRoom...
example:room_ICU-3W-217 a example:AirborneIsolationRoom .  # explicit
example:room_ICU-3W-217 a example:IsolationRoom .          # inferred!
example:room_ICU-3W-217 a example:Room .                   # inferred!

# Since room_ICU-3W-216 is a ContactIsolationRoom...
example:room_ICU-3W-216 a example:ContactIsolationRoom .   # explicit
example:room_ICU-3W-216 a example:IsolationRoom .          # inferred!
example:room_ICU-3W-216 a example:Room .                   # inferred!
```

### Your Task

Now write the same query **with reasoning enabled** (assuming your SPARQL endpoint supports RDFS entailment).

```sparql
PREFIX example: <http://www.example.com/>

SELECT ?patientName ?roomID ?roomType
WHERE {
  ?patient a example:Patient ;
           example:hasName ?patientName ;
           example:locatedIn ?room .
  
  ?room example:hasRoomID ?roomID ;
        a example:IsolationRoom ;  # Simple! Just check for the parent class
        a ?roomType .
  
  FILTER (?roomType != example:Room && ?roomType != example:IsolationRoom)
}
```

### Compare the Queries

**Without Reasoning:**
```sparql
FILTER (?roomType = example:AirborneIsolationRoom || 
        ?roomType = example:ContactIsolationRoom)
```

**With Reasoning:**
```sparql
?room a example:IsolationRoom .
```

Much simpler! And if we add a NEW isolation type later (e.g., `RadiationIsolationRoom`), the query still works without modification.
