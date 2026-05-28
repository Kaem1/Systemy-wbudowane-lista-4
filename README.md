# Diagramy – Paczkomat
## PU1–PU5: Diagramy decyzyjne i diagramy przepływu

---

## PU1 – Odbiór paczki

### Diagram decyzyjny (PU1)

```mermaid
flowchart TD
    A([Start]) --> B{Połączenie\nz systemem?}
    B -- NIE --> ERR1[/Operacja niemożliwa/]
    ERR1 --> Z1([Koniec])
    B -- TAK --> C[Użytkownik wybiera\n'Odbiór paczki']
    C --> D[Wprowadza kod\nlub skanuje QR]
    D --> E{Kod\npoprawny?}
    E -- NIE --> ERR2[/Dostęp\nzablokowany/]
    ERR2 --> Z2([Koniec])
    E -- TAK --> F{Wymagana\npłatność?}
    F -- NIE --> G[Otwórz skrytkę]
    F -- TAK --> H{Płatność\nzatwierdzona?}
    H -- NIE --> ERR3[/Brak wydania\npaczki/]
    ERR3 --> Z3([Koniec])
    H -- TAK --> G
    G --> I[Użytkownik odbiera paczkę]
    I --> J[Rejestracja odbioru\nzamknięcie skrytki]
    J --> Z4([Koniec])
```

---

### Diagram przepływu (PU1)

```mermaid
flowchart TD
    A([Start]) --> B[Użytkownik wybiera\nopcję 'Odbiór paczki']
    B --> C{Użytkownik\nanuluje?}
    C -- TAK --> Z1([Koniec – anulowanie])
    C -- NIE --> D[Wprowadzenie kodu\nlub skanowanie QR]
    D --> E[System weryfikuje\ndane w systemie centralnym]
    E --> F{Weryfikacja\npomyślna?}
    F -- NIE\nbłędny kod --> ERR1[Blokada dostępu\nzapis próby]
    ERR1 --> Z2([Koniec – błąd])
    F -- TAK --> G[System sprawdza\nstatus płatności]
    G --> H{Wymagana\npłatność?}
    H -- NIE --> I[System otwiera skrytkę\nsygnał rygla]
    H -- TAK --> J[Wyświetlenie kwoty\nprzekazanie do terminala]
    J --> K[Użytkownik dokonuje\npłatności]
    K --> L{Płatność\npotwierdzona?}
    L -- NIE --> ERR2[Odmowa wydania\nanulowanie transakcji]
    ERR2 --> Z3([Koniec – odmowa])
    L -- TAK --> I
    I --> M[Użytkownik odbiera paczkę]
    M --> N[Skrytka zamykana\npo odebraniu]
    N --> O[Rejestracja zdarzenia\nwysyłka do systemu centralnego]
    O --> Z4([Koniec])
```

### Diagram stanów (PU1)

```mermaid
stateDiagram-v2
    [*] --> Oczekiwanie : użytkownik wybiera "Odbiór paczki"
 
    Oczekiwanie --> WeryfikacjaPolaczenia : żądanie odbioru
 
    WeryfikacjaPolaczenia --> WprowadzanieKodu : połączenie OK
    WeryfikacjaPolaczenia --> BrakPolaczenia : brak połączenia
 
    BrakPolaczenia --> [*] : operacja niemożliwa
 
    WprowadzanieKodu --> WeryfikacjaKodu : kod / QR podany
 
    WeryfikacjaKodu --> SprawdzaniePlatnosci : kod poprawny
    WeryfikacjaKodu --> KodNiepoprawny : błędny kod
 
    KodNiepoprawny --> [*] : dostęp zablokowany
 
    SprawdzaniePlatnosci --> ObslugaPlatnosci : płatność wymagana
    SprawdzaniePlatnosci --> SkrytkaOtwarta : płatność niewymagana
 
    ObslugaPlatnosci --> OczekiwanieNaPlatnosc : terminal aktywowany
    OczekiwanieNaPlatnosc --> SkrytkaOtwarta : płatność zatwierdzona
    OczekiwanieNaPlatnosc --> PlatnoscOdrzucona : płatność nieudana
    OczekiwanieNaPlatnosc --> AnulowaniePlatnosci : użytkownik anuluje
 
    PlatnoscOdrzucona --> [*] : brak wydania paczki
    AnulowaniePlatnosci --> [*] : operacja anulowana
 
    SkrytkaOtwarta --> OdbiórPaczki : użytkownik odbiera paczkę
    OdbiórPaczki --> RejestrowanieOdbioru : skrytka zamknięta
    RejestrowanieOdbioru --> [*] : log wysłany do systemu centralnego
```

