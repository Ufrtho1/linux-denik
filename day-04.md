
# 23. 9. 2026 — Day 4: Installing software, exploring the file structure

## Distribuce a rodiny

**Linux = jen jádro (motor).** Distribuce = celé auto kolem něj:
jádro + nástroje + balíčkovač + nastavení.

Distribuce se dělí do rodin jako auta do koncernů — díly ze Škodovky
sednou do Seatu, do Toyoty ne.

| Rodina | Distribuce | Balíček | Nízká úroveň | Vysoká úroveň |
|---|---|---|---|---|
| Debian | Debian, Ubuntu, Mint | `.deb` | `dpkg` | `apt` |
| Red Hat | RHEL, Rocky, Alma, Fedora | `.rpm` | `rpm` | `dnf` (dřív `yum`) |

"Na debianové bázi" = Ubuntu je postavené na Debianu → `.deb`, `apt`.
Na RedHat rodině je to stejné, jen `dnf` místo `apt`.

## dpkg vs apt

- **`dpkg`** — nainstaluje jeden `.deb` soubor. Neřeší, co dalšího potřebuje.
  Když něco chybí, spadne. Montáž jednoho dílu.
- **`apt`** — stáhne balíček z repozitáře **i se vším, co k němu patří**.
  Objednávka celé sady.
- `aptitude`, `synaptic` — jiné přední panely ke stejnému systému.
  Synaptic je grafický, s okny.

Viděl jsem to sám: `mc` potřeboval ještě bzip2, mailcap, mc-data a unzip.
Apt je dohledal a nainstaloval sám, já jen potvrdil.

"Suggested packages" = jen návrh, neinstalují se.

## Instalace, když neznám jméno balíčku

    apt search "midnight commander"   → balíček se jmenuje mc
    sudo apt install mc

Hledat přes `apt search` — jméno balíčku často není jméno programu.

## Midnight Commander (mc)

Dvoupanelový správce souborů v terminálu, jako Total Commander.
Ven: **F10** — na Macu **Fn+F10**. Nouzově `Esc` a pak `0`.

## Adresářová struktura

`man hier` — popis celé hierarchie adresářů.

**`/etc` = konfigurace.** Nastavení skoro všeho v systému je v textových
souborech tady. Např. `less /etc/ssh/sshd_config` = nastavení SSH serveru.

## Co nešlo

**`sudo install apt mc` → "No such file or directory".** Ne "command not found",
protože `install` je skutečný příkaz — kopíruje soubory. Hledal soubor jménem
`apt`. **Pořadí: nejdřív nástroj, pak co s ním** → `sudo apt install mc`.

**Z mc jsem nemohl ven.** Psal jsem `10`, `Quit`, `q` do bashe, ale mc už bylo
zavřené, nebo jsem nemačkal F10. Na Macu Fn+F10.

## Odkaz na lekci
https://linuxupskillchallenge.org/04/

## Otázky do příště
-
