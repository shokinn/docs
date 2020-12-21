Willkommen zum IO wargame im smash the stack Netzwerk.
------------------------------------------------------

Du hast den schwierigsten Teil schon hinter dir indem du zu uns gefunden hast. Hier bekommst du die Möglichkeit mit sowohl klassischen als auch neueren Verwundbarkeiten in Software zu experimentieren. Weil einige von Euch vielleicht nicht genau wissen, wie ein solches Wargame funktioniert, geben wir euch hier einen Crashkurs in den folgenden Abschnitten. Wenn Du ein erfahrener Wargamer bist, wirst Du all das schon kennen und zum letzten Abschnitt springen wollen, wo die Besonderheiten dieses Spiels erklärt werden.

Die Probleme werden in Form von Programmen gestellt. Zu Beginn beschränken diese sich auf wenige Zeilen mit offensichtlichen Fehlern; später werden sie größer und schließlich geht es um echte Software. Der Sinn ist immer, den Fehler so auszunutzen, dass Du das Programm kontrollieren kannst und es tut, was Du willst; meistens wirst Du eine Shell spawnen wollen.

Das ganze basiert darauf, dass die Programme sog. SUID-binaries sind (http://de.wikipedia.org/wiki/Setuid). Kurz gesagt bedeutet das, dass die Programme nicht als der User laufen, als der Du eingeloggt bist. Der Punkt ist, Kontrolle über die Ausführung zu übernehmen und Deinen eigenen Shellcode auszuführen sodass Du schließlich das Passwort für das nächste Level auslesen kannst.


Wie fängt man an
----------------

Zuerst werde ich dich durch's erste Level führen. Du bist als User "level1" angemeldet und kannst dadurch auch nur auf Dateien zugreifen, die level1 besitzt oder für jeden zugreifbar sind.

  level1@io:~# cd /levels
  level1@io:/levels# ls -las level1
  8 -r-sr-x--- 1 level2 level1 7500 Nov 16  2007 level1

Wenn Du es ausführst, fragt es nach einem Passwort, das Du irgendwie finden musst. Wenn Du es dann eingibst, kriegst Du eine neue Shell mit level2-Rechten, über die dann das Passwort für level2 ausgelesen werden kann.

  level1@io:/levels$ ./level1 [hier kommt das Passwort rein, das du finden musst]
  Win.
  level1@io:/levels$ id
  uid=1001(level1) gid=1001(level1) euid=1002(level2) groups=1001(level1),1029(nosu)

Wie Du an der Ausgabe von "id" erkennen kannst, hast du nun die euid (effective user id) von level2. Jetzt kannst du Dateien, die zu level2 gehören lesen und genau das muss auch getan werden um die Passwort-Datei vom nächsten Level auszulesen.

  level1@io:/levels$ cat /home/level2/.pass
  [PASSWORT FÜR LEVEL2]

Jetzt hast Du das level2 Passwort und kannst dich nun damit einloggen (schließe dazu die aktuelle Verbindung). Wenn du dich als level2 angemeldet hast, willst du das der Welt vielleicht mitteilen; das kannst du tun, indem du deinen Usernamen, schlauen Spruch oder irgendwas anderes (nettes) in das tagfile schreibst. Zum Beispiel:

  level2@io:~$ echo "<p>superleetzor was here and pwnd level1</p>" >> tags

Das kriegt man dann online unter
http://io.netgarage.org:84/tags/level2.html
zu Gesicht.

Das war's im Großen und Ganzen auch schon. Es ist so ziemlich alles in den tagfiles erlaubt, also sei kreativ, vergiss aber dabei nicht den gesunden Menschenverstand und schalte Javascript ab, wenn du sie Dir im Browser ansiehst...


FAQ
---

Q: Ich bin ziemlich neu, was das alles angeht. Wie schwer ist das Spiel? Ist das für mich machbar?
A: Es ist ein stufenweises Spiel, das bis ungefähr level10 relativ einfach ist. Diese levels solltest du unabhängig von deiner Vorgeschichte, Alter und Geschlecht lösen können, wenn du durchhalten kannst und vielleicht ein bisschen nach Hilfe fragst. Danach wirst Du die Freude gehabt haben, die Basics ziemlich gut erlernt zu haben. Das Spiel geht dann in Richtung der etwas fortgeschritteneren Levels weiter und es ist keine Schande, dort stecken zu bleiben, nach Hilfe oder Tipps zu fragen oder es einfach mal eine Zeit lang ruhen zu lassen. Smashthestack war und wird für die nächste Zeit stabil bleiben.

Q: Kann ich irgendwo Dateien schreiben?
A: Ja, ins /tmp Verzeichnis kannst du schreiben. Der Ordner ist aber so aufgebaut, dass Du die vorhandenen Dateien nicht auflisten oder sehen kannst, damit man keinen Einblick in die Dateien anderer hat. Du solltest dir deinen eigenen Unterordner machen, in dem Du dann arbeiten kannst, zum Beispiel so:

   mkdir /tmp/etwasschwerzuerratendes
   cd /tmp/etwasschwerzuerratendes

Jetzt kannst du Dateien schreiben, auflisten, speichern und so weiter. Wir räumen in regelmäßigen Abständen das Verzeichnis auf, was normalerweise im Chat angekündigt wird. Normalerweise ist es eine gute Idee ein lokales Backup Deiner Arbeit zu haben.

Q: Habt ihr eine Liste von Papers, die man für Level X lesen kann?
A: Normalerweise gibt es einige Sachen, die du lesen kannst, aber keine levelspezifischen Listen. Andererseits ist Nachforschungen anstellen und das Problem erkennen Teil des Spiels und deshalb wird es nicht immer ein allwissendes Dokument geben, IO ist nämlich kein Lesen-und-Verstehen-Test.

Q: Warum kann ich su nicht benutzen?
A: Su erstellt viele Prozesse. Warum auch immer. Weil wir versuchen, den Server für jeden Stabil zu behalten, limitieren wir die Anzahl der Prozesse und haben auch su deaktiviert. Daher musst Du stattdessen neu verbinden.

Q: Warum kann ich nicht nano, vim, ... zum editieren der tagfiles nutzen?
A: Die tagfiles sind "nur anhängen" gesetzt und wegen dem sog. "Editorbug" schreiben Editoren ganze Teile der Datei neu, anstatt nur anzuhängen. Du wirst den anhängen-Operator (>>) verwenden müssen.

Q: Ich finde dieses Readme wirklich gut, soll ich es übersetzen?
A: Gerne! Komm einfach in unseren IRC oder maile es jemandem. Es sollten email-Adressen in der MOTD stehen.

Q: Ich bemühe mich sehr, zu lernen aber jeder verdammte Shellcode, den ich teste segfaultet!
A: Du versuchst wahrscheinlich die levels oder Deinen testcode manuell zu kompilieren, ohne in Betracht zu ziehen, dass manche Speicherbereiche normalerweise nicht ausführbar sind. Das ist die momentane Einstellung und wir versuchen das nicht vor Spielern zu verstecken. Die meisten unserer Level haben einen ausführbaren Stack und es gibt dafür mehrere Gründe. Hauptsächlich weil die Methoden, spezielle Schutzmechanismen zu umgehen zu aufwendig sind, um in jedem Level verwendet zu werden. Die späteren Level greifen diese Themen aber durchaus an.

Falls du deinen shellcode testen willst, kannst du einen solchen Code verwenden:

#include <sys/mman.h>
#include <string.h>
#include <stdio.h>

char sc[]= "your shellcode here";

int main(){
        void * a = mmap(0, 4096, PROT_EXEC |PROT_READ | PROT_WRITE, MAP_ANONYMOUS | MAP_SHARED, -1, 0);
        printf("allocated executable memory at: %p\n", a);
        ((void (*)(void)) memcpy(a, sc, sizeof(sc)))();
}

Q: Warum enthält dieses Dokument absolut keine Schreibfehler?
A: Es wurde von bla geschrieben, aber nicht übersetzt.


Spiel-Spezifisches
------------------

- levels sind im Verzeichnis /levels
- Passwörter werden im home-Verzeichnis des levels gespeichert, die Datei heißt .pass. Zum Beispiel enthält /home/level2/.pass das Passwort für den User "level2".
- Chat:
	Es gibt einen Chatroom auf unserem IRC Netzwerk irc.netgarage.org, ssl port 6697. Du kannst auch den Webclienten zum Verbinden nutzen.
	http://www.netgarage.org/cgiirc/
- Forum:
	Auf unserer Webseite http://forum.netgarage.org/ gibt es eines. Wahrscheinlich wird Dir der Chatroom aber schneller und besser helfen.
- ASLR ist aus und die meisten level haben einen ausführbaren Stack.