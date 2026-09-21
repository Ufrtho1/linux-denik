# 21. 9. 2026 — Day 3: Power trip!

## Úkoly dne
- [x] Hostname — `sudo hostnamectl set-hostname linuxlab`, pak `bash`
- [x] Časové pásmo — `timedatectl` ukázal Europe/Prague, nic neměnit
- [–] Heslo — záměrně přeskočeno. Na lokální VM zbytečné.
      `passwd` hodil "Authentication token manipulation error",
      protože uživatel ubuntu na Multipassu žádné heslo nemá.
      Na VPS ho dostanu při `adduser`.

SSH v Day 3 vlastně není, jen v extensions. Skutečné bude až na VPS.

---

## Tři typy uživatelů

| Kdo | Co smí |
|---|---|
| `root` | úplně všechno, globální změny i změny za kohokoliv |
| sudoers | běžní uživatelé, kteří smí používat `sudo` |
| běžný uživatel | jen své soubory a své nastavení |

**Globální změna** ovlivní všechny uživatele. **Lokální** jen jednoho.
Podrobně Day 13 a 14.

Na root se nepřihlašuje — pracuje se jako běžný uživatel
a práva se zvedají přes `sudo`.

---

## sudo

`sudo` = super user do. Provede jeden příkaz jako root.

**Test na `/etc/shadow`:**
- `ls -l /etc/shadow` → `-rw-r----- 1 root shadow` — soubor vidím,
  patří rootovi, ostatní ho nesmí číst
- `less /etc/shadow` → Permission denied
- `sudo less /etc/shadow` → přečtu ho

`/etc/shadow` = uložená zahashovaná hesla. Hlavní cíl útočníků —
ukradnou ho a hesla lámou offline. Proto ho běžný uživatel nesmí číst.

Na Multipassu `sudo` nechce heslo — normálně chce, ale dá se to vypnout
v konfiguraci sudoers. U mě vypnuté je.

---

## sudo -i

Nejde o jeden příkaz — **zůstanu** rootem, dokud neodejdu.

- `sudo příkaz` — jeden příkaz jako root
- `sudo -i` — přepne mě do relace roota

Poznám to na promptu: `ubuntu@...:~$` → `root@...:~#`
Ověřím přes `whoami` → `root`

Ven: `exit`, `logout` nebo Ctrl+D

**Pravidlo: vejdi, udělej, odejdi.** Jako root nemám žádnou pojistku.

Není to vyšší **priorita**, ale vyšší **práva**. Priorita v Linuxu znamená,
kolik procesorového času dostane proces — sloupce PR a NI v `top`.

---

## reboot

- `reboot` → "requires interactive authentication"
- `sudo reboot` → restartuje celý stroj, ne jen spojení

Odpojilo mě to proto, že stroj, na kterém moje relace běžela,
na chvíli přestal existovat.

Z Macu přes `multipass list` jsem viděl stav **Restarting**, pak **Running**.
Po `multipass stop` stav **Stopped**.
Po `sudo reboot` stroj naběhne sám — `multipass start` netřeba,
stačí `multipass shell server1`. `start` je jen na vypnutý stroj.

Ověření: `uptime` po přihlášení ukáže 0 minut.

**Na skutečném serveru = výpadek.** Všechno na něm je tu minutu offline.
Proto se to ohlásí a naplánuje. Restartuje se hlavně po aktualizaci
jádra — nové jádro běží až po restartu.

---

## Logy — /var/log/auth.log

Log přihlášení a použití `sudo`.

    less /var/log/auth.log
    grep sudo /var/log/auth.log

`grep` = najdi řádky, které obsahují slovo. Nejdřív **co**, pak **kde**.

Vypsalo **všechno, co jsem kdy přes sudo spustil** — kdo, kdy, odkud a co.
Včetně překlepů: `apt install net tools`, `iftop -i eth0`.
Log si pamatuje všechno. Tohle hlídá bezpečák.

---

## Administrativní úkoly

**`hostnamectl`** — informace o stroji a jeho jméno.
Ukazuje i že jde o VM (`Virtualization: qemu`, `Chassis: vm`).

    sudo hostnamectl set-hostname linuxlab

Po změně se prompt nepřepsal — pořád `server1`.
Po `bash` ukázal `linuxlab`. Proč, viz sekce bash níž.

**`timedatectl`** — datum, čas, časové pásmo

    timedatectl list-timezones
    sudo timedatectl set-timezone Europe/Prague

Časové pásmo ovlivňuje hlavně naplánované úlohy a časy v logách.

---

## bash

**Bash je shell — program, se kterým celou dobu mluvím.**
Každý příkaz píšu do bashe. On ho přečte, najde program, spustí ho
a ukáže výsledek. Prompt kreslí bash.

Napsat `bash` = spustit novou relaci uvnitř té současné.
Relace se vnořují jako `sudo -i` — každá potřebuje svůj `exit`.

