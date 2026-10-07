# 22. 9. 2026 — Vlastní průzkum (mimo kurz)

Den mimo osnovu. Neučil jsem se nové lekce, procházel poznámky z Day 1–3
a ověřoval si, co jsem si zapsal.

---

## Disk vs paměť — rozdíl, co jsem si pletl

Server nemá obrazovku. `df -h` a `free -h` jsou dvě různé otázky.

| | Disk (`df -h`) | RAM (`free -h`) |
|---|---|---|
| K čemu | soubory natrvalo | co běžící programy právě používají |
| Po restartu | zůstane | vymaže se |
| Velikost u mě | 3,9 GB | 943 MB |
| Rychlost | pomalejší | velmi rychlá |

- **Plný disk** → nejde nic zapsat, služby padají. Nejčastější příčina výpadků,
  protože logy tiše rostou.
- **Plná RAM** → zpomalení, pak jádro zabije nějaký proces (OOM killer).
  V `top` je vidět proces `oom_reaper` — čeká, kdyby bylo potřeba.

**Past u `free -h`:** sloupec `free` ukazuje jen 167 Mi a vypadá to, že paměť
došla. Nedošla. Linux volnou paměť půjčuje jako cache a vrátí ji, jakmile ji
někdo potřebuje. **Koukat na `available`** — u mě 629 Mi.

`df` = disk free, `du` = disk usage.

---

## `df -h` — co z toho číst

Jediný řádek, co mě zajímá:

    /dev/sda1    3.9G  2.6G  1.3G  68%  /

- `/dev/sda1` — `/dev` = adresář zařízení, `sd` = disk, `a` = první, `1` = první oddíl
- `/` ve sloupci Mounted on — je připojený jako kořen, takže **všechno**
  (`/etc`, `/home`, `/bin`) leží fyzicky tady
- 68 % zabráno

Rychlejší: `df -h /` vypíše jen tenhle řádek.

**Ostatní řádky ignorovat.** `tmpfs` a `none` nejsou disky, ale kousky RAM,
které se tváří jako adresář. Proto je `/tmp` a `/dev/shm` po restartu prázdné.
Disk versus RAM v jednom výpisu vedle sebe.

Zbylé skutečné: `/dev/sda13` = `/boot` (jádro), `/dev/sda15` = `/boot/efi` (start).

**68 % roste a samo neklesá.** Na Day 1 to bylo 64 %. Přibyl tldr, net-tools,
iftop. Přesně takhle se v praxi zaplňují servery.

---

## Kolik má virtuálka a proč tak málo

Multipass při zakládání uřízl z disku Macu kus a ten je pro Ubuntu celý svět.
Ubuntu o zbytku notebooku neví — z jeho pohledu sedí na stroji
s 5GB diskem, 1GB RAM a jedním procesorem (`lsblk`, `lscpu`).

Místo se na Macu nezabírá naráz, ale postupně podle skutečného využití.
**Co jednou zabere, to už Macu nevrátí**, ani když v Ubuntu soubory smažu.
Proto se to jednou za čas řeší tím, že se virtuálka zahodí a postaví znovu —
a proto se servery staví tak, aby šly postavit znovu podle poznámek.

Zvenčí z Macu: `multipass info server1` → přidělené i využité.

---

## `top` — pět řádků hlavičky

| Řádek | Co ukazuje |
|---|---|
| `top - ... up ... 2 users, load average:` | čas, jak dlouho běží, relace, vytížení |
| `Tasks: 101 total, 1 running, 100 sleeping` | procesy |
| `%Cpu(s): ... 99.7 id ...` | procesor |
| `MiB Mem: 943.1 total ... 541.9 buff/cache` | paměť — totéž co `free -h` |
| `MiB Swap: 0.0 total ... 624.2 avail Mem` | swap |

**Matoucí:** `avail Mem` stojí na řádku Swap, ale se swapem nesouvisí — patří
k paměti z řádku nad ním. `top` ho tam dal, protože tam bylo místo.

Dole v tabulce totéž pro každý proces: `%CPU` a `%MEM`.

**`top` = `uptime` + `free -h` + seznam procesů na jedné obrazovce.**
Proto ho admini otevírají jako první. Ven klávesou `q`.

### Řádek %Cpu(s)

Nejrychlejší je koukat na **`id` (idle)** — u mě 99.7, takže vytížení 0,3 %.
100 minus `id`. Ostatní:

- `us` user — běžné programy
- `sy` system — samotné jádro
- `ni` nice — programy se sníženou prioritou
- **`wa` wait** — procesor čeká na disk. Vysoké = problém není v procesoru,
  ale v disku.
