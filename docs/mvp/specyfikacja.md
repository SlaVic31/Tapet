# Tapet — pełna specyfikacja MVP

**Wersja:** 1.0, 23 października 2026. **Status:** gotowe do planowania realizacji. **Rodzaj pracy:** wyłącznie dokumentacja; dokument nie zawiera implementacji ani kodu aplikacji.

## 1. Cel i decyzje produktu

Tapet to osobista aplikacja łącząca porządek projektów i sprintów znany z Jira z szybkim przechwytywaniem i planowaniem zadań w stylu Things. Użytkownik ma widzieć, co wymaga uporządkowania, co chce zrobić dzisiaj i co zaplanował na później. AI pomaga tworzyć, edytować, podsumowywać i planować pracę, ale każda zmiana danych wymaga świadomej akceptacji.

MVP jest aplikacją webową desktop-first, uruchamianą lokalnie przez Docker Compose, dla jednego właściciela. Dane przechowuje lokalny PostgreSQL. Aplikacja działa bez internetu poza funkcjami AI. Nie jest to usługa publiczna ani aplikacja zespołowa. Specyfikacja funkcjonalna jest podstawą dla osobno przygotowywanego designu; wariant wizualny nie zmienia opisanych reguł.

### 1.1. Co musi wejść do MVP

- Przestrzenie Praca, Życie, Projekty poboczne; projekty opcjonalne dla zadania.
- Zadania, dokładnie jeden poziom podzadań, tagi, trzy stałe statusy i blokada z powodem.
- Skrzynka, Dzisiaj, Nadchodzące oraz jednoznaczny przełącznik przestrzeni i Wszystkie.
- Lista i kanban projektów oraz przestrzeni; podstawowe filtry i wyszukiwanie.
- Sprinty wieloprojektowe, maksymalnie jeden aktywny na przestrzeń.
- AI z jednym dostawcą przez backend: jawny kontekst, podsumowania oraz propozycje zmian, wybór operacji, walidacja, atomowe zatwierdzenie, konflikty, idempotencja i ograniczone cofanie.
- Trwałość danych, obsługa błędów, eksport JSON oraz sprawdzona procedura backupu i odtworzenia.

### 1.2. Co jest poza MVP

Wielu użytkowników, role i współdzielenie; publiczny hosting i dostęp LAN; aplikacje natywne; synchronizacja urządzeń; tryb offline przeglądarki; integracje Jira/Things/kalendarz/e-mail; powiadomienia push i przypomnienia; cykliczność; załączniki; komentarze; time tracking, estymaty, punkty, velocity i burndown; zależności między zadaniami; dowolne workflow; epiki i głębsze hierarchie; rozbudowane raporty; automatyczne planowanie w tle; autonomia AI i wykonywanie poleceń systemowych; wielu providerów i przełącznik modeli; RAG, embeddingi i baza wektorowa; pełny audyt każdej ręcznej edycji; dowolne cofanie historii; import JSON z UI; kosz i odzyskiwanie pojedynczych usunięć; pełna zgodność mobile i aplikacja PWA.

Brak tych funkcji nie blokuje odbioru MVP. AI oraz backup nie są dodatkami „na później”: bez bezpiecznego zatwierdzania AI i próby odtworzenia MVP pozostaje nieukończone.

## 2. Słownik i zakres

| Pojęcie | Znaczenie |
|---|---|
| Przestrzeń | Obszar życia zawierający projekty, zadania bez projektu i sprinty. Każde zadanie należy dokładnie do jednej przestrzeni. |
| Wszystkie | Widok agregujący aktywne przestrzenie; nie jest przestrzenią ani miejscem zapisu. |
| Projekt | Opcjonalny kontener zadań w jednej przestrzeni. |
| Zadanie główne | Zadanie bez rodzica, podstawowa jednostka list, kanbanu i liczenia postępu. |
| Podzadanie | Dziecko zadania głównego, bez możliwości posiadania dzieci. |
| Przetworzenie | Świadome uporządkowanie przechwyconego zadania i usunięcie go ze Skrzynki; nie jest zmianą statusu wykonania. |
| Data planu | Dzień, w którym użytkownik zamierza pracować nad zadaniem. |
| Deadline | Ostatni oczekiwany dzień ukończenia; nie jest planem wykonania. |
| Paczka AI | Niezmienna propozycja operacji wraz ze źródłowymi wersjami danych i stanem zatwierdzenia. |

### 2.1. Przestrzenie i Wszystkie

Pierwsze uruchomienie tworzy trzy wskazane przestrzenie. MVP pozwala zmienić ich nazwy, lecz nie dodawać, usuwać ani archiwizować przestrzeni. Nazwa po usunięciu białych znaków: 1–80 znaków, unikalna bez uwzględniania wielkości liter. Pozwala to uniknąć rozbudowy zarządzania strukturą na starcie.

Nagłówek każdego widoku pokazuje „Wszystkie przestrzenie” albo nazwę przestrzeni. Ten wybór obejmuje Skrzynkę, Dzisiaj, Nadchodzące, listę, kanban i wyszukiwanie. Projekt i sprint zawsze mają własną przestrzeń widoczną w nagłówku. Wejście do projektu lub sprintu ustawia jego przestrzeń. Zmiana przestrzeni z otwartego projektu wraca do listy przestrzeni zamiast pozostawiać niezgodny filtr.

Wszystkie grupuje zadania według przestrzeni i pokazuje etykietę przestrzeni na wyniku. Nie pokazuje jednego „aktywnego sprintu”, ponieważ aktywne sprinty mogą istnieć równolegle w różnych przestrzeniach. Przycisk dodawania we Wszystkie wymaga wyboru przestrzeni przed zapisem; nie zgaduje jej z ostatniej edycji. W konkretnej przestrzeni pole ma wartość domyślną tej przestrzeni.

## 3. Reguły domenowe

### 3.1. Zadania i daty

Zadanie ma tytuł 1–200 znaków po przycięciu, opis zwykłym tekstem do 10 000 znaków, status Do zrobienia/W toku/Gotowe, opcjonalny projekt, datę planu, deadline i tagi. Nowe zadanie jest Do zrobienia. Priorytet zadania ma wartości Brak/Niski/Średni/Wysoki, domyślnie Brak, i jest dostępny dla rodziców oraz dzieci. Priorytety P0/P1 w backlogu oznaczają kolejność implementacji.

Daty planu i deadline są datami kalendarzowymi bez godziny. Ustawienie jednej nie ustawia drugiej. Plan po deadline jest dozwolony, lecz UI ostrzega; to nie jest błąd walidacji. Deadline w przeszłości nie zmienia statusu. Zadanie Gotowe zachowuje daty i zapisuje czas ukończenia. Ponowne otwarcie czyści czas ukończenia.

Ustawienie strefy czasowej właściciela jest wymagane przy pierwszym uruchomieniu; onboarding proponuje strefę z przeglądarki/systemu i zapisuje ją po jawnym potwierdzeniu użytkownika (dla tego użytkownika Europe/London); przy braku rozpoznania proponuje Europe/Warsaw jako fallback. Strefę można zmienić w ustawieniach. „Dzisiaj” wylicza serwer według tej strefy. Znaczniki czasu są przechowywane w UTC. Widok odświeża przynależność po zmianie dnia, przy ponownym otwarciu karty i po zmianie strefy, bez zmiany samych dat zadania.

### 3.2. Skrzynka, Dzisiaj, Nadchodzące

| Widok | Dokładna przynależność i prezentacja |
|---|---|
| Skrzynka | Nieukończone zadania główne z flagą nieprzetworzone. Projekt, daty, tag lub sprint nie usuwają automatycznie tej flagi. Akcja „Przetwórz” usuwa ją bez obowiązku przypisania projektu/dat. Można ręcznie przywrócić do Skrzynki. Ukończenie usuwa flagę. |
| Dzisiaj | Nieukończone zadania z datą planu równą dziś lub wcześniejszą; dodatkowo zadania z deadline dziś lub wcześniej, nawet bez planu. Sekcje: zaległe deadline, deadline dziś, plan zaległy, plan dziś. Każdy rekord trafia do pierwszej pasującej sekcji, więc nie występuje dwukrotnie. |
| Nadchodzące | Nieukończone zadania mające przyszłą datę planu lub przyszły deadline. Oś według najwcześniejszej przyszłej daty; przy zadaniu widoczne obie daty i ich znaczenie. Jeden rekord w widoku. Może równocześnie należeć do Dzisiaj, np. plan dziś i deadline jutro; widoki nie są rozłącznym podziałem. |

Skrzynka jest kolejką przetwarzania, a nie synonimem „bez projektu” lub „bez daty”. Szybkie dodanie tworzy zadanie nieprzetworzone. Pełny formularz domyślnie również zachowuje tę flagę; ma wyraźne „Zapisz i przetwórz”. AI może jawnie proponować zmianę flagi.

