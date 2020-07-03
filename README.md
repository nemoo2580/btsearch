# BTSearch v2
Repozytorium zawierające kompletny kod źródłowy serwisu [beta.btsearch.pl](http://beta.btsearch.pl), napisany w języku Python z wykorzystaniem frameworku webowego Django.

## Lokalne środowisko developerskie
Uruchomienie projektu w lokalnym środowisku pozwoli na bezpośrednie grzebanie w kodzie, wdrażanie poprawek, usprawnień czy nowych funkcji. Poniżej zapis kroków, które sam wykonałem, aby projekt lokalnie uruchomić.

Lokalne środowisko osobiście odpalam w Linuxie (Ubuntu-18.04), zainstalowanym ze sklepu Microsoft Store wewnątrz Windowsa 10. To z kolei wymaga aktywacji WSL (Windows Subsystem for Linux) w Windows 10. Jeśli działasz natywnie w Linuxie, oczywiście pomijasz etap związany z uruchomieniem Linuxa w Windowsie.

**TL;DR dla tych bardziej w temacie**
Projekt btsearch wymaga zainstalowania `python` (2.x), menedżera pakietów pythonowych `pip`, serwera `mysql-server` oraz `virtualenv`. Kod źródłowy klonujemy z własnego forka GitHub. Następnie wewnątrz utworzonego i aktywowanego virtualenv instalujemy projektowe *dependencies* (django, south, mysql-python itp.). Następnie za pośrednictwem konsoli Django `manage.py` tworzymy strukturę bazy i odpalamy webserver na porcie 8000.

### Aktywacja WSL oraz instalacja Ubuntu-18.04 wewnątrz Windows 10
Wykonujemy krok po kroku instrukcje z tego przewodnika:
https://docs.microsoft.com/en-us/windows/wsl/install-win10

Ja u siebie działam na nowszym subsystemie WSL v2. Polecam także aplikację [Windows Terminal](https://www.microsoft.com/pl-pl/p/windows-terminal/9n0dx20hk701?activetab=pivot:overviewtab) z Microsoft Store. Umożliwia wygodniejszą pracę w porównaniu do domyślnej powłoki linuxowej w Windowsie.

### Instalacja niezbędnych pakietów w Ubuntu
Po zainstalowaniu i uruchomieniu Ubuntu-18.04 w Windows 10 instalujemy niezbędne do dalszej pracy pakiety za pomocą `apt-get`.

Aktualizacja repozytoriów z pakietami:
```sh
$ sudo apt-get update
```
Instalacja Pythona 2.x (na nim _jeszcze_ jest oparty projekt):
```sh
$ sudo apt-get install python
```
Instalacja menedżera pakietów pythonowych `pip`:
```sh
$ sudo apt-get install python-pip
```
Instalacja serwera mysql:
```sh
$ sudo apt-get install mysql-server
$ sudo apt-get install libmysqlclient-dev
```
    Tutaj istotna uwaga - szczególnie jeśli uruchamiasz projekt w WSL/Windows. W trakcie instalacji mysql-server _teoretycznie_ powinna nastąpić wstępna konfiguracja serwera, z podaniem nazwy użytkownika i hasła administratora serwera. U mnie nie wiedzieć czemu to nie nastąpiło. Guglałem za rozwiązaniem problemu, ale poszedłem na skróty i posiłkowałem się hasłem dla systemowego usera `debian-sys-maint`, które podane jest czystym tekstem w pliku `/etc/mysql/debian.cnf`. Z tą kombinacją user/pass zalogowałem się do serwera (klient dowolny - z linii poleceń `mysql`, phpmyadmin, MySQL Workbench - co kto woli) i utworzyłem osobną kombinację user/pass oraz pustą bazę do projektu btsearch.

### Klonowanie kodu źródłowego btsearch z GitHub'a
Następnie tworzymy katalog `/home/USERNAME/dev/` (lub inny wg uznania), do którego sklonujemy repozytorium z kodem źródłowym projektu btsearch. **Uwaga!** Najpierw w GitHub zrób fork repozytorium na swoje konto GH. Wówczas  w miejsce `XYZ` wpisz swój username z GH. Dzięki temu klonujesz swój fork repozytorium z Twojego własnego konta GH.

```sh
$ mkdir dev
$ cd dev
$ git clone git@github.com:XYZ/btsearch.git
$ cd btsearch
```
Po wykonaniu w/w kroków, bieżącą ścieżką powinno być `/home/USERNAME/dev/btsearch/`.

### Utworzenie i aktywacja virtualenv
Pakiety pythonowe niezbędne do uruchomienia projektu btsearch instalujemy wewnątrz virtualenv'a - czyli wirtualnego środowiska pythonowego, niezależnego od pakietów globalnych (widocznych dla całego systemu). Zamykając niezbędne dla projektu pakiety wewnątrz virtualenv nie zaśmiecamy systemu - i potencjalnie innych, sąsiadujących projektów pythonowych - pakietami, które poza uruchomieniem tego konkretnego projektu nie są do niczego potrzebne.

Instalacja i utworzenie virtualenv o nazwie `venv-btsearch` w folderze domowym `/home/USERNAME/venv-btsearch/`:
```sh
$ pip install virtualenv
$ cd ~
$ virtualenv venv-btsearch
$ . ~/venv-btsearch/bin/activate
```
Po utworzeniu i aktywacji virtualenv'a wg wskazówek powyżej, w przedrostku linii poleceń powinna znaleźć się nazwa aktywowanego virtualenv'a:
```sh
(venv-btsearch) adlorenz@komputer btsearch $
```

### Instalacja projektowych zależności
Po aktywacji virtualenv'a można przystąpić do zainstalowania pakietów pythonowych, niezbędnych do uruchomienia projektu btsearch. Zakładając, że bieżącą ścieżką jest `/home/USERNAME/dev/btsearch/`, odpalamy instalację zależności z pliku `requirements.txt`:
```sh
$ pip install -r src/deploy/requirements.txt
```
**Uwaga!** Istnieje prawdopodobieństwo, że w instalacja wywali się na etapie pobierania i kompilowania pakietu `uwsgi`. Na moment pisania tego przewodnika jeszcze nie znalazłem rozwiązania tego problemu, dlatego na wypadek wystąpienia tej usterki, w pliku `src/deploy/requirements.txt` chwilowo zakomentowujemy linię:
```sh
#uwsgi==1.9.19
```
I odpalamy proces instalacji poleceniem `pip install -r src/deploy/requirements.txt` jeszcze raz.

### Kofiguracja dostępu do bazy danych dla projektu
W pliku `src/conf/local.py` uzupełniamy user/pass/dbname do lokalnej bazy danych mysql, którą konfigurowaliśmy kilka kroków wyżej.
```sh
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'USER': '__user__',
        'PASSWORD': '__password__',
        'HOST': 'localhost',
        'NAME': '__dbname__',
    }
}
```

### Uruchomienie konsoli Django
(Przed)ostatni krok to próba uruchomienia konsoli Django `manage.py`. Jeśli otrzymasz wynik jak poniżej - sukces! Projekt jest (prawie) gotowy do lokalnej pracy. :)
```sh
$ cd src
$ ./manage.py --version
1.5
```

### Utworzenie struktury bazy danych i uruchomienie webserwera projektowego
Ostatnim krokiem jest utworzenie struktury bazy danych projektu btsearch. Odpalamy dwie komendy - `syncdb` tworzy domyślną strukturę i tabele wspólne dla Django, natomiast `migrate` tworzy i migruje strukturę modeli aplikacji btsearch (tabele mysql, w których trzymane są dane BTS, UKE itp.).
```sh
$ ./manage.py syncdb
$ ./manage.pu migrate
```
Na koniec pozostaje uruchomienie wewnętrznego webservera Django:
```sh
$ ./manage.py runserver
Validating models...

0 errors found
July 04, 2020 - 01:06:06
Django version 1.5, using settings 'src.settings'
Development server is running at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```
Po ujrzeniu w/w rezultatu, odpalamy przeglądarkę i wpisujemy URL `http://127.0.0.1:8000/`.

Voila!

## Pro-tipy

**Jak się dostać do linuxowego systemu plików w Windowsie?**
Podając taką ścieżkę, np. w Eksploratorze Windows: `\\wsl$\DISTRONAME\`, czyli np. `\\wsl$\Ubuntu-18.04\`

## Co dalej?
Na tym etapie mamy w pełni skonfigurowany i odpalony lokalnie projekt btsearch, który ładnie ładuje się w przeglądarce - aczkolwiek baza danych jest pusta, więc na mapce nie wyświetlają się żadne dane. W planach na uporządkowanie kodu źródłowego w bliżej nieokreślonej przyszłości mam m.in.:

- Przygotowanie testowych danych do bazy danych dla środowiska lokalnego.
- Rozwiązanie problemu z instalacją uwsgi.
- Uaktualnienie zależności projektowych (`requirements.txt`) - obecne są bardzo przestarzałe i już od dawna nie wspierane, np. Django 1.5 - przydałaby się aktualizacja do 2.x.
- Migracja do Python 3.x.

## Kolaboracja i kontakt
Jeśli dobrnąłeś/-aś z sukcesem do tego momentu i masz projekt btsearch lokalnie odpalony w przeglądarce pod adresem `http://127.0.0.1:8000`, to znaczy, że jesteś gotowy/-a do samodzielnego wdrażania poprawek, usprawnień i zmian w kodzie źródłowym.

Gorąco zatem zachęcam do wsparcia i kolaboracji przy projekcie. Kontaktujcie się na maila d.lorenz@btsearch.pl lub poprzez GitHub (Issues). Dzięki z góry!

Dawid Lorenz