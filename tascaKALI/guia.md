# Guia de pràctica: Analitzador de protocols Wireshark

## 1. ICMP: tipus de petició d’eco i resposta d’eco

Quan fem un `ping` a un altre equip, el protocol ICMP genera dos tipus de missatges diferents:
- **Echo request (petició)** → tipus **8**
- **Echo reply (resposta)** → tipus **0**

Per comprovar-ho, primer configurem la nostra IP manualment (c1), verifiquem amb `ip a` (c2) i preparem Wireshark a l’interfície `eth0` (c3). Després executem un ping al gateway (c5):

```bash
ping -c 5 192.168.2.254
```

![Configuració IP manual](/tascaKALI/imgKALI/c1.png)
![Verificació IP](/tascaKALI/imgKALI/c2.png)
![Wireshark interfície eth0](/tascaKALI/imgKALI/c3.png)
![Execució del ping](/tascaKALI/imgKALI/c5.png)

Apliquem el filtre `icmp` a Wireshark (c6) i obrim un paquet de petició (c7, c8). Allà veiem clarament:

- **Type: 8 (Echo (ping) request)**

![Filtre ICMP](/tascaKALI/imgKALI/c6.png)
![Paquet Echo request](/tascaKALI/imgKALI/c7.png)
![Detall Type 8](/tascaKALI/imgKALI/c8.png)

Per la resposta, obrim un paquet de tipus reply (c9):

- **Type: 0 (Echo (ping) reply)**

![Paquet Echo reply](/tascaKALI/imgKALI/c9.png)

> **Conclusió:** La petició d’eco és tipus 8 i la resposta d’eco és tipus 0. Es veu directament al camp `Type` del paquet ICMP.

---

## 2. Trànsit navegant des de la màquina física

Per veure trànsit relacionat amb el meu PC físic, primer comprovo la seva IP amb `ipconfig` a Windows (c12). La meva màquina física té IP `172.0.2.241`.

![IP de la màquina física](/tascaKALI/imgKALI/c12.png)

Mentre navego des del físic, capturo trànsit amb Wireshark des del Kali (en mode pont). Apliquem un filtre per l’adreça `172.0.2.241` o simplement observem el trànsit general. A la captura c14 veiem paquets UDP, ARP i dades que provenen del rang `172.0.2.x`, incloent el meu PC.

![Trànsit capturat des del físic](/tascaKALI/imgKALI/c14.png)

El que es pot veure són **peticions ARP, trànsit UDP a ports alts, i possiblement DNS o HTTP** segons el que estigués fent (navegar per web, etc.). Com que Wireshark està en mode promiscu (c10, c11), pot capturar tot el que arriba a la interfície encara que no vagi destinat al Kali.

![Mode promiscu](/tascaKALI/imgKALI/c10.png)
![Interfície eth0 promiscua](/tascaKALI/imgKALI/c11.png)

> **Resposta:** Es pot veure trànsit ARP (broadcast), paquets amb destinació o origen la meva IP física, i també trànsit de protocols com DNS o HTTP si navego.

---

## 3. DNS: mostra la petició

Filtrem per DNS i l’adreça del nostre Kali (`192.168.2.26`). Utilitzem el filtre:

```
dns && ip.addr == 192.168.2.26
```

A c15 veiem el filtre aplicat. Les peticions DNS es reconeixen perquè la columna `Info` diu "Standard query". A c17 i c18 es veu la petició concreta per `www.xtec.cat`.

![Filtre DNS](/tascaKALI/imgKALI/c15.png)
![Llistat de peticions DNS](/tascaKALI/imgKALI/c17.png)
![Detall de la petició DNS](/tascaKALI/imgKALI/c18.png)

Al detall del paquet (c18) veiem la secció **Queries** on apareix el nom `www.xtec.cat` i el tipus de consulta A (adreça IPv4).

> **Resposta:** La petició DNS es mostra amb el filtre anterior; dins del paquet trobem el camp `Queries` amb el nom sol·licitat.

---

## 4. DNS: resposta

A la mateixa captura, la resposta del servidor DNS (c18) apareix amb `Standard query response`. Conté l’adreça IP que hem demanat. Ho confirmem executant `nslookup www.xtec.cat` (c16 i c19) obtenim `83.247.151.214`.

