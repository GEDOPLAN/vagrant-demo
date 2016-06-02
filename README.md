# vagrant-demo
Demo Beispiel für den Einsatz von Vagrant für Bereitstellung von Entwicklungsumgebungen.
Vorraussetzung: VirtualBox und Vagrant müssen installiert sein.
Nach dem Klonen einfach im Projektverzeichnis "vagrant up" auf der Konsole eingeben und die virtuelle Umgebung wird eingerichtet. (Kann einige Zeit dauern, da einiges heruntergeladen wird)

Die Umgebung enthält Java, eine MySql-Datenbank (DB: demo; User:demouser; Password: demopassword), einen Wildfly mit Anbindung an die Datenbank (Datasource:demo) + Managementuser (admin:admin) und Maven. Der Wildfly wird direkt als Servie gestartet und kann über systemd/systemctl gestartet und gestoppt werden. IP der virtuellen Maschine im lokalem Netzwerk auf dem Host ist:172.28.128.4.

Um zu testen das alles funktioniert:
Hochfahren mit: vagrant up
Einloggen mit: vagrant ssh
Wechseln zu Projektordner: cd /vagrant
Bauen + Deployen: mvn wildfly:deploy
-> Unter 172.28.128.4:8080/vagrant-demo sollte jetzt die Webanwendung bereit stehen.

Der Wildfly wird im Shareverzeichnis abgelegt, sodass man ihn auch direkt auf dem Entwicklerrechner in die IDE einbinden kann. Bei vagrant destroy wird der Wildfly wieder entfernt.
