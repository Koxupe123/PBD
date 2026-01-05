<style>
  body {
    font-size: 10px;
  }
  code, pre {
    font-size: 9px !important; 
    white-space: pre !important; 
  }
  table {
    font-size: 9px; 
  }
</style>
<code>

# Raport nr 1: Projekt Bazy Danych

**Przedmiot:** Podstawy Baz Danych 2025/2026
**Zespół:** 2
**Termin zajęć:** Środa, 11:30
**Autorzy:**

* Jakub Staniszewski
* Bartosz Gryń
* Bartłomiej Sromek

---

## 1. Opis funkcji realizowanego systemu

Celem projektu jest stworzenie bazy danych dla firmy produkcyjno-usługowej zajmującej się produkcją i sprzedażą mebli (m.in. krzesła, biurka, fotele gamingowe). System ma za zadanie obsługiwać kluczowe procesy biznesowe: od zarządzania magazynem surowców, przez planowanie produkcji, aż po sprzedaż i raportowanie.

### Główne moduły funkcjonalne:

**A. Zarządzanie Produkcją i Zasobami**

* **Definicja Produktów (BOM):** System przechowuje informacje o produktach oraz ich strukturze (*Bill of Materials*). Każdy produkt ma zdefiniowaną listę części (zasobów) niezbędnych do jego wytworzenia (typ części i ilość).
* **Planowanie Produkcji:** System umożliwia planowanie zleceń produkcyjnych (`ProductionOrders`) z uwzględnieniem czasu potrzebnego na wykonanie oraz dostępności surowców w magazynie.
* **Magazyn Surowców:** Monitorowanie stanów magazynowych zasobów (`Resources`) oraz ich kosztów, co pozwala na przewidywanie kosztów produkcji.


**B. Sprzedaż i Obsługa Klienta**

* **Obsługa Zamówień:** Rejestracja zamówień od klientów indywidualnych i firmowych                       (`Clients`, `ClientOrders`). System obsługuje statusy zamówień oraz terminy oczekiwane.
* **Polityka Rabatowa i Płatności:** Możliwość przydzielania rabatów do zamówień oraz ewidencja płatności (`Payments`).
* **Integracja z Produkcją:** Zamówienia klienckie mogą generować zapotrzebowanie na produkcję, jeśli towaru nie ma w magazynie.

**C. Analityka i Raportowanie**
Projekt uwzględnia stworzenie struktur (widoki, procedury) umożliwiających generowanie raportów wymaganych przez kadrę zarządczą, w tym:

* Raporty kosztów produkcji i sprzedaży w ujęciu czasowym (tygodniowym, miesięcznym).
* Analiza stanów magazynowych i planów produkcyjnych.
* Historia zamówień klientów.

**D. Integracja Produkcja - Sprzedaż**
* Automatyczne generowanie produkcji
* Synchronizacja stanów magazynów
* Zarządzanie terminami

---

## 2. Pierwsza wersja schematu bazy danych

Poniżej przedstawiono schemat logiczny bazy danych (diagram) oraz opis poszczególnych tabel wraz z kodem DDL.

### Diagram ERD (Entity Relationship Diagram)

![diagram](super123.png)

### Opisy poszczególnych tabel:

#### Tabela: BillOfMaterials

```sql
CREATE TABLE BillOfMaterials (
    bom_id int  NOT NULL,
    product_id int  NOT NULL,
    resource_id int  NOT NULL,
    quantity_required decimal(10,2)  NOT NULL,
    CONSTRAINT BillOfMaterials_pk PRIMARY KEY  (bom_id)
);
```


* **Opis:** Tabela łącząca definiująca strukturę produktu (recepturę). Określa, jakie zasoby i w jakiej ilości są potrzebne do wytworzenia jednego produktu.


| Nazwa atrybutu      | Typ             | Opis/Uwagi                                                          |
| :-------------------- | :---------------- | :-------------------------------------------------------------------- |
| `bom_id`            | `int`           | Klucz główny (PK). Unikalny identyfikator wpisu w recepturze.     |
| `product_id`        | `int`           | Klucz obcy (FK) wskazujący na tabelę`Products`.                   |
| `resource_id`       | `int`           | Klucz obcy (FK) wskazujący na tabelę`Resources`.                  |
| `quantity_required` | `decimal(10,2)` | Ilość danego zasobu wymagana do produkcji jednej sztuki produktu. |

---

#### Tabela: Categories

```sql
CREATE TABLE Categories (
    category_id int  NOT NULL,
    name varchar(40)  NOT NULL,
    CONSTRAINT Categories_pk PRIMARY KEY  (category_id)
);
```

* **Opis:** Tabela słownikowa przechowująca kategorie produktów oferowanych przez firmę (np. krzesła, biurka).


| Nazwa atrybutu | Typ           | Opis/Uwagi                      |
| :--------------- | :-------------- | :-------------------------------- |
| `category_id`  | `int`         | Klucz główny (PK).            |
| `name`         | `varchar(40)` | Nazwa kategorii. Pole wymagane. |

---

#### Tabela: ClientOrders

