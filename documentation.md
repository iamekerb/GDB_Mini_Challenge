# Mini Challenge GDB

**Gruppe:** Boran Eker, Murat Kayhan


### A.1 Welche Collections gibt es und wie viele Dokumente beinhalten sie jeweils?

```javascript
show dbs
```

```javascript
use sample_training
```

```javascript
show collections
```

```javascript
db.companies.countDocuments()
db.grades.countDocuments()
db.inspections.countDocuments()
db.posts.countDocuments()
db.routes.countDocuments()
db.trips.countDocuments()
db.zips.countDocuments()
```
Antwort:
- companies: 9500
- grades: 100000
- inspections: 80047
- posts: 500
- routes: 66958
- trips: 10000
- zips: 29470

### A.2 Beschreibe in Worten, was die folgenden Anweisungen jeweils ausgeben?

- `db.zips.find( {"state":"AL"} )`: Gibt alle Dokumente aus der `zips`-Collection zurück bei denen das Feld `state` den Wert `"AL"` (Alabama) hat.
- `db.zips.find( {"state":"AL"} ).count()`: Gibt die Anzahl der Dokumente zurück, die in der vorherigen Abfrage gefunden wurden (Dokumente, bei denen `state: "AL"` ist).
- `db.zips.find( {"city":"HEIDELBERG"} )`: Gibt alle Dokumente aus der `zips`-Collection zurück, bei denen das Feld `city` den Wert `"HEIDELBERG"` hat.
- `db.zips.find( {"city":"PARIS"} ).sort({"pop":-1}).limit( 2 )`: Findet alle Dokumente mit der Stadt `"PARIS"` und sortiert sie nach der Bevölkerungszahl (`pop`) in absteigender Reihenfolge.
- `db.zips.find( {"city":"PARIS", "pop":{$gt:16000} } )`: Gibt alle Städte namens `"PARIS"` zurück, bei denen die Bevölkerung (`pop`) grösser als 16.000 ist.

### A.3 Gib das Ergebnis der folgenden Anweisungen an und suche nach einer Er klärung, warum die Ergebnisse unterschiedlich sind:

- `db.zips.countDocuments()`: Gibt die Gesamtanzahl der Dokumente in der `zips`-Collection zurück.
- `db.zips.distinct("zip").length`: Gibt die Anzahl der eindeutigen Werte im Feld `zip` innerhalb der `zips`-Collection zurück.

### A.4  Wir gehen davon aus, dass eine Gemeinde eindeutig durch ihren Namen (city) und Bundesstaat (state) festgelegt ist.

#### Wie viele Gemeinden mit Namen PARIS gibt es in den USA? Gib die entsprechende Anweisung an und beantworte die Frage mit Begründung.

```javascript
db.zips.aggregate([
  { $match: { city: "PARIS" } },
  { $group: { _id: { city: "$city", state: "$state" } } },
  { $count: "totalParis" }
])
```
Antowrt:
- totalParis: 11

#### Gibt es eine Gemeinde mit Namen NEW YORK ausserhalb des Bundesstaates New York (NY)?

```javascript
db.zips.find(
  { city: "NEW YORK", state: { $ne: "NY" } }
)
```

Antwort:
- Es gibt keine Einträge.

### A.5 Die folgende Abfrage stellt beispielhaft eine Aggregation Pipeline vor, ein wesentliches Konzept zur Datenabfrage und -verarbeitung in MongoDB.

#### a) Identifiziere die 4 Stages (match, group, sort, limit) der Aggregation und beschreibe deren Funktion


- `$match`: Filtert Dokumente, bei denen `state: "CO"` (Colorado) ist.
- `$group`: Gruppiert die Dokumente nach `city`, berechnet die Gesamtbevölkerung (totalPop) für jede Stadt mithilfe von `$sum`: `"$pop"` und zählt die Anzahl der Dokumente (Einträge) für jede Stadt (`count: { $sum: 1 }`).
- `$sort`: Sortiert die Städte basierend auf `totalPop` in absteigender Reihenfolge (`-1`).
- `$limit`: Begrenzt die Ergebnisse auf die 4 bevölkerungsreichsten Städte.