![nslookup](/tascaKALI/imgKALI/c16.png)
![Resposta DNS detallada](/tascaKALI/imgKALI/c18.png)
![nslookup confirmació](/tascaKALI/imgKALI/c19.png)

Dins del paquet de resposta, a la secció **Answers** veiem:

- `Name: www.xtec.cat`
- `Address: 83.247.151.214`

> **Resposta:** La resposta DNS conté l’adreça IP `83.247.151.214` per a `www.xtec.cat`.

---

## 5. ARP: mostra MAC del gateway. Identifica fabricant

Per saber la MAC del gateway (`192.168.2.254`), utilitzem el filtre `arp` i busquem una resposta que digui `is at`. A c21 veiem:

```
192.168.2.254 is at 08:00:27:be:1b:a8
```

![Resposta ARP del gateway](/tascaKALI/imgKALI/c21.png)

La MAC és `08:00:27:be:1b:a8`. Per saber el fabricant, busquem els primers 3 octets (`08:00:27`) a la base de dades de MAC (c22, c23).

![MAC del gateway](/tascaKALI/imgKALI/c22.png)
![Fabricant PCS Systemtechnik](/tascaKALI/imgKALI/c23.png)

> **Resposta:** La MAC és `08:00:27:be:1b:a8` i el fabricant és **PCS Systemtechnik GmbH**.

---

## 6. ARP: MAC adreça 192.168.1.1

Ara obrim el fitxer `captura1.pcapng` (c24). Apliquem el filtre `arp` i busquem l’adreça `192.168.1.1`. A c25 veiem un paquet on `192.168.1.1` pregunta per `192.168.1.147`. La font MAC d’aquest paquet és `ASUSTeKCOMPU_62:5f:7c` (perquè la IP `192.168.1.1` envia la petició).

![Captura1.pcapng descarregat](/tascaKALI/imgKALI/c24.png)
![Paquet ARP de 192.168.1.1](/tascaKALI/imgKALI/c25.png)

