# Tapet — brief designu MVP

Samodzielna, klikalna makieta osobistego organizera Jira + Things. Polski interfejs, desktop web z charakterem macOS. Makieta [tapet-mvp.html](tapet-mvp.html) zawiera cały CSS i niewielki skrypt demonstracyjny; bez zasobów zewnętrznych, zależności, backendu i trwałego zapisu. Dane są fikcyjne, a data scenariusza to 23 października 2026. Odświeżenie resetuje zmiany.

## Ekrany i flow

- **Skrzynka:** nieprzetworzone zadania główne z jawną akcją Przetwórz, niezależną od projektu i dat; szybkie dodawanie. Przy zakresie Wszystkie dodanie wymaga wyboru przestrzeni.
- **Dzisiaj:** otwarte zadania z planem lub deadline’em dzisiaj albo wcześniej, w sekcjach zaległy deadline / deadline dziś / zaległy plan / plan dziś, bez duplikatów. Deadline jest widoczny niezależnie od planowanego dnia.
- **Nadchodzące:** otwarte zadania z przyszłym planem lub deadline’em, grupowane według najwcześniejszej przyszłej daty. Jeden rekord w widoku, obie daty podpisane. Stały dzień scenariusza: 23.10.2026. Makieta pokazuje wszystkie przyszłe daty bez stronicowania.
- **Projekt:** lista oraz kanban korzystają z jednego zestawu danych. Status na karcie zmienia także listę. Nowe zadanie otwiera edytowalne szczegóły.
- **Szczegóły:** tytuł, opis zadania (textarea, np. kontekst i kryteria ukończenia), trzy statusy, priorytet, projekt, planowany dzień, deadline, tagi, blokada z wymaganym powodem, sprint oraz jeden poziom podzadań. Zapis jest wyraźną akcją demo.
- **Sprint:** cel, daty, postęp i zakres z dwóch projektów w jednej przestrzeni. Zakończenie wymaga potwierdzenia; niegotowe wracają do puli bez sprintu. Start kolejnego sprintu zawiera nazwę, cel, start i koniec z walidacją kolejności dat. Nowy zakres jest pusty; zadania dodaje się w szczegółach.
- **AI:** prompt → jawne stałe przykłady → wybór propozycji → podgląd przed/po → zatwierdzenie. Można wrócić lub zamknąć panel. Brak prawdziwego modelu; prompt nie steruje gotowym scenariuszem.
- **Ustawienia:** jeden dostawca AI z konfiguracji backendu, informacja o kluczach poza makietą, działający eksport aktualnych danych demo do JSON, reset.

## Reguły

Przestrzenie Praca / Życie / Projekty poboczne oraz jawny zakres Wszystkie. Projekt i sprint należą do jednej przestrzeni. Sprint może zawierać różne projekty, ale tylko jeden sprint jest aktywny w danej przestrzeni. Statusy: Do zrobienia / W toku / Gotowe. Blokada jest niezależną flagą, nigdy czwartym statusem. Podzadania mają jeden poziom; przestrzeń, projekt i sprint wynikają z rodzica. Makieta nie demonstruje dowolnego tworzenia projektów ani edycji już aktywnego sprintu; te funkcje nie mają pozornych przycisków.

## Design tokens

| Token | Wartość / rola |
|---|---|
| Baza | `#f7f6f2`, spokojne kremowe tło |
| Powierzchnia | `#fffefa`, panele i karty |
| Sidebar | `#eeede7` |
| Tekst | `#30332e`, grafit |
| Pomocniczy | `#73776c` |
| Akcent | `#626f43`, oliwka; jasny wariant `#e9eddc` |
| Separator | `#e2e3d9` |
| Ostrzeżenie | `#a35541`, tekst deadline / blokada |
| Typografia | system macOS / Segoe UI, baza 14 px, H1 30 px, metadane 11–12 px |
| Geometria | promienie 8–12 px, odstępy 8 / 12 / 16 / 24 / 32 px |

## Komponenty i stany

Sidebar, selektor przestrzeni, breadcrumb, przełącznik lista/kanban, wiersz zadania, checkbox ukończenia, karta kanban, badge, formularz szczegółów, szuflada modalna, panel AI, propozycja z checkboxem, potwierdzenie, toast i pusty widok.

