# Viikko 2 – SNMP

## 1. Johdanto

Tässä tehtävässä tutustuttiin SNMP-protokollaan ja sen käyttöön verkon laitteiden valvonnassa. SNMP:n avulla voidaan kerätä laitteista erilaisia tietoja verkon kautta. Tehtävässä asennettiin SNMP-agentti palvelimelle ja tehtiin SNMP-kyselyitä Ansible-koneelta. Lopuksi SNMP-agentti asennettiin useammalle laitteelle ja niiden tietoja vertailtiin.


## 2. Asennus

### SNMP-agentin asennus

SNMP-agentti asennettiin web1-palvelimelle. Ensin päivitettiin pakettilista ja sen jälkeen asennettiin SNMP sekä SNMP-agentti, eli snmpd.

```bash
apt update
apt install snmp snmpd -y
```
Seuraavaksi tuli konfiguroida /etc/snmp/snmpd.conf -tiedostoa:

```bash
nano /etc/snmp/snmpd.conf
```
Tarkastettiin, että sisällöstä löytyi rocommunity public. Seuraavaksi lisättiin systemonly-näkymään seuraava OID-haara:

```bash
view systemonly included .1.3.6.1.2
```
Tämä laajentaa SNMP:n kautta saatavilla olevia tietoja.
Muutosten jälkeen SNMP-palvelu käynnistettiin uudelleen, jotta juuri tekemämme OID-laajennus saatiin käyttöön:

```bash
service snmpd restart
```

## 3. Kerätyt tiedot


### SNMP-kyselyt
```bash
root@ansible:/# snmpget -v2c -c public web1 sysName.0
SNMPv2-MIB::sysName.0 = STRING: web1
```

```bash
root@ansible:/# snmpget -v2c -c public web1 sysDescr.0
SNMPv2-MIB::sysDescr.0 = STRING: Linux web1 6.18.33.2-microsoft-standard-WSL2 #1 SMP PREEMPT_DYNAMIC Thu Jun 18 21:54:43 UTC 2026 x86_64
```

```bash
root@ansible:/# snmpget -v2c -c public web1 sysUpTime.0
DISMAN-EVENT-MIB::sysUpTimeInstance = Timeticks: (246082) 0:41:00.82
```


## 4. Verkkorajapinnat

Web1-palvelimen verkkorajapinnat selvitettiin SNMP:n avulla:

```bash
root@ansible:/# snmpwalk -v2c -c public web1 ifDescr
IF-MIB::ifDescr.1 = STRING: lo
IF-MIB::ifDescr.61 = STRING: eth0
IF-MIB::ifDescr.90 = STRING: eth1
```
SNMP-kyselyn perusteella web1-palvelimella on kolme verkkorajapintaa: lo, eth0 ja eth1. lo on loopback-rajapinta, eth0 on Containerlabin hallintaverkon rajapinta ja eth1 on Server LAN -verkon rajapinta.



## 5. OID-analyysi


| OID | Tarkoitus |
|---|---|
| `sysName.0` | Järjestelmän nimi |
| `sysDescr.0` | Järjestelmän kuvaus, esimerkiksi käyttöjärjestelmä ja versiotietoja |
| `sysUpTime.0` | Aika, jonka SNMP-agentti on ollut käynnissä |
| `ifDescr` | Verkkorajapintojen nimet ja kuvaukset |
| `ifOperStatus` | Verkkorajapinnan toimintatila, esimerkiksi up tai down |

## 6. Usean laitteen valvonta

| Laite | Nimi | Käyttöjärjestelmä | Uptime |
|---|---|---|---|
| web1 | web1 | Linux |  2:21:55.11 |
| db1 | db1 | Linux |  0:06:58.70 |
| branch-client | branch-client | Linux |  0:10:35.98 |

## 7. Pohdinta

Tehtävässä SNMP:n perusidea tuli mielestäni hyvin esille. SNMP-agentti toimii valvottavalla laitteella ja siltä voidaan kysyä tietoja toiselta koneelta verkon kautta. Aluksi SNMP:n toimintaan saaminen oli melko hankalaa, koska agentti kuunteli vain localhost-osoitteessa eikä siihen saanut yhteyttä toiselta koneelta. Myös MIB-tiedostojen käyttöönotossa oli omat vaiheensa.

Kun asetukset saatiin kuntoon, SNMP-kyselyiden tekeminen oli melko yksinkertaista. Niiden avulla saatiin helposti selville esimerkiksi laitteen nimi, järjestelmätietoja, käyttöaika ja verkkorajapinnat. Useamman laitteen valvonnassa SNMP:n hyöty tuli myös paremmin esille, koska samalla tavalla voidaan kerätä tietoja useista eri laitteista.

Tehtävässä käytetty tekoälyä mm. ongelmatilanteiden selvittelyssä.