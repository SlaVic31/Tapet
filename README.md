# Tapet

Osobisty organizer łączący projekty i sprinty w stylu Jira z szybkim przechwytywaniem oraz planowaniem zadań w stylu Things. AI ma pomagać w planowaniu i edycji, a zmiany wymagają akceptacji użytkownika.

Status: etap koncepcji i dokumentacji MVP. Repozytorium zawiera specyfikację oraz statyczną, klikalną makietę; aplikacja nie jest jeszcze zaimplementowana.

## Dokumentacja i makieta

- [Specyfikacja MVP — 30 ticketów i 31 scenariuszy akceptacyjnych](docs/mvp/specyfikacja.md)
- [Brief designu i zakres demonstracji](docs/design/brief.md)
- [Samodzielna makieta HTML](docs/design/tapet-mvp.html)
- Finalne zrzuty: [menu](docs/design/screenshots/tapet-final-menu.png), [kanban](docs/design/screenshots/tapet-final-kanban.png), [szczegóły zadania](docs/design/screenshots/tapet-final-drawer.png), [AI](docs/design/screenshots/tapet-final-ai.png).

Otwórz `docs/design/tapet-mvp.html` bezpośrednio w przeglądarce, np. dwuklikiem. Makieta jest samodzielna: nie wymaga instalacji zależności, budowania ani serwera.

## Ograniczenia demo

Makieta korzysta z fikcyjnych danych i nie ma backendu, trwałego zapisu ani prawdziwego AI. Odświeżenie resetuje zmiany. Onboarding, katalog projektów i zarządzanie tagami opisano w specyfikacji, ale nie zamockowano. Brief opisuje potwierdzone szerokości 1440/390 CSS px oraz ograniczenie weryfikacji pointera.

## Docelowy stack

Modularny monolit Laravel / PHP, React z TypeScript i Inertia, Tailwind CSS i shadcn/ui, PostgreSQL oraz Docker Compose. AI przez backend z jednym dostawcą i adapterem Laravel AI SDK; kolejka database. Ustalone wersje i źródła znajdują się w specyfikacji; ten zapis nie tworzy konfiguracji ani implementacji.
