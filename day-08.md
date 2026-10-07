# 5. 10. 2026 — Day 8: Práce s textem a roury

## Tři datové proudy

Každý program má tři kanály:

| Číslo | Jméno | Odkud / kam |
|---|---|---|
| 0 | stdin — standardní vstup | obvykle klávesnice |
| 1 | stdout — standardní výstup | obrazovka |
| 2 | stderr — chybový výstup | taky obrazovka, ale zvlášť |

Proto chybová hláška vyskočí na obrazovku, i když výstup přesměruju
do souboru — jde jiným kanálem. Chyby se dají přesměrovat zvlášť: `2>`.

## Příkazy na text

| Příkaz | Co dělá |
|---|---|
| `echo "text"` | vypíše text |
| `echo $PS1` | vypíše proměnnou — tady nastavení vzhledu promptu |
| `cat soubor` | vypíše celý soubor |
| `tac soubor` | totéž pozpátku, od posledního řádku |
| `head -5 soubor` | prvních 5 řádků (bez čísla 10) |
| `tail soubor` | posledních 10 řádků |
| `tail -f soubor` | sleduje soubor živě, nové řádky ukazuje hned. Ven Ctrl+C |
| `wc -l soubor` | počet řádků (wc = word count) |
| `sort` | seřadí řádky abecedně |
| `uniq` | vyhodí opakující se řádky — funguje jen po `sort` |
| `grep "slovo" soubor` | vypíše jen řádky, kde slovo je |
| `cut -d":" -f5 soubor` | vyřízne z každého řádku jeden sloupec |

**`tail -f` v praxi:** v jednom okně `tail -f /var/log/apache2/access.log`,
v druhém `curl localhost` → v prvním okamžitě naskočí nový řádek.
Takhle se sleduje, co se na serveru děje právě teď.

## cut na /etc/passwd

    cut -d":" -f5 /etc/passwd

- `-d":"` — oddělovač (delimiter) je dvojtečka
- `-f5` — chci pátý sloupec (field) = popis uživatele

Prázdné řádky ve výsledku = uživatelé, kteří popis nemají.

## Přesměrování a roury

| Znak | Co dělá |
|---|---|
| `>` | výstup do souboru — **přepíše** obsah |
| `>>` | výstup do souboru — **přidá** na konec |
| `\|` | roura: výstup jednoho příkazu = vstup dalšího |

Ověřeno: `ls -lha > list` a pak `ls /etc/ > list` → první obsah zmizel.
`echo "hello" >> list` → hello přibylo na konec.

**Opravuju si poznámku:** `>` není „výstup jako vstup" — to je roura `|`.
`>` posílá výstup do souboru.

Roury se řetězí:

    grep "root" /var/log/auth.log | grep -o "IP regex" | sort | uniq > ~/attackers.txt

najdi řádky s root → vytáhni z nich IP adresy → seřaď → nech každou jednou → ulož.

## RegEx — regulární výrazy

Vzor pro hledání textu. IP adresa:

    [0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}

- `[0-9]` — jedna číslice
- `\{1,3\}` — jednou až třikrát
- `\.` — tečka (bez lomítka by tečka znamenala „jakýkoli znak")

`grep -o` — vypíše jen tu část, která odpovídá vzoru, ne celý řádek.

## Co nešlo

- `attackers.txt` zůstal prázdný, dva důvody:
  1. regex končil `\.` → hledal tečku za čtvrtým číslem, ta tam není
  2. na lokální virtuálce útočníci nejsou — jediná IP v logu je můj Mac.
     Smysl to bude mít na veřejném serveru.
- Překlepy: `service`, `apasche2`, `psswd`, `loq` — pokaždé „No such file".
  Tab by je doplnil správně.

## Odkaz na lekci
https://linuxupskillchallenge.org/08/

## Otázky do příště
-
