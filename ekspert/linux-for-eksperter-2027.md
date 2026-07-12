# Linux for Eksperter 2027

**Tredje bok i serien – for deg som vil mestre systemet på kildenivå og drifte egen infrastruktur som en proff.**

*Samlet utgave – alle kapitler i én fil. Generert 2026-07-12.*

## Innhold

- Forord
- 1. Kjernen
- 2. Prosesser, signaler og cgroups
- 3. Minne og ytelse
- 4. Røntgensyn: strace, lsof og perf
- 5. Lagring på alvor
- 6. Python for systemadministratorer
- 7. Ansible
- 8. Egen Git-server og CI/CD hjemme
- 9. 🟡 NixOS
- 10. Nettverk på ekspertnivå
- 11. Reverse proxy og TLS overalt
- 12. Overvåking og varsling
- 13. Containere på dypet
- 14. Virtualisering og hjemmelab-arkitektur
- 15. Sikkerhet på alvor
- 16. Feilsøking uten sikkerhetsnett
- 17. Den dagen skrivebordet ikke starter
- 18. Pakk og del programvaren din
- 19. 🔴 Linux From Scratch
- 20. Gi tilbake
- 21. Å fortsette reisen
- Bonus: FAQ for eksperter
- Vedlegg A: Utvidet hurtigreferanse
- Vedlegg B: Ordliste for eksperter
- Vedlegg C: Referansearkitektur for hjemmelabben
- Vedlegg D: Mesterprøven

---

# Forord

**Tredje bok i serien – for deg som forstår systemet ditt og nå vil mestre det på kildenivå, og drifte egen infrastruktur som en proff.**

Du har fullført *Linux for Viderekommende* (eller tilsvarende): du er hjemme i terminalen, skripter med god samvittighet, kjører tjenester på en hjemmeserver og bruker Git daglig. Denne boken tar det siste steget – fra «forstår Linux» til «kan forklare, feilsøke og bygge alt».

**Hva denne boken IKKE er:** sertifiseringspensum (LPIC/RHCSA), enterprise-drift for store organisasjoner, eller programmeringskurs. Det er fortsatt en bok for entusiasten – men nå med hele maskineriet åpent.

**Den røde tråden:** Gjennom boken bygger du ut **hjemmelabben** din til et komplett, overvåket, automatisert og herdet miljø – beskrevet som kode, slik at hele oppsettet kan gjenskapes med én kommando.

Velkommen til den tredje boken. Du har allerede oppdaget at Linux ikke er et operativsystem du «kan», men et verksted du blir stadig bedre kjent med. Nå skal vi åpne alle dørene.

«Ekspert» betyr her ikke at du husker alle flagg til `tar` eller kan sitere `man`-sider utenat. Det betyr at du har utviklet en intuisjon for hvordan systemet henger sammen, og at du kan *resonnere* deg frem til løsninger på problemer du aldri har sett før. Du skal lære å stille de riktige spørsmålene til maskinen – og å forstå svarene.

Boken er bygget rundt **hjemmelabben** din. Den starter som et par maskiner og noen containere, og vokser gjennom kapitlene til et fullverdig, overvåket og selvhelbredende miljø. Det viktigste er likevel ikke sluttresultatet, men alle beslutningene og forståelsen du tilegner deg underveis. Fra kapittel 7 og utover blir alt du gjør versjonshåndtert og automatisert – målet er at du på siste side kan gjenskape hele labben med én enkelt kommando.

Vi holder oss til den samme vennlige tonen som i de tidligere bøkene. Du vil finne «Prøv selv»-oppgaver, og utfordrende partier er merket 🟡 (anbefalt smak) eller 🔴 (valgfri finale). Kapitlene følger en fast mal, men prinsippene går foran installasjonsmanualer. Verktøy endrer seg, forståelse består.

Tre nye grep skiller denne boken fra de forrige:

**«Mål først».** Fra nå av gjelder én regel for all tuning: *aldri juster noe du ikke har målt*. Entusiasten setter `vm.swappiness=10` fordi en bloggpost sa det; eksperten måler før og etter. Verktøyene får du underveis: `fio` for disk, `iperf3` for nettverk, `hyperfine` for kommandoer.

**«Anatomi av en hendelse».** Hver del avsluttes med en ekte feilsøkingshistorie, fortalt slik den utspiller seg: symptom → hypoteser → kommandoer → funn → lærdom. Verktøy kan slås opp – det er resonneringen som gjør deg til ekspert.

**Mesterprøven.** Bakerst i boken (vedlegg D) venter ti oppgaver der noe *er* galt i labben din – plantet med vilje av en sabotasje-playbook. Ingen fasit før du har prøvd.

## Hva du trenger

| Nivå | Utstyr | Holder til |
|------|--------|-----------|
| **Minimum** | Én maskin med 16 GB RAM og VM-er | Alt i boken – hele labben kan være virtuell |
| **Anbefalt** | PC + én labbmaskin (gammel PC eller Pi) | Mer realistisk nettverk og drift |
| **Luksus** | Egen hypervisor-vert + Pi + NAS | Referansearkitekturen i vedlegg C |

Du klarer deg altså med maskinen du har. Tidsmessig: Del 1 og 2 er kveldsstoff (2–4 timer per kapittel), Del 3 er helgeprosjekter, og Del 4 tar du når krisene (eller nysgjerrigheten) melder seg. Labbens byggetrinn kapittel for kapittel finner du i vedlegg C.

**Testet på:** Kommandoer og pakkenavn i boken er verifisert mot Debian 13 og Ubuntu 24.04-baserte systemer (inkludert Linux Mint 22). Der distroene skiller lag, sier teksten fra. Bruker du noe annet, er avvikene små – og nå har du verktøyene til å finne dem. Vi skriver `apt` hele veien; på Fedora heter det `dnf`, på Arch `pacman` – på ditt nivå er den oversettelsen triviell (distro-safarien i bok 2 ga deg kartet).

Nå er det på tide å se hva som virkelig foregår under panseret.

---

# 1. Kjernen – motoren du aldri ser

*Del 1: Systemet på dypet*

Linux-kjernen er den første koden som kjører etter bootloaderen, og den siste som gir slipp når maskinen slås av. For de fleste er den et usynlig lag – du merker den bare når noe går galt. I dette kapittelet gjør vi kjernen synlig.

## 1.1 Kjernemoduler

Kjernen er modulær. Drivere og filsystemstøtte lastes ved behov. Med `lsmod` ser du hvilke moduler som er aktive, og med `modprobe` kan du laste eller fjerne dem manuelt. Moduler ligger under `/lib/modules/$(uname -r)/`. Når du kobler til en USB-disk, laster udev automatisk `usb-storage` og `ext4` (eller hva disken er formatert med). Du kan svarteliste moduler du ikke vil ha, for eksempel en problematisk Wi-Fi-driver, via `/etc/modprobe.d/`.

**Prøv selv:** Finn ut hvilken modul som driver nettverkskortet ditt: `lspci -k` viser driveren i bruk, sammenlign med `lsmod | grep <driver>`. Øv deretter på å laste og fjerne en *ufarlig* modul: `sudo modprobe pcspkr && sudo modprobe -r pcspkr` (PC-høyttaleren), og se hendelsene i `sudo dmesg | tail`. Prøver du deg på lydmodulen (`sudo modprobe -r snd_hda_intel`) får du sannsynligvis «Module is in use» – PipeWire holder den. Den feilmeldingen er i seg selv lærerik: kjernen nekter å fjerne moduler noen er avhengige av.

## 1.2 Kjerneparametere og sysctl

Kjernens oppførsel kan justeres i sanntid via `/proc/sys/`. Verktøyet `sysctl` gir deg en ryddig vei til disse. Skriv `sysctl -a` og la deg fascinere – her finner du alt fra nettverksinnstillinger (`net.core.somaxconn`) til minnehåndtering (`vm.swappiness`). Ved å legge linjer i `/etc/sysctl.d/` kan du gjøre endringer permanente.

Typiske parametere du vil justere på en server:
- `vm.swappiness=10` – gjør at systemet holder seg unna swap så lenge som mulig.
- `net.ipv4.ip_forward=1` – slår på ruting.
- `kernel.sysrq=1` – aktiverer Magic SysRq for nødgjenoppretting.

## 1.3 GRUB og oppstartsparametere

Bootloaderen (GRUB) sender kommandolinjeargumenter til kjernen. Du har kanskje brukt `nomodeset` for å få grafikk til å fungere. Disse parameterne finnes i `/etc/default/grub` under `GRUB_CMDLINE_LINUX_DEFAULT`. Etter endring kjører du `update-grub`. Med `quiet` og `splash` fjerner du mesteparten av oppstartsmeldingene, men som ekspert lar du dem gjerne være på for å fange opp feil.

## 1.4 Kjernepanikk – når alt stopper

En kjernepanikk er kjernens måte å si «jeg kan ikke fortsette uten å risikere datakorrupsjon». Da fryser skjermen med en dump av registre og en peker til instruksjonen der feilen oppsto. For å tolke panikken trenger du å dechiffrere RIP (instruksjonspekeren) mot kjernens symboltabell. I praksis er det ofte en defekt maskinvare eller en bug i en driver.

🔴 **Valgfri finale: Kompiler din egen kjerne.** Dette er mer dannelsesreise enn nødvendighet. Du laster ned kildekoden fra kernel.org, konfigurerer med `make menuconfig`, bygger med `make -j$(nproc)` og installerer. Når du starter opp med en kjerne du selv har satt sammen, forsvinner mye av mystikken – og du får en uvurderlig forståelse av hvilke komponenter som faktisk utgjør en kjerne.

---

# 2. Prosesser, signaler og cgroups

*Del 1: Systemet på dypet*

**I dette kapittelet lærer du:**

- Hva en prosess *egentlig* er – og hvordan du leser alt om den i `/proc`.
- Signaler: språket du snakker til prosesser med, og hvorfor SIGKILL bør være siste utvei.
- Prosesstreet og tilstandene – sannheten om zombier og de udrepelige.
- Prioriteter med `nice` og `ionice` – og hvorfor de bare er ønsker.
- Cgroups: hvordan systemd (og containere) setter grenser prosesser ikke kan rømme fra.
- To systemd-triks du bruker resten av boken: drop-ins og template-units.

---

En prosess er et program i utførelse – så langt nybegynnerdefinisjonen. Eksperten trenger mer: hvordan prosesser blir til, hvordan de dør, hvordan du snakker til dem mens de lever, og hvordan du setter grenser de ikke kan rømme fra. Det siste er grunnmuren under containerne i kapittel 13 – og under hendelsen som avslutter del 1, da OOM-killeren valgte feil offer.

## 2.1 Hva en prosess egentlig er

Hver prosess har en **PID** og en forelder (**PPID**) – alle er etterkommere av PID 1 (systemd). Men det virkelig nyttige er at hele prosessens indre liv ligger åpent i `/proc/<PID>/` – «alt er en fil»-prinsippet fra kapittel 1 i praksis:

```bash
ls /proc/$$/            # $$ = PID-en til skallet du sitter i
cat /proc/$$/cmdline    # kommandolinjen den ble startet med
ls -l /proc/$$/cwd      # symlenke til arbeidskatalogen – akkurat nå
ls -l /proc/$$/fd       # alle åpne filer, sockets og pipes
grep -E 'State|VmRSS' /proc/$$/status   # tilstand og faktisk minnebruk
```

`/proc/<PID>/fd` er undervurdert i feilsøking: «hvilken fil skriver denne prosessen egentlig til?» besvares med én `ls -l`. (Verktøyet `lsof` fra kapittel 4 er i praksis en pen frontend til akkurat dette.)

Og når `ps aux` viser for mye: bygg din egen visning med `-o`:

```bash
ps -o pid,ppid,stat,ni,rss,cmd -p $$      # nøyaktig kolonnene du bryr deg om
ps -eo pid,ppid,stat,cmd --forest | less  # hele treet, med innrykk
```

## 2.2 Signaler – språket til prosesser

`kill` sender signaler, ikke nødvendigvis død. De du faktisk møter:

| Signal | Nr | Betydning | Kan fanges? |
|--------|----|-----------|-------------|
| SIGTERM | 15 | «Vennligst avslutt» – prosessen får rydde opp | Ja |
| SIGKILL | 9 | «Dø nå» – kjernen fjerner prosessen direkte | **Nei** |
| SIGHUP | 1 | «Linjen røk» – tjenester bruker den ofte til å laste konfig på nytt | Ja |
| SIGINT | 2 | Ctrl+C i terminalen | Ja |
| SIGSTOP / SIGCONT | 19/18 | Frys / tin (SIGSTOP kan heller ikke fanges) | Nei / Ja |
| SIGUSR1/2 | 10/12 | «Til fri bruk» – ber f.eks. `dd` om statusrapport | Ja |

(`kill -l` lister alle; `man 7 signal` forklarer dem.)

