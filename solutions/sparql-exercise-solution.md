# SPARQL Query Exercises - Solutions

This file contains the complete solutions for all exercises in the SPARQL Query Exercises.

---

## Exercise 1: Basic Patient Query

### Solution

```sparql
PREFIX example: <http://www.example.com/>

SELECT ?patient ?name ?riskScore
WHERE {
  ?patient a example:Patient ;
           example:hasName ?name ;
           example:hasFallRiskScore ?riskScore .
}
```

### Expected Output

```
| patient                      | name               | riskScore |
|------------------------------|-------------------|-----------|
| example:patient_MRN-7834521  | Maria Van den Berg| 85        |
| example:patient_MRN-7834522  | Jan Vermeulen     | 45        |
| example:patient_MRN-7834523  | Sophie Dubois     | 92        |
| example:patient_MRN-7834524  | Lucas Peeters     | 38        |
| example:patient_MRN-7834525  | Emma Janssens     | 88        |
```

---

## Exercise 2: Filter High-Risk Patients

### Solution

```sparql
PREFIX example: <http://www.example.com/>

SELECT ?patient ?name ?riskScore
WHERE {
  ?patient a example:Patient ;
           example:hasName ?name ;
           example:hasFallRiskScore ?riskScore .
  
  FILTER (?riskScore > 80)
}
```

### Expected Output

```
| patient                      | name               | riskScore |
|------------------------------|-------------------|-----------|
| example:patient_MRN-7834521  | Maria Van den Berg| 85        |
| example:patient_MRN-7834523  | Sophie Dubois     | 92        |
| example:patient_MRN-7834525  | Emma Janssens     | 88        |
```

---

## Exercise 3: Join Patient and Room Data

### Solution

```sparql
PREFIX example: <http://www.example.com/>

SELECT ?patientName ?roomID ?roomType ?temperature ?humidity
WHERE {
  ?patient a example:Patient ;
           example:hasName ?patientName ;
           example:locatedIn ?room .
  
  ?room example:hasRoomID ?roomID ;
        example:hasTemperature ?temperature ;
        example:hasHumidity ?humidity ;
        a ?roomType .
}
```

### Expected Output (with Filter)

```
| patientName        | roomID       | roomType                        | temperature | humidity |
|-------------------|--------------|---------------------------------|-------------|----------|
| Maria Van den Berg| 3-WEST-214   | example:GeneralRoom             | 21.5        | 72       |
| Jan Vermeulen     | 3-WEST-215   | example:GeneralRoom             | 22.0        | 58       |
| Sophie Dubois     | ICU-3W-216   | example:ContactIsolationRoom    | 20.8        | 55       |
| Lucas Peeters     | 2-EAST-101   | example:GeneralRoom             | 21.2        | 60       |
| Emma Janssens     | ICU-3W-217   | example:AirborneIsolationRoom   | 21.0        | 70       |
```

---

## Exercise 4: High Fall Risk in High Humidity Rooms 

### Solution

```sparql
PREFIX example: <http://www.example.com/>

SELECT ?patientName ?riskScore ?roomID ?humidity
WHERE {
  ?patient a example:Patient ;
           example:hasName ?patientName ;
           example:hasFallRiskScore ?riskScore ;
           example:locatedIn ?room .
  
  ?room example:hasRoomID ?roomID ;
        example:hasHumidity ?humidity .
  
  FILTER (?riskScore > 80)
  FILTER (?humidity > 60)
}
ORDER BY DESC(?riskScore)
```

### Expected Output

```
| patientName        | riskScore | roomID       | humidity |
|-------------------|-----------|--------------|----------|
| Emma Janssens     | 88        | ICU-3W-217   | 70       |
| Maria Van den Berg| 85        | 3-WEST-214   | 72       |
```

### Explanation

- **Maria**: Fall risk 85, humidity 72% (12% above spec)
- **Emma**: Fall risk 88, humidity 70% (10% above spec)
- **Sophie** (risk 92) does NOT appear because her room humidity is 55% (within spec)

---

## Exercise 5: Isolation Room Types

### Solution - Approach 1: Using FILTER

```sparql
PREFIX example: <http://www.example.com/>

SELECT ?patientName ?roomID ?roomType
WHERE {
  ?patient a example:Patient ;
           example:hasName ?patientName ;
           example:locatedIn ?room .
  
  ?room example:hasRoomID ?roomID ;
        a ?roomType .
  
  FILTER (?roomType = example:AirborneIsolationRoom || 
          ?roomType = example:ContactIsolationRoom)
}
```

### Solution - Approach 2: Using UNION

