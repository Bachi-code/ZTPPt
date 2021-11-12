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

![sql](https://user-images.githubusercontent.com/52716721/141515155-42fd0ef6-dd41-4529-9628-a86b3fde97fd.png)

![image](https://user-images.githubusercontent.com/52716721/141533062-d4f0b4b6-4639-486f-bd20-947c012012d5.png)
![image](https://user-images.githubusercontent.com/52716721/141533167-0eb874b8-da4b-470f-af05-8a3f6688f1bd.png)
![image](https://user-images.githubusercontent.com/52716721/141533938-7c74d7bb-2572-46cb-9c1b-6d3e69b38044.png)
![image](https://user-images.githubusercontent.com/52716721/141538424-7dc4155e-2069-444b-84bc-52e3b1423fa6.png)
![image](https://user-images.githubusercontent.com/52716721/141538446-29f86d74-0f14-4c76-a043-a88f993a633c.png)