```sql
CREATE TABLE ClientOrders (
    order_id int  NOT NULL,
    client_id int  NOT NULL,
    order_date datetime  NOT NULL DEFAULT current_timestamp,
    expected_date date  NULL,
    total_amount decimal(10,2)  NOT NULL DEFAULT 0,
    status varchar(20)  NOT NULL DEFAULT 'new',
    discount decimal(5,2)  NOT NULL,
    shipper_id int  NOT NULL,
    freight decimal(10,2) NOT NULL DEFAULT 0
    CONSTRAINT ClientOrders_pk PRIMARY KEY  (order_id)
);
```

* **Opis:** Tabela nagłówkowa zamówień, przechowująca ogólne informacje o zamówieniu złożonym przez klienta.


| Nazwa atrybutu  | Typ             | Opis/Uwagi                                                                                 |
| :---------------- | :---------------- | :------------------------------------------------------------------------------------------- |
| `order_id`      | `int`           | Klucz główny (PK).                                                                       |
| `client_id`     | `int`           | Klucz obcy (FK) powiązany z tabelą`Clients`.                                             |
| `order_date`    | `datetime`      | Data złożenia zamówienia. Domyślnie bieżący czas.                                    |
| `expected_date` | `date`          | Oczekiwana data realizacji (może być pusta - NULL).                                      |
| `total_amount`  | `decimal(10,2)` | Łączna kwota zamówienia. Domyślnie 0.                                                  |
| `status`        | `varchar(20)`   | Status zamówienia ('new','processing','shipped','completed','cancelled')Domyślnie 'new'. |
| `discount`      | `decimal(5,2)`  | Wartość udzielonego rabatu 'w procentach %'.                                             |
| `shipper_id`    | `int`           | ID shippera.                                                                               |
| `freight`       | `decimal(10,2)` | Opłata dla przewoźnika.                                                                  |

---

#### Tabela: Clients

```sql
CREATE TABLE Clients (
    client_id int  NOT NULL,
    first_name varchar(15)  NOT NULL,
    last_name varchar(20)  NULL,
    email varchar(60)  NULL,
    client_type varchar(30)  NOT NULL,
    country varchar(30)  NOT NULL,
    city varchar(30)  NOT NULL,
    street varchar(50)  NOT NULL,
    CONSTRAINT Clients_pk PRIMARY KEY  (client_id)
);
```

* **Opis:** Baza danych klientów firmy (zarówno osoby prywatne, jak i firmy).


| Nazwa atrybutu | Typ           | Opis/Uwagi                                     |
| :--------------- | :-------------- | :----------------------------------------------- |
| `client_id`    | `int`         | Klucz główny (PK).                           |
| `first_name`   | `varchar(15)` | Imię klienta lub pierwszy człon nazwy firmy. |
| `last_name`    | `varchar(20)` | Nazwisko klienta (pole opcjonalne - NULL).     |
| `email`        | `varchar(60)` | Adres e-mail (pole opcjonalne - NULL).         |
| `client_type`  | `varchar(30)` | Typ klienta (np. 'indywidualny', 'firma').     |
| `country`      | `varchar(30)` | Kraj klienta.                                  |
| `city`         | `varchar(30)` | Miejscowość klienta.                         |
| `street`       | `varchar(50)` | Ulica klienta.                                 |

---

#### Tabela: OrderDetails

```sql
CREATE TABLE OrderDetails (
    detail_id int  NOT NULL,
    order_id int  NOT NULL,
    product_id int  NOT NULL,
    quantity int  NOT NULL,
    unit_price decimal(10,2)  NOT NULL,
    CONSTRAINT OrderDetails_pk PRIMARY KEY  (detail_id)
);
```

* **Opis:** Tabela przechowująca szczegóły zamówienia (konkretne pozycje towarowe w ramach jednego zamówienia).


| Nazwa atrybutu | Typ             | Opis/Uwagi                                                  |
| :--------------- | :---------------- | :------------------------------------------------------------ |
| `detail_id`    | `int`           | Klucz główny (PK).                                        |
| `order_id`     | `int`           | Klucz obcy (FK) wskazujący na zamówienie w`ClientOrders`. |
| `product_id`   | `int`           | Klucz obcy (FK) wskazujący na produkt w`Products`.         |
| `quantity`     | `int`           | Liczba zamówionych sztuk danego produktu.                  |
| `unit_price`   | `decimal(10,2)` | Cena jednostkowa produktu w momencie zakupu.                |

---

#### Tabela: Payments

```sql
CREATE TABLE Payments (
    payment_id int  NOT NULL,
    order_id int  NOT NULL,
    amount decimal(10,2)  NOT NULL,
    payment_date datetime  NOT NULL DEFAULT current_timestamp,
    status varchar(20) NOT NULL DEFAULT 'unpaid'
    CONSTRAINT Payments_pk PRIMARY KEY  (payment_id)
);
```

* **Opis:** Rejestr płatności dokonanych do zamówień.


| Nazwa atrybutu | Typ             | Opis/Uwagi                                            |
| :--------------- | :---------------- | :------------------------------------------------------ |
| `payment_id`   | `int`           | Klucz główny (PK).                                  |
| `order_id`     | `int`           | Klucz obcy (FK) wskazujący na opłacane zamówienie. |
| `amount`       | `decimal(10,2)` | Wpłacona kwota.                                      |
| `payment_date` | `datetime`      | Data płatności. Domyślnie bieżący czas.          |
| `status`       | `varchar(20)`   | Status płatności (unpaid,paid).                     |

---

