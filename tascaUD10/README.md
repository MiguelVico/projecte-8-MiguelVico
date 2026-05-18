# README.md – Servidor de fitxers amb Samba (Ubuntu Server + Windows 11)

Aquest projecte consisteix en la implementació completa d’un servidor Samba a Ubuntu Server (Zorin) per compartir fitxers en xarxa amb clients Windows i Linux. S’han seguit els criteris d’una activitat d’avaluació que inclou: instal·lació del servei, creació d’usuaris, configuracions de permisos, carpetes personals, recursos compartits amb diferents nivells d’accés i bloqueig d’arxius `.zip`. També es mostra la interoperabilitat amb Windows 11 compartint una carpeta en mode lectura.

---

## 📋 Requisits previs

- **Màquina servidor:** Zorin OS (o qualsevol Ubuntu Server) amb accés root o sudo.
- **Màquina client:** Windows 11 (per a les comprovacions) i el mateix Zorin (per a proves locals).
- **Xarxa:** Les dues màquines han de tenir connectivitat IP (màquina virtual amb adaptador en mode pont o xarxa interna).

---

## 🚀 Resum de la configuració realitzada

### 1️⃣ Instal·lació de Samba
```bash
sudo apt update && sudo apt install samba -y
```

### 2️⃣ Estructura de directoris
- `/srv/samba/publica` → accessible anònimament (només lectura)
- `/srv/samba/compartida` → accés restringit per usuaris
- Carpeta personal de cada usuari (`/home/samba1`, etc.) → accessible via `[homes]`

**Permisos:** `770` (root:sambashare) per a `publica` i `compartida`.

### 3️⃣ Usuaris creats
| Usuari | Grup           | Shell               | Accés Samba |
|--------|----------------|---------------------|-------------|
| samba1 | sambashare     | /usr/sbin/nologin   | Sí          |
| samba2 | sambashare     | /usr/sbin/nologin   | Sí          |
| samba3 | sambashare     | /usr/sbin/nologin   | Sí          |

Tots tenen carpeta personal i contrasenya Samba.

### 4️⃣ Fitxer `smb.conf` final

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
    guest ok = yes
    read only = yes

[homes]
    comment = Directori personal de $U
    browsable = no
    read only = no
    valid users = %S
    create mask = 0700
    directory mask = 0700

[compartida]
    comment = Area de treball compartida
    path = /srv/samba/compartida
    read only = yes
    write list = samba1
    read list = samba2
    valid users = samba1, samba2
    veto files = /*.zip/
```

### 5️⃣ Comprovacions realitzades

| Exercici | Comprovació |
|----------|--------------|
| Accés anònim a `publica` | `smbclient //10.0.2.6/publica -N` llegeix `prova123.txt` |
| Carpeta personal de `samba1` | Accés des de Windows `\\10.0.2.6\samba1` amb escriptura |
| `compartida` per a `samba1` | Lectura + escriptura (pot crear/modificar fitxers) |
| `compartida` per a `samba2` | Només lectura |
| `compartida` per a `samba3` | Accés denegat |
| Bloqueig d’arxius `.zip` | Els fitxers `.zip` no es mostren al client |
| Compartició de Windows | Carpeta `compartidaWINDOWS` muntada a Zorin amb `cifs-utils` |

---

## 🖼️ Captures de pantalla

Totes les evidències gràfiques es troben a la carpeta [`/tascaUD10/imgUD10/`](./tascaUD10/imgUD10/) amb noms `c1.png` fins `c43.png`.  
Per a una explicació detallada, consulta la **[Guia completa (guia.md)](guia.md)**.

---

## 🧪 Resultats finals

✅ Samba instal·lat i funcionant.  
✅ Carpeta pública accessible anònimament en només lectura.  
✅ Carpeta `compartida` amb permisos granulars:  
- `samba1` pot llegir i escriure.  
- `samba2` només pot llegir.  
- `samba3` no hi pot accedir.  
✅ Bloqueig efectiu d’arxius `.zip`.  
✅ Carpetes personals accessibles per cada usuari.  
✅ Interoperabilitat Windows → Linux: recurs compartit de Windows muntat correctament a Zorin.  

---

## 📁 Estructura del repositori

```
/
├── guia.md                 # Explicació pas a pas amb anàlisi de captures
├── README.md               # Aquest fitxer
└── tascaUD10/
    └── imgUD10/            # Totes les captures (c1.png ... c43.png)
```

---

## 👤 Autor

Treball realitzat per [Miquel Vico] com a part de la UD10 – Servidor de fitxers amb Samba.  
Data: Maig 2026

---

## 📌 Nota

Aquest README és un resum executiu. Per a una correcció detallada segons la rúbrica de puntuació, consulta la **guia.md** on cada criteri està vinculat a la seva evidència gràfica.