- **`st` steal** — čas, který hypervizor vzal a dal jiné virtuálce.
  Na Multipassu nic neznamená, na VPS ano: vysoké = fyzický stroj pode mnou
  je přetížený ostatními zákazníky.

`top` je otáčkoměr, ne teploměr — ukazuje vytížení, ne teplotu.

### load average

Tři čísla = průměrné vytížení za 1, 5 a 15 minut.
**Mám jeden procesor, takže 1.00 = plné vytížení.** 3.00 by znamenalo,
že na procesor čekají tři úlohy a všechno se táhne.

### 2 users

Počet **relací**, ne lidí. Jsem tam sám, ale mám dvě spojení — `w` ukazuje
`pts/0` (terminál, kam píšu) a `sshd-session` (spojení, přes které jsem tam).

**Multipass se k virtuálce připojuje přes SSH**, jen s vlastním klíčem, který si
spravuje sám. Takže SSH jsem nepřeskočil — používám ho od prvního dne.
Adresa 192.168.252.1 je můj Mac na virtuální síti mezi ním a Ubuntu.

---

## `ip address` — co hledat

1. **`state UP`** — běží rozhraní? Když `DOWN`, karta je vypnutá a tím konec.
2. **Řádek `inet`** — má adresu? Když rozhraní běží a `inet` chybí, nesehnalo
   si adresu (problém s DHCP).
3. **Které rozhraní to je** — tohle jsem potřeboval u iftopu, když jsem
   napsal `eth0` místo `enp0s1`.

Mám dvě rozhraní:
- **`lo`** — loopback, smyčka do sebe. Není karta, ale způsob, jak spolu
  můžou programy na jednom stroji mluvit po síti. Vždy 127.0.0.1 = `localhost`.
- **`enp0s1`** — skutečná (virtuální) karta ven.

**`dynamic`** = adresa propůjčená na čas, `valid_lft 2461sec` je zbytek zápůjčky.
Automaticky se prodlužuje. Servery mívají adresu napevno, aby se neměnila.

Tyhle adresy jsou vnitřní, z internetu na ně nikdo nedosáhne. Citlivé jsou
hesla a soukromé klíče, ne adresy.

---

## `man -k` funguje líp, než jsem myslel

**`man -ks 1 <slovo>`** — filtr na sekci 1 = příkazy, co píšu do terminálu.
Ostatní sekce jsou pro programátory.

Porovnání: `man -k memory` → dvě stě řádků. `man -ks 1 memory` → osm řádků
a `free` mezi nimi.

**Funguje i celá fráze:** `man -ks 1 "who is logged"` → vypadlo `w`.
`man -ks 1 running` → `uptime`.

Ale je to hit or miss — `man -k disk` nenašlo `df`, protože jeho popis zní
"report file system disk space usage" a utopilo se to mezi fdisk, cfdisk, sgdisk.

**Pozor:** `man -ks running` bez čísla sekce hodí "apropos what?" — chce buď
sekci, nebo `man -k`.

---

## Co jsem objevil sám

**`iostat`** — vytížení disků. Když je v `top` vysoké `wa`, tohle řekne na kterém
disku a proč. U mě `%iowait 0.01`, `%idle 99.84` — klid.

**`mpstat`** — vytížení procesoru, u víc jader po jádrech.
`mpstat -I` samo nestačí, chce upřesnění: `mpstat -I SUM`.

Obojí je z balíčku `sysstat` a **v obrazu už je** — nemusel jsem nic instalovat.

**`loop0` až `loop3` v `iostat`** nejsou disky. Takhle Linux připojuje snapy —
instaloval jsem `tldr` přes snap a systém si k němu vyrobil tahle zařízení.
Rozdíl apt vs snap v praxi: snap se připojí jako samostatný souborový systém,
balíček z aptu se rozbalí do systému.

---

## Co nešlo a jak jsem to vyřešil

**`iftop` bez sudo → "You don't have permission ... CAP_NET_RAW may be required".**
Odposlech síťového provozu smí jen správce — jinak by si běžný uživatel mohl
číst cizí data. **Ta chyba je funkce, ne závada.** V Day 1 to šlo, protože
jsem psal `sudo iftop`.

**Zasekl jsem se na promptu `>`.** Napsal jsem `man -k "memmory` a zapomněl
zavírací uvozovku. Bash čekal, až ji dopíšu, a všechno další bral jako
pokračování. **Ten znak `>` na začátku řádku je celá informace** — bash říká
"pokračuj, ještě jsi neskončil". Ven: **Ctrl+C** (naráz, ne napsat "^C").
Funguje i u neuzavřené závorky nebo apostrofu.

Uvozovky u `man -k` vůbec nepotřebuju, kromě víceslovné fráze.
A hledat se musí anglicky.