#### Tabela: ProductionOrders
```sql
CREATE TABLE ProductionOrders (
    production_id int  NOT NULL,
    detail_id int  NOT NULL,
    quantity_to_produce int  NOT NULL,
    status varchar(20)  NOT NULL DEFAULT 'unfinished',
    client_order_id int  NULL,
    start_date datetime  NOT NULL,
    end_date datetime  NOT NULL,
    CONSTRAINT ProductionOrders_pk PRIMARY KEY  (production_id)
    
);
```

* **Opis:** Tabela służąca do planowania i monitorowania zleceń produkcyjnych.


| Nazwa atrybutu        | Typ           | Opis/Uwagi                                                 |
| :---------------------- | :-------------- | :----------------------------------------------------------- |
| `production_id`       | `int`         | Klucz główny (PK).                                       |
| `detail_id`           | `int`         | Klucz obcy (FK) - produkt, który jest wytwarzany.         |
| `quantity_to_produce` | `int`         | Zaplanowana ilość do produkcji.                          |
| `status`              | `varchar(20)` | Status zlecenia (np. 'finished'). Domyślnie 'unfinished'. |
| `start_date`          | `datetime`    | Planowana data rozpoczęcia.                               |
| `end_date`            | `datetime`    | Planowana data zakończenia.                               |
| `shipper_id`          | `int`         | ID przewoźnika.                                           |

---

#### Tabela: Products

```sql
CREATE TABLE Products (
    product_id int  NOT NULL,
    name varchar(25)  NOT NULL,
    current_stock int  NOT NULL DEFAULT 0,
    base_price decimal(10,2)  NOT NULL,
    per_day int  NOT NULL,
    category_id int  NOT NULL,
    CONSTRAINT Products_pk PRIMARY KEY  (product_id)
);
```

* **Opis:** Główna tabela produktów gotowych oferowanych przez firmę.


| Nazwa atrybutu  | Typ             | Opis/Uwagi                                          |
| :---------------- | :---------------- | :---------------------------------------------------- |
| `product_id`    | `int`           | Klucz główny (PK).                                |
| `name`          | `varchar(25)`   | Nazwa produktu.                                     |
| `current_stock` | `int`           | Aktualny stan magazynowy. Domyślnie 0.             |
| `base_price`    | `decimal(10,2)` | Cena bazowa produktu.                               |
| `per_day`       | `int`           | Ilość wyprodukowanego produktu dziennie.          |
| `category_id`   | `int`           | Klucz obcy (FK) wskazujący na kategorię produktu. |

---

#### Tabela: Resources

```sql
CREATE TABLE Resources (
    resource_id int  NOT NULL,
    name varchar(25)  NOT NULL,
    unit varchar(10)  NOT NULL,
    current_stock decimal(10,2)  NOT NULL DEFAULT 0,
    unit_cost decimal(10,2)  NOT NULL DEFAULT 0,
    CONSTRAINT Resources_pk PRIMARY KEY  (resource_id)
);
```

* **Opis:** Tabela magazynowa surowców i części niezbędnych do produkcji.


| Nazwa atrybutu  | Typ             | Opis/Uwagi                              |
| :---------------- | :---------------- | :---------------------------------------- |
| `resource_id`   | `int`           | Klucz główny (PK).                    |
| `name`          | `varchar(25)`   | Nazwa surowca.                          |
| `unit`          | `varchar(10)`   | Jednostka miary (np. kg, szt).          |
| `current_stock` | `decimal(10,2)` | Ilość w magazynie. Domyślnie 0.      |
| `unit_cost`     | `decimal(10,2)` | Koszt jednostkowy zakupu. Domyślnie 0. |

---

#### Tabela: Shippers

```sql
CREATE TABLE Shippers (
    shipper_id int  NOT NULL,
    company_name varchar(40)  NOT NULL,
    phone varchar(15)  NOT NULL,
    CONSTRAINT Shippers_pk PRIMARY KEY  (shipper_id)
);
```

* **Opis:** Tabela z przewoźnikami


| Nazwa atrybutu | Typ           | Opis/Uwagi                   |
| :--------------- | :-------------- | :----------------------------- |
| `shipper_id`   | `int`         | Klucz główny (PK).         |
| `company_name` | `varchar(25)` | Nazwa przewoźnika.          |
| `phone`        | `varchar(40)` | Numer telefonu przewoźnika. |

---

### Definicja Relacji (Klucze Obce)

Poniższy kod uzupełnia powyższe tabele relacje między tabelami.

