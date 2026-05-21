# Guia de configuració del proxy Squid a IPFire

## Descripció de la tasca

En aquesta pràctica hem configurat el servei de proxy Squid a IPFire per controlar l'accés a Internet des de la xarxa local. Les activitats realitzades inclouen la instal·lació de llistes negres, el bloqueig per categories, per dominis, per URLs concretes, per paraules clau (amb excepcions) i finalment per franges horàries.

---

## 1. Configuració inicial del proxy

Abans de començar amb els filtres, hem activat el proxy a la interfície GREEN (xarxa interna) i hem configurat el mode transparent perquè els clients no necessitin configurar res manualment.

![Configuració del proxy](/tascaIPfire/imgIPfire/c12.png)

**Configuració aplicada:**
- **Activado en Green:** marcat (proxy actiu a la xarxa interna)
- **Transparente en Green:** marcat (intercepta el trànsit automàticament)
- **Puerto del proxy:** 800
- **Puerto transparente:** 3128

> **Anàlisi:** El mode transparent simplifica molt la configuració dels clients, ja que no caldrà posar manualment les dades del proxy a cada navegador. El port 800 és per als clients que vulguin configurar-lo manualment, mentre que el 3128 serveix per a la redirecció automàtica.

---

## 2. Verificació de la configuració de xarxa

Per assegurar que el proxy funcioni correctament, hem comprovat les interfícies de xarxa:

![Interfície GREEN](/tascaIPfire/imgIPfire/c5.png)

![Interfície RED](/tascaIPfire/imgIPfire/c4.png)

**Dades de configuració:**
- **GREEN (xarxa interna):** 192.169.26.254/24
- **RED (xarxa externa):** obté IP per DHCP (10.0.2.8)

També hem verificat que el client està a la mateixa subxarxa:

![Client xarxa](/tascaIPfire/imgIPfire/c7.png)

**Client:**
- Adreça IP: 192.169.26.1
- Porta d'enllaç: 192.169.26.254 (el nostre IPFire)

> **Anàlisi:** El client està correctament a la xarxa interna (192.169.26.0/24) i el gateway apunta a l'IPFire. Això permet que el proxy transparent pugui interceptar tot el trànsit de navegació.

---

## 3. Instal·lació de les llistes negres

Les llistes negres contenen dominis organitzats per categories. Sense elles, no podem bloquejar per temàtiques com "bank" o "radio".

![Instal·lació llistes negres](/tascaIPfire/imgIPfire/c10.png)

![Origen de llistes](/tascaIPfire/imgIPfire/c9.png)

**Configuració:**
- **Activar actualització automàtica:** marcat
- **Planificació:** mensualment
- **Origen de descàrrega:** IPFire DBL (Database Library)

> **Anàlisi:** Hem triat la base de dades oficial d'IPFire per assegurar que les categories estiguin actualitzades. L'actualització automàtica mensual garanteix que els nous dominis maliciosos o no desitjats s'incorporin periòdicament.

---

## 4. Configuració del proxy al client

Com que hem activat el mode transparent, el client no hauria de necessitar configuració. Tot i això, per confirmar que funciona, podem comprovar que el proxy està actiu:

![Configuració manual client](/tascaIPfire/imgIPfire/c13.png)

**Configuració manual (opcional):**
- **HTTP Proxy URL:** 192.169.26.254
- **Port:** 800

> **Anàlisi:** Si el mode transparent no funcionés per algun motiu, podem configurar manualment el client. En aquesta pràctica, el mode transparent ha funcionat correctament.

---

## 5. Bloqueig per categories: bank i radio

Un cop instal·lades les llistes negres, vam bloquejar les categories "bank" i "radio" i vam provar amb ing.es i ah.fm.

![Categories bloquejades](/tascaIPfire/imgIPfire/c11.png)

**Categories marcades:**
- bank: ☑
- radio: ☑

### Prova amb ing.es (categoria bank)

![Error ing.es](/tascaIPfire/imgIPfire/c14.png)

**Resultat:** `ERR_TUNNEL_CONNECTION_FAILED` - La connexió ha estat bloquejada pel proxy.

### Prova amb ah.fm (categoria radio)

> *(Nota: no es disposa de captura de ah.fm, però el comportament és similar a ing.es)*

**Resultat:** També queda bloquejat per pertànyer a la categoria "radio".

> **Anàlisi:** El bloqueig per categories funciona correctament, però cal tenir en compte que depèn de les llistes negres descarregades. Si algun domini no apareix a la base de dades, no es bloquejarà automàticament.

---

## 6. Bloqueig de dominis concrets

