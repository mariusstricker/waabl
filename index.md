---
layout: default
---

In dieser Projektdokumentation werden die verschiedenen Arbeitsschritte, von der Idee einer virtuellen Ausstellung mit IIIF-Images bis zur veröffentlichten Website, beschrieben. Die Umsetzung des Projekts basiert auf dem Workflow [Wax](https://minicomp.github.io/wiki/wax/) von [Minicomp](https://github.com/minicomp/). Sämtliche Files und Objekte befinden sich im Repositorium [Leminbi](https://github.com/mariusstricker/leminbi) auf GitHub, diese Projektwebsite enthält lediglich dokumentarische Information.


## Projektziel
Das Ziel meiner Projektarbeit bestand ursprünglich darin, mit [Annonatate (Herokuapp)](https://annonatate.herokuapp.com/) eine Sammlung mit annotationsfähigen, metadatenangereicherten IIIF-Images zu erstellen und diese über eine Projektwebsite online zugänglich zu machen; ursprünglich deshalb, weil sich im Verlauf des Projekts herausstellte, dass der Weg über Annonatate hätte übersprungen werden können. Aber dazu später mehr.  

Das Projektziel als Fragestellung: Wie lässt sich eine Sammlung IIIF-konformer Bilder vollständig, metadatenangereichert und effizient über eine Website «ausstellen»?


## Eckdaten

Die Entwickler beschreiben ihren Wax Workflow folgendermassen: «Wax is a minimal computing (minicomp) project […] for producing digital exhibitions focused on longevity, low costs, and flexibility.» ([Quelle](https://minicomp.github.io/wax/)). Für die notwendigen Installationen und Setups für den Workflow steht ein Wiki zur Verfügung. Die unterschiedlichen Stadien des Workflows werden der verlinkten Grafik anschaulich visualisiert:

[Workflow](https://minicomp.github.io/wiki/assets/wax_workflow.jpeg)

Der Workflow enthält Ruby Gems für Bildprozessierung und statisches Webpublishing (Jekyll, Markdown und YAML-Config), zudem werden HTML, CSS und JavaScript angewendet, Git Bash für Befehle zum Datenaustausch zwischen lokalem und Git-Repository. 

Alle Installationsen und Prozesse auf dem lokalen Rechner und Repository wurden in Visual Studio Code (VSC) mit der Linux-Distribution Ubuntu durchgeführt. 

Diese Projektdokumentation folgt, wenn nicht speziell darauf hingewiesen wird, den Prozessschritten im [Wax Wiki](https://minicomp.github.io/wiki/wax/).

Projekt- und Sammlungsbezeichnung: Glasdiashow.


## Systemsetup

Bei ersten Requirementchecks und Installationsversuchen hat sich relativ rasch herausgestellt, dass es keine gute Idee war, wie ursprünglich geplant, den Workflow auf Desktopebene des Rechners aufzusetzen, denn der Desktop läuft auf dem Filehosting-Dienst OneDrive (Arbeitgeber).

```
(base) marius@WN128567:/mnt/c/'Users/marius.stricker/OneDrive - PHZH/Desktop'
```
Die beiden Commands:
```
> sudo apt install bundler        # ruby gem
```
```
> sudo apt install libvips-tools  # fast image processing library
```
erzeugten wiederholt unlösbare Probleme. Also setzte ich das Projekt auf dem Laufwerk C: neu auf und reinitialisierte den Installationsprozess. Vermutlich hätte OneDrive später womöglich nicht einmal Probleme bereitet. Für die Installation war jedenfalls gar nicht OneDrive die Fehlerquelle, sondern die Tatsache, dass ich die Commands nicht auf Systemebene (globally) ausgeführt hatte, sondern direkt im Directory. 


Nach Directory-Change und den erneuten Installationen wurden bei der Überprüfung die korrekten Versionen der installierten Elemente und Packete ausgegeben:

```
> ruby -v             # ruby 2.7.0p0 
> bundler -v          # Bundler version 2.1.4
> git –version        # git version 2.25.1
> convert -version    # GraphicsMagick 1.3.35
> gs -version         # hostscript 9.50 (2019-10-15)
> vips -version       # vips-8.9.1 (Laufwerk C :)

```


## Sammlungs- und Metadaten aufbereiten

Beim Aufbereiten des Metadatenfiles traten mehrere, teils hartnäckige Probleme auf. Gemäss Wiki kann Wax mit Metadatenfiles in .csv, .json und .yml umgehen. Von Microsoft Excel wird dringend abgeraten, empfohlen seien Google Sheets. Aus Routine öffnete ich das heruntergeladene Google-Dokument nicht mit einem Editor, sondern mit Excel. Denn ich ging davon aus, dass wegen der Sprach- und Zeichenencodierung von Excel abgeraten wird und meinte dies überlisten zu können, indem ich ein leeres Excel-File als CSV-UTF-8 abspeichere und die Metadaten hineinkopiere. 

#### Fehleranalyse (1)
Die späteren Wax:Tasks können aber tatsächlich keinerlei Daten aus einem Excel-File verarbeiten, das Terminal gibt als Fehlermeldung zurück, dass die PID der ersten Zeile fehlt (was erwiesenermassen falsch ist). Der Grund eigentliche Grund für die Schwierigkeiten mit Excel ist nicht die Sprachcodierung, sondern unsichtbare Formatierungen und Zeichen, die während der Wax:Tasks nicht interpretiert werden können. Zwar versuchte ich diese über OpenRefine zu tilgen, aber offenbar erfolglos.

#### Lösung (1)
Metadateninhalte in den Microsoft Editor kopieren und als .csv-Datei abspeichern; der Editor in VS Code eignete sich dafür nicht, weil er kein CSV-Format generieren kann.

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


## Setup Website

Als erster Schritt wird das Git-Repository [Wax](https://github.com/minicomp/wax) in den eigenen Git-Workspace kopiert (mit _Use this template_) und anschliessend auf den lokalen Rechner geklont, bereinigt und mit den Raw-Images sowie dem aufbereiteten Metadaten-File ergänzt. 

VS Code: Terminal
```
> git clone                           # anschliessend ins Repository wechseln
> bundle exec rake wax:clobber qatar  # löscht alle Files aus den Directories qatar und img/derivatives (oder manuell)
```
```
> alle Qatar-Files aus _data löschen
> Raw-Images und Metadaten im Repository hinterlegen
> Anpassen des _config.yml-File (Dokumentation im Wiki): vorallem collection name und Pfad zu Raw-Images/Metadaten
```



### Wax:Tasks
VS Code: Terminal

```
> bundle exec rake –tasks                             # Übersicht aller wax:tasks_
> bundle exec rake wax:derivatives:iiif Glasdiashow   # generiert IIIF-Derivatives: fügt im Metadaten-File die Felder full, thumbnail, manifest hinzu und schreibt für jedes Objekt mit PID entsprechende Inhalte aus den JSON-Dateien
```

> Rechenzeit für 20 Images à je 14 Megabyte: 2.5h.

```
> bundle exec rake wax:pages Glasdiashow              # erstellt im lokalen Repository ein Directory mit _Glasdiashow, schreibt pro Objekt ein Markdownfile mit Daten aus dem Metadatenfile
> bundle exec rake wax:search                         # schreibt einen JSON-Index anhand der Markdownfiles aller Objekte
```



### Website testen

Die Projektwebsite ist nun (fast) bereit, um online zu gehen, darum werden ausführliche Tests durchgeführt während die Seite über den lokalen Server (localhost:4000) gehostet wird.

```
> bundle exec jekyll serve    # startet das Hosting erstellt beim ersten Mal das Directory site im lokalen Repository 
```
Das site-Directory enthält:
*   Index-File (HTML) für die Homepage und die 404-Seite
*   pro Seite/Unterseite je ein HTML-File (insgesamt 28) 
*   Directory img mit den IIIF-Files (JSON), dupliziert aus Repository           
*   Directory assets mit den Booting- und Style-Files (.js, .min, .css, .min.js), dupliziert aus Repository   


> Der Pfad zum Booten der Website:
> _config.yml > _site > assets > Index > …



> **Aber: die Website konnte nicht geladen werden.**



#### Fehleranalyse (2)
Der Workflow sieht leider keine vollständige File-Kontrolle vor den Wax:Tasks vor, was sehr nützlich wäre, und so stellte ich nach ehlend langer Fehlersuche im Anschluss an die Webseitentests fest, dass das Metadatenfile noch immer nicht der notwenigen Qualität entsprach, als ich die Wax-Tasks startete, worauf das File selbst nicht korrekt komplettiert wurde (Hauptfehler) und die dadurch erzeugten Fehler auch in den JSON- und Markdown-Files repräsentiert waren. 

Mir scheint rätselhaft, wie der Workflow einwandfrei funktionieren soll, wenn wie die Metadaten, wie auf dem [Wax-Repository (Git)](https://github.com/minicomp/wax-facets/edit/main/_data/qatar.csv) ersichtlich, wirklich mit Kommas getrennt werden (csv). Denn bei der Verarbeitung von Raw-Images und Metadaten zu IIIF-Manifesten mit API und URI im Task _wax:derivatives:iiif_, schreibt der Code Kommaelemente direkt in die Thumbnail-URI:

```
/img/derivatives/iiif/images/obj1/full/250,/0/default.jpg
```

Logischerweise werden genau diese Thumbnail-Kommas vom Code als Trennungszeichen interpretiert, was zu einem fehlerhaften Metadatenfile und somit einem fehlerhaften Kernelement für den gesamten Workflows führte – eine doch eher ungünstige Ausgangslage, die jedoch in vollem Umfang erst einiges später auffalen wird. 

Zunächst: Der erste Lösungsversuch mit Semikolons anstatt Kommas als Trennungszeichen im Metadatenfile kann auf GitHub nicht korrekt ausgelesen werden. So sieht die [Tabelle korrekt](https://github.com/mariusstricker/leminbi/blob/main/_data/arumacula.tsv) aus, so [falsch mit Semikolons](https://github.com/mariusstricker/arumacula/blob/main/_data/Glasdias_original.csv).  

#### Lösung (2)
Weil .tsv für den Erstellungsprozess der IIIF-Manifeste (wax:derivatives:iiif) nicht unterstützt wird, sieht die tatsächliche und doch ungewohnt umständliche Lösung folgendermassen aus:
1. Metadatenfile (.csv) mit Semikolon für die Tasks aufbereiten
1. dadurch wird es wärend den Wax-Tasks korrekt verarbeitet, d.h. mit den Manifest-URIs ergänzt, die Markdown-Files der Manifeste und alle weiteren davon abhängigen Dokumente korrekt geschrieben
1. für das spätere Webhosting mit Git Page müssen die Semikolons durch Kommas, idealer durch Tabs ersetzt werden; dies sollte zwingend in einem Editor erfolgen, weil eingefügte Tabs im Git-File von Git selbst nicht als solche interpretiert werden (!).


#### Fehleranalyse (3)
Das Git-Repository Glasdiashow hatte ich nicht direkt auf das Laufwerk C: geklont, sondern ins Directory C:/Glasdiashow und neutral als Repository bezeichnet, was später bei den Tests über den lokalen Server (localhost:4000) 404-Fehler erzeugte.

Zudem erwies es sich als fatal, auf GitHub das Projekt und die Sammlung gleichlautend zu bezeichnen (beide: Glasdiashow), denn wie im Fall oben wurden dadurch Fehler in der Pfadabhängigkeit erzeugt (IIIF-Manifeste).


#### Lösung (3) 
Nach all diesen Malheurs schien nur eine radikale Lösung als geeignet: Neues Projekt aufsetzen und die Erkenntnisse aus den Fehleranalysen anwenden. Neue Terminologie: _Leminbi_ (Repository), _arumacula_ (Sammlung)


# Workflow 2.0 und Git Pages

Der zweite durchlauf des Workflows lief (dann fast) reibungslos, jedenfalls bis zum Hochladen des lokalen Leminbi-Repository auf GitHub. Zunächst aber lieferten die Tests auf dem lokalen Server mehrheitlich gute Resultate, einige Fehler – diesmal jedoch alles andere als selbstverursachte – mussten trotzdem analysiert und behoben werden. Kleine Auswahl:

```
> index.md im Repository mit falscher Collection-ID (Repository-Name anstatt Collection-Name, also leminbi statt arumacula)
> im Directory Pages enthielten die .md-Files für Collection und Reuse für das Label Collection qatar (also die Sammlung aus dem Wax Template-Repository) anstatt arumacula
> index.html im Directory _site hatte keine URL im Parallax-Element (Hintergrundbild auf Homepage)
```

Alle Bugs konnten erfolgreich ausgemerzt werden, danach konnte das Hosting auf GitHub vorbereitet werden:
```
> GitHub-Repo > Settings > Pages > Source (main), Root (folder) > Save    # Repository-URL wird erzeugt
> _config-File: URL und BaseURL (bis anhin: www.localhost:4000) mit der Repository-URL und -BaseURL ersetzen
```
Mit Git Bash Terminal
```
> cd c/leminbi          # lokales Repository ansteuern
> git init [URL-Rep]    # aktiviert Verbindung von lokalem zu remote Repository
> git add .             # bereitet alle abweichenden oder neuen Files lokal für den Upload vor
> git commit            # öffnet automatisch den Editor (VS Code) für die Kontrolle der neuen/geänderten Files
> git push origin main  # erweitert/ändert das Git-Repository
```

Zwei nützliche Manuals für Git Bash [hier](https://www.atlassian.com/de/git/tutorials/syncing) und [hier](https://git-scm.com/docs/user-manual).


> **Finally – die Website bootet über Git Pages.**


Gewöhnungsbedürftig ist die Tatsache, dass für das einwandfreie Git-Webhosting das für den Betrieb über den lokalen Server absolut zentrale site-Directory offfenbar nicht erforderlich ist, denn trotz _git push_ wurde es nicht ins Git-Repository hochgeladen. Dies, wie sich später klären wird, wegen der Funktion _git ignor_, mit der lokal relevante Directories oder Files beim Upload in ein inaktives Directory verschoben werden. Die Architektur der Website über Git Pages weicht insofern von jener auf dem lokalen Server gehosteten ab und das hiess, sich (kurz) eine neue Logik erschliessen. Nach einigen Layout- und Inhaltsanpassungen ist es aber endlich soweit und die Website geht «offiziell» live:

[https://mariusstricker.github.io/leminbi/](https://mariusstricker.github.io/leminbi/)  



## Weiterentwicklung
*   Sammlung erweitern mit weiteren Pflanzendias (20 liegen noch bereit), wobei aber noch nicht eindeutig geklärt ist, ob es dafür sinnvoller wäre, einen komplett neuen Workflow mit separatem Repository, aber namensgleicher Collection (_arumacula_) durchzugehen, oder ob es funktionieren würde, das Metadatenfile mit den neuen Objektmetadaten zu ergänzen und lediglich die Wax:Tasks durchrechnen zu lassen; Letzteres könnte fehlerbehaftet sein, Begründung: gemäss Wiki ist die strikte Einhaltung der Taskreihenfolge elementar.
*    Objektmetadaten optimieren, z.B. über differenzierte Informationen aus der botanischen Klassifikation, ggf. kombiniert mit einer Karte der geografischen Ausbreitung oder Links auf andere Onlineressourcen.
*    Websitedesign optimieren.
*    Hyperlinks erzeugen und -pfad definieren sowie hinterlegen, damit einzelne oder gruppierte Objekte in weiteren Image Viewern/Editoren geöffnet werden können (Mirador, Annonatate).



## Persönliches Fazit
Einfach «nur» eine IIIF-Collection mit Annotationen und angereicherten Metadaten zu erstellen, erschien mir für das Projekt irgendwie zu einfach, also entschied ich mich kurzerhand dafür, die geplante Sammlung über eine eigene Website zu präsentieren. Einigermassen ahnungslos (oder naiv) darüber, wie umfangreich, komplex und anspruchsvoll ein Website-System ist – «das kann doch nicht so schwierig sein, schliesslich nutze ich täglich unzählige von ihnen, kenne HTML-Dokumente und Backend-Applikationen (CMS) – legte ich los. Die Ernüchterung hielt nicht lange hinterm Berg, mein Hochmut wurde sogleich zermalmt. Gäbe es den Wax-Workflow nicht und hätte ich eine Website _from the scratch_ konzipieren/umsetzen müssen – es wäre kaum mehr als eine Frustrationskaskade daraus entstanden. So aber war meine Lernkurve im Projekt enorm steil und aber es hat sich mehr als gelohnt: Die anfängliche Ahnungslosigkeit wechselte ich aus mit einer realistische(re)n Einschätzung über das Wesen von Websites, die Logik der Websitearchitektur habe ich kognitiv durchdrungen (wenigstens die einer simplen, statischen Website) und mit GitHub entdeckte ich unerwartet eine interessante Spielwiese. _Last but not least_ erkannte ich das wirkliche Potenzial der IIIF-Innovation erst richtig, als ich einmal hinabgestiegen bin in den verwinkelten Betriebsraum der (Bild-)Interoperabilität, und dann auch den Weg hinaus wieder gefunden habe. Zwischendurch beneidete ich sogar Systemadministratoren, Softwareentwicklerinnen und IT-Nerds, denn was in meiner Berufsroutine zu kurz kommt, haben sie bestimmt reichlich im Überschuss: Rätsel zu lösen.


## Anektote zum Schluss

Hätte ich das Wiki vor Beginn gründlich und vollständig durchgelesen, wäre mir ein rechen- und zeitintensiver Umweg von ungefähr vier Stunden erspart geblieben: In der Annahme, mit Wax «nur» eine Online-Repräsentation meiner Sammlung zu realisieren, liess ich vorschnell für jedes der 20 Glasdias mit Annonatate ein IIIF-Manifest und Image Tiles erstellen (zugänglich in meinem [Annonatate-Repository](https://github.com/mariusstricker/annonatate)) – «werch ein Illtum». 

Übrigens: Bei grossen Image-Files muss der Prozess mit Annonatate direkt aus dem Git-Repository gesteuert werden. Meine Empfehlung ist, jedes File einzeln in Annonatate uploaden und in Git den Workflow manuell aktivieren (Actions), denn bei zu grosser Datenmenge würgt Git den Prozess ab.



_aber_

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
