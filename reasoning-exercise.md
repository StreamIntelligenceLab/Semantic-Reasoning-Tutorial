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
           example:locatedIn ?room ;                                                
  
  ?room a example:HighRiskRoom.
}
```