```sql
-- Reference: ClientOrders_Shippers (table: ClientOrders)
ALTER TABLE ClientOrders ADD CONSTRAINT ClientOrders_Shippers
    FOREIGN KEY (shipper_id)
    REFERENCES Shippers (shipper_id);

-- Reference: FK_BOM_Products (table: BillOfMaterials)
ALTER TABLE BillOfMaterials ADD CONSTRAINT FK_BOM_Products
    FOREIGN KEY (product_id)
    REFERENCES Products (product_id);

-- Reference: FK_BOM_Resources (table: BillOfMaterials)
ALTER TABLE BillOfMaterials ADD CONSTRAINT FK_BOM_Resources
    FOREIGN KEY (resource_id)
    REFERENCES Resources (resource_id);

-- Reference: FK_ClientOrders_Clients (table: ClientOrders)
ALTER TABLE ClientOrders ADD CONSTRAINT FK_ClientOrders_Clients
    FOREIGN KEY (client_id)
    REFERENCES Clients (client_id);

-- Reference: FK_OrderDetails_ClientOrders (table: OrderDetails)
ALTER TABLE OrderDetails ADD CONSTRAINT FK_OrderDetails_ClientOrders
    FOREIGN KEY (order_id)
    REFERENCES ClientOrders (order_id);

-- Reference: FK_OrderDetails_Products (table: OrderDetails)
ALTER TABLE OrderDetails ADD CONSTRAINT FK_OrderDetails_Products
    FOREIGN KEY (product_id)
    REFERENCES Products (product_id);

-- Reference: FK_Payments_ClientOrders (table: Payments)
ALTER TABLE Payments ADD CONSTRAINT FK_Payments_ClientOrders
    FOREIGN KEY (order_id)
    REFERENCES ClientOrders (order_id);

-- Reference: FK_Products_Categories (table: Products)
ALTER TABLE Products ADD CONSTRAINT FK_Products_Categories
    FOREIGN KEY (category_id)
    REFERENCES Categories (category_id);

-- Reference: ProductionOrders_OrderDetails (table: ProductionOrders)
ALTER TABLE ProductionOrders ADD CONSTRAINT ProductionOrders_OrderDetails
    FOREIGN KEY (detail_id)
    REFERENCES OrderDetails (detail_id);
```

---

### Funkcje

1. Oblicz koszt materiałowy (surowców) dla jednego produktu
   Obliczanie kosztu materiałowego. Funkcja pobiera identyfikator produktu (`product_id`) i na podstawie tabeli `BillOfMaterials` (receptury) oraz `Resources` (cennik surowców) sumuje koszt wszystkich składników niezbędnych do wytworzenia jednej sztuki mebla. Zwraca kwotę (DECIMAL), która stanowi bazę do wyliczenia marży.

```sql
CREATE FUNCTION f_GetMaterialCost (
    @product_id INT
)
RETURNS DECIMAL(10,2)
AS
BEGIN
    DECLARE @cost DECIMAL(10,2);
  
    SELECT @cost = SUM(bom.quantity_required * r.unit_cost)
    FROM BillOfMaterials bom
    JOIN Resources r ON bom.resource_id = r.resource_id
    WHERE bom.product_id = @product_id;

    RETURN ISNULL(@cost, 0);
END;
GO
```

2. Oblicz zysk (marżę) na jednej sztuce produktu
   Kalkulacja marży produktu. Funkcja wylicza zysk na pojedynczej sztuce. Pobiera cenę sprzedaży z tabeli `Products` i odejmuje od niej koszt materiałowy obliczony przez funkcję `f_GetMaterialCost`. Pozwala na szybką analizę rentowności poszczególnych produktów.
   Wykorzystuje funkcję nr 1

```sql
CREATE FUNCTION f_CalculateMargin (
    @product_id INT
)
RETURNS DECIMAL(10,2)
AS
BEGIN
    DECLARE @selling_price DECIMAL(10,2);
    DECLARE @material_cost DECIMAL(10,2);

    SELECT @selling_price = base_price FROM Products WHERE product_id = @product_id;
    SET @material_cost = dbo.f_GetMaterialCost(@product_id);

    RETURN ISNULL(@selling_price - @material_cost, 0);
END;
GO
```

3. Oblicz wartość zamówienia po rabacie
   Wycena zamówienia z rabatem. Funkcja służy do ostatecznego rozliczenia zamówienia. Sumuje wartość wszystkich pozycji (`OrderDetails`) dla danego zamówienia, a następnie pomniejsza tę kwotę o procentowy rabat zdefiniowany w nagłówku zamówienia (`ClientOrders`). Zwraca kwotę netto do zapłaty.

```sql
CREATE FUNCTION f_CalculateOrderValueWithDiscount (
    @order_id INT
)
RETURNS DECIMAL(10,2)
AS
BEGIN
    DECLARE @raw_total DECIMAL(10,2);
    DECLARE @discount_val DECIMAL(5,2); -- np. 0.3
    DECLARE @final_value DECIMAL(10,2);

    SELECT @raw_total = SUM(quantity * unit_price) 
    FROM OrderDetails 
    WHERE order_id = @order_id;

    SELECT @discount_val = discount 
    FROM ClientOrders 
    WHERE order_id = @order_id;


    SET @final_value = ISNULL(@raw_total, 0) * (1.00 - ISNULL(@discount_val, 0));

    RETURN @final_value;
END;
GO
```

4. Policz sprzedaż produktu z ostatnich 30 dni (Trend sprzedaży)
   Analiza trendu sprzedaży (30 dni). Funkcja analityczna zwracająca liczbę sztuk konkretnego produktu sprzedanych w ciągu ostatnich 30 dni. Służy do identyfikacji bestsellerów oraz produktów zalegających, co wspomaga decyzje o planowaniu produkcji.

```sql
CREATE FUNCTION f_GetProductSalesLastMonth (
    @product_id INT
)
RETURNS INT
AS
BEGIN
    DECLARE @sold_qty INT;

    SELECT @sold_qty = SUM(od.quantity)
    FROM OrderDetails od
    JOIN ClientOrders co ON od.order_id = co.order_id
    WHERE od.product_id = @product_id
      AND co.order_date >= DATEADD(DAY, -30, GETDATE());

    RETURN ISNULL(@sold_qty, 0);
END;
GO
```

