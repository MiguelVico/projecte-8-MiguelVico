# Guia pràctica: NAT i VPN amb IPFire

En aquesta activitat hem configurat dos mètodes per accedir a serveis interns d’una xarxa des de l’exterior: **DNAT** (port forwarding) i **VPN** (OpenVPN). Utilitzem tres màquines virtuals: un **Zorin** (servidor intern amb SSH i web), un **IPFire** (tallafoc i router) i un **client Windows** (simula l’exterior). Totes les captures estan ordenades per seguir el procés pas a pas.

---

## 1. Preparació de l’entorn i instal·lació de serveis al Zorin

Primer actualitzem els repositoris i instal·lem els serveis **SSH** i **Apache** al Zorin. Això ens permetrà exposar després aquests serveis cap a l’exterior.

```bash
sudo apt update
```

![Actualització de repositoris](/tascaVPN/imgVPN/c1.png)

> **Explicació:** `sudo apt update` actualitza la llista de paquets disponibles. És el primer pas abans d’instal·lar qualsevol programari.

```bash
sudo apt install openssh-server apache2 -y
```

![Instal·lació SSH i Apache](/tascaVPN/imgVPN/c2.png)

> **Explicació:** Amb `-y` respon automàticament que sí a tot. `openssh-server` permet connexions remotes per terminal, i `apache2` és el servidor web.

Ara editem la pàgina web per defecte perquè mostri un missatge personalitzat (el nostre nom).

```bash
sudo nano /var/www/html/index.html
```

![Edició de l’index.html](/tascaVPN/imgVPN/c3.png)

Hi posem el següent contingut:

```html
<h1>Hola, soc el Miquel</h1>
<p>Aquest es el servidor Zorin darrere l' IPFire</p>
```

![Contingut de l’index.html](/tascaVPN/imgVPN/c4.png)

> **Explicació:** Aquest fitxer és el que es veurà quan algú accedeixi al nostre servidor web. Canviar-lo és una bona pràctica per identificar clarament el nostre equip.

Comprovem localment que els serveis funcionen.

```bash
ssh localhost
```

![Prova SSH local](/tascaVPN/imgVPN/c5.png)

> **Explicació:** `ssh localhost` intenta connectar al mateix equip. Ens demana confirmar la clau del servidor i després la contrasenya de l’usuari. Si tot va bé, ens deixa dins del terminal del Zorin.

```bash
curl http://localhost
```

![Prova web local](/tascaVPN/imgVPN/c6.png)

> **Explicació:** `curl` és una eina per fer peticions HTTP. Veiem que retorna exactament el nostre missatge personalitzat. El servidor web ja funciona.

---

## 2. Configuració de DNAT (Destination NAT) a IPFire

El DNAT permet redirigir peticions que arriben a la IP pública de l’IPFire cap a un equip de la xarxa interna (Zorin). Al nostre cas, la IP “pública” del IPFire és `10.0.2.8` (a la xarxa NAT de VirtualBox).

Accedim a la interfície web de l’IPFire i anem a **Firewall > Regles del Cortafuegos**. Allà crearem dues regles: una per al SSH (port 22) i una per al web (port 80).

![Pantalla de regles buida](/tascaVPN/imgVPN/c7.png)

> **Explicació:** Encara no hi ha cap regla. Les regles de DNAT es defineixen a l’apartat de NAT de destinació.

Omplim el formulari per redirigir el port 80 (HTTP) al Zorin (IP `192.169.26.1`).

![Formulari DNAT per HTTP](/tascaVPN/imgVPN/c8.png)

**Paràmetres importants:**
- **Usar traducció de direccions de xarxa (NAT)** → marcat.
- **NAT de destinació (DNAT - reenviament de ports)** → seleccionat.
- **Destí** → IP del Zorin: `192.169.26.1`.
- **Protocol** → `TCP`.
- **Port de destí** → `80` (el port intern del servei).
- **Port extern (NAT)** → buit (es manté el mateix).

Afegim la regla i l’apliquem. Repetim el mateix per al port `22` (SSH). Un cop fetes, veiem les regles actives.

![Llista de regles DNAT](/tascaVPN/imgVPN/c9.png)

> **Explicació:** Aquí es mostra la regla que hem creat: tot el tràfic TCP que arribi a la interfície RED (pública) amb destí al port 80 serà redirigit a la IP `192.169.26.1` (Zorin), també al port 80.

