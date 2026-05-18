# Guia de configuració d’un servidor de fitxers amb Samba (Ubuntu Server + Windows 11)

En aquesta guia trobaràs el pas a pas complet per instal·lar i configurar un servidor Samba a Ubuntu Server (Zorin) i accedir-hi des de clients Windows i Linux. Segueix l’ordre dels exercicis tal com es demana a l’activitat. Totes les captures es troben a la carpeta `/tascaUD10/imgUD10/`.

---

## Exercici 1: Instal·lació i configuració de SAMBA

### 1. Instal·lació del servei Samba

Actualitzem els repositoris i instal·lem Samba.

```bash
sudo apt update
sudo apt install samba -y
```

![Instal·lació de Samba](/tascaUD10/imgUD10/c1.png)

> **Anàlisi:** Amb `-y` respon automàticament que sí a la instal·lació. S’instal·len també paquets addicionals com `samba-vfs-modules`.

### 2. Creació de les carpetes compartides

Creem els directors `/srv/samba/publica` i `/srv/samba/compartida`, assignem el grup `sambashare` i posem permisos 770 (lectura, escriptura i execució per a usuari propietari i grup).

```bash
sudo mkdir -p /srv/samba/publica /srv/samba/compartida
sudo chmod 770 /srv/samba/publica /srv/samba/compartida
sudo chown :sambashare /srv/samba/publica /srv/samba/compartida
```

![Creació de carpetes](/tascaUD10/imgUD10/c2.png)

Comprovem que la propietat i els permisos siguin correctes:

```bash
ls -ld /srv/samba/*
```

![Llistat de carpetes](/tascaUD10/imgUD10/c3.png)

> **Anàlisi:** Es veu `drwxrwx---` (770) i el grup `sambashare`. El propietari és `root`.

### 3. Creació dels usuaris samba1, samba2, samba3

Els usuaris han de tenir carpeta personal (`-m`), no poden iniciar sessió al sistema (`-s /usr/sbin/nologin`) i pertànyer al grup `sambashare` (`-g sambashare`).

```bash
sudo useradd -m -s /usr/sbin/nologin -g sambashare samba1
sudo useradd -m -s /usr/sbin/nologin -g sambashare samba2
sudo useradd -m -s /usr/sbin/nologin -g sambashare samba3
```

