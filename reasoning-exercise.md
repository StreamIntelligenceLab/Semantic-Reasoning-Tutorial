# Exercise: Defining Risk Classification Rules

## Introduction

In the previous exercises, you've learned how to:
- Map CSV data to RDF using RML
- Query integrated data with SPARQL
- Use RDFS reasoning for hierarchical queries

Now it's time to combine everything: **defining rules that classify patients based on data from multiple systems**.

This exercise demonstrates one of the most powerful aspects of semantic integration: **cross-system reasoning**.

---

## The Scenario

Your hospital wants to identify high-risk patients who are in suboptimal environmental conditions. You have data from three systems:

1. **EMR**: Patient demographics, fall risk scores
2. **Patient Monitoring**: Alarm counts, call button usage
3. **BMS**: Room humidity, HVAC status

The **goal**: Find high-risk patients in isolation rooms with poor environmental conditions.


---
## The Data (given)

We provide the resulting dataset containing EMR, Patient Monitoring and BMS data:

```
@prefix example: <http://www.example.com/> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .


example:patient_MRN-7834521 a example:Patient ;
    example:hasID "MRN-7834521" ;
    example:hasName "Maria Van den Berg" ;
    example:hasFallRiskScore 85 ;
    example:locatedIn example:room_3-WEST-214 ;
    example:assignedTo example:nurse_NURSE-102 .

example:patient_MRN-7834522 a example:Patient ;
    example:hasID "MRN-7834522" ;
    example:hasName "Jan Vermeulen" ;
    example:hasFallRiskScore 45 ;
    example:locatedIn example:room_3-WEST-215 ;
    example:assignedTo example:nurse_NURSE-102 .

example:patient_MRN-7834523 a example:Patient ;
    example:hasID "MRN-7834523" ;
    example:hasName "Sophie Dubois" ;
    example:hasFallRiskScore 92 ;
    example:locatedIn example:room_ICU-3W-216 ;
    example:assignedTo example:nurse_NURSE-103 .

example:patient_MRN-7834524 a example:Patient ;
    example:hasID "MRN-7834524" ;
    example:hasName "Lucas Peeters" ;
    example:hasFallRiskScore 38 ;
    example:locatedIn example:room_2-EAST-101 ;
    example:assignedTo example:nurse_NURSE-104 .

example:patient_MRN-7834525 a example:Patient ;
    example:hasID "MRN-7834525" ;
    example:hasName "Emma Janssens" ;
    example:hasFallRiskScore 88 ;
    example:locatedIn example:room_ICU-3W-217 ;
    example:assignedTo example:nurse_NURSE-103 .

example:patient_MRN-7834521 
    example:hasAlarmCount 3 ;
    example:hasCallCount 4 ;
    example:hasLastAlarmTime "2024-11-15T14:30:00"^^xsd:dateTime ;
    example:hasAvgHeartRate 78 ;
    example:hasAvgSpO2 96 ;
    example:hasMonitoringStatus "active" .

example:patient_MRN-7834522 
    example:hasAlarmCount 1 ;
    example:hasCallCount 1 ;
    example:hasLastAlarmTime "2024-11-15T08:15:00"^^xsd:dateTime ;
    example:hasAvgHeartRate 72 ;
    example:hasAvgSpO2 98 ;
    example:hasMonitoringStatus "active" .

example:patient_MRN-7834523 
    example:hasAlarmCount 7 ;
    example:hasCallCount 1 ;
    example:hasLastAlarmTime "2024-11-15T15:45:00"^^xsd:dateTime ;
    example:hasAvgHeartRate 88 ;
    example:hasAvgSpO2 94 ;
    example:hasMonitoringStatus "active" .

example:patient_MRN-7834524 
    example:hasAlarmCount 1 ;
    example:hasCallCount 1 ;
    example:hasLastAlarmTime "2024-11-14T22:10:00"^^xsd:dateTime ;
    example:hasAvgHeartRate 68 ;
    example:hasAvgSpO2 99 ;
    example:hasMonitoringStatus "active" .

example:patient_MRN-7834525 
    example:hasAlarmCount 2 ;
    example:hasCallCount 3 ;
    example:hasLastAlarmTime "2024-11-15T13:20:00"^^xsd:dateTime ;
    example:hasAvgHeartRate 82 ;
    example:hasAvgSpO2 95 ;
    example:hasMonitoringStatus "active" .


example:room_3-WEST-214 a example:GeneralRoom, example:Room ;
    example:hasRoomID "3-WEST-214" ;
    example:hasZone "3-WEST" ;
    example:hasCapacity 2 ;
    example:hasTemperature 21.5 ;
    example:hasHumidity 72 ;
    example:hasHVACStatus "operational" ;
    example:lastUpdatedMinutesAgo 15 .

example:room_3-WEST-215 a example:GeneralRoom, example:Room ;
    example:hasRoomID "3-WEST-215" ;
    example:hasZone "3-WEST" ;
    example:hasCapacity 1 ;
    example:hasTemperature 22.0 ;
    example:hasHumidity 58 ;
    example:hasHVACStatus "operational" ;
    example:lastUpdatedMinutesAgo 15 .

example:room_ICU-3W-216 a example:ContactIsolationRoom, example:IsolationRoom, example:Room ;
    example:hasRoomID "ICU-3W-216" ;
    example:hasZone "ICU-3W" ;
    example:hasCapacity 1 ;
    example:hasTemperature 20.8 ;
    example:hasHumidity 55 ;
    example:hasHVACStatus "operational" ;
    example:lastUpdatedMinutesAgo 15 .

example:room_2-EAST-101 a example:GeneralRoom, example:Room ;
    example:hasRoomID "2-EAST-101" ;
    example:hasZone "2-EAST" ;
    example:hasCapacity 2 ;
    example:hasTemperature 21.2 ;
    example:hasHumidity 60 ;
    example:hasHVACStatus "operational" ;
    example:lastUpdatedMinutesAgo 15 .

example:room_ICU-3W-217 a example:AirborneIsolationRoom, example:IsolationRoom, example:Room ;
    example:hasRoomID "ICU-3W-217" ;
    example:hasZone "ICU-3W" ;
    example:hasCapacity 1 ;
    example:hasTemperature 21.0 ;
    example:hasHumidity 70 ;
    example:hasHVACStatus "operational" ;
    example:lastUpdatedMinutesAgo 15 .

example:room_3-WEST-220 a example:AirborneIsolationRoom, example:IsolationRoom, example:Room ;
    example:hasRoomID "3-WEST-220" ;
    example:hasZone "3-WEST" ;
    example:hasCapacity 1 ;
    example:hasTemperature 20.5 ;
    example:hasHumidity 65 ;
    example:hasHVACStatus "maintenance_required" ;
    example:lastUpdatedMinutesAgo 120 .

example:room_ICU-3W-218 a example:GeneralRoom, example:Room ;
    example:hasRoomID "ICU-3W-218" ;
    example:hasZone "ICU-3W" ;
    example:hasCapacity 1 ;
    example:hasTemperature 21.8 ;
    example:hasHumidity 62 ;
    example:hasHVACStatus "operational" ;
    example:lastUpdatedMinutesAgo 15 .

```

