# 6. 10. 2026 — Day 9: Porty a firewall

## Představa: panelák

- **IP adresa** = adresa domu (u mě 192.168.252.2)
- **Port** = číslo bytu. Na jedné adrese bydlí víc aplikací, každá ve svém bytě.
- **Firewall** = vrátný, který rozhoduje, koho k jakému bytu pustí

Běžné porty:

| Port | Služba |
|---|---|
| 22 | SSH |
| 23 | Telnet (starý, nešifrovaný — nepoužívat) |
| 53 | DNS |
| 80 | HTTP |
| 443 | HTTPS |

Celý seznam je v `/etc/services`.

## Kdo poslouchá na jakém portu

    netstat           všechna spojení
    netstat -l        jen porty, které poslouchají (l = listening)
    ss -ltp           novější náhrada: l = listening, t = TCP, p = proces
    sudo ss -ltp      i s názvem programu — bez sudo se proces neukáže

Co u mě poslouchá:

| Adresa:port | Co to je |
|---|---|
| 0.0.0.0:22 | SSH — poslouchá na všech adresách |
| *:80 | Apache |
| 127.0.0.53:53 a 127.0.0.54:53 | místní DNS (systemd-resolved), jen uvnitř stroje |

`0.0.0.0` nebo `*` = přístupné zvenku. `127.x.x.x` = jen zevnitř stroje.

V `netstat` byla dvě navázaná spojení `ssh` od `_gateway` — to je můj Mac
přes Multipass.

## nmap — pohled zvenku

    sudo apt install nmap
    nmap localhost

Výsledek: otevřené 22 (ssh) a 80 (http), 998 portů zavřených.

`ss` ukazuje, co poslouchá **zevnitř**. `nmap` zkouší porty **zvenku**,
jako by to udělal útočník. Na veřejném serveru se má kontrolovat obojí.

## ufw — firewall

    sudo ufw status          stav
    sudo ufw allow ssh       povolit SSH
    sudo ufw allow http      povolit web (port 80)
    sudo ufw deny http       zakázat web
    sudo ufw enable          zapnout firewall
    sudo ufw disable         vypnout firewall

**ZLATÉ PRAVIDLO: nejdřív `sudo ufw allow ssh`, teprve potom `sudo ufw enable`.**
Po zapnutí firewall blokuje všechno příchozí, co není povolené. Bez pravidla
pro SSH se na vzdálený server už nepřihlásím — zamknu se venku.
Proto při `enable` vyskočilo varování „may disrupt existing ssh connections".

Port 80 jsem blokoval jen pro ukázku. Na webserveru se nikdy neblokuje.

**Jak ověřit, že firewall opravdu blokuje:** z Macu v prohlížeči
`http://192.168.252.2`. `curl localhost` uvnitř serveru to neukáže —
provoz uvnitř stroje (loopback) firewall propouští.

## Co se povedlo navíc
Přepsal jsem výchozí stránku Apache na vlastní (`/var/www/html/index.html`,
přes `sudo vim`) → `curl localhost` vrací „my little box".

## Co nešlo
- `sudo ufw enable http` → Invalid syntax. `enable` nebere žádný argument,
  zapíná firewall celý.
- `curl --localhost`, `curl -localhost` → nesmysl. Co začíná pomlčkou, je
  volba. Adresa se píše bez pomlček: `curl localhost`.
- `apace2` — překlep.

## Pozor na disk
`/` je na 85 %. Zvětšit disk virtuálky (multipass set ... disk=15G),
než budu instalovat další věci.

## Odkaz na lekci
https://linuxupskillchallenge.org/09/

## Otázky do příště
-
