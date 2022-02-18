# Logsoft - Lite WMS 
# Interface wymiany danych, komunikat Dokument


Dokument zawiera ustandaryzowany format pozwalający wymieniać informacje związane z awizacją dostaw i zamówieniami od klientów.


## Obiekt naglowek


| Pole | Wymagane | Opis | Typ danych| Pole WMS |Od wersji 
|--|--|--|--|--|--|
|typ| T | określa typ dokumentu przyjmowane wartości **OUT** dla dokumentów wejściowych **IN** dla dokumentów wyjściowych | varchar(5)  
|zleceniodawca|T |kod zleceniodawcy z WMS Z punktu widzenia klienta (systemu ERP) mozna traktowac jako stałą)| nvarchar(25)  | `dord_code`
centrum_logistyczne|T |centrum logistyczne z WMS | nvarchar(25)  |`whc_code`
data_realizacji|N |żądana data realizacji|smalldatetime | `door_expectedCompletion`
priorytet|N |priorytet wartiosci od 0 - 89 priorytet zero jest najniższy. Priorytety 90-99 zarezerwowane jako specjalne do uzytku wewnętrznego| smallint |`door_priority`
nr_alternatywny_dokumentu|T |kod alternatywny zamówienia - kod zamówienia z systemu ERP klienta| nvarchar(50) | `door_alternativeCode`
opis|N|Opis do dokumentu|nvarchar(500) | `door_description`
kontrahent|T| Obiekt zawiera dane kontrahenta (klienta/dostawcy w zależności od typu dokumentu|Obiekt|
kurier|N| Obiekt zawiara dane potrzebne do wystawiania listu przewozowego z poziomu systemu WMS|Obiekt|
atrybuty_dokumentu|N| Atrybuty nagłówka dokumentu|Kolekcja|
produkty|N| Dane słownikowe produktów wykorzystywanych w zamówiniu |Kolekcja||1.1


### Komunikat zwrotny
Zawiera to co komunikat wejściowy poszerzone o pola:


| Pole | Wymagane | Opis | Typ danych| Pole WMS |Od wersji 
|--|--|--|--|--|--|
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

## Kolekcja atrybuty_dokumentu


Kolekcja atrybuty może posiadać Max 20 obiektów. Atrybuty, których nazwa będzie niezgodna z nazwą w definicji atrybutów systemu WMS będą ignorowane. Niewymagane


| Pole | Wymagane | Opis | Typ danych| Pole WMS |
|--|--|--|--|--|
|nazwa|T | kod atrybutu z definicji atrybutów systemu WMS | varchar(50) |`pdef_code`
|Wartosc|T |wartość wstawiana do odpowiedniego atrybutu nagłówka dokumentu|varchar(50) |`door_attribXX`




Przykład JSON: 
```json
  "atrybuty_dokumentu": {
    "atrybut": [
      {
        "nazwa": "Nr_dokumentu_celnego",
        "wartosc": "123456"
      },
      {
        "nazwa": "Nr_zamowienia",
        "wartosc": "hth/2020/829347"
      }
    ]
  }
```

Przykład XML:

```XML
        <atrybuty_dokumentu>
            <atrybut>
                <nazwa>Nr_dokumentu_celnego</nazwa>
                <wartosc>123456</wartosc>
            </atrybut>
            <atrybut>
                <nazwa>Nr_zamowienia</nazwa>
                <wartosc>hth/2020/829347</wartosc>
            </atrybut>
        </atrybuty_dokumentu>
```
## Produkty
| Pole | Wymagane | Opis | Typ danych| Pole WMS |Od wersji 
|--|--|--|--|--|--|
|nazwa|N |nazwa produktu (jeśli pole puste przy zakładaniu nowego produktu jako nazwa zostanie wykorzystany kod produktu). Nazwa nie musi byc unikatowa.|nvarchar(250) |`prd_name`
|EAN|N |kod kreskowy dla podstawowej jednostki miary (np EAN13) - kod nie musi byc unikatowy. W przypadku wystepienia innego kodu niż wczesniej dodany oryginalny wpis sie nie zaktulizuje, dodany zostanie nowy z bieżacym kodem |varchar(25) |`prd_name`
|opakowania|N |Struktura pakowania produktu sekcja w zasadzie powinna być wstawiana głownie w przypadku awizacji dostaw.|kolekcja
grupa_magazynowa|N |Powinna odpowiadac istniejącej grupie magazynowej w sytemie WMS|varchar(250)||1.1
|atrybuty_produktu|N |Atrybuty nagłówka dokumentu Jeśli nie będzie zdefiniowanego atrybutu Status jakości wstawiona zostanie wartość domyślna dla statusu jakości|kolekcja

### Opakowania

Struktura pakowania ***nie aktualizuje sie*** zakładana jest przy pierwszym dodaniu produktu. 

| Pole | Wymagane | Opis | Typ danych| Pole WMS |Od wersji 
|--|--|--|--|--|--|
|jednostka_miary|N |podstawowa jednostka miary. Kod jednostki miary pownien byc zgodny ze słownikiem w systemie WMS. (jeśli pole puste przy zakładaniu nowego produktu jako nazwa zostań domyślna jednostka miary) |varchar(25) |`uom_code`
|waga|N |Waga brutto dla podstawowej jednostki miary wyrazona w ***kg***| decimal(18,6) |pplv_weight
|objetosc|N |Objętość podstawowej jednostki miary wyrazona w ***m3***| decimal(18,6) |pplv_volume
|jedn_podstawowych_w_opakowaniu|N |ilość jednostek podatwowych w opakowaniu(kartonie). Tworzony jest nowy poziom struktury pakowania. Ten poziom opakowań oznaczany jest automatycznie jako opakowanie zbiorcze `pplv_calcAsOpa` | decimal(18,6) |
|jednostka_miary_opakowanie|N |jednostka miary dla opakowania (kartonu) |varchar(25) |`uom_code`|1.1
|jedn_podstawowych_na_palecie|N |ilość jednostek podatwowych na palecie. Tworzony jest nowy poziom struktury pakowania.Ten poziom opakowań oznaczany jest automatycznie jako paleta `pplv_isLoadUnit` | decimal(18,6) |
|jednostka_miary_paleta|N |jednostka miary dla palety |varchar(25) |`uom_code`|1.1
      

## Pozycja dokumentu
Reprezentuje pozycje dokumentu. 


| Pole | Wymagane | Opis | Typ danych| Pole WMS |Od wersji 
|--|--|--|--|--|--|
|LP|N | Numer linii - pole wykorzystywane w przypadku gdy systemy ERP w  komunikatach zwrotnych wymagają tej informacji np. SAP R3| int |`dori_lineNr`
|kod|T |kod porduktu jednoznacznie identyfikuje produkt musi byc unikatowy w obrębie jednego zleceniodawcy|nvarchar(50) |`prd_code`
|ilosc_zamowiona|T |Ilość zamówiona w podstawowych jednostkach miary|decimal(18,6) |`dori_basicQuantity`
|SSCC|N |Numer nośnika stosowany tylko w przypadku awiza dostawy **typ = IN** |varchar(25) |`dori_SSCC`
|typ_palety|N |typ nośnika stosowany tylko w przypadku awiza dostawy. Używany tylko w przypadku wypełniania pola SSCC|varchar(50) |`dori_luType`
|atrybuty_pozycji|N |Atrybuty pozycji dokumentu Jeśli nie będzie zdefiniowanego atrybutu Status jakości wstawiona zostanie wartość domyślna dla statusu jakości|kolekcja



### Komunikat zwrotny
Zawiera to co komunikat wejściowy poszerzone o pola:


| Pole | Wymagane | Opis | Typ danych| Pole WMS |
|--|--|--|--|--|
OUT_ilosc_zrealizowana | N |[Tylko dla komunikatu zwrotnego] Zrealizowana ilość w jednostkach podstawowych |decimal(18,6)|`door_confirmedQuantity`




Przykład dokumet typu OUT:
```json
{
  "dokument": {
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
      "atrybuty_dokumentu": {
        "atrybut": [
          {
            "nazwa": "Nr_dokumentu_celnego",
            "wartosc": "123456"
          },
          {
            "nazwa": "Nr_zamowienia",
            "wartosc": "hth/2020/829347"
          }
        ]
      }
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
        "atrybuty_pozycji": {
          "atrybut": [
            {
              "nazwa": "nr_LOT",
              "wartosc": "ABCD826"
            },
            {
              "nazwa": "Status_jakosci",
              "wartosc": "OK"
            }
          ]
        }
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
}
```
### XML
Przykład skonwertowany do formatu XML

```XML
<?xml version="1.0" encoding="UTF-8"?>
<dokument>
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
			<atrybut>
				<nazwa>Nr_dokumentu_celnego</nazwa>
				<wartosc>123456</wartosc>
			</atrybut>
			<atrybut>
				<nazwa>Nr_zamowienia</nazwa>
				<wartosc>hth/2020/829347</wartosc>
			</atrybut>
		</atrybuty_dokumentu>
	</naglowek>
	<produkty>
		<produkt>
			<kod>PM YOS9</kod>
			<nazwa>Pluszowy miś</nazwa>
			<ean>5091234567890</ean>
			<grupa_magazynowa>Zabawki</grupa_magazynowa>
			<opakowania>
				<jednostka_miary>szt</jednostka_miary>
				<waga>12</waga>
				<objetosc>0.02</objetosc>
				<jedn_podstawowych_w_opakowaniu>10</jedn_podstawowych_w_opakowaniu>
				<jednostka_miary_opakowanie>KRT</jednostka_miary_opakowanie>
				<jedn_podstawowych_na_palecie>100</jedn_podstawowych_na_palecie>
				<jednostka_miary_paleta>EP</jednostka_miary_paleta>
			</opakowania>
			<atrybuty_produktu>
				<atrybut>
					<nazwa>Producent</nazwa>
					<wartosc>Smyk</wartosc>
				</atrybut>
				<atrybut>
					<nazwa>Kolor</nazwa>
					<wartosc>Biały</wartosc>
				</atrybut>
			</atrybuty_produktu>
		</produkt>
		<produkt>
			<kod>GK A314</kod>
			<nazwa>Krem do rąk 250ml</nazwa>
			<ean>5090987654321</ean>
			<grupa_magazynowa>Kosmetyki</grupa_magazynowa>
			<opakowania>
				<jednostka_miary>szt</jednostka_miary>
			</opakowania>
		</produkt>
	</produkty>
	<pozycje>
		<pozycja>
			<LP>1</LP>
			<kod>PM YOS9</kod>
			<ilosc_zamowiona>1</ilosc_zamowiona>
		</pozycja>
		<pozycja>
			<LP>2</LP>
			<kod>GK A314</kod>
			<ilosc_zamowiona>4</ilosc_zamowiona>
			<atrybuty_pozycji>
				<atrybut>
					<nazwa>nr_LOT</nazwa>
					<wartosc>ABCD826</wartosc>
				</atrybut>
				<atrybut>
					<nazwa>Status_jakosci</nazwa>
					<wartosc>OK</wartosc>
				</atrybut>
			</atrybuty_pozycji>
		</pozycja>
	</pozycje>
</dokument>



```

