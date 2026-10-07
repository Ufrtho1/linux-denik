# 28. 9. 2026 — Day 6: Vim

## Proč vim, když existuje nano

`vi` je na **každém** Linuxu, i na holém serveru, kde nic jiného není.
Nano tam být nemusí. Na cizím stroji přes SSH je vi jistota.

Na juniora stačí **přežití**: otevřít, změnit, uložit, odejít.
Zbytek se nabaluje časem.

## Přežití — tohle musím umět

| Co | Jak |
|---|---|
| Otevřít soubor | `vim soubor` |
| Začít psát | `i` |
| Přestat psát | `Esc` |
| Uložit a odejít | `:wq` |
| Odejít bez uložení | `:q!` |
| Když nevím, kde jsem | `Esc` několikrát, pak `:q!` |

**Soubor vznikne až při uložení.** `vim filename` a pak `:q!` = žádný soubor.
Ověřeno: po `:q!` nebyl `filename` v `ls`.

## Režimy

Vim nemá jeden režim jako normální editor. Klávesy dělají různé věci
podle toho, kde jsem.

| Režim | Jak se tam dostanu | Co dělám |
|---|---|---|
| Normal | `Esc` | pohyb, příkazy — klávesy nejsou text |
| Insert | `i` | píšu text, dole `-- INSERT --` |
| Visual | `v` | označuju text |
| Command | `:` | příkazy dole — uložit, odejít, hledat |

Dneska mě chytlo: byl jsem v Insert a psal `exit` — bralo se to jako text.

## Pohyb (Normal)

- šipky, nebo `h` ← `j` ↓ `k` ↑ `l` →
- `gg` začátek souboru, `G` konec

## Úpravy (Normal)

| Klávesa | Co dělá |
|---|---|
| `x` | smaže **jeden znak** pod kurzorem |
| `dd` | smaže (vyjme) celý řádek |
| `yy` | zkopíruje celý řádek (yank) |
| `p` | vloží (paste) |
| `u` | undo |
| **Ctrl+R** | redo — **samotné `r` je něco jiného**, nahradí jeden znak |

Kus textu: `v` → označit šipkami → `d` smaže, `y` zkopíruje.

## Příkazy (:)

    :w              uložit
    :w myfile       uložit pod jiným jménem
    :q              odejít
    :q!             odejít bez uložení
    :wq             uložit a odejít
    :set number     čísla řádků
    :set nonumber   vypnout čísla

## Hledání

    /slovo          hledat
    n               další výskyt

## Najít a nahradit

    :%s/staré/nové/gc

- `:` příkazový režim
- `%` všechny řádky
- `s` substitute, nahradit
- `/staré/nové/` co za co
- `g` všechny výskyty na řádku, ne jen první
- `c` potvrdit každou náhradu

## Kde se to naučit pořádně

- `vimtutor` — interaktivní návod přímo v terminálu, cca 30 minut
- Vim Adventures — hra v prohlížeči

## Co jsem zkoušel

    cp /etc/services ./     zkopíroval systémový soubor k sobě
                            (./ = aktuální adresář)
    vim services            otevřel, zkoušel se v něm pohybovat
    :w myfile               uložil kopii pod jiným jménem

## Co nešlo

- `cd file1` → "Not a directory". `file1` je soubor, ne adresář.
- `file1` jako příkaz → "command not found". Je to soubor, ne program.
  Bash se snažil najít příkaz toho jména a nabídl podobné.

## Co jsem si pletl
- Myslel jsem, že `vim filename` soubor vytvoří hned. Vznikne až po uložení.
- Myslel jsem, že `r` je redo. Redo je Ctrl+R, `r` nahrazuje znak.

## Odkaz na lekci
https://linuxupskillchallenge.org/06/

## Otázky do příště
-
