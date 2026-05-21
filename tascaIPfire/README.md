# Configuració de Proxy Squid a IPFire

## Descripció

Aquest projecte documenta la configuració d'un servei de proxy **Squid** a **IPFire** per controlar l'accés a Internet des d'una xarxa local. S'han implementat diferents mètodes de filtrat de contingut.

## Entorn

| Element | Configuració |
|---------|--------------|
| Servidor IPFire | 2.29 (x86_64) |
| Xarxa GREEN (interna) | 192.169.26.254/24 |
| Xarxa RED (externa) | DHCP (10.0.2.8) |
| Client | 192.169.26.1 |
| Proxy mode | Transparent (port 3128) |

## Activitats realitzades

### 1. Configuració inicial del proxy

- Activació del proxy a la interfície GREEN
- Mode transparent activat
- Port proxy: 800
- Port transparent: 3128

### 2. Instal·lació de llistes negres

- Actualització automàtica activada (periòdicament)
- Origen: base de dades oficial d'IPFire
- Permet bloquejar per categories predefinides

### 3. Bloqueig per categories

Categories bloquejades:
- **bank** (ex: ing.es)
- **radio** (ex: ah.fm)

**Resultat:** Accés denegat als llocs pertanyents a aquestes categories.

### 4. Bloqueig de dominis concrets

Dominis bloquejats a la llista negra personalitzada:
- elnacional.cat
- tecnocampus.cat

**Resultat:** Accés completament denegat a aquests dominis.

### 5. Bloqueig de URL concreta

URL bloquejada específicament:
- https://www.adidas.es/

**Resultat:** La URL específica queda bloquejada (altres pàgines del domini també queden afectades per coincidència parcial).

### 6. Bloqueig per paraula clau amb excepció

**Configuració:**
- Paraula bloquejada: "anime"
- Excepció (llista blanca): animenewsnetwork.com

**Resultats:**
| Lloc | Resultat |
|------|----------|
| animejuegos.com | ❌ Bloquejat |
| animenetwork.com | ❌ Bloquejat |
| animenewsnetwork.com | ✅ Permès |

> *Nota: Cal activar la llista blanca personalitzada perquè l'excepció funcioni.*

### 7. Bloqueig per franges horàries

**Regla de firewall:**
- Origen: GREEN (xarxa interna)
- Destí: RED (Internet)
- Dies: dimarts, dijous, divendres
- Franja horària: 12:45 - 13:45

**Resultat:** Fora d'aquest horari, la navegació queda bloquejada.


## Conclusions

El proxy Squid d'IPFire permet un control d'accés flexible mitjançant diferents estratègies:
- **Categories:** ideal per bloquejos generals per temàtica
- **Dominis/URLs:** precís per a llocs concrets
- **Paraules clau:** útil per bloquejar contingut específic
- **Horaris:** control temporal d'accés

Autor: Miquel Vico