### Diagram sekwencji (PU1)

```mermaid
sequenceDiagram
    autonumber
    actor Klient
    participant Ekran as Ekran dotykowy
    participant Skaner as Skaner kodów
    participant Komputer as Komputer centralny
    participant Modul as Moduł łączności
    participant SystemC as System centralny
    participant Terminal as Terminal płatniczy
    participant SystemP as System płatniczy
    participant Sterownik as Sterownik skrytek
    participant Zaczepy as Elektrozaczepki

    Note over Klient, Komputer: KROK 1: Wprowadzenie danych (jedna z opcji)
    alt Klient używa klawiatury
        Klient->>Ekran: Wpisuje kod
        Ekran->>Komputer: Kod paczki
    else Klient używa aplikacji
        Klient->>Skaner: Skanuje kod QR
        Skaner->>Komputer: Zdeserializowany kod QR
    end

    Note over Komputer, SystemC: KROK 2: Weryfikacja paczki
    Komputer->>Modul: Zapytanie o dane paczki
    Modul->>SystemC: Zapytanie (zaszyfrowane HTTPS)
    SystemC->>Modul: Informacja zwrotna na temat paczki*
    Modul->>Komputer: Wypakowana informacja na temat paczki

    Note over Komputer, Zaczepy: KROK 3: Obsługa statusu paczki
    alt Paczki nie ma w systemie
        Komputer->>Ekran: Informacja o niepoprawnym kodzie
    else Wymagana jest płatność
        Komputer->>Terminal: Informacja o kwocie do zapłaty
        Terminal->>SystemP: Zapytanie o obciążenie konta z karty
        
        alt Płatność pomyślna
            SystemP->>Terminal: Informacja o pomyślności płatności
            Terminal->>Komputer: Informacja o pomyślności płatności
        else Płatność niepomyślna
            SystemP->>Terminal: Brak środków / błąd
            Terminal->>Komputer: Informacja o błędzie płatności
            Komputer->>Ekran: Informacja o niepomyślnej płatności
        end
    end

    opt Paczka w systemie (Opłacona lub nie wymaga dopłaty)
        Note over Komputer, Zaczepy: KROK 4: Wydanie paczki
        Komputer->>Ekran: Informacja o otwarciu skrytki
        Komputer->>Sterownik: Informacja o otwarciu skrytki + jej numer
        Sterownik->>Zaczepy: Sygnał otwierający skrytkę
    end
```

---

## PU2 – Nadanie paczki

### Diagram decyzyjny (PU2)

```mermaid
flowchart TD
    A([Start]) --> B{Połączenie\nz systemem?}
    B -- NIE --> ERR1[/Operacja niemożliwa/]
    ERR1 --> Z1([Koniec])
    B -- TAK --> C[Użytkownik wybiera\n'Nadaj paczkę']
    C --> D{Użytkownik\nrezygnuje?}
    D -- TAK --> Z2([Koniec – anulowanie])
    D -- NIE --> E[Wprowadzenie danych\nprzesyłki]
    E --> F{Wolna skrytka\ndostępna?}
    F -- NIE --> ERR2[/Brak wolnych skrytek\noperacja przerwana/]
    ERR2 --> Z3([Koniec])
    F -- TAK --> G[System przydziela\nskrytkę]
    G --> H[Skrytka otwierana]
    H --> I{Paczka umieszczona\nw skrytce?}
    I -- NIE\nTimeout --> ERR3[/Anulowanie\noperacji/]
    ERR3 --> Z4([Koniec])
    I -- TAK --> J[Skrytka zamykana]
    J --> K[Generowanie kodu\nodbioru]
    K --> Z5([Koniec])
```