```sparql
PREFIX example: <http://www.example.com/>

SELECT ?patientName ?roomID ?roomType
WHERE {
  ?patient a example:Patient ;
           example:hasName ?patientName ;
           example:locatedIn ?room .
  
  ?room example:hasRoomID ?roomID .
  
  {
    ?room a example:AirborneIsolationRoom .
    BIND(example:AirborneIsolationRoom AS ?roomType)
  }
  UNION
  {
    ?room a example:ContactIsolationRoom .
    BIND(example:ContactIsolationRoom AS ?roomType)
  }
}
```

### Solution - Approach 3: Using VALUES

```sparql
PREFIX example: <http://www.example.com/>

SELECT ?patientName ?roomID ?roomType
WHERE {
  ?patient a example:Patient ;
           example:hasName ?patientName ;
           example:locatedIn ?room .
  
  ?room example:hasRoomID ?roomID ;
        a ?roomType .
  
  VALUES ?roomType {
    example:AirborneIsolationRoom
    example:ContactIsolationRoom
  }
}
```

### Expected Output

```
| patientName    | roomID      | roomType                        |
|---------------|-------------|---------------------------------|
| Sophie Dubois | ICU-3W-216  | example:ContactIsolationRoom    |
| Emma Janssens | ICU-3W-217  | example:AirborneIsolationRoom   |
```

---

## Exercise 6: Room Capacity and Occupancy

### Solution

```sparql
PREFIX example: <http://www.example.com/>

SELECT ?roomID ?capacity (COUNT(?patient) AS ?occupancy)
WHERE {
  ?room example:hasRoomID ?roomID ;
        example:hasCapacity ?capacity .
  
  OPTIONAL {
    ?patient example:locatedIn ?room .
  }
}
GROUP BY ?roomID ?capacity
ORDER BY ?roomID
```

### Alternative Solution (Only rooms with patients)

```sparql
PREFIX example: <http://www.example.com/>

SELECT ?roomID ?capacity (COUNT(?patient) AS ?occupancy)
WHERE {
  ?room example:hasRoomID ?roomID ;
        example:hasCapacity ?capacity .
  
  ?patient example:locatedIn ?room .
}
GROUP BY ?roomID ?capacity
ORDER BY ?roomID
```

### Expected Output

```
| roomID       | capacity | occupancy |
|--------------|----------|-----------|
| 2-EAST-101   | 2        | 1         |
| 3-WEST-214   | 2        | 1         |
| 3-WEST-215   | 1        | 1         |
| 3-WEST-220   | 1        | 0         |
| ICU-3W-216   | 1        | 1         |
| ICU-3W-217   | 1        | 1         |
| ICU-3W-218   | 1        | 0         |
```

### Analysis

- All occupied rooms are at or below capacity ✓
- No rooms are over capacity ✓
- 2 rooms are empty: 3-WEST-220 and ICU-3W-218

---

## Exercise 7: Rooms Needing Maintenance

### Solution

```sparql
PREFIX example: <http://www.example.com/>

SELECT ?roomID ?hvacStatus ?humidity ?lastUpdate
WHERE {
  ?room example:hasRoomID ?roomID ;
        example:hasHVACStatus ?hvacStatus ;
        example:hasHumidity ?humidity ;
        example:lastUpdatedMinutesAgo ?lastUpdate .
  
  FILTER (
    ?hvacStatus = "maintenance_required" ||
    ?lastUpdate > 60 ||
    ?humidity < 40 ||
    ?humidity > 60
  )
}
ORDER BY ?roomID
```

### Expected Output

```
| roomID       | hvacStatus            | humidity | lastUpdate |
|--------------|-----------------------|----------|------------|
| 3-WEST-214   | operational           | 72       | 15         |
| 3-WEST-220   | maintenance_required  | 65       | 120        |
| ICU-3W-217   | operational           | 70       | 15         |
```

### Analysis

- **3-WEST-214**: High humidity (72%, spec is 40-60%)
- **3-WEST-220**: Multiple issues - maintenance required, high humidity (65%), last update 120 min ago
- **ICU-3W-217**: High humidity (70%)

### Extended Solution (With Patient Information)

```sparql
PREFIX example: <http://www.example.com/>

SELECT ?roomID ?hvacStatus ?humidity ?lastUpdate ?patientName
WHERE {
  ?room example:hasRoomID ?roomID ;
        example:hasHVACStatus ?hvacStatus ;
        example:hasHumidity ?humidity ;
        example:lastUpdatedMinutesAgo ?lastUpdate .
  
  OPTIONAL {
    ?patient example:locatedIn ?room ;
             example:hasName ?patientName .
  }
  
  FILTER (
    ?hvacStatus = "maintenance_required" ||
    ?lastUpdate > 60 ||
    ?humidity < 40 ||
    ?humidity > 60
  )
}
ORDER BY ?roomID
```

### Expected Output (Extended)

