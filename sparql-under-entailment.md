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
<http://www.example.com/Room> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.w3.org/2000/01/rdf-schema#Class> .
<http://www.example.com/Room> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.w3.org/2002/07/owl#Class> .
<http://www.example.com/Room> <http://www.w3.org/2000/01/rdf-schema#label> "Room" .
<http://www.example.com/Room> <http://www.w3.org/2000/01/rdf-schema#comment> "A hospital room" .

<http://www.example.com/Patient> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.w3.org/2000/01/rdf-schema#Class> .
<http://www.example.com/Patient> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.w3.org/2002/07/owl#Class> .
<http://www.example.com/Patient> <http://www.w3.org/2000/01/rdf-schema#label> "Patient" .
<http://www.example.com/Patient> <http://www.w3.org/2000/01/rdf-schema#comment> "A hospital patient" .
  
<http://www.example.com/HighRiskPatient> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.w3.org/2000/01/rdf-schema#Class> .
<http://www.example.com/HighRiskPatient> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.w3.org/2002/07/owl#Class> .
<http://www.example.com/HighRiskPatient> <http://www.w3.org/2000/01/rdf-schema#label> "High Risk Patient" .
<http://www.example.com/HighRiskPatient> <http://www.w3.org/2000/01/rdf-schema#subClassOf> <http://www.example.com/Patient> .
<http://www.example.com/HighRiskPatient> <http://www.w3.org/2000/01/rdf-schema#comment> "Patient with elevated risk factors" .

<http://www.example.com/Nurse> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.w3.org/2000/01/rdf-schema#Class> .
<http://www.example.com/Nurse> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.w3.org/2002/07/owl#Class> .
<http://www.example.com/Nurse> <http://www.w3.org/2000/01/rdf-schema#label> "Nurse" .
<http://www.example.com/Nurse> <http://www.w3.org/2000/01/rdf-schema#comment> "A healthcare nurse" .

<http://www.example.com/locatedIn> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.w3.org/1999/02/22-rdf-syntax-ns#Property> .
<http://www.example.com/locatedIn> <http://www.w3.org/2000/01/rdf-schema#label> "located in" .
<http://www.example.com/locatedIn> <http://www.w3.org/2000/01/rdf-schema#domain> <http://www.example.com/Patient> .
<http://www.example.com/locatedIn> <http://www.w3.org/2000/01/rdf-schema#range> <http://www.example.com/Room> .
<http://www.example.com/locatedIn> <http://www.w3.org/2000/01/rdf-schema#comment> "Patient is located in a room" .

<http://www.example.com/assignedTo> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.w3.org/1999/02/22-rdf-syntax-ns#Property> .
<http://www.example.com/assignedTo> <http://www.w3.org/2000/01/rdf-schema#label> "assigned to" .
<http://www.example.com/assignedTo> <http://www.w3.org/2000/01/rdf-schema#domain> <http://www.example.com/Patient> .
<http://www.example.com/assignedTo> <http://www.w3.org/2000/01/rdf-schema#range> <http://www.example.com/Nurse> .
<http://www.example.com/assignedTo> <http://www.w3.org/2000/01/rdf-schema#comment> "Patient is assigned to a nurse" .

<http://www.example.com/patient1> <http://www.example.com/assignedTo> <http://www.example.com/nurse1> .
<http://www.example.com/patient2> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.example.com/HighRiskPatient> .  
<http://www.example.com/patient3> <http://www.example.com/locatedIn> <http://www.example.com/room1> .
```

---

## The Challenge: Querying with Class Hierarchies and domain/range definitions


**Question:** "Find all patients"

Note that there are no explicit patient definitions.


---

## Exercise: Query With Reasoning

### What the Reasoner Infers

When RDFS reasoning is enabled, the reasoner automatically infers additional triples based on rdfs:subClassOf and rdfs:domain / rdfs:range statements:

**From the ontology:**
```turtle
example:HighRiskPatient rdfs:subClassOf example:Patient .

example:assignedTo rdfs:domain example:Patient .
example:assignedTo rdfs:range example:Nurse .

example:locatedIn rdfs:domain example:Patient .
example:locatedIn rdfs:range example:Room .
```
And data:
```
example:patient1 example:assignedTo> example:nurse1 .
example:patient2 a example:HighRiskPatient .  
example:patient3 example:locatedIn example:room1 .
```

### Your Task

Now write RDFS rules to support subClassOf, domain and range definitions.


```sparql
PREFIX example: <http://www.example.com/>

SELECT *
WHERE {
  ?patient a example:Patient.
}
```
Rules in Kolibrie follow the syntex:
```sparql
RULE :SomeRuleName:- 
CONSTRUCT { 
   ?s a :Head.

}
WHERE { 
   ?s a :Body.
}
```
As a starting point, the rule in Kolibrie syntax to write the subclassOf rule is
```sparql
RULE :RDFS9:- 
CONSTRUCT { 
   ?p a ?s.

}
WHERE { 
   ?c <http://www.w3.org/2000/01/rdf-schema#subClassOf> ?s.
   ?p <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> ?c .
}
```