**Hvorfor rekkefølgen TERM → KILL betyr noe:** SIGTERM kan prosessen *fange* (med `trap`, som i skriptene dine fra bok 2) og bruke til å lukke filer, tømme buffere og logge ut av databasen. SIGKILL leveres aldri til prosessen – kjernen bare fjerner den: ingen opprydding, halvskrevne filer, etterlatte låsefiler. Derfor: `kill PID`, vent noen sekunder, *så* eventuelt `kill -9`. (systemd gjør nettopp dette ved `systemctl stop`: TERM, frist, så KILL.)

I praksis treffer du prosesser på navn, ikke nummer:

```bash
pgrep -a jellyfin                  # finn og vis – alltid pgrep FØR pkill
pkill -HUP nginx                   # be alle nginx-prosessene lese konfig på nytt
sudo systemctl kill -s HUP nginx   # samme, men presist avgrenset til tjenestens cgroup
```

Den siste er ekspertens variant: `systemctl kill` treffer *nøyaktig* prosessene som tilhører tjenesten – aldri en tilfeldig prosess som deler navn. (Og unngå å venne deg til `killall`: på enkelte andre Unix-systemer betyr den bokstavelig talt «drep alt».)

## 2.3 Prosesstreet, zombier og de udrepelige

`pstree -p` viser slektskapet. To skjebner er verdt å forstå:

**Foreldreløse** («orphans»): dør forelderen først, adopteres barnet av PID 1 (eller en *subreaper*, som en terminalmultiplekser). Helt udramatisk – det er slik nohup-jobber og tmux-økter overlever.

**Zombier** (`Z` i STAT-kolonnen): prosessen er *ferdig*, men forelderen har ikke hentet ut exit-koden ennå. En zombie bruker ingen CPU og intet minne – bare en rad i prosesstabellen. Den kan ikke drepes, for den er allerede død; det som hjelper, er at *forelderen* rydder opp (eller selv dør, så PID 1 overtar). Se en bli født og dø:

```bash
python3 -c 'import os,time
if os.fork() == 0: exit()      # barnet avslutter straks
time.sleep(15)'                # forelderen venter uten å hente exit-koden
# i en annen terminal, innen 15 sekunder:
ps -eo pid,stat,cmd | grep -w Z    # <defunct> – der er zombien
```

Etter 15 sekunder dør forelderen, og zombien forsvinner med den. Mange zombier fra samme program er en bug i programmet – ikke i Linux.

**De virkelig udrepelige** er ikke zombiene, men `D`-tilstanden: *uavbrytelig søvn*, nesten alltid venting på I/O (døende disk, hengt NFS-montering). En D-prosess ignorerer selv SIGKILL – den kan ikke avbrytes midt i et kjernekall. Ser du prosesser låst i D: slutt å skyte på prosessen og finn I/O-problemet (`iotop`, `dmesg` – kapittel 4 tar jakten videre).

Tilstandene du leser i `ps`: `R` kjører, `S` sover (normalt), `D` venter på I/O, `T` stoppet, `Z` zombie.

## 2.4 Nice og ionice – prioritet uten garantier

Før de harde grensene: de myke. `nice` senker CPU-prioriteten (0 er standard, 19 er «bakerst i køen»), `renice` endrer en kjørende prosess, og `ionice` gjør det samme for disk:

```bash
nice -n 19 tar czf backup.tar.gz ~/data   # start snilt
sudo renice 19 -p 4242                    # gjør en kjørende snill
ionice -c 3 -p 4242                       # I/O-klasse idle: disk kun når ingen andre vil ha den
```

Merk begrensningen: nice hjelper bare når det er *kamp* om ressursene, og garanterer ingenting. Til batchjobber er det ofte alt du trenger – hendelse #3 senere i boken løses med `IOSchedulingClass=idle`, som er systemds måte å si `ionice -c 3` på. Trenger du garantier, går du til cgroups.

## 2.5 Cgroups – harde grenser, målt forbruk

Control groups (cgroups v2) er kjernens mekanisme for å **begrense, måle og isolere** ressursbruk for *grupper* av prosesser. Systemd organiserer hele maskinen i et cgroup-tre – se det selv med `systemd-cgls`:

*Figuren viser: direktiver som `MemoryMax=` og `CPUQuota=` settes på grener og arves nedover i hierarkiet – nginx kan ikke rømme fra kvoten ved å starte flere workers.*

Grensene settes i unit-filer (eller drop-ins, se 2.6):

```ini
[Service]
MemoryHigh=400M     # myk grense: over denne bremses prosessen og minne gjenvinnes aggressivt
MemoryMax=500M      # hard grense: over denne → OOM-drap, men KUN innenfor denne cgroupen
CPUQuota=50%        # hard CPU-grense: maks en halv kjerne, uansett hvor ledig maskinen er
CPUWeight=50        # myk: halv vekt NÅR det er kamp om CPU (standard er 100)
```

Skillet myk/hard er ekspertkunnskapen her: `MemoryHigh` gir tjenesten en sjanse til å oppføre seg (og deg et utslag i metrikker), `MemoryMax` er giljotinen – men en *lokal* giljotin som beskytter resten av maskinen. Det er nøyaktig medisinen fra hendelse #1: fotoindekseringen fikk `MemoryHigh`, og databasen slapp å dø for dens synder.

Og alt kan *måles* mens det kjører – cgroups fører regnskapet uansett om du har satt grenser:

```bash
cat /sys/fs/cgroup/system.slice/ssh.service/memory.current   # faktisk bruk, akkurat nå
systemd-cgtop                                                # «top», men per cgroup
```

Engangsjobber trenger ikke unit-fil – `systemd-run` lager en midlertidig:

```bash
systemd-run --user -p CPUQuota=10% dd if=/dev/zero of=/dev/null
```

Dette er grunnfjellet containerne i kapittel 13 står på: en container er i bunn og grunn prosesser i en cgroup pluss namespaces. Forstår du treet over, har du allerede forstått halve containerteknologien.

## 2.6 To systemd-triks eksperter bruker daglig

**Overstyr uten å røre pakkens filer:** `sudo systemctl edit tjeneste` åpner en tom «drop-in»-fil i `/etc/systemd/system/tjeneste.service.d/`. Alt du skriver der overstyrer pakkens unit-fil – og overlever oppgraderinger. Det er slik du legger `MemoryMax=` på en tjeneste du ikke eier. `systemctl cat tjeneste` viser resultatet med alle drop-ins.

**Template-units:** En fil som heter `backup@.service` kan startes som `backup@dokumenter.service` og `backup@bilder.service` – `%i` i filen erstattes med det som står etter krøllalfaen. Én definisjon, mange instanser. Du møter mønsteret overalt: `getty@tty1`, `wg-quick@wg0`.

---

**Prøv selv:**

1. Utforsk ditt eget skall: `ls -l /proc/$$/fd` og `grep State /proc/$$/status`. Åpne en fil med `less` i et annet vindu og se den dukke opp under `fd`.
2. Kjør zombie-demoen fra 2.3 og se `<defunct>` komme og gå i `ps`.
3. Start `systemd-run --user -p CPUQuota=10% dd if=/dev/zero of=/dev/null`, se at `top` viser ~10 %, finn den i `systemd-cgls --user`, og les `memory.current` for den under `/sys/fs/cgroup/user.slice/`. Stopp med `systemctl --user stop <navnet du fikk>`.
4. 🟡 Test giljotinen trygt: `systemd-run --user --scope -p MemoryMax=100M stress --vm 1 --vm-bytes 200M` – jobben OOM-drepes *inne i sin egen cgroup* uten at maskinen merker noe. Se beviset med `journalctl --user -e`.
5. Send `SIGUSR1` til en kjørende `dd` (`pkill -USR1 dd`) og se den rapportere fremdrift – signaler er mer enn drap.

---

**Det viktigste fra dette kapittelet**

- `/proc/<PID>/` er prosessens åpne journal – `fd`, `status` og `cmdline` svarer på det meste.
- TERM før KILL, alltid: SIGKILL gir null opprydding. `systemctl kill` treffer presist.
- Zombier er ufarlige rader i en tabell; `D`-tilstand betyr «finn I/O-problemet, ikke skyt prosessen».
- nice/ionice er ønsker; cgroups er garantier. `MemoryHigh` bremser, `MemoryMax` halshugger – lokalt.
- Grenser settes på grener i cgroup-treet og arves nedover – det er dette containere er laget av.
- `systemctl edit` (drop-ins) og `navn@.service` (templates) er verktøy du bruker resten av boken.

---

# 3. Minne og ytelse – sannheten om «full RAM»

*Del 1: Systemet på dypet*

«Linux spiser opp alt minnet!» hører du nybegynnere si. Vi lærer deg å lese minnestatistikken riktig, og å forstå hva som egentlig foregår.

## 3.1 Page cache – gratis ytelse

Linux bruker ledig RAM som disk-cache. Når du leser en fil, kopieres den til minnet. Leser du den igjen, går det lynraskt. Skriver du, går data først til cache og skylles til disk senere (write-back). `free` viser «available»-minne, som er den reelle mengden ledig for nye prosesser – cache telles her som tilgjengelig fordi kjernen kan kaste den umiddelbart.

## 3.2 Swap og swappiness

Swap er en nødløsning når det fysiske minnet er fullt. Med `vm.swappiness` styrer du hvor aggressivt kjernen flytter sider ut til swap. En lav verdi (10) holder data i RAM så lenge som mulig. På SSD-baserte systemer kan du trygt bruke swap, men på eldre HDD-er gir det voldsom treghet. Moderne alternativ er `zram` – en komprimert swap i RAM, som gir mer effektivt minne uten å røre disk.

## 3.3 OOM-killeren

Når minnet er fullt og kjernen ikke kan frigjøre mer, må den velge et offer. Out-of-Memory-killeren (OOM) gir hver prosess en «badness-score» som i hovedsak følger minnebruken (root-prosesser får en liten rabatt). Du kan justere sårbarheten via `/proc/<PID>/oom_score_adj`: skalaen går fra `-1000` (full immunitet – sshd bruker den, så du ikke låses ute av en minnelekk server) til `+1000` (frivillig førsteoffer). En verdi som `-500` *reduserer* bare sannsynligheten – den gir ikke immunitet. Se gjeldende score med `cat /proc/<PID>/oom_score`. Historikken finnes i `dmesg`.

## 3.4 Verktøykassa

- `free -h` – enkel oversikt.
- `cat /proc/meminfo` – alle detaljer.
- `vmstat 1` – kontinuerlig oppdatering av minne, swap, I/O og CPU.
- `smem` – viser faktisk minnebruk per prosess (med delt bibliotek trukket fra).

**Prøv selv:** Kjør `free -h` og noter «available». Start en minnekrevende jobb, f.eks. `stress --vm 1 --vm-bytes 2G` (`sudo apt install stress`). Vent litt og se hvordan `available` synker, mens `buff/cache` kanskje minker. Avbryt med Ctrl+C, og se at minnet frigjøres igjen.

---

# 4. Røntgensyn: strace, lsof og perf

*Del 1: Systemet på dypet*

Når et program oppfører seg merkelig, vil du se hva det egentlig gjør. Disse tre verktøyene gir deg røntgensyn.

## 4.1 strace – systemkall i sanntid

`strace` fanger opp alle systemkall et program gjør. Du kan feste det til en kjørende prosess med `strace -p <PID>` eller starte et program under strace. Typisk scenario: en prosess henger. `strace -p $(pgrep nginx)` avslører kanskje at den venter på en `connect()` til en DNS-server som ikke svarer. Eller at `open()` feiler med `ENOENT` fordi en konfigurasjonsfil mangler.

**Prøv selv:** Kjør `strace ls -l /ingen/ting` og legg merke til `openat`-kallet som returnerer `-1 ENOENT`. Følg et nettverksprogram med `strace -e trace=network curl http://example.com` og se alle sokkelkall.

## 4.2 lsof – liste åpne filer

I Unix er alt filer. `lsof` viser alle åpne filer – sockets, pipes, regulære filer, enheter. Nyttige eksempler:
- `lsof -i :80` – hvilken prosess lytter på port 80.
- `lsof -p <PID>` – alt en bestemt prosess har åpent.
- `lsof /var/log/syslog` – hvem som skriver til loggfilen.

## 4.3 perf – CPU-profilering

`perf` er et kraftig verktøy for å måle ytelse. Du kan se hvilke funksjoner i kjernen eller i et program som bruker mest CPU:
`perf record -g ./mitt_program`
`perf report`
Du får en interaktiv rapport med flame graphs (med støtteverktøy). Dette er uvurderlig når du skal finne ut *hvorfor* noe er tregt.

`perf` ligger i pakken `linux-tools-common` (+ `linux-tools-$(uname -r)`) på Ubuntu/Mint, og som vanlig bruker må du kanskje senke `kernel.perf_event_paranoid` – en sysctl-parameter, akkurat som i kapittel 1.

## 4.4 🟡 bpftrace – røntgensyn på hele systemet samtidig

