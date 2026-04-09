# 🚀 TP2 — Application Web Java Multi-VM : Vagrant + Tomcat 9 + MySQL

![Vagrant](https://img.shields.io/badge/Vagrant-2.x-1563FF?logo=vagrant&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-20.04_LTS-E95420?logo=ubuntu&logoColor=white)
![Java](https://img.shields.io/badge/JDK-8_|_11_|_17-007396?logo=openjdk&logoColor=white)
![Tomcat](https://img.shields.io/badge/Tomcat-9.0.115-F8DC75?logo=apachetomcat&logoColor=black)
![MySQL](https://img.shields.io/badge/MySQL-8.0-4479A1?logo=mysql&logoColor=white)
![License](https://img.shields.io/badge/Licence-MIT-brightgreen)

> Déploiement d'une architecture **2-tiers** : une VM applicative (`srv-app`) hébergeant Tomcat 9 + une app Java Web, et une VM base de données (`srv-db`) hébergeant MySQL 8. Les deux VMs communiquent via un réseau privé Vagrant.

---

## 📋 Table des matières

- [Architecture](#-architecture)
- [Prérequis](#-prérequis)
- [Structure du projet](#-structure-du-projet)
- [Démarrage rapide](#-démarrage-rapide)
- [Partie 1 — Création des VMs Vagrant](#-partie-1--création-des-vms-vagrant)
- [Partie 2 — srv-app : JDK & Tomcat 9](#-partie-2--srv-app--installation-jdk--tomcat-9)
- [Partie 3 — srv-db : MySQL](#-partie-3--srv-db--installation-et-configuration-mysql)
- [Partie 4 — Déploiement de l'application Java](#-partie-4--déploiement-de-lapplication-java-web)
- [Partie 5 — Tests de connexion](#-partie-5--tests-de-connexion-et-validation)
- [Résultat final](#-résultat-final)
- [Commandes utiles](#-commandes-utiles)

---

## 🏛️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Machine Hôte                            │
│                                                             │
│  ┌──────────────────────────┐  ┌────────────────────────┐  │
│  │       srv-app            │  │       srv-db            │  │
│  │   192.168.56.10          │  │   192.168.56.20         │  │
│  │                          │  │                         │  │
│  │  ┌─────────────────┐     │  │  ┌──────────────────┐  │  │
│  │  │  JDK 8/11/17    │     │  │  │   MySQL 8.0      │  │  │
│  │  │  Tomcat 9 :8080 │─────┼──┼─▶│   DB: appdb      │  │  │
│  │  │  myapp.war      │     │  │  │   User: appuser  │  │  │
│  │  └─────────────────┘     │  │  └──────────────────┘  │  │
│  └──────────────────────────┘  └────────────────────────┘  │
│                                                             │
│  Réseau privé : 192.168.56.0/24                             │
│  Port forwarding : localhost:8080 → srv-app:8080            │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔧 Prérequis

| Outil | Version | Lien |
|-------|---------|------|
| VirtualBox | 6.x+ | https://www.virtualbox.org/wiki/Downloads |
| Vagrant | 2.x+ | https://developer.hashicorp.com/vagrant/downloads |
| Terminal | — | — |

```bash
vagrant --version    # Vagrant 2.x.x
VBoxManage --version # 6.x.x
```

---

## 📁 Structure du projet

```
tp2/
├── Vagrantfile                    ← Définition des 2 VMs
├── README.md                      ← Ce fichier
│
├── app/
│   ├── src/
│   │   └── main/
│   │       ├── java/
│   │       │   └── com/tp2/
│   │       │       ├── model/
│   │       │       │   └── Etudiant.java
│   │       │       ├── dao/
│   │       │       │   └── EtudiantDAO.java
│   │       │       └── servlet/
│   │       │           └── EtudiantServlet.java
│   │       └── webapp/
│   │           ├── WEB-INF/
│   │           │   └── web.xml
│   │           └── index.jsp
│   └── pom.xml
│
├── sql/
│   └── init.sql                   ← Script de création BDD
│
├── deploy.sh                      ← Script de gestion Tomcat
│
└── screenshots/
    ├── 01_vagrant_up.png
    ├── 02_vagrant_status.png
    ├── 03_ssh_srv_app.png
    ├── 04_jdk_install.png
    ├── 05_java_alternatives.png
    ├── 06_tomcat_install.png
    ├── 07_tomcat_status.png
    ├── 08_ssh_srv_db.png
    ├── 09_mysql_install.png
    ├── 10_mysql_db_create.png
    ├── 11_mysql_user_create.png
    ├── 12_war_deploy.png
    ├── 13_app_browser.png
    ├── 14_connexion_db.png
    └── 15_test_final.png
```

---

## ⚡ Démarrage rapide

```bash
# 1. Cloner le dépôt
git clone https://github.com/<votre-username>/tp2-vagrant-tomcat-mysql.git
cd tp2-vagrant-tomcat-mysql

# 2. Démarrer les deux VMs
vagrant up

# 3. Vérifier les statuts
vagrant status

# 4. Accéder à l'application
# http://localhost:8080/myapp
```

---

## 🏗️ Partie 1 — Création des VMs Vagrant

### Vagrantfile (2 VMs)

```ruby
Vagrant.configure("2") do |config|

  # ─── VM 1 : Serveur Applicatif ──────────────────────────────────
  config.vm.define "srv-app" do |app|
    app.vm.box      = "ubuntu/focal64"
    app.vm.hostname = "srv-app"

    app.vm.network "private_network",  ip: "192.168.56.10"
    app.vm.network "forwarded_port",   guest: 8080, host: 8080

    app.vm.provider "virtualbox" do |vb|
      vb.name   = "srv-app"
      vb.memory = "2048"
      vb.cpus   = 2
    end

    app.vm.synced_folder "./app", "/vagrant/app"

    app.vm.provision "shell", inline: <<-SHELL
      sudo apt update
      sudo apt install -y openjdk-8-jdk openjdk-11-jdk openjdk-17-jdk maven wget tar

      # Définir JAVA_HOME (JDK 11)
      echo 'export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64' >> /etc/environment
      echo 'export PATH=$JAVA_HOME/bin:$PATH'                    >> /etc/environment
      source /etc/environment

      # Installer Tomcat 9
      cd /opt
      wget -q https://archive.apache.org/dist/tomcat/tomcat-9/v9.0.115/bin/apache-tomcat-9.0.115.tar.gz
      tar -xzf apache-tomcat-9.0.115.tar.gz
      mv apache-tomcat-9.0.115 tomcat9
      chmod +x tomcat9/bin/*.sh

      # Créer le service systemd
      cat > /etc/systemd/system/tomcat.service << 'EOF'
[Unit]
Description=Apache Tomcat 9
After=network.target
[Service]
Type=forking
User=root
Environment="JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64"
Environment="CATALINA_HOME=/opt/tomcat9"
ExecStart=/opt/tomcat9/bin/startup.sh
ExecStop=/opt/tomcat9/bin/shutdown.sh
Restart=always
[Install]
WantedBy=multi-user.target
EOF

      systemctl daemon-reload
      systemctl enable tomcat
      systemctl start tomcat
    SHELL
  end

  # ─── VM 2 : Serveur Base de Données ─────────────────────────────
  config.vm.define "srv-db" do |db|
    db.vm.box      = "ubuntu/focal64"
    db.vm.hostname = "srv-db"

    db.vm.network "private_network", ip: "192.168.56.20"

    db.vm.provider "virtualbox" do |vb|
      vb.name   = "srv-db"
      vb.memory = "1024"
      vb.cpus   = 1
    end

    db.vm.synced_folder "./sql", "/vagrant/sql"

    db.vm.provision "shell", inline: <<-SHELL
      sudo apt update
      sudo apt install -y mysql-server

      # Autoriser les connexions distantes
      sed -i 's/127.0.0.1/0.0.0.0/' /etc/mysql/mysql.conf.d/mysqld.cnf
      systemctl restart mysql

      # Initialiser la base de données
      mysql -u root << 'SQL'
CREATE DATABASE IF NOT EXISTS appdb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER IF NOT EXISTS 'appuser'@'%' IDENTIFIED BY 'AppPass@2024';
GRANT ALL PRIVILEGES ON appdb.* TO 'appuser'@'%';
FLUSH PRIVILEGES;
USE appdb;
CREATE TABLE IF NOT EXISTS etudiants (
  id         INT AUTO_INCREMENT PRIMARY KEY,
  nom        VARCHAR(100) NOT NULL,
  prenom     VARCHAR(100) NOT NULL,
  email      VARCHAR(150) UNIQUE NOT NULL,
  filiere    VARCHAR(80),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
INSERT INTO etudiants (nom, prenom, email, filiere) VALUES
  ('Diallo',  'Amadou',  'amadou.diallo@univ.sn',  'Informatique'),
  ('Ndiaye',  'Fatou',   'fatou.ndiaye@univ.sn',   'Réseaux'),
  ('Sarr',    'Ibrahima','ibrahima.sarr@univ.sn',   'Génie Logiciel');
SQL
    SHELL
  end

end
```

### Démarrage des VMs

```bash
vagrant up           # Démarre et provisionne les 2 VMs
vagrant status       # Vérifie que les 2 VMs sont running
```

| Capture | Description |
|---------|-------------|
| ![01](./screenshots/01_vagrant_up.png) | `vagrant up` — provisioning des 2 VMs |
| ![02](./screenshots/02_vagrant_status.png) | `vagrant status` — srv-app et srv-db running |

---

## 🖥️ Partie 2 — srv-app : Installation JDK & Tomcat 9

### Connexion SSH

```bash
vagrant ssh srv-app
# vagrant@srv-app:~$
```

| Capture | Description |
|---------|-------------|
| ![03](./screenshots/03_ssh_srv_app.png) | Connexion SSH à srv-app |

### Vérification JDK 8, 11, 17

```bash
# Vérifier les JDKs installés par le provisioning
/usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java  -version
/usr/lib/jvm/java-11-openjdk-amd64/bin/java      -version
/usr/lib/jvm/java-17-openjdk-amd64/bin/java      -version

# Enregistrer les alternatives
sudo update-alternatives --install /usr/bin/java java \
  /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java 1

sudo update-alternatives --install /usr/bin/java java \
  /usr/lib/jvm/java-11-openjdk-amd64/bin/java 2

sudo update-alternatives --install /usr/bin/java java \
  /usr/lib/jvm/java-17-openjdk-amd64/bin/java 3

# Sélectionner la version active
sudo update-alternatives --config java
# → Choisir JDK 11 (recommandé pour Tomcat 9)

java -version
# openjdk version "11.x.xx"
```

| Capture | Description |
|---------|-------------|
| ![04](./screenshots/04_jdk_install.png) | JDK 8, 11, 17 — vérification des versions |
| ![05](./screenshots/05_java_alternatives.png) | `update-alternatives` — JDK 11 sélectionné |

### Vérification Tomcat 9

```bash
sudo systemctl status tomcat
# ● tomcat.service - Apache Tomcat 9
#    Active: active (running)

# Test local
curl -I http://localhost:8080
# HTTP/1.1 200
```

| Capture | Description |
|---------|-------------|
| ![06](./screenshots/06_tomcat_install.png) | Tomcat 9 installé dans `/opt/tomcat9` |
| ![07](./screenshots/07_tomcat_status.png) | `systemctl status tomcat` — active (running) |

---

## 🗄️ Partie 3 — srv-db : Installation et Configuration MySQL

### Connexion SSH

```bash
# Depuis un nouveau terminal
vagrant ssh srv-db
# vagrant@srv-db:~$
```

| Capture | Description |
|---------|-------------|
| ![08](./screenshots/08_ssh_srv_db.png) | Connexion SSH à srv-db |

### Vérification MySQL

```bash
sudo systemctl status mysql
# ● mysql.service - MySQL Community Server
#    Active: active (running)

mysql --version
# mysql  Ver 8.0.xx
```

| Capture | Description |
|---------|-------------|
| ![09](./screenshots/09_mysql_install.png) | MySQL 8 — active (running) |

### Création de la base de données

```bash
sudo mysql -u root
```

```sql
-- Créer la base de données
CREATE DATABASE IF NOT EXISTS appdb
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;

-- Créer l'utilisateur applicatif
CREATE USER 'appuser'@'%' IDENTIFIED BY 'AppPass@2024';

-- Accorder les droits
GRANT ALL PRIVILEGES ON appdb.* TO 'appuser'@'%';
FLUSH PRIVILEGES;

-- Vérification
SHOW DATABASES;
SELECT User, Host FROM mysql.user WHERE User = 'appuser';
```

| Capture | Description |
|---------|-------------|
| ![10](./screenshots/10_mysql_db_create.png) | Création de la BDD `appdb` |
| ![11](./screenshots/11_mysql_user_create.png) | Création de `appuser` avec ses droits |

### Création de la table etudiants

```sql
USE appdb;

CREATE TABLE etudiants (
  id         INT AUTO_INCREMENT PRIMARY KEY,
  nom        VARCHAR(100) NOT NULL,
  prenom     VARCHAR(100) NOT NULL,
  email      VARCHAR(150) UNIQUE NOT NULL,
  filiere    VARCHAR(80),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Données de test
INSERT INTO etudiants (nom, prenom, email, filiere) VALUES
  ('Diallo',  'Amadou',  'amadou.diallo@univ.sn',   'Informatique'),
  ('Ndiaye',  'Fatou',   'fatou.ndiaye@univ.sn',    'Réseaux'),
  ('Sarr',    'Ibrahima','ibrahima.sarr@univ.sn',    'Génie Logiciel');

SELECT * FROM etudiants;
```

### Configuration accès distant

```bash
# Vérifier la configuration (déjà modifiée par le provisioning)
grep bind-address /etc/mysql/mysql.conf.d/mysqld.cnf
# bind-address = 0.0.0.0

# Test de connexion depuis srv-app
mysql -u appuser -p'AppPass@2024' -h 192.168.56.20 appdb -e "SELECT * FROM etudiants;"
```

---

## ☕ Partie 4 — Déploiement de l'Application Java Web

### Structure Maven (`pom.xml`)

```xml
<project>
  <groupId>com.tp2</groupId>
  <artifactId>myapp</artifactId>
  <version>1.0</version>
  <packaging>war</packaging>

  <dependencies>
    <!-- Servlet API -->
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>4.0.1</version>
      <scope>provided</scope>
    </dependency>
    <!-- Connecteur MySQL -->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>8.0.33</version>
    </dependency>
    <!-- JSTL -->
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>jstl</artifactId>
      <version>1.2</version>
    </dependency>
  </dependencies>
</project>
```

### Modèle — `Etudiant.java`

```java
package com.tp2.model;

public class Etudiant {
    private int id;
    private String nom, prenom, email, filiere;

    // Getters & Setters
    public int    getId()      { return id; }
    public String getNom()     { return nom; }
    public String getPrenom()  { return prenom; }
    public String getEmail()   { return email; }
    public String getFiliere() { return filiere; }

    public void setId(int id)           { this.id = id; }
    public void setNom(String nom)      { this.nom = nom; }
    public void setPrenom(String p)     { this.prenom = p; }
    public void setEmail(String e)      { this.email = e; }
    public void setFiliere(String f)    { this.filiere = f; }
}
```

### DAO — `EtudiantDAO.java`

```java
package com.tp2.dao;

import com.tp2.model.Etudiant;
import java.sql.*;
import java.util.*;

public class EtudiantDAO {

    private static final String URL  = "jdbc:mysql://192.168.56.20:3306/appdb?useSSL=false&serverTimezone=UTC";
    private static final String USER = "appuser";
    private static final String PASS = "AppPass@2024";

    private Connection getConnection() throws SQLException {
        return DriverManager.getConnection(URL, USER, PASS);
    }

    public List<Etudiant> findAll() throws SQLException {
        List<Etudiant> list = new ArrayList<>();
        String sql = "SELECT * FROM etudiants ORDER BY id";
        try (Connection cn = getConnection();
             Statement  st = cn.createStatement();
             ResultSet  rs = st.executeQuery(sql)) {
            while (rs.next()) {
                Etudiant e = new Etudiant();
                e.setId(rs.getInt("id"));
                e.setNom(rs.getString("nom"));
                e.setPrenom(rs.getString("prenom"));
                e.setEmail(rs.getString("email"));
                e.setFiliere(rs.getString("filiere"));
                list.add(e);
            }
        }
        return list;
    }

    public void save(Etudiant e) throws SQLException {
        String sql = "INSERT INTO etudiants (nom, prenom, email, filiere) VALUES (?,?,?,?)";
        try (Connection cn = getConnection();
             PreparedStatement ps = cn.prepareStatement(sql)) {
            ps.setString(1, e.getNom());
            ps.setString(2, e.getPrenom());
            ps.setString(3, e.getEmail());
            ps.setString(4, e.getFiliere());
            ps.executeUpdate();
        }
    }

    public void delete(int id) throws SQLException {
        String sql = "DELETE FROM etudiants WHERE id = ?";
        try (Connection cn = getConnection();
             PreparedStatement ps = cn.prepareStatement(sql)) {
            ps.setInt(1, id);
            ps.executeUpdate();
        }
    }
}
```

### Servlet — `EtudiantServlet.java`

```java
package com.tp2.servlet;

import com.tp2.dao.EtudiantDAO;
import com.tp2.model.Etudiant;
import javax.servlet.*;
import javax.servlet.http.*;
import javax.servlet.annotation.WebServlet;
import java.io.IOException;
import java.util.List;

@WebServlet("/etudiants")
public class EtudiantServlet extends HttpServlet {

    private final EtudiantDAO dao = new EtudiantDAO();

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        try {
            List<Etudiant> liste = dao.findAll();
            req.setAttribute("etudiants", liste);
            req.getRequestDispatcher("/index.jsp").forward(req, resp);
        } catch (Exception e) {
            throw new ServletException("Erreur BDD : " + e.getMessage(), e);
        }
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        String action = req.getParameter("action");
        try {
            if ("ajouter".equals(action)) {
                Etudiant e = new Etudiant();
                e.setNom(req.getParameter("nom"));
                e.setPrenom(req.getParameter("prenom"));
                e.setEmail(req.getParameter("email"));
                e.setFiliere(req.getParameter("filiere"));
                dao.save(e);
            } else if ("supprimer".equals(action)) {
                dao.delete(Integer.parseInt(req.getParameter("id")));
            }
        } catch (Exception e) {
            throw new ServletException(e);
        }
        resp.sendRedirect("etudiants");
    }
}
```

### Vue — `index.jsp`

```jsp
<%@ page contentType="text/html;charset=UTF-8" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<!DOCTYPE html>
<html>
<head>
  <title>Gestion des Étudiants</title>
  <meta charset="UTF-8">
  <style>
    body { font-family: Arial, sans-serif; max-width: 900px; margin: 40px auto; }
    h1   { color: #2E75B6; }
    table{ width:100%; border-collapse:collapse; margin-top:20px; }
    th   { background:#2E75B6; color:#fff; padding:10px; }
    td   { border:1px solid #ddd; padding:9px; }
    tr:nth-child(even){ background:#f2f7fc; }
    form { margin-top:30px; background:#f9f9f9; padding:20px; border-radius:8px; }
    input,select { padding:8px; margin:5px; border:1px solid #ccc; border-radius:4px; }
    button { background:#2E75B6; color:#fff; padding:9px 18px; border:none; border-radius:4px; cursor:pointer; }
  </style>
</head>
<body>
  <h1>🎓 Gestion des Étudiants</h1>
  <p>Connexion BDD : <strong>192.168.56.20:3306 / appdb</strong></p>

  <table>
    <tr><th>ID</th><th>Nom</th><th>Prénom</th><th>Email</th><th>Filière</th><th>Action</th></tr>
    <c:forEach var="e" items="${etudiants}">
    <tr>
      <td>${e.id}</td>
      <td>${e.nom}</td>
      <td>${e.prenom}</td>
      <td>${e.email}</td>
      <td>${e.filiere}</td>
      <td>
        <form method="post" action="etudiants" style="margin:0;padding:0;background:none;">
          <input type="hidden" name="action" value="supprimer">
          <input type="hidden" name="id"     value="${e.id}">
          <button style="background:#dc3545;">Supprimer</button>
        </form>
      </td>
    </tr>
    </c:forEach>
  </table>

  <form method="post" action="etudiants">
    <h3>Ajouter un étudiant</h3>
    <input type="hidden" name="action" value="ajouter">
    <input type="text"   name="nom"     placeholder="Nom"     required>
    <input type="text"   name="prenom"  placeholder="Prénom"  required>
    <input type="email"  name="email"   placeholder="Email"   required>
    <input type="text"   name="filiere" placeholder="Filière">
    <button type="submit">➕ Ajouter</button>
  </form>
</body>
</html>
```

### Build et déploiement

```bash
# Sur srv-app — compiler et packager
cd /vagrant/app
mvn clean package -DskipTests

# Déployer le WAR
sudo cp target/myapp-1.0.war /opt/tomcat9/webapps/myapp.war

# Vérifier le déploiement
ls /opt/tomcat9/webapps/
# myapp/   myapp.war   ROOT/  ...

# Consulter les logs Tomcat
sudo tail -f /opt/tomcat9/logs/catalina.out
```

| Capture | Description |
|---------|-------------|
| ![12](./screenshots/12_war_deploy.png) | WAR déployé dans `/opt/tomcat9/webapps/` |
| ![13](./screenshots/13_app_browser.png) | Application accessible sur `http://localhost:8080/myapp/etudiants` |

---

## 🧪 Partie 5 — Tests de connexion et validation

### Test 1 — Ping entre les VMs

```bash
# Depuis srv-app → pinger srv-db
vagrant ssh srv-app -c "ping -c 3 192.168.56.20"

# Résultat attendu :
# PING 192.168.56.20 : 64 bytes from 192.168.56.20 : icmp_seq=1 ttl=64 time=0.4ms
```

### Test 2 — Connexion MySQL depuis srv-app

```bash
vagrant ssh srv-app

# Tester la connexion à la BDD distante
mysql -u appuser -p'AppPass@2024' -h 192.168.56.20 appdb \
  -e "SELECT * FROM etudiants;"
```

Résultat attendu :

```
+----+--------+---------+--------------------------+----------------+
| id | nom    | prenom  | email                    | filiere        |
+----+--------+---------+--------------------------+----------------+
|  1 | Diallo | Amadou  | amadou.diallo@univ.sn    | Informatique   |
|  2 | Ndiaye | Fatou   | fatou.ndiaye@univ.sn     | Réseaux        |
|  3 | Sarr   | Ibrahima| ibrahima.sarr@univ.sn    | Génie Logiciel |
+----+--------+---------+--------------------------+----------------+
```

### Test 3 — Curl sur l'application

```bash
curl -s -o /dev/null -w "%{http_code}" http://localhost:8080/myapp/etudiants
# 200
```

### Test 4 — Ajout d'un étudiant via l'interface

```bash
curl -X POST http://localhost:8080/myapp/etudiants \
  -d "action=ajouter&nom=Fall&prenom=Omar&email=omar.fall@univ.sn&filiere=DevOps"

# Vérification en BDD
mysql -u appuser -p'AppPass@2024' -h 192.168.56.20 appdb \
  -e "SELECT * FROM etudiants WHERE nom='Fall';"
```

### Test 5 — Vérification des services

```bash
# Sur srv-app
sudo systemctl status tomcat
java -version

# Sur srv-db
sudo systemctl status mysql
mysql -u root -e "SHOW DATABASES; SELECT User,Host FROM mysql.user;"
```

| Capture | Description |
|---------|-------------|
| ![14](./screenshots/14_connexion_db.png) | Test MySQL depuis srv-app — connexion réussie |
| ![15](./screenshots/15_test_final.png) | Application finale — liste des étudiants affichée |

---

## 📜 Script deploy.sh

```bash
#!/bin/bash
# deploy.sh — Gestion de Tomcat 9 + déploiement WAR

RED='\033[0;31m'; GREEN='\033[0;32m'; YELLOW='\033[1;33m'
BLUE='\033[0;34m'; CYAN='\033[0;36m'; BOLD='\033[1m'; RESET='\033[0m'

TOMCAT_HOME="/opt/tomcat9"
WEBAPPS="$TOMCAT_HOME/webapps"
LOGS="$TOMCAT_HOME/logs/catalina.out"
SERVICE="tomcat"
DB_HOST="192.168.56.20"
DB_USER="appuser"
DB_PASS="AppPass@2024"
DB_NAME="appdb"

print_header() {
  clear
  echo -e "${BLUE}${BOLD}"
  echo "  ╔════════════════════════════════════════════════╗"
  echo "  ║    🚀  TP2 — GESTION TOMCAT + MYSQL — MENU    ║"
  echo "  ║    srv-app:192.168.56.10  ↔  srv-db:192.168.56.20  ║"
  echo "  ╚════════════════════════════════════════════════╝"
  echo -e "${RESET}"
  # Statut Tomcat
  if systemctl is-active --quiet $SERVICE; then
    echo -e "  Tomcat : ${GREEN}● RUNNING${RESET}   |   DB : ${CYAN}$DB_HOST/$DB_NAME${RESET}"
  else
    echo -e "  Tomcat : ${RED}● ARRÊTÉ${RESET}    |   DB : ${CYAN}$DB_HOST/$DB_NAME${RESET}"
  fi
  echo ""
}

while true; do
  print_header
  echo -e "  ${GREEN}1)${RESET}  Démarrer Tomcat"
  echo -e "  ${RED}2)${RESET}  Arrêter Tomcat"
  echo -e "  ${YELLOW}3)${RESET}  Redémarrer Tomcat"
  echo -e "  ${CYAN}4)${RESET}  Statut Tomcat"
  echo -e "  ${CYAN}5)${RESET}  Déployer un WAR"
  echo -e "  ${CYAN}6)${RESET}  Supprimer une application"
  echo -e "  ${CYAN}7)${RESET}  Logs Tomcat (catalina.out)"
  echo -e "  ${CYAN}8)${RESET}  Lister les applications"
  echo -e "  ${CYAN}9)${RESET}  Tester connexion MySQL"
  echo -e "  ${RED}0)${RESET}  Quitter"
  echo ""
  read -rp "  Votre choix [0-9] : " CHOICE
  echo ""

  case $CHOICE in
    1)  sudo systemctl start   $SERVICE && echo -e "${GREEN}✅ Tomcat démarré.${RESET}" ;;
    2)  sudo systemctl stop    $SERVICE && echo -e "${GREEN}✅ Tomcat arrêté.${RESET}" ;;
    3)  sudo systemctl restart $SERVICE && echo -e "${GREEN}✅ Tomcat redémarré.${RESET}" ;;
    4)  sudo systemctl status  $SERVICE --no-pager ;;
    5)
        read -rp "  Chemin du WAR : " WAR_PATH
        if [ -f "$WAR_PATH" ]; then
          WAR_NAME=$(basename "$WAR_PATH"); APP="${WAR_NAME%.war}"
          sudo cp "$WAR_PATH" "$WEBAPPS/"
          echo -e "${GREEN}✅ Déployé : http://192.168.56.10:8080/$APP${RESET}"
        else
          echo -e "${RED}❌ Fichier introuvable.${RESET}"
        fi ;;
    6)
        ls "$WEBAPPS" | grep -v 'ROOT\|manager\|host-manager\|examples\|docs'
        read -rp "  Nom de l'app : " APP
        sudo rm -rf "$WEBAPPS/$APP" "$WEBAPPS/${APP}.war"
        echo -e "${GREEN}✅ '$APP' supprimée.${RESET}" ;;
    7)  sudo tail -n 50 "$LOGS" ;;
    8)  for d in "$WEBAPPS"/*/; do
          echo -e "  ${GREEN}▶${RESET}  $(basename $d)  →  http://192.168.56.10:8080/$(basename $d)"
        done ;;
    9)
        echo -e "${CYAN}🔍 Test connexion MySQL sur $DB_HOST...${RESET}"
        mysql -u "$DB_USER" -p"$DB_PASS" -h "$DB_HOST" "$DB_NAME" \
          -e "SELECT COUNT(*) AS nb_etudiants FROM etudiants;" 2>&1
        if [ $? -eq 0 ]; then
          echo -e "${GREEN}✅ Connexion MySQL réussie !${RESET}"
        else
          echo -e "${RED}❌ Connexion échouée. Vérifiez srv-db.${RESET}"
        fi ;;
    0)  echo -e "${GREEN}Au revoir !${RESET}"; exit 0 ;;
    *)  echo -e "${RED}❌ Option invalide.${RESET}" ;;
  esac
  read -rp "  [Entrée] pour continuer..."
done
```

---

## ✅ Résultat final

| Composant | Valeur |
|-----------|--------|
| srv-app IP | `192.168.56.10` — Ubuntu 20.04 LTS |
| srv-db  IP | `192.168.56.20` — Ubuntu 20.04 LTS |
| JDK 8 | `/usr/lib/jvm/java-8-openjdk-amd64` |
| JDK 11 ✅ | `/usr/lib/jvm/java-11-openjdk-amd64` (actif) |
| JDK 17 | `/usr/lib/jvm/java-17-openjdk-amd64` |
| Tomcat 9 | `/opt/tomcat9` — port `8080` |
| MySQL 8 | `srv-db:3306` — base `appdb` |
| Utilisateur DB | `appuser` / `AppPass@2024` |
| Application | `http://localhost:8080/myapp/etudiants` |
| Script | `/opt/tomcat9/deploy.sh` |

---

## 🛠️ Commandes utiles

```bash
# ── Vagrant ──────────────────────────────────────────────────
vagrant up              # Démarrer les 2 VMs
vagrant up srv-app      # Démarrer une VM spécifique
vagrant halt            # Éteindre toutes les VMs
vagrant ssh srv-app     # SSH sur srv-app
vagrant ssh srv-db      # SSH sur srv-db
vagrant status          # Voir l'état des VMs
vagrant destroy -f      # Supprimer toutes les VMs
vagrant reload --provision  # Reprovisionner

# ── Tomcat (depuis srv-app) ───────────────────────────────────
sudo systemctl start|stop|restart|status tomcat
sudo tail -f /opt/tomcat9/logs/catalina.out
ls /opt/tomcat9/webapps/

# ── MySQL (depuis srv-db) ─────────────────────────────────────
sudo systemctl start|stop|status mysql
sudo mysql -u root
mysql -u appuser -p'AppPass@2024' -h 192.168.56.20 appdb

# ── Build Maven (depuis srv-app) ─────────────────────────────
cd /vagrant/app
mvn clean package -DskipTests
sudo cp target/myapp-1.0.war /opt/tomcat9/webapps/myapp.war
```

---

## 📄 Licence

Distribué sous licence MIT.

---

*TP2 réalisé dans le cadre du cours DevOps / Administration Système*
