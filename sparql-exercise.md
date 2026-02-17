# SPARQL Query Exercises

## Setup

For these exercises, you'll write SPARQL queries against the RDF data you generated in the RML mapping exercises.

**Tool:** [SPARQL Playground](https://atomgraph.github.io/SPARQL-Playground/) or [Kolibrie's UI](https://github.com/StreamIntelligenceLab/Kolibrie)

**Goal:** Learn how to query integrated data across multiple sources using SPARQL, discovering insights that span patient and room information.

---

## Prerequisites

Before starting these exercises, make sure you have:

1. Completed the RML mapping exercises (or use the provided solution)
2. Generated RDF output from both patient and room data
3. The RDF data loaded in the Matey tool (or another SPARQL endpoint)

OR

Use the following dataset:
```
<http://www.example.com/patient_MRN-7834521> <http://www.example.com/assignedTo> <http://www.example.com/nurse_NURSE-102> .
<http://www.example.com/patient_MRN-7834521> <http://www.example.com/hasFallRiskScore> "85"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/patient_MRN-7834521> <http://www.example.com/hasID> "MRN-7834521" .
<http://www.example.com/patient_MRN-7834521> <http://www.example.com/hasName> "Maria Van den Berg" .
<http://www.example.com/patient_MRN-7834521> <http://www.example.com/locatedIn> <http://www.example.com/room_3-WEST-214> .
<http://www.example.com/patient_MRN-7834521> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.example.com/Patient> .
<http://www.example.com/patient_MRN-7834522> <http://www.example.com/assignedTo> <http://www.example.com/nurse_NURSE-102> .
<http://www.example.com/patient_MRN-7834522> <http://www.example.com/hasFallRiskScore> "45"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/patient_MRN-7834522> <http://www.example.com/hasID> "MRN-7834522" .
<http://www.example.com/patient_MRN-7834522> <http://www.example.com/hasName> "Jan Vermeulen" .
<http://www.example.com/patient_MRN-7834522> <http://www.example.com/locatedIn> <http://www.example.com/room_3-WEST-215> .
<http://www.example.com/patient_MRN-7834522> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.example.com/Patient> .
<http://www.example.com/patient_MRN-7834523> <http://www.example.com/assignedTo> <http://www.example.com/nurse_NURSE-103> .
<http://www.example.com/patient_MRN-7834523> <http://www.example.com/hasFallRiskScore> "92"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/patient_MRN-7834523> <http://www.example.com/hasID> "MRN-7834523" .
<http://www.example.com/patient_MRN-7834523> <http://www.example.com/hasName> "Sophie Dubois" .
<http://www.example.com/patient_MRN-7834523> <http://www.example.com/locatedIn> <http://www.example.com/room_ICU-3W-216> .
<http://www.example.com/patient_MRN-7834523> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.example.com/Patient> .
<http://www.example.com/patient_MRN-7834524> <http://www.example.com/assignedTo> <http://www.example.com/nurse_NURSE-104> .
<http://www.example.com/patient_MRN-7834524> <http://www.example.com/hasFallRiskScore> "38"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/patient_MRN-7834524> <http://www.example.com/hasID> "MRN-7834524" .
<http://www.example.com/patient_MRN-7834524> <http://www.example.com/hasName> "Lucas Peeters" .
<http://www.example.com/patient_MRN-7834524> <http://www.example.com/locatedIn> <http://www.example.com/room_2-EAST-101> .
<http://www.example.com/patient_MRN-7834524> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.example.com/Patient> .
<http://www.example.com/patient_MRN-7834525> <http://www.example.com/assignedTo> <http://www.example.com/nurse_NURSE-103> .
<http://www.example.com/patient_MRN-7834525> <http://www.example.com/hasFallRiskScore> "88"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/patient_MRN-7834525> <http://www.example.com/hasID> "MRN-7834525" .
<http://www.example.com/patient_MRN-7834525> <http://www.example.com/hasName> "Emma Janssens" .
<http://www.example.com/patient_MRN-7834525> <http://www.example.com/locatedIn> <http://www.example.com/room_ICU-3W-217> .
<http://www.example.com/patient_MRN-7834525> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.example.com/Patient> .
<http://www.example.com/room_2-EAST-101> <http://www.example.com/hasCapacity> "2"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_2-EAST-101> <http://www.example.com/hasHVACStatus> "operational" .
<http://www.example.com/room_2-EAST-101> <http://www.example.com/hasHumidity> "60"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_2-EAST-101> <http://www.example.com/hasRoomID> "2-EAST-101" .
<http://www.example.com/room_2-EAST-101> <http://www.example.com/hasTemperature> "21.2"^^<http://www.w3.org/2001/XMLSchema#decimal> .
<http://www.example.com/room_2-EAST-101> <http://www.example.com/hasZone> <http://www.example.com/zone_2-EAST> .
<http://www.example.com/room_2-EAST-101> <http://www.example.com/lastUpdatedMinutesAgo> "15"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_2-EAST-101> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.example.com/GeneralRoom> .
<http://www.example.com/room_3-WEST-214> <http://www.example.com/hasCapacity> "2"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_3-WEST-214> <http://www.example.com/hasHVACStatus> "operational" .
<http://www.example.com/room_3-WEST-214> <http://www.example.com/hasHumidity> "72"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_3-WEST-214> <http://www.example.com/hasRoomID> "3-WEST-214" .
<http://www.example.com/room_3-WEST-214> <http://www.example.com/hasTemperature> "21.5"^^<http://www.w3.org/2001/XMLSchema#decimal> .
<http://www.example.com/room_3-WEST-214> <http://www.example.com/hasZone> <http://www.example.com/zone_3-WEST> .
<http://www.example.com/room_3-WEST-214> <http://www.example.com/lastUpdatedMinutesAgo> "15"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_3-WEST-214> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.example.com/GeneralRoom> .
<http://www.example.com/room_3-WEST-215> <http://www.example.com/hasCapacity> "1"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_3-WEST-215> <http://www.example.com/hasHVACStatus> "operational" .
<http://www.example.com/room_3-WEST-215> <http://www.example.com/hasHumidity> "58"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_3-WEST-215> <http://www.example.com/hasRoomID> "3-WEST-215" .
<http://www.example.com/room_3-WEST-215> <http://www.example.com/hasTemperature> "22.0"^^<http://www.w3.org/2001/XMLSchema#decimal> .
<http://www.example.com/room_3-WEST-215> <http://www.example.com/hasZone> <http://www.example.com/zone_3-WEST> .
<http://www.example.com/room_3-WEST-215> <http://www.example.com/lastUpdatedMinutesAgo> "15"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_3-WEST-215> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.example.com/GeneralRoom> .
<http://www.example.com/room_3-WEST-220> <http://www.example.com/hasCapacity> "1"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_3-WEST-220> <http://www.example.com/hasHVACStatus> "maintenance_required" .
<http://www.example.com/room_3-WEST-220> <http://www.example.com/hasHumidity> "65"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_3-WEST-220> <http://www.example.com/hasRoomID> "3-WEST-220" .
<http://www.example.com/room_3-WEST-220> <http://www.example.com/hasTemperature> "20.5"^^<http://www.w3.org/2001/XMLSchema#decimal> .
<http://www.example.com/room_3-WEST-220> <http://www.example.com/hasZone> <http://www.example.com/zone_3-WEST> .
<http://www.example.com/room_3-WEST-220> <http://www.example.com/lastUpdatedMinutesAgo> "120"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_3-WEST-220> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.example.com/AirborneIsolationRoom> .
<http://www.example.com/room_ICU-3W-216> <http://www.example.com/hasCapacity> "1"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_ICU-3W-216> <http://www.example.com/hasHVACStatus> "operational" .
<http://www.example.com/room_ICU-3W-216> <http://www.example.com/hasHumidity> "55"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_ICU-3W-216> <http://www.example.com/hasRoomID> "ICU-3W-216" .
<http://www.example.com/room_ICU-3W-216> <http://www.example.com/hasTemperature> "20.8"^^<http://www.w3.org/2001/XMLSchema#decimal> .
<http://www.example.com/room_ICU-3W-216> <http://www.example.com/hasZone> <http://www.example.com/zone_ICU-3W> .
<http://www.example.com/room_ICU-3W-216> <http://www.example.com/lastUpdatedMinutesAgo> "15"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_ICU-3W-216> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.example.com/ContactIsolationRoom> .
<http://www.example.com/room_ICU-3W-217> <http://www.example.com/hasCapacity> "1"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_ICU-3W-217> <http://www.example.com/hasHVACStatus> "operational" .
<http://www.example.com/room_ICU-3W-217> <http://www.example.com/hasHumidity> "70"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_ICU-3W-217> <http://www.example.com/hasRoomID> "ICU-3W-217" .
<http://www.example.com/room_ICU-3W-217> <http://www.example.com/hasTemperature> "21.0"^^<http://www.w3.org/2001/XMLSchema#decimal> .
<http://www.example.com/room_ICU-3W-217> <http://www.example.com/hasZone> <http://www.example.com/zone_ICU-3W> .
<http://www.example.com/room_ICU-3W-217> <http://www.example.com/lastUpdatedMinutesAgo> "15"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_ICU-3W-217> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.example.com/AirborneIsolationRoom> .
<http://www.example.com/room_ICU-3W-218> <http://www.example.com/hasCapacity> "1"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_ICU-3W-218> <http://www.example.com/hasHVACStatus> "operational" .
<http://www.example.com/room_ICU-3W-218> <http://www.example.com/hasHumidity> "62"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_ICU-3W-218> <http://www.example.com/hasRoomID> "ICU-3W-218" .
<http://www.example.com/room_ICU-3W-218> <http://www.example.com/hasTemperature> "21.8"^^<http://www.w3.org/2001/XMLSchema#decimal> .
<http://www.example.com/room_ICU-3W-218> <http://www.example.com/hasZone> <http://www.example.com/zone_ICU-3W> .
<http://www.example.com/room_ICU-3W-218> <http://www.example.com/lastUpdatedMinutesAgo> "15"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_ICU-3W-218> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.example.com/GeneralRoom> .
```

---

## Background: The Data

From the RML exercises, we have RDF data representing:

**Patients:**
- 5 patients with names, IDs, fall risk scores (38-92), room locations, and assigned nurses

**Rooms:**
- 7 rooms with types (GeneralRoom, ContactIsolationRoom, AirborneIsolationRoom)
- Environmental data: temperature, humidity, HVAC status
- Last update timing

**The Connection:**
Patients are linked to rooms via the `example:locatedIn` property.

---

## Exercise 1: Basic Patient Query

### Task

Write a SPARQL query to find **all patients with their names and fall risk scores**.

### Starter Template

```sparql
PREFIX example: <http://www.example.com/>

SELECT ?patient ?name ?riskScore
WHERE {
  ?patient a example:Patient.
}
```

### Hints

- Patients have type `example:Patient`
- Use `example:hasName` for the patient name
- Use `example:hasFallRiskScore` for the risk score

### Expected Output

You should see 5 patients with their names and risk scores.

---

## Exercise 2: Filter High-Risk Patients

### Task

Find **patients with fall risk scores above 80**.

### Your Task

Modify the query from Exercise 1 to filter only high-risk patients.

### Hints

- Use the `FILTER` keyword
- Check if `?riskScore > 80`

### Expected Output

You should see 3 patients: Maria (85), Sophie (92), and Emma (88).

---

## Exercise 3: Join Patient and Room Data

### Task

Write a query to find **all patients with their room information** (room ID, room type, temperature, and humidity).

### Starter Template

```sparql
PREFIX example: <http://www.example.com/>

SELECT ?patientName ?roomID ?roomType ?temperature ?humidity
WHERE {
  ?patient a example:Patient ;
           example:hasName ?patientName ;
           example:locatedIn ?room .
}
```
 TODO: Add triple patterns for room information
### Hints

- The patient is already linked to the room via `example:locatedIn`
- Rooms have properties: `example:hasRoomID`, `example:hasTemperature`, `example:hasHumidity`
- For room type, use: `?room a ?roomType`
- You might want to filter out `example:Room` and only show specific types

### Expected Output

You should see patients matched with their room details.

---

## Exercise 4: High Fall Risk in High Humidity Rooms 

### Task

This is the main challenge! Find **patients with fall risk scores above 80 who are located in rooms with humidity above 60%**.

This combines:
- Patient risk assessment (from EMR data)
- Environmental conditions (from BMS data)

### Why This Matters

High humidity can affect patient comfort and may contribute to:
- Increased sweating and discomfort
- Potential for slips on damp floors
- Respiratory issues for some patients

Combining high fall risk with poor environmental conditions helps identify patients who need immediate attention.

### Starter Template

```sparql
PREFIX example: <http://www.example.com/>

SELECT ?patientName ?riskScore ?roomID ?humidity
WHERE {
  # Patient information
  ?patient a example:Patient ;
           example:hasName ?patientName ;
           example:hasFallRiskScore ?riskScore ;
           example:locatedIn ?room .
  
  # Room information
  # TODO: Add room properties
  
  # Filters
  # TODO: Filter for high fall risk (> 80)
  # TODO: Filter for high humidity (> 60)
}
ORDER BY DESC(?riskScore)
```

### Expected Output

You should see **2 patients**:
- **Maria** (risk: 85) in room 3-WEST-214 (humidity: 72%)
- **Emma** (risk: 88) in room ICU-3W-217 (humidity: 70%)

Both patients have high fall risk AND are in rooms with humidity above the 40-60% specification range.



## Resources

- **SPARQL 1.1 Specification**: https://www.w3.org/TR/sparql11-query/
- **SPARQL Tutorial**: https://www.w3.org/2009/Talks/0615-qbe/
- **SPARQL by Example**: https://www.w3.org/TR/sparql11-query/#examples
- **Interactive SPARQL Tutorial**: https://www.wikidata.org/wiki/Wikidata:SPARQL_tutorial

---

Good luck with your queries! 