5. Sprawdź czy zamówienie jest opóźnione
   Weryfikacja opóźnień. Funkcja logiczna zwracająca wartość `1` (Prawda) lub `0` (Fałsz). Sprawdza, czy dla danego zamówienia minęła oczekiwana data realizacji (`expected_date`), a status zamówienia nadal wskazuje na brak realizacji (nie jest wysłane, zakończone ani anulowane).

```sql
CREATE FUNCTION f_IsOrderLate (
    @order_id INT
)
RETURNS BIT
AS
BEGIN
    DECLARE @expected_date DATE;
    DECLARE @status VARCHAR(20);
    DECLARE @is_late BIT = 0;

    SELECT @expected_date = expected_date, @status = status
    FROM ClientOrders
    WHERE order_id = @order_id;

    -- Jeśli data minęła, a status nie jest 'shipped' ani 'completed'
    IF @expected_date < GETDATE() AND @status NOT IN ('shipped', 'completed', 'cancelled')
        SET @is_late = 1;

    RETURN @is_late;
END;
GO
```

---

### Widoki

Raport Kosztów Produkcji.

Widok finansowy służący do analizy kosztów wytworzenia produktów w ujęciu historycznym. Agreguje dane tylko dla produkcji zakończonej (status = 'completed').

Kluczowe funkcje:

• Grupowanie czasowe: Rok, Kwartał, Miesiąc.

• Wyliczanie kosztu jednostkowego na podstawie aktualnych cen surowców i receptur (BOM).

• Podsumowanie kosztów całkowitych dla grup produktów.

• Służy do monitorowania rentowności produkcji.

```sql
CREATE VIEW vw_ProductionCostsReport AS
SELECT
    YEAR(pr.start_date) AS rok,
    DATEPART(QUARTER, pr.start_date) AS kwartal,
    MONTH(pr.start_date) AS miesiac,
    p.category_id,
    cat.name AS grupa_produktow,
    p.product_id,
    p.name AS produkt,
    COUNT(pr.production_id) AS liczba_produkcji,
    SUM(pr.quantity_to_produce) AS ilosc_wyprodukowana,
    SUM(bom.quantity_required * r.unit_cost) / NULLIF(COUNT(DISTINCT p.product_id), 1) AS koszt_jednostkowy,
    SUM(pr.quantity_to_produce * bom.quantity_required * r.unit_cost) AS koszt_calkowity
FROM ProductionOrders pr
JOIN OrderDetails od ON pr.detail_id = od.detail_id
JOIN Products p ON od.product_id = p.product_id
JOIN Categories cat ON p.category_id = cat.category_id
JOIN BillOfMaterials bom ON p.product_id = bom.product_id
JOIN Resources r ON bom.resource_id = r.resource_id
WHERE pr.status = 'completed'
GROUP BY
    YEAR(pr.start_date),
    DATEPART(QUARTER, pr.start_date),
    MONTH(pr.start_date),
    p.category_id,
    cat.name,
    p.product_id,
    p.name;
```

Raport Magazynowy.

Kluczowy widok dla logistyki, łączący dane o stanach fizycznych z planami produkcyjnymi i sprzedażowymi.

Logika statusów (status_magazynu):

• 'NISKI STAN': Gdy fizyczny stan spadnie poniżej 10 sztuk.

• 'WYMAGA PRODUKCJI': Gdy popyt (zamówienia oczekujące) przewyższa dostępność (stan obecny).

• 'OK': W pozostałych przypadkach.

Widok uwzględnia również towary będące w trakcie produkcji (zaplanowane_do_produkcji).

```sql
CREATE VIEW vw_InventoryStatusReport AS
SELECT
    p.product_id,
    p.name AS produkt,
    cat.name AS kategoria,
    p.current_stock AS stan_biezacy,
    ISNULL((
        SELECT SUM(pr2.quantity_to_produce)
        FROM ProductionOrders pr2
        JOIN OrderDetails od2 ON pr2.detail_id = od2.detail_id
        WHERE od2.product_id = p.product_id
        AND pr2.status IN ('planned', 'in_progress')
    ), 0) AS zaplanowane_do_produkcji,
    ISNULL((
        SELECT SUM(od3.quantity)
        FROM OrderDetails od3
        JOIN ClientOrders o3 ON od3.order_id = o3.order_id
        WHERE od3.product_id = p.product_id
        AND o3.status NOT IN ('cancelled', 'completed')
    ), 0) AS zamowienia_oczekujace,
    CASE
        WHEN p.current_stock < 10 THEN 'NISKI STAN'
        WHEN p.current_stock < ISNULL((
            SELECT SUM(od3.quantity)
            FROM OrderDetails od3
            JOIN ClientOrders o3 ON od3.order_id = o3.order_id
            WHERE od3.product_id = p.product_id
            AND o3.status NOT IN ('cancelled', 'completed')
        ), 0) THEN 'WYMAGA PRODUKCJI'
        ELSE 'OK'
    END AS status_magazynu
FROM Products p
LEFT JOIN Categories cat ON p.category_id = cat.category_id;
```

Historia Zamówień i Analiza Rabatów.

Widok CRM (Customer Relationship Management). Prezentuje pełną historię zrealizowanych transakcji w podziale na klientów i okresy czasu.

Kluczowe metryki:

• Wyliczenie kwotowe udzielonego rabatu (rabat_kwota).

