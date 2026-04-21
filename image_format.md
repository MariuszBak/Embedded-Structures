# Format obrazu dla embedded

## Ogólny opis

Monochromatyczny obraz bitmapowy przechowywany jako tablica `uint8_t` w języku C. Używa tego samego **kolumnowego kodowania, LSB = górny wiersz** co format czcionki, z 4-bajtowym nagłówkiem `uint16 LE` określającym wymiary obrazu.

## Struktura tablicy

```
[nagłówek: 4 bajty] [dane pikseli]
```

### Nagłówek (4 bajty)

| Bajty | Typ | Opis |
|-------|-----|------|
| 0–1 | `uint16 LE` | Szerokość obrazu w pikselach |
| 2–3 | `uint16 LE` | Wysokość obrazu w pikselach |

Kodowanie little-endian: wartość `20` (dziesiętnie) zapisywana jest jako `0x14, 0x00`.

### Dane pikseli

Łączna liczba bajtów danych: `szerokość × ceil(wysokość / 8)`

## Kodowanie bitów

Identyczne z formatem czcionki — **kolumnowe, LSB = górny wiersz**:

```
indeks bajtu = floor(wiersz / 8) × szerokość + kolumna
indeks bitu  = wiersz % 8
```

Przy wysokości ≤ 8 każda kolumna to dokładnie 1 bajt.
Przy wysokości > 8 kolumny zajmują kilka bajtów; wszystkie kolumny jednego wiersza bajtów poprzedzają następny.

### Przykład: obraz 20×8

```
bajty/kol   = ceil(8/8) = 1
bajty łącznie = 4 + 20 × 1 = 24

Nagłówek:  0x14, 0x00,  // szerokość = 20
           0x08, 0x00,  // wysokość  = 8

Dane:      [k0][k1]...[k19]   ← po jednym bajcie na kolumnę, wiersze 0–7
```

Każdy bajt danych koduje 8 wierszy jednej kolumny:

```
bit 0 (LSB) = wiersz 0  (góra)
bit 1       = wiersz 1
...
bit 7 (MSB) = wiersz 7  (dół)
```

### Przykład: obraz 10×16

```
bajty/kol   = ceil(16/8) = 2
bajty łącznie = 4 + 10 × 2 = 24

Nagłówek: 0x0A, 0x00,  // szerokość = 10
          0x10, 0x00,  // wysokość  = 16

Układ danych (20 bajtów):
  [R0C0][R0C1]...[R0C9]   ← wiersze 0–7,  wszystkie 10 kolumn
  [R1C0][R1C1]...[R1C9]   ← wiersze 8–15, wszystkie 10 kolumn
```

## Deklaracja C

```c
const uint8_t img_com_tcu_full[24] = {
    0x14, 0x00,  // szerokość = 20 (uint16 LE)
    0x08, 0x00,  // wysokość  = 8  (uint16 LE)
    0x7E, 0xFF, 0xFF, 0xFD, 0xFD, 0x81, 0xFD, 0xFF, 0xC3, 0xBD,
    0xBD, 0xDB, 0xFF, 0xC1, 0xBF, 0xBF, 0xC1, 0xFF, 0xFF, 0x7E,
};
```

## Obliczanie rozmiaru

```
bajty/kol     = ceil(wysokość / 8)
bajty łącznie = 4 + szerokość × bajty/kol
```

| Rozmiar | bajty/kol | Bajty łącznie |
|---------|-----------|---------------|
| 20×8    | 1         | 24            |
| 16×16   | 2         | 68            |
| 32×32   | 4         | 132           |
| 128×64  | 8         | 1028          |

## Konwencja pikseli

- Bit `1` = piksel **włączony** (świeci)
- Bit `0` = piksel **wyłączony** (ciemny)

Nieużywane bity w ostatnim wierszu bajtów (gdy `wysokość % 8 ≠ 0`) są zawsze `0`.