`strace` ser én prosess – og bremser den kraftig, fordi den stopper prosessen ved hvert systemkall. **eBPF** løser samme problem fra motsatt kant: i stedet for å stanse programmet og titte inn, laster du et bittelite program *inn i kjernen selv*, festet til et målepunkt (et systemkall, en kjernefunksjon, en nettverkshendelse). Kjernen kjører det hver gang punktet passeres og leverer tallene til deg – uten å stoppe noen. Før programmet slippes inn, bevises det matematisk av kjernens *verifier* at det ikke kan krasje eller henge; derfor er dette trygt selv i produksjon. Resultatet: du ser *hele* systemet samtidig, nesten gratis.

Verktøyet `bpftrace` gjør dette til én-linjere:

```bash
sudo apt install bpftrace
# Hvem åpner hvilke filer, akkurat nå, på hele maskinen?
sudo bpftrace -e 'tracepoint:syscalls:sys_enter_openat { printf("%s → %s\n", comm, str(args->filename)); }'
```

Pakken `bpfcc-tools` gir deg ferdige klassikere: `opensnoop` (alle filåpninger), `execsnoop` (alle nye prosesser – avslører hva et «magisk» skript egentlig starter) og `biolatency` (disk-latens som histogram). Dette er den moderne observabilitets-verktøykassen – og grunnen til at eksperter i 2027 sjeldnere trenger å gjette.

**Prøv selv:** Kjør `sudo execsnoop-bpfcc` i én terminal og `git status` i en annen. Se hvor mange prosesser én «enkel» kommando faktisk starter.

---

# 5. Lagring på alvor – RAID, LVM, btrfs og ZFS

*Del 1: Systemet på dypet*

Filsystemer og volumer er mer enn bare plass til filer. Dette kapittelet lærer deg å bygge fleksibel, selvhelbredende lagring.

## 5.1 Inoder, lenker og sletting

Hver fil har en inode – metadata (eier, rettigheter, tidsstempler) og pekere til datablokker. Et filnavn er bare en lenke til en inode. `ln` lager en ny lenke (hard link) – så lenge minst én lenke finnes, beholdes dataene. `rm` sletter kun lenken; inoden og blokkene fjernes først når siste lenke er borte og ingen prosess har filen åpen. Det er derfor du kan slette en loggfil mens en tjeneste fortsatt skriver til den – plassen frigjøres når tjenesten lukker filen.

```
 rapport.txt ──────┐
                   ├──► inode 5324 ──► [datablokker på disken]
 hardlenke.txt ────┘      │
                          └── eier, rettigheter, tidsstempler
 snarvei.txt ··· (symlenke: peker på NAVNET «rapport.txt», ikke inoden)

 rm rapport.txt  → inoden lever (hardlenke.txt holder den)
 rm hardlenke.txt → inoden frigjøres … når siste åpne filhåndtak lukkes
```

*Figuren viser: filnavn er bare lenker til en inode – dataene lever så lenge minst én lenke eller ett åpent filhåndtak finnes.*

## 5.2 Programvare-RAID med mdadm

RAID kan speile eller stripe data over flere disker. Med `mdadm` kan du lage RAID 1 (speiling) eller RAID 5/6 (paritet). Kombinert med LVM blir dette en solid og billig redundans. Slik lager du et speil:
```
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdc1
```

> **⚠️ RAID er ikke backup.** RAID beskytter mot *diskhavari* – ingenting annet. Sletter du en fil, speiles slettingen i sanntid til alle diskene. Ransomware krypterer alle kopiene samtidig. RAID gir oppetid; backup (kapittel og bok 2) gir angrefrist. Du trenger begge, og de løser hver sin jobb.

**Brannøvelse uten ekstra maskinvare:** Med loop-enheter kan du øve på diskhavari helt trygt:

```bash
# Lag to «disker» av filer, bygg et speil av dem
for i in 1 2; do truncate -s 500M disk$i.img; done
sudo losetup /dev/loop101 disk1.img && sudo losetup /dev/loop102 disk2.img
sudo mdadm --create /dev/md9 --level=1 --raid-devices=2 /dev/loop101 /dev/loop102

sudo mdadm --fail /dev/md9 /dev/loop101     # «disken» dør!
cat /proc/mdstat                            # [_U] – degradert, men dataene lever
sudo mdadm --remove /dev/md9 /dev/loop101   # ta ut den døde
sudo mdadm --add /dev/md9 /dev/loop101      # sett inn «ny» disk
watch cat /proc/mdstat                      # se gjenoppbyggingen live
```

Den dagen en ekte disk dør, har fingrene dine gjort dette før. Det er hele forskjellen på panikk og rutine.

## 5.3 LVM – fleksible volumer

Logical Volume Manager abstraherer bort de fysiske diskene. Du oppretter PV (physical volume), VG (volume group) og LV (logical volumes). Du kan utvide, krympe, ta snapshots og flytte data mens systemet kjører. Sammen med RAID får du en lagringsløsning som vokser med behovene dine.

## 5.4 Neste generasjon: btrfs og ZFS

Disse moderne filsystemene har innebygde snapshots, komprimering, sjekksummer og selvhelbredelse ved bitrot. btrfs er innebygd i Linux-kjernen og lett å ta i bruk. ZFS kommer via OpenZFS og er elsket for sin stabilitet og funksjonsrikdom. (Hvorfor er ikke ZFS i kjernen når alle elsker det? Lisensen: ZFS er CDDL, kjernen er GPL, og de fleste – men ikke alle – jurister mener de ikke kan blandes i samme kildetre. Ubuntus advokater er uenige og skipper ferdigbygde moduler; Debian holder seg på den sikre siden og bygger dem lokalt med DKMS hos deg; kjerneutviklerne vil ikke ta sjansen, spesielt ikke med Oracle som rettighetshaver. Ingen har fasit – det er derfor installasjonen føles ulik fra distro til distro.) Begge gir deg muligheten til å ta et snapshot før du gjør en endring – og rulle tilbake på sekunder.

## 5.5 SMART – disker snakker

Alle moderne disker har SMART (Self-Monitoring, Analysis and Reporting Technology). Med `smartctl -a /dev/sda` kan du se temperatur, antall reallokerte sektorer, timer i drift og mye mer. Sett opp regelmessig overvåking med `smartd` slik at du får beskjed *før* disken dør.

## 5.6 Mål først: fio

Skal du velge mellom RAID-nivåer, filsystemer eller «er SSD-en min egentlig treg?» – mål med `fio`:

```bash
sudo apt install fio
fio --name=lesetest --filename=/tmp/fiotest --size=1G --rw=randread \
    --bs=4k --runtime=30 --time_based --end_fsync=1
```

Se på IOPS og latens (`clat`). Kjør samme test før og etter endringen – tallene avgjør, ikke magefølelsen. (Slett testfilen etterpå.)

**Prøv selv:** Hvis du har en ekstra disk (eller USB-stick), opprett et LVM-volum: `pvcreate`, `vgcreate`, `lvcreate`, formater med `mkfs.ext4`, monter og utvid det. Ta et snapshot med `lvcreate -s` og eksperimenter. Kjør deretter RAID-brannøvelsen over med loop-enheter.

---

## Anatomi av en hendelse #1: Databasen døde kl. 03:12

*En sann historie fra en hjemmelab nær deg. Les den som en kriminalgåte – og legg merke til rekkefølgen på spørsmålene.*

**Symptomet:** Du våkner til at Nextcloud viser «Internal Server Error». Alt annet på serveren svarer.

**Hypotesene:** Full disk? Krasjet database? Nettverk? Du sjekker det billigste først:

```bash
df -h                        # disk: 62 % – ikke det
systemctl status mariadb     # «inactive (dead)» – DER
```

**Sporet:** Hvorfor døde den? Tjenestens egen logg slutter brått:

```bash
journalctl -u mariadb -b | tail -3
# ...
# mariadb.service: Main process exited, code=killed, status=9/KILL
```

`status=9/KILL` – noen sendte SIGKILL. Men ingen mennesker var våkne kl. 03:12. Når en prosess drepes med ni uten avsender i egen logg, er hovedmistenkt alltid den samme: kjernen. Du sjekker kjerneloggen:

```bash
journalctl -k -b | grep -i oom
# Out of memory: Killed process 1247 (mariadbd) total-vm:2847216kB ...
```

**Funnet:** OOM-killeren. Men hvorfor gikk maskinen tom kl. 03? `systemctl list-timers` avslører at fotoindekseringsjobben (ny i forrige uke) starter 03:00 – og den leser hele bildebiblioteket inn i minnet, på en VM med 2 GB RAM der databasen alt bruker halvparten. Indekseringen presset minnet, og OOM-killeren valgte det *største* offeret: databasen. Den skyldige overlevde; den uskyldige døde. Slik er badness-score.

**Fiksen:** `systemctl start mariadb` nå. Deretter den ordentlige: `sudo systemctl edit foto-indeksering` → `MemoryHigh=512M` (triks fra kapittel 2.5), og `OOMScoreAdjust=-800` i mariadb sin drop-in, så databasen aldri er førstevalg igjen.

**Lærdommen:** SIGKILL uten forklaring i tjenestens logg → se i *kjernens* logg. OOM velger den største, ikke den skyldige. Og ressursgrenser (kapittel 2) er billigere enn nattevåk.

---

# 6. Python for systemadministratorer

*Del 2: Infrastruktur som kode*

Når bash-skriptet ditt runder 50 linjer, er det på tide å vurdere Python. Her får du verktøyene for å bygge robuste systemverktøy.

## 6.1 argparse – profesjonelle CLI-verktøy

Med `argparse` gir du brukeren flagg, hjelpetekst og validering gratis:
```python
import argparse
parser = argparse.ArgumentParser(description='Sikkerhetskopiering')
parser.add_argument('kilde', help='Sti til kildedata')
parser.add_argument('--dest', default='/backup', help='Destinasjonskatalog')
args = parser.parse_args()
```
Du slipper å parse `$@` manuelt, og verktøyet ditt oppfører seg som alle andre *nix-kommandoer.

## 6.2 subprocess – å kjøre eksterne programmer

`subprocess.run()` kjører en kommando og returnerer en `CompletedProcess` med returkode og output. Fang stdout og stderr, og reager på feil:
```python
import subprocess
result = subprocess.run(['rsync', '-av', args.kilde, args.dest], capture_output=True, text=True)
if result.returncode != 0:
    print(f"Feil: {result.stderr}")
```
Unngå `shell=True` for å slippe injeksjonsangrep – del kommandoen i en liste.

## 6.3 API-kall og systemd-integrasjon

Ofte vil verktøyet ditt snakke med et web-API, for eksempel Prometheus. Bruk `requests` og parse JSON. For å gjøre skriptet til en systemd-tjeneste, skriver du en unit-fil med `Type=oneshot` eller `Type=simple` og lar Python-skriptet kjøre. Husk å håndtere `SIGTERM` pent for å kunne stoppe tjenesten.

**Eksempel som knytter sammen kapitlene:** node_exporter (kapittel 12) har en *textfile collector* – den leser `.prom`-filer fra en mappe og eksponerer innholdet som metrikker. Dermed kan et lite Python-skript gjøre hva som helst målbart:

```python
#!/usr/bin/env python3
# backup_metrikk.py – hvor gammel er siste backup? (kjøres av en systemd-timer)
import time
from pathlib import Path

nyeste = max(Path('/backup').glob('*.tar.gz'), key=lambda p: p.stat().st_mtime)
alder_timer = (time.time() - nyeste.stat().st_mtime) / 3600

Path('/var/lib/node_exporter/textfile/backup.prom').write_text(
    f'backup_alder_timer {alder_timer:.1f}\n'
)
```

I kapittel 12 setter du opp varselregelen `backup_alder_timer > 26` – og fra da av *vet* du at backupen går, i stedet for å tro det. Ti linjer Python, og «mål først» gjelder plutselig backupen din også.

## 6.4 Eksempel: et lite backup-verktøy

Vi skriver et backup-verktøy som tar katalog, destinasjon og ekskluderingsmønstre, sender melding til et API når det er ferdig, og fungerer som systemd-tjeneste. Du finner full kode i lab-repoet.

## 6.5 Mål først: hyperfine

Er Python-versjonen din raskere enn bash-skriptet den erstattet? Ikke synes – mål. `hyperfine` kjører kommandoene mange ganger og gir deg statistikk med varmkjøring og standardavvik:

```bash
sudo apt install hyperfine
hyperfine './gammel.sh' './ny.py'
# Summary: './ny.py' ran 3.42 ± 0.18 times faster than './gammel.sh'
```

Dette er bokens «mål først»-prinsipp i miniatyr: ett verktøy, og diskusjonen om «hva som er raskest» er over på ti sekunder.

**Prøv selv:** Skriv et Python-script som tar en URL som argument, sjekker HTTP-statuskoden, og skriver ut OK eller FEIL med passende returkode. Gjør det kjørbart, test det – og sammenlign det med `curl -o /dev/null -w '%{http_code}'` i hyperfine.

---

# 7. Ansible – beskriv tilstanden, ikke kommandoene

*Del 2: Infrastruktur som kode*

Med Ansible blir hele hjemmelabben reproduserbar. Du skriver *hva* du vil ha, ikke *hvordan* du oppnår det. Dette er idempotens i praksis: å kjøre playbooken to ganger gir det samme sluttresultatet.