### Comprovació des del client (exterior)

Des de la màquina client (Windows) obrim un navegador i escrivim la IP pública del IPFire: `10.0.2.8`

![Accés web via DNAT](/tascaVPN/imgVPN/c10.png)

> **Explicació:** Com que hem configurat el DNAT, la connexió a `10.0.2.8` (port 80) és rebuda per l’IPFire i redirigida al Zorin. El navegador mostra el nostre missatge. Això demostra que l’accés extern funciona.

> **Nota:** Per connectar per SSH des del client s’hauria de fer `ssh usuari@10.0.2.8` (suposant que l’usuari existeixi al Zorin). La prova no està capturada però seria similar.

---

## 3. Configuració de VPN amb OpenVPN

La VPN ens permet connectar el client remot **com si estigués dins de la xarxa interna**. Així podrem accedir al Zorin utilitzant la seva IP privada (`192.169.26.1`) en comptes de la IP pública.

### 3.1 Crear la connexió VPN a IPFire

A l’IPFire, anem a **Servicios > OpenVPN**. La llista de connexions està buida.

![Llista de connexions buida](/tascaVPN/imgVPN/c11.png)

Clicke **Agregar** i triem el tipus **Host-to-Net (Roadwarrior)**. Això permet que un únic client (l’ordinador portàtil d’un treballador) es connecti a tota la nostra xarxa.

![Selecció de tipus de connexió](/tascaVPN/imgVPN/c12.png)

Configurem la connexió amb nom `VPN26`, activada, i generem un certificat per a l’usuari **Miquel**.

![Dades de la connexió i generació de certificat](/tascaVPN/imgVPN/c13.png)

**Paràmetres destacats:**
- **Nom:** VPN26
- **Grup d’adreces IP:** Dinàmic (per defecte)
- **Generar certificat:** posem el nom complet, organització `foodlogistic`, país `Spain`, validesa 730 dies.
- **Contrasenya PKCS12:** serveix per protegir el certificat que descarregarem.

A la pestanya **Opcions avançades** podem configurar si volem redirigir tot el trànsit del client per la VPN (Gateway redirect). Ho deixem per defecte.

![Opcions avançades](/tascaVPN/imgVPN/c14.png)

Un cop desat, apareixerà un enllaç per descarregar dos fitxers: `VPN26.ovpn` (perfil de configuració) i `VPN26.p12` (certificat + clau privada).

![Fitxers per descarregar](/tascaVPN/imgVPN/c15.png)

> **Explicació:** L’arxiu `.ovpn` conté la configuració que el client OpenVPN necessita per connectar-se. El `.p12` és el certificat digital que autentica l’usuari. Cal enviar aquests fitxers al client (en el nostre cas, els copiem manualment).

### 3.2 Configuració del client (Windows)

Al client Windows, primer editem l’arxiu `hosts` perquè resolgui correctament el nom del servidor IPFire (`ipfire.foodlogistic.test`). Obrim el bloc de notes com a administrador i obrim `C:\Windows\System32\drivers\etc\hosts`.

![Edició de l’arxiu hosts](/tascaVPN/imgVPN/c16.png)

Afegim la línia:
```
192.169.26.254    ipfire.foodlogistic.test
```

> **Explicació:** El fitxer `.ovpn` fa referència al nom `ipfire.foodlogistic.test`. Si no el resolem, el client no trobarà el servidor. Com que no tenim DNS intern, afegim aquesta entrada manual.