• Ostateczna wartość sprzedaży netto (kwota_po_rabacie).

• Analiza wolumenu zakupów (liczba produktów i łączne ilości).

```sql
CREATE VIEW vw_ClientOrderHistoryReport AS
SELECT
    c.client_id,
    c.first_name + ' ' + ISNULL(c.last_name, '') AS klient,
    YEAR(o.order_date) AS rok,
    DATEPART(QUARTER, o.order_date) AS kwartal,
    MONTH(o.order_date) AS miesiac,
    o.order_id,
    o.order_date,
    o.status,
    o.total_amount,
    o.discount AS rabat_procent,
    o.total_amount * (o.discount) AS rabat_kwota,
    o.total_amount - (o.total_amount * (o.discount)) AS kwota_po_rabacie,
    COUNT(od.detail_id) AS liczba_produktow,
    SUM(od.quantity) AS laczna_ilosc
FROM Clients c
JOIN ClientOrders o ON c.client_id = o.client_id
JOIN OrderDetails od ON o.order_id = od.order_id
WHERE o.status = 'completed'
GROUP BY
    c.client_id, c.first_name, c.last_name,
    YEAR(o.order_date), DATEPART(QUARTER, o.order_date), MONTH(o.order_date),
    o.order_id, o.order_date, o.status, o.total_amount, o.discount;
```

Zarządczy Raport Sprzedaży i Rentowności.

Zaawansowany widok analityczny dla kadry zarządzającej. Zestawia przychody ze sprzedaży z kosztami produkcji w ujęciu tygodniowym i miesięcznym.

Kluczowe obliczenia:

• Przychód Netto: Sprzedaż pomniejszona o udzielone rabaty.

• Koszt Produkcji: Koszt materiałowy sprzedanych wolumenów.

• Zysk: Różnica między przychodem netto a kosztem produkcji.

Pozwala na szybką ocenę kondycji finansowej firmy.

```sql
CREATE VIEW vw_ManagementSalesReport AS
SELECT
    YEAR(o.order_date) AS rok,
    MONTH(o.order_date) AS miesiac,
    DATEPART(WEEK, o.order_date) AS tydzien,
    p.category_id,
    cat.name AS grupa_produktow,
    COUNT(DISTINCT o.order_id) AS liczba_zamowien,
    SUM(od.quantity) AS sprzedana_ilosc,
    SUM(od.quantity * od.unit_price) AS przychod_brutto,
    SUM(od.quantity * od.unit_price * (1 - o.discount)) AS przychod_netto,
    SUM(od.quantity * bom.quantity_required * r.unit_cost) AS koszt_produkcji,
    SUM(od.quantity * od.unit_price * (1 - o.discount)) -
    SUM(od.quantity * bom.quantity_required * r.unit_cost) AS zysk
FROM ClientOrders o
JOIN OrderDetails od ON o.order_id = od.order_id
JOIN Products p ON od.product_id = p.product_id
JOIN Categories cat ON p.category_id = cat.category_id
JOIN BillOfMaterials bom ON p.product_id = bom.product_id
JOIN Resources r ON bom.resource_id = r.resource_id
WHERE o.status = 'completed'
GROUP BY
    YEAR(o.order_date),
    MONTH(o.order_date),
    DATEPART(WEEK, o.order_date),
    p.category_id,
    cat.name;
```

Harmonogram Operacyjny Produkcji.

Widok operacyjny dla kierownika hali produkcyjnej. Pokazuje aktywne zlecenia (planned, in_progress).

Logika Priorytetów:

System dynamicznie oblicza priorytet na podstawie terminu zamówienia klienta:

• WYSOKI: Mniej niż 3 dni do terminu.

• ŚREDNI: 3-7 dni do terminu.

• NISKI: Powyżej tygodnia.

Widok pokazuje również szacowany czas trwania produkcji (dni_produkcji).

```sql
CREATE VIEW vw_ProductionPlanReport AS
SELECT
    YEAR(pr.start_date) AS rok,
    DATEPART(QUARTER, pr.start_date) AS kwartal,
    MONTH(pr.start_date) AS miesiac,
    DATEPART(WEEK, pr.start_date) AS tydzien,
    p.product_id,
    p.name AS produkt,
    cat.name AS kategoria,
    pr.production_id,
    pr.quantity_to_produce AS planowana_ilosc,
    pr.start_date AS data_rozpoczecia,
    pr.end_date AS data_zakonczenia,
    pr.status AS status_produkcji,
    o.order_id,
    o.expected_date AS termin_zamowienia,
    c.first_name + ' ' + ISNULL(c.last_name, '') AS klient,
    DATEDIFF(DAY, pr.start_date, pr.end_date) AS dni_produkcji,
    CASE
        WHEN DATEDIFF(DAY, GETDATE(), o.expected_date) < 3 THEN 'WYSOKI'
        WHEN DATEDIFF(DAY, GETDATE(), o.expected_date) < 7 THEN 'ŚREDNI'
        ELSE 'NISKI'
    END AS priorytet
FROM ProductionOrders pr
JOIN OrderDetails od ON pr.detail_id = od.detail_id
JOIN Products p ON od.product_id = p.product_id
JOIN Categories cat ON p.category_id = cat.category_id
JOIN ClientOrders o ON od.order_id = o.order_id
LEFT JOIN Clients c ON o.client_id = c.client_id
WHERE pr.status IN ('planned', 'in_progress');
```

