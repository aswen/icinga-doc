Objektvererbung
===============

Einführung
----------

Dieses Dokument versucht Objektvererbung zu erklären und wie sie in
Ihren [Objektdefinitionen](#objectdefinitions) genutzt werden kann.

Wenn Sie nach dem Lesen verwirrt sind, wie Rekursion und Vererbung
arbeiten, sollten Sie einen Blick in die
Beispielobjektkonfigurationsdateien in der NAME-ICINGA-Distribution
werfen. Wenn das immer noch nicht hilft, dann senden Sie eine
(englischsprachige) e-Mail mit einer *detaillierten* Beschreibung Ihres
Problems an die *icinga-users*-Mailing-List.

Grundlagen
----------

Es gibt drei Variablen in allen Objektdefinitionen, die Rekursion und
Vererbung beeinflussen. Sie sind wie folgt *"dargestellt"*:

     define {
             ...
            name          
            use           
            register      [0/1]
            }

Die erste Variable heißt *name*. Das ist lediglich ein "Vorlagen"-Name
(template name), auf den in anderen Objektdefinitonen verwiesen wird, so
dass diese die Objekteigenschaften/Variablen erben. Vorlagennamen müssen
innerhalb der Objekte des gleichen Typs eindeutig sein, so dass Sie
nicht zwei oder mehr Host-Definitionen mit "hosttemplate" als Namen
haben können.

Die zweite Variable heißt *use*. Hier geben Sie den Namen der Vorlage
an, deren Eigenschaften/Variablen Sie erben möchten. Der Name, den Sie
für diese Variable angeben, muss als Vorlage definiert sein (mit Hilfe
der *name*-Variable).

Die dritte Variable heißt *register*. Diese Variable wird benutzt, um
anzuzeigen, ob die Objektdefinition "registriert" werden soll. Per
Default werden alle Objektdefinitionen registriert. Wenn Sie eine
partielle Objektdefinition als Vorlage nutzen, möchten Sie verhindern,
dass sie registriert wird (ein Beispiel dazu folgt). Die Werte sind wie
folgt: 0 = die Objektdefinition NICHT registrieren, 1 = die
Objektdefinition registrieren (das ist der Default). Diese Variable wird
NICHT vererbt, bei jeder als Vorlage genutzten (Teil-) Objektdefinition
muss explizit die *register*-Direktive auf *0* gesetzt werden. Dies
verhindert die Notwendigkeit, eine vererbte *register*-Direktive für
jedes zu registrierende Objekt mit einem Wert von *1* zu übersteuern.

Lokale Variablen gegenüber vererbten Variablen
----------------------------------------------

Bei der Vererbung ist es wichtig zu wissen, dass "lokale"
Objektvariablen immer Vorrang vor Variablen aus der Vorlage haben.
Werfen Sie einen Blick auf das folgende Beispiel mit zwei
Host-Definitionen (nicht alle notwendigen Variablen sind dargestellt):

     define host{
            host_name               bighost1
            check_command           check-host-alive
            notification_options    d,u,r
            max_check_attempts      5
            name                    hosttemplate1
            }
     define host{
            host_name               bighost2
            max_check_attempts      3
            use                     hosttemplate1
            }

Sie werden bemerken, dass die Definiton für den Host *bighost1* mit
Hilfe der Vorlage *hosttemplate1* definiert wurde. Die Definition für
Host *bighost2* nutzt die Definition von *bighost1* als Vorlagenobjekt.
Sobald NAME-ICINGA diese Daten verarbeitet hat, wäre die resultierende
Definition von *bighost2* äquivalent zu dieser Definition:

     define host{
            host_name               bighost2
            check_command           check-host-alive
            notification_options    d,u,r
            max_check_attempts      3
            }

Sie sehen, dass die *check\_command*- und
*notification\_options*-Variablen vom Vorlagenobjekt geerbt wurden (wo
Host *bighost1* definiert wird). Trotzdem wurden die *host\_name*- und
*check\_attempts*-Variablen nicht vom Vorlagenobjekt geerbt, weil sie
lokal definiert wurden. Erinnern Sie sich, dass von einem Vorlagenobjekt
geerbte Variablen von lokal definierten Variablen überschrieben werden.
Das sollte ein ziemlich einfach zu verstehendes Konzept sein.

![](../images/tip.gif) Hinweis: wenn Sie möchten, dass lokale
Zeichenketten-Variablen an geerbte Zeichenkettenwerte angehängt werden,
können Sie das tun. Lesen Sie [weiter
unten](#objectinheritance-add_string) mehr darüber, wie das erreicht
werden kann.

Vererbungsverkettung
--------------------

Objekte können Eigenschaften/Variablen aus mehreren Ebenen von
Vorlagenobjekten erben. Nehmen Sie das folgende Beispiel:

     define host{
            host_name               bighost1
            check_command           check-host-alive
            notification_options    d,u,r
            max_check_attempts      5
            name                    hosttemplate1
            }
     define host{
            host_name               bighost2
            max_check_attempts      3
            use                     hosttemplate1
            name                    hosttemplate2
            }
     define host{
            host_name               bighost3
            use                     hosttemplate2
            }

Sie werden bemerken, dass die Definition von Host *bighost3* Variablen
von der Definition von *bighost2* erbt, die wiederum Variablen von der
Definition von Host *bighost1* erbt. Sobald NAME-ICINGA diese
Konfigurationsdaten verarbeitet, sind die resultierenden Host-Definition
äquivalent zu den folgenden:

     define host{
            host_name               bighost1
            check_command           check-host-alive
            notification_options    d,u,r
            max_check_attempts      5
            }
     define host{
            host_name               bighost2
            check_command           check-host-alive
            notification_options    d,u,r
            max_check_attempts      3
            }
     define host{
            host_name               bighost3
            check_command           check-host-alive
            notification_options    d,u,r
            max_check_attempts      3
            }

Es gibt keine eingebaute Beschränkung, wie "tief" Vererbung gehen kann,
aber Sie sollten sich vielleicht selbst auf ein paar Ebenen beschränken,
um die Übersicht zu behalten.

Unvollständige Objektdefinitionen als Vorlagen nutzen
-----------------------------------------------------

Es ist möglich, unvollständige Objektdefinitionen als Vorlage für andere
Objektdefinitionen zu nutzen. Mit "unvollständiger" Definition meinen
wir, dass nicht alle benötigten Variablen in der Objektdefinition
angegeben wurden. Es mag komisch klingen, unvollständige Definitionen
als Vorlagen zu nutzen, aber es ist tatsächlich empfohlen, dies zu tun.
Warum? Nun, sie können als ein Satz von Defaults für alle anderen
Objektdefinitionen dienen. Nehmen Sie das folgende Beispiel:

     define host{
            check_command           check-host-alive
            notification_options    d,u,r
            max_check_attempts      5
            name                    generichosttemplate
            register                0
            }
     define host{
            host_name               bighost1
            address                 192.168.1.3
            use                     generichosttemplate
            }
     define host{
            host_name               bighost2
            address                 192.168.1.4
            use                     generichosttemplate
            }

Beachten Sie, dass die erste Host-Definition unvollständig ist, weil die
erforderliche *host\_name*-Variable fehlt. Wir müssen keinen Host-Namen
angeben, weil wir diese Definition als Vorlage nutzen wollen. Um
NAME-ICINGA daran zu hindern, diese Definition als einen normalen Host
anzusehen, setzen wir die *register*-Variable auf 0.

Die Definitionen von *bighost1* und *bighost2* erben ihre Werte von der
generischen Host-Definition. Die einzige Variable, die überschrieben
wird, ist die *address*-Variable. Das bedeutet, dass beide Hosts exakt
die gleichen Eigenschaften haben, bis auf die *host\_name*- und
*address*-Variablen. Sobald NAME-ICINGA die Konfigurationsdaten im
Beispiel verarbeitet, wären die resultierenden Host-Definitionen
äquivalent zu folgenden:

     define host{
            host_name               bighost1
            address                 192.168.1.3
            check_command           check-host-alive
            notification_options    d,u,r
            max_check_attempts      5
            }
     define host{
            host_name               bighost2
            address                 192.168.1.4
            check_command           check-host-alive
            notification_options    d,u,r
            max_check_attempts      5
            }

Die Nutzung einer Vorlagendefinition für Default-Werte erspart Ihnen
mindestens eine Menge Tipparbeit. Es spart Ihnen auch eine Menge
Kopfschmerzen, wenn Sie später die Default-Werte von Variablen für eine
große Zahl von Hosts wollen.

eigene Objektvariablen (custom object variables)
------------------------------------------------

Jede [eigene Objektvariable](#customobjectvars), die Sie in Ihren Host-,
Service- oder Kontaktdefinitionen definieren, wird wie jede andere
Standardvariable vererbt. Nehmen Sie das folgende Beispiel:

     define host{
            _customvar1             somevalue  ; <-- Custom host variable
            _snmp_community         public  ; <-- Custom host variable
            name                    generichosttemplate
            register                0
            }
     define host{
            host_name               bighost1
            address                 192.168.1.3
            use                     generichosttemplate
            }

Der Host *bighost1* wird die eigenen Host-Variablen *\_customvar1* und
*\_snmp\_community* von der *generichosttemplate*-Definition erben,
zusammen mit den entsprechenden Werten. Die daraus resultierende
Definition für *bighost1* sieht wie folgt aus:

     define host{
            host_name               bighost1
            address                 192.168.1.3
            _customvar1             somevalue
            _snmp_community         public
            }

Vererbung für Zeichenketten-Werte aufheben
------------------------------------------

In einigen Fällen möchten Sie vielleicht nicht, dass Ihre Host-,
Service- oder Kontakt-Definitionen Werte von Zeichenketten-Variablen aus
Vorlagen erben. Wenn das der Fall ist, können Sie "**null**" (ohne
Anführungszeichen) als den Wert der Variable, die Sie nicht erben
möchten. Nehmen Sie das folgende Beispiel:

     define host{
            event_handler           my-event-handler-command
            name                    generichosttemplate
            register                0
            }
     define host{
            host_name               bighost1
            address                 192.168.1.3
            event_handler           null
            use                     generichosttemplate
            }

In diesem Fall wird der Host *bighost1* nicht den Wert der
*event\_handler*-Variable erben, die in der
*generichosttemplate*-Vorlage definiert ist. Die resultierende
Definition von *bighost1* sieht wie folgt aus:

     define host{
            host_name               bighost1
            address                 192.168.1.3
            }

additive Vererbung von Zeichenketten-Werten
-------------------------------------------

NAME-ICINGA gibt lokalen Variablen Vorrang vor Werten, die von Vorlagen
vererbt werden. In den meisten Fällen überschreiben lokale
Variablenwerte jene, die in Vorlagen definiert sind. In einigen Fällen
ist es sinnvoll, dass NAME-ICINGA die Werte von geerbten *und* lokalen
Variablen gemeinsam nutzt.

Diese "additive Vererbung" kann durch Voranstellen eines Pluszeichens
(**+**) vor den lokalen Variablenwert erreicht werden. Dieses Feature
ist nur für Standard-Variablen verfügbar, die Zeichenketten-Werte
enthalten. Nehmen Sie das folgende Beispiel:

     define host{
            hostgroups              all-servers
            name                    generichosttemplate
            register                0
            }
     define host{
            host_name               linuxserver1
            hostgroups              +linux-servers,web-servers
            use                     generichosttemplate
            }

In diesem Fall wird der *linuxserver1* den Wert der lokalen
*hostgroups*-Variablen dem der *generichosttemplate*-Vorlage hinzufügen.
Die resultierende Definition von *linuxserver1* sieht wie folgt aus:

     define host{
            host_name               linuxserver1
            hostgroups              all-servers,linux-servers,web-servers
            }

Implizite Vererbung
-------------------

Normalerweise müssen Sie entweder explizit den Wert einer erforderlichen
Variable in einer Objektdefinition angeben oder sie von einer Vorlage
erben. Es gibt ein paar Ausnahmen zu dieser Regel, in denen NAME-ICINGA
annimmt, dass Sie einen Wert benutzen wollen, der statt dessen von einem
verbundenen Objekt kommt. Die Werte einiger Service-Variablen werden zum
Beispiel vom Host kopiert, mit dem der Service verbunden ist, wenn Sie
diese nicht anderweitig angeben.

Die folgende Tabelle führt die Objektvariablen auf, die implizit von
verbundenen Objekten vererbt werden, wenn Sie deren Werte nicht explizit
angeben oder sie von einer Vorlage erben.

  -------------- ---------------------------- ----------------------------
  **Objekttyp**  **Objektvariable**           **implizite Quelle**

  **Services**   *contact\_groups*            *contact\_groups* in der
                                              verbundenen Host-Definition

  *contacts*     *contacts* in der
                 verbundenen Host-Definition

  *notification\ *notification\_interval* in
  _interval*     der verbundenen
                 Host-Definition

  *notification\ *notification\_period* in
  _period*       der verbundenen
                 Host-Definition

  **Host         *contact\_groups*            *contact\_groups* in der
  Escalations**                               verbundenen Host-Definition

  *contacts*     *contacts* in der
                 verbundenen Host-Definition

  *notification\ *notification\_interval* in
  _interval*     der verbundenen
                 Host-Definition

  *escalation\_p *notification\_period* in
  eriod*         der verbundenen
                 Host-Definition

  **Service      *contact\_groups*            *contact\_groups* in der
  Escalations**                               verbundenen
                                              Service-Definition

  *contacts*     *contacts* in der
                 verbundenen
                 Service-Definition

  *notification\ *notification\_interval* in
  _interval*     der verbundenen
                 Service-Definition

  *escalation\_p *notification\_period* in
  eriod*         der verbundenen
                 Service-Definition
  -------------- ---------------------------- ----------------------------

> **Note**
>
> Diese Werte werden nur im Falle des Zustandswechsels eines Objekts
> vererbt, so dass sich "in der verbundenen ... Definition" nur auf die
> eine Host/Service-Kombination bezieht, die fehlschlägt/sich erholt,
> obwohl es möglich ist, einen Service für ein oder mehrere Hostgruppen
> auszuführen.

implizite/additive Vererbung bei Eskalationen
---------------------------------------------

Service- und Host-Eskalationsdefinitionen können eine spezielle Regel
benutzen, die die Möglichkeiten von impliziter und additiver Vererbung
kombiniert. Wenn Eskalationen 1) nicht die Werte ihrer
*contact\_groups*- oder *contacts*-Direktiven von anderen
Eskalationsvorlagen erben und 2) ihre *contact\_groups*- oder
*contacts*-Direktiven mit einen Plus-Zeichen (+) beginnen, dann werden
die Werte der *contact\_groups* oder *contacts*-Direktiven der
entsprechenden Host- oder Service-Definitionen in der additiven
Vererbungslogik benutzt.

Verwirrt? Hier ein Beispiel:

     define host{
            name            linux-server
            contact_groups  linux-admins
            ...
            }
     define hostescalation{
            host_name               linux-server
            contact_groups  +management
            ...
            }

Das ist ein viel einfacheres Äquivalent zu:

     define hostescalation{
            host_name               linux-server
            contact_groups  linux-admins,management
            ...
        }

Wichtige Werte (important values)
---------------------------------

Service-Vorlagen können eine spezielle Regel benutzen, die ihrem
check\_command-Wert Vorrang gibt. Wenn das check\_command mit einem
Ausrufungszeichen (!) beginnt, dann wird das check\_command der Vorlage
als wichtig markiert und wird statt des im Service definierten
check\_command (dies ist der CSS-Syntax nachempfunden, die ! als
wichtiges Attribut benutzt).

Warum ist das nützlich? Es ist hauptsächlich dann sinnvoll, wenn ein
unterschiedliches check\_command für verteilte Systeme gesetzt wird. Sie
wollen vielleicht einen Frische-Schwellwert und ein check\_command
setzen, der den Service in einen fehlerhaften Status versetzt, aber das
funktioniert nicht mit dem normalen Vorlagensystem. Dieses
"wichtig"-Kennzeichen erlaubt es, das angepasste check\_command zu
schreiben, aber eine allgemeine verteilte Vorlage zu benutzen, die das
check\_command überlagert, wenn es auf dem zentralen NAME-ICINGA-Server
eingesetzt wird.

Zum Beispiel:

    # On master
    define service {
            name                   service-distributed
            register               0
            active_checks_enabled  0
            check_freshness        1
            check_command          !set_to_stale
            }
    # On slave
    define service {
            name                   service-distributed
            register               0
            active_checks_enabled  1
            }
    # Service definition, used by master and slave
    define service {
            host_name              host1
            service_description    serviceA
            check_command          check_http...
            use                    service-distributed
            ...
            }

> **Note**
>
> Bitte beachten Sie, dass nur eine Vererbungsebene bei diesen wichtigen
> Werten möglich ist. Das bedeutet, dass Sie nicht das check\_command
> von einer Vorlage zu einer weiteren und von dort zum Service vererben
> können.
>
>      Template1 => Service1                <== funktioniert
>      Template1 => Template2 => Service1   <== funktioniert NICHT

Mehrere Vererbungsquellen
-------------------------

Bisher haben alle Beispiele Objektdefinitionen gezeigt, die
Variablen/Werte von einer einzelnen Quelle erben. Sie können für
komplexere Konfigurationen auch Variablen/Werte von mehreren Quellen
erben, wie unten gezeigt.

  ------------------------------------ ------------------------------------
       # Generic host template         ![](../images/multiple-templates1.pn
       define host{                    g)
              name                     
  generic-host                         
              active_checks_enabled    
  1                                    
              check_interval           
  10                                   
              ...                      
              register                 
  0                                    
              }                        
       # Development web server templa 
  te                                   
       define host{                    
              name                     
  development-server                   
              check_interval           
  15                                   
              notification_options     
  d,u,r                                
              ...                      
              register                 
  0                                    
              }                        
       # Development web server        
       define host{                    
              use                      
  generic-host,development-server      
              host_name                
  devweb1                              
              ...                      
              }                        
  ------------------------------------ ------------------------------------