```
| roomID       | hvacStatus            | humidity | lastUpdate | patientName        |
|--------------|-----------------------|----------|------------|--------------------|
| 3-WEST-214   | operational           | 72       | 15         | Maria Van den Berg |
| 3-WEST-220   | maintenance_required  | 65       | 120        |                    |
| ICU-3W-217   | operational           | 70       | 15         | Emma Janssens      |
```

**Note:** Room 3-WEST-220 needs maintenance but is currently empty (no patient).

---

## Bonus Exercise: Complex Analytics Query

### Solution

```sparql
PREFIX example: <http://www.example.com/>

SELECT ?patientName ?riskScore ?roomID ?humidity ?hvacStatus
       (IF(?riskScore > 80 && (?humidity > 60 || ?hvacStatus = "maintenance_required"),
           "CRITICAL",
           IF(?riskScore > 80 || ?humidity > 60,
              "HIGH",
              IF(?riskScore > 50, "MEDIUM", "NORMAL")
           )
        ) AS ?riskLevel)
WHERE {
  ?patient a example:Patient ;
           example:hasName ?patientName ;
           example:hasFallRiskScore ?riskScore ;
           example:locatedIn ?room .
  
  ?room example:hasRoomID ?roomID ;
        example:hasHumidity ?humidity ;
        example:hasHVACStatus ?hvacStatus .
}
ORDER BY 
  (IF(?riskLevel = "CRITICAL", 1,
      IF(?riskLevel = "HIGH", 2,
         IF(?riskLevel = "MEDIUM", 3, 4))) 
  )
  DESC(?riskScore)
```

### Expected Output

```
| patientName        | riskScore | roomID      | humidity | hvacStatus  | riskLevel |
|-------------------|-----------|-------------|----------|-------------|-----------|
| Emma Janssens     | 88        | ICU-3W-217  | 70       | operational | CRITICAL  |
| Maria Van den Berg| 85        | 3-WEST-214  | 72       | operational | CRITICAL  |
| Sophie Dubois     | 92        | ICU-3W-216  | 55       | operational | HIGH      |
| Jan Vermeulen     | 45        | 3-WEST-215  | 58       | operational | NORMAL    |
| Lucas Peeters     | 38        | 2-EAST-101  | 60       | operational | NORMAL    |
```

### Analysis

**CRITICAL Priority (2 patients):**
- Emma: Fall risk 88 + humidity 70% (both conditions met)
- Maria: Fall risk 85 + humidity 72% (both conditions met)

**HIGH Priority (1 patient):**
- Sophie: Fall risk 92 (but humidity is OK at 55%)

**NORMAL Priority (2 patients):**
- Jan: Fall risk 45, humidity 58%
- Lucas: Fall risk 38, humidity 60%

---

## Bonus Exercise: Property Paths (Advanced)

### Solution

This assumes you completed the advanced RML exercise with the room type hierarchy.

```sparql
PREFIX example: <http://www.example.com/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?patientName ?roomID ?roomType
WHERE {
  ?patient a example:Patient ;
           example:hasName ?patientName ;
           example:locatedIn ?room .
  
  ?room example:hasRoomID ?roomID ;
        a ?roomType .
  
  ?roomType rdfs:subClassOf* example:IsolationRoom .
}
```

### Alternative: Using Property Paths for Hierarchy

```sparql
PREFIX example: <http://www.example.com/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?patientName ?roomID ?roomType ?parentType
WHERE {
  ?patient a example:Patient ;
           example:hasName ?patientName ;
           example:locatedIn ?room .
  
  ?room example:hasRoomID ?roomID ;
        a ?roomType .
  
  ?roomType rdfs:subClassOf+ ?parentType .
  
  FILTER (?parentType = example:IsolationRoom)
}
```

### Expected Output

```
| patientName    | roomID      | roomType                        |
|---------------|-------------|---------------------------------|
| Sophie Dubois | ICU-3W-216  | example:ContactIsolationRoom    |
| Emma Janssens | ICU-3W-217  | example:AirborneIsolationRoom   |
```

### Explanation of Property Paths

- `rdfs:subClassOf*` means "zero or more steps" (includes the class itself)
- `rdfs:subClassOf+` means "one or more steps" (excludes the class itself)
- This allows you to query the class hierarchy without explicitly listing all subclasses

---

## Additional Useful Queries

### Query 1: Patients with Their Nurses

```sparql
PREFIX example: <http://www.example.com/>

SELECT ?patientName ?nurseID
WHERE {
  ?patient a example:Patient ;
           example:hasName ?patientName ;
           example:assignedTo ?nurse .
  
  BIND(STRAFTER(STR(?nurse), "nurse_") AS ?nurseID)
}
ORDER BY ?nurseID ?patientName
```

### Query 2: Average Fall Risk Score by Room Type

