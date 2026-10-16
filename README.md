# Ticket-API Demo-Webshop

Ein Demo- und Schulungs-Webshop für eine Ticket-/Kassen-JSON-API. Er besteht aus einer einzigen Datei (`index.html`) und läuft ohne Backend im Browser.
Für jeden Schritt zeigt er, wann der Call gebraucht wird, den genauen Request, die Response und welche Werte in den nächsten Schritt fließen.

## Start
1. `index.html` im Browser öffnen (oder über GitHub Pages).
2. Oben eintragen: API-Basis-URL (z. B. `https://api.example.com/json`), ShopId, API-Passwort und den Contract-Namespace für die `$type`-Angaben.
3. „Neue SessionId“ klicken und den gewünschten Tab wählen.

Die API muss CORS erlauben (`Access-Control-Allow-Origin`). Calls werden als `POST <Basis-URL>/<Modul>/<Methode>/` gesendet. Datumswerte gehen als lokale Zeit ohne „Z“ raus.

## Tabs (17 + State)
0 Kaufablauf (Diagramm) · 1 Artikelkauf · 2 Warenkorb-Profi (Lock/Relock/Unlock, Teilzahlung) · 3 Variabler Gutschein · 4 Abos/Mehrfachkarten · 5 Ausstellungen/Zeitfenster · 6 Storno · 7 Kunden · 8 Registrierung · 9 Familie · 10 Kundenpreise · 11 Bestellhistorie & E-Tickets · 12 Produktfilter · 13 Reservierung (Stammdaten) · 14 Fehler & Recovery · 15 Sprache · 16 Spielwiese · State

Jeder Request lässt sich als curl oder Postman exportieren, das Passwort steht dort als `{{PASSWORD}}`.

## Hinweise
- CheckoutBasket und CreditSale buchen echt. Vor beiden Calls kommt eine Bestätigungsabfrage.
- Pro Warenkorb bzw. Kauf eine neue SessionId verwenden.
- Eingaben werden geprüft: GUID-Format von ShopId/SessionId, Mengen als ganze Zahl ≥ 0 (0 = Position entfernen), Gutscheinbetrag > 0. Ein Doppelklick-Schutz verhindert doppelte Calls.