**Hvorfor Ansible og ikke Puppet, Chef eller Salt?** Tre grunner som teller for en labb: Ansible er *agentløst* – det trenger bare SSH, som du allerede har på alle maskinene (bok 2!), mens de andre krever egen programvare (og ofte en master-server) på hver node. Playbooks er YAML du kan lese om et år. Og fellesskapet er størst, så oppskriften du trenger finnes som regel. I store bedrifter kan de andre vinne – i en hjemmelab er Ansible det åpenbare valget.

## 7.1 Første playbook

```yaml
- name: Konfigurer webserver
  hosts: web
  become: yes
  tasks:
    - name: Installer nginx
      apt:
        name: nginx
        state: present
    - name: Start tjenesten
      systemd:
        name: nginx
        state: started
        enabled: yes
```
Kjør med `ansible-playbook playbook.yml -i inventory.ini`. Hvis nginx allerede er installert, skjer ingenting.

## 7.2 Roller, hemmeligheter og inventar

Når labben vokser, organiserer du med roller. En rolle for `docker`, en for `overvåking`, osv. Hemmeligheter som API-nøkler legger du i `ansible-vault` – en kryptert fil i repoet. (🟡 Alternativet mange velger etter hvert er **sops** med **age**-nøkler: hemmelighetene ligger kryptert i Git, men *feltvis*, så `git diff` fortsatt viser hvilke nøkler som endret seg – uten å avsløre verdiene.) Inventaret definerer maskinene dine, grupper og variabler. Et typisk inventar:
```
[servere]
labserver ansible_host=192.168.1.10
pi ansible_host=192.168.1.11
```

## 7.3 Fra null til ferdig

Målet er at en ny Debian-installasjon skal bli en fullverdig nod i labben med én kommando:
```
ansible-playbook -i inventory site.yml --ask-become-pass
```
Etter dette har du NFS-monteringer, Docker, overvåking og alle dine tjenester. Dette er essensen av Infrastructure as Code (IaC).

**Prøv selv:** Skriv en playbook som oppretter en bruker med SSH-nøkkel, installerer `htop` og setter tidssone. Kjør den mot en VM.

---

# 8. Egen Git-server og CI/CD hjemme

*Del 2: Infrastruktur som kode*

Når du administrerer infrastruktur som kode, vil du ha kontroll over koden din. Forgejo (en lettvekts Git-tjeneste) kjører glimrende på en Raspberry Pi. (Har du hørt om Gitea? Forgejo er en fellesskapsstyrt avgrening av nettopp Gitea, opprettet i 2022 da Gitea fikk kommersiell eier – de er fortsatt svært like, men Forgejo forvaltes av ideelle Codeberg og driver Codeberg.org. Derfor velger boken den.)

## 8.1 Sett opp Forgejo

Bruk en container eller installer binærfilen. Sett opp en omvendt proxy (se kapittel 11) og logg inn. Opprett et repo for Ansible-koden din og et for applikasjonene dine.

## 8.2 CI/CD med Actions

Forgejo har innebygd CI/CD (kompatibelt med GitHub Actions). Du kan automatisk kjøre ShellCheck på skriptene dine, teste Ansible-playbooks og bygge containerimages ved push. Et eksempel på workflow:
```yaml
on: [push]
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: shellcheck *.sh
```
Når du pusher en ny commit, får du umiddelbart tilbakemelding om du har innført en feil.

## 8.3 Automatisk deploy

Siste steg i pipelinen kan være å trigge Ansible-playbooken mot labben – for eksempel via en SSH-kommando. Da blir kodeendringer automatisk rullet ut i infrastrukturen. Dette er hjemlig GitOps.

## 8.4 Dokumentér for fremtids-deg

IaC-repoet forteller *hva* labben er; dokumentasjon forteller *hvorfor* – og *hva du gjør når det brenner*. Eksperter dokumenterer, og de gjør det i repoet, ikke i et løsrevet wiki-dokument ingen finner. Tre vaner:

- **README per rolle/tjeneste** – fem linjer holder: hva den gjør, hvorfor den finnes, hvilke porter, hva den avhenger av.
- **Runbooks for kjente feil** – «Jellyfin hakker → sjekk io-wait i Grafana → er det backupjobben? → …». Skriv den *mens* du feilsøker, ikke etterpå – hendelses-casene i denne boken er runbooks i fortellerform.
- **Driftsjournalen er `git log`** – hvis commit-meldingene sier *hvorfor* («Flyttet backup til 03:30 – mettet disken på dagtid»), har du en komplett, tidsstemplet journal gratis.

Testen for alt du skriver: **«future me»-prinsippet**. Om seks måneder husker du ingenting av dette. Skriv til den personen – hen er den viktigste leseren din.

## 8.5 🟡 Git-verktøyene du oppdager som ekspert

Bok 2 ga deg arbeidsflyten; disse fire er verdt å *vite om* – man-sidene tar resten den dagen du trenger dem:

- **`git tag kapittel-12`** – merk milepæler. Lab-repoet bruker dette per kapittel; gjør det samme ved «alt virker»-øyeblikk.
- **`git stash`** – legg bort halvferdig arbeid på to sekunder når noe akutt dukker opp; `git stash pop` henter det tilbake.
- **`git bisect`** – binærsøk i historikken. «DNS-en virket for 30 commits siden»: `git bisect start`, merk god og dårlig commit, test halvveis-punktene Git foreslår – etter ~5 hopp står synder-commiten der. Kombinert med IaC betyr det at du kan finne *nøyaktig hvilken endring* som knakk labben.
- **`git worktree`** – flere grener utsjekket samtidig i hver sin mappe, uten kloning.

**Prøv selv:** Installer Forgejo i en container, opprett et repo og aktiver Actions med et enkelt «hello world»-steg.

---

# 9. 🟡 NixOS – deklarativt til fingerspissene

*Del 2: Infrastruktur som kode*

NixOS er en Linux-distribusjon der hele systemet – pakker, konfigurasjon, tjenester – beskrives i én deklarativ fil (`/etc/nixos/configuration.nix`). Oppgraderinger er atomiske: hvis noe feiler, booter du bare forrige generasjon.

Dette kapittelet gir en smakebit: sett opp NixOS i en VM og deklarer hele tjenesteoppsettet i konfigurasjonsfilen. Slik ser «en server» ut som kode:

```nix
{
  services.openssh.enable = true;
  services.nginx.enable = true;
  virtualisation.docker.enable = true;
  users.users.glenn = {
    isNormalUser = true;
    extraGroups = [ "wheel" "docker" ];
    openssh.authorizedKeys.keys = [ "ssh-ed25519 AAAA… glenn@laptop" ];
  };
}
```

`sudo nixos-rebuild switch` bygger og aktiverer – atomisk. Gikk noe galt? `sudo nixos-rebuild switch --rollback`, eller velg forrige *generasjon* i GRUB-menyen: hver rebuild er et komplett, bootbart system, og gamle generasjoner ligger igjen til du rydder. Det er Timeshift-ideen fra bok 1, innebygd i selve operativsystemet.

Sammenlign med Ansible: playbooken *endrer* en maskin mot ønsket tilstand; Nix *bygger* tilstanden fra bunnen hver gang – drift (konfigurasjonsslark over tid) er umulig per definisjon. Prisen er en mental omstilling og et eget språk. Verdt det når reproduserbarhet trumfer alt; overkill hvis Ansible-oppsettet ditt allerede gjør jobben.

**Prøv selv:** Følg den offisielle NixOS-quickstarten i en VM. Legg til `services.nginx.enable = true;` og test.

---

## Anatomi av en hendelse #2: Playbooken som stengte alle dørene samtidig

**Symptomet:** Du ruller ut en «liten» SSH-innstramming med Ansible mot *alle* maskinene. Playbooken melder grønt. Tretti sekunder senere: `ssh: connect refused` – på samtlige maskiner. Også de du skal fikse de andre fra.

**Hva skjedde:** Template-filen for `sshd_config` brukte variabelen `ssh_allow_users` – som var definert i gruppevariablene for *én* maskingruppe, men ikke de andre. Der ble `AllowUsers` stående tom. Handleren restartet sshd overalt, sshd nektet å starte med ugyldig konfigurasjon – og siden playbooken kjørte mot alle verter parallelt, døde alle dørene i samme sekund. Idempotens hjelper ikke når *innholdet* er feil: playbooken gjorde nøyaktig det du ba om, overalt, effektivt.

**Redningen:** Én terminal hadde fortsatt en *åpen* SSH-økt fra før utrullingen (etablerte tilkoblinger overlever sshd-restart). Derfra: fiks variabelen, kjør playbooken på nytt. Uten den økten hadde det blitt tastatur og skjerm i boden.

**Lærdommene – tre linjer i playbooken hadde stoppet alt:**

```yaml
- name: Skriv sshd_config
  template:
    src: sshd_config.j2
    dest: /etc/ssh/sshd_config
    validate: 'sshd -t -f %s'     # 1: nekter å skrive config sshd selv avviser
```

Pluss `serial: 1` på play-nivå (2: rull ut én maskin av gangen – første feil stopper resten) og vanen `ansible-playbook --check --diff` før hver ekte kjøring (3: se diffen før den skjer). Og hold alltid én SSH-økt åpen når du endrer SSH.

---

# 10. Nettverk på ekspertnivå

*Del 3: Drift som proff*

Nettverk er nervesystemet i labben. Her tar vi kontroll over alt fra brannmur til DNS.

## 10.1 nftables – moderne brannmur

`nftables` erstatter iptables/ufw. Det har en renere syntaks og bedre ytelse. Regler defineres i tabeller med chains. Eksempel på en enkel brannmur som tillater SSH og HTTPS:
```
table inet filter {
    chain input {
        type filter hook input priority 0; policy drop;
        iif lo accept
        ct state established,related accept
        tcp dport {22, 443} accept
    }
}
```
For mer komplekse oppsett kan du inkludere regelfiler og bruke sets. Verktøyet `nft` gir deg full kontroll.

## 10.2 VLAN – del opp nettverket

Med VLAN kan du isolere IoT-dingser, gjester og servere i hvert sitt segment. En trunk-port fra ruteren bærer taggede pakker. På Linux lager du VLAN-grensesnitt med `ip link add link eth0 name eth0.10 type vlan id 10`. Kombinert med broer og nftables får du et proft oppsett der en kompromittert IoT-bryter ikke får snoke på lab-trafikken.

```
 Internett ──── ruter ════ trunk (alle VLAN, tagget) ════ svitsj/server
                                                             │
                     ┌───────────────────┬───────────────────┤
                 VLAN 10             VLAN 20             VLAN 30
              administrasjon        tjenester              IoT
              (SSH, Proxmox)       (web, media)     (isolert – kun internett)
                     ▲                   ▲                   │
                     └── nftables styrer hva som får krysse ─┘
```

*Figuren viser: én fysisk trunk-lenke bærer alle VLAN-ene tagget; nftables avgjør hvilken trafikk som får bevege seg mellom segmentene.*

## 10.3 Egen rekursiv DNS med Unbound

De fleste rutere bruker ISP-ens DNS. Med Unbound får du en lokal, rekursiv DNS-tjener som spør rot-serverne direkte – ingen tredjepart som logger dine oppslag. Sett den opp med DNSSEC-validering, og bruk den som forwarder for labben. Konfigurer DHCP (f.eks. med `dnsmasq` eller `kea`) til å dele ut Unbounds adresse.

## 10.4 IPv6 uten frykt

IPv6 er her. Mange ISPer gir deg en /56-prefix. I labben kan du tildele stabile adresser og sette opp brannmur for IPv6 på samme måte som for IPv4. Med Unbound kan du også gjøre rekursive oppslag over IPv6. Det er enklere enn du tror.

**Norsk virkelighet:** Praksis varierer mellom norske ISP-er – noen deler ut romslige prefikser, andre er gjerrigere, og enkelte (særlig mobil- og enkelte fiberleverandører) setter deg bak **CGNAT** på IPv4. Bak CGNAT har du ingen offentlig IPv4-adresse, og portvideresending er umulig – da er IPv6 eller VPN ut (kapittel 15) de reelle veiene inn til labben. Sjekk om du er bak CGNAT: er WAN-adressen på ruteren din i 100.64.0.0/10-området, er svaret ja. Og et pragmatisk alternativ mange glemmer: flere norske ISP-er selger **fast, offentlig IPv4-adresse** for en femtilapp i måneden – ofte den enkleste utveien hvis du vil eksponere tjenester uten å rote med IPv6 eller mesh-VPN.

## 10.5 tcpdump – se selve pakkene

Logger forteller hva programmene *mener* skjedde; `tcpdump` viser hva som faktisk gikk på ledningen:

```bash
sudo tcpdump -i eth0 port 53          # se DNS-oppslagene skje live
sudo tcpdump -i any host 192.168.1.50 # all trafikk til/fra én maskin
sudo tcpdump -i eth0 -w dump.pcap port 443   # lagre for analyse i Wireshark
```