Podzadania mają własne daty. W widokach datowych pasujące dziecko pojawia się pod kontekstowym rodzicem, nawet jeśli rodzic sam nie pasuje. Rodzic kontekstowy nie zwiększa licznika. Zliczanie paska widoku obejmuje unikalne rodziny zadań głównych; dodatkowa informacja może podać liczbę pasujących podzadań. Daty dziecka nie są kopiowane na rodzica. W Skrzynce dzieci nie są osobnymi pozycjami i nie mają własnej flagi przetworzenia.

### 3.3. Status, blokada i rodzic

- Blokada to niezależna flaga, a nie czwarty status. Wymaga przyczyny 1–500 znaków. Odblokowanie usuwa przyczynę. Blokada nie przechodzi na rodzica ani dzieci.
- Zablokowanego zadania nie można ukończyć. Trzeba je wcześniej odblokować, ewentualnie w tym samym jawnym formularzu lub paczce.
- Rodzica można oznaczyć Gotowe tylko wtedy, gdy wszystkie jego dzieci są Gotowe i sam rodzic nie jest zablokowany. Próba zamknięcia pokazuje listę przeszkód. MVP nie zamyka dzieci automatycznie.
- Ukończenie ostatniego dziecka nie zamyka rodzica automatycznie. Rodzic wymaga osobnego potwierdzenia.
- Ponowne otwarcie dziecka przy ukończonym rodzicu wymaga jawnej akcji „Otwórz również rodzica”; oba zapisy są atomowe, rodzic przechodzi do Do zrobienia. Bez zgody nic się nie zmienia. Dodanie nowego nieukończonego dziecka ma tę samą regułę.
- Przejście z Gotowe do W toku jest dozwolone dla rodzica; nie zmienia statusów dzieci. Kanban korzysta z tych samych walidacji co formularz i AI.

### 3.4. Dziedziczenie i przenoszenie

Podzadanie dziedziczy przestrzeń, projekt i sprint rodzica. Nie wolno ustawić tych wartości niezależnie. Może mieć własny status, blokadę, opis, daty i tagi. Brak konwersji zadania na podzadanie, odpinania dziecka i zmiany rodzica w MVP.

Zmiana projektu rodzica w tej samej przestrzeni zmienia projekt wszystkich dzieci w jednej transakcji; sprint zostaje. Przeniesienie rodzica do innej przestrzeni wymaga wyboru projektu docelowego lub „bez projektu” i potwierdzenia usunięcia rodziny z dotychczasowego sprintu; nie przypisuje nowego sprintu automatycznie. Przed zapisem UI pokazuje liczbę dzieci i skutki. Historia zakończonych sprintów zachowuje migawkę, więc transfer nie przepisuje przeszłego raportu.

Usunięcie rodzica wymaga potwierdzenia i usuwa jego dzieci oraz bieżące relacje; nie da się tego cofnąć przyciskiem cofania AI. Usunięcie dziecka wymaga potwierdzenia. Usuwanie nie zmienia historycznych migawek sprintów. AI nie może usuwać zadań, projektów, tagów ani sprintów w MVP.

### 3.5. Projekty i tagi

Projekt ma nazwę 1–120 znaków unikalną w przestrzeni, opcjonalny opis do 5 000 znaków oraz stan aktywny/zarchiwizowany. Archiwizacja wymaga braku nieukończonych zadań i członkostwa w planowanym/aktywnym sprincie. Można przywrócić projekt. W MVP nie ma usuwania projektu. Archiwum jest dostępne przez osobny filtr; zadania zarchiwizowanych projektów nie znikają z eksportu ani wyszukiwania z opcją archiwum. Nowych zadań nie można dodawać do archiwum.

Tagi są globalne dla właściciela, o nazwach 1–40 znaków unikalnych bez uwzględniania wielkości liter. Zadanie może mieć do 10 tagów. Zmiana nazwy aktualizuje prezentację wszystkich przypisań. Usunięcie tagu usuwa tylko przypisania, nie zadania. Tagi nie nadają uprawnień i nie rozszerzają zakresu AI.

### 3.6. Sprinty i liczenie

Sprint należy do jednej przestrzeni, może obejmować zadania z wielu jej projektów i zadania bez projektu. Ma nazwę 1–120 znaków, wymagany cel 1–1000 znaków po przycięciu, datę początku i końca (koniec nie wcześniejszy od początku), stan Planowany/Aktywny/Zakończony. Daty nie aktywują ani nie kończą go automatycznie. Sprintów planowanych może być wiele; maksymalnie jeden jest aktywny w przestrzeni, wymuszone również przy równoczesnych żądaniach.

Zadanie główne może należeć najwyżej do jednego planowanego lub aktywnego sprintu. Dodanie go do innego wymaga jawnej operacji przeniesienia pokazującej źródło i cel. Dzieci dziedziczą członkostwo. Gotowych zadań nie dodaje się do sprintu, ale zadania ukończone w trakcie pozostają do jego zamknięcia. Sprint nie zmienia dat planu ani deadline.

Zamknięcie sprintu pokazuje podsumowanie i wymaga wskazania dla nieukończonych zadań: wróć do backlogu (brak sprintu) albo przenieś do konkretnego planowanego/aktywnego sprintu tej samej przestrzeni. Jedna decyzja domyślna może dotyczyć wszystkich, z możliwością wyboru na pozycjach. Zamknięcie, przeniesienia i migawka są jedną transakcją. Brak ponownego otwierania zakończonego sprintu w MVP.

Lista i kanban pokazują główne karty oraz licznik dzieci Gotowe/wszystkie. Postęp projektu, przestrzeni i sprintu to liczba ukończonych zadań głównych / liczba zadań głównych w danym zbiorze. Podzadania nie zwiększają mianownika ani licznika. Pusty zbiór pokazuje „0 zadań”, bez procentu. Filtr pasujący wyłącznie do dziecka pokazuje rodzica jako kontekst, bez uznawania go za osobne dopasowanie. Zakończony sprint pokazuje postęp z migawki zamknięcia, nie dzisiejsze statusy.

## 4. Ekrany i stany interfejsu

Desktop jest głównym środowiskiem odbioru: 1280×800 i 1440×900. Przy 1024 px podstawowe operacje pozostają dostępne; pełny projekt telefonu jest poza MVP. Polskie etykiety, format dat jednoznaczny dla użytkownika. Kolor nie jest jedynym nośnikiem statusu.

| Ekran | Zawartość i operacje | Wymagane stany |
|---|---|---|
| Pierwsze uruchomienie | Nazwa właściciela, strefa czasowa, informacja o lokalnym dostępie; wybór opcjonalnej blokady hasłem. | Nowa instalacja, walidacja, zapis, zakończenie bez duplikowania trzech przestrzeni. |
| Powłoka aplikacji | Nawigacja Skrzynka/Dzisiaj/Nadchodzące/Projekty/Sprinty, przełącznik zakresu, szybkie dodanie, wyszukiwanie, AI, ustawienia. | Zakres aktywny, pusty wynik, błąd połączenia, zachowany adres i filtry po odświeżeniu. |
| Skrzynka | Lista rodzin nieprzetworzonych, edycja projektu/dat/tagów, akcja Przetwórz. | Pusta „Skrzynka uporządkowana”, zapis w toku, błąd, konflikt edycji. |
| Dzisiaj | Cztery sekcje z reguł przynależności, oznaczenia plan/deadline, ukończenie i przeplanowanie. | Brak planów, zaległość, przekroczony deadline, zmiana dnia, zadanie pasujące przez dziecko. |
| Nadchodzące | Sekcje dat z planem i deadline, domyślnie 30 dni; przejście dalej bez ukrycia dalszych zadań. | Pusto, dalszy horyzont, podwójna data jednego zadania, błąd ładowania kolejnej strony. |
| Przestrzeń / Wszystkie | Lista lub kanban; filtry projekt/bez projektu/status/priorytet/tag/blokada/sprint; widoczny zakres. | Brak danych vs brak wyników filtrów, grupy przestrzeni, zapis filtra w URL, odświeżenie. |
| Projekty | Katalog projektów w zakresie, utworzenie, aktywne/archiwum. | Pusty katalog, walidacja nazwy, blokada archiwizacji z powodem. |
| Projekt | Nagłówek z przestrzenią, opis, lista/kanban, filtry, postęp, wejście AI w zakresie projektu. | Aktywny, archiwalny tylko do odczytu z możliwością przywrócenia, pusty, błąd. |
| Szczegóły zadania | Panel lub strona z pełnymi polami, dziećmi, blokadą, sprintem, przetworzeniem; zapis i usunięcie. | Nowe/edycja, niezapisane zmiany, walidacja pól, zapis, konflikt, nieistniejący rekord, odmowa zamknięcia rodzica. |
| Kanban | Trzy stałe kolumny; główne karty z tagami, datami, blokadą i postępem dzieci. | Pusta kolumna, przeciąganie, odrzucenie ruchu i przywrócenie karty, alternatywna zmiana statusu klawiaturą. |
| Sprinty / sprint | Lista według stanów; formularz; backlog i członkostwo; start, koniec, historyczna migawka. | Brak sprintu, aktywny w innej przestrzeni, konflikt startu, zamknięcie z nieukończonymi, historia. |
| AI | Tryb Utwórz/Edytuj/Podsumuj/Zaplanuj; jawna przestrzeń lub projekt; prompt i informacja o przekazywaniu danych providerowi. | Brak klucza, gotowość, generowanie, błąd, timeout, odmowa, limit, wynik bez zmian, diff, konflikt, zapisano, cofnięto. |
| Przegląd paczki AI | Lista operacji z wartościami przed/po, checkboxy, zależności operacji, liczba wybranych, Zatwierdź/Odrzuć. | Niepełny wybór, walidacja, dane zmienione, zatwierdzanie, ponowny request, sukces z linkami, cofanie niedostępne z powodem. |
| Ustawienia i dane | Nazwy przestrzeni, tagi, strefa, tryb hasła, stan konfiguracji AI bez ujawnienia klucza, eksport, instrukcja backupu. | Błąd eksportu, pobieranie, AI wyłączone, zapis ustawień, informacja że odtworzenie jest operacją poza UI. |

