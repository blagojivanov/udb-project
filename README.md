# Лабораториска вежба за граф бази на податоци по предметот Неструктурирани Бази на Податоци

## Вовед и инсталација на neo4j
Neo4j е високо перформансна, оpen-source граф база на податоци наменета за ефикасно складирање, управување и пребарување на поврзани податоци. За разлика од традиционалните релациони бази на податоци кои користат табели и редови, Neo4j користи граф структура составена од јазли, релации и својства. Овој модел природно ги претставува реалните системи и нивните врски, што го прави идеален за апликации како социјални мрежи, препорачувачки системи, откривање измами и knowledge графови.

Во сржта на Neo4j е моделот на „property graph“, каде јазлите претставуваат ентитети (на пример луѓе или производи), релациите претставуваат врски меѓу нив (како „ПРИЈАТЕЛ СО“ или „КУПИЛ“), а и јазлите и релациите можат да имаат својства во форма на key-value парови. Ова овозможува брзо и ефикасно следење на сложени релации со користење на Cypher — декларативен јазик за пребарување специфичен за Neo4j.

Neo4j нуди поддршка за ACID трансакции, добра интеграција со современи програмски технологии, како и локално и cloud хостирање. Идеален е за секоја апликација каде што поврзаноста е подеднакво важна како и самите податоци.

### Вовед во Cypher

**Cypher** е декларативен јазик за прашалници дизајниран специјално за работа со граф бази на податоци како што е **Neo4j**. Наместо табели и редови, Cypher овозможува лесно и читливо изразување на врски меѓу ентитети преку ASCII-стилска синтакса. На пример, јазлите се претставуваат со загради `()`, а релациите со стрелки `-->`.

---

#### Основни CRUD операции во Cypher

##### Create (Креирање)

```cypher
CREATE (p:Person {name: "Anna", age: 30})
```

Креира јазол со label `Person` и својства `name` и `age`.

```cypher
MATCH (a:Person {name: "Anna"}), (b:Person {name: "Bart"})
CREATE (a)-[:FRIEND_WITH]->(b)
```

Се креира релација `FRIEND_WITH` помеѓу два јазли.
a
---

##### Read (Читање / Пребарување)

```cypher
MATCH (p:Person) RETURN p
```

Ги враќа сите јазли со лабела `Person`.

```cypher
MATCH (a:Person)-[:FRIEND_WITH]->(b:Person) RETURN a.name, b.name
```

Ги враќа имињата на сите лица што се пријатели.

---

##### Update (Ажурирање)

```cypher
MATCH (p:Person {name: "Anna"})
SET p.age = 31
```

Ја ажурира возраста на Ана.

```cypher
MERGE (p:Person {name: "Чедо"})
SET p.age = 25
```

Доколку не постои јазолот се креира нов, или ажурира постоечки.

---

##### Delete (Бришење)

```cypher
MATCH (p:Person {name: "Чедо"})
DETACH DELETE p
```

Го брише јазолот и сите негови релации.

---
Секако! Еве го објаснувањето за **филтрирање**, **агрегации** и **најкратки патишта** во **Cypher**, на македонски јазик:

---

#### Филтрирање (WHERE услови)

`WHERE` се користи за филтрирање на резултатите според одредени услови:

За да ги добиеме лицата постари од 30 години:
```cypher
// Лица постари од 30 години
MATCH (p:Person)
WHERE p.age > 30
RETURN p.name, p.age
```

За да ги добиеме лицата чие име почнува на "A"
```cypher
MATCH (p:Person)
WHERE p.name STARTS WITH "А"
RETURN p
```

Филтрирање според својства на релација

```cypher
MATCH (a:Person)-[r:FRIEND_WITH]->(b:Person)
WHERE r.since > 2020
RETURN a.name, b.name
```

---

#### Агрегации (COUNT, AVG, SUM, итн.)**

Cypher поддржува агрегатни функции за групирање и анализа:

За да изброиме колку јазли има со label `Person` во базата:

```cypher
MATCH (p:Person)
RETURN count(p) AS vkupno_lica
```
За да пронајвеме просечна возраст на лицата:
```cypher
MATCH (p:Person)
RETURN avg(p.age) AS prosecna_vozrast
```