```sparql
PREFIX example: <http://www.example.com/>

SELECT ?roomType (AVG(?riskScore) AS ?avgRisk) (COUNT(?patient) AS ?patientCount)
WHERE {
  ?patient a example:Patient ;
           example:hasFallRiskScore ?riskScore ;
           example:locatedIn ?room .
  
  ?room a ?roomType .
  
  FILTER (?roomType != example:Room)
}
GROUP BY ?roomType
ORDER BY DESC(?avgRisk)
```

### Query 3: Rooms with Temperature Outside Comfort Range (20-22°C)

```sparql
PREFIX example: <http://www.example.com/>

SELECT ?roomID ?temperature ?patientName
WHERE {
  ?room example:hasRoomID ?roomID ;
        example:hasTemperature ?temperature .
  
  OPTIONAL {
    ?patient example:locatedIn ?room ;
             example:hasName ?patientName .
  }
  
  FILTER (?temperature < 20 || ?temperature > 22)
}
ORDER BY ?temperature
```

### Query 4: All Environmental Issues Summary

```sparql
PREFIX example: <http://www.example.com/>

SELECT ?roomID ?patientName ?riskScore ?temperature ?humidity ?hvacStatus ?lastUpdate
       (IF(?temperature < 20 || ?temperature > 22, "Temp Issue", "") AS ?tempIssue)
       (IF(?humidity < 40 || ?humidity > 60, "Humidity Issue", "") AS ?humidityIssue)
       (IF(?hvacStatus = "maintenance_required", "HVAC Issue", "") AS ?hvacIssue)
       (IF(?lastUpdate > 60, "Update Delayed", "") AS ?updateIssue)
WHERE {
  ?room example:hasRoomID ?roomID ;
        example:hasTemperature ?temperature ;
        example:hasHumidity ?humidity ;
        example:hasHVACStatus ?hvacStatus ;
        example:lastUpdatedMinutesAgo ?lastUpdate .
  
  OPTIONAL {
    ?patient example:locatedIn ?room ;
             example:hasName ?patientName ;
             example:hasFallRiskScore ?riskScore .
  }
  
  FILTER (
    ?temperature < 20 || ?temperature > 22 ||
    ?humidity < 40 || ?humidity > 60 ||
    ?hvacStatus = "maintenance_required" ||
    ?lastUpdate > 60
  )
}
ORDER BY ?roomID
```

---

## Tips for Checking Your Solutions

1. **Count your results**: Make sure you get the expected number of rows
2. **Check specific values**: Verify that Maria's fall risk is 85, not 86
3. **Test edge cases**: What happens with exactly 60% humidity? (Should not appear if filtering > 60)
4. **Verify joins**: Make sure patient names match with correct room IDs
5. **Check data types**: Numbers should not have quotes around them in output
6. **Test OPTIONAL**: Try queries with and without OPTIONAL to see the difference
7. **Validate filters**: Change filter values slightly to ensure they're working

---

## Common Mistakes and How to Fix Them

### Mistake 1: Forgetting the PREFIX

**Wrong:**
```sparql
SELECT ?patient
WHERE {
  ?patient a Patient .
}
```

**Right:**
```sparql
PREFIX example: <http://www.example.com/>

SELECT ?patient
WHERE {
  ?patient a example:Patient .
}
```

### Mistake 2: Using = instead of > for numeric comparisons

**Wrong:**
```sparql
FILTER (?riskScore = 80)  # Only matches exactly 80
```

**Right:**
```sparql
FILTER (?riskScore > 80)  # Matches 81, 82, 85, 92, etc.
```

### Mistake 3: Forgetting to connect patient and room

**Wrong:**
```sparql
SELECT ?patientName ?roomID
WHERE {
  ?patient example:hasName ?patientName .
  ?room example:hasRoomID ?roomID .
}
```

**Right:**
```sparql
SELECT ?patientName ?roomID
WHERE {
  ?patient example:hasName ?patientName ;
           example:locatedIn ?room .
  ?room example:hasRoomID ?roomID .
}
```

### Mistake 4: Missing the semicolon in property chains

**Wrong:**
```sparql
?patient example:hasName ?name
         example:hasFallRiskScore ?score .
```

**Right:**
```sparql
?patient example:hasName ?name ;
         example:hasFallRiskScore ?score .
```

---

## Resources for Further Learning

- **SPARQL 1.1 Query Language**: https://www.w3.org/TR/sparql11-query/
- **SPARQL Tutorial**: https://jena.apache.org/tutorials/sparql.html
- **SPARQL Cheat Sheet**: https://www.iro.umontreal.ca/~lapalme/ift6281/sparql-1_1-cheat-sheet.pdf
- **Interactive SPARQL**: https://query.wikidata.org/ (practice on real data)

---

Good luck! 
