# SPARQL Query Exercises

## Setup

For these exercises, you'll write SPARQL queries against the RDF data you generated in the RML mapping exercises.

**Tool:** [SPARQL Playground](https://atomgraph.github.io/SPARQL-Playground/)

**Goal:** Learn how to query integrated data across multiple sources using SPARQL, discovering insights that span patient and room information.

---

## Prerequisites

Before starting these exercises, make sure you have:

1. Completed the RML mapping exercises (or use the provided solution)
2. Generated RDF output from both patient and room data
3. The RDF data loaded in the Matey tool (or another SPARQL endpoint)

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
  # TODO: Add your triple patterns here
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
  
  # TODO: Add triple patterns for room information
}
```

### Hints

- The patient is already linked to the room via `example:locatedIn`
- Rooms have properties: `example:hasRoomID`, `example:hasTemperature`, `example:hasHumidity`
- For room type, use: `?room a ?roomType`
- You might want to filter out `example:Room` and only show specific types

### Expected Output

You should see patients matched with their room details.

---

## Exercise 4: High Fall Risk in High Humidity Rooms 🎯

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

### Follow-up Questions

1. What should the hospital do with this information?
2. Why didn't Sophie (risk: 92) appear in the results? (Hint: check her room's humidity)
3. How would you modify the query to find patients in rooms that need HVAC maintenance?

---

## Exercise 5: Isolation Room Types

### Task

Find **all patients who are in isolation rooms** (either Airborne or Contact isolation).

### Hints

- You need to check if the room type is either `example:AirborneIsolationRoom` or `example:ContactIsolationRoom`
- You can use the `VALUES` keyword or multiple triple patterns with `UNION`

### Approach 1: Using FILTER

```sparql
PREFIX example: <http://www.example.com/>

SELECT ?patientName ?roomID ?roomType
WHERE {
  ?patient a example:Patient ;
           example:hasName ?patientName ;
           example:locatedIn ?room .
  
  ?room example:hasRoomID ?roomID ;
        a ?roomType .
  
  # TODO: Add FILTER to check if roomType is one of the isolation types
}
```

### Approach 2: Using UNION

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

### Expected Output

You should see **2 patients**:
- Sophie in ICU-3W-216 (ContactIsolationRoom)
- Emma in ICU-3W-217 (AirborneIsolationRoom)

---

## Exercise 6: Room Capacity and Occupancy

### Task

Find **rooms with their capacity and current number of patients**.

This requires counting how many patients are in each room.

### Starter Template

```sparql
PREFIX example: <http://www.example.com/>

SELECT ?roomID ?capacity (COUNT(?patient) AS ?occupancy)
WHERE {
  ?room example:hasRoomID ?roomID ;
        example:hasCapacity ?capacity .
  
  # Optional: also count patients (some rooms might be empty)
  OPTIONAL {
    ?patient example:locatedIn ?room .
  }
}
GROUP BY ?roomID ?capacity
ORDER BY ?roomID
```

### Questions to Consider

1. Are any rooms at full capacity?
2. Are any rooms over capacity? (occupancy > capacity)
3. Which rooms are empty?

---

## Exercise 7: Rooms Needing Maintenance

### Task

Find **rooms that need maintenance attention** based on:
- HVAC status is "maintenance_required", OR
- Last update was more than 60 minutes ago, OR
- Humidity is outside the 40-60% specification range

### Your Task

Write a query that identifies rooms meeting any of these criteria.

### Hints

- Use `FILTER` with logical operators (`||` for OR)
- Check: `?hvacStatus = "maintenance_required"`
- Check: `?lastUpdate > 60`
- Check: `?humidity < 40 || ?humidity > 60`

### Expected Output

You should find several rooms with issues:
- Room 3-WEST-220: maintenance_required + high humidity (65%)
- Room 3-WEST-214: high humidity (72%)
- Room ICU-3W-217: high humidity (70%)

### Extension

Modify your query to also show if any patients are currently in these rooms needing attention.

---

## Bonus Exercise: Complex Analytics Query

### Task

Write a query that provides a **comprehensive patient-room risk assessment** showing:

- Patient name and fall risk score
- Room ID and type
- Room temperature and humidity
- HVAC status
- A calculated "risk level" based on multiple factors

### Criteria for Risk Levels

- **CRITICAL**: Fall risk > 80 AND (humidity > 60 OR hvac = "maintenance_required")
- **HIGH**: Fall risk > 80 OR humidity > 60
- **MEDIUM**: Fall risk > 50
- **NORMAL**: Everything else

### Starter Template

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
  # TODO: Add your triple patterns here
}
ORDER BY DESC(?riskLevel) DESC(?riskScore)
```

