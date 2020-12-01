- [Installatie](#installatie)

# Installatie

Om xv6 te debuggen, zullen we gebruik maken van [GDB][gdb].
Helaas is er heeft Ubuntu standaard geen packages voor de RISC-V versie van GDB en zullen we deze handmatig moeten installeren.
Download daarvoor het bestand `gdb-riscv64-unknown-elf_10.1-1_amd64.deb` van Toledo (onder "Documenten", "Oefenzittingen") en voer de volgende commando's uit in een terminal:

```shell
cd Downloads
sudo dpkg -i gdb-riscv64-unknown-elf_10.1-1_amd64.deb
```

Als je geen foutmeldingen kreeg, zou de RISC-V versie van GDB nu geïnstalleerd moeten zijn.
Check dit met het volgende commando:

```shell
riscv64-unknown-elf-gdb -v
```

Als alles goed is gegaan, zou dit onder andere het volgende moeten printen: "GNU gdb (GDB) 10.1".

Voor nu het volgende commando uit om GDB te configureren:

```shell
echo "set auto-load safe-path /" > ~/.gdbinit
```

Ga nu weer naar de `xv6-riscv` directory in een terminal en voer het volgende commando uit:

```shell
make qemu-gdb
```

Dit start xv6 met ondersteuning voor het gebruik van GDB.
De kernel begint niet uit te voeren voordat we er een debugger aan verbinden.
Open hiervoor een andere terminal en ga daarin ook naar de `xv6-riscv` directory.
Start de debugger nu:

```shell
riscv64-unknown-elf-gdb
```

De output zou er als volgt moeten uitzien (`...` betekent oninteressante output):
<pre>
GNU gdb (GDB) 10.1
...
0x0000000000001000 in ?? ()
(gdb)
</pre>

We kunnen nu commando's ingeven achter de `(gdb)`-prompt.

Dit zijn enkele belangrijke commando's:
- `continue` (`c`): ga verder met uitvoeren.
  Dit kan gebruikt worden om xv6 te starten als het nog niet aan het uitvoeren is of om de uitvoering na een breakpoint verder te zetten.
- `break symbol` (`b`): zet een breakpoint op het adres van `symbol`.
  Als dit adres wordt uitgevoerd, zal de processor stoppen en kan je in GDB commando's uitvoeren.
- `backtrace` (`bt`): print een backtrace uit.
- `quit` (`q`, <kbd>CTRL</kbd> + <kbd>D</kbd>): sluit GDB af.

De commando's worden kort getoond in de volgende GDB-sessie:

<pre>
0x0000000000001000 in ?? ()
(gdb) b fork
Breakpoint 1 at 0x80001ee0: file kernel/proc.c, line 267.
(gdb) c
Continuing.
[Switching to Thread 1.2]

Thread 2 hit Breakpoint 1, fork () at kernel/proc.c:267
267     {
(gdb) bt
#0  fork () at kernel/proc.c:267
#1  0x0000000080002cf8 in sys_fork () at kernel/sysproc.c:29
#2  0x0000000080002c6c in syscall () at kernel/syscall.c:142
#3  0x0000000080002956 in usertrap () at kernel/trap.c:67
#4  0x0000000000000050 in ?? ()
(gdb) q
Detaching from program: , process 1
Ending remote debugging.
[Inferior 1 (process 1) detached]
</pre>

# Breakpoints

Een _breakpoint_ kan gebruikt worden om GDB automatisch te laten stoppen wanneer er een bepaald punt in het programma bereikt wordt.
Wanneer GDB stopt, kan je weer commando's ingeven in de prompt.

Er zijn meerdere manieren om de locatie van een breakpoint te geven via het `break` (`b`) commando:
- `break symbol`: Zet een breakpoint wanneer het geven `symbol` (de naam van een functie) bereikt wordt.
  Bijvoorbeeld: `break kalloc`.
- `break filename:linenum`. Zet een breakpoint op een bepaalde lijn in een file.
  Bijvoorbeeld: `break kernel/exec.c:52`.

Je kan meerdere breakpoints tegelijkertijd zetten en GDB zal stoppen wanneer één van de breakpoint bereikt word.
Nog een aantal handige commando's:
- `info breakpoints`: Toon alle breakpoints.
  Elke breakpoint heeft een identifier (`Num`) waarmee je ernaar kan verwijzen in andere commando's.
- `enable/disable num`: Zet een breakpoint tijdelijk aan of uit.
- `delete num`. Verwijder een breakpoint permanent.

Als je klaar bent met het inspecteren van het programma tijdens een breakpoint, kan je de uitvoering verderzetten met `continue` (`c`).
GDB zal dan uitvoeren tot aan het volgende breakpoint.

Je kan GDB op elk moment handmatig stoppen door <kbd>CTRL</kbd>+<kbd>C</kbd> te typen.

[![asciicast](img/breakpoints.svg)](https://asciinema.org/a/376454)

# Stepping

In plaats van het programma verder te laten uitvoeren tot het volgende breakpoint met `continue`, kan je het ook in kleinere stappen laten uitvoeren:
- `step` (`s`): Voer uit tot de volgende regel C-code.
  Dit zal functie oproepen binnengaan.
- `next` (`n`): Voor uit tot de volgende regel C-code _binnen de huidige functie_.
  Functie oproepen worden dus uitgevoerd tot ze terugkeren in de huidige functie.
- `finish` (`fin`): Voer de huidige functie uit tot net na de return.
  De eventuele returnwaarde wordt afgeprint.

[![asciicast](img/stepping.svg)](https://asciinema.org/a/376462)

[gdb]: https://www.gnu.org/software/gdb/