Wspólne zachowanie: loader nie usuwa wpisanego tekstu; formularz zachowuje dane po błędzie. Błąd ma zrozumiałą przyczynę i akcję ponowienia. Puste wyniki filtrów oferują wyczyszczenie filtrów. Wyjście z niezapisanego formularza wymaga potwierdzenia. Akcje destrukcyjne wskazują nazwę i skutki. Brak sieci z backendem nie oznacza zapisu offline. Dostępne nazwy przycisków, widoczny fokus, obsługa klawiaturą i komunikat czytnika po błędzie są częścią odbioru.

## 5. Główne przepływy użytkownika

1. **Przechwyć i uporządkuj:** wybierz przestrzeń → szybkie dodanie tytułu → zapis do Skrzynki → otwórz szczegóły → opcjonalnie projekt/tag/plan/deadline → Zapisz i przetwórz → zadanie opuszcza Skrzynkę, pozostaje w przestrzeni bez względu na projekt.
2. **Zaplanuj dzień:** Dzisiaj w wybranym zakresie → przejrzyj zaległe deadline i plany → zmień datę planu albo ukończ zadanie → deadline pozostaje bez zmian → po odświeżeniu wynik zgodny z regułami dat.
3. **Wykonaj rodzinę:** dodaj dzieci → ustaw W toku → ukończ dzieci → osobno ukończ rodzica. Próba wcześniejszego zamknięcia pokazuje nieukończone lub zablokowane elementy. Ponowne otwarcie dziecka wymaga atomowego otwarcia rodzica.
4. **Przenieś zadanie:** szczegóły rodzica → wybór nowej przestrzeni → projekt docelowy lub brak → podgląd skutków dla dzieci i sprintu → zatwierdź → rodzina przeniesiona albo całość odrzucona przy konflikcie.
5. **Sprint:** wybierz przestrzeń → utwórz planowany sprint → dodaj główne zadania z co najmniej dwóch projektów lub bez projektu → rozpocznij → wykonuj w kanbanie → zakończ z jawnym wyborem losu pozostałych zadań → zobacz migawkę postępu.
6. **AI utwórz:** wybierz przestrzeń/projekt → opisz potrzebę → backend przygotowuje propozycję → wybierz rodziców i dzieci → przejrzyj daty i przetworzenie → zatwierdź → przejdź do utworzonych rekordów. Sama odpowiedź modelu niczego nie zapisuje do domeny.
7. **AI zaplanuj/edytuj:** wybierz zakres → wskaż intencję i opcjonalny sprint → propozycja z wartościami przed/po i uzasadnieniem → odznacz część zmian → walidacja wybranego zestawu → jeden atomowy zapis. Konflikt wymaga nowej propozycji na aktualnych danych.
8. **AI podsumuj:** wybierz zakres → otrzymaj tekst z odnośnikami do rzeczywistych zadań i zaznaczeniem, że jest to interpretacja AI → bez zmian danych; ewentualna prośba o działanie uruchamia osobny przepływ propozycji.
9. **Cofnij:** po paczce kliknij Cofnij ostatnią paczkę → backend sprawdza wersje i powiązania → całość cofnięta albo całość odrzucona ze wskazaniem konfliktu. Nie ma częściowego cofania.
10. **Zabezpiecz dane:** eksportuj JSON dla przenośnego odczytu → niezależnie wykonaj operacyjny backup → przećwicz odtworzenie na pustym środowisku → porównaj dane i uruchom aplikację.

## 6. AI: kontrakt bezpieczeństwa i stany

### 6.1. Dostawca i zakres

Jeden provider jest konfigurowany przy wdrożeniu, jeden model wskazany w konfiguracji; UI nie wybiera dostawców. Backend używa adaptera Laravel AI SDK. Klucz jest sekretem środowiska backendu, nie polem danych przeglądarki ani eksportu. Brak klucza wyłącza AI, ale nie blokuje reszty aplikacji. Wysłanie zadania do zewnętrznego modelu wymaga internetu; „lokalne dane” nie oznaczają lokalnego modelu.

Każda prośba wskazuje dokładnie jedną przestrzeń albo jeden projekt. We Wszystkie trzeba najpierw wybrać taki zakres. Nagłówek pokazuje jego nazwę oraz liczbę przekazywanych zadań. Projektowy zakres nie pozwala edytować innych projektów ani zadań bez projektu; przestrzenny dopuszcza projekty i zadania bez projektu tej przestrzeni. Zmiana zakresu unieważnia dotychczasowy wynik. Przenoszenie między przestrzeniami jest wyłącznie ręczne w MVP.

Provider otrzymuje tylko prompt i jawny zestaw pól potrzebnych w tym zakresie: identyfikatory, tytuły, opisy, statusy, priorytety, daty, blokady, tagi i strukturę rodzin/sprintów. Nie otrzymuje kluczy, ustawień konta ani danych innych przestrzeni. Treść zadania jest danymi, nie poleceniem nadpisującym reguły agenta. Model nie dostaje narzędzi zapisujących do bazy, sieci ani systemu plików. Maksymalny kontekst to 200 rekordów zadań, łącznie z dziećmi; przekroczenie wymaga zawężenia zakresu, bez cichego ucinania.

### 6.2. Dozwolone operacje

AI może proponować utworzenie projektów i zadań/podzadań, zmianę treści, statusu, priorytetu, blokady, dat, przetworzenia, istniejących tagów oraz projektu i sprintu w wybranej przestrzeni. Nowy projekt jest aktywny; nowe dziecko wymaga istniejącego lub tworzonego w tej paczce rodzica. Tworzenie nowych tagów pozostaje ręczne. W projekcie AI może tworzyć zadania tylko w tym projekcie, bez tworzenia innych projektów. AI nie tworzy/uruchamia/zamyka sprintów, nie zmienia ustawień ani przestrzeni, nie archiwizuje i nie usuwa. Plan sprintu to propozycja przypisania zadań do istniejącego planowanego/aktywnego sprintu, a nie samodzielny start sprintu.

Maksimum paczki: 50 operacji domenowych. Operacja dotyczy jednego rekordu i może zmienić kilka pól. Dziedziczone aktualizacje dzieci są jawnie pokazane jako skutki i objęte kontrolą wersji. Limit obejmuje dotykane rekordy, także dzieci. Przekroczenie kończy się czytelnym błędem, bez zapisania części paczki.

### 6.3. Generowanie i zatwierdzanie

1. Backend sprawdza zakres i limity, zapisuje żądanie i wersje kontekstu, zleca generowanie do kolejki database; nie trzyma transakcji bazodanowej podczas połączenia z providerem.
2. Odpowiedź musi dać się odwzorować na dozwolone, ustrukturyzowane operacje. Serwer sprawdza format, identyfikatory, zakres, limity pól, hierarchię i reguły domenowe. Nie ufa walidacji modelu. Niepoprawny wynik nie trafia jako zatwierdzalny diff.
3. UI prezentuje pola przed/po, nowe rekordy, zależności i skutki dziedziczenia. Użytkownik wybiera całe operacje, nie pojedyncze pola. Aby zmienić treść operacji, odrzuca propozycję i generuje nową albo edytuje zadanie ręcznie.
4. Operacja zależna od nowego rodzica/projektu wymaga zaznaczenia jego utworzenia. Odznaczenie zależności odznacza zależne operacje po wyjaśnieniu skutków. Pusty wybór nie może być zatwierdzony. Selekcja zamknięcia rodzica bez ukończenia dzieci jest odrzucana z wyjaśnieniem.
5. Zatwierdzenie przesyła identyfikator paczki, dokładny zestaw wybranych operacji, oczekiwane wersje i klucz idempotencji. Serwer ponownie sprawdza zakres i reguły oraz blokuje dotykane rekordy i istotne relacje. Dziecko dodane od generowania jest konfliktem przy zamykaniu lub przenoszeniu rodzica.
6. Wszystkie wybrane operacje, wynik paczki i zapis danych do cofania zatwierdzają się w jednej transakcji. Błąd dowolnego elementu oznacza zero zmian domenowych.
7. Paczka jest jednokrotnego użytku. Po akceptacji nie można zaakceptować pozostałych odznaczonych operacji; trzeba wygenerować nową propozycję.

