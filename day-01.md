# 19. 9. 2026 — Day 1: Get to know your server

## Co jsem udělal
- Nainstaloval Multipass přes brew (brew = správce balíčků na macOS),
  spustil Ubuntu 26.04 LTS jako VM na Macu
- Poprvé se dostal do Linuxu a projel všechny příkazy z Day 1
- Day 1 splněn — oba úkoly (přihlášení + kontrola stavu serveru)
- SSH přeskočeno, běžím na lokální VM. Kurz sám píše, že u lokální VM to jde
  přeskočit. Pořádně se probírá na Day 3.

---

## Hlavní věc dneška

Server nemá obrazovku. Žádný správce úloh, žádný Finder, žádná ikonka disku.
Stojí někde v serverovně a já se na něj nemůžu podívat — můžu se ho jen ptát.
Každý příkaz je jedna otázka.

**Když hledám závadu, ptám se v tomhle pořadí:**

| Otázka | Příkaz |
|---|---|
| Není plný disk? *(nejčastější příčina)* | `df -h` |
| Nedošla paměť? | `free -h` |
| Nežere něco procesor? | `top` |
| Nerestartoval se? | `uptime` |
| Má síť? | `ip address` |
| Dělá tu někdo něco? | `w` |

**Pamatovat si mám otázky, ne příkazy.** Příkaz si dohledám. Otázku ne — buď
mě napadne se zeptat, nebo ne.

Kurz ukázal asi 30 příkazů, ale reálně používaných je těch šest.
`lshw`, `lspci`, `lsusb`, `vmstat` otevřu možná dvakrát za rok — a na virtuálce
mi stejně o hardwaru neřeknou skoro nic, protože žádný skutečný není.

---

## Jak se dostanu od otázky k příkazu

`man` na tohle **není** — funguje, jen když jméno příkazu už znám.

1. **Dva Taby** — napíšu `ls` a dvakrát Tab, vypíše všechno, co tak začíná
2. **`apropos <klíčové slovo>`** — hledá v popisech příkazů. Vyzkoušeno
   `apropos disk`: vysypalo padesát řádků o oddílech a SCSI a `df` mezi nimi
   nebylo. Hodně hit or miss.
3. **Web (googlit, AI)** — často nejrychlejší, a dělá to i člověk s patnácti
   lety praxe

**Pořadí: znám otázku → najdu příkaz (Tab, apropos, web) → `man` mi řekne volby.**
`man` je poslední krok, ne první.

---

## Vzor v názvech

`ls` = list, vypiš. Nejsou to náhodné příkazy, je to rodina.

Dva Taby po `ls` ukázaly, že jich je mnohem víc, než bylo v kurzu:
`lsattr lsblk lsclocks lsfd lshw lsinitramfs lsinitrd lsipc lsirq lslocks
lslogins lsmem lsmod lsmtd lsns lsof lspci lspgpot lspower lsusb`

Výjimka: `lsb_release` — tam LSB = Linux Standard Base, s výpisem nesouvisí.

---

## Co jsem pochopil

**Řetěz, jak vzniklo Ubuntu:** brew nainstaloval jen Multipass (nástroj od
Canonicalu). Multipass si pak sám stáhl obraz Ubuntu a nastartoval ho jako
virtuální počítač. Brew s Ubuntu nemá nic společného.

**Prompt `ubuntu@server1:~$`** = uživatel @ stroj : kde jsem, a znak na konci.
`$` = běžný uživatel, `#` = root.

**`ls -la`** — nejsou to "podrobnosti", jsou to dvě volby slepené:
- `-l` long, výpis po řádcích s právy, vlastníkem, velikostí, datem
- `-a` all, včetně skrytých souborů (ty s tečkou)

Samotné `ls` v domovském adresáři nevypsalo nic — jsou tam zatím jen skryté
soubory (`.bashrc`, `.ssh`, `.profile`).

**`sudo`** — není režim ani povolení. Provede **jeden** příkaz jako root a hned
jsem zpátky jako normální uživatel. Smysl: běžně pracuju jako někdo, kdo nemůže
rozbít systém, a práva si zvednu jen pro ten konkrétní krok.

**`apt`** — správce balíčků. Stahuje software z repozitářů = oficiálních skladů
Ubuntu. Balíček = nainstalovaný software.
- `apt update` = stáhnu si čerstvý seznam toho, co je k dispozici a v jaké verzi.
  Starý seznam se přepíše novým. Na systému to nemění nic.
- `apt upgrade` = podle toho seznamu nainstaluje novější verze balíčků,
  které mám staré.
- `&&` = spusť druhý příkaz, jen když první dopadl dobře.

Přirovnání: seznam je leták, repozitář je sklad.

---

## Kde vlastně jsem — vrstvy

Pletl jsem si to. Nejsem nikde na serveru, sedím doma u Macu a ten "server"
je program běžící uvnitř mého notebooku.

MacBook (fyzický hardware) └── macOS └── hypervizor (spustil ho Multipass) └── Ubuntu 26.04 = "server1" ← tady jsem

**Hypervizor typu 1 vs typu 2:**
- Typ 1 běží přímo na železe, pod ním není žádný OS. VMware ESXi, Proxmox.
  Takhle to jede v datacentrech.
- Typ 2 běží jako aplikace nad existujícím systémem. Multipass, VirtualBox,
  Parallels. To je můj případ.

Až přejdu na Hetzner, budu mít virtuálku na hypervizoru typu 1 v německém
datacentru. Pak už věta "jsem na serveru" bude sedět.

---

## Jak se odhlásit a vypnout

`exit` mě jen odhlásí ze shellu a vrátí na Mac. Ubuntu poběží dál na pozadí
a žere paměť a baterku. **Odhlásit se ≠ vypnout.**

Na konci dne:
exit multipass stop server1

Zpátky:
multipass start server1 multipass shell server1

Stav: `multipass list`
Nikdy: `multipass delete` — zahodí virtuálku i s prací.

---

## Co nešlo a jak jsem to vyřešil

**`sudo iftop -i eth0` → "No such device exists".**
V návodu bylo `enp1s0` jako příklad a hned pod tím napsáno, ať si dosadím vlastní
rozhraní. Moje je `enp0s1` — viděl jsem ho ve výstupu `ip address` o pár příkazů
dřív. Po opravě na `sudo iftop -i enp0s1` to naběhlo.
**Nezkopírovat a doufat, ale přečíst si skutečný stav z výstupu vlastního stroje.**

**`sudo apt install net tools` → "Unable to locate package".**
Balíček se jmenuje `net-tools`, s pomlčkou. Bez ní to apt bere jako dva balíčky.

**`ifconfig` a `netstat` neexistovaly.**
Ubuntu je nahradilo modernějším `ip address`. Systém sám poradil, že se dají
doinstalovat přes `net-tools`. Stejně tak `ifstat` a `iftop`.

---

## Co jsem si pletl
- Myslel jsem, že `sudo` je povolení správce. Je to provedení jednoho příkazu jako root.
- Říkal jsem `apt` "služba". Služba v Linuxu = něco, co běží na pozadí pořád
  (webserver, databáze). `apt` je program, co proběhne a skončí.
- Řekl jsem "novější verze funkcí". Správně **balíčků**.
- Myslel jsem, že od otázky k příkazu vede `man`. Nevede.
- Myslel jsem, že sedím na nějakém vzdáleném serveru. Sedím doma u Macu.

---