**`multipass` uvnitř Ubuntu → command not found**, celkem třikrát.
Hláška nabízí `sudo snap install multipass` — to je past. Ubuntu jen vidí
neznámé slovo a hledá balíček toho jména. **Dokud mi hláška nabízí instalaci
něčeho, co už na druhém stroji mám, je to signál, že jsem ve špatném okně.**

---

## Pravidlo, které mi tohle vyřeší

Terminál je jen okno. Podle toho, co jsem naposled napsal, koukám buď na Mac,
nebo do Ubuntu. Nic na obrazovce to neřekne — mění se jen začátek řádku.

| Prompt | Kde jsem | Co tam funguje |
|---|---|---|
| `jaroslavkopecky@Jaroslav--MacBook-Pro ~ %` | Mac | `multipass`, `brew` |
| `ubuntu@linuxlab:~$` | Ubuntu | `df`, `free`, `top`, `apt` |

**Než něco napíšu, mrknu na začátek řádku.**

Postup na konci dne:
    exit                      ← zpátky na Macu
    multipass stop server1    ← vypne virtuálku (ne Multipass, ten běží dál)

`pwd` na obou: Mac → `/Users/jaroslavkopecky`, Ubuntu → `/home/ubuntu`.
Oba unixové, domovské adresáře jinde.

---

## Oprava mých vlastních poznámek

V Day 1 mám napsáno, že od otázky k příkazu `man` nevede. Platí to jen
napůl — **`man -ks 1 <slovo>`** vede docela často. `man` bez filtru ne.

---

---

## Vytváření souborů a adresářů

| Příkaz | Co dělá |
|---|---|
| `mkdir test1` | vytvoří adresář |
| `touch file1` | vytvoří prázdný soubor |
| `mv file1 test1` | přesune soubor do adresáře |
| `rm -r test1` | smaže adresář i s obsahem |

**Kam se to uloží:** do adresáře, kde zrovna stojím (`pwd`), u mě `/home/ubuntu`.
Fyzicky na `/dev/sda1` — ten jediný skutečný disk z `df -h`.

**`/test1/file` → No such file or directory.** Lomítko na začátku = začni od kořene,
takže jsem hledal `/test1`. Můj adresář byl `/home/ubuntu/test1`.
Absolutní vs relativní cesta z Day 2, potřetí.

## Jak do souboru něco napsat

    echo "text" > file1      přepíše celý obsah
    echo "text" >> file1     přidá na konec
    cat file1                zkontroluje výsledek

**Jedna šipka přepisuje, dvě přidávají.** Jednou šipkou na plný soubor = přijdu o obsah.

Na delší text `nano file1` — Ctrl+O a Enter uloží, Ctrl+X zavře.

`cat file1` po `touch` nevypsal nic, protože `touch` vytváří soubor **prázdný**.

## ls vs less — podobný název, nic společného

- `ls` = **list** — jaké soubory v adresáři jsou (jen jména)
- `less` — obsah **jednoho** souboru, listování šipkami, `q` ven

Původně byl `more` (jen dopředu), pak vznikla lepší verze a pojmenovali ji
`less` — "less is more". Proto se Day 5 jmenuje "More or less".

Tři způsoby, jak se podívat dovnitř:
- `cat` — vysype celý obsah najednou (na krátké soubory)
- `less` — listování (na dlouhé, proto jsem ho použil na auth.log)
- `head` / `tail` — prvních / posledních deset řádků

## Proč jsem nenašel touch

`man -ks 1 "create file"` → nothing appropriate.
`man -ks 1 create` → našlo `mkdir`, ale **`touch` ne**.

Důvod: `touch` má v manuálu popis "Update the access and modification times of
each FILE". Oficiálně je to nástroj na časová razítka — **vytvoření souboru je
jen vedlejší efekt**, když soubor neexistuje. Proto ho hledání slova "create"
nenajde.

Tohle je limit `man -k`: hledá shodu slov v popisu, nechápe, co chci.

## Co nešlo
- `cd..` bez mezery → command not found. Správně `cd ..` — `cd` je příkaz,
  `..` je jeho argument, mezi nimi musí být mezera.
- `cat current` → "Is a directory". `cat` čte soubory, ne adresáře.
- Pustil jsem `cat` bez argumentu — čekal na vstup z klávesnice, ven Ctrl+C.

## Co jsem si prošel navíc
`ls /` — kořen: bin, boot, dev, etc, home, lib, media, mnt, opt, proc, root,
run, sbin, snap, srv, sys, tmp, usr, var.

V domovském adresáři mám `snap/tldr` s podadresáři `792`, `common`, `current` —
takhle si snap ukládá data pro uživatele. `792` je číslo verze,
`current` na ni ukazuje.