Stan żądania AI: oczekuje → generuje → propozycja gotowa / podsumowanie gotowe / odmowa / błąd / timeout. Propozycja przechodzi do odrzucona, wygasła, konflikt albo zaakceptowana; zaakceptowana może zostać cofnięta. Propozycja wygasa po 30 minutach; samo wygaśnięcie niczego nie zmienia. Odrzucenie jest terminalne. Szczegóły terminalnej paczki są do odczytu, bez ponownego przycisku akceptacji.

### 6.4. Idempotencja, wersje i konflikty

Każdy mutowalny rekord domenowy ma rosnącą wersję. Każda zmiana zwiększa wersję, także zmiana dziecka zwiększa wersję strukturalną rodzica, gdy wpływa na operację zbiorową. Ręczny zapis również wymaga oczekiwanej wersji. Nie ma strategii „ostatni zapis wygrywa”. Przy konflikcie UI zachowuje lokalny tekst i pokazuje wersję bieżącą; ponowny zapis wymaga świadomego wprowadzenia zmian na aktualnych danych.

Ta sama paczka, ten sam klucz i ta sama selekcja zwracają ten sam zapisany rezultat bez ponownego wykonania. Ten sam klucz z inną treścią to konflikt. Nowy klucz nie pozwala zastosować już zaakceptowanej paczki. Wynik akceptacji jest przechowywany razem z paczką, bez krótkiego limitu ważności klucza. Równoległe dwa zatwierdzenia tej samej paczki dają jedną transakcję i ten sam rezultat lub informację „już zastosowano”.

Zmiana wersji dowolnego dotykanego rekordu, usunięcie lub zmiana istotnej relacji oznacza odrzucenie całej selekcji. Zmiana rekordu niedotykanego nie blokuje z definicji paczki, chyba że zmieniła warunek domenowy, np. stan sprintu lub listę dzieci. Serwer nie scala automatycznie nawet rozłącznych pól. Nowa propozycja musi używać aktualnego kontekstu i wymaga nowej akceptacji.

### 6.5. Błędy i ponowienia

| Sytuacja | Zachowanie |
|---|---|
| Brak klucza | Komunikat w panelu AI i odnośnik do instrukcji konfiguracji; żadne zlecenie do providera nie powstaje. |
| Provider niedostępny / błąd / limit | Żądanie kończy się błędem z możliwością jawnego ponowienia; domena bez zmian. Klucz i surowe błędy dostawcy nie trafiają do UI. |
| Timeout | Limit pojedynczego żądania 60 s, limit joba 90 s. Status timeout; spóźniona odpowiedź jest ignorowana. Brak automatycznych ponowień wywołania AI w MVP; ponowienie użytkownika tworzy nowe żądanie i może ponownie naliczyć koszt. |
| Odmowa providera | Czytelna informacja o odmowie, bez diffu i bez zmian; użytkownik może zmienić prompt. |
| Błędna struktura / niedozwolona operacja | Nie ma zatwierdzalnej paczki; informacja o nieprawidłowym wyniku i jawne ponowienie. |
| Zmienione dane / archiwizacja projektu | Konflikt podczas akceptacji, zero zmian; odświeżenie i nowa propozycja. |
| Zerwane połączenie podczas zatwierdzania | UI odpytuje stan paczki; ponawia z tym samym kluczem. Nie generuje nowego klucza, dopóki nie zna wyniku. |
| Ponowne zatwierdzenie | Zapisany wynik albo „już zastosowano”; nigdy duplikaty. |
| Użytkownik odrzuca | Paczka oznaczona jako odrzucona, domena bez zmian. |

### 6.6. Ograniczone cofanie

Dostępne jest cofnięcie wyłącznie ostatniej zaakceptowanej paczki modyfikującej dane, przez 24 godziny od akceptacji, jednokrotnie. Nie ma ponawiania cofnięcia jako „redo”, ani przejścia do starszych paczek po cofnięciu ostatniej. Odrzucenie propozycji lub podsumowanie nie zmienia tego wyboru. Akceptacja nowej paczki zastępuje poprzednią jako kandydat do cofania.

Backend zapisuje wartości przed/po, wersje po zapisie, utworzone identyfikatory i istotne powiązania. Cofanie jest możliwe tylko, jeśli wszystkie dotknięte rekordy nadal mają oczekiwane wersje i powiązania, a odwrócenie nie naruszy domeny. Nowe dziecko, zadanie w utworzonym projekcie, nowe użycie utworzonego rekordu albo ręczna zmiana któregokolwiek objętego rekordu blokują cofnięcie całej paczki. Edycja zupełnie niezwiązanego zadania nie blokuje.

Cofnięcie odtwarza stare wartości i usuwa wyłącznie rekordy utworzone tą paczką, jeśli nie mają nowych zależności. Wersje nadal rosną, nie wracają do starego numeru. Jest atomowe i idempotentne; powtórzone wywołanie zwraca stan „cofnięto”. Po konflikcie UI wyjaśnia przyczynę i pozwala poprawić dane ręcznie, bez wymuszania cofania. To ograniczona pomoc po AI, nie backup ani pełny dziennik zmian.

## 7. Model danych — logiczny, bez implementacji

Wszystkie identyfikatory są stabilne i nie zależą od tytułów. Relacje mają integralność w bazie; walidacja aplikacji nie zastępuje ograniczeń. Rekordy mają znaczniki utworzenia/zmiany, a mutowalne agregaty wersję.

| Encja | Główne dane i relacje |
|---|---|
| Właściciel / ustawienia | Jeden właściciel, nazwa, strefa, tryb blokady, opcjonalny hash hasła. Brak rejestracji kolejnych kont. |
| Przestrzeń | ID, nazwa, kolejność trzech stałych pozycji, wersja. |
| Projekt | ID, przestrzeń, nazwa, opis, stan aktywny/archiwalny, wersja. |
| Zadanie | ID, przestrzeń, opcjonalny projekt, opcjonalny rodzic, tytuł, opis, status, priorytet (Brak/Niski/Średni/Wysoki, domyślnie Brak; rodzice i dzieci), plan, deadline, blokada/powód, czas ukończenia, flaga nieprzetworzone dla głównego, wersja. |
| Tag / przypisanie | Globalny tag i unikalna relacja zadanie–tag. Zmiana przypisania zwiększa wersję zadania. |
| Sprint | ID, przestrzeń, nazwa, wymagany cel 1–1000 znaków po przycięciu, daty, stan, moment startu/zamknięcia, wersja. |
| Bieżące członkostwo sprintu | Sprint–zadanie główne; jeden planowany/aktywny sprint na zadanie. Dziedziczenie dzieci wyliczane z rodzica, bez niezależnej edycji. |
| Migawka zakończonego sprintu | Identyfikatory oraz tytuły, projekty, statusy głównych zadań w chwili zamknięcia, liczniki; zachowuje historię mimo późniejszych edycji i usunięć. |
| Żądanie AI | ID, tryb, zakres, prompt, migawka kontekstu/wersje, stan, provider/model użyty, czasy, bezpieczny kod błędu. |
| Paczka AI / operacje | Żądanie, stan, termin ważności, stabilne ID operacji, zależności, przed/po, oczekiwane wersje, wybrany zestaw, wynik akceptacji. |
| Zastosowanie i cofnięcie | Paczka, klucze idempotencji, skrót treści żądania, wynik, wartości do odwrócenia, wersje po zmianie, czas i wynik cofnięcia. |
| Zadania kolejki | Techniczne joby database; nie są zadaniami użytkownika i nie trafiają do eksportu domenowego. |

Przestrzeń i projekt dziecka są spójne z rodzicem; logicznie dziedziczone, a jeśli implementacja utrwala je dla wydajności, aktualizacja całej rodziny musi być transakcyjna. Klucz sprintu nie jest swobodnym polem dziecka. Reguły jednego aktywnego sprintu oraz jednokrotnego zastosowania paczki muszą wytrzymać równoczesne żądania, nie tylko walidację formularza.

Nie przechowujemy sekretu providera w bazie domenowej. Prompt i kontekst mogą zawierać prywatne treści: są lokalne, nie trafiają do zwykłych logów. Pełne konteksty i treść propozycji można usuwać operacyjnie po 30 dniach; trwały minimalny zapis stanu paczki, selekcji, klucza i wyniku chroni idempotencję. Dane potrzebne do cofania zachowuje się co najmniej do upływu jego okna. Backup ma własną retencję i może zawierać wcześniejsze treści.

## 8. Granice API i transakcji

MVP nie wymaga publicznego REST API ani osobnego SPA backendu. Inertia dostarcza ekrany i żądania do tego samego Laravel. Poniższe granice opisują odpowiedzialności, a nie gotowe endpointy czy kod.

