# RML Mapping Exercises

## Setup

For these exercises, we'll use **Matey** - a browser-based tool for creating and testing RML mappings using the YARRRML syntax (a human-friendly notation for RML).

**Tool:** https://rml.io/yarrrml/matey/#

**Goal:** Learn how to map CSV data to RDF triples using RML, extending our patient monitoring use case.

---

## Exercise 1: Add Patient Fall Risk Score

### Background

We have a CSV file with patient information that's already partially mapped to RDF. Currently, we're mapping the patient ID, room location, and assigned nurse, but we're **not yet capturing the fall risk score** — a critical piece of information for patient safety.

### Current Data (patients.csv)

```csv
patient_mrn,patient_name,fall_risk_score,room_location,assigned_nurse_id
MRN-7834521,Maria Van den Berg,85,3-WEST-214,NURSE-102
MRN-7834522,Jan Vermeulen,45,3-WEST-215,NURSE-102
MRN-7834523,Sophie Dubois,92,ICU-3W-216,NURSE-103
MRN-7834524,Lucas Peeters,38,2-EAST-101,NURSE-104
MRN-7834525,Emma Janssens,88,ICU-3W-217,NURSE-103
```

### Current RML Mapping (YARRRML)

```yaml
prefixes:
  example: http://www.example.com/
mappings:
  patients:
    sources:
      - [patients.csv~csv]
    s: example:patient_$(patient_mrn)
    po:
      - [a, example:Patient]
      - [example:hasID, $(patient_mrn)]
      - [example:locatedIn, example:room_$(room_location)~iri]
      - [example:assignedTo, example:nurse$(assigned_nurse_id)~iri]
```

### Your Task

**Add the fall risk score property to the mapping.**

1. Open Matey: https://rml.io/yarrrml/matey/#
2. Copy the patient CSV data into the "Input: Data Source" section (middle left)
3. Copy the current YARRRML mapping into the "YARRRML" section (middle right)
4. **Extend the mapping** to include a new property: `example:hasFallRiskScore`
   - The value should come from the `fall_risk_score` column
5. Click "Integrate!" to see the generated RDF output

### Expected Output (excerpt)

You should see triples like:

```turtle
example:patient_MRN-7834521 a example:Patient ;
    example:hasID "MRN-7834521" ;
    example:hasFallRiskScore "85" ;
    example:locatedIn example:room_3-WEST-214 ;
    example:assignedTo example:nurseNURSE-102 .
```


---

## Exercise 2: Add BMS Room Data Source

### Background

Our Building Management System (BMS) tracks room conditions and metadata. Currently, our RDF only references rooms by ID (e.g., `example:room_3-WEST-214`), but we have no information about the rooms themselves — their type, capacity, environmental conditions, etc.

In this exercise, you'll add a **second data source** and create a **second mapping** to describe the rooms.

### New Data Source (rooms.csv)

Create this as a second CSV file:

```csv
room_id,room_type,zone,capacity,temperature_celsius,humidity_percent,hvac_status,last_update_minutes
3-WEST-214,GeneralRoom,3-WEST,2,21.5,72,operational,15
3-WEST-215,GeneralRoom,3-WEST,1,22.0,58,operational,15
ICU-3W-216,ContactIsolationRoom,ICU-3W,1,20.8,55,operational,15
2-EAST-101,GeneralRoom,2-EAST,2,21.2,60,operational,15
ICU-3W-217,AirborneIsolationRoom,ICU-3W,1,21.0,70,operational,15
3-WEST-220,AirborneIsolationRoom,3-WEST,1,20.5,65,maintenance_required,120
ICU-3W-218,GeneralRoom,ICU-3W,1,21.8,62,operational,15
```

**Note:** This includes the rooms from our patient data plus a few additional rooms to show different types.

### Room Types

- **GeneralRoom**: Standard patient room
- **ContactIsolationRoom**: For patients with contact-transmitted infections (e.g., MRSA, C. diff)
- **AirborneIsolationRoom**: Negative pressure room for airborne infections (e.g., TB, COVID)