---

### Diagram przepływu (PU2)

```mermaid
flowchart TD
    A([Start]) --> B[Użytkownik wybiera\nopcję 'Nadaj paczkę']
    B --> C{Użytkownik\nrezygnuje?}
    C -- TAK --> Z1([Koniec – anulowanie])
    C -- NIE --> D[Użytkownik wprowadza dane\nlub używa aplikacji mobilnej]
    D --> E[System sprawdza dostępność\nwolnych skrytek]
    E --> F{Wolna skrytka\ndostępna?}
    F -- NIE --> ERR1[Informacja o braku\nwolnych skrytek]
    ERR1 --> Z2([Koniec – brak skrytki])
    F -- TAK --> G[System przydziela skrytkę\nID, rozmiar, lokalizacja]
    G --> H[System otwiera skrytkę\nsygnał rygla]
    H --> I[Użytkownik umieszcza\npaczkę w skrytce]
    I --> J[System zamyka skrytkę\ni weryfikuje zamknięcie]
    J --> K[System generuje\nkod odbioru UUID]
    K --> L[Rejestracja przesyłki\nw systemie centralnym]
    L --> M[Kod wysyłany\ndo odbiorcy]
    M --> Z3([Koniec])
```

### Diagram stanów (PU2)

```mermaid
stateDiagram-v2
    [*] --> Oczekiwanie : użytkownik wybiera "Nadaj paczkę"
 
    Oczekiwanie --> WprowadzanieDanych : akcja użytkownika
    Oczekiwanie --> [*] : użytkownik rezygnuje
 
    WprowadzanieDanych --> SprawdzanieSkrytek : dane zatwierdzone
    WprowadzanieDanych --> [*] : użytkownik anuluje
 
    SprawdzanieSkrytek --> PrzydzielanieSrytki : wolna skrytka dostępna
    SprawdzanieSkrytek --> BrakSkrytek : brak wolnych skrytek
 
    BrakSkrytek --> [*] : operacja przerwana
 
    PrzydzielanieSrytki --> SkrytkaOtwarta : sygnał otwarcia rygla
 
    SkrytkaOtwarta --> OczekiwanieNaPaczke : skrytka gotowa
 
    OczekiwanieNaPaczke --> SkrytkaZamknieta : paczka umieszczona
    OczekiwanieNaPaczke --> Timeout : upłynął czas oczekiwania
 
    Timeout --> [*] : operacja anulowana, skrytka zamknięta
 
    SkrytkaZamknieta --> GenerowanieKodu : zamknięcie potwierdzone
    GenerowanieKodu --> RejestrowanieNadania : kod UUID wygenerowany
    RejestrowanieNadania --> WysylanieKodu : log w systemie centralnym
    WysylanieKodu --> [*] : kod wysłany do odbiorcy
```

### Diagram sekwencji (PU2)