#### b) Was gibt die Anweisung aus? Formuliere dies in eigenen Worten, z.B. «Kurzbezeichnung der US-Bundesstaaten mit Fläche, aufsteigend sortiert nach Fläche»

Die Anweisung gibt eine Liste der vier bevölkerungsreichsten Städte im US-Bundesstaat Colorado aus. Dabei wird zunächst gefiltert, dass nur Datensätze berücksichtigt werden, die dem Bundesstaat Colorado (CO) zugeordnet sind. Anschliessend werden die Daten nach Städten gruppiert, sodass die Gesamtbevölkerung jeder Stadt berechnet wird, indem die Bevölkerungszahlen aller zugehörigen Postleitzahlen summiert werden. Zusätzlich wird gezählt, wie viele verschiedene Postleitzahlen zu jeder Stadt gehören. Danach werden die Städte basierend auf ihrer Gesamtbevölkerung in absteigender Reihenfolge sortiert, und schliesslich werden nur die vier Städte mit der höchsten Gesamtbevölkerung in der Ausgabe angezeigt. Das Ergebnis enthält für jede Stadt die Informationen zur Gesamtbevölkerung und der Anzahl der Postleitzahlen, die zu dieser Stadt gehören.

#### c) Wie würde eine zu dieser Abfrage analoge SQL-Abfrage aussehen, wenn man davon ausgeht, dass eine Tabelle ZIPS mit entsprechenden Attributen existiert? Gib diese SQL-Anweisung an.

```javascript
SELECT city, SUM(pop) AS totalPop, COUNT(*) AS count
FROM zips
WHERE state = 'CO'
GROUP BY city
ORDER BY totalPop DESC
LIMIT 4;
```

### A.6 Ändere die Abfrage aus A.5, indem in der group-Stage "$city" durch null ersetzt wird. Was hat dies für einen Effekt, was wird ausgegeben? (In eigenen Worten, keine reine Auflistung des Ergebnisses.)

Wenn in der `group`-Stage der Aggregation `$city` durch `null` ersetzt wird, hat dies den Effekt, dass alle Dokumente aus der vorherigen `match`-Stage in eine einzige Gruppe zusammengefasst werden. Anstelle der Gruppierung nach einzelnen Städten wird somit eine Gesamtauswertung über den gesamten Bundesstaat Colorado durchgeführt.

Das Ergebnis der Abfrage enthält dann die Gesamtbevölkerung aller Städte in Colorado sowie die Anzahl der Postleitzahlen, die in der Datenbank für Colorado existieren. Es wird keine Aufteilung nach Städten mehr vorgenommen, sondern nur eine aggregierte Darstellung für den gesamten Bundesstaat gezeigt.

### A.7 Verwende A.5 als Grundlage und formuliere eine Abfrage, welche die 3 Bundesstaaten mit den wenigsten Einwohnern ermittelt und diese (zusammen mit der Einwohnerzahl) in ausgibt, wobei die Ausgabe nach Bundesstaat aufsteigend ausgegeben werden soll.

```javascript
db.zips.aggregate([
  { $group: { 
      _id: "$state", 
      totalPop: { $sum: "$pop" } 
    } 
  },
  { $sort: { totalPop: 1 } },
  { $limit: 3 } 
])
```

### A.8 Verwende A.5 als Grundlage und formuliere eine Abfrage, welche die 6 grössten (bzgl. Einwohnerzahl) Gemeinden der USA zusammen mit deren Einwohnerzahl in absteigender Reihenfolge ausgibt. (Beachte die Bemerkung in A.4)

```javascript
db.zips.aggregate([
  { $group: { 
      _id: { city: "$city", state: "$state" }, 
      totalPop: { $sum: "$pop" } 
    } 
  },
  { $sort: { totalPop: -1 } },
  { $limit: 6 }
])
```

### A.9 Wie viele Routen gibt es jeweils mit 0, 1, 2, … Zwischenstopps (Attribut/Key: stops)? Formuliere hierzu im Idealfall eine einzige Abfrage, die diese Frage beantwortet.