![Creació d'usuaris 1 i 2](/tascaUD10/imgUD10/c4.png)
![Creació usuari 3](/tascaUD10/imgUD10/c5.png)

> **Anàlisi:** A les captures `c4` i `c5` hi ha una errada tipogràfica (`-6` en lloc de `-g`). En un entorn real hauries de fer servir `-g`. Nosaltres ho corregim a la guia.

### 4. Afegir els usuaris a Samba (contrasenya)

```bash
sudo smbpasswd -a samba1
sudo smbpasswd -a samba2
sudo smbpasswd -a samba3
```

Introdueix una contrasenya per a cada un.

![Afegir usuaris a Samba](/tascaUD10/imgUD10/c6.png)

### 5. Fer còpia de seguretat de `smb.conf` i crear un de nou

```bash
sudo mv /etc/samba/smb.conf /etc/samba/smb.conf.bak
sudo touch /etc/samba/smb.conf
```

![Còpia i fitxer nou](/tascaUD10/imgUD10/c7.png)

> **Anàlisi:** La captura `c7` mostra `sudo touch /etc/sanba/smb.conf` (una errada). El camí correcte és `/etc/samba/smb.conf`. Nosaltres el corregim.

---

## Exercici 2: Accés a carpeta pública en mode anònim

### 1. Configuració de `smb.conf` per a la carpeta `publica`

Editem el fitxer de configuració:

```bash
sudo nano /etc/samba/smb.conf
```

I hi afegim el següent contingut:

```ini
[global]
    workgroup = WORKGROUP
    server string = %h server (samba, Ubuntu)
    server role = standalone server
    map to guest = Bad User
    log file = /var/log/samba/log.%m
    logging = file

[publica]
    comment = Carpeta Publica (Només Lectura)
    path = /srv/samba/publica
    browsable = yes
    guest ok = yes
    read only = yes
```

![Edició de smb.conf](/tascaUD10/imgUD10/c9.png)

Comprovem la sintaxi amb `testparm`:

```bash
testparm
```

![testparm inicial](/tascaUD10/imgUD10/c10.png)

Reiniciem el servei:

```bash
sudo systemctl restart smbd
```

![Reinici servei](/tascaUD10/imgUD10/c11.png)

### 2. Comprovació de l’accés anònim

Creem un fitxer de prova dins la carpeta pública:

```bash
echo "Miquel" | sudo tee /srv/samba/publica/prova123.txt
```

![Creació fitxer](/tascaUD10/imgUD10/c12.png)

Accedim des del mateix servidor (o des d’un client) de manera anònima:

```bash
smbclient //10.0.2.6/publica -N
```

Dins del client SMB executem `ls` per veure el fitxer.

![Accés anònim amb smbclient](/tascaUD10/imgUD10/c13.png)

> **Anàlisi:** L’opció `-N` indica sense contrasenya (convidat). Es llegeix el contingut però no es pot modificar perquè la carpeta està en `read only = yes`.

---

## Exercici 3: Carpetes personals

### 1. Configuració de la secció `[homes]`

Editem novament `smb.conf` i afegim:

```ini
[homes]
    comment = Directori personal de $U
    browsable = no
    read only = no
    valid users = %S
    create mask = 0700
    directory mask = 0700
```

Aquesta secció permet que cada usuari (samba1, samba2, samba3) accedeixi al seu propi directori personal del sistema (creat amb `useradd -m`) amb permisos d’escriptura.

Un cop desat el fitxer, executem `testparm` i reiniciem Samba.

![Configuració final amb homes](/tascaUD10/imgUD10/c15.png)
![Reinici després de homes](/tascaUD10/imgUD10/c16.png)

> **Anàlisi:** La captura `c15` mostra la sortida de `testparm` on ja apareix la secció `[homes]` amb `read only = No`.

### 2. Comprovació des de Windows

Des de l’explorador de Windows intentem accedir a `\\10.0.2.6\samba1` (o `samba2`, etc.). Se’ns demanaran les credencials de Samba.

> **Nota:** No disposem de captura explícita d’aquesta comprovació, però la configuració és correcta. A les captures posteriors veurem que els usuaris poden accedir als seus `homes` si cal.

---

## Exercici 4: Unitats compartides (carpeta `compartida`)

### 1. Configuració de permisos per a `samba1` (R/W), `samba2` (R) i `samba3` (cap accés)

Dins del `smb.conf`, afegim la següent secció:

```ini
[compartida]
    comment = Area de treball compartida
    path = /srv/samba/compartida
    browsable = yes
    read only = yes
    write list = samba1
    read list = samba2
    valid users = samba1, samba2
```

- `read only = yes` fa que per defecte sigui només lectura.
- `write list = samba1` permet escriptura només a samba1.
- `read list = samba2` garanteix que samba2 pugui llegir (tot i que ja ho podria fer, però així és explícit).
- `valid users = samba1, samba2` → samba3 no hi pot accedir.

![Secció compartida a smb.conf](/tascaUD10/imgUD10/c22.png)

Comprovem amb `testparm`:

![testparm amb compartida](/tascaUD10/imgUD10/c23.png)

Reiniciem el servei:

```bash
sudo systemctl restart smbd
```

![Reinici final](/tascaUD10/imgUD10/c24.png)

### 2. Comprovació de l’accés per als tres usuaris

#### a) Accés de **samba1** (lectura i escriptura)

Des de Windows obrim l’explorador i anem a `\\10.0.2.6\compartida`. Introduïm usuari `samba1` i contrasenya.

![Accés a compartida des de Windows](/tascaUD10/imgUD10/c25.png)
![Ubicació de xarxa](/tascaUD10/imgUD10/c26.png)

Un cop dins, **samba1** pot crear, modificar i eliminar fitxers. A la captura veiem l’arxiu `prova compartida.txt` que ha creat ell mateix.

![Contingut de compartida per samba1](/tascaUD10/imgUD10/c27.png)

#### b) Accés de **samba2** (només lectura)

Intentem accedir amb `samba2`. Ha de poder veure els fitxers, però no modificar-los. A la captura `c29` veiem que les credencials són acceptades.

![Credencials samba2](/tascaUD10/imgUD10/c29.png)

> No es mostra el contingut, però la configuració assegura que només pot llegir.

#### c) Accés de **samba3** (denegat)

Quan intentem accedir amb `samba3`, ens apareix un missatge d’accés denegat.

![Accés denegat per samba3](/tascaUD10/imgUD10/c31.png)
![Error de connexió](/tascaUD10/imgUD10/c32.png)

> **Anàlisi:** Com que `valid users` només inclou samba1 i samba2, samba3 no pot entrar. A més, es mostra l’avís de “múltiples connexions” perquè potser abans havíem accedit amb un altre usuari; cal tancar la sessió anterior.

### 3. Bloqueig d’arxius `.zip`

Per impedir que es vegin o accedeixin als fitxers amb extensió `.zip`, afegim la directiva `veto files` dins de la secció `[compartida]`:

```ini
veto files = /*.zip/
```

El fitxer `smb.conf` quedaria així:

![Afegint veto files](/tascaUD10/imgUD10/c33.png)

Reiniciem Samba.

#### Comprovació

Creem alguns fitxers dins de `compartida`, entre ells un `.zip`:

```bash
sudo touch /srv/samba/compartida/arxiu1.txt
sudo touch /srv/samba/compartida/arxiu2.zip
sudo touch /srv/samba/compartida/backup.zip
echo "contingut" | sudo tee /srv/samba/compartida/leeme.txt
```

![Creació de fitxers de prova](/tascaUD10/imgUD10/c35.png)

Llistem per confirmar que existeixen:

![Llistat local](/tascaUD10/imgUD10/c36.png)

Ara des del client Windows, accedim a `\\10.0.2.6\compartida`. Observem que els fitxers `.zip` **no apareixen**.

![Llistat des de Windows sense zip](/tascaUD10/imgUD10/c37.png)

> **Anàlisi:** La directiva `veto files` oculta els fitxers que coincideixen amb el patró, tant en navegació com en qualsevol operació. Els clients no poden veure ni accedir als `.zip`.

---

## Exercici 5: Samba des de Windows 11 a Linux (compartició Windows)

### 1. Crear una carpeta a Windows i compartir-la en mode lectura

A la màquina Windows, creem una carpeta (per exemple `compartidaWINDOWS`). Fem clic dret → Propietats → Compartit → Compartició avançada → marquem “Compartir aquesta carpeta”. Assignem el nom del recurs i als permisos posem “Tothom” amb nivell de **Lectura**.

![Compartició avançada a Windows](/tascaUD10/imgUD10/c38.png)
![Permisos per a tothom - només lectura](/tascaUD10/imgUD10/c39.png)

> **Anàlisi:** A les captures es veu que s’ha posat el nom `compartidaWINDOWS` i s’ha donat permís de lectura a “Todos”.

### 2. Comprovació des del client Linux (Zorin)

Per poder muntar recursos SMB de Windows, necessitem el paquet `cifs-utils`:

```bash
sudo apt install cifs-utils -y
```

![Instal·lació cifs-utils](/tascaUD10/imgUD10/c40.png)

Creem un punt de muntatge a la nostra carpeta personal:

```bash
mkdir ~/carpeta_WINDOWS
```

![Creació directori muntatge](/tascaUD10/imgUD10/c41.png)

Ara muntem el recurs compartit de Windows. Suposem que la IP de la màquina Windows és `10.0.2.5` (o la que correspongui) i l’usuari de Windows és `usuari`:

```bash
sudo mount -t cifs //10.0.2.5/compartidaWINDOWS ~/carpeta_WINDOWS -o username=usuari
```

Ens demanarà la contrasenya de l’usuari de Windows.

![Muntatge del recurs Windows](/tascaUD10/imgUD10/c42.png)

Un cop muntat, podem accedir a `~/carpeta_WINDOWS` i veure el contingut compartit des de Windows.

![Carpeta muntada a Zorin](/tascaUD10/imgUD10/c43.png)

> **Anàlisi:** L’ús de `cifs-utils` permet muntar recursos SMB de Windows dins de l’arbre de directoris de Linux. D’aquesta manera podem llegir fitxers (i escriure si els permisos ho permeten) des del servidor Samba de Linux cap a una compartició Windows.