```mermaid
sequenceDiagram
    autonumber
    actor Klient
    participant Ekran as Ekran dotykowy
    participant Skaner as Skaner kodów
    participant Komputer as Komputer centralny
    participant Modul as Moduł łączności
    participant SystemC as System centralny
    participant Terminal as Terminal płatniczy
    participant SystemP as System płatniczy
    participant Sterownik as Sterownik skrytek
    participant Zaczepy as Elektrozaczepki

    Note over Klient, Komputer: KROK 1: Wprowadzenie danych (jedna z opcji)
    alt Klient używa klawiatury
        Klient->>Ekran: Wpisuje kod nadania
        Ekran->>Komputer: Kod nadania paczki
    else Klient używa aplikacji
        Klient->>Skaner: Skanuje kod QR
        Skaner->>Komputer: Zdeserializowany kod QR
    end

    Note over Komputer, Zaczepy: KROK 2: Weryfikacja wolnej skrytki
    Komputer->>Sterownik: Zapytanie o wolną skrytkę z uwzględnieniem wielkości paczki
    alt Nie ma wolnej skrytki
        Sterownik->>Komputer: Informacja o braku wolnej skrytki
        Komputer->>Ekran: Informacja o braku wolnej skrytki
   else Wolna skrytka
       Sterownik->>Komputer: Informacja o wolnej skrytce (numer)
   end
   
    Note over Komputer, Zaczepy: KROK 3: Otwarcie skrytki
    Komputer->>Sterownik: Prośba o otwarcie konkretnej skrytki
    Sterownik->>Zaczepy: Sygnał otwierający skrytkę
    alt Skrytka nie została otwarta
        Zaczepy->>Sterownik: Sygnał, że skrytka nie została otwarta
        Sterownik->>Komputer: Informacja o problemie ze skrytką
        Komputer->>Ekran: Informacja o problemie technicznej i prośba o ponownie próby
    else Skrytka została otwarta
        Zaczepy->>Sterownik: Sygnał, że skrytka została otwarta
        Sterownik->>Komputer: Informacja o otwarciu skrytki
        Komputer->>Ekran: Prośba o włożenie paczki i zamknięcie skrytki
    end

    opt Skrytka została zamknięta
        Note over Komputer, Zaczepy: KROK 4: Wysłanie informacji do systemu centralnego
        Komputer->>Modul: Informacja o nadaniu paczki + kod nadania
        Modul->>SystemC: Informacja o nadaniu paczki zapakowana w protokół HTTPS
    end
```

---

## PU3 – Dostarczenie paczki przez kuriera

### Diagram decyzyjny (PU3)

```mermaid
flowchart TD
    A([Start]) --> B[Kurier loguje się\ndo systemu]
    B --> C{Autoryzacja\nkuriera?}
    C -- NIE\nbłąd danych --> ERR1[/Brak dostępu/]
    ERR1 --> Z1([Koniec])
    C -- TAK --> D{Paczki\ndo dostarczenia?}
    D -- NIE --> ERR2[/Brak przypisanych\npaczek/]
    ERR2 --> Z2([Koniec])
    D -- TAK --> E[System identyfikuje\nskrytki do zapełnienia]
    E --> F{Skrytka\ndostępna?}
    F -- NIE --> ERR3[/Skrytka zajęta\nlub uszkodzona/]
    ERR3 --> G
    F -- TAK --> G{Kolejna\nskrytka?}
    G -- TAK --> F
    G -- NIE --> H[Wszystkie skrytki\notwarte]
    H --> I[Kurier umieszcza paczki]
    I --> J[System zamyka skrytki]
    J --> K[Dane wysyłane\ndo systemu centralnego]
    K --> Z3([Koniec])
```

---

### Diagram przepływu (PU3)

```mermaid
flowchart TD
    A([Start]) --> B[Kurier skanuje dane\nlub wpisuje login]
    B --> C[System weryfikuje\ntożsamość kuriera]
    C --> D{Autoryzacja\npomyślna?}
    D -- NIE --> ERR1[Komunikat o błędzie\nlogowania]
    ERR1 --> Z1([Koniec – brak dostępu])
    D -- TAK --> E[System pobiera listę\npaczek do dostarczenia]
    E --> F[System otwiera\nprzypisane skrytki]
    F --> G[Kurier dostarcza paczki\ndo otwartych skrytek]
    G --> H{Kurier pomija\nskrytkę?}
    H -- TAK --> I[Skrytka pozostaje\npusta, log zdarzenia]
    H -- NIE --> J[Paczka umieszczona\nw skrytce]
    I --> K{Kolejna\nskrytka?}
    J --> K
    K -- TAK --> G
    K -- NIE --> L[System zamyka\nwszystkie skrytki]
    L --> M[Aktualizacja statusów\nw systemie centralnym]
    M --> N[Powiadomienia do\nodpowiednich użytkowników]
    N --> Z2([Koniec])
```

### Diagram stanów (PU3)