Instal·lem el client OpenVPN Community Edition. Descarreguem l’instal·lador des de [openvpn.net](https://openvpn.net/community-downloads/) i executem amb opcions per defecte.

![Instal·lador OpenVPN](/tascaVPN/imgVPN/c17.png)

Després d’instal·lar, copiem els dos fitxers descarregats (`VPN26.ovpn` i `VPN26.p12`) a la carpeta de configuració de l’usuari: `C:\Users\<nom_usuari>\OpenVPN\config\`.

Dins d’aquesta carpeta hauríem de tenir els dos fitxers (el .ovpn i una subcarpeta amb el .p12? En realitat el .p12 es pot posar al mateix directori que el .ovpn).

![Carpeta config amb els fitxers](/tascaVPN/imgVPN/c18.png)

> **Explicació:** OpenVPN GUI busca automàticament els perfils `.ovpn` dins de la carpeta `config`. Si el perfil requereix un certificat `.p12`, ha d’estar accessible (normalment al mateix lloc).

Ara obrim l’OpenVPN GUI des de la barra d’icones (amagat a la safata del sistema). Fem clic dret i seleccionem **Import**.

![Menú Import](/tascaVPN/imgVPN/c19.png)

Seleccionem el fitxer `VPN26.ovpn`. Llavors, fem clic dret sobre el nou perfil i triem **Connect**.

Ens demanarà la contrasenya del certificat (la que vam posar al crear-lo a IPFire). La introduïm i guardem si volem.

![Sol·licitud de contrasenya](/tascaVPN/imgVPN/c20.png)

Mentrestant, podem veure el contingut del fitxer `.ovpn`:

![Contingut del fitxer .ovpn](/tascaVPN/imgVPN/c21.png)

> **Explicació:** Observem que el `remote` apunta a `10.0.2.8` (la IP pública de l’IPFire) i al port `1194` (per defecte d’OpenVPN). També especifica protocols de seguretat i autenticació.

### 3.3 Estat de la connexió

Quan la connexió s’estableix, apareix un missatge de log. En aquest cas hi ha un advertiment sobre rutes (no greu).

![Log de connexió](/tascaVPN/imgVPN/c22.png)

> **Explicació:** L’error “Some routes were not successfully added” pot passar si el client ja té una ruta per defecte diferent, però normalment la connexió funciona igualment. El més important és que el `Initialization Sequence Completed` indica que la VPN està activa.

A la interfície web de l’IPFire, a **OpenVPN > Control y estado de conexión**, veiem l’usuari **Miquel** com a `CONECTADO`.

![Estat de la connexió a IPFire](/tascaVPN/imgVPN/c23.png)

> **Explicació:** Això confirma que el servidor IPFire ha acceptat la connexió del client i li ha assignat una IP virtual (dins del rang `10.151.246.0/24`).

### 3.4 Accés als serveis interns via VPN

Ara, des del client Windows, podem accedir al servei web del Zorin utilitzant la seva **IP privada** (`192.169.26.1`) perquè la VPN ens col·loca virtualment dins de la xarxa interna.

![Accés web via VPN](/tascaVPN/imgVPN/c24.png)

> **Explicació:** Tot i que el client està físicament a una xarxa externa (NAT de VirtualBox), la túnica VPN li permet comunicar-se directament amb la IP `192.169.26.1`. Això és molt útil per a serveis que no volem exposar públicament amb DNAT.

Per comprovar l’accés SSH, des del terminal del client es podria executar `ssh usuari@192.169.26.1`. Funcionaria igual que si estiguéssim connectats a la mateixa xarxa local.

---

## 4. Comparativa entre DNAT i VPN

| Aspecte | DNAT (Port Forwarding) | VPN (OpenVPN) |
|---------|------------------------|----------------|
| **Complexitat** | Baixa. Només cal crear una regla al tallafoc. | Alta. Cal generar certificats, configurar client i servidor. |
| **Seguretat** | Exposa ports concrets a tot Internet. Si el servei té vulnerabilitats, quedes exposat. | Molt alta. Tot el trànsit va xifrat i autenticat. Només s’accedeix després d’establir la túnica. |
| **Visibilitat** | L’equip intern rep connexions directament des de l’exterior (la seva IP es pot amagar parcialment). | El client obté una IP de la xarxa interna i pot veure tots els equips (si el routing ho permet). |
| **Ús típic** | Exposar serveis concrets com un servidor web, SSH o jocs. | Accés complet d’un empleat remot a tota la xarxa de l’empresa. |
| **Escalabilitat** | Cada servei nou requereix una regla nova. | Un sol túnels VPN dona accés a tots els serveis interns. |
| **Rendiment** | Gairebé nul·la sobrecàrrega (només NAT). | El xifrat pot reduir una mica la velocitat, però generalment acceptable. |

**Conclusió:**  
- **DNAT** és ràpid i senzill per a exposar serveis puntuals, però menys segur.  
- **VPN** és més segur i flexible, ideal per a teletreball o administració remota de tota la xarxa.  

A la pràctica, moltes empreses combinen ambdues: VPN per als empleats i DNAT per a serveis públics com una pàgina web corporativa.

---

## Referències

- [Wiki d’IPFire](https://wiki.ipfire.org)
- [HOWTO d’OpenVPN Community](https://community.openvpn.net/openvpn/wiki/HOWTO)