Klassikeren: en tjeneste «svarer ikke». `tcpdump port 8096` viser SYN-pakker inn – men ingen SYN-ACK tilbake. Da vet du at pakkene *kommer frem*, men ingen lytter (eller brannmuren dropper): feilen er på serveren, ikke i nettet. Ett minutt med tcpdump erstatter en time med gjetting.

 For koseligere lesing av lagrede dumper: `termshark` (Wireshark i terminalen) eller Wireshark på skrivebordet.

## 10.6 Mål først: iperf3

Før du skylder på WiFi, kabler eller VLAN-oppsettet – mål:

```bash
iperf3 -s                     # på den ene maskinen (server)
iperf3 -c 192.168.1.10        # fra den andre: måler faktisk gjennomstrømning
```

Kjør før og etter nettverksendringer. «Det føles tregt» er en hypotese; 940 Mbit/s er et faktum.

**Prøv selv:** Sett opp Unbound i en container eller på en VM. Konfigurer en klient til å bruke den og kjør `dig @<unbound-ip> linux.no`. Verifiser at DNSSEC-flagget (`ad`) er satt.

---

# 11. Reverse proxy og TLS overalt

*Del 3: Drift som proff*

«Hvorfor må jeg huske portnumre?» Med en reverse proxy får du vanlige URL-er og automatisk HTTPS – selv på hjemmenettet.

## 11.1 Caddy – omvendt proxy med automatisk TLS

Caddy får Let's Encrypt-sertifikater automatisk og fornyer dem selv. En enkel Caddyfile:
```
jellyfin.hjemme.no {
    reverse_proxy 192.168.50.25:8096
}
```

```
                         ┌──► jellyfin   :8096
 nettleser ──HTTPS:443──► Caddy ──HTTP──►├──► nextcloud  :8080
 jellyfin.hjemme.no       │              └──► grafana    :3000
                          └─ ett sertifikat, én port åpen,
                             ruting på domenenavn
```

*Figuren viser: all trafikk går kryptert til Caddy på port 443, som ruter videre til riktig tjeneste basert på domenenavnet.*
For interne domener uten offentlig DNS, bruk DNS-utfordring – da trenger du ikke åpne porter. **Merk:** DNS-moduler følger ikke med standard-Caddy; du bygger en utgave med din DNS-leverandørs modul via `xcaddy`, eller laster ned en ferdigbygget variant fra caddyserver.com/download med modulen huket av. Konkret, med Cloudflare som eksempel:

```bash
xcaddy build --with github.com/caddy-dns/cloudflare   # bygg Caddy med DNS-modul
```

```
jellyfin.dittdomene.no {
    tls {
        dns cloudflare {env.CF_API_TOKEN}    # bevis domene-eierskap via DNS – ingen åpne porter
    }
    reverse_proxy 192.168.50.25:8096
}
```

Sjekk om din norske DNS-leverandør har API-støtte i modullisten før du velger leverandør.

## 11.2 Interne sertifikater