```mermaid
stateDiagram-v2
    [*] --> Logowanie : kurier inicjuje sesję
 
    Logowanie --> WeryfikacjaKuriera : dane podane
    WeryfikacjaKuriera --> PobieranieListy : autoryzacja OK
    WeryfikacjaKuriera --> BladAutoryzacji : błędne dane
 
    BladAutoryzacji --> [*] : brak dostępu
 
    PobieranieListy --> OtwieranieSrytek : lista skrytek pobrana
    PobieranieListy --> BrakPaczek : brak przypisanych paczek
 
    BrakPaczek --> [*] : sesja zakończona
 
    OtwieranieSrytek --> DostarczaniePaczek : skrytki otwarte
 
    DostarczaniePaczek --> PaczkaUmieszczona : kurier wkłada paczkę
    DostarczaniePaczek --> SkrytkaPominięta : kurier pomija skrytkę
 
    PaczkaUmieszczona --> SprawdzanieKolejnej : log umieszczenia
    SkrytkaPominięta --> SprawdzanieKolejnej : log pominięcia
 
    SprawdzanieKolejnej --> DostarczaniePaczek : kolejna skrytka
    SprawdzanieKolejnej --> ZamykanieSkrytek : wszystkie obsłużone
 
    ZamykanieSkrytek --> AktualizacjaStatusow : sygnały zamknięcia
    AktualizacjaStatusow --> WysyłaniePowiadomien : log w systemie centralnym
    WysyłaniePowiadomien --> [*] : powiadomienia do użytkowników wysłane
```

### Diagram sekwencji (PU3)

```mermaid
sequenceDiagram
    autonumber
    actor Kurier
    actor Klient
    participant Ekran as Ekran dotykowy
    participant Skaner as Skaner kodów
    participant Komputer as Komputer centralny
    participant Modul as Moduł łączności
    participant SystemC as System centralny
    participant Terminal as Terminal płatniczy
    participant SystemP as System płatniczy
    participant Sterownik as Sterownik skrytek
    participant Zaczepy as Elektrozaczepki


    Note over Kurier, Komputer: KROK 1: Wprowadzenie danych (jedna z opcji)
    alt Kurier używa klawiatury
        Kurier->>Ekran: Wpisuje kod logowania
        Ekran->>Komputer: Dane logowania
        alt Autoryzacja nie przebiegła pomyślnie
            Komputer-->Ekran: Błędne dane logowania
        else Autoryzacja przebiegła pomyślnie
            Komputer-->Ekran: Wybierz czynność
        end
    else Kurier używa aplikacji
        Kurier->>Skaner: Skanuje kod QR
        Skaner->>Komputer: Zdeserializowany kod QR
        alt Autoryzacja nie przebiegła pomyślnie
            Komputer-->Ekran: Błędne dane logowania
        else Autoryzacja przebiegła pomyślnie
            Komputer-->Ekran: Wybierz czynność
        end
    end

    opt Wybrana czynność: Dostarczam paczki
       Note over Komputer, Zaczepy: Dla każdej paczki dostarczanej przez kuriera
       Note over Komputer, Zaczepy: KROK 2: Weryfikacja wolnej skrytki
       Komputer->>Sterownik: Zapytanie o wolną skrytkę z uwzględnieniem wielkości paczki
       alt Nie ma wolnej skrytki
           Sterownik->>Komputer: Informacja o braku wolnej skrytki
           Komputer->>Ekran: Informacja o braku wolnej skrytki
       else Wolna skrytka
           Sterownik->>Komputer: Informacja o wolnej skrytce (numer)
       end
   end

    opt Weryfikacja wolnej skrytki przebiegła pomyślnie
        Note over Komputer, Zaczepy: KROK 3: Otwarcie skrytki
        Komputer->>Sterownik: Prośba o otwarcie konkretnej skrytki
        Sterownik->>Zaczepy: Sygnał otwierający skrytkę
        alt Skrytka nie została otwarta
            Zaczepy->>Sterownik: Sygnał, że skrytka nie została otwarta
            Sterownik->>Komputer: Informacja o problemie ze skrytką
            Komputer->>Ekran: Informacja o problemie technicznej i prośba o ponownie próby
        else Skrytka została otwarta
            Zaczepy->>Sterownik: Sygnał, że skrytka została otwarta
            Sterownik->>Komputer: Informacja o otwarciu skrytki
            Komputer->>Ekran: Prośba o włożenie paczki i zamknięcie skrytki
        end
    end

    opt Skrytka została zamknięta
        Note over Komputer, Zaczepy: KROK 4: Wysłanie informacji do systemu centralnego
        Komputer->>Modul: Informacja o dostarczeniu paczki
        Modul->>SystemC: Informacja Informacja o dostarczeniu paczki zapakowana w protokół HTTPS
        SystemC->>Klient: Kod odbioru + Kod QR
    end
```

