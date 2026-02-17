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

The **goal**: Find high-risk patients in high-risk rooms.


---
## The Data (given)

We provide the resulting dataset containing EMR, Patient Monitoring and BMS data:

```
<http://www.example.com/patient_MRN-7834521> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.example.com/Patient> .
<http://www.example.com/patient_MRN-7834521> <http://www.example.com/hasID> "MRN-7834521" .
<http://www.example.com/patient_MRN-7834521> <http://www.example.com/hasName> "Maria Van den Berg" .
<http://www.example.com/patient_MRN-7834521> <http://www.example.com/hasFallRiskScore> "80"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/patient_MRN-7834521> <http://www.example.com/locatedIn> <http://www.example.com/room_3-WEST-214> .
<http://www.example.com/patient_MRN-7834521> <http://www.example.com/assignedTo> <http://www.example.com/nurse_NURSE-102> .
<http://www.example.com/patient_MRN-7834521> <http://www.example.com/hasAlarmCount> "3"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/patient_MRN-7834521> <http://www.example.com/hasCallCount> "4"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/patient_MRN-7834521> <http://www.example.com/hasLastAlarmTime> "2024-11-15T14:30:00"^^<http://www.w3.org/2001/XMLSchema#dateTime> .
<http://www.example.com/patient_MRN-7834521> <http://www.example.com/hasAvgHeartRate> "78"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/patient_MRN-7834521> <http://www.example.com/hasAvgSpO2> "96"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/patient_MRN-7834521> <http://www.example.com/hasMonitoringStatus> "active" .
<http://www.example.com/patient_MRN-7834522> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.example.com/Patient> .
<http://www.example.com/patient_MRN-7834522> <http://www.example.com/hasID> "MRN-7834522" .
<http://www.example.com/patient_MRN-7834522> <http://www.example.com/hasName> "Jan Vermeulen" .
<http://www.example.com/patient_MRN-7834522> <http://www.example.com/hasFallRiskScore> "45"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/patient_MRN-7834522> <http://www.example.com/locatedIn> <http://www.example.com/room_3-WEST-215> .
<http://www.example.com/patient_MRN-7834522> <http://www.example.com/assignedTo> <http://www.example.com/nurse_NURSE-102> .
<http://www.example.com/patient_MRN-7834522> <http://www.example.com/hasAlarmCount> "1"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/patient_MRN-7834522> <http://www.example.com/hasCallCount> "1"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/patient_MRN-7834522> <http://www.example.com/hasLastAlarmTime> "2024-11-15T08:15:00"^^<http://www.w3.org/2001/XMLSchema#dateTime> .
<http://www.example.com/patient_MRN-7834522> <http://www.example.com/hasAvgHeartRate> "72"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/patient_MRN-7834522> <http://www.example.com/hasAvgSpO2> "98"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/patient_MRN-7834522> <http://www.example.com/hasMonitoringStatus> "active" .
<http://www.example.com/patient_MRN-7834523> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.example.com/Patient> .
<http://www.example.com/patient_MRN-7834523> <http://www.example.com/hasID> "MRN-7834523" .
<http://www.example.com/patient_MRN-7834523> <http://www.example.com/hasName> "Sophie Dubois" .
<http://www.example.com/patient_MRN-7834523> <http://www.example.com/hasFallRiskScore> "92"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/patient_MRN-7834523> <http://www.example.com/locatedIn> <http://www.example.com/room_ICU-3W-216> .
<http://www.example.com/patient_MRN-7834523> <http://www.example.com/assignedTo> <http://www.example.com/nurse_NURSE-103> .
<http://www.example.com/patient_MRN-7834523> <http://www.example.com/hasAlarmCount> "7"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/patient_MRN-7834523> <http://www.example.com/hasCallCount> "1"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/patient_MRN-7834523> <http://www.example.com/hasLastAlarmTime> "2024-11-15T15:45:00"^^<http://www.w3.org/2001/XMLSchema#dateTime> .
<http://www.example.com/patient_MRN-7834523> <http://www.example.com/hasAvgHeartRate> "88"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/patient_MRN-7834523> <http://www.example.com/hasAvgSpO2> "94"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/patient_MRN-7834523> <http://www.example.com/hasMonitoringStatus> "active" .
<http://www.example.com/patient_MRN-7834524> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.example.com/Patient> .
<http://www.example.com/patient_MRN-7834524> <http://www.example.com/hasID> "MRN-7834524" .
<http://www.example.com/patient_MRN-7834524> <http://www.example.com/hasName> "Lucas Peeters" .
<http://www.example.com/patient_MRN-7834524> <http://www.example.com/hasFallRiskScore> "38"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/patient_MRN-7834524> <http://www.example.com/locatedIn> <http://www.example.com/room_2-EAST-101> .
<http://www.example.com/patient_MRN-7834524> <http://www.example.com/assignedTo> <http://www.example.com/nurse_NURSE-104> .
<http://www.example.com/patient_MRN-7834524> <http://www.example.com/hasAlarmCount> "4"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/patient_MRN-7834524> <http://www.example.com/hasCallCount> "1"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/patient_MRN-7834524> <http://www.example.com/hasLastAlarmTime> "2024-11-14T22:10:00"^^<http://www.w3.org/2001/XMLSchema#dateTime> .
<http://www.example.com/patient_MRN-7834524> <http://www.example.com/hasAvgHeartRate> "68"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/patient_MRN-7834524> <http://www.example.com/hasAvgSpO2> "99"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/patient_MRN-7834524> <http://www.example.com/hasMonitoringStatus> "active" .
<http://www.example.com/patient_MRN-7834525> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.example.com/Patient> .
<http://www.example.com/patient_MRN-7834525> <http://www.example.com/hasID> "MRN-7834525" .
<http://www.example.com/patient_MRN-7834525> <http://www.example.com/hasName> "Emma Janssens" .
<http://www.example.com/patient_MRN-7834525> <http://www.example.com/hasFallRiskScore> "88"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/patient_MRN-7834525> <http://www.example.com/locatedIn> <http://www.example.com/room_ICU-3W-217> .
<http://www.example.com/patient_MRN-7834525> <http://www.example.com/assignedTo> <http://www.example.com/nurse_NURSE-103> .
<http://www.example.com/patient_MRN-7834525> <http://www.example.com/hasAlarmCount> "2"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/patient_MRN-7834525> <http://www.example.com/hasCallCount> "3"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/patient_MRN-7834525> <http://www.example.com/hasLastAlarmTime> "2024-11-15T13:20:00"^^<http://www.w3.org/2001/XMLSchema#dateTime> .
<http://www.example.com/patient_MRN-7834525> <http://www.example.com/hasAvgHeartRate> "82"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/patient_MRN-7834525> <http://www.example.com/hasAvgSpO2> "95"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/patient_MRN-7834525> <http://www.example.com/hasMonitoringStatus> "active" .
<http://www.example.com/room_3-WEST-214> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.example.com/GeneralRoom> .
<http://www.example.com/room_3-WEST-214> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.example.com/Room> .
<http://www.example.com/room_3-WEST-214> <http://www.example.com/hasRoomID> "3-WEST-214" .
<http://www.example.com/room_3-WEST-214> <http://www.example.com/hasZone> "3-WEST" .
<http://www.example.com/room_3-WEST-214> <http://www.example.com/hasCapacity> "2"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_3-WEST-214> <http://www.example.com/hasTemperature> "24.2"^^<http://www.w3.org/2001/XMLSchema#decimal> .
<http://www.example.com/room_3-WEST-214> <http://www.example.com/hasHumidity> "72"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_3-WEST-214> <http://www.example.com/hasHVACStatus> "operational" .
<http://www.example.com/room_3-WEST-214> <http://www.example.com/lastUpdatedMinutesAgo> "15"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_3-WEST-215> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.example.com/GeneralRoom> .
<http://www.example.com/room_3-WEST-215> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.example.com/Room> .
<http://www.example.com/room_3-WEST-215> <http://www.example.com/hasRoomID> "3-WEST-215" .
<http://www.example.com/room_3-WEST-215> <http://www.example.com/hasZone> "3-WEST" .
<http://www.example.com/room_3-WEST-215> <http://www.example.com/hasCapacity> "1"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_3-WEST-215> <http://www.example.com/hasTemperature> "22.0"^^<http://www.w3.org/2001/XMLSchema#decimal> .
<http://www.example.com/room_3-WEST-215> <http://www.example.com/hasHumidity> "58"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_3-WEST-215> <http://www.example.com/hasHVACStatus> "operational" .
<http://www.example.com/room_3-WEST-215> <http://www.example.com/lastUpdatedMinutesAgo> "15"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_ICU-3W-216> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.example.com/ContactIsolationRoom> .
<http://www.example.com/room_ICU-3W-216> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.example.com/IsolationRoom> .
<http://www.example.com/room_ICU-3W-216> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.example.com/Room> .
<http://www.example.com/room_ICU-3W-216> <http://www.example.com/hasRoomID> "ICU-3W-216" .
<http://www.example.com/room_ICU-3W-216> <http://www.example.com/hasZone> "ICU-3W" .
<http://www.example.com/room_ICU-3W-216> <http://www.example.com/hasCapacity> "1"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_ICU-3W-216> <http://www.example.com/hasTemperature> "20.8"^^<http://www.w3.org/2001/XMLSchema#decimal> .
<http://www.example.com/room_ICU-3W-216> <http://www.example.com/hasHumidity> "55"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_ICU-3W-216> <http://www.example.com/hasHVACStatus> "operational" .
<http://www.example.com/room_ICU-3W-216> <http://www.example.com/lastUpdatedMinutesAgo> "15"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_2-EAST-101> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.example.com/GeneralRoom> .
<http://www.example.com/room_2-EAST-101> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.example.com/Room> .
<http://www.example.com/room_2-EAST-101> <http://www.example.com/hasRoomID> "2-EAST-101" .
<http://www.example.com/room_2-EAST-101> <http://www.example.com/hasZone> "2-EAST" .
<http://www.example.com/room_2-EAST-101> <http://www.example.com/hasCapacity> "2"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_2-EAST-101> <http://www.example.com/hasTemperature> "24.2"^^<http://www.w3.org/2001/XMLSchema#decimal> .
<http://www.example.com/room_2-EAST-101> <http://www.example.com/hasHumidity> "61"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_2-EAST-101> <http://www.example.com/hasHVACStatus> "operational" .
<http://www.example.com/room_2-EAST-101> <http://www.example.com/lastUpdatedMinutesAgo> "15"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_ICU-3W-217> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.example.com/AirborneIsolationRoom> .
<http://www.example.com/room_ICU-3W-217> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.example.com/IsolationRoom> .
<http://www.example.com/room_ICU-3W-217> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.example.com/Room> .
<http://www.example.com/room_ICU-3W-217> <http://www.example.com/hasRoomID> "ICU-3W-217" .
<http://www.example.com/room_ICU-3W-217> <http://www.example.com/hasZone> "ICU-3W" .
<http://www.example.com/room_ICU-3W-217> <http://www.example.com/hasCapacity> "1"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_ICU-3W-217> <http://www.example.com/hasTemperature> "21.0"^^<http://www.w3.org/2001/XMLSchema#decimal> .
<http://www.example.com/room_ICU-3W-217> <http://www.example.com/hasHumidity> "70"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_ICU-3W-217> <http://www.example.com/hasHVACStatus> "operational" .
<http://www.example.com/room_ICU-3W-217> <http://www.example.com/lastUpdatedMinutesAgo> "15"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_3-WEST-220> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.example.com/AirborneIsolationRoom> .
<http://www.example.com/room_3-WEST-220> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.example.com/IsolationRoom> .
<http://www.example.com/room_3-WEST-220> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.example.com/Room> .
<http://www.example.com/room_3-WEST-220> <http://www.example.com/hasRoomID> "3-WEST-220" .
<http://www.example.com/room_3-WEST-220> <http://www.example.com/hasZone> "3-WEST" .
<http://www.example.com/room_3-WEST-220> <http://www.example.com/hasCapacity> "1"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_3-WEST-220> <http://www.example.com/hasTemperature> "24.1"^^<http://www.w3.org/2001/XMLSchema#decimal> .
<http://www.example.com/room_3-WEST-220> <http://www.example.com/hasHumidity> "65"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_3-WEST-220> <http://www.example.com/hasHVACStatus> "maintenance_required" .
<http://www.example.com/room_3-WEST-220> <http://www.example.com/lastUpdatedMinutesAgo> "120"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_ICU-3W-218> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.example.com/GeneralRoom> .
<http://www.example.com/room_ICU-3W-218> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.example.com/Room> .
<http://www.example.com/room_ICU-3W-218> <http://www.example.com/hasRoomID> "ICU-3W-218" .
<http://www.example.com/room_ICU-3W-218> <http://www.example.com/hasZone> "ICU-3W" .
<http://www.example.com/room_ICU-3W-218> <http://www.example.com/hasCapacity> "1"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_ICU-3W-218> <http://www.example.com/hasTemperature> "21.8"^^<http://www.w3.org/2001/XMLSchema#decimal> .
<http://www.example.com/room_ICU-3W-218> <http://www.example.com/hasHumidity> "62"^^<http://www.w3.org/2001/XMLSchema#integer> .
<http://www.example.com/room_ICU-3W-218> <http://www.example.com/hasHVACStatus> "operational" .
<http://www.example.com/room_ICU-3W-218> <http://www.example.com/lastUpdatedMinutesAgo> "15"^^<http://www.w3.org/2001/XMLSchema#integer> .

```

