# progress-bar

Bogata biblioteka pasków postępu i spinnerów dla **H#**, inspirowana
`indicatif` (Rust), `cli-spinners`/`cli-progress` i podobnymi bibliotekami
znanymi z innych ekosystemów — ale w 100% napisana w H#, zgodna z
konwencjami projektu `bit` (pakiet-menedżer H#) i wbudowanym paskiem
postępu, który `bit` pokazuje przy instalacji.

## Co dostajesz

- **21 wbudowanych motywów paska** — w tym dokładnie te same 6, których
  używa natywny pasek postępu w `bit` (`default`, `arrow`, `dotted`,
  `cargo`, `equals`, `blocks`), plus 15 kolejnych: `ascii`, `fine`, `fine2`,
  `rough`, `github`, `retro`, `material`, `thin`, `hearts`, `stars`,
  `circles`, `hash`, `shade`, `double`, `minimal`.
- **~39 wbudowanych spinnerów** — `dots` to dokładnie ten sam spinner co w
  `bit`; do tego dziesiątki kolejnych w duchu popularnych bibliotek
  spinnerów: `line`, `arc`, `bouncing_ball`, `moon`, `earth`, `clock`,
  `weather`, `triangle`, `circle_half`, `toggle`, `wave`, `pulse` i wiele,
  wiele innych (pełna lista: `spinner_frames::spinner_names()`).
- **W pełni własne paski i spinnery** — własne znaki (`theme::custom_theme`)
  i własne klatki animacji (`spinner::spinner_custom`), bez modyfikowania
  biblioteki.
- **Elastyczny silnik szablonów** — `{bar} {pos}/{len} {percent}% {msg}
  {eta} {elapsed} {bytes} {total_bytes} {speed} {spinner} {label}` w
  dowolnej kombinacji i kolejności.
- **`MultiProgress`** — wiele pasków naraz, przypiętych do dolnych linii
  terminala, z możliwością wstawiania trwałych logów nad blokiem bez
  zaburzania animacji.
- **ETA i prędkość** — prosty wariant (średnia od startu) i wygładzony
  (ruchome okno próbek, jak w `indicatif`).
- **W pełni fluent API** — `builder::new_bar(100, "Etykieta").theme("blocks").width(30).build()`.

Cała biblioteka jest napisana w tym samym, sprawdzonym stylu co sam `bit`
(zob. `bit/src/progress.h#`): stan trzymany w niemutowalnych structach,
przekazywany dalej jako nowa wartość z każdej funkcji — żadnych ukrytych
mutacji przez zmienne przechwycone w closures.

## Instalacja

**Wariant A — wendorowanie (zawsze działa, zalecane):**
Skopiuj cały katalog `src/` do swojego projektu (np. jako `src/progressbar/`
albo bezpośrednio do własnego `src/`, jeśli nazwy plików się nie
gryzą), po czym w swoim kodzie:

```
mod bar
mod builder
;; ... i inne moduły, których używasz — patrz "Moduły" niżej
```

Dokładnie tak samo, jak `bit` sam siebie buduje z wielu plików w
`bit/src/` (`mod cli`, `mod config`, ...).

**Wariant B — jako zależność `bit` (gdy pakiet trafi do rejestru):**

```
bit add progressbar
```

a potem w kodzie (składnia importu jak dla pakietów z GitHuba w README H#):

```
use "progressbar -> lib" from "pb"
```

## Szybki start

```
use "std -> time" from "t"
mod bar

fn main() is
    let mut pb = bar::bar_new(100, "Kopiowanie plików")

    let mut i: int = 0
    while i < 100 is
        pb = pb.inc(1)
        pb.draw()
        t::sleep_ms(40)
        i += 1
    end

    pb.finish("Skopiowano 100 plików")
end
```

Ważne — **żadna metoda nie mutuje odbiornika w miejscu**. Każda zwraca
nowy `ProgressBar`/`Spinner`/`ProgressStyle`/`MultiProgress`, dlatego
zawsze `pb = pb.inc(1)`, nigdy samo `pb.inc(1)`.

## Moduły

| moduł              | co zawiera |
|--------------------|------------|
| `core`             | `ProgressState` — wspólny stan (pozycja, total, komunikat, czas startu) |
| `theme`            | 21 motywów paska + `custom_theme(...)` na własne znaki |
| `spinner_frames`   | ~39 zestawów klatek spinnera |
| `human`            | formatowanie bajtów, czasu trwania, liczb z separatorem tysięcy |
| `eta`              | proste i wygładzone ETA/prędkość |
| `style`            | `ProgressStyle` — fluent builder stylu (`.with_theme()`, `.with_template()`, ...) |
| `template`         | silnik placeholderów, zamienia szablon na gotowy tekst |
| `bar`              | `ProgressBar` — gotowy pasek na jednej linii |
| `spinner`          | `Spinner` — animowany wskaźnik bez znanego postępu |
| `multi`            | `MultiProgress` — kilka pasków naraz |
| `builder`          | `ProgressBuilder` — najwygodniejsze wejście łączące `style`+`bar`/`spinner` |
| `lib`              | punkt wejścia paczki + kilka gotowych "szybkich skrótów" |

## Placeholdery szablonu

| placeholder     | co wstawia |
|-----------------|------------|
| `{bar}`         | sam pasek (albo animacja ping-pong, gdy total nieznane) |
| `{spinner}`     | bieżąca klatka spinnera |
| `{label}`       | stały tytuł zadania |
| `{msg}`         | dynamiczny komunikat (`set_message`) |
| `{pos}` / `{len}` | bieżąca pozycja / total |
| `{percent}`     | 0–100 |
| `{elapsed}`     | czas od startu, np. `3.4s`, `1m 05s` |
| `{eta}`         | szacowany czas do końca |
| `{bytes}` / `{total_bytes}` | pozycja/total sformatowane jako rozmiar (B/KB/MB/...) |
| `{speed}`       | tempo (jednostek/s), sformatowane jak rozmiar |

Przykład własnego szablonu:

```
let sty = style::style_default()
    .with_template("{{msg}}\n{{bar}} {{bytes}}/{{total_bytes}}  {{speed}}/s  ETA {{eta}}")
    .with_theme("blocks")
```

## Własny motyw i własny spinner

```
mod theme
mod spinner

;; własne znaki paska: otwiera "{", zamyka "}", wypełnienie "@", puste "."
let my_theme: string = theme::custom_theme("{", "}", "@", ".", "")
let sty = style::style_default().with_theme(my_theme)

;; własne klatki spinnera — cokolwiek, byle tablica stringów
let sp = spinner::spinner_custom("Szukam…", ["◐ ", " ◓", "◑ ", " ◒"])
```

## MultiProgress

```
mod multi
mod builder

let mut mp = multi::multi_new()
mp = mp.add(builder::new_bar(120, "core.hlib").theme("blocks").build())
mp = mp.add(builder::new_bar(80, "assets.hlib").theme("cargo").build())
mp = mp.draw()

mp = mp.inc(0, 5)
mp = mp.inc(1, 3)
mp = mp.log("assets.hlib: cache trafiony")   ;; trwały log NAD blokiem

mp = mp.finish_all("Wszystkie pliki pobrane")
```

## Przykłady

Katalog `examples/` zawiera 6 gotowych, samodzielnych mini-projektów
`bit` (każdy z zawendorowaną kopią biblioteki w swoim `src/`, więc
uruchamiają się od razu, bez dodatkowej konfiguracji):

| katalog | co pokazuje |
|---|---|
| `01-basic-bar` | najprostszy pasek: `inc` + `draw` + `finish` |
| `02-all-styles` | podgląd wszystkich 21 wbudowanych motywów naraz |
| `03-spinner-gallery` | kilka spinnerów animowanych na żywo |
| `04-download-progress` | pasek pobierania z `{bytes} {speed} {eta}` |
| `05-multi-progress` | trzy paski naraz + log wstawiany w trakcie |
| `06-custom-style` | w pełni własny motyw paska i własny spinner |

Uruchomienie (z wnętrza katalogu przykładu):

```
bit run
```

albo bezpośrednio przez interpreter:

```
h# preview src/main.h#
```

## Testy

Testy (`#[test]`) są dopisane bezpośrednio w plikach, których dotyczą —
`theme.h#`, `human.h#`, `template.h#` — zgodnie z konwencją H# pokazaną w
jego README. Uruchomienie: `bit test` (albo odpowiednik `h# check` z
Twojej wersji narzędzia).

## Zgodność z `bit`

Sześć pierwszych motywów paska i spinner `dots` są renderowane identycznie
jak natywny pasek postępu instalatora `bit` — jeśli chcesz, żeby Twoje
narzędzie CLI wyglądało spójnie z `bit`, po prostu użyj
`bar::bar_new(...)` (domyślny motyw = `default`) albo
`spinner::spinner_new(...)` (domyślny zestaw = `dots`).

## Licencja

MIT — patrz `LICENSE`.