```javascript
db.routes.aggregate([
  { $group: { 
      _id: "$stops", 
      count: { $sum: 1 } 
    } 
  },
  { $sort: { _id: 1 } }
])
```

Antwort:
- _id: 0, count: 66974
- _id: 1, count: 11

### A.10 Wie viele Airlines starten am Flughafen München (MUC)?

```javascript
db.routes.aggregate([
  { $match: { src_airport: "MUC" } },
  { $group: { 
      _id: "$airline", 
      count: { $sum: 1 } 
    } 
  },
  { $count: "totalAirlines" }
])
```

Antwort:
- totalAirlines: 77

### A.11 Welche Flughäfen sind vom Flughafen Zürich direkt (ohne Umsteigen) erreichbar? (Das Attribut «stops» kann hierbei ignoriert werden. Selbst wenn stops>0 muss man als Passagiert das Flugzeug nicht verlassen.)

```javascript
db.routes.aggregate([
  { $match: { src_airport: "ZRH" } },
  { $group: { 
      _id: "$dst_airport" 
    } 
  }
])
```

Antwort:
{ _id: 'LGW' }, { _id: 'CGN' },
{ _id: 'GDN' }, { _id: 'RMF' },
{ _id: 'FUE' }, { _id: 'ATH' },
{ _id: 'AMM' }, { _id: 'SOF' },
{ _id: 'LED' }, { _id: 'DXB' },
{ _id: 'PRN' }, { _id: 'HAJ' },
{ _id: 'VCE' }, { _id: 'MCT' },
{ _id: 'LCA' }, { _id: 'TGD' },
{ _id: 'MUC' }, { _id: 'ATL' },
{ _id: 'DME' }, { _id: 'HAM' }

#### Wie viele Flughäfen sind vom Flughafen Zürich direkt erreichbar?

```javascript
db.routes.aggregate([
  { $match: { src_airport: "ZRH" } }, 
  { $group: { 
      _id: "$dst_airport" 
    } 
  },
  { $count: "totalDestinations" } 
])
```

Antwort:
- totalDestinations: 137

### A.12 Gibt es einen Flughafen, von dem aus jeder (andere) Flughafen ohne Umsteigen erreichbar ist? Es gibt verschiedene Möglichkeiten, wie man argumentieren kann. 

```javascript
db.routes.aggregate([
  { $match: { stops: 0, dst_airport: { $ne: NaN } } },
  { $group: { _id: {
    "src_airport": "$src_airport",
    "dst_airport": "$dst_airport"
  } } },
  { $group: {
    _id: "$_id.src_airport",
    "totalCount": { "$sum": 1 }
  } },
  { $sort: { "totalCount": -1 } },
  { $limit: 1 }
])
```

Antwort:
- _id: 'FRA', totalCount: 239


### A.13 Welche Flughäfen sind von Frankfurt (FRA) mit einmal Umsteigen erreichbar, die nicht direkt erreichbar sind? «Einmal umsteigen» bedeutet, dass man einen Anschlussflug benutzen darf; gibt es also einen Flug A→B und einen Flug B→C, dann ist C von A mit einmal Umsteigen (nämlich in B) erreichbar. (Bitte Flughäfen nicht alle ausgeben, es sind recht viele.)

```javascript
db.routes.aggregate([
  { $match: { src_airport: "FRA" } }, 
  {
    $lookup: {
      from: "routes",
      localField: "dst_airport", 
      foreignField: "src_airport", 
      as: "second" 
    }
  },
  {
    $project: {
      _id: 0,
      direct: "$dst_airport", 
      second: "$second.dst_airport" 
    }
  },
  { $unwind: "$second" }, 
  {
    $facet: {
      direct: [{ $group: { _id: "$direct" } }],
      second: [{ $group: { _id: "$second" } }] 
    }
  },
  {
    $project: {
      common: { $setDifference: ["$second._id", "$direct._id"] }
    }
  },
  { $unwind: "$common" }, 
  { $group: { _id: "$common" } },
  { $sort: { _id: -1 } } 
]);
```

#### Wie viele Flughäfen sind dies?

