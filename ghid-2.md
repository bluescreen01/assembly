# Subiectul 2 — Ghid (funcții proprii + libc + recursivitate)

Afișare OBLIGATORIU cu `printf()`. Reține convenția din `00_START_convention.md`.
Fiecare titlu are un snippet cu același nume în `S2_cod.asm`.

**Regula de aur a recursivității în assembly:** orice valoare de care ai nevoie DUPĂ
apelul recursiv (argumentul, un rezultat parțial) trebuie salvată într-un registru
callee-saved (`rbx`, `r12`...) sau pe stivă, fiindcă apelul strică registrele caller-saved.

---

## [FUN] Sumă vector — iterativ
**Ce se cere:** `int sum(int *arr, int n)` — suma elementelor.
**Pași:** acumulator în `eax`, index în `ecx`. Cât timp `index < n`, aduni `arr[index]`
(adresat cu `[rdi + rcx*4]`), incrementezi. La final returnezi `eax`.
**Verificare:** apelezi cu un vector cunoscut, afișezi rezultatul cu `%d`.

## [FUN] Sumă vector — recursiv
**Ce se cere:** `int sum_rec(int *arr, int n)`.
**Pași:** caz de bază `n <= 0` → returnezi 0. Altfel: salvezi `arr` și `n`, apelezi
`sum_rec(arr, n-1)`, apoi aduni `arr[n-1]` la rezultat.
**Verificare:** trebuie să dea același rezultat ca varianta iterativă.

## [FUN] Putere — iterativ / recursiv
**Ce se cere:** `int pow(int a, int b)` = a^b.
**Pași iterativ:** rezultat = 1, înmulțești cu `a` de `b` ori.
**Pași recursiv:** caz de bază `b == 0` → 1. Altfel `a * pow_rec(a, b-1)`. Salvezi `a`
înainte de apel.
**Verificare:** pow(3,4) = 81.

## [FUN] Fibonacci recursiv
**Ce se cere:** `int fibo(int n)` — al n-lea Fibonacci recursiv.
**Pași:** caz de bază `n < 2` → returnezi `n`. Altfel salvezi `n`, apelezi `fibo(n-1)`,
salvezi rezultatul (într-un callee-saved!), apelezi `fibo(n-2)`, aduni cele două.
**Verificare:** fibo(7) = 13.

## [FUN] Valoare absolută
**Ce se cere:** `int myabs(int x)`.
**Pași (branchless):** copiezi x, faci `sar` cu 31 (obții masca de semn: tot 1 dacă
negativ, 0 altfel), XOR cu masca, apoi scazi masca. Sau simplu: dacă e negativ, `neg`.
**Verificare:** myabs(-5) = 5, myabs(5) = 5.

## [FUN] Putere a lui 4
**Ce se cere:** `int ispow(int n)` — 1 dacă n e putere a lui 4.
**Pași:** n > 0, e putere a lui 2 (`n & (n-1) == 0`), ȘI bitul setat e pe poziție pară
(indexul dat de `bsf` e par). Puterile lui 4 au bitul pe pozițiile 0,2,4...
**Verificare:** ispow(16)=1, ispow(8)=0, ispow(4)=1.

## [LIBC] strlen + printf
**Ce se cere:** aloci un șir, afișezi șirul și lungimea.
**Pași:** `lea rdi, [rel str]`, `call strlen` → lungimea în `rax`. Afișezi cu `printf`.
**Verificare:** compari cu numărul de caractere din șir.

## [LIBC] malloc + strcpy + free
**Ce se cere:** aloci dinamic, copiezi un șir, îl afișezi, eliberezi.
**Pași:** `mov edi, <dim>` + `call malloc` → pointer în `rax` (salvează-l în `rbx`!).
Verifici NULL. `strcpy(rbx, sursa)`. Afișezi. La final `free(rbx)`.
**Verificare:** afișezi șirul copiat; `valgrind ./main` nu raportează leak-uri.

## [LIBC] memset + memcpy
**Ce se cere:** umpli o zonă cu un octet, apoi copiezi un șir peste.
**Pași:** `memset(ptr, valoare, count)`. `memcpy(dst, src, len)`. Ordinea argumentelor:
rdi=dst, rsi=valoare/src, rdx=count/len.
**Verificare:** afișezi primii octeți cu `%llx` / afișezi șirul copiat.

## [FUN] Număr de apariții ale unui caracter (numc)
**Ce se cere:** `numc(char *s, char c)` — de câte ori apare `c`.
**Pași:** parcurgi până la `\0`, compari fiecare caracter cu `c` (e în `sil`),
incrementezi contorul la potrivire.
**Verificare:** afișezi contorul pentru un caz cunoscut.

## [LIBC] Inversare majuscule/minuscule (islower/isupper/tolower/toupper)
**Ce se cere:** transformi minusculele în majuscule și invers.
**Pași:** pentru fiecare caracter, apelezi `islower`; dacă da → `toupper`; altfel
`isupper`; dacă da → `tolower`. Salvezi rezultatul înapoi în șir. Atenție: reîncarci
caracterul în `edi` înainte de fiecare apel (apelurile strică registrele).
**Verificare:** afișezi șirul rezultat.

## [FUN] lowercase manual (fără libc)
**Ce se cere:** transformi majusculele în minuscule.
**Pași:** pentru fiecare caracter, dacă e între 'A' și 'Z', aduni 32. Restul rămâne.
**Verificare:** afișezi șirul.

## [FUN] Numărare cuvinte
**Ce se cere:** `int count_words(char *s)` — cuvinte separate de spații.
**Pași:** flag „sunt în cuvânt". Când dai de un caracter ≠ spațiu și flag-ul e 0,
incrementezi numărul de cuvinte și setezi flag-ul. Când dai de spațiu, resetezi flag-ul.
**Verificare:** "a b c" → 3.

## [LIBC] Verificare substring (strstr)
**Ce se cere:** un șir conține un `needle`? → YES/NO.
**Pași:** `strstr(haystack, needle)` → pointer sau NULL. Testezi `rax` cu 0.
**Verificare:** afișezi YES/NO.

## [FUN] Pointer la funcție — map
**Ce se cere:** `map(int *buf, int len, void (*f)(int*))` — aplică `f` pe fiecare element.
**Pași:** salvezi buf, len, f în callee-saved. Buclă: pentru fiecare i, pui în `rdi`
adresa `&buf[i]` și faci `call` pe registrul cu `f`. `call r13` apelează funcția.
**Verificare:** afișezi bufferul după map.

## [BITSET] Inserare / cardinal
**Ce se cere:** setezi bitul x într-un bitset; numeri câte elemente conține.
**Pași:** octetul = `x / 8` (shift dreapta 3), bitul în octet = `x % 8` (AND 7).
Setezi cu `or [bitset + octet], (1 << bit)`. Cardinal = popcount pe toți octeții.
**Verificare:** afișezi octetul modificat / cardinalul.

## [PARSE] MM:SS în secunde
**Ce se cere:** `time_to_seconds("MM:SS")` → total secunde.
**Pași:** minute = (s[0]-'0')*10 + (s[1]-'0'); secunde = (s[3]-'0')*10 + (s[4]-'0');
total = minute*60 + secunde. (Caracterul ':' e la index 2.)
**Verificare:** "01:30" → 90.

## [TIME] Măsurare timp (get_nano)
**Ce se cere:** compari care din două funcții e mai rapidă.
**Pași:** `call get_nano` → t0 (salvează în rbx). Apelezi funcția. `call get_nano` → t1.
Durata = t1 - t0. Repeți pentru a doua funcție și compari.
**Verificare:** afișezi care a fost mai rapidă.
