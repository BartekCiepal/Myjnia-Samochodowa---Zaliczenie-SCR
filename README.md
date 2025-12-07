# 1. Tytuł modelu
**Model Systemu Myjni Samochodowej w języku AADL**

---

# 2. Dane studenta
* **Imię i Nazwisko:** Bartosz Ciepał
* **E-mail:** bartekciepal@student.agh.edu.pl

---

# 3. Opis modelowanego systemu

## 3.1. Opis ogólny
Projekt przedstawia model architektury systemu funkcjonowania myjni samochodowej, zrealizowany w środowisku **OSATE** przy użyciu języka **AADL** (Architecture Analysis & Design Language).

System został zaprojektowany jako układ hierarchiczny, składający się z systemu głównego (`Myjnia_Samochodowa`) oraz czterech wyspecjalizowanych podsystemów:
1.  **Panel Operatorski** – odpowiada za interakcję z użytkownikiem (wrzut monet, wybór programu).
2.  **Panel Informacyjny** – wyświetla czas mycia oraz wybrany tryb.
3.  **Zespół Zbiorników** – zarządza zasobami (woda, szampon, wosk, itp.) i ich dystrybucją.
4.  **Zespół Końcówek** – elementy wykonawcze (lanca, szczotka, kompresor).

Architektura sprzętowa opiera się na*trzech dedykowanych procesorach połączonych magistralami danych, co pozwala na zrównoleglenie zadań:
* **Procesor 1:** Obsługa płatności i wizualizacja danych na wyświetlaczach.
* **Procesor 2:** Logika sterowania trybami pracy i aktywacja odpowiednich pomp w zbiornikach.
* **Procesor 3:** Inteligentny rozdział mediów na odpowiednie końcówki (np. piana na lancę, powietrze na kompresor).

W projekcie przeprowadzono również **analizę harmonogramowania (Scheduling Analysis)**, definiując parametry czasowe wątków (okres, czas wykonania, deadline), co pozwoliło zweryfikować obciążenie procesorów.


## 3.2. Opis dla użytkownika
Z punktu widzenia użytkownika końcowego, system działa w następujący sposób:

1.  **Inicjalizacja:** Klient wrzuca monetę do otworu w Panelu Operatorskim. Sygnał jest przetwarzany i na Panelu Informacyjnym pojawia się dostępny czas mycia.
2.  **Wybór programu:** Użytkownik wybiera jeden z przycisków funkcyjnych:
    * Piana Aktywna
    * Mycie Szamponem
    * Spłukiwanie
    * Nabłyszczanie
    * Powłoka Ochronna
    * Osuszanie
3.  **Sterowanie:** System przetwarza wybór, wyświetla informację o trybie na ekranie oraz uruchamia odpowiednie pompy w Zespole Zbiorników.
4.  **Wykonanie:** Odpowiednie środki (np. woda z szamponem) są transportowane do właściwej końcówki (np. Myjki Ciśnieniowej), umożliwiając umycie pojazdu.

---

# 4. Spis komponentów AADL

W projekcie wykorzystano następujące komponenty z języka AADL do odwzorowania rzeczywistych elementów myjni:

### System (`system`)
Najwyższy poziom elementów myjni. Reprezentuja cała myjnie i jej główne elementy.
* `Myjnia_Samochodowa`: Główny system spinający wszystkie elementy.
* `Panel_Operatorski`: Podsystem umożliwiający wybór trybu pracy i ustawienie czasu pracy myjni.
* `Panel_Informacyjny`: Podsystem umożliwiający użytkownikowi odczytanie wybranego trybu pracy i przez ile czasu myjnia będzie działać.
* `Zespół_Zbiorników`: Podsystem decydujący na bazie wybranego trybu z jakiego zbiornika używać chemikalia do mycia.
* `Zespół_Końcówek`: Podsystem decydujący na bazie sygnału z jakiego zbiornika będą szły środki jaka końcówka powinna zostać uruchomiona.

### Procesor (`processor`)
Modeluje jednostki obliczeniowe wykonujące kod sterujący.
* `Procesor_Glowny_Monety`: Odpowiada za przeliczanie czasu i obsługę wyświetlaczy.
* `Procesor_Glowny_Tryby`: Odpowiada za logikę sterowania zaworami zbiorników.
* `Procesor_Rozdzielacz`: Steruje fizycznym przekierowaniem strumienia do odpowiedniej końcówki.

### Magistrala (`bus`)
Medium transmisyjne łączące komponenty.
* `Magistrala_Danych`: Modeluje połączenia kablowe przesyłające sygnały sterujące między panelami a procesorami. Zastosowano mechanizm `bus access` do wizualizacji połączeń fizycznych.

### Urządzenie (`device`)
Komponenty wykonawcze potrzebne do funkcjonowania myjni samochodwej. Można je podzielić na urządzenia wejścia i urządzenia wyjścia.
* **Wejścia:** Wszystkie urządzenia, które zadaje się działanie myjni takie jak: (przyciski trybu pracy lub otówr na monety).
* **Wyjścia/Elementy wykonawcze:** Wszystkie elementy, które będą wykonywać pracę na podstawie danych wejściowych i są to przykładowo (Wyświetlacz trybu pracy lub końcówki do mycia).

### Proces (`process`)
Logiczna jednostka oprogramowania zawierająca wątki. Odpowiada za dzielenie zasobów pamięci.
* `Proces_Panelu_Operatorskiego`: Zarządza sygnałami wejściowymi od użytkownika.
* `Proces_Zbiornikow`: Zarządza logiką przepływu mediów.

### Wątek (`thread`)
Najmniejsza jednostka wykonawcza, realizująca konkretne zadania w czasie rzeczywistym.
* `Watek_Sygnalu`: Uniwersalny wątek przesyłający informacje o stanie (0/1).
* **Właściwości czasowe:** Dla celów analizy zdefiniowano właściwości:
    * `Dispatch_Protocol => Periodic` (Uruchamianie cykliczne)
    * `Period => 50 ms` (Częstotliwość odświeżania 20Hz)
    * `Compute_Execution_Time => 1 ms .. 2 ms` (Czas zajętości procesora)
    * `Deadline => 50 ms` (Wymagany czas zakończenia zadania)