```javascript
db.routes.aggregate([
  { $match: { src_airport: "FRA" } }, 
  {
    $lookup: {
      from: "routes",
      localField: "dst_airport", 
      foreignField: "src_airport", 
      as: "second" 
    }
  },
  {
    $project: {
      _id: 0,
      direct: "$dst_airport", 
      second: "$second.dst_airport" 
    }
  },
  { $unwind: "$second" }, 
  {
    $facet: {
      direct: [{ $group: { _id: "$direct" } }],
      second: [{ $group: { _id: "$second" } }] 
    }
  },
  {
    $project: {
      common: { $setDifference: ["$second._id", "$direct._id"] }
    }
  },
  { $unwind: "$common" }, 
  { $group: { _id: 0, count: { $sum: 1 } } }
]);
```

Antwort:
- _id: 0, count: 1735

### B.1 Welche verschiedenen Bett-Typen gibt es?

```javascript
db.listingsAndReviews.distinct("bed_type")
```

### B.2 Wie viele AirBnB-Plätze (Dokumente) gibt es, deren Adresse in Portugal liegt und die über ein echtes Bett (siehe B.1) verfügen?

```javascript
db.listingsAndReviews.countDocuments({
  "address.country_code": "PT",
  "bed_type": "Real Bed"       
})
```

### B.3 Ermittle für jeden AirBnB-Platz mit Bett den Preis pro Bett Anmerkung: Viele Plätze haben mehr als ein Bett, wobei «price» den Preis für den gesamten Platz, also alle Betten, angibt.

```javascript
db.listingsAndReviews.aggregate([
  { $match: { beds: { $gt: 0 } } }, 
  { $project: { 
      name: 1, 
      pricePerBed: { $divide: [ "$price", "$beds" ] } 
    } 
  }
])
```

### B.4 Welche AirBnB-Plätze in Portugal mit mindestens 3 und höchstens 5 Betten haben den günstigsten Preis pro Bett. Gib die 10 günstigsten Plätze sortiert nach Preis pro Bett aus, den günstigsten zuerst.

```javascript
db.listingsAndReviews.aggregate([
  { $match: { 
      "address.country_code": "PT", 
      beds: { $gte: 3, $lte: 5 }   
    } 
  },
  { $project: { 
      name: 1, 
      address: 1, 
      pricePerBed: { $divide: [ "$price", "$beds" ] } 
    } 
  },
  { $sort: { pricePerBed: 1 } }, 
  { $limit: 10 }
])
```

### B.5 Ermittle die 10 häufigsten Namen der Gastgeber und gebe diese zusammen mit der Häufigkeit ihres Vorkommens (in absteigender Reihenfolge) aus. (Wir betrachten den Wert des Keys host.host name als Namen, ohne diesen in Vor- und Nachname zu trennen.) 

```javascript
db.listingsAndReviews.aggregate([
  { $group: { 
      _id: "$host.host_name",
      count: { $sum: 1 }    
    } 
  },
  { $sort: { count: -1 } }, 
  { $limit: 10 }            
])
```

### B.6 Du besuchst Porto (Portugal) mit 3 weiteren Personen und suchst eine Unterkunft mit genau 4 Betten, die nicht mehr als 1200 Meter von der Kathedrale von Porto (Geokoordinaten:-8.611311, 41.142347) entfernt ist. Gib die drei nächst gelegenen Resultate aus, die diese Bedingungen erfüllen.

```javascript
db.listingsAndReviews.aggregate([
  { 
    $geoNear: {
      near: { type: "Point", coordinates: [-8.611311, 41.142347] }, 
      distanceField: "distance",
      maxDistance: 1200, 
      query: { 
        "address.country": "Portugal",
        beds: 4 
      },
      spherical: true
    }
  },
  { $limit: 3 }, 
  { $project: { 
      name: 1, 
      distance: 1,
      price: 1, 
      address: 1 
    } 
  }
])
```

### C.1a Ermittle alle Personen, die in Filmen mitgespielt haben (ACTED_IN), in denen auch Meg Ryan mitgespielt hat. Meg Ryan soll auch ausgegeben werden. 