Vam bloquejar els dominis `elnacional.cat` i `tecnocampus.cat` mitjançant la llista negra personalitzada.

![Dominis bloquejats](/tascaIPfire/imgIPfire/c15.png)

**Configuració:**
- **Activar Lista Negra personalizada:** ☑
- **Dominis bloquejats:**
  - elnacional.cat
  - tecnocampus.cat

### Prova amb tecnocampus.cat

![Error tecnocampus](/tascaIPfire/imgIPfire/c17.png)

**Missatge:** `ACCESS DENIED` - L'accés ha estat denegat pel proxy.

> **Anàlisi:** Aquest mètode és molt útil per bloquejar llocs específics que no estiguin a les categories predefinides o per fer bloquejos més fins.

---

## 7. Bloqueig d'una URL concreta d'un domini

Vam provar de bloquejar només una pàgina específica d'un domini, permetent la resta del lloc.

![URL concreta bloquejada](/tascaIPfire/imgIPfire/c18.png)

**Configuració:**
- **URLs bloquejada:** https://www.adidas.es/

### Prova amb la URL bloquejada

![Error adidas](/tascaIPfire/imgIPfire/c19.png)

**Missatge:** `The requested URL could not be retrieved`

> **Anàlisi:** Hem bloquejat específicament adidas.es. Si provéssim una altra pàgina del mateix domini (per exemple, adidas.es/outlet) també quedaria bloquejada perquè la coincidència és parcial. Per bloquejar només una URL molt concreta, caldria ser més específic amb el path complet.

---

## 8. Bloqueig per paraula clau amb excepció

L'activitat més complexa va ser bloquejar totes les pàgines que continguessin la paraula "anime", excepte el domini `animenewsnetwork.com`.

![Frases bloquejades i llista blanca](/tascaIPfire/imgIPfire/c20.png)

**Configuració:**
- **Frases bloquejades:** anime
- **Activar lista de frases personalizada:** ☑
- **Dominis permesos (llista blanca):** animenewsnetwork.com
- **Activar Lista Blanca personalizada:** ☐ (NO activat)

### Prova amb animejuegos.com (hauria de bloquejar)

![Error animejuegos](/tascaIPfire/imgIPfire/c22.png)

**Missatge:** `Unable to determine IP address` - Tot i que aquest error és de DNS, el proxy hauria d'haver bloquejat la petició.

### Prova amb animenetwork.com (hauria de bloquejar)

![Error animenetwork](/tascaIPfire/imgIPfire/c23.png)

**Missatge:** `ACCESS DENIED` - Correcte, s'ha bloquejat per contenir "anime" a l'URL.

### Prova amb animenewsnetwork.com (hauria de permetre)

![Animenewsnetwork permès](/tascaIPfire/imgIPfire/c21.png)

**Resultat:** La pàgina carrega correctament perquè està a la llista blanca.

> **Anàlisi:** Perquè l'excepció funcioni correctament, cal activar la **Lista Blanca personalitzada**. En la captura està desactivada (☐), per això l'excepció no s'aplica. Un cop activada, els dominis de la llista blanca prevalen sobre les frases bloquejades.

---

## 9. Bloqueig per franges horàries

Finalment, vam configurar una regla al firewall per restringir l'accés a Internet en un horari concret.

![Regles de firewall](/tascaIPfire/imgIPfire/c26.png)

**Configuració de la regla:**
- **Origen:** Green (xarxa interna)
- **Destí:** RED (Internet)
- **Protocol:** TCP
- **Acció:** Permet (però amb restricció horària)
- **Dies:** Mar, Jue, Vie
- **Horari:** 12:45 - 13:45

![Configuració horària](/tascaIPfire/imgIPfire/c25.png)

**Paràmetres:**
- Usar restriccions de temps: ☑
- Dies seleccionats: Dilluns? (segons configuració)
- Franja horària: 12:45 a 13:45

> **Anàlisi:** Aquesta regla permet la navegació només en l'horari especificat. Fora d'aquest horari, el trànsit cap a Internet queda bloquejat. És una eina molt útil per controlar l'ús d'Internet en entorns educatius.

---

## Conclusions

El proxy Squid d'IPFire és una eina potent i flexible per controlar l'accés a Internet. Amb les llistes negres podem bloquejar per categories senceres, però també podem fer bloqueigs individuals per domini o per paraules clau. El mode transparent simplifica molt la implementació, i les regles horàries del firewall complementen el control d'accés.

**Recomanacions finals:**
- Mantenir actualitzades les llistes negres periòdicament
- Per bloquejos precisos, combinar categories, dominis i paraules clau
- En cas d'excepcions a paraules clau, recordar activar la llista blanca