| Obszar | Odczyt / komendy | Granica walidacji i zapisu |
|---|---|---|
| Nawigacja i planowanie | Zbiory Skrzynka/Dzisiaj/Nadchodzące, lista/kanban, wyszukiwanie | Jawny zakres; paginacja; identyczne reguły zapytań dla widoku i liczników. |
| Zadania | Utwórz, edytuj, przetwórz, zmień status/blokadę, usuń | Oczekiwana wersja; rodzina jako agregat dla zamknięcia, otwarcia dziecka, przeniesienia i usunięcia. |
| Projekty/tagi | Utwórz/edytuj, archiwizuj/przywróć projekt, usuń tag | Unikalność nazw, aktywny projekt, wpływ na przypisania i wersje. |
| Sprinty | Utwórz/edytuj planowany, przypisz/przenieś zadanie, start, zamknij | Blokada przestrzeni/sprintu, limit aktywnego, rodziny i migawka w jednej transakcji. |
| AI — generowanie | Zleć, pobierz stan i wynik, odrzuć | Sprawdzenie zakresu przed odczytem kontekstu; provider poza transakcją domenową. |
| AI — mutacja | Zatwierdź wybrane operacje, pobierz wynik, cofnij | Wersje, relacje, idempotencja, pełna walidacja i jeden commit bazy. |
| Dane/ustawienia | Eksport, zmiana strefy/nazw/blokady, odczyt stanu AI | Brak sekretów w odpowiedziach; spójny snapshot eksportu; odtworzenie poza UI. |

Listy mają strony po 50 rodzin zadań, a wyszukiwanie dopasowuje tytuł i opis oraz uwzględnia filtry. Brak zewnętrznej wyszukiwarki. Domyślny porządek list: nieukończone przed ukończonymi, następnie data planu rosnąco (brak na końcu), czas utworzenia i ID jako rozstrzygnięcie. Kanban używa tego porządku w kolumnie; przeciąganie zmienia tylko status, bez ręcznego rankingu. W Dzisiaj i Nadchodzące pierwszeństwo mają opisane reguły sekcji i dat.

Odpowiedzi rozróżniają brak rekordu, walidację pól, konflikt wersji, niepoprawny stan domeny, zbyt duży kontekst i błąd usługi. Każdy błąd ma stabilny kod oraz polski komunikat. Błędy nie zwracają kluczy, stack trace ani treści innych zakresów. CSRF, walidacja żądań i kontrola lokalnej sesji obowiązują także bez hasła. Obsługa równoległych kart jest wymaganiem mimo pojedynczego użytkownika.

## 9. Architektura i uruchomienie

### 9.1. Rekomendacja

**Modularny monolit Laravel 13 / PHP 8.5 / React 19 / TypeScript / Inertia 3 / Tailwind CSS 4 / shadcn/ui / PostgreSQL / Docker Compose.** Moduły odpowiedzialności: Organizacja, Zadania, Planowanie i Sprinty, AI, Dane i Operacje. Jedna aplikacja i jedna baza, bez mikroserwisów, Redis, brokera ani osobnego API gateway. Kolejka database, osobny proces workera korzystający z tego samego obrazu aplikacji.