Im obigen Beispiel erbt *devweb1* Variablen/Werte von zwei Quellen:
*generic-host* und *development-server*. Sie werden bemerken, dass in
beiden Quellen eine *check\_interval*-Variable definiert ist. Weil
*generic-host* die erste in *devweb1* durch die *use*-Direktive
angegebene Vorlage ist, wird der Wert für die *check\_interval*-Variable
durch den *devweb1*-Host vererbt. Nach der Vererbung sieht die
Definition von *devweb1* wie folgt aus:

     # Development web server
     define host{
            host_name               devweb1
            active_checks_enabled   1
            check_interval          10
            notification_options    d,u,r
            ...
            }

Vorrang bei mehreren Vererbungsquellen
--------------------------------------

Wenn Sie mehrere Vererbungsquellen nutzen, ist es wichtig zu wissen, wie
NAME-ICINGA Variablen behandelt, die in mehreren Quellen definiert sind.
In diesen Fällen wird NAME-ICINGA die Variable/den Wert aus der ersten
Quelle benutzen, die in der *use*-Direktive angegeben ist. Weil
Vererbungsquellen ebenfalls Variablen/Werte aus ein oder mehreren
Quellen erben können, kann es kompliziert werden herauszufinden, welche
Variablen/Werte-Paare Vorrang haben.

  ------------------------------------ ------------------------------------
  Betrachten Sie die folgende          ![](../images/multiple-templates2.pn
  Host-Definition, die drei Vorlagen   g)
  referenziert:                        
                                       
       # Development web server        
       define host{                    
              use  1,  4,  8           
              host_name devweb1 ...    
       }                               
                                       
  Wenn einige dieser referenzierten    
  Vorlagen selbst Variablen/Werte von  
  ein oder mehreren Vorlagen erben,    
  werden die Vorrangregeln auf der     
  rechten Seite gezeigt.               
                                       
  Test, Versuch und Irrtum werden      
  Ihnen helfen, besser zu verstehen,   
  wie die Dinge in komplexen           
  Vererbungssituationen wie dieser     
  funktionieren. :-)                   
  ------------------------------------ ------------------------------------

Objektvererbung
Objektvererbung
Vererbung für Zeichenketten-Werte aufheben
Objektvererbung
additive Vererbung von Zeichenketten-Werten
Objektvererbung
Implizite Vererbung
Objektvererbung
Implizite/additive Vererbung in Eskalationen
Objektvererbung
mehrere Vererbungsquellen