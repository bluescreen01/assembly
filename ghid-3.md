# Subiectul 3 — Ghid (reverse engineering + Makefile + linking)

Nu scrii algoritmi. Inspectezi binare ca să afli ce input/argument declanșează un mesaj,
și repari/extinzi build-ul. Mai jos: comenzile utile + rețetele de Makefile.

---

## Pasul 0 — mereu începe cu astea

```bash
file binar                 # ce fel de fișier e (ELF 32/64, PIE, stripped?)
strings binar              # toate șirurile din binar - vezi mesajele-țintă
strings binar | grep -i cuvant     # cauți un mesaj anume
```

## Inspecția codului (disassembly)

```bash
objdump -d binar | less                 # disassembly AT&T
objdump -d -M intel binar | less        # sintaxă Intel (mai ușor de citit)
objdump -d binar | grep -A20 '<main>:'  # doar funcția main
nm binar                                # simbolurile (funcții, variabile)
readelf -a binar | less                 # tot ELF-ul (secțiuni, simboluri)
```

Ce cauți în disassembly:
- un `cmp`/`test` urmat de un salt condiționat care duce la mesajul bun vs cel rău;
- un apel la `strcmp`/`strncmp` (înseamnă că se compară inputul cu o parolă);
- valoarea cu care se compară `argc` (câte argumente trebuie să dai);
- un `xor`/mască aplicată înainte de afișare (mesaj „criptat").

## Vezi apelurile de bibliotecă în timp real (foarte util pentru parole)

```bash
ltrace ./binar             # arată strcmp("input", "PAROLA") - parola apare direct!
ltrace ./binar input
strace ./binar             # apeluri de sistem (citiri, scrieri)
```

## Rularea cu input / argumente / mediu

```bash
./binar arg1 arg2                  # argumente în linia de comandă (argv)
./binar $(python3 -c 'print("A"*5)')   # argument generat
echo "parola" | ./binar            # input pe stdin
./binar <<< "parola"               # idem, here-string
VAR=valoare ./binar                # variabilă de mediu
./binar; echo $?                   # vezi codul de retur
```

## gdb — când trebuie să inspectezi la runtime

```bash
gdb ./binar
(gdb) break main
(gdb) run
(gdb) disassemble               # codul funcției curente
(gdb) info registers            # toate registrele
(gdb) x/s adresa                # afișează șirul de la o adresă
(gdb) x/20xb adresa             # 20 octeți în hex
(gdb) print (int *)C            # HINT din subiect: adresa lui argc
(gdb) continue
```

Trucuri gdb pentru a „forța" o ramură:
```bash
(gdb) set $rax = 1              # schimbi o valoare de retur
(gdb) jump *adresa             # sari direct la instrucțiunea dorită
(gdb) set var variabila = valoare
```

---

## Rețete de Makefile

### Compilare NASM 64-bit + link cu gcc
```make
NASM = nasm
ASMFLAGS = -f elf64 -g
CC = gcc
CFLAGS = -no-pie -g

%.o: %.asm
	$(NASM) $(ASMFLAGS) $< -o $@

prog: prog.o
	$(CC) $(CFLAGS) prog.o -o prog
```

### Link cu un fișier obiect dat (ex. `c.o`, `msg.o`, `castle.o`)
Scrii un `main.c`/`main.asm` care declară funcția ca externă și o apelează:
```c
// main.c
extern void print_message(void);   // sau semnătura corectă din enunț
int main(void) { print_message(); return 0; }
```
```make
prog: main.c msg.o
	gcc -no-pie main.c msg.o -o prog
```
Dacă nu știi semnătura funcției din `.o`, o afli cu `nm msg.o` și `objdump -d msg.o`.

### Schimbarea entry-point-ului (main -> entry)
Funcția de start nu mai e `main`, ci `entry` din `entry.o`:
```make
prog: entry.o
	gcc -no-pie -nostartfiles -e entry entry.o -o prog
```
- `-e entry` setează punctul de intrare;
- `-nostartfiles` evită codul de start al libc care ar apela `main`.
Alternativ, direct la linker: `-Wl,-e,entry`.

### Două binare din ACELAȘI sursă, cu comportament diferit (`-D`)
Sursa e nemodificată; folosești un define de preprocesor care activează un `#ifdef`:
```make
main1: main.c
	gcc -DPRINT_MSG main.c -o main1     # va afișa mesajul

main2: main.c
	gcc main.c -o main2                 # nu-l afișează
```
Funcționează dacă în `main.c` mesajul e sub `#ifdef PRINT_MSG`. Dacă ți se cere să
„adaugi în main.c", adaugi tu acel `#ifdef`.

### Obținerea unui mesaj gen „It works a=5!" fără a modifica logica
De obicei trebuie doar să definești o macro-valoare la compilare:
```make
main1: main.c
	gcc -DA=5 main.c -o main1
main2: main.c
	gcc -DA=5 main.c -o main2
```

---

## Tipare frecvente de „declanșare a mesajului"

| Ce vezi în binar | Ce faci |
|------------------|---------|
| `strcmp(input, "xyz")` | dai `xyz` ca argument/input (vezi cu `ltrace`) |
| `cmp argc, N` | dai exact `N-1` argumente pe lângă numele programului |
| `cmp argv[1], valoare` | dai argumentul potrivit |
| `getenv("VAR")` + compare | rulezi cu `VAR=valoare ./binar` |
| `xor`/mască pe un șir | mesajul e „criptat"; îl decodezi mental sau cu gdb după decriptare |
| verificare cod retur | rulezi și verifici cu `echo $?` |

## Reguli din enunț de respectat
- „Fără a modifica fișierul X" → soluția e în Makefile sau în modul de rulare, nu în X.
- „Executabilul trebuie să conțină și codul din file.c" → file.c trebuie linkat, nu ocolit.
- „Adăugați o regulă în Makefile" → nu strici regulile existente, adaugi una nouă.
