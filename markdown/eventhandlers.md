Eventhandler
============

Einführung
----------

![](../images/eventhandlers.png)

Eventhandler sind optionale Systemkommandos (Scripts oder Programme),
die gestartet werden, wenn ein Host- oder Service-Zustandswechsel
stattfindet. Sie werden auf dem System ausgeführt, auf dem die Prüfung
eingeplant (initiiert) wurde.

Ein einleuchtender Einsatz von Eventhandlern ist die Möglichkeit von
NAME-ICINGA, proaktiv Probleme zu beheben, bevor jemand benachrichtigt
wird. Einige andere Anwendungsmöglichkeiten für Eventhandler umfassen:

-   neustarten eines ausgefallenen Service

-   anlegen eines Trouble-Tickets in einem Helpdesk-Systems

-   eintragen von Ereignisinformationen in eine Datenbank

-   Strom aus- und einschalten bei einem Host\*

-   etc.

\* Strom durch ein automatisiertes Script bei einem Host aus- und
einzuschalten, der Probleme hat, sollte wohlüberlegt sein. Betrachten
Sie sorgfältig die möglichen Konsequenzen, bevor Sie automatische
Reboots implementieren. :-)

Wann werden Eventhandler ausgeführt?
------------------------------------

Eventhandler werden ausgeführt, wenn ein Service oder Host

-   in einem SOFT-Problemzustand ist

-   in einen HARD-Problemzustand wechselt

-   aus einem SOFT- oder HARD-Problemzustand zurückkehrt