Wersje są oparte na przeczytanych oficjalnych źródłach, nie prognozie: Laravel 13 obsługuje PHP 8.3–8.5 i udostępnia first-party AI SDK. [Laravel — Release Notes](https://laravel.com/framework/docs/releases). Oficjalny starter React wskazuje React 19, TypeScript, Inertia 3, shadcn/ui oraz Tailwind 4. [Laravel — Starter Kits](https://laravel.com/starter-kits). PHP 8.5 ma aktywne wsparcie do 31 grudnia 2027 i bezpieczeństwa do 31 grudnia 2029. [PHP — Supported Versions](https://www.php.net/supported-versions.php).

To rekomendacja projektu, nie twierdzenie, że gotowy starter zawiera całą domenę lub zabezpieczenia AI. Konkretne wersje poprawek, PostgreSQL, pakietu AI SDK i obrazy należy dobrać oraz przypiąć w lockfile/obrazach podczas implementacji po sprawdzeniu kompatybilności. Nie zakładamy tu nieweryfikowanych numerów. Wybór komercyjnego providera/modelu i jego kosztu pozostaje konfiguracją wdrożenia; MVP implementuje dokładnie jeden adapter, bez wielodostawcowej warstwy produktu.

### 9.2. Lokalność i konto

Port HTTP jest publikowany wyłącznie na 127.0.0.1; baza nie ma publicznego portu hosta. Brak tuneli, publicznej domeny, otwierania routera i ekspozycji do LAN. Przy konfiguracji dostępu poza loopback aplikacja ma odmówić startu w trybie MVP. Lokalny reverse proxy i PHP mogą być osobnymi kontenerami albo jednym uzgodnionym obrazem; nie zmienia to granic produktu.

Domyślnie działa jeden lokalny właściciel bez formularza logowania: aplikacja ufa kontu systemu operacyjnego i lokalnemu komputerowi. Nie jest to ochrona przed inną osobą mającą dostęp do tego komputera. Opcjonalna blokada hasłem jest częścią MVP: jedno hasło, sesja, wylogowanie i limit prób, bez rejestracji, e-maili, odzyskiwania pocztą i OAuth. Włączenie wymaga dwukrotnego podania hasła, wyłączenie aktualnego hasła. Reset odbywa się wyłącznie lokalną procedurą operacyjną przez właściciela hosta. Tryb hasła nie zezwala na publiczne wdrożenie.

### 9.3. Docker i operacje

Minimalne procesy: HTTP/aplikacja, PostgreSQL i worker database. Dane w nazwanym wolumenie; obraz aplikacji zawiera zbudowane assety. Restart kontenerów nie usuwa danych. Konfiguracja i sekrety są poza repozytorium i eksportem; logowanie bez promptów, opisów i sekretów, z identyfikatorem błędu. Osobne statusy zdrowia HTTP, bazy i workera; UI AI pokazuje błąd niedostępnego workera zamiast nieskończonego oczekiwania. Zatrzymanie workera przekraczające 90 s kończy osierocone żądanie jako timeout po wznowieniu/odczycie stanu, a nie automatycznym ponowieniem providera.

Compose nadaje się do uruchomienia na jednym hoście; dokumentacja Docker opisuje dostosowanie konfiguracji, zmiennych, restartów i odtwarzania kontenerów. Nasz wybór loopback i brak publicznej ekspozycji jest decyzją Tapet. [Docker — Use Compose in production](https://docs.docker.com/compose/how-tos/production/).

### 9.4. Eksport, backup i odtworzenie

**Eksport JSON:** ręczne pobranie z UI, spójny snapshot z wersją schematu, datą eksportu, strefą, przestrzeniami, projektami, tagami, zadaniami i relacjami rodzic–dziecko, sprintami, bieżącym członkostwem oraz migawkami zakończonych sprintów. Zawiera ID i wersje rekordów. Nie zawiera haseł, kluczy, sesji, kolejek, promptów ani logów AI. To format czytelnego wydobycia danych, nie pełny backup i nie obietnica importu w MVP.

**Backup operacyjny:** lokalny, spójny zrzut PostgreSQL obejmujący również paczki AI i ich stan; osobno chroniona kopia konfiguracji, klucza aplikacji i sekretów potrzebnych do odtworzenia. Nie kopiować surowego wolumenu uruchomionej bazy jako jedynej kopii. Backup wykonywany codziennie przez mechanizm hosta i zawsze przed aktualizacją; 7 kopii dziennych i 4 tygodniowe, co najmniej jedna na innym nośniku. Instrukcja zawiera wybór lokalizacji i sprawdzenie ostatniego sukcesu. Aplikacja nie buduje własnego panelu harmonogramu backupów.

**Odtworzenie:** zatrzymaj aplikację i worker → zachowaj obecną bazę/kopię bezpieczeństwa → uruchom pustą bazę zgodną z backupem i pasującą wersję aplikacji → odtwórz bazę i konfigurację → nie wykonuj zaległych jobów AI, oznacz oczekujące/generujące jako przerwane → sprawdź integralność i liczniki → uruchom HTTP/worker → ręcznie sprawdź zadanie z dzieckiem, sprint, eksport i status AI. Przed aktualizacją schematu zachowaj obraz/wersję potrzebną do odtworzenia; migracje nie zastępują planu powrotu.

Cel operacyjny MVP: utrata najwyżej 24 godzin pracy przy prawidłowym codziennym backupie (RPO), odtworzenie przez właściciela w ciągu 60 minut na tym samym sprawnym hoście po przygotowaniu obrazu i kopii (RTO). To cel odbioru do przećwiczenia, nie gwarancja przy utracie hosta lub nośnika. Backup zawiera prywatne dane i wymaga ograniczonych uprawnień; kopia na zewnętrznym nośniku powinna być szyfrowana. Eksport i backup nie wysyłają niczego automatycznie do chmury.

## 10. Backlog realizacyjny — 30 ticketów

**P0:** fundament lub warunek bezpiecznego wydania. **P1:** wymagana funkcja MVP budowana po fundamentach; nie oznacza opcjonalności. Wszystkie 30 ticketów jest potrzebne do deklaracji pełnego MVP. Zależności oznaczają warunki zakończenia, a nie zakaz wcześniejszego projektowania. AC dotyczą obserwowalnego zachowania i wymagają dowodu odbioru.

### TAP-01 — Lokalne środowisko Compose
**Priorytet:** P0. **Zależności:** brak.
**AC:** (1) Nowe środowisko startuje według jednej instrukcji i pokazuje stronę aplikacji. (2) Port HTTP jest dostępny przez 127.0.0.1 i niedostępny przez adres LAN; PostgreSQL nie publikuje portu hosta. (3) Restart zachowuje przykładowy rekord w wolumenie. (4) Worker i baza mają sprawdzalny stan zdrowia.

### TAP-02 — Model i integralność danych
**Priorytet:** P0. **Zależności:** TAP-01.
**AC:** (1) Można utrwalić i odczytać wszystkie encje z rozdziału 7. (2) Próba drugiego poziomu dzieci, niespójnego projektu/przestrzeni i duplikatu członkostwa jest odrzucona. (3) Rekordy mają wersje, a rollback transakcji nie pozostawia części relacji.

### TAP-03 — Pierwsze uruchomienie i pojedynczy właściciel
**Priorytet:** P0. **Zależności:** TAP-02.
**AC:** (1) Pierwsze uruchomienie tworzy dokładnie trzy przestrzenie o wymaganych nazwach i jednego właściciela. (2) Ponowne uruchomienie inicjalizacji nie duplikuje danych. (3) Użytkownik ustawia strefę; nie istnieje publiczna rejestracja kolejnego konta.

### TAP-04 — Powłoka i jawny zakres
**Priorytet:** P0. **Zależności:** TAP-03.
**AC:** (1) Każdy ekran pokazuje przestrzeń lub Wszystkie. (2) Przełączenie zakresu zmienia dane wszystkich widoków głównych i zachowuje go po odświeżeniu. (3) Dodanie we Wszystkie bez wybranej przestrzeni jest niemożliwe. (4) Wejście do projektu ustawia jego przestrzeń.

### TAP-05 — Projekty i archiwum
**Priorytet:** P1. **Zależności:** TAP-04.
**AC:** (1) Utworzenie/edycja respektują długości i unikalność nazw w przestrzeni. (2) Archiwizacja z nieukończonym zadaniem albo członkostwem sprintu jest odrzucona z powodem. (3) Projekt archiwalny można wyświetlić i przywrócić, lecz nie dodać do niego zadania.

### TAP-06 — Zadania i pełny formularz
**Priorytet:** P0. **Zależności:** TAP-04, TAP-05.
**AC:** (1) Zadanie zapisuje się z projektem i bez projektu. (2) Granice tytułu/opisu są walidowane na serwerze. (3) Edycja po odświeżeniu zachowuje wszystkie pola, w tym priorytet Brak/Niski/Średni/Wysoki (domyślnie Brak), dostępny dla rodziców i dzieci. (4) Usunięcie wymaga potwierdzenia, a anulowanie nie zmienia danych.

### TAP-07 — Szybkie dodanie i Skrzynka
**Priorytet:** P1. **Zależności:** TAP-06.
**AC:** (1) Szybkie dodanie tworzy nieprzetworzone zadanie główne. (2) Przypisanie projektu/dat nie usuwa go ze Skrzynki. (3) Przetwórz usuwa je ze Skrzynki bez wymogu projektu, a przywrócenie ponownie je pokazuje. (4) Ukończone zadanie nie pozostaje w Skrzynce.

### TAP-08 — Daty, strefa i niezależny deadline
**Priorytet:** P0. **Zależności:** TAP-06.
**AC:** (1) Zmiana planu nie zmienia deadline i odwrotnie. (2) Plan po deadline zapisuje się z ostrzeżeniem. (3) Granica północy i zmiana czasu letniego nie przesuwają dat kalendarzowych. (4) Zmiana strefy zmienia „dziś”, bez przepisywania dat zadań.

### TAP-09 — Dzisiaj i Nadchodzące
**Priorytet:** P1. **Zależności:** TAP-07, TAP-08.
**AC:** (1) Zestaw danych z każdą kombinacją dat daje wyniki zgodne z tabelą w 3.2. (2) Zadanie nie występuje dwukrotnie w jednym widoku. (3) Widok uwzględnia zakres i ukrywa Gotowe. (4) Dalsze niż 30 dni zadanie jest osiągalne przez rozszerzenie horyzontu.

### TAP-10 — Statusy i blokada
**Priorytet:** P0. **Zależności:** TAP-06.
**AC:** (1) Dostępne są dokładnie trzy statusy. (2) Blokada bez powodu jest odrzucona. (3) Zablokowanego zadania nie można ukończyć; odblokowanie i ukończenie w jednym jawnym zapisie jest dozwolone. (4) Ukończenie ustawia czas, ponowne otwarcie go czyści.

### TAP-11 — Podzadania i reguły rodzica
**Priorytet:** P0. **Zależności:** TAP-08, TAP-10.
**AC:** (1) Dziecko nie może mieć dziecka i dziedziczy projekt/przestrzeń. (2) Nie można zamknąć rodzica z nieukończonym dzieckiem. (3) Ostatnie ukończone dziecko nie zamyka rodzica. (4) Otwarcie dziecka przy Gotowe rodzicu wymaga zgody i atomowo otwiera rodzica. (5) Datowy widok pokazuje pasujące dziecko z kontekstem rodzica bez dodatkowego liczenia.

### TAP-12 — Przenoszenie i usuwanie rodzin
**Priorytet:** P0. **Zależności:** TAP-11, TAP-17.
**AC:** (1) Zmiana projektu rodzica zmienia projekt wszystkich dzieci. (2) Transfer przestrzeni pokazuje skutki i atomowo usuwa bieżący sprint całej rodziny. (3) Konflikt dowolnego dziecka powoduje zero zmian. (4) Potwierdzone usunięcie rodzica usuwa jego dzieci, lecz nie historyczną migawkę sprintu.

### TAP-13 — Tagi
**Priorytet:** P1. **Zależności:** TAP-06.
**AC:** (1) Nazwy unikalne bez wielkości liter i limit 10 tagów na zadanie są wymuszane serwerowo. (2) Zmiana nazwy jest widoczna na wszystkich przypisaniach. (3) Usunięcie tagu pozostawia zadania. (4) Przypisanie tagu zwiększa wersję zadania.

### TAP-14 — Lista, filtry, wyszukiwanie i liczniki
**Priorytet:** P1. **Zależności:** etap B — TAP-09, TAP-11, TAP-13; etap C — dodatkowo TAP-17 dla filtra sprintu i pełnego zakończenia TAP-14.
**AC:** (1) W etapie B filtry projekt/bez projektu/status/priorytet/tag/blokada łączą się w ramach jawnego zakresu; filtr sprintu nie jest warunkiem zakończenia etapu B. W etapie C, po TAP-17, filtr sprintu łączy się z pozostałymi filtrami i domyka TAP-14. (2) Tytuł i opis są przeszukiwane, a filtry pozostają w URL. (3) Paginacja po 50 rodzin nie pomija ani nie duplikuje wyników przy stabilnych danych. (4) Rodzic z dwoma dziećmi liczy się jako jedno zadanie główne; dopasowanie tylko dziecka pokazuje kontekst.

### TAP-15 — Kanban
**Priorytet:** P1. **Zależności:** TAP-10, TAP-11, TAP-14.
**AC:** (1) Trzy kolumny odpowiadają statusom, z głównymi kartami i postępem dzieci. (2) Przeciągnięcie zapisuje wyłącznie status. (3) Niedozwolony ruch wraca do poprzedniej kolumny i pokazuje przyczynę. (4) Tę samą zmianę można wykonać bez myszy.

### TAP-16 — Sprinty: definicja i start
**Priorytet:** P0. **Zależności:** TAP-04, TAP-10.
**AC:** (1) Można utworzyć kilka planowanych sprintów przestrzeni; każdy wymaga celu 1–1000 znaków po przycięciu, a brak celu lub przekroczenie limitu jest odrzucone. (2) Nieprawidłowy zakres dat jest odrzucony. (3) Dwa równoczesne starty w tej samej przestrzeni dają dokładnie jeden aktywny sprint. (4) Starty w dwóch różnych przestrzeniach są dozwolone.

### TAP-17 — Członkostwo wieloprojektowe i postęp sprintu
**Priorytet:** P0. **Zależności:** TAP-11, TAP-16.
**AC:** (1) Sprint przyjmuje zadania z dwóch projektów i bez projektu tej samej przestrzeni. (2) Odrzuca zadanie innej przestrzeni i niezależny przydział dziecka. (3) Przeniesienie między sprintami wymaga jawnego potwierdzenia. (4) Rodzic plus trzy dzieci daje jeden element postępu.

### TAP-18 — Zamknięcie i historia sprintu
**Priorytet:** P0. **Zależności:** TAP-17.
**AC:** (1) Zamknięcie wymaga decyzji o każdym nieukończonym zadaniu. (2) Zamknięcie i przeniesienia są atomowe. (3) Późniejsza edycja/usunięcie zadania nie zmienia migawki. (4) Zakończonego sprintu nie można ponownie uruchomić.

### TAP-19 — Wersje i konflikty ręcznych zapisów
**Priorytet:** P0. **Zależności:** TAP-02, TAP-06.
**AC:** (1) Dwie karty edytujące ten sam rekord nie nadpisują się po cichu. (2) Nieaktualna wersja daje konflikt i zachowuje wpisany tekst. (3) Zmiana dziecka unieważnia nieaktualną operację zbiorową rodzica. (4) Odczyt świeżych danych pozwala świadomie ponowić edycję.

### TAP-20 — AI: adapter, kolejka i konfiguracja
**Priorytet:** P0. **Zależności:** TAP-01, TAP-19.
**AC:** (1) Jeden provider działa wyłącznie z backendu przez adapter SDK i kolejkę database. (2) Klucz nie występuje w HTML, ruchu do przeglądarki, eksporcie ani logach. (3) Bez klucza pozostałe funkcje działają, a AI pokazuje instrukcję. (4) Job nie utrzymuje transakcji domenowej podczas oczekiwania na providera.

### TAP-21 — AI: kontekst i granice zakresu
**Priorytet:** P0. **Zależności:** TAP-14, TAP-17, TAP-20.
**AC:** (1) We Wszystkie AI wymaga projektu lub przestrzeni. (2) Zarejestrowany testowy request do providera nie zawiera zadań innego zakresu ani sekretów. (3) 201 rekordów wymaga zawężenia zamiast cichego ucięcia. (4) Zmiana zakresu unieważnia wynik; tekst zadania nie może nadać agentowi prawa zapisu.

### TAP-22 — AI: tworzenie i edycja propozycji
**Priorytet:** P0. **Zależności:** TAP-12, TAP-13, TAP-18, TAP-21.
**AC:** (1) AI proponuje nowe zadanie z dzieckiem oraz edycję istniejącego zadania bez zapisu do domeny. (2) Niedozwolony identyfikator, usuwanie, drugi poziom dzieci i ponad 50 dotykanych rekordów nie dają zatwierdzalnej propozycji. (3) Propozycja przechowuje stabilne ID operacji, zależności i wersje źródłowe.

### TAP-23 — AI: podsumowanie i planowanie
**Priorytet:** P1. **Zależności:** TAP-22.
**AC:** (1) Podsumowanie nie zmienia ani jednego rekordu domenowego. (2) Pokazuje wyłącznie odnośniki do istniejących zadań w zakresie. (3) Plan proponuje odrębne zmiany dat planu i przypisań do istniejącego sprintu, bez automatycznej zmiany deadline. (4) Nie uruchamia i nie zamyka sprintu.

### TAP-24 — AI: diff i wybór operacji
**Priorytet:** P0. **Zależności:** TAP-22.
**AC:** (1) Każda operacja pokazuje przed/po i zależne skutki. (2) Odznaczenie tworzonego rodzica odznacza jego nowe dzieci z wyjaśnieniem. (3) Pustego wyboru nie można wysłać. (4) Odrzucenie i wygaśnięcie po 30 minutach blokują akceptację bez zmian domenowych.

### TAP-25 — AI: atomowe zatwierdzenie i idempotencja
**Priorytet:** P0. **Zależności:** TAP-19, TAP-24.
**AC:** (1) Celowy błąd ostatniej z pięciu wybranych operacji pozostawia zero zmian. (2) Trzy powtórzenia tego samego żądania dają te same identyfikatory i tylko jeden zestaw rekordów. (3) Ten sam klucz z inną selekcją daje konflikt. (4) Inny klucz nie stosuje ponownie zaakceptowanej paczki. (5) Nieaktualna wersja, nowe dziecko lub zakończony sprint blokują całą selekcję.

### TAP-26 — AI: stany błędów i odzyskanie wyniku
**Priorytet:** P0. **Zależności:** TAP-20, TAP-25.
**AC:** (1) Brak klucza, błąd dostawcy, limit, odmowa, niepoprawna odpowiedź i timeout mają odrębny zrozumiały stan. (2) Spóźniona odpowiedź po timeout nie staje się propozycją. (3) Po utracie odpowiedzi akceptacji UI odczytuje wynik i ponawia z tym samym kluczem. (4) Restart workera nie wywołuje automatycznie ponownie providera dla przerwanego żądania.

### TAP-27 — AI: ograniczone cofnięcie
**Priorytet:** P0. **Zależności:** TAP-25, TAP-26.
**AC:** (1) Ostatnią paczkę można atomowo cofnąć przed 24 godzinami, a starszej nie można. (2) Zmiana objętego rekordu lub nowa zależność blokuje całe cofnięcie. (3) Zmiana niezwiązanego zadania nie blokuje. (4) Powtórzenie cofnięcia nie zmienia danych ponownie; po cofnięciu nie udostępnia się poprzedniej paczki ani redo.

### TAP-28 — Eksport i odtworzenie
**Priorytet:** P0. **Zależności:** TAP-18, TAP-27.
**AC:** (1) JSON ma wersję schematu i komplet danych z 9.4 bez sekretów i promptów. (2) Relacje w eksporcie nie wskazują brakujących rodziców/projektów. (3) Backup zostaje odtworzony w pustej bazie z porównaniem liczby przestrzeni/projektów/rodzin/dzieci/sprintów i próbą otwarcia przykładowych rekordów. (4) Próba spełnia RTO 60 min; dokumentacja określa codzienny backup, retencję i RPO. (5) Przerwane joby AI nie wykonują się po odtworzeniu.

### TAP-29 — Ustawienia i opcjonalna blokada hasłem
**Priorytet:** P1. **Zależności:** TAP-03, TAP-08, TAP-20.
**AC:** (1) Można zmienić nazwy trzech przestrzeni i strefę bez utraty relacji. (2) Włączenie hasła wymaga potwierdzenia hasła, wyłączenie aktualnego hasła. (3) Po wylogowaniu tryb hasła blokuje odczyt i mutacje, a próby logowania podlegają limitowi. (4) Instrukcja opisuje lokalny reset; nie ma rejestracji ani resetu przez e-mail. (5) Tryb bez hasła nadal odrzuca mutacje bez poprawnej ochrony CSRF.

### TAP-30 — Odbiór całości i użyteczność
**Priorytet:** P0. **Zależności:** TAP-01–TAP-29.
**AC:** (1) Wszystkie scenariusze z rozdziału 12 kończą się oczekiwanym wynikiem. (2) Przy 2 000 zadaniach głównych i 4 000 dzieciach p95 odczytu listy oraz ręcznego zapisu wynosi ≤1 s na opisanym w protokole lokalnym hoście, z wyłączeniem zimnego startu i AI, w 30 powtórzeniach każdej operacji. (3) Główne przepływy da się wykonać klawiaturą na 1280×800. (4) Nie ma nierozwiązanych błędów utraty danych, zakresu AI, częściowych transakcji ani podwójnego zapisu. (5) Protokół odbioru podaje środowisko i ograniczenia pomiaru.

## 11. Kolejność etapów

| Etap | Tickety | Warunek wyjścia |
|---|---|---|
| A. Fundament lokalny | TAP-01–04, kontrakt wersji TAP-19 (bez deklaracji zakończenia ticketu), projekt kontraktów TAP-20/21/25 | Uruchomienie lokalne, jeden właściciel, poprawny zakres, uzgodniony kontrakt wersji; kontrakt AI uzgodniony przed rozbudową UI. |
| B. Codzienne zadania | TAP-05–11, TAP-13–15 (TAP-14 bez filtra sprintu), pełny TAP-19 po TAP-06 | Przechwycenie → przetworzenie → plan → wykonanie z podzadaniami; spójne liczenie i widoki oraz pełny TAP-19; filtr sprintu nie jest warunkiem zakończenia B. |
| C. Sprinty i relacje | TAP-16–18, TAP-12, domknięcie TAP-14 po TAP-17 | Sprint wieloprojektowy i bezpieczne przenoszenie; migawki oraz konflikt równoległych kart; filtr sprintu TAP-14 działa razem z pozostałymi filtrami. |
| D. AI wczesny przekrój | TAP-20–22, TAP-24–25 | Jedno rzeczywiste wywołanie providera: zakres → propozycja → wybór → atomowy zapis; test konfliktu i ponowienia. Adapter i atrapę providera można budować już po A. |
| E. AI kompletne | TAP-23, TAP-26–27 | Podsumowanie i plan, pełne stany błędów, timeout i cofanie bez konfliktów. |
| F. Operacje i odbiór | TAP-28–30 | Eksport, odtworzenie, blokada opcjonalna, wydajność, dostępność i akceptacja. Procedurę backupu szkicować od A, wykonać na pełnym modelu. |

Nie odkładać integralności ani idempotencji do końcowego „utwardzania”. UI AI można projektować równolegle z B, lecz rzeczywista akceptacja paczek wymaga ukończonych reguł domenowych. Rzeczywisty provider służy do krótkiego testu integracyjnego; błędy, limity, odmowy i timeout sprawdza się powtarzalnym kontrolowanym adapterem testowym, aby odbiór nie zależał od losowej odpowiedzi modelu.

## 12. Scenariusze akceptacyjne

Każdy scenariusz ma odnotowany wynik, środowisko i w razie błędu identyfikator problemu. „Bez zmian” oznacza porównanie danych domenowych przed i po, nie tylko komunikat UI.

| ID | Warunki i działanie | Oczekiwany wynik |
|---|---|---|
| A01 | Nowa instalacja, dwa starty inicjalizacji. | Jeden właściciel, dokładnie trzy przestrzenie, brak duplikatów. |
| A02 | We Wszystkie dodaj zadanie bez wskazania przestrzeni, następnie wybierz Życie i brak projektu. | Pierwszy zapis zablokowany, drugi poprawny; zadanie w Życie i Skrzynce. |
| A03 | Zadaniu ze Skrzynki przypisz projekt i plan, następnie kliknij Przetwórz. | Po samej edycji nadal Skrzynka; po przetworzeniu znika tylko z niej. |
| A04 | Plan wczoraj, deadline jutro; zmień plan na dziś. | Deadline bez zmian; jeden wpis w Dzisiaj i jeden w Nadchodzące, właściwe etykiety. |
| A05 | Brak planu, deadline wczoraj; następnie ukończ. | Zaległe deadline w Dzisiaj; po ukończeniu znika z widoków datowych. |
| A06 | Rodzic bez dat, dziecko z planem dziś. | Rodzina widoczna w Dzisiaj z kontekstem rodzica, bez podwójnego licznika. |
| A07 | Rodzic ma dwa dzieci, jedno nieukończone; spróbuj zamknąć z formularza i kanbanu. | Obie próby odrzucone, karta wraca; po ukończeniu dzieci rodzic nadal wymaga osobnego zamknięcia. |
| A08 | Gotowy rodzic i dzieci; otwórz dziecko najpierw bez zgody, potem z potwierdzeniem. | Najpierw zero zmian, potem dziecko i rodzic Do zrobienia w jednej transakcji. |
| A09 | Zablokuj W toku z powodem, spróbuj ukończyć, następnie odblokuj i ukończ. | Blokada niezależna od kolumny, ukończenie wymaga odblokowania. |
| A10 | Przenieś rodzinę ze sprintu Praca do Życie bez projektu. | Podgląd skutków; wszystkie rekordy w Życie, brak bieżącego sprintu, brak niezgodnych dzieci. |
| A11 | Dwa równoczesne starty sprintu w Praca i osobny start w Życie. | W Praca dokładnie jeden sukces, w Życie osobny aktywny dozwolony. |
| A12 | Sprint zawiera dwa projekty i zadanie bez projektu, rodzic ma trzy dzieci. | Członkostwo poprawne; rodzina zwiększa postęp o jeden element, dzieci nie są osobnymi kartami. |
| A13 | Zamknij sprint z nieukończonymi, przenieś je do planowanego; potem zmień tytuł i status starego zadania. | Zamknięcie atomowe, migawka nie zmienia się po edycji. |
| A14 | Edytuj zadanie w dwóch kartach, zapisz po kolei. | Drugi zapis ma konflikt, nie nadpisuje pierwszego i zachowuje tekst do ręcznego porównania. |
| A15 | Wyłącz klucz AI i użyj listy, formularza, panelu AI. | Zadania działają; panel wskazuje brak konfiguracji, brak wywołania dostawcy. |
| A16 | AI w projekcie A, prompt żąda zmiany projektu B i zawiera polecenie obejścia reguł w opisie zadania. | Kontekst nie zawiera B; operacja spoza zakresu jest odrzucona; zero zmian. |
| A17 | AI proponuje projekt, rodzica i dzieci; odznacz utworzenie projektu. | Zależne operacje są odznaczone; nie można zastosować niepełnej zależności. |
| A18 | Zatwierdź trzy z pięciu niezależnych operacji. | Dokładnie trzy zapisane, dwie nigdy nie stosują się później z tej paczki. |
| A19 | Ostatnia operacja paczki łamie regułę zamknięcia rodzica. | Cała selekcja odrzucona, brak częściowych zmian i rekordów osieroconych. |
| A20 | Po wygenerowaniu zmień zadanie lub dodaj dziecko; zaakceptuj starą propozycję. | Konflikt i zero zmian; nowa propozycja wymaga nowej akceptacji. |
| A21 | Zatwierdź i zgub odpowiedź; ponów z tym samym kluczem, następnie innym kluczem. | Jeden zapis i ten sam rezultat; inny klucz nie omija stanu zaakceptowania. |
| A22 | Użyj ponownie klucza zatwierdzania z innym wyborem operacji. | Konflikt treści, bez nowych zmian. |
| A23 | Wymuś kolejno błąd, limit, timeout, spóźniony wynik, odmowę i niepoprawny format providera. | Właściwe stany, zero zmian, jawne ponowienie; spóźniony wynik ignorowany. |
| A24 | Odrzuć propozycję oraz spróbuj zaakceptować propozycję starszą niż 30 minut. | Obie niezatwierdzalne, brak zmian. |
| A25 | Poproś o podsumowanie i plan tygodnia w przestrzeni. | Podsumowanie bez mutacji; plan to diff dat planu/sprintu, deadline nie zmienia się bez osobnej intencji i operacji. |
| A26 | Cofnij ostatnią paczkę bez późniejszych zmian i ponów cofnięcie. | Pełne odwrócenie raz, rosnące wersje, drugi request zwraca ten sam stan. |
| A27 | Zmień objęte zadanie lub dodaj zależność po paczce; spróbuj cofnąć. | Całe cofnięcie zablokowane z powodem; brak częściowego usunięcia nowych rekordów. |
| A28 | Zmień niezwiązane zadanie; cofnij ostatnią paczkę. Następnie sprawdź starszą paczkę i granicę 24 h. | Niezwiązana zmiana nie blokuje; starsza i przeterminowana paczka niedostępne. |
| A29 | Eksportuj, wykonaj backup i odtwórz na pustej bazie z oczekującym jobem AI w kopii. | Eksport kompletny bez sekretów; dane i relacje odtworzone; job nie dzwoni do providera; RTO zmierzony. |
| A30 | Przejdź podstawowe flows klawiaturą, przełącz dzień/strefę, uruchom próbę danych 2 000+4 000. | Widoczny fokus, komunikaty, poprawne daty i zakres; spełniony limit wydajności z TAP-30. |
| A31 | Włącz hasło, wyloguj, spróbuj odczytu i zapisu; sprawdź tryb bez hasła z błędnym CSRF. | Hasło chroni odczyt i zapis; bez hasła błędny CSRF nadal blokuje mutacje. |

## 13. Definition of Done

MVP jest gotowe, gdy:

- Wszystkie 30 ticketów ma spełnione AC i dowód weryfikacji; wszystkie scenariusze A01–A31 przeszły albo problem został naprawiony i sprawdzony ponownie.
- Domena ma sprawdzoną integralność rodzic–dziecko, przynależność do przestrzeni, niezależne daty, brak podwójnego liczenia oraz odporność na równoczesne operacje.
- Każda ścieżka mutacji, w tym kanban i AI, korzysta z tych samych reguł; nie ma cichych nadpisań ani częściowych paczek.
- Testy automatyczne obejmują kluczowe reguły domenowe, zapytania widoków, atomowość, wersje, idempotencję, zakres AI, cofanie i błędy providera; przeglądarka potwierdza główne przepływy. Jeden test integracyjny z rzeczywistym wybranym providerem potwierdza adapter, bez uzależniania całej weryfikacji od sieci.
- Wszystkie ekrany mają puste stany, walidację, błędy i ładowanie zgodnie z rozdziałem 4; wprowadzony tekst nie znika po błędzie.
- Nie ma znanych błędów krytycznych utraty/wycieku danych, błędnego zakresu AI, podwójnego zapisu, blokady uruchomienia ani nieskutecznego odtworzenia. Drobne znane ograniczenia są jawnie opisane i nie przeczą AC.
- Docker działa lokalnie po czystym starcie i restarcie, wyłącznie na loopback; sekrety nie trafiają do przeglądarki, eksportu ani logów.
- Istnieją instrukcje pierwszego uruchomienia, konfiguracji jednego providera, startu/stopu workera, aktualizacji, backupu, odtworzenia, resetu opcjonalnego hasła i ograniczeń eksportu.
- Przeprowadzono odtworzenie i pomiar wydajności w opisanym środowisku; zapisano wynik RTO oraz włączono lub jawnie skonfigurowano codzienny backup hosta przed użyciem produkcyjnym.
- Polski interfejs i oddzielny projekt wizualny są zgodne z tą specyfikacją. Różnica designu wpływająca na domenę wymaga aktualizacji dokumentu przed realizacją.

## 14. Dalszy rozwój po MVP

Pierwszy krok po stabilnym użyciu to obserwacja codziennej pracy: czy Skrzynka jest przetwarzana, daty mają jasne znaczenie, a propozycje AI są często odrzucane lub ręcznie poprawiane. Dopiero na tej podstawie rozważyć cykliczność i przypomnienia, wygodniejszy mobile, import/eksport z walidacją, integrację kalendarza, bogatsze raporty lub kolejnych providerów. Publiczny hosting i wielu użytkowników wymagają osobnego projektu bezpieczeństwa, własności danych i uprawnień; nie są prostym przełączeniem konfiguracji tego MVP.