### Your Task

**Create a second mapping for room data.**

1. In Matey, add the rooms.csv data to your input (you can have multiple CSV blocks separated by `---`)
2. Add a **new mapping** called `rooms` to your YARRRML file
3. The mapping should:
   - Use `rooms.csv` as the source
   - Create subjects with IRI: `example:room_$(room_id)`
   - Map the room to its type as a class (e.g., `example:GeneralRoom`)
     - Hint: Use `[a, example:$(room_type)~iri]` to make the room type a class
   - Include properties for:
     - `example:hasZone` → zone
     - `example:hasCapacity` → capacity
     - `example:hasTemperature` → temperature_celsius
     - `example:hasHumidity` → humidity_percent
     - `example:hasHVACStatus` → hvac_status
     - `example:lastUpdatedMinutesAgo` → last_update_minutes

### Starter Template

```yaml
prefixes:
  example: http://www.example.com/

mappings:
  patients:
    sources:
      - [patients.csv~csv]
    s: example:patient_$(patient_mrn)
    po:
      - [a, example:Patient]
      - [example:hasID, $(patient_mrn)]
      - [example:hasFallRiskScore, $(fall_risk_score)]
      - [example:locatedIn, example:room_$(room_location)~iri]
      - [example:assignedTo, example:nurse$(assigned_nurse_id)~iri]
  
  rooms:
    sources:
      - [rooms.csv~csv]
    s: example:room_$(room_id)
    po:
      # TODO: Add your mappings here
      - [a, example:$(room_type)~iri]
      # Add more properties...
```

### Expected Output (excerpt)

You should see triples like:

```turtle
example:room_3-WEST-214 a example:GeneralRoom ;
    example:hasZone example:zone_3-WEST ;
    example:hasCapacity "2" ;
    example:hasTemperature "21.5" ;
    example:hasHumidity "72" ;
    example:hasHVACStatus "operational" ;
    example:lastUpdatedMinutesAgo "15" .

example:room_ICU-3W-216 a example:ContactIsolationRoom ;
    example:hasZone example:zone_ICU-3W ;
    example:hasCapacity "1" ;
    example:hasTemperature "20.8" ;
    example:hasHumidity "55" ;
    example:hasHVACStatus "operational" ;
    example:lastUpdatedMinutesAgo "15" .
```


---

## Bonus Challenge: Add Data Type Specifications

### Task

Modify your mappings to specify proper data types for numeric values:

- `fall_risk_score` should be `xsd:integer`
- `capacity` should be `xsd:integer`
- `temperature_celsius` should be `xsd:decimal`
- `humidity_percent` should be `xsd:integer`
- `last_update_minutes` should be `xsd:integer`

**Hint:** Use the datatype annotation syntax: `$(column_name)~xsd:integer`

Example:
```yaml
- [example:hasCapacity, $(capacity)~xsd:integer]
```

---



## Resources

- **Matey tool**: https://rml.io/yarrrml/matey/#
- **YARRRML documentation**: https://rml.io/yarrrml/spec/
- **RML specification**: https://rml.io/specs/rml/
- **Common RDF datatypes**: http://www.w3.org/TR/rdf11-concepts/#section-Datatypes

---

## Learning Objectives

By completing these exercises, you should be able to:

✓ Read and understand YARRRML syntax  
✓ Add new properties to existing RML mappings  
✓ Work with multiple data sources in a single RML configuration  
✓ Create IRIs (URIs) from data values  
✓ Specify data types for literals  
✓ Understand how mappings from different sources connect via shared IRIs  
✓ Validate your RDF output against your source data  

---

## Tips

- **Start simple**: Get the basic mapping working before adding complexity
- **Check the output**: Always look at the generated RDF to verify it matches your intent
- **Use meaningful IRIs**: Make sure your room and patient IRIs match between mappings
- **Test incrementally**: Add one property at a time and verify each works
- **Datatypes matter**: Numeric data should be typed appropriately for later queries

Good luck! 