---

## PU4 – Odbiór paczek przez kuriera

### Diagram decyzyjny (PU4)

```mermaid
flowchart TD
    A([Start]) --> B[Kurier loguje się\ndo systemu]
    B --> C{Autoryzacja\nkuriera?}
    C -- NIE --> ERR1[/Brak dostępu/]
    ERR1 --> Z1([Koniec])
    C -- TAK --> D[Kurier wybiera opcję\n'Odbiór paczek']
    D --> E{Paczki gotowe\ndo odbioru?}
    E -- NIE --> ERR2[/Brak paczek\ndo odbioru/]
    ERR2 --> Z2([Koniec])
    E -- TAK --> F[System identyfikuje\nskrytki z paczkami]
    F --> G{Skrytka\ngotowa?}
    G -- NIE\nzajęta/błąd --> ERR3[/Pominięcie\nskrytki/]
    ERR3 --> H
    G -- TAK --> H{Kolejna\nskrytka?}
    H -- TAK --> G
    H -- NIE --> I[System otwiera\nwszystkie skrytki]
    I --> J{Kurier pomija\nskrytkę?}
    J -- TAK --> K[Log pominięcia\nskrytki]
    J -- NIE --> L[Kurier odbiera paczkę]
    K --> M{Kolejna\npaczka?}
    L --> M
    M -- TAK --> J
    M -- NIE --> N[Skrytki zamykane]
    N --> O[Aktualizacja statusów\nw systemie centralnym]
    O --> Z3([Koniec])
```

---

### Diagram przepływu (PU4)

```mermaid
flowchart TD
    A([Start]) --> B[Kurier loguje się\ni wybiera 'Odbiór paczek']
    B --> C[System weryfikuje\ndane kuriera]
    C --> D{Autoryzacja\npomyślna?}
    D -- NIE --> ERR1[Komunikat o błędzie]
    ERR1 --> Z1([Koniec – brak dostępu])
    D -- TAK --> E[System identyfikuje skrytki\nz paczkami do odbioru]
    E --> F{Są paczki\ndo odbioru?}
    F -- NIE --> ERR2[Komunikat: brak\npaczek do odbioru]
    ERR2 --> Z2([Koniec – brak paczek])
    F -- TAK --> G[System otwiera\nprzypisane skrytki]
    G --> H[Kurier odbiera paczki\nz otwartych skrytek]
    H --> I{Kurier pomija\nskrytkę?}
    I -- TAK --> J[Skrytka pozostaje zamknięta\nrejestracja zdarzenia]
    I -- NIE --> K[Potwierdzenie odbioru\npaczki ze skrytki]
    J --> L{Kolejna\nskrytka?}
    K --> L
    L -- TAK --> H
    L -- NIE --> M[System zamyka\nwszystkie skrytki]
    M --> N[Aktualizacja statusów przesyłek\nw systemie centralnym]
    N --> Z3([Koniec])
```

### Diagram stanów (PU4)