За да изброиме колку лица има по град и сортирање според бројот во опаѓачки редослед:
```cypher
MATCH (p:Person)
RETURN p.city, count(*) AS broj_lica
ORDER BY broj_lica DESC
```

---

#### Најкратки патишта (shortestPath) и Dijkstra**

Со `shortestPath()` можеш да најдеш најкратка врска помеѓу два јазла:

За да го пронајдеме најкраткиот пат од Anna до Bob

```cypher
MATCH (a:Person {name: "Anna"}), (b:Person {name: "Bob"})
MATCH path = shortestPath((a)-[:FRIEND_WITH*]-(b))
RETURN path
```
Со операторот .. може да се ограничи и бројот на чекори за пребарување:

```cypher
MATCH (a:Person {name: "Anna"}), (b:Person {name: "Bob"})
MATCH path = shortestPath((a)-[:FRIEND_WITH*..4]-(b))
RETURN path
```
На големи графови, `shortestPath()` може да биде бавно. Може да се искористи `LIMIT`, `WHERE` или `*..n` за ограничување на длабочината.

#### Dijkstra

Neo4j работи со in-memory проекции за алгоритми од библиотеката GDS

1. Потребно е да се направи проекција на графот
```cypher
CALL gds.graph.project(
  'cityGraph',                  // име на проектираниот граф
  'City',                       // јазли
  {
    ROAD: {
      type: 'ROAD',
      properties: 'distance'    // тежинско својство
    }
  }
)
```
2. Се повикува функцијата за да се пронајде најкраткиот пат:
```cypher
CALL gds.shortestPath.dijkstra.stream('cityGraph', {
  sourceNode: gds.util.asNodeId($start),  // ID на почетниот јазол
  targetNode: gds.util.asNodeId($end),    // ID на целниот јазол
  relationshipWeightProperty: 'distance'
})
YIELD index, sourceNode, targetNode, totalCost, nodeIds, costs
RETURN
  gds.util.asNode(sourceNode).name AS from,
  gds.util.asNode(targetNode).name AS to,
  totalCost,
  [nodeId IN nodeIds | gds.util.asNode(nodeId).name] AS path
```
3. Потребно е да се избрише проекцијата
```cypher
CALL gds.graph.drop('cityGraph')
```

### Сложени прашалници
#### `OPTIONAL MATCH` – за необврзувачки врски (left outer join)

```cypher
MATCH (p:Person)
OPTIONAL MATCH (p)-[:FRIEND_WITH]->(f:Person)
RETURN p.name, f.name
```

#### `WITH` – за пренесување резултати или агрегации помеѓу чекори

```cypher
MATCH (p:Person)-[:FRIEND_WITH]->(f)
WITH p, count(f) AS friendCount
WHERE friendCount > 5
RETURN p.name, friendCount
```
Се користи и за **филтрирање по агрегати**, **групирање** или **paging** (`SKIP`, `LIMIT`).

#### `UNWIND` – од листа прави редови

```cypher
UNWIND [1, 2, 3] AS x
RETURN x * 2 AS doubled
```

Или за batch внесување:

```cypher
UNWIND [
  {name: "Ана", age: 25},
  {name: "Бојан", age: 30}
] AS row
CREATE (:Person {name: row.name, age: row.age})
```
#### `FOREACH` – за извршување акција врз елементи од листа

```cypher
MATCH (p:Person {name: "Ана"})
SET p.tags = ["dev", "neo4j"]

FOREACH (tag IN p.tags |
  CREATE (p)-[:HAS_TAG]->(:Tag {name: tag})
)
```

#### `MERGE` – „match or create“

```cypher
MERGE (p:Person {name: "Ана"})
ON CREATE SET p.age = 25
ON MATCH SET p.lastSeen = date()
```

### Индекси и ограничувања

Индексите значително го забрзуваат `MATCH` со филтри (`WHERE p.name = "Ана"`).
#### Креирање индекс:

```cypher
CREATE INDEX FOR (p:Person) ON (p.name)
```

#### UNIQUE Constraint:

```cypher
CREATE CONSTRAINT FOR (p:Person) REQUIRE p.email IS UNIQUE
```