---

### Trigery

1. Wydanie towaru z magazynu (Wysyłka)
   Cel: Gdy status zamówienia w `ClientOrders` zmieni się na `'shipped'` (wysłane), trigger zdejmuje gotowe produkty ze stanu magazynowego `Products`. Tabela: `ClientOrders`

```sql
CREATE TRIGGER TR_ShipProducts
ON ClientOrders
AFTER UPDATE
AS
BEGIN
    SET NOCOUNT ON;

    -- Sprawdź, czy status zmienił się na 'shipped'
    IF EXISTS (
        SELECT 1 
        FROM Inserted i 
        JOIN Deleted d ON i.order_id = d.order_id 
        WHERE i.status = 'shipped' AND d.status <> 'shipped'
    )
    BEGIN
        -- Zmniejsz stan magazynowy produktów gotowych
        UPDATE p
        SET current_stock = p.current_stock - od.quantity
        FROM Products p
        JOIN OrderDetails od ON p.product_id = od.product_id
        JOIN Inserted i ON od.order_id = i.order_id;
    END
END;
GO
```

2.Pobranie surowców przy starcie produkcji (MRP)
Cel: W momencie utworzenia zlecenia produkcyjnego (`ProductionOrders`), trigger automatycznie odejmuje potrzebne surowce z magazynu `Resources` na podstawie receptury (`BOM`). Tabela: `ProductionOrders`

```sql
CREATE TRIGGER TR_ConsumeRawMaterials
ON ProductionOrders
AFTER INSERT
AS
BEGIN
    SET NOCOUNT ON;

    UPDATE r
    SET current_stock = r.current_stock - (bom.quantity_required * i.quantity_to_produce)
    FROM Resources r
    JOIN BillOfMaterials bom ON r.resource_id = bom.resource_id
    JOIN OrderDetails od ON bom.product_id = od.product_id 
    JOIN Inserted i ON i.detail_id = od.detail_id        
    WHERE i.status IN ('planned', 'in_progress');
END;
GO
```

Automatyczne przeliczanie wartości zamówienia
Cel: Gdy dodasz, usuniesz lub zmienisz pozycję w `OrderDetails`, trigger automatycznie przeliczy sumę w `ClientOrders` (uwzględniając rabat). Tabela: `OrderDetails`

```sql
CREATE TRIGGER TR_CalculateOrderTotal
ON OrderDetails
AFTER INSERT, UPDATE, DELETE
AS
BEGIN
    SET NOCOUNT ON;
    DECLARE @order_id INT;

    SELECT TOP 1 @order_id = order_id FROM Inserted;
    IF @order_id IS NULL 
        SELECT TOP 1 @order_id = order_id FROM Deleted;

    UPDATE ClientOrders
    SET total_amount = (
        SELECT ISNULL(SUM(quantity * unit_price), 0)
        FROM OrderDetails
        WHERE order_id = @order_id
    ) * (1.00 - ISNULL(discount, 0)) 
    WHERE order_id = @order_id;
END;
GO
-- automatyczne dodanie produktu do magazynu po zakończeniu jego produkcji

CREATE TRIGGER TR_AddToStockWhenProduced
ON ProductionOrders
AFTER UPDATE
AS
BEGIN
    SET NOCOUNT ON;
    IF EXISTS (
        SELECT 1
        FROM Inserted i
        JOIN Deleted d ON i.production_id = d.production_id
        WHERE i.status = 'finished' AND d.status <> 'finished'
    )
    BEGIN
        UPDATE p
        SET p.current_stock = p.current_stock + i.quantity_to_produce
        FROM Products p
        JOIN OrderDetails od ON p.product_id = od.product_id
        JOIN Inserted i ON od.detail_id = i.detail_id;
    END
END;
GO

-- automatyczne zlecenie produkcji, gdy brakuje towaru

CREATE TRIGGER TR_AutoCreateProductionOrder
ON OrderDetails
AFTER INSERT
AS
BEGIN
    SET NOCOUNT ON;
    INSERT INTO ProductionOrders (
        production_id,
        quantity_to_produce,
        status,
        start_date,
        end_date,
        detail_id
    )
    SELECT
        NEXT VALUE FOR sys.sequence_object_id,
        i.quantity - p.current_stock,
        'unfinished',
        GETDATE(),
        DATEADD(DAY, CEILING(CAST(i.quantity - p.current_stock AS FLOAT) / p.per_day), GETDATE()),
        i.detail_id
    FROM Inserted i
    JOIN Products p ON i.product_id = p.product_id
    WHERE p.current_stock < i.quantity;
END;
GO
```

Automatyzacja przychodu z produkcji.

Wyzwalacz uruchamia się w momencie aktualizacji statusu zlecenia produkcyjnego na 'finished'. Automatycznie zwiększa stan magazynowy (current_stock) w tabeli Products o ilość wyprodukowaną w ramach tego zlecenia. Zamyka to cykl produkcyjny, udostępniając towar do wysyłki

```sql
-- automatyczne dodanie produktu do magazynu po zakończeniu jego produkcji

CREATE TRIGGER TR_AddToStockWhenProduced
ON ProductionOrders
AFTER UPDATE
AS
BEGIN
    SET NOCOUNT ON;
    IF EXISTS (
        SELECT 1
        FROM Inserted i
        JOIN Deleted d ON i.production_id = d.production_id
        WHERE i.status = 'finished' AND d.status <> 'finished'
    )
    BEGIN
        UPDATE p
        SET p.current_stock = p.current_stock + i.quantity_to_produce
        FROM Products p
        JOIN OrderDetails od ON p.product_id = od.product_id
        JOIN Inserted i ON od.detail_id = i.detail_id;
    END
END;
GO
```