```mermaid
stateDiagram-v2
    [*] --> Logowanie : kurier inicjuje sesję odbioru
 
    Logowanie --> WeryfikacjaKuriera : dane podane
    WeryfikacjaKuriera --> IdentyfikacjaSkrytek : autoryzacja OK
    WeryfikacjaKuriera --> BladAutoryzacji : błędne dane
 
    BladAutoryzacji --> [*] : brak dostępu
 
    IdentyfikacjaSkrytek --> OtwieranieSrytek : skrytki z paczkami znalezione
    IdentyfikacjaSkrytek --> BrakPaczek : brak paczek do odbioru
 
    BrakPaczek --> [*] : sesja zakończona – nic do odbioru
 
    OtwieranieSrytek --> OdbiorPaczek : skrytki otwarte
 
    OdbiorPaczek --> PaczkaOdebrana : kurier odbiera paczkę
    OdbiorPaczek --> SkrytkaPominięta : kurier pomija skrytkę
 
    PaczkaOdebrana --> SprawdzanieKolejnej : potwierdzenie odbioru
    SkrytkaPominięta --> SprawdzanieKolejnej : log pominięcia
 
    SprawdzanieKolejnej --> OdbiorPaczek : kolejna skrytka
    SprawdzanieKolejnej --> ZamykanieSkrytek : wszystkie obsłużone
 
    ZamykanieSkrytek --> AktualizacjaStatusow : sygnały zamknięcia rygla
    AktualizacjaStatusow --> [*] : statusy przesyłek zaktualizowane w systemie centralnym
```

### Diagram sekwencji (PU4)

```mermaid
sequenceDiagram
    autonumber
    actor Kurier
    participant Ekran as Ekran dotykowy
    participant Skaner as Skaner kodów
    participant Komputer as Komputer centralny
    participant Modul as Moduł łączności
    participant SystemC as System centralny
    participant Terminal as Terminal płatniczy
    participant SystemP as System płatniczy
    participant Sterownik as Sterownik skrytek
    participant Zaczepy as Elektrozaczepki


    Note over Kurier, Komputer: KROK 1: Wprowadzenie danych (jedna z opcji)
    alt Kurier używa klawiatury
        Kurier->>Ekran: Wpisuje dane logowania
        Ekran->>Komputer: Dane logowania
        alt Autoryzacja nie przebiegła pomyślnie
            Komputer-->Ekran: Błędne dane logowania
        else Autoryzacja przebiegła pomyślnie
            Komputer-->Ekran: Wybierz czynność
        end
    else Kurier używa aplikacji
        Kurier->>Skaner: Skanuje kod QR
        Skaner->>Komputer: Zdeserializowany kod QR
        alt Autoryzacja nie przebiegła pomyślnie
            Komputer-->Ekran: Błędne dane logowania
        else Autoryzacja przebiegła pomyślnie
            Komputer-->Ekran: Wybierz czynność
        end
    end

    opt Wybrana czynność: "Odbieram paczki"
        Note over Komputer, Zaczepy: Skrytki rozpatrywane są po kolei dla każdej skrytki zawierającej paczkę do wysłania
        Note over Komputer, Zaczepy: KROK 2: Otwieranie skrytek z paczkami do wysłania 
        Komputer->>Sterownik: Zapytanie o otwarcie skrytki 
        Sterownik->>Zaczepy: Sygnał otwierający skrytkę
        alt Skrytka nie została otwarta
            Zaczepy->>Sterownik: Sygnał, że skrytka nie została otwarta
            Sterownik->>Komputer: Informacja o problemie ze skrytką
            Komputer->>Ekran: Informacja o problemie technicznym i prośba o ponownie próby
        else Skrytka została otwarta
            Zaczepy->>Sterownik: Sygnał, że skrytka została otwarta
            Sterownik->>Komputer: Informacja o otwarciu skrytki
            Komputer->>Ekran: Prośba o wyjęcie paczki i zamknięcie skrytki
        end
    end
  

    opt Skrytka została zamknięta
        Note over Komputer, Zaczepy: KROK 3: Wysłanie informacji do systemu centralnego
        Komputer->>Modul: Informacja o odbiorze paczki przez kuriera
        Modul->>SystemC: Informacja o odbiorze paczki przez kuriera zapakowana w protokół HTTPS
    end
```

---

## PU5 – Płatność przy odbiorze

### Diagram decyzyjny (PU5)

