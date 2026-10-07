# 25. 9. 2026 — Day 5: More or less

## more

Stránkovač — vypisuje text po jedné obrazovce a čeká.
Bez něj by dlouhý soubor profrčel a viděl bych jen konec.

    more /var/log/auth.log

- Enter — o řádek dolů
- mezerník — o stránku dolů
- `h` — nápověda
- `q` — ven

Umí jen dopředu.

## less

Lepší `more` — umí i zpátky. Jméno z "less is more".

    less /var/log/auth.log

| Klávesa | Co dělá |
|---|---|
| šipky | posun po řádcích |
| `g` | začátek souboru |
| `G` | konec souboru |
| `/slovo` | hledání |
| `n` / `N` | další / předchozí výskyt |
| `h` | nápověda |
| `q` | quit — konečně vím proč `q` |

**`G` na konci je v praxi nejdůležitější** — logy se zapisují dolů,
nejnovější záznamy jsou na konci.

## Doplňování Tabem

Napíšu začátek, zmáčknu Tab a bash doplní zbytek.

**Doplní jen když je možnost jediná.** Když jich je víc, nic se nestane
a na druhý Tab je vypíše.

- `le` + Tab → nic, na `le` začíná less, lessfile, lesspipe, let…
- `less /var/l` + Tab → víc možností, vypíše je
- `less /var/log/au` + Tab → jediná možnost → doplní `auth.log`

Funguje na příkazy i na cesty. Šetří psaní i překlepy.

## History

    history          všechny příkazy, co jsem napsal
    history 10       posledních 10
    !267             spustí příkaz číslo 267
    !!               spustí poslední příkaz (= !-1)
    !sudo            spustí poslední příkaz začínající na "sudo"
    sudo !!          zopakuje poslední příkaz se sudo

`sudo !!` je nejpraktičtější — když zapomenu `sudo` a dostanu "permission denied",
nemusím celý příkaz psát znovu.

**Clear:** `clear` nebo **Ctrl+L**.

### Kde se historie ukládá

    ~/.bash_history

Pozor: zapisuje se tam až **při ukončení relace**. Příkazy z aktuálního
okna tam ještě nemusí být, i když je `history` ukazuje.

**Bezpečnost:** historie zaznamená všechno, co napíšu. Proto se nikdy
nepíšou hesla přímo do příkazu — zůstala by v souboru čitelná.

## Dot files = skryté soubory

Soubory začínající tečkou. Vidím je přes `ls -a`.

**`.bashrc`** — skript, který se spustí pokaždé, když startuje nový bash.
Nastavuje se tu vzhled promptu, zkratky a podobně.

Souvisí s Day 3: když jsem napsal `bash` a prompt se změnil, nový bash
si přečetl i tenhle soubor.

    less .bashrc      jen prohlédnout
    nano .bashrc      upravit

Změny se projeví v novém bashi nebo po `source ~/.bashrc`.

**nano:** Ctrl+O a Enter uloží, Ctrl+X zavře.

## Co nešlo

- `history10` bez mezery → command not found. Stejná chyba jako `cd..` —
  příkaz a jeho argument musí oddělovat mezera.
- `le` + Tab nic nedoplnilo — víc příkazů začíná na `le`.

## Odkaz na lekci
https://linuxupskillchallenge.org/05/

## Otázky do příště
-