### Procedury

1.Rejestracja nowego klienta.
Procedura dodaje rekord do tabeli Clients. Przyjmuje podstawowe dane adresowe (imię, nazwisko, typ, adres). Unikalny identyfikator (client_id) jest generowany losowo wewnątrz procedury, co upraszcza proces dodawania rekordu przez aplikację zewnętrzną.

```sql
CREATE PROCEDURE AddClient
    @first_name VARCHAR(15),
    @last_name VARCHAR(20),
    @client_type VARCHAR(30),
    @country VARCHAR(30),
    @city VARCHAR(30),
    @street VARCHAR(50)
AS
BEGIN
    INSERT INTO Clients (client_id, first_name, last_name, email, client_type, country, city, street)
    VALUES (
        CAST(ABS(CHECKSUM(NEWID())) % 1000000 AS INT),  -- losowe ID
        @first_name,
        @last_name,
        NULL,
        @client_type,
        @country,
        @city,
        @street
        )
END;
```

2.Usuwanie klienta.
Procedura realizuje bezpieczne usunięcie klienta z bazy danych. Aby zachować spójność danych, najpierw usuwa wszystkie powiązane zamówienia z tabeli ClientOrders (dla wskazanego client_id), a dopiero następnie usuwa właściwy rekord klienta z tabeli Clients.

```sql
CREATE PROCEDURE DelClient
    @client_id int
AS
BEGIN
    DELETE FROM ClientOrders WHERE client_id = @client_id;
    DELETE FROM Clients WHERE client_id = @client_id;
END;
```

3.Zmiana statusu zamówienia.
Służy do aktualizacji pola status w tabeli ClientOrders. Procedura posiada wbudowaną walidację – przed zmianą sprawdza, czy zamówienie istnieje oraz czy nowy status należy do listy dozwolonych wartości (np. 'shipped', 'cancelled', 'processing'). W przypadku błędu zwraca komunikat przy pomocy THROW.

```sql
CREATE PROCEDURE UpdateOrderStatus
    @order_id INT,
    @new_status VARCHAR(20)
AS
BEGIN
    IF NOT EXISTS (SELECT 1 FROM ClientOrders WHERE order_id = @order_id)
    BEGIN
        DECLARE @msg_id VARCHAR(100) = 'Zamówienie ' + CAST(@order_id AS VARCHAR(20)) + ' nie istnieje.';
        ;THROW 51000, @msg_id, 1;
    END

    IF @new_status NOT IN ('new','processing','shipped','completed','cancelled')
    BEGIN
        DECLARE @msg_status VARCHAR(100) = 'Nieprawidłowy status: ' + @new_status;
        ;THROW 51001, @msg_status, 1;
    END

    UPDATE ClientOrders
    SET status = @new_status
    WHERE order_id = @order_id;
END;
```

4.Anulowanie zamówienia.
Procedura obsługująca proces rezygnacji z zamówienia. Weryfikuje, czy zamówienie nie zostało już zrealizowane (completed). Jeśli anulowanie jest możliwe, procedura aktualizuje status w tabeli ClientOrders, a następnie kaskadowo anuluje powiązane zlecenia produkcyjne (ProductionOrders) oraz płatności (Payments), zapewniając spójność wszystkich modułów systemu.

```sql
CREATE PROCEDURE CancelOrder
    @order_id INT
AS
BEGIN
    IF @order_id NOT IN (SELECT order_id FROM ClientOrders)
    BEGIN
        DECLARE @msg VARCHAR(100) = 'Nieprawidłowy numer zamówienia: ' + CAST(@order_id AS VARCHAR(20));
        ;THROW 51000, @msg, 1;
    END

    DECLARE @current_status VARCHAR(20);
    SELECT @current_status = status FROM ClientOrders WHERE order_id = @order_id;

    IF @current_status = 'cancelled'
    BEGIN
        RETURN;
    END

    IF @current_status = 'completed'
    BEGIN
        ;THROW 51001, 'Nie można anulować zrealizowanego zamówienia.', 1;
    END

    UPDATE ClientOrders
    SET status = 'cancelled'
    WHERE order_id = @order_id;

    UPDATE ProductionOrders
    SET status = 'cancelled'
    WHERE detail_id IN (
        SELECT detail_id
        FROM OrderDetails
        WHERE order_id = @order_id
    );

    UPDATE Payments
    SET status = 'cancelled'
    WHERE order_id = @order_id;

END;
```

Dodatkowe obiekty (Typy danych)
Aby procedura CreateOrder mogła przyjąć listę produktów, zdefiniowano własny typ tabelaryczny:

Typ: OrderItemList Jest to struktura tabelaryczna przechowywana w pamięci, służąca do przekazywania listy produktów (ID produktu i ilość) do procedury składowanej. Pozwala to na dodanie wielu pozycji zamówienia w ramach jednego wywołania procedury.

```sql
CREATE TYPE OrderItemList AS TABLE (
    product_id INT NOT NULL,
    quantity INT NOT NULL,
);
```
</code>