---

## The Query (Given)

Here's the query the clinical team wants to run:

```sparql
PREFIX example: <http://www.example.com/>

SELECT ?patient ?name ?room ?humidity ?hvacStatus           
       ?alarmCount ?callCount                               
WHERE {                                                     
  ?patient a example:HighRiskPatient ;
           example:hasName ?name ;                                 
           example:locatedIn ?room ;                             
           example:hasAlarmCount ?alarmCount ;                     
           example:hasCallCount ?callCount .                       
  
  ?room a example:IsolationRoom ;
        example:hasHumidity ?humidity ;                            
        example:hasHVACStatus ?hvacStatus .                        
  
  FILTER (?humidity > 50)  
}
ORDER BY DESC(?alarmCount)
```

### What This Query Does

- Finds patients classified as **HighRiskPatient** (reasoning determines this!)
- Who are in **IsolationRoom** (includes AirborneIsolation and ContactIsolation via reasoning)
- Where room **humidity > 60%** (outside specification)
- Shows their alarm and call counts to assess distress level

### The Problem

**The `example:HighRiskPatient` class doesn't exist in your raw data!**

Your data only has:
- `fall_risk_score` (from EMR)
- `alarm_count` (from Patient Monitoring)  
- `call_count` (from Nurse Call)

You need to **define a rule** that classifies patients as `HighRiskPatient` based on these indicators.