**Proč `bash` po změně hostname:** bash si jméno stroje přečte,
když startuje, a pak ho kreslí pořád stejné. Změna jména to starému bashi
neřekne. Nový bash si načte aktuální jméno.
Ověřeno — viz sekce Vnořené bashe na konci.

**K čemu mi to bude:** Day 20 = skripty v bashi. Automatizace
vlastní práce — v NOC nejrychlejší cesta nahoru.

---

## Další pojmy

**`less`** — prohlížeč textu po stránkách. Šipky posouvají, `q` ven.
Místo `cat`, když je soubor dlouhý. Samotné `less` bez souboru nejde —
"Missing filename". Pořádně Day 5.

**`sudoers`** — není příkaz, je to soubor `/etc/sudoers`.
Určuje, kdo smí `sudo`. Upravuje se přes `visudo`. Day 13.

**`sudo-rs`** — sudo přepsané do jazyka Rust. Ubuntu 26.04 ho používá
jako `sudo`, proto se jeho nápověda hlásí jako sudo.

---

## Co jsem zjistil sám

**`multipass list` uvnitř Ubuntu → command not found.**
Spustil jsem příkaz pro Mac uvnitř VM. Jsem v jiném počítači
a ten o Multipassu nic neví. Vrstvy z Day 1 v praxi.

**`less var/log/auth.log` → No such file. `less /var/log/auth.log` → funguje.**
Bez lomítka hledal v /home/ubuntu/var/log. Absolutní vs relativní cesta z Day 2.

**`sudo -i` dvakrát = root v rootovi.** Musel jsem dvakrát ven.
V auth.log dvě otevřené a dvě zavřené relace.

**Překlepy systém opraví:** `timedatectl list-timezoes` →
"did you mean list-timezones?". `hostnamctl` → nabídl hostnamectl.

---

## Co jsem si pletl
- `grep /var/log/auth.log` — chybí co hledat. Správně `grep <co> <kde>`.
- `ls -l /etc/shadow` nic nevyhledává — vypíše podrobnosti o souboru.
- `sudo -i` jsem nazval vyšší prioritou. Jsou to vyšší práva.
- `sudoers` jsem zkoušel spustit jako příkaz. Je to soubor.
- Bash mi připadal jako zbytečnost. Je to program, do kterého píšu
  všechno — jen ten příkaz `bash` je drobnost.
- Zkoušel jsem `multipass stop` uvnitř Linuxu. Multipass je jen na Macu.
- Myslel jsem, že po změně hostname se virtuálka jmenuje `linuxlab`
  i v Multipassu. Nejmenuje — pro Multipass je pořád `server1`.

---

## Kde jsem — jak to poznat z promptu

| Prompt | Kde jsem |
|---|---|
| `jaroslavkopecky@Jaroslav--MacBook-Pro ~ %` | na Macu |
| `ubuntu@linuxlab:~$` | v Linuxu |
| `root@linuxlab:~#` | v Linuxu jako root |

**`%` a jméno MacBooku = Mac. `$` nebo `#` a `ubuntu@` / `root@` = Linux.**

Příkaz `multipass` existuje jen na Macu. Uvnitř Linuxu hodí
"command not found" — Linux o Multipassu nic neví.
Tohle se mi stalo dvakrát, takže: **nejdřív přečíst prompt, pak psát.**

---

## Dvě jména, dvě úrovně

Po změně hostname má jeden stroj dvě jména:

- **`server1`** — jméno virtuálky v Multipassu. Tak ji zná Mac.
  Používám ho v `multipass start/stop/shell server1`.
- **`linuxlab`** — hostname uvnitř Linuxu. Tak se systém představuje
  sám sobě a vidím ho v promptu.

Změna hostname uvnitř Linuxu **nepřejmenuje** virtuálku v Multipassu.
Proto `multipass stop linuxlab` nefunguje ani na Macu — Multipass
žádné `linuxlab` nezná.

Zase vrstvy z Day 1: každá úroveň má svoje jméno pro tu samou věc.

---

## Vnořené bashe — ověřeno

Postup a co se dělo:

1. Přihlášení → bash č. 1, prompt `server1`
2. `bash` → bash č. 2, prompt pořád `server1`
3. `sudo hostnamectl set-hostname linuxlab`
4. `bash` → bash č. 3, prompt `linuxlab`
5. `exit` → zpátky v bashi č. 2, prompt **zase `server1`**
6. `exit` → bash č. 1, prompt `server1`
7. `exit` → konečně na Macu

Krok 5 je důkaz: systém se jmenuje `linuxlab`, ale starý bash
si při startu zapamatoval `server1` a pořád ho kreslí.

**Každý `bash` i `sudo -i` je jedna vrstva a každá potřebuje svůj `exit`.**
Když nevím, v kolikáté jsem, podívám se na prompt.

---

## Otázky do příště
-
