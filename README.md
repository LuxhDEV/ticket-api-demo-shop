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

## Audit gegen Doku und echte Responses (16.10.2026)
- **Preisgruppen-Typen korrigiert:** 0 = General (gilt überall), 1 = Platzreservierung, 2 = Ticketing, 3 = Eintritt, 4 = Verkauf & Ausstellungen, 5 = Verleih, 6 = Wellness. Die frühere Angabe (0 = Platzreservierung … 5 = Wellness) war um eins verschoben.
- **Reihenfolge:** ListPaymentMethods kommt vor ValidateBasket. Die Zahlungen stehen in `basket.payments[]`.
- **Kunde:** ValidateBasket braucht eine gültige CustomerId, entweder die eingeloggte Person oder den Web-Systemkunden (Gastkauf, siehe unten).
- **Felder direkt am Produkt**, jetzt in der Oberfläche angezeigt und vorab geprüft:
  - Artikel: `allowPriceChangeWebshop`, `maximumSalesQuantity` (0 = unbegrenzt), `isDigitalWalletItem`, `isSubscriptionItem`, `blocked`/`unAvailable`
  - Ausstellung: `askNames`, `minimumAge`, `maxTicketsPerWebSale`, `controlType`/`maxVisitorsPerGroup`, `enablePresale`, `blocked`
  - Preis: `minSaleQuantity`/`maxSaleQuantity`, `askNames`, `hasAllowedSubscriptions`, `mandatoryBuyMinQtyWeb`
  - Zeitfenster: `finalSubscriptionDate`, `blocked`/`isExcluded`
  - Abo: `isEticket`, `familyItem`, `maxPersons`, `personsMinimumAge`, `periodDuration`
- **E-Ticket-URL** endet mit `/`. Bei Aktivitäten lautet sie `…/ActivitySubscriptions/{ShopId}/{Sprache}/{SubscriptionId}`.
- **ExtendLockPeriod:** Danach wird das neue LockTicket aus der Response weiterverwendet.

## Gastkauf (16.10.2026)
- Korrektur: Ein Kauf ohne Konto geht doch, die frühere Aussage „anonym nicht möglich“ war falsch. Als CustomerId dient die ID des Web-Systemkunden (Feld „Gast-Kunden-ID“). Die Käuferdaten kommen in `basket.anonymousPerson` (Typ `Anonymous`: Name, FirstName, Street1, Street2, Number, Box, ZipCode, Home, Country, Email, Telephone, Newsletter).
- In der Oberfläche: Schalter „als Gast kaufen“ im Checkout und ein Formular für die Käuferdaten (Name und E-Mail sind Pflicht).
- Nur mit Kundenkonto (Badge, im Gastmodus gesperrt): Abos und Mehrfachkarten (`isSubscriptionItem`, alle Artikel aus FindSubscriptionArticles), Wallet-Artikel (`isDigitalWalletItem`) und Aktivitäten (`ActivityReservation`). Die API lehnt sie im Gastmodus nicht ab, darum sperrt der Shop sie.

## Dokument-URLs nach dem Checkout (16.10.2026)
Welcher Endpunkt von `WebShopDocumentService.svc` passt, ergibt sich aus `salesItems[].$type` und den Feldern der Position. Ein Feld „documentType“ gibt es nicht. Schritt 6 baut die Links automatisch und zeigt die Zuordnungstabelle.
- ExpositionSalesItem: `ETickets/…/{id}/`, `ETicketByOrganisedVisit/…/{organisedVisitId}/`, `OrganisedVisits/…/{organisedVisitId}`
- ArticleSalesItem: mit `barcodes`/`isMembership` → `ETickets/…/{id}/`, sonst `Coupons/…/{id}` bzw. `GiftCertificates/…/{id}`; mit `directDebitProcessing` → `DirectDebitContractSetUp`/`DirectDebitContractPayment`
- ActivityReservationSalesItem: `ActivityOrder/{orderId}`, `ActivitySubscriptions/{id}`, `ETicketByActivityEntry/{entryId}/`. Weitere Typen (Kultur, Fitness, Verleih, Platzreservierung, Abo-Verlängerung, Dossier) laut `/help`.
- Ergebnis des Checkouts: `DigitalReceipt/{ShopId}/{salesSeriesId}/` (Kassenbeleg) und `Invoice/…/{invoiceId}`
- Real als PDF bestätigt: ETickets (Ausstellung, 10er Karte), ETicketByOrganisedVisit, OrganisedVisits (Vorlage ohne Daten), Coupons (Gutschein (Druck)), DigitalReceipt. 0 Byte (keine Vorlage): GiftCertificates, DirectDebit.
- Slash am Ende nur bei ETickets, ETicketBy… und DigitalReceipt (sonst 307). Bei den übrigen führt der Slash zu 404.

## Retest (16.10.2026, 19:40)
Behoben: Kundenwechsel (Login, Logout, Gast) verwirft den berechneten Warenkorb. Technische Fehler (`succes:false`, HTTP ≠ 200) erscheinen als roter Hinweis. ExtendLockPeriod meldet `failed[]` und weist darauf hin, dass „succeeded“ auch bei freigegebenen Locks kommt. Gastkauf prüft die Gast-Kunden-ID vor Validate.
