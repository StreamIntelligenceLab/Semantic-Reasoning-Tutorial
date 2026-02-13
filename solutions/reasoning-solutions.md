```
PREFIX example: <http://www.example.com/>

RULE :Patient1:- 
CONSTRUCT { 
   ?p a example:HighRiskPatient.

}
WHERE { 
   ?p a example:Patient.
  ?p example:hasFallRiskScore ?score.
  FILTER (?score >90)
  	
}
RULE :Patient2:- 
CONSTRUCT { 
   ?p a example:HighRiskPatient.
}
WHERE { 
   ?p a example:Patient.
  ?p example:hasCallCount ?count.
  FILTER(?count >2)  	
}
RULE :Room1:- 
CONSTRUCT { 
   ?p a example:HighRiskRoom.
}
WHERE { 
   ?p a example:Room.
  ?p example:hasHumidity ?hum.
  ?p example:hasTemperature ?temp.
  FILTER(?hum >30)
  FILTER(?temp >21)
}
```