Noen ganger vil du ha sertifikater for `*.lab.lan`. Du kan opprette en egen CA (Certificate Authority) med `mkcert` (ligger i pakkebrønnene, perfekt til å komme i gang) eller `step-ca` (kraftigere, med ACME-støtte så Caddy kan hente interne sertifikater automatisk – men **ikke** i apt: installeres fra Smallsteps eget repo eller som binærfil, se [smallstep.com/docs/step-ca/installation](https://smallstep.com/docs/step-ca/installation)), og utstede sertifikater som alle maskiner i labben stoler på. Dette gir full kryptering internt uten avhengighet til eksterne tjenester.

## 11.3 Hvorfor HTTPS hjemme?

Kryptering hindrer snoking på Wi-Fi, og mange moderne apper (Jellyfin, Home Assistant) krever HTTPS for å fungere fullt ut. I tillegg lærer du hvordan PKI fungerer, en ferdighet som er gull verdt.

**Prøv selv:** Sett opp Caddy for en enkel web-app som kjører på en høy port. Gi den et lokalt domene i `/etc/hosts` og se at du får grønt hengelås.

---

# 12. Overvåking og varsling – Prometheus og Grafana

*Del 3: Drift som proff*

Du skal gå fra «jeg tror alt kjører» til «jeg vet».

## 12.1 Start enkelt: uptime-kuma og ntfy

Før vi bygger katedralen, sett opp kapellet – på fem minutter:

- **uptime-kuma** (én container): en statusside som pinger tjenestene dine og HTTP-sjekker web-grensesnittene. Grønt/rødt, responstider, historikk. For en labb med en håndfull tjenester er dette ofte *nok*.
- **ntfy**: push-varsler til mobilen uten app-butikk-kontoer: `curl -d "Disken på nas er 91 % full" ntfy.sh/min-hemmelige-kanal` – og telefonen plinger. Merk: på offentlige ntfy.sh er kanalnavnet hele «passordet» – alle som gjetter det kan både lese og sende. Bruk en lang, tilfeldig streng (`openssl rand -hex 16`), eller selvhost tjenesten.

Disse to gir deg 80 % av tryggheten for 5 % av innsatsen. Resten av kapittelet bygger de siste 20 – metrikker over tid, logger og intelligent varsling.

## 12.2 Prometheus og node_exporter

Node_exporter samler CPU, minne, disk, nettverk fra hver maskin. Prometheus scraper dette og lagrer tidsserier. Med `ansible` ruller du ut node_exporter på alle noder og legger dem til i prometheus.yml.

## 12.3 Grafana – dashboards som viser alt

Grafana henter data fra Prometheus og viser grafer. Importer ferdige dashboards (f.eks. ID 1860 for node-eksport) og få umiddelbar oversikt. Du kan lage egne paneler, for eksempel «ledig diskplass på /data», og sette fargeterskler.

## 12.4 Logging med Loki

For sentralisert logging sender du syslog og Docker-logger til Loki med `promtail`. I Grafana kan du søke i loggene side om side med metrikkene. Feilsøk et problem ved å se grafen for feilrater og deretter bore ned i loggene for det nøyaktige tidsrommet.

## 12.5 Alertmanager – varsling før krisen

Konfigurer Prometheus med regler, for eksempel «diskbruk over 90 %». Alertmanager sender melding til e-post, Matrix eller ntfy.sh. Det betyr at du får beskjed *før* disken er full – slik at du kan utvide LVM eller rydde i tide.

**Prøv selv:** Installer Prometheus og Grafana på labmaskinen, legg til minst én eksport og et dashboard. Sett opp en dummy-alert for å teste varslingskanalen.

---

# 13. Containere på dypet

*Del 3: Drift som proff*

Du kjører allerede containere. Nå skal du forstå dem uten magi.

## 13.1 Bygg dine egne images

Med en Containerfile (Dockerfile) bygger du et image lagvis. Hvert `RUN`-kommando blir et nytt lag – minimer dem for å spare plass. Multi-stage builds gir deg et slankt produksjonsimage uten byggeverktøy.

## 13.2 Hva en container egentlig er

En container er en prosess med egne namespaces (pid, net, mnt, etc.) og cgroups. Med `unshare` kan du lage din egen primitive container for hånd:
```
unshare --mount --pid --fork --mount-proc chroot /mitt/rootfs /bin/bash
```
Dette avmystifiserer Docker. Alle fancy funksjoner er bare Linux-kjernefunksjonalitet pakket pent inn.

## 13.3 Rootless og sikkerhet

Kjør containere uten root-rettigheter med Podman eller Docker rootless. Det gir et ekstra sikkerhetslag. Bruk seccomp-profiler og `--cap-drop=ALL` for å begrense prosessens rettigheter.

## 13.4 🟡 Slektningene: system-containere

Docker og Podman kjører *applikasjons*containere – idealet er én prosess per container. **LXC** og **systemd-nspawn** kjører *system*containere: en hel distro med egen init, brukere og tjenester – som en VM, men uten hypervisor-kostnaden. Proxmox (kapittel 14) bruker LXC til akkurat dette, og nspawn følger med systemd du allerede har: `sudo systemd-nspawn -D /mitt/rootfs -b` booter et helt Debian i en mappe. Samme kjernemekanismer som i 13.2 – bare et annet bruksmønster: velg applikasjonscontainer for én tjeneste, systemcontainer når du trenger «en hel maskin» billig.

## 13.5 🟡 k3s – Kubernetes i lite format

k3s er en lettvekts Kubernetes-distribusjon perfekt for hjemmebruk. Du kan deklarativt kjøre pods med YAML. *Når* er det overkill? Som oftest – med mindre du har mange tjenester og trenger horisontal skalering, automatisk failover og GitOps. Vi setter opp en testklynge, men konkluderer ærlig: for de fleste holder Docker Compose i lang tid.

**Prøv selv:** Bygg et image for en enkel Python-app. Kjør det rootless med Podman. Inspiser namespaces med `lsns`.

---

# 14. Virtualisering og hjemmelab-arkitektur

*Del 3: Drift som proff*

Noen ganger trenger du et helt operativsystem, ikke bare en container. KVM/libvirt gir deg virtuelle maskiner med nær native ytelse.

## 14.1 KVM og libvirt fra kommandolinjen

`virsh` lar deg administrere VM-er: starte, stoppe, ta snapshots. Kombinert med `virt-install` kan du opprette nye VM-er fra CLI. Bruk `virt-viewer` for grafisk konsoll, eller sett opp SSH direkte.

## 14.2 Cloud-init – maskiner som konfigurerer seg selv

Cloud-init lar deg injisere SSH-nøkler, brukere og skript ved førstegangsoppstart. Du lager en ISO med `cloud-localds` og bruker den som CD-ROM – så er VM-en klar etter boot. Perfekt for automatisert provisioning med Ansible.

## 14.3 Proxmox – hypervisorplattform

Proxmox VE pakker KVM og LXC i et web-grensesnitt med klyngestøtte, snapshots og live migrasjon. Dette kan være hjertet i en større lab. Vi diskuterer arkitektur: hvilke tjenester kjører i containere, hvilke trenger egne VM-er, og hvordan du segmenterer med virtuelle nett.

## 14.4 Labben og strømregningen 🇳🇴

En hjemmelab går døgnet rundt – og i Norge er strøm en reell driftskostnad. Regnestykket er enkelt: **watt × 8,76 = kWh per år**. En Raspberry Pi 5 (~7 W) koster deg rundt 61 kWh/år; en gammel stasjonær som «bare står der» (~80 W) drar ~700 kWh/år – en tier mot en hundrelapp i måneden, avhengig av spotpris. Mål det faktiske forbruket med en smartplugg med effektmåling før du bestemmer arkitektur: ofte er konklusjonen «Pi + gammel PC som bare vekkes ved behov» (Wake-on-LAN!) i stedet for alt-på-hele-tiden. Typiske tall (mål selv – varierer med last og alder):

| Enhet | Typisk effekt | kWh/år | Kostnad/år v/1,5 kr/kWh |
|-------|--------------|--------|--------------------------|
| Raspberry Pi 5 | ~7 W | ~61 | **~90 kr** |
| Mini-PC/NUC | ~12 W | ~105 | **~160 kr** |
| 2-skuffs NAS | ~25 W | ~219 | **~330 kr** |
| Gammel stasjonær | ~80 W | ~700 | **~1050 kr** |

Tre praktiske grep: **Wake-on-LAN** (`sudo apt install wakeonlan` – vekk den kraftige maskinen bare når du trenger den, la Pi-en stå for det som må være oppe), **smartplugg med effektmåling** (gir deg både tallene og muligheten til å fjernstyre strømmen), og for de ivrigste: **spotpris-styring** – kjør de tunge, uviktige jobbene (transkoding, LFS-bygg!) i timene strømmen er billigst.

Og selvsagt regner en ekspert selv, med én linje:

```bash
watt=25; pris=1.50; awk "BEGIN{printf \"%.0f kr/år\n\", $watt*8.76*$pris}"
```

Effektbudsjettet hører hjemme i vedlegg C sammen med resten av arkitekturen.

**Prøv selv:** Installer libvirt, opprett en Debian-VM med `virt-install` og gi den en cloud-init-konfigurasjon. Når den booter, skal den ha din SSH-nøkkel.

---

# 15. Sikkerhet på alvor

*Del 3: Drift som proff*

Sikkerhet handler om trusselmodellen din. Hva vil du beskytte mot? Vi tar grep uten panikk.

## 15.1 AppArmor og SELinux

Mandatory Access Control (MAC) begrenser hva en prosess kan gjøre, selv om den kjører som root. AppArmor (Debian/Ubuntu) bruker profiler per applikasjon. Med `aa-status` ser du aktive profiler. SELinux (RHEL) er mer granulært, men har et rykte for å være komplisert. Vi lærer å sette opp en egendefinert AppArmor-profil for en tjeneste.

## 15.2 Systemd-sandboxing

Systemd kan isolere tjenester uten ekstra verktøy. Direktiver som `ProtectSystem=strict`, `PrivateTmp=true`, `NoNewPrivileges=yes`, og `ReadOnlyPaths=` gjør at tjenesten ikke kan skrive til systemet. Dette er enkelt og effektivt – gjør det til en vane.

Og her kommer bokens «mål først»-prinsipp til sikkerheten: **`systemd-analyze security <tjeneste>`** gir tjenesten en eksponerings-score fra 0 (innelåst) til 10 (vidåpen) og lister *nøyaktig* hvilke direktiver som mangler. Kjør den før og etter herding – tallet som synker er fremgangen din:

```bash
systemd-analyze security            # scoretabell for ALLE tjenester
systemd-analyze security nginx      # detaljert sjekkliste for én
```

Få verktøy gir så mye herding per minutt – og nesten ingen kjenner det.

**Konkret:** En typisk egenlaget tjeneste starter på **9.6** («UNSAFE»). Denne drop-in-filen (`sudo systemctl edit min-tjeneste`) tar den ned i **4-tallet** på ett minutt:

```ini
[Service]
NoNewPrivileges=yes
ProtectSystem=strict
ProtectHome=yes
PrivateTmp=yes
PrivateDevices=yes
RestrictAddressFamilies=AF_INET AF_INET6 AF_UNIX
CapabilityBoundingSet=
```

Restart, kjør `systemd-analyze security min-tjeneste` igjen, og test at tjenesten fortsatt virker (trenger den å skrive et sted, åpne akkurat den stien med `ReadWritePaths=`). Stram til gradvis – scoren viser veien.

## 15.3 Auditd – hvem gjorde hva?

Auditd logger sikkerhetshendelser: filtilganger, systemkall, pålogginger. Med `ausearch` kan du spore nøyaktig hvilken bruker som endret en sensitiv fil. En uvurderlig ressurs ved hendelsesrespons.

## 15.4 Herding og trusselmodell

Vi går gjennom CIS-benchmarks i forenklet form: deaktiver unødvendige tjenester, konfigurer sikre ssh-innstillinger (nøkkelbasert, ingen passord), aktiver automatiske sikkerhetsoppdateringer, og bruk `fail2ban`. Men alt starter med spørsmålet: «Hva er verdt bryet?» For en hjemmelab er ofte isolerte segmenter og gode brannmurregler det viktigste.

## 15.5 🟡 Mesh-VPN: Tailscale og headscale

Ren WireGuard (bok 2) er stjerneformet: alt går via hjemmeserveren. **Tailscale** bygger et *mesh* på WireGuard: alle enhetene dine når hverandre direkte, med automatisk nøkkelhåndtering, NAT-traversering (fungerer bak CGNAT!) og tilgangsregler. Avveiningen er en skybasert koordineringsserver – vil du eie den selv, kjører du **headscale**, den åpne selvhostede utgaven. Ærlig vurdering: Tailscale er den beste «det bare virker»-opplevelsen i denne boken; headscale er riktig når prinsippet om selvhosting veier tyngst.

**Prøv selv:** Lag en systemd-tjeneste med sandboxing og en egen AppArmor-profil. Prøv å lese `/etc/shadow` fra tjenesten – den skal nektes.

---

## Anatomi av en hendelse #3: Alt er tregt – men bare på onsdager

**Symptomet:** Familien klager: Jellyfin hakker. Men bare av og til, og aldri når du sjekker. «Det føles tregt» er alt du har.

**Fra anekdote til data:** Uten overvåking hadde dette vært uløselig. Med Grafana (kapittel 12) åpner du node_exporter-dashbordet og ser to uker bakover. Der: CPU-en er frisk, minnet er frisk – men **io-wait** (`wa` fra kapittel 3-verktøyene) spiker hver onsdag 14:00–15:30. Nøyaktig når filmkveldene hakker.

**Sporet:** Hva skjer onsdager kl. 14? `systemctl list-timers` – og der står den: `restic-backup.timer`, flyttet til onsdag ettermiddag «midlertidig» for tre måneder siden. Du bekrefter live neste onsdag: `sudo iotop -o` viser restic som metter disken, og `biolatency` (kapittel 4) viser disk-latens som tidobles.

**Fiksen – tre lag:**

```bash
sudo systemctl edit restic-backup.service
```

```ini
[Service]
IOSchedulingClass=idle      # backup får disk KUN når ingen andre vil ha den
Nice=19
```

Pluss timeren tilbake til `OnCalendar=*-*-* 03:30` – og en Grafana-graf uken etter som *beviser* at io-wait-spikene er borte. Mål først, mål etterpå.

**Lærdommen:** Periodiske problemer krever historiske data – overvåkingen du satte opp i fredstid er den eneste veien fra «det føles tregt» til «det ER backupjobben, onsdager kl. 14». Og `IOSchedulingClass=idle` burde vært standard på enhver batchjobb fra dag én.

---

# 16. Feilsøking uten sikkerhetsnett

*Del 4: Mesterbrevet*

Når maskinen ikke booter, disken klikker, eller alt ser mørkt ut – da trer du inn.

## 16.1 Boot-problemer og initramfs-skallet

Hvis kjernen eller initramfs er ødelagt, dropper du til et BusyBox-skall. Med `dmesg` og `blkid` identifiserer du problemet (mangler root-filsystem? feil driver?). Fra live-USB bruker du `chroot` for å gjenoppbygge initramfs (`update-initramfs -u`) eller reinstallere GRUB.

## 16.2 GRUB-reparasjon og kjernepanikk

`grub-install` og `update-grub` redder bootloaderen. Ved gjentatte panikker sjekker du maskinvare med `memtest86+` og ser etter BIOS-oppdateringer. `journalctl -k -b -1` viser kjerneloggen fra forrige boot, inkludert panikk.

## 16.3 Disken døde: ddrescue og testdisk

`ddrescue` kopierer en defekt disk blokk for blokk, og hopper over dårlige områder. Deretter kan du bruke `testdisk` for å gjenopprette partisjonstabellen eller `photorec` for å redde filer. Husk: regelmessig backup gjør dette til en treningsøvelse i stedet for en katastrofe.

## 16.4 Brannøvelsen – øv mens det ikke haster

Brannvesenet øver ikke under brann. Sett av en time i kvartalet til **planlagte katastrofer** – i en VM med snapshot, så alt kan nullstilles:

1. **Gjenopprett noe fra backup** – én fil og én hel mappe, med klokke på. (Er restic-passordet ditt egentlig tilgjengelig når disken med passordbehandleren er den som døde?)
2. **Boot fra live-ISO og chroot-reparer** VM-en etter at du selv har ødelagt initramfs.
3. **RAID-havari:** kjør loop-øvelsen fra kapittel 5.2 til den sitter i fingrene.
4. **Strømbrudd:** hard-stopp en VM midt i skriving (`virsh destroy`) og verifiser at journalført filsystem + tjenester kommer pent opp igjen.

Før hver øvelse: skriv ned hva du *tror* vil skje. Etterpå: hva som faktisk skjedde. Gapet mellom de to er pensumlisten din. Og den ultimate versjonen av dette venter i **vedlegg D: Mesterprøven** – der vet du ikke engang hva som er ødelagt.

**Prøv selv:** Simuler en bootfeil ved å endre `GRUB_CMDLINE_LINUX` til en ugyldig rot-device, og lær å fikse det fra GRUB-kommandolinjen.

---

# 17. Den dagen skrivebordet ikke starter

*Del 4: Mesterbrevet*

*En Linux-ekspert er ikke den som kan flest kommandoer. Det er den som fortsatt får jobben gjort når det grafiske grensesnittet er borte.*

**Scenen:** Lørdag kveld. Du skal bare se en film. Du vekker maskinen, skjermen lyser – og så blir den svart. Ingen innloggingsboks. Ingen musepeker. Du hamrer på taster i panikk, og ingenting ser ut til å skje. Pulsen stiger. Så husker du: *dette har jeg trent på* – feilsøkingsmetoden fra bok 2, brannøvelsene fra kapittel 16, boot-feilen vi plantet i Mesterprøven. Du prøver `Ctrl+Alt+F3`, venter et sekund – og der: `login:`. Det finnes en vei inn. Pusten senker seg.

Dette kapittelet handler ikke om nye kommandoer. Det handler om å beholde hodet kaldt når skrivebordet er borte, og bruke hele verktøykassen fra de foregående kapitlene til å finne ut *hva* som er galt – og fikse det.

## 17.1 Første steg: kom til et skall

Du har to hovedveier inn når grafikken er borte:

**Fysisk tekstkonsoll:** `Ctrl+Alt+F3` (F1–F6 fungerer på de fleste systemer) gir deg et rent tekstskall. Det er nesten alltid tilgjengelig selv om display manageren har krasjet – gi det et par sekunder, og ikke bli forvirret av at skjermen er svart *før* du trykker. Logg inn med brukernavn og passord, og du er hjemme.

**SSH fra en annen maskin:** Står maskinen fortsatt på nettverket, er `ssh bruker@maskin-ip` ofte den *komfortable* veien inn – full terminal, fra sofaen, med kopier/lim. På en labbmaskin kjører sshd uansett.

Har du verken tekstkonsoll eller SSH, er det GRUB recovery mode og kapittel 16 som gjelder – men prøv alltid de to første.

Vel inne: `journalctl -b -f` i ett vindu (tmux!). Du skal snart se hva som klager.

## 17.2 Diagnostikk: hva sier loggene?

Grafikk-stakken involverer display manager (gdm/lightdm/sddm), kompositor/X-server og skjermkortdriver. Sil ut det relevante fra denne booten:

```bash
journalctl -b | grep -iE "gdm|lightdm|sddm|xorg|wayland|drm|nvidia|amdgpu|error|fail"
```

**Vanlige syndere og sporene deres:**

- **Display manager krasjer:** loggen stopper brått etter `Started GNOME Display Manager`. Prøv å starte den manuelt og se på klagene i sanntid: `sudo systemctl restart gdm` + `journalctl -u gdm -f`. Meldinger som «cannot open display» eller «no screens found» peker videre.
- **Driver ikke lastet:** `lspci -k | grep -iA3 vga` – står det `Kernel driver in use:` med tomt eller feil navn, er skjermkortmodulen problemet (kapittel 1). Klassikeren: NVIDIA-modulen røk i en kjerneoppgradering – boot forrige kjerne fra GRUB, så bygger DKMS den på nytt.
- **X11-detaljene:** `grep EE /var/log/Xorg.0.log` viser X-serverens egne feil (der den fortsatt brukes).
- **Maskinvarespor:** `sudo dmesg | grep -iE "error|fail"` avslører disk- og GPU-problemer under alt det andre.

> 🟡 **Wayland-tider:** På moderne oppsett er kompositoren selve skjermtjeneren, og `startx`-trikset fra gamle dager krever `xinit`-pakken og en X11-stack. Den moderne varianten av samme test: velg «GNOME on Xorg» (eller en annen sesjonstype) fra tannhjulet på innloggingsskjermen – starter *den*, vet du at feilen bor i Wayland-økten, ikke i driveren.

## 17.3 Når maskinvaren er synderen

Alt fra kapittel 5 og 16 gjelder her:

- **Disk:** `sudo smartctl -a /dev/nvme0 | grep -iE "error|media"` – stigende feiltall betyr at grafikk-krasjet bare var symptomet (jf. hendelse #4).
- **Minne:** memtest86+ kjøres *ikke* fra et skall – den er sitt eget lille operativsystem. Installer pakken `memtest86+`, så dukker den opp som valg i GRUB-menyen; boot den derfra (eller fra live-USB) og la den kjøre en full runde.
- **Varme/strøm:** fryser maskinen uten spor i loggene, sjekk temperaturene – `sensors` (pakken `lm-sensors`), eller les dem rett fra `/sys/class/thermal` slik du lærte i kapittel 1. CPU/GPU som pendler rundt 90–100 °C før krasj er svaret sitt eget.

## 17.4 Reparasjon – trinn for trinn

| Problem | Løsning |
|---------|---------|
| Display manager krasjer | `sudo apt install --reinstall gdm3`, eller bytt midlertidig: `sudo apt install lightdm` |
| Ødelagt X-konfigurasjon | Flytt bort `/etc/X11/xorg.conf` (om den finnes) – X klarer seg uten |
| Driver mangler etter kjerneoppgradering | Boot forrige kjerne fra GRUB; `sudo apt install --reinstall nvidia-dkms-*` |
| Brukerens egen konfig | `mv ~/.xinitrc ~/.xinitrc.bak` – og sjekk `~/.config` for ferske endringer |
| Disk full (GUI-er tåler det dårlig!) | `df -h`, rydd med `ncdu`, `apt autoremove`, `docker system prune` |
| Halvferdig oppgradering | `sudo dpkg --configure -a && sudo apt install -f` |

Og systemd-verktøyet folk glemmer: `sudo systemctl default` prøver å ta systemet til normal (grafisk) tilstand igjen – mens `sudo systemctl isolate multi-user.target` bevisst parkerer det i tekstmodus mens du reparerer.

## 17.5 Tekstlivet – å leve uten mus

Selv med grafikken tilbake er dette verktøy verdt å kjenne – for servere, SSH-økter og neste krise:

- `mc` (Midnight Commander) – filbehandler i terminalen
- `nmtui` – NetworkManager med menyer (redningen når WiFi-oppsett må endres uten GUI)
- `links` – nettleser i terminalen; god nok til å søke opp en feilmelding
- `btop`, `ncdu`, `tmux` – de gamle kjente, som nå virkelig får skinne

**Prøv selv:** Installer `mc` og `links` nå (mens alt virker), og bruk ti minutter i hver. Den dagen du trenger dem, er ikke dagen for å lære dem.

## 17.6 Den store utfordringen – en dag uten GUI

Sett av én dag der du gjør *alt* i terminalen: e-post (`mutt`), filer (`mc`), nettsøk (`links`), musikk (`mpd` + `ncmpcpp`), notater (vim), systemarbeid (alt du alt kan). Noter det du savner – halvparten viser seg å ha en terminalløsning du ikke kjente, og resten er en ærlig liste over hva GUI-en faktisk gir deg.

Dette er ikke asketisme for moro skyld: det er trening i å være *hjemme* i miljøet du møter på hver server, hver SSH-økt og hver krise.

## 17.7 Øv på det – i labben

Tre brannøvelser (VM med snapshot, som alltid):

1. `sudo systemctl stop gdm` + omstart → inn via tekstkonsoll, finn og start tjenesten.
2. Legg en ugyldig linje i en X-konfigurasjonsfil → reparer fra konsollen med loggene som kart.
3. Fyll rotdisken nesten helt: sjekk ledig plass med `df -h /`, og lag en fil som spiser det meste av den, f.eks. `sudo fallocate -l 40G /var/fyll` (juster tallet – og merk: *ikke* bruk `/tmp`, den er ofte tmpfs og fyller minnet i stedet!) → se hvordan GUI-en oppfører seg, logg inn i konsollen, finn synderen med `ncdu` og rydd.

---

**Det viktigste fra dette kapittelet**

- Veien inn: `Ctrl+Alt+F3` eller SSH – én av dem er nesten alltid åpen.
- `journalctl -b` + `lspci -k` + `dmesg` forteller *hvilket lag* som feiler: display manager, driver eller maskinvare.
- Kjerneoppgradering + NVIDIA er klassikeren – forrige kjerne i GRUB er angreknappen.
- Terminalkompetanse er ikke nostalgi; det er beredskap. Øv i fredstid.

---

# 18. Pakk og del programvaren din

*Del 4: Mesterbrevet*

Du har laget nyttige verktøy. Gjør dem installérbare.

## 18.1 Bygg en .deb-pakke

Med `dpkg-deb` og en kontrollfil (`DEBIAN/control`) lager du en enkel Debian-pakke. Legg til pre/post-installeringsskript, og definer avhengigheter. Da kan du dele verktøyet ditt med kollegaer eller installere det på tvers av labmaskinene.

## 18.2 Eget apt-repo

Med `reprepro` oppretter du et lokalt apt-repo. Legg inn .deb-filene dine, signer med GPG-nøkkel, og konfigurer klientene til å hente fra ditt repo. Nå kan du oppdatere alle maskiner med en `apt update && apt upgrade`.

## 18.3 Flatpak – distribusjon på tvers av distroer

Flatpak pakker applikasjonen med avhengigheter i en sandbox. Du skriver et manifest og bygger med `flatpak-builder`. Perfekt for GUI-verktøy eller når du trenger isolasjon.

**Prøv selv:** Lag en .deb-pakke av backup-verktøyet fra kapittel 6, opprett et apt-repo og installer det på en annen VM.

---

# 19. 🔴 Linux From Scratch – dannelsesreisen

*Del 4: Mesterbrevet*

Vi bygger et komplett Linux-system fra kildekode, trinn for trinn. Dette er den ultimate avmystifiseringen.

> **Realistisk forventning:** Du får dyp forståelse av hvordan en Linux-distribusjon henger sammen – forståelse ingen annen øvelse gir. Du får *ikke* en produksjonsklar server: ingen pakkebehandler, ingen sikkerhetsoppdateringer, ingenting du bør kjøre ekte tjenester på. Gjør det i en VM med snapshots, sett av en helg eller tre, og behandle det som en intellektuell øvelse – ikke som en ny distro du skal bruke daglig. Følg den offisielle boken på [linuxfromscratch.org](https://www.linuxfromscratch.org) – den er gratis, grundig og oppdateres jevnlig.

Ruten går slik, og vår rolle er å være kartleser ved siden av den offisielle boken: (1) bygg en midlertidig *verktøykjede* (binutils → GCC → glibc – kompilatoren som skal bygge alt annet), (2) bygg kjerneverktøyene i et chroot-miljø (bash, coreutils, alt du tar for gitt), (3) kompiler kjernen og sett opp bootloaderen. Når du taster `exec /bin/bash` og ser prompten i et system du selv har snekret, forstår du virkelig hva en distribusjon er. Det er en dannelsesreise – krevende, men transformerende.

**Prøv selv:** Start med LFS i en VM. Bygg de første tre pakkene (binutils, GCC, glibc) og føl triumfen.

---

# 20. Gi tilbake – på ekspertnivå

*Del 4: Mesterbrevet*

Sirkelen sluttes. Nå er du klar til å bidra.

## 20.1 Les kilden – ferdigheten over alle

Bok 2 lærte deg man-sider; eksperten går ett steg til: når dokumentasjonen slutter, **les koden**. Det er mindre skummelt enn det høres ut – du skal ikke forstå alt, bare finne svaret ditt:

```bash
apt source pakkenavn          # hent kildekoden til det du kjører (krever deb-src-linjer)
rg "kryptisk-innstilling" .   # finn hvor konfigflagget faktisk parses
```

Tre situasjoner der kilden slår alt annet: en konfigurasjonsopsjon er udokumentert (les parseren – da ser du også standardverdien), en feilmelding gir null treff på nett (søk *ordrett* i prosjektets GitHub – i både issues og kode), og «hva gjør egentlig denne pakken ved installasjon?» (`less /var/lib/dpkg/info/pakke.postinst`). Å lese andres kode er dessuten den beste forberedelsen til å bidra med egen.

## 20.2 Fra feilrapport til patch

Finn et prosjekt du bruker. Les `CONTRIBUTING.md`. Test og reproduser en bug. Skriv en ryddig rapport, eller enda bedre: en pull request med løsning.

**En konkret førstereise – slik kan den se ut:** Du bruker `tldr` daglig (kapittel 7) og oppdager at `tldr rsync` mangler norsk oversettelse. Perfekt førstebidrag: (1) Fork `tldr-pages/tldr` på GitHub. (2) `git clone` din fork, opprett gren: `git switch -c norsk-rsync`. (3) Kopier den engelske siden til `pages.no/` og oversett – følg formatet til de eksisterende norske sidene. (4) Commit, push, og åpne pull request – malen forteller deg hva du skal fylle ut. (5) En vedlikeholder kommenterer kanskje «bruk anførselstegn her» – du retter, pusher igjen, og PR-en oppdateres automatisk. Noen dager senere: *merged*. Navnet ditt står i historikken til et verktøy med millioner av brukere, og hele reisen brukte bare ferdigheter fra kapittel 21 i bok 2. Etikettene «good first issue» / «help wanted» på GitHub er laget for akkurat dette.

## 20.3 Pakkevedlikehold

Mange distribusjoner søker pakkevedlikeholdere. Du kan adoptere en foreldreløs pakke og vedlikeholde den: oppdatere versjoner, fikse avhengigheter, kommunisere med upstream. Dette er en uvurderlig gave til fellesskapet.

## 20.4 Del kunnskapen

Skriv en bloggpost, hold et lynkurs på jobben, eller lag en video. Å forklare hva du har lært befester din egen forståelse og hjelper neste generasjon entusiaster.

**Prøv selv:** Finn en «good first issue» i et verktøy du bruker, og send inn en pull request – selv om det bare er en dokumentasjonsforbedring.

---

## Anatomi av en hendelse #4: Maskinen som ikke ville opp etter strømbruddet

**Symptomet:** Etter et strømbrudd booter ikke labserveren. Skjermen viser `(initramfs)` og en BusyBox-prompt. Pulsen stiger – men dette promptet er et verktøy, ikke en dødsdom.

**Første spørsmål: hvorfor stoppet den her?** Bla opp i meldingene (eller `dmesg | tail`): `ALERT! UUID=… does not exist. Dropping to a shell!` Kjernen fant ikke rotfilsystemet. To hovedhypoteser: disken er død, eller filsystemet er skittent etter strømkuttet.

**Sporet:** `blkid` fra initramfs-skallet viser disken – den *finnes*, med riktig UUID. Ikke død, altså. Da er hypotese to styrket: journalen på filsystemet trenger opprydding før den lar seg montere:

```bash
fsck -y /dev/nvme0n1p2      # -y: svar ja til reparasjonene
# ... journal recovered, clean ...
exit                         # initramfs fortsetter booten – og maskinen er oppe!
```

**Men én gang til:** Uken etter skjer det igjen – uten strømbrudd. Gjentakende «tilfeldige» filsystemfeil på samme disk er et rødt flagg. `sudo smartctl -a /dev/nvme0` (kapittel 5.5): `Media and Data Integrity Errors: 214` og stigende. Disken er døende; strømbruddet var bare budbringeren. `ddrescue` til ny disk *nå*, mens den fortsatt leser (kapittel 16.3) – og backupen fra i natt gjør hele operasjonen udramatisk.

**Lærdommen:** initramfs-promptet gir deg verktøyene akkurat der du trenger dem (`blkid`, `fsck`). Fiks symptomet, men *jag årsaken*: én filsystemfeil er uflaks, to er en mistenkt. Og SMART-tallene visste det hele tiden – hadde `smartd`-varsling vært på plass (kapittel 5.5 + 12), hadde du byttet disken før strømbruddet i det hele tatt.

---

# 21. Å fortsette reisen

*Del 4: Mesterbrevet*

Denne boken slutter her. Det gjør ikke du. «Ekspert» er ikke en tilstand du når, men en retning du holder – og det som skiller de som fortsetter å vokse, er ikke talent, men *kildene de drikker fra*. Her er de.

## Nyhetsstrømmen – hold deg oppdatert uten støy

- **[LWN.net](https://lwn.net)** – ukens viktigste Linux-journalistikk, skrevet av folk som leser kildekoden. Kjernenyheter, dype tekniske artikler, ingen clickbait. Verdt abonnementet – dette er *avisen* for folk som deg nå.
- **Release notes** – les dem faktisk, før du oppgraderer. Distroens og kjernens utgivelsesnotater er der endringene som treffer deg står.
- **[FOSDEM-opptak](https://video.fosdem.org)** – Europas største frie programvarekonferanse legger alt ut gratis. Én talk til kaffen er en ukesdose læring.

## Kildene nær kjernen

- **[docs.kernel.org](https://docs.kernel.org)** – kjernens eget `Documentation/`-tre, pent formatert. Start med *admin-guide* – den er skrevet for driftere, ikke kjerneutviklere, og forklarer alt fra sysctl-parametere til OOM-oppførsel fra hestens egen munn.
- **[man-pages-prosjektet](https://www.kernel.org/doc/man-pages/)** – seksjon 2 og 3 (systemkall og C-bibliotek) er dypere enn du tror du trenger – helt til strace-utskriften din (kapittel 4) plutselig gir mening på et nytt nivå.
- **[lore.kernel.org](https://lore.kernel.org)** – hele LKML-arkivet, søkbart. Du skal ikke *abonnere* på LKML (ingen gjør egentlig det lenger) – men å lese diskusjonen bak en beslutning du lurer på, er å se hvordan verdens største programvareprosjekt tenker høyt.

## Referanseverkene

- **[Arch Wiki](https://wiki.archlinux.org)** – fortsatt best, uansett distro (det visste du fra bok 2).
- **[systemd.io](https://systemd.io)** og `man systemd.directives` – *alle* direktivene, indeksert. Svaret på «finnes det en innstilling for …» er som regel ja.
- **[GNU-manualene](https://www.gnu.org/manual/)** (`info` eller web) – når `--help` og man-siden ikke strekker til for bash, coreutils eller make.
- **[Debian Policy](https://www.debian.org/doc/debian-policy/)** – lavmælt dokument, stor effekt: den forklarer *hvorfor* et Debian-system er skrudd sammen som det er. Å lese den er å forstå tankegangen bak halve Linux-verdenen.
- **[OpenZFS-dokumentasjonen](https://openzfs.github.io/openzfs-docs/)** – valgte du ZFS i kapittel 5, er dette hjemmet ditt.

## Menneskene

- **[Brendan Gregg](https://www.brendangregg.com)** – ytelsesanalysens far i moderne tid. Nettsiden hans og bøkene *Systems Performance* og *BPF Performance Tools* er den naturlige fortsettelsen av kapittel 3, 4 og 12. Når denne boken sa «mål først» – han skrev metodene.
- **Vedlikeholderne av verktøyene dine** – følg bugtrackerne og utgivelsesnotatene til de fem verktøyene du er mest avhengig av. Det er mer verdt enn hundre generelle nyhetsbrev.

## Praksisen som binder det sammen

Når du lurer på «hvorfor er det slik?» – ikke nøy deg med et forumsvar. Finn commiten (`git log`, `git blame` i kildekoden – kapittel 20.1), les diskusjonen (lore.kernel.org, prosjektets PR-er), og se beslutningen bli tatt. Det tar ti minutter og gir en forståelse ingen oppsummering kan.

Og til slutt, sirkelen fra kapittel 20: den dagen du *svarer* på et spørsmål på et forum, retter en man-side eller sender en patch – da er du ikke lenger bare nedstrøms for kunnskapen. Da er du en av kildene.

**Prøv selv:** (1) Les én LWN-artikkel om et emne fra denne boken. (2) Finn dokumentasjonen for en sysctl-parameter du har endret, i kjernens admin-guide. (3) Slå opp `man 2 openat` og gjenkjenn den fra strace-utskriften i kapittel 4. Kjennes det som å møte en gammel bekjent? Da har boken gjort jobben sin.

---

# Bonus: FAQ for eksperter

**Bør jeg bruke AI-assistenter i terminalen?**
Ja – de er blitt gode til å forklare, utforske og utkast-skrive. Men ekspertens regler er de samme som for curl-skript i bok 2: forstå kommandoen *før* du kjører den, verifiser mot man-siden når det gjelder (AI-er dikter opp flagg i ny og ne), og pipe aldri generert tekst rett i `sudo`. Bruk dem som en kunnskapsrik kollega du dobbeltsjekker – ikke som autopilot med root-tilgang.

**Bør jeg lære C eller Rust?**
For å bidra til kjernen eller skrive systemnære verktøy er Rust i vinden, men C er fortsatt fundamentet. Start med C for å forstå pekere og minnehåndtering. Rust kan komme senere.

**Er Kubernetes hjemme galskap?**
Som regel ja. Med mindre du eksplisitt ønsker å lære Kubernetes, gir Docker Compose (eller Podman pods) det meste du trenger med langt mindre kompleksitet. k3s er en fin mellomting, men vent til du faktisk savner en funksjon.

**Trenger jeg sertifiseringer?**
De kan åpne dører hos arbeidsgivere, men de måler ikke ekte ekspertise. Er du ute etter kunnskap, er denne boken og din egen lab langt mer verdifull. En sertifisering tvinger deg gjennom emner du kanskje ville hoppet over – vurder det som strukturert læring, ikke et mål i seg selv.

**Hvordan tar jeg backup av selve labben?**
Skill mellom *konfigurasjon* og *data*. Konfigurasjonen ER Git-repoet ditt (kapittel 7–8) – maskinene kan gjenskapes derfra, så VM-er trenger sjelden image-backup. Dataene (bilder, dokumenter, databaser) følger 3-2-1-regelen fra bok 2 med restic. Konkret per tjeneste: **Forgejo** har `forgejo dump` – ett arkiv med repoer, database og config. **Docker-volumer**: bind-mount dataene til en sti restic allerede tar (enklest), eller stopp containeren og ta volumstien under `/var/lib/docker/volumes/`. **Grafana**: ikke ta backup – *provisjoner* dashboards og datakilder som kode i Git, så er de gjenskapbare per definisjon. **Prometheus**: tidsseriene er forbruksvare (historikk er kjekt, ikke kritisk) – config og varselregler bor i Git. Ser du mønsteret? I en kodifisert labb er den beste backupen den du ikke trenger å ta.

Testen som avgjør om du har lykkes: kan du gjenskape hele labben fra Git-repoet + siste restic-snapshot? Mesterprøve-verdig øvelse.

**Må labben stå på hele døgnet?**
Nei. Del tjenestene i to: det *nettverket* trenger (DNS/Pi-hole, VPN – legg dem på en Pi til 7 W) og det *du* trenger av og til (mediaserver, byggemaskiner – Wake-on-LAN og la dem sove). En labb som bruker 100 W døgnkontinuerlig fordi «det er enklest», er en umålt labb – og du vet hva boken mener om umålte ting.

---

# Vedlegg A: Utvidet hurtigreferanse

## nftables
- `nft add table ip filter` – opprett tabell
- `nft add chain ip filter input { type filter hook input priority 0 ; }` – lag input-kjede
- `nft add rule ip filter input tcp dport 22 accept` – tillat SSH

## strace
- `strace -e trace=open,read,write -o output.log kommando` – logg spesifikke kall
- `strace -c kommando` – vis systemkallstatistikk

## mdadm
- `mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdc1` – lag RAID 1
- `mdadm --detail /dev/md0` – status

## Ansible
- `ansible all -m ping` – test forbindelse
- `ansible-playbook playbook.yml --check` – dry run

## perf
- `perf record -g kommando` – samle data
- `perf report` – vis rapport

## Observasjon og måling
- `sudo tcpdump -i eth0 port 53` – se pakkene live; `-w fil.pcap` for Wireshark
- `sudo execsnoop-bpfcc` / `opensnoop-bpfcc` – alle prosessstarter/filåpninger (eBPF)
- `iperf3 -s` / `iperf3 -c <vert>` – mål nettverksgjennomstrømning
- `fio --name=test --rw=randread --bs=4k --size=1G --runtime=30` – mål disk-IOPS
- `hyperfine 'kommando1' 'kommando2'` – benchmark kommandoer statistisk
- `systemd-analyze security <tjeneste>` – eksponerings-score og herdingssjekkliste

---

# Vedlegg B: Ordliste for eksperter

- **Cgroup:** Control group – mekanisme for å begrense, regnskapsføre og isolere ressursbruk.
- **Idempotens:** En operasjon kan gjentas uten å endre resultatet etter første gang.
- **Inode:** Datastruktur som beskriver en fil (metadata og plassering av data).
- **Namespace:** Kjernefunksjon som isolerer prosessers syn på systemressurser (PID, nett, montering).
- **OOM-killer:** Komponent som avslutter prosesser når systemet går tom for minne.
- **Page cache:** Kjernens buffer for diskinnhold i RAM.
- **RAID:** Redundant Array of Independent Disks – teknikk for speiling/striping.
- **Signal:** Asynkron melding til en prosess (f.eks. SIGTERM).
- **Swap:** Diskbasert utvidelse av RAM.
- **Trunk:** Nettverkslenke som bærer trafikk for flere VLAN.

---

# Vedlegg C: Referansearkitektur for hjemmelabben

Hele bokens prosjekt ender opp i et definert oppsett som kan gjenskapes med én kommando. Men viktigere enn *hva* arkitekturen er, er *hvordan man resonnerer seg til den* – rekkefølgen er selve poenget:

1. **DNS først** – alt annet refererer til navn. Uten egen DNS blir hver tjeneste en huskelapp med IP-adresser.
2. **Overvåking tidlig, ikke til slutt** – du kan ikke forbedre (eller feilsøke) det du ikke måler, og overvåking satt opp i fredstid er den som redder deg i krig (se hendelse #3).
3. **Backup før tjenestene får data** – dag én. En tjeneste uten backup-plan er ikke «nesten ferdig», den er ikke påbegynt.
4. **Tilgang via VPN før noe eksponeres** – porten du aldri åpnet, er porten du aldri må forsvare.
5. **Ett ansvar per vert der det går** – compute-nodene kan dø uten at DNS eller overvåking dør med dem.

Og en ærlig innrømmelse: gateway er et bevisst *single point of failure*. Hjemme aksepterer vi det (redundant gateway koster mer enn nedetiden den sparer); i produksjon hadde svaret vært et annet. Å *vite* hvor SPOF-ene dine er, er halvparten av designarbeidet.

Her er arkitekturen som diagram og det tilhørende Ansible-repoet.

**Maskiner:**
- **gateway:** Proxmox-vert med KVM, kjører Caddy, Unbound, nftables, VLAN-ruter.
- **nas:** Debian med btrfs/ZFS, NFS-server, SMART-overvåking.
- **compute1, compute2:** VMer/containere for applikasjoner (Jellyfin, Home Assistant, Git).
- **monitor:** VM med Prometheus, Grafana, Loki.

*Figuren viser: gateway er eneste vei inn (kun via VPN), tjenester og overvåking bor i adskilte soner, og alle noder rapporterer metrikker til monitor.*

**Nett:**
- VLAN 10: Ledelse (SSH, Proxmox GUI)
- VLAN 20: Tjenester (web, media)
- VLAN 30: IoT (isolert, begrenset til internett via proxy)
- Reverse proxy ruter `*.hjemme.no` til riktig tjeneste uavhengig av VLAN.

**Ansible-repo struktur:**
```
ansible-lab/
├── inventory/
│   ├── production/
│   │   ├── hosts.yml
│   │   └── group_vars/
├── playbooks/
│   ├── site.yml
│   ├── gateway.yml
│   ├── nas.yml
│   └── common.yml
├── roles/
│   ├── base/
│   ├── docker/
│   ├── monitoring/
│   └── caddy/
└── vault/  (ansible-vault krypterte filer)
```

Med `ansible-playbook -i inventory/production/hosts.yml playbooks/site.yml` blir en ny maskin en del av arkitekturen. Hele oppsettet er dokumentert i repoets README.

**Labbens byggetrinn – kapittel for kapittel:**

| Etter kapittel | Labben består av |
|----------------|------------------|
| 1–5 | Én maskin (gjerne VM) du kjenner ut og inn: kjerne, prosesser, lagring |
| 6 | + dine første Python-verktøy |
| 7 | + `ansible-lab`-repoet er født – alt herfra er kode |
| 8 | + egen Git-server med CI som tester koden din |
| 10–11 | + segmentert nett, egen DNS, reverse proxy med TLS |
| 12 | + overvåking og varsling på alle noder |
| 13–14 | + containere og VM-er provisjonert automatisk |
| 15 | + herdet, målt med `systemd-analyze security` |
| 16–19 | Du øver, pakker, deler og bidrar – labben er komplett |

**Repoet følger boken:** `ansible-lab`-repoet publiseres sammen med boken, med én commit (eller tag) per kapittel – leseren kan sjekke ut `kapittel-12` og se nøyaktig hvordan labben så ut der, eller diffe seg fremover. «Alt som kode»-prinsippet, beviselig. Repoet inneholder også `sabotasje/`-mappen med Mesterprøvens playbook – og en egen `README.md` der med regler og kjøringsinstrukser, siden erfaringsmessig hopper halvparten av leserne rett dit.

---

# Vedlegg D: Mesterprøven – ti feil å finne og fikse

Teori er lett med fasit ved siden av. Mesterprøven fjerner fasiten: en Ansible-playbook – `sabotasje.yml`, publisert med lab-repoet – planter én tilfeldig valgt feil i en labb-VM. Din jobb: finn den, forstå den, fiks den. **Ta VM-snapshot først. Aldri kjør den mot ekte maskiner.**

```yaml
# sabotasje.yml (konsept – full versjon i lab-repoet)
- hosts: offer
  become: yes
  vars:
    feil: "{{ range(1, 11) | random }}"
  tasks:
    - name: Utfør sabotasje nr. {{ feil }}
      include_tasks: "sabotasje/{{ feil }}.yml"
```

Reglene og kjøringsinstruksene ligger i `sabotasje/README.md` i lab-repoet – les den før første forsøk.

**De ti oppgavene** (stigende vanskelighetsgrad – fasit med fremgangsmåte står til slutt i lab-repoet, ikke her):

1. En tjeneste familien bruker er stoppet – og starter ikke ved omstart.
2. Disken fylles med 1 GB søppel i minuttet. Finn kilden og stopp den.
3. DNS-oppslag feiler på hele maskinen, men `ping 1.1.1.1` fungerer.
4. En fstab-linje er ødelagt – maskinen booter til nødskall ved neste omstart.
5. SSH godtar plutselig passordinnlogging igjen, og en ukjent nøkkel ligger i `authorized_keys`.
6. En cron-jobb kjører hvert minutt og spiser CPU – men `crontab -l` er tom. (Hint: cron har flere hjem.)
7. Webtjenesten svarer lokalt på serveren, men ikke fra nettverket.
8. `apt upgrade` feiler med en holdt, ødelagt pakke.
9. Loggene viser OOM-drap hver natt kl. 02. Finn synderen og sett grenser.
10. Maskinen er «treg»: én prosess med uskyldig navn bruker all I/O. Avslør og uskadeliggjør.

**Reglene:** Sett klokke (30 min per oppgave er ambisiøst, 60 er ærlig). Skriv feilsøkingslogg underveis – kommandoer og hypoteser. Sammenlign etterpå med fasitens fremgangsmåte: målet er ikke bare riktig svar, men *kortest mulig vei* dit. Klarer du 7 av 10 uten fasit, har du bestått. Klarer du å skrive din egen ellevte sabotasje – da er du ferdig med denne boken.

---

---

**Du har nå verktøyene, forståelsen og selvtilliten. Gå videre og bygg noe fantastisk. Og husk: gi tilbake til fellesskapet som har gitt deg alt dette.**
