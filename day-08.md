# 29. 9. 2026 — Day 7: Server a služby (Apache, systemctl)

Ráno nejdřív incident s místem na disku — viz samostatný zápis.

## Instalace webserveru

    sudo apt install apache2      (napoprvé překlep apsache2 → "Unable to locate package")

Po instalaci se Apache **sám spustí** a nastaví se, aby startoval i po restartu.
Ve výpisu apt to je vidět:

    Created symlink '/etc/systemd/system/multi-user.target.wants/apache2.service'

## Ověření, že běží — zevnitř serveru

    curl localhost        vysype HTML výchozí stránky
    curl -I localhost     jen hlavička odpovědi

- `curl` = stáhne stránku a vypíše, co server vrátil (prohlížeč bez obrázků)
- `localhost` = tenhle stroj sám (127.0.0.1, rozhraní `lo`)
- **`HTTP/1.1 200 OK`** = stránka je, všechno v pořádku
- `Connection refused` by znamenalo, že nikdo neposlouchá

**Postup po vrstvách:** nejdřív ověřit službu zevnitř (curl), teprve pak síť
k ní (prohlížeč z Macu). Když curl funguje a prohlížeč ne, problém je po cestě,
ne v Apachi.

Z Macu v prohlížeči: `http://192.168.252.2`

## systemd a systemctl

**Služba** = program, který běží na pozadí trvale (webserver, SSH…).
**systemd** = správce, který služby spouští, hlídá a zastavuje.
**systemctl** = příkaz, kterým systemd ovládám.

    sudo systemctl status apache2     stav
    sudo systemctl stop apache2       zastavit
    sudo systemctl start apache2      spustit
    sudo systemctl restart apache2    zastavit a znovu spustit
    sudo systemctl enable apache2     startovat po bootu
    sudo systemctl disable apache2    nestartovat po bootu

**Starší systém:** SysVinit, skripty v `/etc/init.d/`. Dnes skoro všude systemd,
ale na starých serverech se s init.d dá potkat.

## Jak číst `systemctl status`

    Loaded: loaded (...; enabled; ...)      enabled = spustí se po bootu
    Active: active (running) since ...      běží teď, a od kdy
    Main PID: 2020 (apache2)                číslo hlavního procesu
    Memory: 39.6M                           kolik zabírá paměti
    CGroup: ├─2020 ... ├─2037 ... └─2038    hlavní proces + pomocníci

Dole posledních pár řádků logu služby.

**enabled ≠ active:**
- `enabled` = co se stane **po restartu**
- `active` = co se děje **teď**

Po `stop` bylo `inactive (dead)`, ale pořád `enabled` — po bootu by
se Apache zase spustil.

Varování `AH00558: Could not reliably determine the server's fully qualified
domain name` je neškodné — Apache jen neví, jak se server jmenuje na síti.
Běží normálně.

## SSH jako služba

    sudo systemctl status sshd

    Server listening on 0.0.0.0 port 22
    Accepted publickey for ubuntu from 192.168.252.1

SSH je taky služba, poslouchá na portu 22. Řádek `Accepted publickey`
je můj Mac, jak se přes Multipass přihlašuje klíčem.

## Kde co u Apache leží

| Co | Kde |
|---|---|
| hlavní konfigurace | `/etc/apache2/apache2.conf` |
| web, který se zobrazuje | `/var/www/html/index.html` |
| logy | `/var/log/apache2/` |

Oba soubory patří rootovi → **upravovat přes `sudo vim`**.
Bez sudo se otevřou, ale nejdou uložit.

## Co nešlo
- `apsache2`, `localhos` — překlepy. Chyba vždy řekne, co nenašla.
- `vim /etc/apache2/apache2.conf` bez sudo → opraveno přes `sudo !!`
- `vim /var/www/html/index.html` bez sudo → nešlo by uložit

## Odkaz na lekci
https://linuxupskillchallenge.org/07/

## Úkol navíc
Přepsat výchozí stránku:
    sudo vim /var/www/html/index.html
Smazat obsah, napsat vlastní text, uložit a načíst `http://192.168.252.2` na Macu.