---

## The Query (Given)

Here's the query the clinical team wants to run:

```sparql
PREFIX example: <http://www.example.com/>

SELECT ?patient ?room                              
WHERE {                                                     
  ?patient a example:HighRiskPatient ;                            
           example:locatedIn ?room .                                                
  
  ?room a example:HighRiskRoom.
}
```

### What This Query Does

- Finds patients classified as **HighRiskPatient** (reasoning determines this!)
- Who are in **HighRiskRoom** (also via reasoning)


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
2. **Call count > 2**, OR  
3. **Alarm count > 3**, OR


Define rules that classify a rooms as `example:HighRiskRooms` when:

1. **Humidity > 60**, AND
2. **Temperature > 24**

---


## Defining  Rules 



### Your Task

Write  rules that classify patients as HighRiskPatient and rooms as HighRiskRooms.

### Kolibrie Rule Syntax Primer



```
RULE :Room1:- 
CONSTRUCT { 
   ?p a example:DoubleRoom.
}
WHERE { 
   ?p a example:Room.
    ?p example:hasCapacity ?cap.
FILTER(?cap >1)
}
```

Reads as: "If ?p is a Room AND ?p has a capacity of ?cap AND ?cap > 1, THEN ?p is a DoubleRoom"

### Patients Rules: Defining HighRiskPatients

Define rules that classify a patient as `example:HighRiskPatient` when:

1. **Fall risk score > 80**, OR
2. **Call count > 2**, OR  
3. **Alarm count > 3**, OR

Test with the query:
```sparql
PREFIX example: <http://www.example.com/>

SELECT ?patient                            
WHERE {                                                     
  ?patient a example:HighRiskPatient .
}
```


### Room Rules: Defining HighRiskRooms

Define rules that classify a rooms as `example:HighRiskRooms` when:

1. **Humidity > 60**, AND
2. **Temperature > 24**

Test with the query

```sparql
PREFIX example: <http://www.example.com/>

SELECT ?room                              
WHERE {                                                                                                   
  
  ?room a example:HighRiskRoom.
}
```

### Combining everything

Finding high risk patients in high risk rooms:

```sparql
PREFIX example: <http://www.example.com/>

SELECT ?patient ?room                              
WHERE {                                                     
  ?patient a example:HighRiskPatient ;                            
           example:locatedIn ?room .                                                
  
  ?room a example:HighRiskRoom.
}
```