```mermaid
flowchart TD
    A([Start]) --> B{Paczka wymaga\npłatności?}
    B -- NIE --> Z1([Koniec – bez płatności])
    B -- TAK --> C{Terminal\ngotowy?}
    C -- NIE --> ERR1[/Terminal niedostępny\noperacja wstrzymana/]
    ERR1 --> Z2([Koniec])
    C -- TAK --> D[Wyświetlenie kwoty\ndo zapłaty]
    D --> E{Użytkownik\nanuluje?}
    E -- TAK --> ERR2[/Anulowanie\nbrak wydania paczki/]
    ERR2 --> Z3([Koniec])
    E -- NIE --> F{Metoda\npłatności?}
    F -- Karta --> G{Karta\nzatwierdzona?}
    F -- BLIK --> H{BLIK\nzatwierdzony?}
    G -- NIE --> ERR3[/Płatność odrzucona\nbrak wydania/]
    H -- NIE --> ERR3
    ERR3 --> Z4([Koniec])
    G -- TAK --> I[Transakcja potwierdzona]
    H -- TAK --> I
    I --> J[Sygnał otwarcia\nskrytki]
    J --> Z5([Koniec – paczka wydana])
```

---

### Diagram przepływu (PU5)

```mermaid
flowchart TD
    A([Start]) --> B[System wykrywa konieczność\npłatności przy odbiorze]
    B --> C[Wyświetlenie informacji\no kwocie do zapłaty PLN]
    C --> D[Aktywacja terminala\npłatniczego]
    D --> E{Terminal\ngotowy?}
    E -- NIE --> ERR1[Komunikat o błędzie\nterminala]
    ERR1 --> Z1([Koniec – błąd terminala])
    E -- TAK --> F[Użytkownik wybiera\nmetodę płatności]
    F --> G{Użytkownik\nanuluje płatność?}
    G -- TAK --> ERR2[Anulowanie transakcji\nbrak wydania paczki]
    ERR2 --> Z2([Koniec – anulowanie])
    G -- NIE --> H[Przetwarzanie\ntransakcji]
    H --> I{Transakcja\nzatwierdzona?}
    I -- NIE --> J{Kolejna\npróba?}
    J -- TAK --> F
    J -- NIE --> ERR3[Ostateczna odmowa\npłatności]
    ERR3 --> Z3([Koniec – odmowa])
    I -- TAK --> K[Potwierdzenie transakcji\nparagon opcjonalnie]
    K --> L[System wysyła sygnał\notwarcia skrytki]
    L --> M[Rejestracja płatności\nw systemie centralnym]
    M --> Z4([Koniec – paczka wydana])
```

### Diagram stanów (PU5)

```mermaid
stateDiagram-v2
    [*] --> WykryciePlatnosci : system sprawdza status paczki
 
    WykryciePlatnosci --> AktywacjaTerminala : płatność wymagana
    WykryciePlatnosci --> [*] : płatność niewymagana – kontynuacja odbioru
 
    AktywacjaTerminala --> TerminalGotowy : terminal odpowiada
    AktywacjaTerminala --> BladTerminala : brak odpowiedzi terminala
 
    BladTerminala --> [*] : operacja wstrzymana
 
    TerminalGotowy --> OczekiwanieNaPlatnosc : kwota wyświetlona
 
    OczekiwanieNaPlatnosc --> PrzetwarzanieTransakcji : użytkownik inicjuje płatność
    OczekiwanieNaPlatnosc --> AnulowaniePlatnosci : użytkownik anuluje
 
    AnulowaniePlatnosci --> [*] : paczka niewydana
 
    PrzetwarzanieTransakcji --> TransakcjaZatwierdzona : autoryzacja OK
    PrzetwarzanieTransakcji --> TransakcjaOdrzucona : odmowa banku / błąd
 
    TransakcjaOdrzucona --> OczekiwanieNaPlatnosc : użytkownik ponawia próbę
    TransakcjaOdrzucona --> [*] : ostateczna odmowa – paczka niewydana
 
    TransakcjaZatwierdzona --> RejestrowanieTransakcji : potwierdzenie do systemu centralnego
    RejestrowanieTransakcji --> SygnalOtwarciaSkrytki : log płatności zapisany
    SygnalOtwarciaSkrytki --> [*] : skrytka otwarta – paczka wydana
```
