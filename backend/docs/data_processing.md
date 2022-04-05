# Przetwarzanie danych
Zalogowani użytkownicy będą mogli wysłać pomiary do analizy, czyli klasyfikacji
swojego treningu.

## Propozycja
Niech proces przetwarzania zaczyna się po otrzymaniu zapytania od użytkownika.
Ponieważ w trakcie przetwarzania tego zapytania, inni użytkownicy mogą przesłać swoje pomiary,
te pomiary muszą być przechowywane w bazie danych i oznaczone jako nieprzetworzone. Klasyfikacja może być wymagająca sprzętowo, dlatego nie będzie przetwarzania równoległego.

Proces przetwarzania nie zakończy się dopóki w bazie danych nie będzie już więcej rekordów oznaczonych
jako "nieprzetworzone". Po skończeniu przetwarzania jednego pomiaru, z bazy danych wyciągnie kolejny jeszcze nieprzetworzony.

### Proces odbierania zapytania
```mermaid
flowchart
  a[Użytkownik kończy pomiar i wysyła zapytanie]
  b[Dodanie pomiaru do bazy danych jako nieprzetworzone]
  c[Wywołanie przetwarzania pomiarów]
  d[Odesłanie wiadomości o poprawnym odebraniu pomiaru]

  a --> b
  b --> c
  c --> d
```

### Proces przetwarzania pomiarów
```mermaid
flowchart
  a[/Wywołanie przetwarzania pomiarów/]
  b[Pobranie najstarszego nieprzetworzonego pomiaru]
  c[Klasyfikacja pomiaru]
  d[Zapisanie klasyfikacji do BD]
  e{{Czy są nieprzetworzne pomiary?}}
  f[/Koniec/]

  a --> e
  e -->|tak| b
  e ----->|nie| f
  b --> c
  c --> d
  d --> e

```

```mermaid
sequenceDiagram
  participant U as User
  participant HC as HTTPController
  participant ML as MLService
  participant m as MeasurementService

  U->>+HC: send measurement
  HC->>m: save unprocessed measurement
  par
    HC->>+ML: trigger processing
  and
    HC->>U: measurement saved
  end
  loop there are unprocessed measurements
    ML->>m: fetch oldest unprocessed measurement
    m-->>ML: return measurement
    ML->>ML: classify measurement
    ML->>-m: save classification
  end
```