---

## Your Task

Define rules that classify a patient as `example:HighRiskPatient` when:

1. **Fall risk score > 80**, OR
2. **Alarm count > 5 in last 24 hours**, OR  
3. **Call count > 3 in last 24 hours**, OR
4. **Combination**: Fall risk > 50 AND (alarm count > 3 OR call count > 2)

You'll do this in **three approaches** to learn different techniques:

- **Approach A**: SPARQL CONSTRUCT (simplest, no reasoner needed)
- **Approach B**: RDFS + SPARQL Rules (using RDFS reasoning)
- **Approach C**: SWRL Rules (most powerful, widely supported)

---


## Defining  Rules 



### Your Task

Write  rules that classify patients as HighRiskPatient.

### SWRL Syntax Primer

SWRL rules have the form: `Antecedent → Consequent`

```
Patient(?p) ∧ hasFallRiskScore(?p, ?score) ∧ greaterThan(?score, 80)
→ HighRiskPatient(?p)
```

Reads as: "If ?p is a Patient AND ?p has fall risk score ?score AND ?score > 80, THEN ?p is a HighRiskPatient"

### Rule 1: High Fall Risk (Example)

```swrl
Patient(?p) ∧ hasFallRiskScore(?p, ?score) ∧ swrlb:greaterThan(?score, 80)
→ HighRiskPatient(?p)
```

### Rule 2: High Alarm Count (Your Turn)

```swrl
# TODO: Complete this rule
Patient(?p) ∧ hasAlarmCount(?p, ?count) ∧ swrlb:greaterThan(?count, ???)
→ ???(?p)
```

### Rule 3: High Call Count (Your Turn)

```swrl
# TODO: Complete this rule
```

### Rule 4: Combination Criteria (Advanced)

For "fall risk > 50 AND (alarms > 3 OR calls > 2)", you'll need multiple rules:

```swrl
# Sub-rule 4a: Fall risk > 50 AND alarms > 3
Patient(?p) ∧ 
hasFallRiskScore(?p, ?risk) ∧ swrlb:greaterThan(?risk, 50) ∧
hasAlarmCount(?p, ?alarms) ∧ swrlb:greaterThan(?alarms, 3)
→ HighRiskPatient(?p)

# TODO: Complete sub-rule 4b for calls > 2
```

### SWRL Built-ins Reference

Common comparison operators:
- `swrlb:greaterThan(?x, ?y)` - x > y
- `swrlb:lessThan(?x, ?y)` - x < y
- `swrlb:greaterThanOrEqual(?x, ?y)` - x >= y
- `swrlb:equal(?x, ?y)` - x = y

### How to Use SWRL Rules

1. **Protégé**: Use the SWRL tab to add rules
2. **Jena**: Use the Jena Rules syntax (similar to SWRL)
3. **Stardog/GraphDB**: Import SWRL rules into your ontology
4. **RDFLib**: Use the OWL-RL reasoner with custom rules

---

## Testing Your Rules

### Test Data

Use the patient and room data from previous exercises:

