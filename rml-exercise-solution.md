```yaml
# Complete Solution for RML Mapping Exercises
# This file contains the full solution from both exercises 

prefixes:
  example: http://www.example.com/
  xsd: http://www.w3.org/2001/XMLSchema#
  rdfs: http://www.w3.org/2000/01/rdf-schema#

mappings:
  # Mapping for patient data (Exercise 1 solution)
  patients:
    sources:
      - [patients.csv~csv]
    s: example:patient_$(patient_mrn)
    po:
      - [a, example:Patient]
      - [example:hasID, $(patient_mrn)]
      - [example:hasName, $(patient_name)]
      - [example:hasFallRiskScore, $(fall_risk_score)~xsd:integer]
      - [example:locatedIn, example:room_$(room_location)~iri]
      - [example:assignedTo, example:nurse_$(assigned_nurse_id)~iri]
  
  # Mapping for room data (Exercise 2 solution)
  rooms:
    sources:
      - [rooms.csv~csv]
    s: example:room_$(room_id)
    po:
      - [a, example:Room]
      - [a, example:$(room_type)~iri]
      - [example:hasRoomID, $(room_id)]
      - [example:hasZone, example:zone_$(zone)~iri]
      - [example:hasCapacity, $(capacity)~xsd:integer]
      - [example:hasTemperature, $(temperature_celsius)~xsd:decimal]
      - [example:hasHumidity, $(humidity_percent)~xsd:integer]
      - [example:hasHVACStatus, $(hvac_status)]
      - [example:lastUpdatedMinutesAgo, $(last_update_minutes)~xsd:integer]

```
