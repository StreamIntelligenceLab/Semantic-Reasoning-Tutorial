```
RULE :RDFS9:- 
CONSTRUCT { 
   ?p a ?s.

}
WHERE { 
   ?c <http://www.w3.org/2000/01/rdf-schema#subClassOf> ?s.
   ?p <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> ?c .
}
RULE :RDFS3:- 
CONSTRUCT { 
   ?y a ?c.

}
WHERE { 
      ?p <http://www.w3.org/2000/01/rdf-schema#range> ?c.
   		?x ?p ?y .
}

RULE :RDFS2:- 
CONSTRUCT { 
   ?x a ?c.

}
WHERE { 
   ?p <http://www.w3.org/2000/01/rdf-schema#domain> ?c.
   ?x ?p ?y .
}
```

```
<http://www.example.com/room1> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.example.com/IsolationRoom> .
<http://www.example.com/patient1> <http://www.example.com/locatedIn> <http://www.example.com/room2> .
```
