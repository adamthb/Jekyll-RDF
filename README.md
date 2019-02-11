# Modulkatalog mit Jekyll-RDF

![modulkatalog collage](https://github.com/adamthb/Jekyll-RDF/blob/master/home.png)

## Übersicht

Auf Basis der semantischen RDF-Ressourcen des Modulkatalogs im Master-Studiengang Wirtschaftsinformatik werden statische HTML-Seiten mit einer PDF-Export-Funktion mithilfe von Jekyll-RDF generiert. Der Prototyp wird im Rahmen des Moduls "Enterprise Knowledge Graph Implementation" entwickelt und ist unter <http://univera.de/FHB/EKGI_ModCat/> erreichbar.

##Angewandte Technologien

### Bei Installation von Jekyll-RDF

- [Ruby](https://www.ruby-lang.org/) >= 2.1
- [RubyGems](https://rubygems.org/)
- [Bundler](https://bundler.io/)
- [Jekyll](https://jekyllrb.com/)
- [Jekyll-RDF](https://github.com/white-gecko/JekyllRDF-Tutorial/)


### Bei Erstellung der Templates für statische Seiten

- [SparQL](https://www.w3.org/TR/sparql11-overview/) version 1.1
- [Liquid](https://shopify.github.io/liquid/)


## Jeykyll-RDF Konfiguration

### Parameter

Die Parameter, die für Konfiguration von Jekyll-RDF nötig sind, befinden sich in `_config.yml`. `url` und `baseurl` werden verwendet, um den Pfad der Hauptressourcen im Root-Pfad umzufassen. Plugin sowie der Pfad der RDF-Ressource (Wissensbasis) müssen definiert werden.

```yaml
baseurl: ""
url: "https://bmake.th-brandenburg.de/module"

plugins:
   - jekyll-rdf

jekyll_rdf:
   path: "_data/graph.ttl"

```

### Ressourcen Beschkänkung und Mapping

Außerdem gibt es die Möglichkeit, in der `_config.yml` die zu generierenden Seiten anhand einer SparQL-Query zu beschränken oder die Ressourcen den Templates zuzuordnen. Z. B. Die Seiten aller Ressourcen, die einen Typ von "https://schema.org/Course" besitzen, werden mit dem Template "course.html" erstellt.

```yaml
restriction: "SELECT ?resourceUri WHERE { { ?resourceUri a <https://schema.org/Course> . } UNION  { ?resourceUri a <https://schema.org/WebPage> } UNION { ?resourceUri a <https://bmake.th-brandenburg.de/module/Lecturer>.}}"
class_template_mappings:
    "https://schema.org/WebPage": "index.html"
    "https://schema.org/Course": "course.html"
    "https://bmake.th-brandenburg.de/module/Lecturer": "person.html"
```

## Aufbau der Templates

Hier handelt es sich um [Liquid-Templates](https://shopify.github.io/liquid/). Im Folgenden werden einige Beispiele aus dem Template "course.html" angezeigt.

### Hauptressouce für eine Seite

Die Seite wird mit einer Hauptressource aufgebaut, die später als Subjekt für alle SparQL-Queries verwendet wird.

    {{ page.rdf }}

### Property mit einzelnem Objekt

```
Englische Modulbeizeichung: {{ page.rdf | rdf_property: '<https://schema.org/name>','en'}}
```

### Property mit mehreren Objekten

Da es mehrere Objekte gibt, wird ein Array zurückgegeben. Um die Werte im Array anzuzeigen, wird eine For-Schleife gebraucht.
```
<ul>
    {% assign results = page.rdf | rdf_property: '<https://schema.org/learningResourceType>', nil, true %}
    {% for res in results %}
    <li> {{res}}<br /></li>
    {% endfor %}
</ul>
```

### Ressourcen mit SparQL

Darüber hinaus gibt es eine Möglichkeit, die Ressoucen über mehrere verkettete Knoten mit SparQL abzufragen. `?resourceUri` ist hier ein Platzhalter, der später durch page.rdf.iri (IRI für die Hauptressource der Seite) ersetzt wird.

```
<td>
    {% assign instructor_query = 'SELECT DISTINCT ?person WHERE {?resourceUri <https://schema.org/accountablePerson> ?person. }' %}
    {% assign resultset = page.rdf | sparql_query: instructor_query %}
    {% for result in resultset %}
    {{ result.person | rdf_property: '<http://www.w3.org/2000/01/rdf-schema#label>' }}<br />
    {% endfor %}
</td>
```

oder 

```
<td>
    {% capture instructor_query %}
    SELECT DISTINCT ?instructor WHERE {?resourceUri <https://schema.org/hasCourseInstance> ?instance. ?instance a <https://schema.org/CourseInstance>; <https://schema.org/instructor> ?instructor.}
    {% endcapture %}
    
    {% assign resultset = page.rdf | sparql_query: instructor_query %}
    {% for result in resultset %}
    {{ result.person | rdf_property: '<http://www.w3.org/2000/01/rdf-schema#label>' }}<br />
    {% endfor %}
</td>
```







