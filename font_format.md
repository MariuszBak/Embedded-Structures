# Format czcionki dla embedded

## Ogólny opis

Czcionka bitmapowa przechowywana jako tablica `uint8_t` w języku C. Każdy znak zakodowany jest jako stała liczba bajtów w układzie **kolumnowym, LSB = górny wiersz**.

## Struktura tablicy

```
[nagłówek: 4 bajty] [dane znaku 0] [dane znaku 1] ... [dane znaku N]
```

### Nagłówek (4 bajty)

| Bajt | Wartość | Opis |
|------|---------|------|
| 0 | `0x00` | Typ / zarezerwowany |
| 1 | `szerokość` | Szerokość znaku w pikselach (kolumny) |
| 2 | `wysokość` | Wysokość znaku w pikselach (wiersze) |
| 3 | `startChar` | Kod ASCII pierwszego znaku (np. `0x20` = spacja) |

### Dane znaku

Każdy znak zajmuje `szerokość × ceil(wysokość / 8)` bajtów.

Znaki przechowywane są kolejno, począwszy od `startChar` do `0x7F`.

## Kodowanie bitów

Dane zapisywane są **kolumna po kolumnie**. W każdej kolumnie wiersze upakowane są w bajty, gdzie **LSB = wiersz 0 (górny)**.

Przy wysokości > 8 każda kolumna zajmuje kilka bajtów. Wszystkie kolumny jednego wiersza bajtów (ang. *byte-row*) poprzedzają kolejny wiersz:

```
indeks bajtu = floor(wiersz / 8) × szerokość + kolumna
indeks bitu  = wiersz % 8
```

### Przykład: czcionka 5×7, znak 'A' (`0x7E, 0x11, 0x11, 0x7E, 0x00`)

```
kol:    0     1     2     3     4
bajt: 0x7E  0x11  0x11  0x7E  0x00

Kol 0 = 0x7E = 0111 1110
         w7=0 w6=1 w5=1 w4=1 w3=1 w2=1 w1=1 w0=0
```

Siatka pikseli (wiersz 0 = góra):

```
· █ █ █ ·
█ · · · █
█ · · · █
█ █ █ █ █
█ · · · █
█ · · · █
█ · · · █
```

### Przykład: czcionka 8×16 (2 wiersze bajtów na kolumnę)

```
bajty/znak = 8 × ceil(16/8) = 16

Układ: [R0K0][R0K1]...[R0K7][R1K0][R1K1]...[R1K7]
        ↑ wiersze 0–7 dla wszystkich kolumn
                              ↑ wiersze 8–15 dla wszystkich kolumn
```

## Deklaracja C

```c
const uint8_t sct_font5x7[] =
{
    0x00, 0x05, 0x07, 0x20,          // typ=0, szer=5, wys=7, start=0x20
    0x00, 0x00, 0x00, 0x00, 0x00,    // znak ' ' (0x20/32)
    0x00, 0x00, 0x2E, 0x00, 0x00,    // znak '!' (0x21/33)
    // ...
};
```

## Obliczanie rozmiaru

```
bajty/znak  = szerokość × ceil(wysokość / 8)
bajty łącznie = 4 + liczba_znaków × bajty/znak
```

**Przykład:** czcionka 5×7, 96 znaków (0x20–0x7F):
`4 + 96 × 5 = 484 bajty`

**Przykład:** czcionka 8×16, 96 znaków:
`4 + 96 × 16 = 1540 bajtów`