```turtle
@prefix example: <http://www.example.com/> .

# Patient data
example:patient_MRN-7834521 a example:Patient ;
    example:hasName "Maria Van den Berg" ;
    example:hasFallRiskScore 85 ;
    example:isLocatedIn example:room_3-WEST-214 ;
    example:hasAlarmCount 3 ;
    example:hasCallCount 4 .

example:patient_MRN-7834523 a example:Patient ;
    example:hasName "Sophie Dubois" ;
    example:hasFallRiskScore 92 ;
    example:isLocatedIn example:room_ICU-3W-216 ;
    example:hasAlarmCount 7 ;
    example:hasCallCount 1 .

example:patient_MRN-7834525 a example:Patient ;
    example:hasName "Emma Janssens" ;
    example:hasFallRiskScore 88 ;
    example:isLocatedIn example:room_ICU-3W-217 ;
    example:hasAlarmCount 2 ;
    example:hasCallCount 3 .

example:patient_MRN-7834522 a example:Patient ;
    example:hasName "Jan Vermeulen" ;
    example:hasFallRiskScore 45 ;
    example:isLocatedIn example:room_3-WEST-215 ;
    example:hasAlarmCount 1 ;
    example:hasCallCount 1 .

# Room data (from BMS)
example:room_3-WEST-214 a example:GeneralRoom ;
    example:hasRoomID "3-WEST-214" ;
    example:hasHumidity 72 ;
    example:hasHVACStatus "operational" .

example:room_ICU-3W-216 a example:ContactIsolationRoom ;
    example:hasRoomID "ICU-3W-216" ;
    example:hasHumidity 55 ;
    example:hasHVACStatus "operational" .

example:room_ICU-3W-217 a example:AirborneIsolationRoom ;
    example:hasRoomID "ICU-3W-217" ;
    example:hasHumidity 70 ;
    example:hasHVACStatus "operational" .

example:room_3-WEST-215 a example:GeneralRoom ;
    example:hasRoomID "3-WEST-215" ;
    example:hasHumidity 58 ;
    example:hasHVACStatus "operational" .
```

### Expected Results

After applying your rules, these patients should be classified as HighRiskPatient:

| Patient | Fall Risk | Alarms | Calls | Why High Risk? |
|---------|-----------|--------|-------|----------------|
| Maria   | 85        | 3      | 4     | Fall risk > 80 ✓<br>Calls > 3 ✓ |
| Sophie  | 92        | 7      | 1     | Fall risk > 80 ✓<br>Alarms > 5 ✓ |
| Emma    | 88        | 2      | 3     | Fall risk > 80 ✓ |
| Jan     | 45        | 1      | 1     | ❌ Not high risk |

### Running the Final Query

After applying your rules, run the main query:

```sparql
PREFIX example: <http://www.example.com/>

SELECT ?patient ?name ?roomID ?humidity ?hvacStatus           
       ?alarmCount ?callCount                               
WHERE {                                                     
  ?patient a example:HighRiskPatient ;
           example:hasName ?name ;                                 
           example:isLocatedIn ?room ;                             
           example:hasAlarmCount ?alarmCount ;                     
           example:hasCallCount ?callCount .                       
  
  ?room a example:IsolationRoom ;
        example:hasRoomID ?roomID ;
        example:hasHumidity ?humidity ;                            
        example:hasHVACStatus ?hvacStatus .                        
  
  FILTER (?humidity > 60)
}
ORDER BY DESC(?alarmCount)
```

### Expected Output

Only **Emma** should appear:
- ✓ Classified as HighRiskPatient (fall risk 88)
- ✓ In IsolationRoom (AirborneIsolationRoom)
- ✓ Room humidity > 60% (70%)

**Why not Sophie?**
- ✓ HighRiskPatient (fall risk 92, alarms 7)
- ✓ In IsolationRoom (ContactIsolationRoom)
- ❌ Room humidity is 55% (within spec)

**Why not Maria?**
- ✓ HighRiskPatient (fall risk 85, calls 4)
- ❌ Not in IsolationRoom (GeneralRoom)

---

## Bonus Challenge: Dynamic Thresholds

### The Problem

Currently, the threshold (fall risk > 80) is hardcoded in rules. What if clinical guidelines change and the threshold becomes 75?

You'd need to update every rule.

### Better Approach: Configuration in the Ontology

Define thresholds as data properties:

```turtle
example:HighRiskThresholds a owl:Thing ;
    example:fallRiskThreshold 80 ;
    example:alarmCountThreshold 5 ;
    example:callCountThreshold 3 .
```

### Advanced SPARQL CONSTRUCT

```sparql
PREFIX example: <http://www.example.com/>

CONSTRUCT {
  ?patient a example:HighRiskPatient .
}
WHERE {
  ?patient a example:Patient ;
           example:hasFallRiskScore ?fallRisk .
  
  example:HighRiskThresholds example:fallRiskThreshold ?threshold .
  
  FILTER (?fallRisk > ?threshold)
}
```

Now updating the threshold is a **data change**, not a **code change**!

### Your Task

Extend your rules to use configurable thresholds for all criteria.

