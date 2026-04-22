Proiect Fii Practic 2026 - DevOps

Acest repository conține bazele unui proiect de CI/CD.

Vagrantfile se ocupă de crearea mașinilor folosite pentru aplicație,
Docker Definește mediul de execuție al aplicației.
Fișierele yaml automatizează instalarea a tot ce e necesar.
ngnix se ocupă de securitate și trafic.
.gitlab-ci.yml reactualizează la fiecare push aplicația automatizând run-ul ei la fiecare schimbare. 


Proiectul utilizează două mașini virtuale gestionate prin Vagrant:

1. gitlab.fiipractic.lan (192.168.100.20)
   Găzduiește GitLab CE și Container Registry 
   Execută pipeline-urile prin GitLab Runner (executor "shell").
   Acționează ca nod de control Ansible.

2. app.fiipractic.lan (192.168.100.10)
   Mașină de producție care are aplicația Spring Boot containerizată (port 8080).
   Utilizează Nginx pentru reverse proxy și securitate.
   Utilizează Netdata pentru monitorizare.

Ce se întâmplă?

Automatizarea se declanșează la fiecare commit prin ".gitlab-ci.yml":
Build: Codul este compilat. Se creează imaginea Docker. Imaginea este urcată în Container Registry.
Deploy: GitLab Runner lansează un playbook Ansible. Containerul vechi de pe mașina de producție este oprit și șters. Noua imagine este descărcată și rulată.

Ce poate să meargă rău? =(

1. Eroare SSL Docker Login: Rezolvată prin copierea certificatului Root CA local în "/etc/docker/certs.d/gitlab.fiipractic.lan:5050/".
2. Timeout la Docker Build (DNS): Mașinile nu avea acces la exterior pentru a descărca imaginile de bază. A  trebuit să adaug DNS-ul "8.8.8.8" în "/etc/resolv.conf".
3. Eroare certificat: Certificatul nu a fost generat cu SAN, a trebui emis unul nou, fortând SAN cu "--subject-alt-name".
4. Blocaj firewall:  firewalld al sistemului bloca accesarea din browser, soluția a fost dezactivarea lui prin "systemctl stop firewalld".
5. Spațiu insuficient: Mașinile trebuie să aibă memorie destulă sau vor da crash, fiind nevoit să le dai reboot, sau chiar să le construiești de la zero. (în special cea de git)
