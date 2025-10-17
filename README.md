**Infrastructure as Code (IaC)** automatiseeritud haldus- ja taastamisskriptid AWS EC2 keskkonnas, mis juhivad kahte Docker-põhist landing-page serverit:

- `docker-test` → arenduse ja testimise masin  
- `restore` → tootmis-/taastemasina automaatne uuesti ülesehitus GitHubi repo põhjal


 🌐 Masinad ja rollid

| Grupp        | IP             | Kirjeldus |
|---------------|----------------|------------|
| **docker-test** | 172.31.39.214 | Arendusserver, kus testitakse ja ehitatakse uus Docker image (landingpage:full) |
| **restore**     | 172.31.40.69  | Taastemasina versioon, mis laeb GitHubist repo, ehitab uue konteineri ja kontrollib HTTPS vastust |
| **controller**  | 172.31.24.251 | Ansible juhtmasin, kus käivitatakse playbookid |

---

 ⚙️ Playbookid

- restore_from_github_https_docker_test.yml`

- Uuendab testmasinat (`docker-test`)  
- Klooni GitHubi repo `AWS-test`  
- Loob/kontrollib self-signed TLS sertifikaadid  
- Ehitab Docker image’i ja käivitab konteineri pordil **443**

**Käivitus:**
ansible-playbook -i inventory.ini restore_from_github_https_docker_test.yml --limit docker_test

Inventory struktuur

[docker]
172.31.39.214 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/IKuusk_EC2test.pem
172.31.40.69 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/IKuusk_EC2test.pem

[restore]
172.31.40.69

[docker_test]
172.31.39.214

SSH võtmete haldus

Kõik serverid kasutavad sama EC2 võtmefaili:
~/.ssh/IKuusk_EC2test.pem

Ansible controlleril on eraldi GitHubi autentimise võti:
~/.ssh/id_ed25519.pub
(See on lisatud GitHubi SSH Keyde alla kui Ansible-controller.)

Vajalikud komponendid

Kõik playbookid eeldavad, et:
EC2 instantsid on Ubuntu 24.04 LTS
Port 443 on avatud (HTTPS)
Docker ja Git on paigaldatud automaatselt
GitHubi repo AWS-test sisaldab toimivat Dockerfile’i ja veebifailide struktuuri (HTML, CSS, pildid, nginx.conf, nginx.crt/key)

Kontroll

Pärast taastamist või uuendust saab tulemuse kontrollida:
curl -k https://<EC2-public-IP>/

Kui vastus sisaldab:
HTTP/1.1 200 OK
Server: nginx/1.29.2

siis süsteem töötab õigesti.

Märkused ja versioonihaldus

GitHubis on iga töökindel versioon tähistatud git tag abil (nt v1.0-restore-ok, v2.0-images-fix).

git tag -a v2.1 -m "Stable restore after SSL fix"
git push origin --tags

