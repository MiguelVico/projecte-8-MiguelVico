# README: NAT i VPN amb IPFire

Aquest repositori conté la resolució completa de l’activitat **UD9.AA4 – NAT i VPN** del curs *Serveis de xarxa* (CFGM SMX). L’objectiu és aprendre a exposar serveis interns a Internet mitjançant **Destination NAT (DNAT)** i a connectar-s’hi de forma segura amb una **VPN (OpenVPN)**.

## 📝 Descripció de la tasca

Disposem de tres màquines virtuals:
- **Zorin** – servidor intern amb SSH i Apache.
- **IPFire** – tallafoc / router (distribució Linux especialitzada).
- **Client Windows** – simula un equip extern a Internet.

Els serveis interns del Zorin s’exposen de dues maneres:
1. **DNAT** (port forwarding) – accessible des de l’exterior usant la IP pública de l’IPFire.
2. **OpenVPN** – el client es connecta a la xarxa interna i accedeix als serveis amb la IP privada del Zorin.

## 🧰 Tecnologies utilitzades

| Tecnologia | Descripció |
|------------|-------------|
| **VirtualBox** | Entorn de virtualització per a les tres VM. |
| **Ubuntu / Zorin OS** | Sistema operatiu del servidor intern. |
| **IPFire** | Distribució Linux per a tallafoc i encaminador. |
| **OpenSSH** | Servei d’accés remot per terminal. |
| **Apache2** | Servidor web. |
| **OpenVPN** | Tecnologia VPN de codi obert. |
| **Windows 11** | Client extern. |

## 📂 Estructura del repositori

```
/
├── README.md                 # Aquest fitxer
├── guia.md                   # Guia pas a pas amb totes les captures i explicacions
└── imgVPN/                   # Carpeta amb les 24 captures (c1.png ... c24.png)
```

## 🚀 Com reproduir l’activitat

1. **Prepara l’escenari**  
   - Crea tres VM amb VirtualBox: Zorin (xarxa interna), IPFire (eth0 NAT, eth1 interna), Client Windows (xarxa NAT).  
   - La xarxa interna ha de ser `192.169.x.0/24`.

2. **Configura els serveis al Zorin**  
   - Instal·la `openssh-server` i `apache2`.  
   - Personalitza la pàgina web (`/var/www/html/index.html`).

3. **Configura el DNAT a IPFire**  
   - Accedeix a la interfície web (`https://ipfire:444`).  
   - Crea regles de Firewall > NAT de destinació per als ports 22 (SSH) i 80 (HTTP) cap al Zorin.

4. **Configura la VPN a IPFire**  
   - Ves a *Servicios > OpenVPN*.  
   - Crea una connexió *Host-to-Net* (Roadwarrior).  
   - Genera certificats per a l’usuari.  
   - Descarrega els fitxers `.ovpn` i `.p12`.

5. **Configura el client Windows**  
   - Edita `C:\Windows\System32\drivers\etc\hosts` per resoldre `ipfire.foodlogistic.test`.  
   - Instal·la OpenVPN Community Edition.  
   - Importa el perfil `.ovpn` i connecta’t.

6. **Comprova**  
   - Des del client, accedeix a `http://10.0.2.8` (DNAT) i a `http://192.169.26.1` (via VPN).

## 📸 Captures de pantalla

Totes les captures numerades (`c1.png` ... `c24.png`) es troben dins de la carpeta [`imgVPN/`](./imgVPN). A la [guia detallada](./guia.md) s’explica cada pas amb la seva imatge corresponent.


> **Nota:** Aquesta activitat forma part del mòdul professional *Serveis de xarxa* (CFGM SMX). Si tens dubtes, consulta el teu professor o la [wiki d’IPFire](https://wiki.ipfire.org).
