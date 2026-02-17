```
PREFIX example: <http://www.example.com/>


RULE :Patient1:- 
CONSTRUCT { 
   ?p a example:HighRiskPatient.

}
WHERE { 
   ?p a example:Patient.
  ?p example:hasFallRiskScore ?score.
  FILTER (?score >80)
  	
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
RULE :Patient3:- 
CONSTRUCT { 
   ?p a example:HighRiskPatient.
}
WHERE { 
   ?p a example:Patient.
  ?p example:hasAlarmCount ?count.
  FILTER(?count >3)  	
}
RULE :Room1:- 
CONSTRUCT { 
   ?p a example:HighRiskRoom.
}
WHERE { 
   ?p a example:Room.
  ?p example:hasHumidity ?hum.
  ?p example:hasTemperature ?temp.
  FILTER(?hum >60)
  FILTER(?temp >24)
}
```