```cypher
MATCH (megRyan:Person {name: 'Meg Ryan'})-[:ACTED_IN]->(movie:Movie)<-[:ACTED_IN]-(actor:Person)
RETURN DISTINCT actor.name AS Name
UNION
MATCH (megRyan:Person {name: 'Meg Ryan'})
RETURN megRyan.name AS Name
```

### C.1b Ermittle alle Filme, in denen Meg Ryan gespielt hat, zusammen mit allen anderen Personen, die ebenfalls in diesen Filmen mitgespielt haben.

```cypher
MATCH (megRyan:Person {name: 'Meg Ryan'})-[:ACTED_IN]->(movie:Movie)<-[:ACTED_IN]-(otherActor:Person)
RETURN movie.title AS Film, collect(otherActor.name) AS Schauspieler
```

### C.2a Gib alle Regieführenden und Filme aus, bei denen die Regie führende Person (DIRECTED) auch mitgespielt hat.

```cypher
MATCH (director:Person)-[:DIRECTED]->(movie:Movie)<-[:ACTED_IN]-(director)
RETURN director.name AS Regisseur, movie.title AS Film
```

### C.2b Beschreibe mit eigenen Worten (kein Aufzählen der Ergebnisliste), was die folgende Anweisung ausgibt. (Hierbei ist „Bla Blo“ eine Person mit diesem Namen):

```cypher
MATCH (p:Person)-[related1]->(m)<-[related2]-(meg:Person{name:"Bla Blo"})
RETURN p.name, type(related1), type(related2),meg.name
```

Die Abfrage gibt die Namen der Personen zurück, die über einen gemeinsamen Knoten mit „Bla Blo“ in Beziehung stehen, sowie die Art der Verbindungen, die diese Beziehungen definieren. Es zeigt somit, wie „Bla Blo“ mit anderen Personen im Netzwerk (direkt oder indirekt) durch bestimmte Knoten und Beziehungstypen verknüpft ist.

### C.3a In wie vielen Filmen hat Meg Ryan mitgespielt (aus Schauspielende)?

```cypher
MATCH (meg:Person {name: "Meg Ryan"})-[:ACTED_IN]->(movie:Movie)
RETURN COUNT(movie) AS NumberOfMovies
```

Antwort:
- NumberOfMovies: 5

### C.3b In wie vielen Filmen hat Tom Hanks mitgewirkt (als Schauspielender, Regisseur oder wie auch immer)?

```cypher
MATCH (tom:Person {name: "Tom Hanks"})-[relation]->(movie:Movie)
RETURN COUNT(movie) AS NumberOfMovies
```

Antwort:
- NumberOfMovies: 13

### C.4 Wann sind jeweils der/die jüngste und der/die älteste Schauspielende, Regie führende, Produzierende etc. (was auch immer) geboren? (Für jede Beziehungsart eine „Ausgabezeile“) (Im Idealfall eine einzige Anweisung.)

```cypher
MATCH (person:Person)-[relation]->(movie:Movie)
WHERE person.born IS NOT NULL
RETURN type(relation) AS RelationType,
       MIN(person.born) AS OldestPersonBorn,
       MAX(person.born) AS YoungestPersonBorn
ORDER BY RelationType;
```

### C.5a Wann ist der/die jüngste und älteste Person geboren, die in einem Film mitgespielt hat, in dem auch Meg Ryan gespielt hat?

```cypher
MATCH (meg:Person {name: "Meg Ryan"})-[:ACTED_IN]->(movie)<-[:ACTED_IN]-(actor:Person)
WHERE actor.born IS NOT NULL
RETURN min(actor.born) AS ÄltesterGeburtstag, max(actor.born) AS JüngsterGeburtstag
```

### C.5b Wie gross ist der maximale Altersunterschied zwischen zwei Schauspielenden des gleichen Films? 

```cypher
MATCH (actor1:Person)-[:ACTED_IN]->(movie)<-[:ACTED_IN]-(actor2:Person)
WHERE actor1.born IS NOT NULL AND actor2.born IS NOT NULL AND actor1 <> actor2
RETURN max(abs(actor1.born - actor2.born)) AS MaximalerAltersunterschied
```
