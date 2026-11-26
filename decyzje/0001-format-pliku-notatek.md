# 0001: Format pliku notatek (z notatnika z Budowy 2)

## Kontekst

Poprzednia wersja programu (`notatnik` z Budowy 2) była desktopową aplikacją w C# (.NET 10, Avalonia UI) dla jednej osoby pracującej lokalnie na własnym komputerze. Potrzebowała prostego i trwałego sposobu zapisywania notatek między uruchomieniami bez stawiania bazy danych ani serwera. Oprócz samej treści notatki trzeba było zapamiętać także jej identyfikator (`Id` jako GUID), tytuł (`Title`), datę utworzenia (`CreatedAt`) oraz datę ostatniej modyfikacji (`UpdatedAt`), żeby móc sortować listę i pokazywać podgląd w interfejsie.

## Decyzja

W notatniku z Budowy 2 wszystkie notatki zapisywane były razem w jednym pliku `notatki.json` w formacie JSON z wcięciami (`WriteIndented = true`), w kodowaniu UTF-8 oraz z opcją `JavaScriptEncoder.UnsafeRelaxedJsonEscaping` (żeby polskie znaki nie zamieniały się w kody `\uXXXX`). Każda notatka jest osobnym obiektem w tablicy z polami `Id`, `Title`, `Content`, `CreatedAt` i `UpdatedAt`. Przy każdym zapisie lub usunięciu notatki program serializuje całą listę i nadpisuje plik `notatki.json` w całości, a przy pierwszym uruchomieniu (gdy pliku jeszcze nie ma) tworzy go automatycznie z notatką powitalną.

## Odrzucone

- **Osobny plik Markdown (`.md`) lub tekstowy (`.txt`) dla każdej notatki:** odrzucone w tamtej wersji, bo wymagałoby pilnowania unikalnych nazw plików przy podobnych tytułach, parsowania nagłówków lub metadanych (`CreatedAt`, `UpdatedAt`, `Id`) z treści pliku oraz zarządzania całym folderem plików zamiast jednego `notatki.json`.
- **Relacyjna baza danych (np. SQLite):** odrzucona, ponieważ dla kilkunastu czy kilkudziesięciu prostych notatek jednej osoby wprowadzałaby migracje schematu, dodatkowe zależności i utrudniała szybkie zajrzenie do danych bez osobnych narzędzi.

## Skutki

**Co ta decyzja daje:**
- Bardzo prosty odczyt i zapis z poziomu C# (jedna serializacja `System.Text.Json`, bez ręcznego parsowania tekstu).
- Cały zbiór notatek razem z metadanymi leży w jednym pliku `notatki.json`, który łatwo skopiować jako kopię zapasową.
- Dzięki `UnsafeRelaxedJsonEscaping` polskie litery zapisują się czytelnie wprost w UTF-8.

**Co ta decyzja utrudnia:**
- **Czytanie i ręczna edycja poza programem są niewygodne:** wielowierszowa treść notatki jest sklejona w jeden ciąg ze znakami `\r\n` i `\n`, cudzysłowy trzeba poprzedzać ukośnikiem, a znaki takie jak emoji i tak zamieniają się w kody typu `\uD83D\uDCDD` — przez to w zwykłym edytorze czyta się to znacznie gorzej niż czysty Markdown.
- **Ryzyko utraty wszystkich notatek naraz:** ponieważ każdy zapis nadpisuje cały plik `notatki.json`, przerwanie zapisu w połowie albo drobny błąd składni po ręcznej poprawce w pliku potrafi unieruchomić wczytanie całego zbioru, a nie tylko jednej notatki.
- **Słaba współpraca z historią zmian i większą liczbą zapisków:** dopisanie jednego zdania w jednej notatce powoduje przepisanie całego pliku od początku do końca, co zaśmieca historię zmian (np. w Gicie) i przy dużej liczbie notatek niepotrzebnie zapisuje na dysk wszystko od nowa.
- **Niejednoznaczne szukanie pliku przy starcie:** w tamtej wersji ścieżka do `notatki.json` była dobierana kolejno przez katalog roboczy i szukanie w górę od folderu uruchomienia aplikacji, przez co uruchomienie programu z innego miejsca mogło utworzyć lub otworzyć inną kopię pliku niż ta z wcześniejszej sesji.
