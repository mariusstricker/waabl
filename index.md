---
layout: default
---

In dieser Projektdokumentation werden die verschiedenen Arbeitsschritte von der Idee einer virtuellen Ausstellung mit IIIF-Images bis zur veröffentlichten Website beschrieben. Die Umsetzung des Projekts basiert auf dem Workflow [Wax](https://minicomp.github.io/wiki/wax/) von [Minicomp](https://github.com/minicomp/), sämtliche Files befinden sich im Repositorium Leminbi auf GitHub, die Projektwebsite enthält ergänzende Informationen.


## Projektziel
Das Ziel meiner Projektarbeit bestand ursprünglich darin, mit [Annonatate (Herokuapp)](https://annonatate.herokuapp.com/) eine Sammlung mit annotationsfähigen, metadatenangereicherten IIIF-Images zu erstellen und diese über eine Projektwebsite online zugänglich zu machen; ursprünglich deshalb, weil sich im Verlauf des Projekts herausstellte, dass der Weg über Annonatate hätte übersprungen werden können. Aber dazu später mehr.  

Das Projektziel als Fragestellung formuliert: Wie lässt sich eine Sammlung IIIF-konformer Bilder vollständig und effizient über eine Website publizieren?


## Eckdaten

Auf Vorschlag der beiden Betreuer habe ich für den webbasierten Zugang zur geplanten Sammlung mit Wax gearbeitet. Die Entwickler beschreiben ihren Workflow wie folgt: «Wax is a minimal computing (minicomp) project […] for producing digital exhibitions focused on longevity, low costs, and flexibility.» ([Quelle](https://minicomp.github.io/wax/)) Für Installation und das spätere Arbeiten mit dem Workflow, steht ein Wiki zur Verfügung, inklusive Manual, zur schrittweisen Aufsetzung des Workflows. Der Umfang des gesamten Workflows skizziert folgende Visualisierung:

![Workflow](https://github.com/mariusstricker/waabl/blob/himbel/assets/img/wax_workflow.jpg)

[Quelle](https://minicomp.github.io/wiki/assets/wax_workflow.jpeg)

Der Wax Workflow  enthält Ruby Gems für Bildprozessierung und statisches Webpublishing (Jekyll, Markdown und YAML-Config), zudem werden HTML, CSS und JavaScript angewendet und Git Bash für Befehle zum Datenaustausch zwischen lokalem und Git-Repository. 

Alle Installations- und Prozessbefehle für die Arbeiten auf dem lokalen Rechner wurden in Visual Studio Code (Terminal) mit der Linux-Distribution Ubuntu durchgeführt. 

Diese Projektdokumentation folgt, wenn nicht speziell darauf hingewiesen wird, den Prozessschritten im [Wax Wiki](https://minicomp.github.io/wiki/wax/).

Projekt- und Sammlungsbezeichnung: Glasdiashow.


## Systemsetup

Bei ersten Requirement-Checks und Installationsversuchen hat sich relativ rasch herausgestellt, dass es keine gute Idee war, wie ursprünglich den Workflow auf Desktopebene des Rechners (PHZH) aufsetzen zu wollen, weil der Desktop wiederum auf dem Filehosting-Dienst OneDrive läuft:

```js
(base) marius@WN128567:/mnt/c/Users/marius.stricker/OneDrive - PHZH/Desktop
```
Die beiden Commands:
```
> sudo apt install bundler        _#ruby gem
```
```
> sudo apt install libvips-tools  _#fast image processing library
```
erzeugten wiederholt unlösbare Probleme. So dass ich das angelegte Projekt auf das Laufwerk C: verschob und den Installationsprozess erneut begann. Vermutlich hätte OneDrive später tatsächlich Probleme erzeugt, aber für die Installation führte nicht OneDrive zu Fehlermeldungen, sondern weil die Commands nicht auf Betriebssystemebene (globally) ausgeführt wurden. 

Nach Anpassung des Directory und den wiederholten Installationen wurden bei der Überprüfung die korrekten Versionen ausgegeben:

```
> ruby -v             _#ruby 2.7.0p0 (2019-12-25 revision 647ee6f091) [x86_64-linux-gnu]
> bundler -v          _#Bundler version 2.1.4
> git –version        _#git version 2.25.1
> convert -version    _#GraphicsMagick 1.3.35
> gs -version         _#hostscript 9.50 (2019-10-15)
> vips -version       _#vips-8.9.1 (Laufwerk C :)

```

## Sammlungs- und Metadaten aufbereiten

Beim Aufbereiten des Metadatenfiles traten mehrere, teils hartnäckige Probleme auf. Gemäss Wiki kann Wax mit .csv, .json und .yml umgehen. Von Microsoft Excel wird dringend abgeraten, empfohlen seien Google Sheets. Aus Routine öffnete ich die heruntergeladene Datei nicht mit einem Editor, sondern automatisch mit Excel, dachte aber, dass es in Ordnung ist, wenn ich dabei den weiteren Hinweis für die Sprachencodierung UTF-8 berücksichtige und die Metadaten aus dem Google-File in ein UTF-8-CSV-File kopiere. 

Die Metadatenfelder habe ich 1:1 von Wax übernommen, Feldbezeichnungen mit zwei oder mehreren Termen müssen zwingend mit _ (underline) verbunden werden:

*   PID 
*   artist 
*   location 
*   label
*   _date
*   object_type 
*   current_location 
*   source
*   order

#### Fehleranalyse (1)
Die späteren wax:tasks können, wie befürchtet, keine Daten aus einem Excel-File verarbeiten, das Terminal gibt als Fehlermeldung zurück, dass die PID der ersten Zeile fehlt. Der Grund für die Schwierigkeiten mit Excel sind unsichtbare Formatierungen und Zeichen, die von den wax:tasks nicht interpretiert werden können. Zwar versuchte ich diese über OpenRefine zu tilgen, aber offenbar erfolglos.

#### Lösung (1)
Metadateninhalte in den Microsoft Editor kopieren und als .csv-Datei abspeichern; der Editor in VS Code eignete sich dafür nicht, weil er kein CSV-Format generieren kann.


## Setup Website

Dafür wird das Git-Repository [Wax](https://github.com/minicomp/wax) in den eigenen Workspace kopiert (Use this template), anschliessend auf den lokalen Rechner geklont, bereinigt und mit den Collection Images sowie dem aufbereiteten Metadaten-File ergänzt. 

VS Code Terminal
```
*   git clone                           _#anschliessend ins Repository wechseln
*   bundle exec rake wax:clobber qatar  _#löscht alle Files aus den Directories _qatar und img/derivatives (oder manuelll)

```
```
*   alle Qatar-Files aus _data löschen
*   Image Collection und Metadaten im Repository hinterlegen
*   Anpassen des _config.yml-File (Dokumentation im Wiki)
```


### Wax:Tasks
VS Code Terminal, Repository

```
* bundle exec rake –tasks                             _#Übersicht aller wax:tasks
* bundle exec rake wax:derivatives:iiif Glasdiashow   _#generiert IIIF-Derivatives: JSON-Manifeste mit Image- und Präsentations-API, URI, Image Tiles und JSON-Dokumente für jedes Image pro Einheit (Canvas, Annotation und Sequence)
                                                      _#fügt im Metadaten-File drei zusätzliche Felder (full, thumbnail, manifest) hinzu und schreibt für jedes Objekt mit PID entsprechende Inhalte aus den JSON-Dateien 
```
```
Rechenzeit für 20 Images von je 14 Megabyte circa 2.5h
```
```
> bundle exec rake wax:pages Glasdiashow              _erstellt im lokalen Repository ein Directory mit _Glasdiashow #schreibt pro Objekt je ein Markdown-File mit Daten aus dem Metadaten-File für die spätere Repräsentation von Images/Metadaten im Viewer OpenSeadragon
> bundle exec rake wax:search                         _schreibt einen JSON-Index anhand der Markdown-Files der Objekte
```

### Website testen

Die Projektwebsite ist nun (fast) bereit, um online zu gehen, zuvor gibt es aber noch ausführliche Tests mit der Online-Seite über den lokalen Server (localhost:4000).

```
> bundle exec jekyll serve                            _#erstellt im lokalen Repository das Directory _site
```
Das site-Directory enthält:
*   Index-File (HTML) für die Homepage und für die 404-Seite
*   pro Seite/Unterseite je ein HTML-File (28) 
*   Directory img mit den IIIF-Files (JSON)  #dupliziert aus Repository
*   Directory assets mit den Booting- und Style-Files (.js, .min, .css, .min.js) #dupliziert aus Repository

```
Der Prozess zum Aufbauen der Website: > _config.yml > _site > assets > Index > …
```

**Aber die Website konnte nicht geladen werden.**


#### Fehleranalyse (2)
Der Workflow sieht leider keine vollständige File-Kontrolle vor den wax:tasks vor, was sehr nützlich wäre, und so konnte ich erst bei den Website-Tests feststellen, dass das File fehlerhaft war und somit auch fehlerhafte JSON- und Markdown-Files geschrieben wurden. 

Mir scheint es rätselhaft, wie der Workflow funktionieren soll, wenn wie die Metadaten tatsächlich, wie auf dem [Wax-Repository (Git)](https://github.com/minicomp/wax-facets/edit/main/_data/qatar.csv) ersichtlich, mit Kommas getrennt werden. Denn während Images und Metadaten später mit dem Task wax:derivatives:iiif zu IIIF-Manifesten mit API und URI verarbeitet werden, schreibt der Code Kommaelemente in die Thumbnail-URI:

```
/img/derivatives/iiif/images/obj1/full/250,/0/default.jpg
```

Logischerweise werden genau diese Thumbnail-Kommas vom Code als Trennungszeichen interpretiert, was zu einem fehlerhaften Metadatenfile als Kernelement des gesamten Workflows führte. Auf dieser Basis wurden fehlerhafte Markdown-Files für das Auslesen der IIIF-Manifeste generiert und der Workflow hob nicht ab.   

Der erste Lösungsweg mit Semikolons anstatt Kommas als Trennungszeichen im Metadaten-File kann wiederum auf GitHub nicht richtig ausgelesen werden. So sieht die [Tabelle korrekt](https://github.com/mariusstricker/leminbi/blob/main/_data/arumacula.tsv) aus, so [falsch mit Semikolons](https://github.com/mariusstricker/arumacula/blob/main/_data/Glasdias_original.csv).  

#### Lösung (2)
Weil .tsv für Erstellungsprozess der IIIF-Manifeste (wax:derivatives:iiif)nicht unterstützt wird, sieht die tatsächliche Lösung wie folgt aus:
1. Metadatenfile (.csv) mit Semikolon für die Tasks aufbereiten
1. dadurch werden es korrekt verarbeitet, d.h. mit den Manifest-URIs ergänzt und die Markdown-Files der Manifeste korrekt geschrieben
1. für das Webpublishing mit Git Page die Semikolons durch Kommas, besser durch Tabs ersetzen, dies aber unbedingt in einem Editor, weil Tabs, die im Git-File direkt gesetzt werden, von Git nicht als solche interpretiert werden (!).


### Fehleranalyse (3)
Ursprüngliches Git-Repository Glasdiashow hatte ich nicht mit identischem Namen ins lokale Repository geklont, sondern ins Directory C:/Glasdiashow/Repository, was später bei den Tests über den lokalen Server (localhost:4000) 404-Fehler erzeugte.

Zudem erwies es sich als fatal, auf GitHub das Projekt und die Sammlung identisch zu bezeichnen (beide Glasdiashow), denn dadurch wurden Fehler in der Pfadabhängigkeit erzeugt (IIIF-Manifeste).

#### Lösung (3) 
Neues Projekt aufsetzen Leminbi (Projekt), Arumacula (Sammlung) und die Erkenntnisse aus den Fehleranalysen anwenden.

# Workflow 2.0 und Git Pages

Der zweite durchlauf des Workflows lief (fast) reibungslos, jedenfalls bis zum Hochladen des lokalen ins Git-Repository. Der Test über den lokalen Server lieferte mehrheitlich gute Resultate, dennoch mussten einige Fehler ausgebügelt werden, wie z.B.

```
*   index.md im Repository mit falscher Collection-ID (Repository-Name anstatt Collection-Name, also leminbi statt arumacula)
*   im Directory Pages enthielten die .md-Files für Collection und Reuse für das Label Collection qatar (also die Sammlung aus dem Wax Template-Repository) anstatt arumacula
*   index.html im Directory _site hatte keine URL im Parallax-Element (Hintergrundbild auf Homepage)
```

Nachdem alle Bugs ausgemerzt wurden, folgten Vorbereitung für und Onlinepublishing über Git Pages, mit dem Git-Repository als Serverinstanz:

```
*   GitHub-Repo > Settings > Pages > Source (main), Root (folder) > Save #Repository-URL wird erzeugt
*   _config-File: URL und BaseURL anpassen
```
Mit Git Bash Terminal
```
> cd c/leminbi          _#lokales Repository ansteuern
> git init [URL-Rep]    _#aktiviert Verbindung von lokale zu remote
> git add .             _#bereitet alle abweichenden oder neuen Files lokal für den Upload vor
> git commit            _#öffnet den Editor (VS Code), Überprüfen der neuen/geänderten Files
> git push origin main  _#erweitert/ändert das Git-Repository
```

Zwei nützliche Manuals für Git Bash [hier](https://www.atlassian.com/de/git/tutorials/syncing) und [hier](https://git-scm.com/docs/user-manual).


**Damit läuft die Website über Git Pages.**


Anfänglich gewöhnungsbedürftig ist die Tatsache, dass auf Git Page das für den lokalen Server elementare site-Directory nutzlos ist respektive beim Upload über die Funktion Git Ignor nicht berücksichtigt wird. Die Websitearchitektur für den Siteaufbau über Git Page weicht also von jener für den lokalen Server ab, das heisst die Logik für allfällige Anpassungen ist eine andere.

[https://mariusstricker.github.io/leminbi/](https://mariusstricker.github.io/leminbi/)  



## Weiterentwicklung
*   Sammlung erweitern mit weiteren Pflanzendias, wobei nicht ganz klar ist, ob es dafür sinnvoller ist, einen komplett neuen Workflow mit separatem Repository und namensidentischer Collection durchzugehen, oder einfach das Metadaten-File mit neuen Objektmetadaten zu ergänzen und nur die wax:tasks abzuwickeln; letzteres könnte fehlerbehaftet sein, Begründung: gemäss Wax Wiki ist die strikte Einhaltung der Taskreihenfolge elementar.
*    Objektmetadaten optimieren, z.B. über Informationen aus botanischer Klassifikation und Verlinkung auf entsprechende Onlineressourcen
*    Websitedesign optimieren
*    Hyperlinkpfad definieren und implementieren, damit die Objekte in weiteren Image Viewern/Editoren geöffnet werden können (Mirador, Annonatate)



## Persönliches Fazit
Einfach «nur» eine IIIF-Collection mit Annotationen und Metadaten zu erstellen, erschien mir irgendwie zu banal, also liebäugelte ich mit der Idee, die Sammlung über eine eigene Website zu präsentieren. Einigermassen ahnungslos (oder naiv) darüber, wie umfangreich, komplex und anspruchsvoll ein Website-System sein kann – «das kann doch nicht so schwierig sein, schliesslich nutze ich täglich unzählige von ihnen, kenne HTML-Dokumente und Backend-Applikationen (CMS) – legte ich los. Die Ernüchterung hielt nicht lange hinterm Berg, mein Hochmut wurde sogleich zermalmt. Gäbe es den Wax Workflow nicht und hätte ich eine Website from the scratch konzipieren und umsetzen müssen – es wäre kaum mehr als eine Frustrationskaskade entstanden. Das heisst, meine Lernkurve im Projekt war enorm steil, teilweise sogar überhängend, aber es hat sich mehr als gelohnt: Die Ahnungslosigkeit wechselte ich aus mit einer realistische(re)n Einschätzung, die Logik einer Websitearchitektur habe ich kognitiv vollständig durchdrungen (wenigstens die einer simplen, statischen Website) und mit GitHub entdeckte ich eine aufregende Spielwiese. Last but not least erkannte ich das wirkliche Potenzial von IIIF erst richtig, als ich hinabgestiegen bin in den verwinkelten Betriebsraum der (Bild-)Interoperabilität, und danach auch wieder heraus gefunden habe. Und zwischendurch beneidete ich sogar Systemadministratoren, Softwareentwicklerinnen und IT-Nerds, denn was in meiner Berufsroutine zu kurz kommt, haben sie bestimmt reichlich im Überschuss: kreative Knobelaufgaben. 



Text can be **bold**, _italic_, or ~~strikethrough~~.

[Link to another page](./another-page.html).

There should be whitespace between paragraphs.

There should be whitespace between paragraphs. We recommend including a README, or a file with information about your project.

# Header 1

This is a normal paragraph following a header. GitHub is a code hosting platform for version control and collaboration. It lets you and others work together on projects from anywhere.

## Header 2

> This is a blockquote following a header.
>
> When something is important enough, you do it even if the odds are not in your favor.

### Header 3

```js
// Javascript code with syntax highlighting.
var fun = function lang(l) {
  dateformat.i18n = require('./lang/' + l)
  return true;
}
```

```ruby
# Ruby code with syntax highlighting
GitHubPages::Dependencies.gems.each do |gem, version|
  s.add_dependency(gem, "= #{version}")
end
```

#### Header 4

*   This is an unordered list following a header.
*   This is an unordered list following a header.
*   This is an unordered list following a header.

##### Header 5

1.  This is an ordered list following a header.
2.  This is an ordered list following a header.
3.  This is an ordered list following a header.

###### Header 6

| head1        | head two          | three |
|:-------------|:------------------|:------|
| ok           | good swedish fish | nice  |
| out of stock | good and plenty   | nice  |
| ok           | good `oreos`      | hmm   |
| ok           | good `zoute` drop | yumm  |

### There's a horizontal rule below this.

* * *

### Here is an unordered list:

*   Item foo
*   Item bar
*   Item baz
*   Item zip

### And an ordered list:

1.  Item one
1.  Item two
1.  Item three
1.  Item four

### And a nested list:

- level 1 item
  - level 2 item
  - level 2 item
    - level 3 item
    - level 3 item
- level 1 item
  - level 2 item
  - level 2 item
  - level 2 item
- level 1 item
  - level 2 item
  - level 2 item
- level 1 item

### Small image

![Octocat](https://github.githubassets.com/images/icons/emoji/octocat.png)

### Large image

![Branching](https://guides.github.com/activities/hello-world/branching.png)


### Definition lists can be used with HTML syntax.

<dl>
<dt>Name</dt>
<dd>Godzilla</dd>
<dt>Born</dt>
<dd>1952</dd>
<dt>Birthplace</dt>
<dd>Japan</dd>
<dt>Color</dt>
<dd>Green</dd>
</dl>

```
Long, single-line code blocks should not wrap. They should horizontally scroll if they are too long. This line should be long enough to demonstrate this.
```

```
The final element.
```
