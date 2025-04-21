# StreetBall App - Opis Projektu

## Ogólny opis projektu

StreetBall App to aplikacja internetowa do organizowania i uczestniczenia w ulicznych grach sportowych w Polsce, z początkowym naciskiem na streetball i koszykówkę, z możliwością rozszerzenia na inne sporty (piłka nożna, siatkówka itp.). Głównym celem aplikacji jest uproszczenie procesu wyszukiwania gier i boisk, a także organizowania własnych gier.

## Podstawowe wymagania funkcjonalne dla bazy danych

### Przechowywanie informacji o użytkownikach:
- Podstawowe dane profilu (nazwa użytkownika, e-mail, hasło)
- Ustawienia powiadomień
- Linki do utworzonych i dołączonych gier

### Przechowywanie informacji o obiektach sportowych:
- Współrzędne geograficzne do wyświetlenia na mapie
- Szczegółowe informacje o boisku (rodzaj nawierzchni, oświetlenie itp.)
- Obsługiwane dyscypliny sportowe
- Oceny i recenzje

### Przechowywanie informacji o grze:
- Czas i czas trwania
- Lokalizacja (boisko)
- Format gry (3x3, 5x5 itp.)
- Lista uczestników
- Status gry (zaplanowana, w toku, zakończona, anulowana)

### Przechowywanie powiadomień:
- Śledzenie przeczytanych/nieprzeczytanych powiadomień
- Komunikacja z użytkownikami i grami
- Typ powiadomienia (przypomnienie, dołączenie gracza itp.)

### Zapewnienie elastycznej struktury danych:
- Możliwość łatwego dodawania nowych dyscyplin sportowych
- Skalowanie w celu zwiększenia bazy użytkowników
