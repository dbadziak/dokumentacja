# Lite WMS 
# Interface wymiany danych, komunikat Dokument


Dokument zawiera ustandaryzowany format pozwalający wymieniać informacje związane z awizacją dostaw i zamówieniami od klientów.


## Obiekt naglowek


| Pole | Wymagane | Opis | Typ danych| Pole WMS |
|--|--|--|--|--|
|typ| T | określa typ dokumentu przyjmowane wartości **OUT** dla dokumentów wejściowych **IN** dla dokumentów wyjściowych | varchar(5)  
|zleceniodawca|T |kod zleceniodawcy z WMS Z punktu widzenia klienta (systemu ERP) mozna traktowac jako stałą)| nvarchar(25)  | `dord_code`
centrum_logistyczne|T |centrum logistyczne z WMS | nvarchar(25)  |`whc_code`
data_realizacji|N |żądana data realizacji|smalldatetime | `door_expectedCompletion`
priorytet|N |priorytet wartiosci od 0 - 89 priorytet zero jest najwyższy| smallint |`door_expectedCompletion`
nr_alternatywny_dokumentu|T |kod alternatywny zamówienia - kod zamówienia z systemu ERP klienta| nvarchar(50) | `door_alternativeCode`
opis|N|Przykładowy opis|nvarchar(500) | `door_description`
kontrahent|T| Obiekt zawiera dane kontrahenta (klienta/dostawcy w zależności od typu dokumentu|Obiekt|
kurier|N| Obiekt zawiara dane potrzebne do wystawiania listu przewozowego z poziomu systemu WMS|Obiekt|
atrybuty|N| Atrybuty nagłówka dokumentu|Kolekcja|


### Komunikat zwrotny
Zawiera to co komunikat wejściowy poszerzone o pola:


| Pole | Wymagane | Opis | Typ danych| Pole WMS |
|--|--|--|--|--|
OUT_nr_dokumentu| |[Tylko dla komunikatu zwrotnego] nr dokumentu w WMS|nvarchar(25)  | `ddoc_code`
OUT_data_utworzenia| |[Tylko dla komunikatu zwrotnego] data utworzenia/importu dokumentu|Datetime | `door_dateCreated`
OUT_data_zamkniecia| |[Tylko dla komunikatu zwrotnego] data zamknięcia dokumentu|Datetime | `door_dateClosed`




## Obiekt kontrahent


Obiekt zawiera dane kontrahenta (klienta/dostawcy) w zależności od typu dokumentu. Jeżeli dane kontrahenta nie istnieją w słowniku firm nowa firma zostanie automatycznie dodana do słownika.


 Pole | Wymagane | Opis | Typ danych| Pole WMS |
--|--|--|--|--|
kod|T |kod_firmy|nvarchar(50)
nazwa|T |nazwa firmy|nvarchar(100)
ulica|T |nazwa ulicy z numerem domu|nvarchar(100) 
kod_pocztowy|T|format zależny od kraju |varchar(10)
miasto|T |nazwa miejscowości|nvarchar(100) 
kraj|N |kod kraju jeśli puste wstawiany **PL**|varchar(2)


Przykład JSON:
```json
"kontrahent": {
                "kod": "F00001",
                "nazwa": "Logsoft",
                "ulica": "Papiernicza 7e",
                "kod_pocztowy": "93-400",
                "miasto": "Łódź",
                "kraj": "PL"
            },
```

Przykład XML:
```XML
<kontrahent>
	<kod>F00001</kod>
	<nazwa>Logsoft</nazwa>
	<ulica>Papiernicza 7e</ulica>
	<kod_pocztowy>93-400</kod_pocztowy>
	<miasto>Łódź</miasto>
	<kraj>PL</kraj>
</kontrahent>
```



## Obiekt kurier


Sekcja nie jest obowiązkowa dotyczy tylko zleceń o typie ***OUT*** i jest używana tylko w przypadku gdy z systemu wystawiany jest list przewozowy


| Pole | Wymagane | Opis | Typ danych| Pole WMS |
|--|--|--|--|--|
|usluga|T |DHL Standard|varchar(50) |`door_tr_service`
|COD|N | kwota pobrania |decimal(18,6)| `door_tr_COD`
|kwota_ubezpieczenia|N |deklarowana kwota do ubezpieczenia (jesli pobranie kwota musi być >= COD)| decimal(18,6)| `door_tr_packageValue`
|telefon|N |numer telefonu kontaktowego odbiorcy|varchar(50) |`door_tr_contactPhone`
|email|N |mail kontaktowy|varchar(50) | `door_tr_contact`
|dodatkowe_info|N |informacje dodatkowe do wydruku na etykiecie|varchar(50) |`door_tr_description`


### Komunikat zwrotny
Zawiera to co komunikat wejściowy poszerzone o pola:


| Pole | Wymagane | Opis | Typ danych| Pole WMS |
|--|--|--|--|--|
|OUT_nr_listu_przewozowego|N |[Tylko dla komunikatu zwrotnego] numer wygenerowanego listu przewozowego |nvarchar(50) |`door_tr_truckingNumber`


> do dodania pola baselinkerID, Inpost_gabaryt, Inpost_paczkomat


Przykład JSON:
```json
    "kurier": {
                "usluga": "DHL Standard",
                "COD": "156.23",
                "kwota_ubezpieczenia": "160",
                "telefon": "555-666-777",
                "email": "klient@kontakt.pl",
                "dodatkowe_info": "inforamcje dodatkowe",
             },
```

Przykład XML:
```XML
<kurier>
	<usluga>DHL Standard</usluga>
	<COD>156.23</COD>
	<kwota_ubezpieczenia>500</kwota_ubezpieczenia>
	<telefon>555-666-777</telefon>
	<email>klient@kontakt.pl</email>
	<dodatkowe_info>Uwaga szkło</dodatkowe_info>
</kurier>
```

## Kolekcja atrybuty


Kolekcja atrybuty może posiadać Max 20 obiektów. Atrybuty, których nazwa będzie niezgodna z nazwą w definicji atrybutów systemu WMS będą ignorowane. Niewymagane


| Pole | Wymagane | Opis | Typ danych| Pole WMS |
|--|--|--|--|--|
|nazwa|T | kod atrybutu z definicji atrybutów systemu WMS | varchar(50) |`pdef_code`
|Wartosc|T |wartość wstawiana do odpowiedniego atrybutu nagłówka dokumentu|varchar(50) |`door_attribXX`




Przykład:
```json
"atrybuty dokumentu": [
        {
          "nazwa": "NR_dokumentu_celnego",
          "wartosc": "2020/78123456"
        },
        {
          "nazwa": "Nr plomby",
          "wartosc": "123456"
        }
      ]
```


## Pozycje dokumentu
Reprezentuje pozycje dokumentu. 


| Pole | Wymagane | Opis | Typ danych| Pole WMS |
|--|--|--|--|--|
|LP|N | Numer linii - pole wykorzystywane w przypadku gdy systemy ERP w  komunikatach zwrotnych wymagają tej informacji np. SAP R3| int |`dori_lineNr`
|SSCC|N |Numer nośnika stosowany tylko w przypadku awiza dostawy|varchar(25) |`dori_SSCC`
|typ_palety|N |typ nośnika stosowany tylko w przypadku wypełniania pola SSCC|varchar(50) |`dori_luType`
|kod|T |kod porduktu|nvarchar(50) |`prd_code`
|nazwa|N |nazwa produktu (jeśli pole puste przy zakładaniu nowego produktu jako nazwa zostanie wykorzystany kod produktu)|nvarchar(250) |`prd_name`
|jednostka_miary|N |podstawowa jednostka miary. (jeśli pole puste przy zakładaniu nowego produktu jako nazwa zostań domyślna jednostka miary) |varchar(25) |`uom_code`
|ilosc_zamowiona|T |Ilość zamówiona w podstawowych jednostkach miary|decimal(18,6) |`dori_basicQuantity`
|atrybuty|N |Atrybuty nagłówka dokumentu Jeśli nie będzie zdefiniowanego atrybutu Status jakości wstawiona zostanie wartość domyślna dla statusu jakości|kolekcja




### Komunikat zwrotny
Zawiera to co komunikat wejściowy poszerzone o pola:


| Pole | Wymagane | Opis | Typ danych| Pole WMS |
|--|--|--|--|--|
OUT_ilosc_zrealizowana | N |[Tylko dla komunikatu zwrotnego] Zrealizowana ilość w jednostkach podstawowych |decimal(18,6)|`door_confirmedQuantity`




Przykład dokumet typu OUT:
```json
 "pozycje": [
      {
        "LP": "1",
        "SSCC": "159036345104856550",
        "kod": "kodtowaru",
        "nazwa": "nazwatowaru",
        "ean": "5091234567890",
        "jednostka_miary": "szt",
        "ilosc_zamowiona": "100",
        "atrybuty": [
            {
              "nazwa": "nr_LOT",
              "wartosc": "ABCD826"
            }
          ]        
      }
    ]
```
```XML
<pozycje>
    <LP>2</LP>
    <kod>HAO YOS314</kod>
    <nazwa>HAO YOS314</nazwa>
    <ean>5090987654321</ean>
    <jednostka_miary>szt</jednostka_miary>
    <ilosc_zamowiona>4</ilosc_zamowiona>
</pozycje>
```

## Kolekcja atrybuty


Sekcja niewymagana Kolekcja atrybuty może posiadać Max 10 obiektów. Atrybuty, których nazwa będzie niezgodna z nazwą w definicji atrybutów systemu WMS będą ignorowane.


| Pole | Wymagane | Opis | Typ danych| Pole WMS |
|--|--|--|--|--|
|nazwa|T | kod atrybutu z definicji atrybutów systemu WMS | nvarchar(50) |`pdef_code`
|Wartosc|T |wartość wstawiana do odpowiedniego atrybutu pozycji dokumentu|nvarchar(50) |`dori_attribXX` lub `dori_itemAttribXX`




## Przykłady

### JSON
Przykład zamówienia od klienta w formacie JSON
```json
{
  "naglowek": {
    "typ": "OUT",
    "zleceniodawca": "Zlec_1",
    "centrum_logistyczne": "CL_Lodz",
    "data_realizacji": "2021-07-10",
    "priorytet": "1",
    "nr_alternatywny_dokumentu": "ZAM/2021/62934",
    "opis": "Przykładowy opis do dokumentu",
    "kontrahent": {
      "kod": "7811903679",
      "nazwa": "Logsoft",
      "ulica": "Papiernicza 7e",
      "kod_pocztowy": "92-318",
      "miasto": "Łódź",
      "kraj": "PL"
    },
    "kurier": {
      "usluga": "DHL Standard",
      "COD": "156.23",
      "kwota_ubezpieczenia": "500",
      "telefon": "555-666-777",
      "email": "klient@kontakt.pl",
      "dodatkowe_info": "Uwaga szkło"
    },
    "atrybuty dokumentu": [
      {
        "nazwa": "Nr_dokumentu_celnego",
        "wartosc": "123456"
      }
    ]
  },
  "pozycje": [
    {
      "LP": "1",
      "kod": "PM YOS9",
      "ilosc_zamowiona": "1",
      "nazwa": "Pluszowy miś",
      "ean": "5091234567890",
      "jednostka_miary": "szt",
      "opakowania": {
        "waga": "12",
        "objetosc": "0.02",
        "jedn_podstawowych_w_kartonie": "10",
        "jedn_podstawowych_na_palecie": "100"
      },
      "atrybuty": [
        {
          "nazwa": "nr_LOT",
          "wartosc": "ABCD826"
        }
      ]
    },
    {
      "LP": "2",
      "kod": "GK A314",
      "nazwa": "Gumowa kaczuszka",
      "ean": "5090987654321",
      "jednostka_miary": "szt",
      "ilosc_zamowiona": "4"
    }
  ]
}
```
### XML
Przykład skonwertowany do formatu XML

```XML
<naglowek>
    <typ>OUT</typ>
    <zleceniodawca>Zlec_1</zleceniodawca>
    <centrum_logistyczne>CL_Lodz</centrum_logistyczne>
    <data_realizacji>2021-07-10</data_realizacji>
    <priorytet>1</priorytet>
    <nr_alternatywny_dokumentu>ZAM/2021/62934</nr_alternatywny_dokumentu>
    <opis>Przykładowy opis do dokumentu</opis>
    <kontrahent>
        <kod>7811903679</kod>
        <nazwa>Logsoft</nazwa>
        <ulica>Papiernicza 7e</ulica>
        <kod_pocztowy>92-318</kod_pocztowy>
        <miasto>Łódź</miasto>
        <kraj>PL</kraj>
    </kontrahent>
    <kurier>
        <usluga>DHL Standard</usluga>
        <COD>156.23</COD>
        <kwota_ubezpieczenia>500</kwota_ubezpieczenia>
        <telefon>555-666-777</telefon>
        <email>klient@kontakt.pl</email>
        <dodatkowe_info>Uwaga szkło</dodatkowe_info>
    </kurier>
    <atrybuty_dokumentu>
        <nazwa>Nr_dokumentu_celnego</nazwa>
        <wartosc>123456</wartosc>
    </atrybuty_dokumentu>
</naglowek>
<pozycje>
    <LP>1</LP>
    <kod>PM YOS9</kod>
    <ilosc_zamowiona>1</ilosc_zamowiona>
    <nazwa>Pluszowy miś</nazwa>
    <ean>5091234567890</ean>
    <jednostka_miary>szt</jednostka_miary>
    <opakowania>
        <waga>12</waga>
        <objetosc>0.02</objetosc>
        <jedn_podstawowych_w_kartonie>10</jedn_podstawowych_w_kartonie>
        <jedn_podstawowych_na_palecie>100</jedn_podstawowych_na_palecie>
    </opakowania>
    <atrybuty>
        <nazwa>nr_LOT</nazwa>
        <wartosc>ABCD826</wartosc>
    </atrybuty>
</pozycje>
<pozycje>
    <LP>2</LP>
    <kod>GK A314</kod>
    <nazwa>Gumowa kaczuszka</nazwa>
    <ean>5090987654321</ean>
    <jednostka_miary>szt</jednostka_miary>
    <ilosc_zamowiona>4</ilosc_zamowiona>
</pozycje>
```