### Expected Output

A prioritized list showing which patients need immediate attention based on the combination of their personal risk factors and environmental conditions.

---

## Bonus Exercise: Property Paths (Advanced)

If you completed the advanced RML exercise with room type hierarchy, you can use SPARQL property paths to query the hierarchy.

### Task

Find **all patients in any type of isolation room** using the class hierarchy.

### Hint

If you defined that `ContactIsolationRoom` and `AirborneIsolationRoom` are both subclasses of `IsolationRoom`, you can query:

```sparql
PREFIX example: <http://www.example.com/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?patientName ?roomType
WHERE {
  ?patient a example:Patient ;
           example:hasName ?patientName ;
           example:locatedIn ?room .
  
  ?room a ?roomType .
  ?roomType rdfs:subClassOf* example:IsolationRoom .
}
```

The `*` means "zero or more" steps following the `rdfs:subClassOf` property, allowing you to match both direct instances and subclass instances.

---

## Tips for Writing SPARQL Queries

1. **Start simple**: Begin with basic triple patterns and add complexity gradually
2. **Test incrementally**: Run your query after each addition to catch errors early
3. **Use meaningful variable names**: `?patient`, `?room`, `?humidity` are clearer than `?x`, `?y`, `?z`
4. **Check your prefixes**: Make sure they match your RDF data
5. **OPTIONAL is your friend**: Use it when some data might be missing
6. **Comment your queries**: Add comments to explain complex logic

---

## Common SPARQL Patterns

### Basic Triple Pattern
```sparql
?subject predicate:property ?object .
```

### Filtering
```sparql
FILTER (?value > 80)
FILTER (?status = "active")
```

### Multiple Conditions (AND)
```sparql
FILTER (?value > 80 && ?status = "active")
```

### Multiple Conditions (OR)
```sparql
FILTER (?value > 80 || ?value < 20)
```

### Optional Data
```sparql
OPTIONAL {
  ?patient example:hasEmail ?email .
}
```

### Aggregation
```sparql
SELECT ?room (COUNT(?patient) AS ?count)
WHERE { ... }
GROUP BY ?room
```

### Sorting
```sparql
ORDER BY DESC(?riskScore)
```

---

## Learning Objectives

By completing these exercises, you should be able to:

✓ Write basic SPARQL SELECT queries  
✓ Filter results using FILTER clauses  
✓ Join data from multiple sources using shared IRIs  
✓ Use comparison operators (>, <, =)  
✓ Aggregate data with COUNT and GROUP BY  
✓ Sort results with ORDER BY  
✓ Combine conditions with logical operators (AND, OR)  
✓ Use OPTIONAL for incomplete data  
✓ Calculate derived values with expressions  
✓ Apply SPARQL queries to real-world data integration scenarios  

---

## Real-World Impact

The query from Exercise 4 (high fall risk in high humidity rooms) demonstrates the power of semantic integration:

**Before Integration:**
- EMR team knows: Maria has fall risk 85
- Facilities team knows: Room 3-WEST-214 has 72% humidity
- **Nobody connects the dots**

**After Integration:**
- One query reveals: Maria (high fall risk) is in a room with poor environmental conditions
- **Action can be taken immediately**: Adjust HVAC, move patient, or increase monitoring

This is the value of semantic integration for healthcare analytics!

---

## Resources

- **SPARQL 1.1 Specification**: https://www.w3.org/TR/sparql11-query/
- **SPARQL Tutorial**: https://www.w3.org/2009/Talks/0615-qbe/
- **SPARQL by Example**: https://www.w3.org/TR/sparql11-query/#examples
- **Interactive SPARQL Tutorial**: https://www.wikidata.org/wiki/Wikidata:SPARQL_tutorial

---

Good luck with your queries! 🚀
