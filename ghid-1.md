# Subiectul 1 — Ghid (operații pe biți, vectori, matrici)

Afișare recomandată: `PRINTF32`/`PRINTF64` sau `printf` direct.
Fiecare titlu de mai jos are un snippet cu același nume în `S1_cod.asm`.

---

## [BIT] Par sau impar
**Ce se cere:** un număr e par sau impar.
**Pași:** paritatea = bitul 0. Testezi bitul 0 cu `test`. Dacă e 0 → par, dacă e 1 → impar.
**Verificare:** afișezi mesajul potrivit; testezi cu un număr par și unul impar.

## [BIT] Semn / pozitiv-negativ
**Ce se cere:** un număr cu semn e pozitiv sau negativ.
**Pași:** semnul stă în bitul cel mai semnificativ (bit 63 la qword, bit 31 la dword).
Compari cu 0: `jg` (strict pozitiv), `js` (negativ), `jns` (≥ 0).
**Verificare:** testezi cu o valoare pozitivă și una negativă.

## [BIT] Testarea unui bit anume (ex. al 2-lea bit)
**Ce se cere:** verifici dacă un bit de pe o poziție dată e 1.
**Pași:** "al 2-lea cel mai nesemnificativ bit" = bitul de index 1 (numerotare de la 0).
Folosești o mască (`test al, 0b10`) sau `bt val, 1` + `jc`.
**Verificare:** numeri câte elemente dintr-un vector îndeplinesc condiția.

## [BIT] Numărul de biți setați (popcount)
**Ce se cere:** câți biți de 1 conține un număr.
**Pași:** varianta sigură = buclă: cât timp valoarea ≠ 0, shift dreapta cu 1, iar dacă
bitul ieșit (CF) e 1, incrementezi contorul. Alternativ `popcnt rcx, rax` dacă e permis.
**Verificare:** afișezi contorul; verifici pe o valoare cu biți cunoscuți.

## [BIT] Numărul de biți NEsetați
**Ce se cere:** câți biți de 0 conține un număr pe o lățime dată (ex. 64).
**Pași:** biți 0 = lățime − popcount. Deci `64 - popcount(x)` pentru un qword.
**Verificare:** popcount + biți-zero trebuie să dea 64.

## [BIT] Paritatea numărului de biți setați
**Ce se cere:** numărul de biți de 1 e par sau impar (ex. „Good/Bad number").
**Pași:** faci popcount, apoi testezi bitul 0 al contorului.
**Verificare:** afișezi mesajul; testezi pe un număr cu număr par și unul impar de biți.

## [BIT] Putere a lui 2
**Ce se cere:** un număr e putere a lui 2.
**Pași:** un număr > 0 e putere a lui 2 dacă `x & (x-1) == 0` (are un singur bit setat).
Atenție: 0 NU e putere a lui 2, tratează-l separat.
**Verificare:** testezi pe 8 (da), 6 (nu), 0 (nu).

## [BIT] Zerouri consecutive de la coadă (trailing zeros)
**Ce se cere:** câți biți 0 sunt la finalul (dreapta) numărului.
**Pași:** buclă: cât timp bitul 0 e 0, incrementezi contorul și faci shift dreapta.
Te oprești la primul bit 1. Alternativ `bsf`/`tzcnt`.
**Verificare:** pentru 0b...1000 trebuie să dea 3.

## [BIT] MSB setat + condiție pe octeți (număr „greu")
**Ce se cere:** MSB e setat ȘI numărul din octeții 3+4 concatenați > 255 (little-endian).
**Pași:** MSB cu `test`/`js`. Octeții „din mijloc" îi iei citind dword-ul potrivit și
izolând partea cerută cu shift + mască. Compari pe dword cu restul octeților zeroizați.
**Verificare:** afișezi mesajul; testezi cu o valoare grea și una ușoară.

## [ROT] ROT13 / cifru Caesar cu deplasare N
**Ce se cere:** rotești fiecare literă a-z (sau A-Z) cu N poziții, cu wrap.
**Pași:** pentru fiecare caracter, dacă e în interval: `c = (c - 'a' + N) % 26 + 'a'`.
Modulo îl faci cu o comparație: dacă rezultatul ≥ 26, scazi 26. Caracterele care nu-s
litere rămân neschimbate.
**Verificare:** ROT13 aplicat de două ori dă șirul original.

## [VEC] Element unic printre duplicate (o singură buclă)
**Ce se cere:** într-un vector unde toate apar de 2 ori afară de unul, găsești unicul.
**Pași:** XOR între toate elementele. Perechile se anulează (`a ^ a = 0`), rămâne unicul.
**Verificare:** afișezi rezultatul; verifici manual pe un vector mic.

## [VEC] Inversare vector in-place
**Ce se cere:** inversezi ordinea elementelor fără vector auxiliar.
**Pași:** doi pointeri, unul la început, unul la sfârșit. Faci swap și îi apropii până
se întâlnesc.
**Verificare:** afișezi vectorul înainte și după.

## [VEC] Extragere valoare din mijlocul unui vector de 8 octeți
**Ce se cere:** întregul cu semn „din mijloc" al unui vector `weird` de 8 octeți.
**Pași:** pentru un dword din mijloc, offsetul e 2 (rămân 2 octeți înainte, 2 după).
Citești `[weird+2]`. Pentru numărul pe 64 de biți citești `[weird]` (little-endian).
**Verificare:** afișezi valoarea în zecimal/hex și compari cu octeții din `.rodata`.

## [MAT] Matrice liniarizată NxN
**Ce se cere:** parcurgi/aduni pe o matrice stocată liniar, unde `(i,j)` e la `i*N+j`.
**Pași:**
- linie cu linie: două bucle, `i` și `j`, index = `i*N+j`.
- suma pe linie: bucla interioară acumulează.
- diagonala principală: elementele `(i,i)`, adică index `i*N+i`.
- max pe coloană: fixezi `j`, parcurgi `i`, ții maximul.
**Verificare:** afișezi rezultatele; verifici pe o matrice 3x3 cunoscută.

## [ADR] Adresa unei instrucțiuni / a stivei
**Ce se cere:** afișezi adresa primei instrucțiuni din `main`, sau adresa stivei.
**Pași:** adresa etichetei cu `lea rax, [rel main]`, afișezi cu `%p`. Adresa stivei
apelantului = `rbp` (sau `[rbp]` pentru rbp-ul apelantului). Afișezi cu `%p`.
**Verificare:** adresa e diferită la fiecare rulare (ASLR) — asta e normal.