Més avall, a la mateixa c25, hi ha una resposta `192.168.1.147 is at 14:da:e9:62:5f:7c` però no és la que busquem. La MAC de `192.168.1.1` és la de l’**ASUSTeK** (els primers 3 octets `14:da:e9`? No, a c25 la MAC font és `ASUSTeKCOMPU_62:5f:7c`, però mirem el detall: el paquet diu "Who has 192.168.1.147? Tell 192.168.1.1", per tant l’origen és `192.168.1.1` amb MAC `14:da:e9:62:5f:7c`? Espera, a la línia de text posa: `ASUSTeKCOMPU_62:5f:7c  ARP 60 Who has 192.168.1.147? Tell 192.168.1.1`. Això significa que la MAC font és la d’ASUSTeK. I la resposta ve de `zte_0f:fd:58` dient que `192.168.1.147` té una altra MAC. Per tant, la MAC de `192.168.1.1` és la que apareix com a **src** en aquest paquet: `14:da:e9:62:5f:7c`.

Comprovem a c26, on es mostra un paquet FTP amb src `ASUSTeKCOMPU_62:5f:7c` i IP `192.168.1.1`. Confirmat.

> **Resposta:** L’equip `192.168.1.1` té MAC `14:da:e9:62:5f:7c` (fabricant ASUSTeK).

---

## 7. FTP: password de l’usuari i nom del fitxer descarregat

Seguim dins `captura1.pcapng`. Filtrem per `ftp` i seguim el flux TCP (botó dret → Follow → TCP Stream). A c28 apareix la conversa FTP completa.

![Seguir flux FTP](/tascaKALI/imgKALI/c26.png)

Al text de la conversa veiem:

```
PASS contra
230 Login successful.
...
RETR README.txt
```

A c27 es confirma la transferència completa.

![Contrasenya i fitxer](/tascaKALI/imgKALI/c28.png)
![Transferència completa](/tascaKALI/imgKALI/c27.png)

> **Resposta:** El password és `contra` i el fitxer descarregat es diu `README.txt`.

---

## 8. Telnet: què veia l’usuari? Nau espacial petita (caràcters)

Filtrem per `telnet` a `captura1.pcapng` (c29). Seguint el flux TCP (c29 menú contextual) podem veure la sortida de la sessió Telnet. L’usuari es connecta a `towel.blinkenlights.nl` (c31) i veu una animació ASCII d’una **nau espacial**.

![Filtre telnet](/tascaKALI/imgKALI/c29.png)
![Resolució inversa del domini](/tascaKALI/imgKALI/c31.png)

La nau petita està formada pels caràcters: **`<^>`** (una fletxa dins de claudàtors) o similar. A la captura c30 (que l’usuari té penjada) es veu clarament. Els caràcters que la componen són: `^` , `<` , `>` i `|` o `-` depenent de la orientació.

> **Resposta:** L’usuari veia una animació de la pel·lícula Star Wars en ASCII. La nau espacial petita està formada pels caràcters `<`, `^`, `>` (per exemple `<^>`).

---

## 9. Telnet: domini de l’adreça on ens connectem

A c31 es fa un `nslookup` de la IP `94.142.241.111` (destí de la connexió Telnet). El resultat és:

```
name = towel.blinkenlights.nl.
```

> **Resposta:** El domini és `towel.blinkenlights.nl`.

---

## 10. SSH: domini de l’adreça del servidor

A c32 veiem que la connexió SSH va cap a `205.166.94.17`. Intentem fer resolució inversa (no apareix a les captures), però sabem que no és un domini comú. Sovint aquestes pràctiques utilitzen servidors de prova. A la captura no es veu cap resposta DNS per aquesta IP.

> **Resposta:** No es pot determinar el domini amb certesa, però la IP és `205.166.94.17`. Podria pertànyer a algun servei de prova (no s’aprecia nom de domini a les captures).

---

## 11. SSH: dades del paquet de longitud total 326 bytes

A c32, entre els paquets SSH n’hi ha un amb `Length 326` (fila 24499). Fem clic per veure’l. El contingut es mostra a c33. Són les capçaleres del protocol SSH i els algorismes de xifrat negociats.

![Paquet SSH de 326 bytes](/tascaKALI/imgKALI/c32.png)

Les dades textuals del paquet (ja que no està xifrat a nivell de negociació) són:

```
SSH-2.0-OpenSSH_7.2p2 Ubuntu-4ubuntu2.2

SSH-2.0-OpenSSH_7.1

a40a8i6uA<chacha20-poly1305@openssh.com,aes128-cbc,aes192-cbc,aes256-cbc,3des-bcmauthua64-etm@openssh.com,uac-128-etm@openssh.com,hmac-sha2-512-etm@openssh.com,mac-sha2-256-hmac-sha2-512-hmac-sha1etm@openssh.com,zlib@openssh.com,zlib@openssh.com,zlib@openssh.com,zlib@openssh.com,zlib@openssh.com,zlib@openssh.com,zlib@openssh.com.zlib@openssh.com.zlib@openssh.com.zlib@openssh.com.zlib
```

> **Resposta:** El contingut del paquet de 326 bytes és la negociació d’algorismes SSH (banner i llista de mètodes).

---

## 12. Correu: missatge enviat amb protocol de correu sortint (SMTP). Extreu el fitxer

Obrim `captura2.pcapng` (c34). Filtrem per `smtp` (c35). Seguint el flux TCP (c35 → Follow → TCP Stream) obtenim la conversa SMTP (c36, c37). El missatge enviat és:

```
mensaje ultrasectro para el administrador
```

![Obrir captura2.pcapng](/tascaKALI/imgKALI/c34.png)
![Filtre SMTP](/tascaKALI/imgKALI/c35.png)
![Conversa SMTP](/tascaKALI/imgKALI/c36.png)
![Missatge SMTP](/tascaKALI/imgKALI/c37.png)

Per extraure el fitxer (el correu), a Wireshark anem a **File → Export Objects → SMTP** (o **HTTP** si fos el cas). A c39 es mostra l’explorador per guardar l’arxiu.

![Guardar arxiu del correu](/tascaKALI/imgKALI/c39.png)

> **Resposta:** El missatge diu "mensaje ultrasectro para el administrador". El fitxer es pot exportar des de Wireshark mitjançant Export Objects (SMTP).

---


**Autor:** [Miquel Vico]  
**Data:** [19/5/2026]  