SOFT- und HARD-Zustände sind ausführlich [hier](#statetypes)
beschrieben.

Eventhandler-Typen
------------------

Es gibt unterschiedliche Typen von optionalen Eventhandlern, die Sie
definieren können, um Host- und Statuswechsel zu behandeln:

-   Globale Host-Eventhandler

-   Globale Service-Eventhandler

-   Host-spezifische Eventhandler

-   Service-spezifische Eventhandler

Globale Host- und Service-Eventhandler werden für *jeden* auftretenden
Host- oder Service-Zustandswechsel durchgeführt, direkt vor einem
möglichen Host- oder Service-spezifischen Eventhandler. Sie können
globale Host- oder Service-spezifische Eventhandler durch die
[global\_host\_event\_handler](#configmain-global_host_event_handler)
und
[global\_service\_event\_handler](#configmain-global_service_event_handler)-Optionen
in der Hauptkonfigurationsdatei angeben.

Einzelne Hosts und Service können ihre eigenen Eventhandler haben, die
ausgeführt werden, um Statuswechsel zu behandeln. Sie können einen
auszuführenden Eventhandler durch die *event\_handler*-Direktive in
Ihren [Host](#objectdefinitions-host)- oder
[Service](#objectdefinitions-service)-Definitionen angeben. Diese Host-
und Service-spezifischen Eventhandler werden direkt nach dem
(optionalen) globalen Host- oder Service-Eventhandler ausgeführt.

Eventhandler aktivieren
-----------------------

Eventhandler können durch die
[enable\_event\_handlers](#configmain-enable_event_handlers)-Direktive
in Ihrer Hauptkonfigurationsdatei programmweit aktiviert oder
deaktiviert werden.

Host- und Service-spezifische Eventhandler werden durch die
*event\_handler\_enabled*-Direktive in Ihrer
[Host](#objectdefinitions-host)- oder
[Service](#objectdefinitions-service)-Definition aktiviert oder
deaktiviert. Host- und Service-spezifische Eventhandler werden nicht
ausgeführt, wenn die globale
[enable\_event\_handlers](#configmain-enable_event_handlers)-Option
deaktiviert ist.

Eventhandler-Ausführungsreihenfolge
-----------------------------------

Wie bereits erwähnt werden globale Host- und Service-Eventhandler direkt
vor Host- oder Service-spezifischen Eventhandlern ausgeführt.

Eventhandler werden bei HARD-Problemen und Erholungszuständen direkt
nach dem Versand von Benachrichtigungen ausgeführt.

Eventhandler-Kommandos schreiben
--------------------------------

Eventhandler werden wahrscheinlich Shell- oder Perl-Scripte sein, aber
es ist jede Art von ausführbarer Datei denkbar, die von der
Kommandozeile aus lauffähig ist. Die Scripte sollten mindestens die
folgenden [Makros](#macros) als Argumente nutzen:

Für Services: [**\$SERVICESTATE\$**](#macrolist-servicestate) ,
[**\$SERVICESTATETYPE\$**](#macrolist-servicestatetype) ,
[**\$SERVICEATTEMPT\$**](#macrolist-serviceattempt)

Für Hosts: [**\$HOSTSTATE\$**](#macrolist-hoststate) ,
[**\$HOSTSTATETYPE\$**](#macrolist-hoststatetype) ,
[**\$HOSTATTEMPT\$**](#macrolist-hostattempt)

Die Scripte sollten die Werte der übergebenen Parameter untersuchen und
darauf basierend notwendige Aktionen ausführen. Der beste Weg, die
Funktionsweise von Eventhandlern zu verstehen, ist der Blick auf ein
Beispiel. Glücklicherweise finden Sie eins
[hier](#serviceeventhandlerexample).

![](../images/tip.gif) Hinweis: Zusätzliche Eventhandler-Scripte finden
Sie im *contrib/eventhandlers/*-Unterverzeichnis der
NAME-ICINGA-Distribution. Einige dieser Beispiel-Scripts demonstrieren
die Benutzung von [externen Befehlen](#extcommands), um
[redundante](#redundancy) und [verteilte](#distributed)
Überwachungsumgebungen zu implementieren.

Berechtigungen für Eventhandler-Befehle
---------------------------------------

Eventhandler werden normalerweise mit den gleichen Berechtigungen
ausgeführt wie der Benutzer, der NAME-ICINGA auf Ihrer Maschine
ausführt. Dies kann ein Problem darstellen, wenn Sie einen Eventhandler
schreiben möchten, der Systemdienste neu startet, da generell
root-Rechte benötigt werden, um diese Aufgaben zu erledigen.

Idealerweise sollten Sie den Typ von Eventhandler einschätzen und dem
NAME-ICINGA-Benutzer gerade genug Berechtigungen gewähren, damit er die
notwendigen Systembefehle ausführen kann. Vielleicht möchten Sie
[sudo](http://www.courtesan.com/sudo/sudo.html) ausprobieren, um das zu
erreichen.

Service Event Handler Beispiel
------------------------------

Das folgende Beispiel geht davon aus, dass Sie den HTTP-Server auf der
lokalen Maschine überwachen und *restart-httpd* als den
Eventhandler-Befehl für die HTTP-Service-Definition angegeben haben.
Außerdem nehmen wir an, dass Sie die Option *max\_check\_attempts* für
den Service auf einen Wert von 4 oder höher gesetzt haben (d.h., der
Service wird viermal geprüft, bevor angenommen wird, dass es ein
richtiges Problem gibt). Eine gekürzte Service-Definition könnte wie
folgt aussehen...

     define service{
            host_name               somehost
            service_description     HTTP
            max_check_attempts      4
            event_handler           restart-httpd
            ...
            }

Sobald der Service mit einem Eventhandler definiert wird, müssen wir
diesen Eventhandler als Befehlsfolge definieren. Eine Beispieldefinition
für *restart-httpd* sehen Sie nachfolgend. Beachten Sie die Makros in
der Kommandozeile, die an das Eventhandler-Script übergeben werden - sie
sind wichtig!

     define command{
            command_name    restart-httpd
            command_line    URL-ICINGA-LIBEXEC/eventhandlers/restart-httpd  $SERVICESTATE$ $SERVICESTATETYPE$ $SERVICEATTEMPT$
            }

Lassen Sie uns nun das Eventhandler-Script schreiben (das ist das
*URL-ICINGA-LIBEXEC/eventhandlers/restart-httpd*-Script).

    #!/bin/sh
    #
    # Eventhandler-Script für den Restart des Web-Servers auf der lokalen Maschine
    #
    # Anmerkung: Dieses Script wird den Web-Server nur dann restarten, wenn der Service
    #       dreimal erneut geprüft wurde (sich in einem "soft"-Zustand befindet)
    #       oder der Web-Service aus irgendeinem Grund in einen "hard"-Zustand fällt 
    # In welchem Status befindet sich der Service?
    case "$1" in
    OK)
            # Der Service hat sich gerade erholt, also tun wir nichts...
            ;;
    WARNING)
            # Wir kümmern uns nicht um WARNING-Zustände, denn der Dienst läuft wahrscheinlich noch...
            ;;
    UNKNOWN)
            # Wir wissen nicht, was einen UNKNOWN-Fehler auslösen könnte, also tun wir nichts...
            ;;
    CRITICAL)
            # Aha!  Der HTTP-Service scheint ein Problem zu haben - vielleicht sollten wir den Server neu starten...
            # Ist dies ein "Soft"- oder ein "Hard"-Zustand?
            case "$2" in
            # Wir sind in einem "Soft"-Zustand, also ist NAME-ICINGA mitten in erneuten Prüfungen, bevor es in einen
            # "Hard"-Zustand wechselt und Kontakte informiert werden...
            SOFT)
                    # Bei welchem Versuch sind wir? Wir wollen den Web-Server nicht gleich beim ersten Mal restarten,
                    # denn es könnte ein Ausrutscher sein!
                    case "$3" in
                    # Warte, bis die Prüfung dreimal wiederholt wurde, bevor der Web-Server restartet wird.
                    # Falls der Check ein viertes Mal fehlschlägt (nachdem wir den Web-Server restartet haben),
                    # wird der Zustandstyp auf "Hard" wechseln und Kontakte werden über das Problem informiert.
                    # Hoffentlich wird der Web-Server erfolgreich restartet, so dass der vierte Check zu einer
                    # "Soft"-Erholung führt. Wenn das passiert, wird niemand informiert, weil wir das Problem gelöst haben.
                    3)
                            echo -n "Restart des HTTP-Service (dritter kritischer "Soft"-Zustand)..."
                            # Aufrufen des Init-Scripts, um den HTTPD-Server zu restarten
                            /etc/rc.d/init.d/httpd restart
                            ;;
                            esac
                    ;;
            # Der HTTP-Service hat es irgendwie geschafft, in einen "Hard"-Zustand zu wechseln, ohne dass das Problem
            # behoben wurde. Er hätte durch den Code restartet werden sollen, aber aus irgendeinem Grund hat es nicht
            # funktioniert. Wir probieren es ein letztes Mal, okay?
            # Anmerkung: Kontakte wurden bereits darüber informiert, dass es ein Problem mit dem Service gibt (solange
            # Sie nicht Benachrichtungen für diesen Service deaktiviert haben.
            HARD)
                    echo -n "Restart des HTTP-Service..."
                    # Aufrufen des Init-Scripts, um den HTTPD-Server zu restarten
                    /etc/rc.d/init.d/httpd restart
                    ;;
            esac
            ;;
    esac
    exit 0

Das mitgelieferte Beispiel-Script wird versuchen, den Web-Server auf der
lokalen Maschine in zwei Fällen zu restarten:

-   nachdem der Service das dritte Mal erneut geprüft wurde und sich in
    einem kritischen "Soft"-Zustand befindet

-   nachdem der Service das erste Mal in einen kritischen "Hard"-Zustand
    wechselt

Das Script sollte theoretisch den Web-Server restarten und das Problem
beheben, bevor der Service in einen "Hard"-Problemzustand wechselt, aber
wir stellen eine Absicherung bereit, falls es nicht das erste Mal
funktioniert. Es ist anzumerken, dass der Eventhandler nur einmal
ausgeführt wird, wenn der Service in einen HARD-Zustand wechselt. Das
hält NAME-ICINGA davon ab, das Script zum Restart des Web-Servers
wiederholt auszuführen, wenn der Service in einem HARD-Problemzustand
bleibt. Das wollen Sie nicht. :-)

Das ist alles! Eventhandler sind ziemlich einfach zu schreiben und zu
implementieren, also versuchen Sie es und sehen, was Sie tun können.

Eventhandler
Eventhandler
Beispiel