Na ekranach roboczych selektor „Stany demonstracyjne” pokazuje empty/loading/error. Są to celowo statyczne przykłady, nie prawdziwe żądania. Empty wyjaśnia brak zadań w zakresie, loading ma komunikat statusu, error ma odzyskanie przez „Spróbuj ponownie”. Panel AI ma osobny przykład konfliktu, bez zastosowania zmian, z ponownym przygotowaniem propozycji. Dodatkowo akceptacja kontroluje, czy dane propozycji nie zmieniły się od ich utworzenia.

## Dostępność i mobile

Widoczny focus, semantyczne formularze i etykiety, nazwy ikon akcji, aria-pressed dla przełączników, status toastu, Escape zamykający szufladę, ograniczenie tabulacji do otwartego panelu i powrót focusu. Przy 390 px sidebar otwiera się z menu, treść pozostaje jednokolumnowa, kanban układa kolumny pionowo, a szuflada zajmuje szerokość ekranu. Brak `100vh`, brak zasobów zdalnych. Układ bazowy ma wysokość wynikającą z treści.

## Zakres walidacji

Kontrola wizualna wykonana na zrzutach PNG ramek `srcdoc` z aktualną makietą. Widoczne pomiary `innerWidth` i `iframe.clientWidth` potwierdzają dokładnie 1440 i 390 CSS px. To kontrola szerokości responsywnych, nie emulacja urządzenia. Obejrzano Dzisiaj, kanban, szczegóły zadania, podgląd AI i menu mobile. Finalne porównania: [tapet-final-menu.png](screenshots/tapet-final-menu.png), [tapet-final-kanban.png](screenshots/tapet-final-kanban.png), [tapet-final-drawer.png](screenshots/tapet-final-drawer.png), [tapet-final-ai.png](screenshots/tapet-final-ai.png).

Jedyna korekta po review: menu mobile otwiera się jako przewijany panel od górnej krawędzi i ma widoczny przycisk „Zamknij nawigację”. Focus + Enter potwierdzają zamknięcie; pointer w iframe nie dał wiarygodnego potwierdzenia (skalowany: błąd trafienia; nieskalowany: dispatched bez potwierdzonej zmiany). Nie deklarujemy pełnego testu pointera. Podczas kontroli nie zgłoszono błędów strony. Dodatkowa kontrola renderowania zwróciła `RENDER_CHECK_FAILED`; powyższe PNG pochodzą ze zrzutów przeglądarki. Kontrola nie wymagała instalacji ani uruchomienia serwera lub implementacji aplikacji. Brak logowania, synchronizacji, trwałego zapisu i prawdziwej integracji AI.

## Doprecyzowanie spójności demo

Ukończenie zablokowanego zadania lub rodzica z otwartymi podzadaniami jest odrzucane z wyjaśnieniem na liście, kanbanie i w formularzu. Dzieci nie zamykają się automatycznie. Przy dodaniu lub otwarciu dziecka gotowego rodzica wymagana jest zgoda „Otwórz również rodzica”; rodzic i dzieci zmieniają się razem przy zapisie; zapis odrzuconego formularza nie zmienia danych. Zakończenie sprintu wymaga potwierdzenia powrotu niezrobionych rodzin do backlogu. Daty i projekty pozostają bez zmian, migawka wyniku jest niezależna od późniejszych edycji. Przenoszenie do innego planowanego sprintu jest opisane jako poza demonstracją. Nowy sprint nie otwiera zakończonego. Priorytet pozostaje zgodnie z uzgodnionym zakresem MVP.

Demonstracja AI obejmuje jedną przykładową paczkę w jawnej przestrzeni Praca, pełne operacje wybierane checkboxami, podgląd, akceptację, anulowanie i konflikt. Brak klucza/błąd/timeout/odmowa, zależności operacji, ponowne zatwierdzenie i ograniczone cofanie pozostają w specyfikacji produktu; nie są symulowane przyciskami tej makiety. Stack implementacyjny jest poza zakresem designu HTML.

## Niezamockowane obszary

Onboarding, katalog i zarządzanie projektami oraz zarządzanie tagami są opisane w specyfikacji, ale nie są zamockowane i nie mają osobnych ekranów w tej makiecie. Szczegóły pozwalają wpisać tagi zadania, ale nie zarządzać ich katalogiem. Zmiana przestrzeni w widoku projektu wraca do Dzisiaj; otwarcie przykładowego projektu z zakresu Wszystkie jawnie ustawia jego przestrzeń Praca. „Zapisz i przetwórz” usuwa flagę Skrzynki; zwykły zapis projektu i dat jej nie zmienia.
