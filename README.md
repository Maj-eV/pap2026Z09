# [Programowanie Aplikacyjne] Projekt Grupy nr 9

## Skład zespołu

| Imię | Nazwisko | Nr indeksu | Uczelniany adres mailowy studenta |
| --- | --- | --- | --- |
| Adam | Wasiak | 343076 | adam.wasiak2.stud@pw.edu.pl |
| Andrzej | Buczma | 343004 | ??? |
| Krystian | Kopeć | 337570 | ??? |
| Oskar | Maj | 343033 | ??? |

## Temat zadania projektowego

### Problem

Początkujący twórcy, hobbyści i inżynierowie budujący prototypy IoT tracą godziny na dobieranie kompatybilnych podzespołów. Przeglądanie tysięcy kart katalogowych rodzi błędy decyzyjne: złe napięcia (spalony układ), brakujące rezystory czy niekompatybilne złącza. Z kolei w tradycyjnych sklepach brakuje doradztwa "pod projekt".

### Rozwiązanie

Aplikacja webowa, która na podstawie opisu (gotowego planu czy też planu uzyskanego w trakcie konwersacji) w języku naturalnym (np. "stacja meteo na baterie z powiadomieniem na telefon") projektuje architekturę sprzętową, weryfikuje kompatybilność fizyczną i buduje gotowy koszyk zakupowy (linki do zakupu poszczególnych komponentów).

### Funkcjonalność

> Fragmenty oznaczone w ten sposób oznaczają część opcjonalną/wymagającą wyboru jednej bądź wielu opcji w dalszym planowaniu.

- Chat połączony z modelem LLM, który po wybraniu docelowego dostawcy części:
    - Dowiaduje się od użytkownika o celu projektu (otrzymuje gotowy opis/dochodzi do celu pytaniami)
    - Planuje jakie kategorie komponentów i jakie ilości będą wymagane
    - Dwie możliwości wysyłania zapytania do bazy danych o części (z filtrowaniem po dostępności i dostawcy):
        > - wyszukiwanie wektorowe po embedding'ach do bazy danych dla poszczególnego poszukiwanego komponentu z filtrowaniem kategorii
        > - zwracanie $x$ wyników z każdej niezbędnej kategorii do użycia i LLM samodzielnie wybiera z każdej dużej listy danej kategorii
    - Deterministyczne zasady *ręcznie* dodane do bazy danych sprawdzają zasady inżynieryjne (poziomy napięć 3.3V vs 5V, bilans prądowy, magistrale I2C). Jeśli czujnik wymaga 5V, a mikrokontroler 3.3V - system automatycznie dokłada do koszyka konwerter napięć.
    - Na podstawie zgromadzonych części przygotowywany jest koszyk zakupowy (lista linków) zapisywany w bazie
    - Historia konwersacji jest zapisywana w bazie
    - > Opcjonalnie można do projektu użytkownika wygenerować i zapisać: przykładowy schemat połączeń lub/i przykładowy kod do mikrokontrolerów
- Użytkownik rejestruje i loguje się do prywatnego konta uzyskując dostęp do chat'a i historii projektów. Istnieje możliwość zmiany adresu e-mail oraz hasła.
- Aplikacja przechowuje historię projektów, do których użytkownik poszukiwał części (konwersacja, znalezione linki zakupowe, inne wygenerowane elementy, jeśli istnieją)
- W aplikacji może zachodzić wewnętrzna **migracja danych z API do bazy**, która polega na wysłaniu zapytania dla kilku kategorii o pewną liczbę produktów (istnieć może również możliwość wykonania migracji danych z pliku CSV), a następnie istnieją różne możliwości (niektóre uzależnione od wyboru sposobu wyszukiwania części):
    > - każdy pojedynczy produkt zwrócony w odpowiedzi API trafia do LLM'a, który generuje jego opis produktu, a na podstawie tego opisu jest tworzony embedding. LLM może również dokonywać translacji wyników z API do wewnętrznego ustalonego formatu danych (np. po otrzymaniu w JSON pola *provider_shop_link_purchase* wstawi je do wewnętrznie zdefiniowanego pola *purchase_link* w bazie, taka funkcjonalność zapewnia niezależność od jednego API, bo umożliwia korzystanie z dowolnego formatu danych zwracanego przez API).
    > - projekt jest uzależniony od jednego API o stałym formacie zwracanych danych, dane są deterministycznie wstawiane do odpowiednich pól w bazie bez przetwarzania LLM'em
    > - projekt jest uzależniony od jednego API o stałym formacie zwracanych danych, dane są deterministycznie wstawiane do odpowiednich pól w bazie oraz jest generowanych opis LLM i embedding
    Każdy produkt otrzymuje jeden rekord w bazie danych.
- W aplikacji może zachodzić wewnętrzne **odświeżenie dostępności produktów**, polegające na wysłaniu jednego z poniższych do wyboru rodzajów zapytań i zaktualizowaniu jego statusu dostępności produktu:
    > - dla każdego dostępnego produktu
    > - tylko dla produktów z najmniejszą dostępną ilością (poniżej ustalonej wartości granicznej)
- Konto administratora pozwalające na uruchomienie migracji danych z API do bazy, odświeżenia dostępności produktów oraz aktualizacji zasad inżynieryjnych
- > Konto administratora pozwala sprawdzić ogólne statystyki:
    > - Historię migracji danych z API do bazy - ich poszczególne daty, statusy oraz ilości zaindeksowanych produktów
    > - Historię odświeżenia dostępności produktów - poszczególne daty, statusy oraz ilości zaktualizowanych produktów
    > - Dotyczące ilości użytkowników oraz ilości/średniej liczby utworzonych przez nich konwersacji

### Architektura

Styl architektoniczny: SPA + REST API + Relacyjno-wektorowa baza danych

> Czy migracja danych z API do bazy lub odświeżenie dostępności produktów wymaga kolejki zadań i worker'ów je przetwarzających?

#### Frontend

Aplikacja webowa z podziałem na:
- ekran logowania, rejestracji
- dostęp zwykłego użytkownika:
    - chat
    - historię konwersacji
    - panel zarządzania kontem
- panel administratora:
    - uruchamianie migracji
    - statystyki
    - aktualizacja zasad inżynieryjnych

#### Backend

REST API z podziałem na moduły:
- konta użytkownika (logowanie, rejestracja, zmiana hasła/e-maila)
- chat'u (konweracji z LLM)
- historii konwersacji
- migracji danych z API do bazy
- dostępności produktów
- > statystyk

### Technologie

- SPA: **React**
- > Biblioteka komponentów do SPA: ???
- REST API: **Java** + framework: **Spring Boot**
- Relacyjno-wektorowa baza danych: **PostgreSQL + pgvector**
- API z danymi: **nexar.com**
- Dostawca LLM: **OpenAI**
- ORM: **Hibernate**
- > Logowanie i rejestracja: Keycloak/Clerk???
- Konteneryzacja: **Docker, Docker Compose**
- > Automatyzacja budowania API: Maven/Gradle???
- Zapytania do dostawcy LLM: **Spring AI**
