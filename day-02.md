# 20. 9. 2026 — Day 2: Basic navigation

## Struktura systému

Všechno začíná u `/` — root directory, kořenový adresář.
Není žádné C: a D:. Vše v systému visí pod tímhle jedním adresářem.

Pár standardních:
- `/bin` — binaries, spustitelné soubory
- `/boot` — boot config
- `/lib` — libraries, knihovny
- `/etc` — konfigurace
- `/home` — domovské adresáře uživatelů

---

## Absolutní vs relativní cesta — lomítka

**Lomítko na začátku mění význam. Lomítko na konci ne.**

- `cd /etc` — začni od kořene. Absolutní cesta, funguje odkudkoliv.
- `cd etc` — začni tam, kde jsem. Relativní cesta, funguje jen když tu nějaké
  `etc` je.
- `cd /etc/init.d/` = `cd /etc/init.d` — lomítko na konci se u adresáře ignoruje.

Proto jsou tyhle dva postupy stejné:
    cd /etc/init.d/          ← absolutní, na jeden zátah
    cd /etc  →  cd init.d/   ← absolutní krok, pak relativní

Na konci začne lomítko hrát roli až u `rsync` a symlinků. Zatím neřeším.

---

## Pohyb

`pwd` — print working directory, kde zrovna jsem → `/home/ubuntu`
`cd` — change directory

Zkratky:
- `cd ~` = `cd /home/ubuntu` — domů
- `cd -` — zpátky na předchozí adresář
- `cd ..` — o úroveň výš, do nadřazeného

Vlnovku `~` napíšu přes anglickou klávesnici: shift + klávesa vedle |    -> na Macbooku

---

## Environment variables

Proměnné prostředí — systém si v nich drží hodnoty, na které se dá odkazovat.

- `$HOME` → `/home/ubuntu`, takže `cd $HOME` = `cd ~`
- `$OLDPWD` → předchozí adresář, `cd $OLDPWD` = `cd -`
- `$PWD` → aktuální adresář (takže `cd $PWD` nedělá nic, jen ukazuje princip)
- `$USER` → moje uživatelské jméno, tedy `ubuntu`. Pozor: `cd $USER` **není**
  cesta domů — zkusí to vlézt do adresáře jménem "ubuntu" v aktuální složce.
  Na vypsání hodnoty je `echo $USER`.

Všechny vypíšu příkazem `printenv`.

---

## Výpis obsahu

`ls` — list files
- `ls -l` — detailní výpis, práva, vlastník, velikost, datum
- `ls -h` — human readable velikosti
- `ls -a` — včetně skrytých (ty s tečkou)
- `ls -lh` = `ls -l -h` — volby jde slepit dohromady

---

## Vytváření a mazání

| Příkaz | Co dělá |
|---|---|
| `mkdir test` | make directory — vytvoří adresář |
| `touch file` | vytvoří prázdný soubor |
| `mv file test` | přesune soubor do adresáře |
| `mv file novy-nazev` | **totéž `mv` i přejmenovává** |
| `cp soubor kopie` | copy — zkopíruje |
| `rm soubor` | remove — smaže soubor |
| `rmdir adresar` | smaže adresář, ale **jen když je prázdný** |
| `rm -r adresar` | smaže adresář i s obsahem |

`mv` dělá dvě věci podle toho, co mu dáš jako druhý argument: když adresář,
přesouvá; když jméno, přejmenovává.

---

## RTFM — Read The Fucking Manual

- `man <příkaz>` — plný manuál
- `man -k <klíčové slovo>` = `apropos` — hledá podle klíčového slova
- `tldr <příkaz>` — zkrácená verze manuálu s praktickými příklady.
  **Nečte adresář** — ukazuje komunitou psané příklady použití příkazu.
  Prakticky nejrychlejší způsob, jak zjistit, jak se co používá.
- `help` — u vestavěných příkazů shellu, kde `man` není
- `type <příkaz>` — řekne, **co ten příkaz vlastně je**: vestavěný příkaz
  shellu, alias, nebo program na disku. Není to o balíčcích.
- `info` — podrobnější dokumentace

---

## Co nešlo a jak jsem to vyřešil

**`rm test` → "cannot remove 'test': Is a directory".**
`rm` sám o sobě maže soubory, ne adresáře. Na prázdný adresář je `rmdir`,
na adresář s obsahem `rm -r`.

**`tldr` nebyl nainstalovaný.**
Systém nabídl tři varianty — přes `snap` i přes `apt`. Ubuntu má dva
paralelní systémy balíčků. Nainstaloval jsem `sudo snap install tldr`.

---

## Co jsem si pletl
- Myslel jsem, že `tldr` čte adresář. Čte manuál a vypíše zkrácené příklady.
- Myslel jsem, že `type` hledá druh balíku. Říká, jestli je příkaz vestavěný
  v shellu, alias, nebo soubor na disku.
- Napsal jsem `rm - r` s mezerami. Správně je `rm -r`, volba se lepí na pomlčku.
- Myslel jsem, že `cd $USER` je cesta domů. Není — `$USER` je jen jméno,
  ne cesta.

---
