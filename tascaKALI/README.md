# Pràctica Wireshark: Anàlisi de protocols de xarxa

Aquest repositori conté la resolució completa d’una pràctica d’anàlisi de trànsit amb **Wireshark** realitzada en un entorn Kali Linux. L’objectiu és aprendre a capturar i interpretar paquets dels protocols **ICMP, ARP, DNS, FTP, Telnet, SSH i SMTP**, així com extreure informació rellevant de fitxers PCAP.

## Contingut

- [`guia.md`](./guia.md) – Document amb les respostes a totes les preguntes de la rúbrica, acompanyades de captures de pantalla numerades (`c1.png` a `c39.png`).
- Carpeta `imgKALI/` – Conté totes les imatges utilitzades a la guia.

## Objectius assolits

| Protocol | Acció / Pregunta resposta |
|----------|----------------------------|
| **ICMP** | Identificats tipus 8 (Echo request) i 0 (Echo reply). |
| **ARP**   | Obtinguda la MAC del gateway (08:00:27:be:1b:a8) i fabricant (PCS Systemtechnik). |
| **DNS**   | Capturada petició i resposta per `www.xtec.cat` → IP `83.247.151.214`. |
| **FTP**   | Descobert el password `contra` i el fitxer `README.txt`. |
| **Telnet**| Visualitzada l’animació de `towel.blinkenlights.nl` i la nau espacial `<^>`. |
| **SSH**   | Analitzat paquet de 326 bytes amb la negociació d’algorismes. |
| **SMTP**  | Extret el missatge de correu: `mensaje ultrasectro para el administrador`. |

## Requisits previs

Per reproduir la pràctica necessites:

- **VirtualBox** amb una màquina **Kali Linux** (o qualsevol Linux amb Wireshark).
- Xarxa en mode pont amb IP `192.168.2.26/24` i gateway `192.168.2.254`.
- Fitxers de captura:
  - `captura1.pcapng` (trànsit FTP, Telnet, SSH)
  - `captura2.pcapng` (trànsit SMTP)

## Estructura de les captures

Totes les imatges estan numerades seguint l’ordre d’execució:

```bash
/tascaKALI/imgKALI/c1.png
/tascaKALI/imgKALI/c2.png
...
/tascaKALI/imgKALI/c39.png
```

Al fitxer `guia.md` s’insereixen amb la sintaxi:

```markdown
![Descripció](/tascaKALI/imgKALI/cX.png)
```

## Com utilitzar aquesta guia

1. Clona el repositori.
2. Obre `guia.md` amb un editor Markdown (VS Code, Typora, o directament a GitHub).
3. Consulta cada apartat per veure l’explicació i la captura associada.

## Autor

Treball realitzat per **[El teu nom]** com a part del mòdul de Fonaments de Xarxes.  
Professor: Carlos Alonso Martínez (basat en material de l’IES Carles Vallbona).

## Llicència

Aquest document està sota llicència **Creative Commons Reconeixement-NoComercial-CompartirIgual 4.0 Internacional (CC BY-NC-SA 4.0)**.
