# Logsoft Lite WMS 
# Data exchange interface, message Document


A document contains a standardized format to exchange information related to purchase orders and customer orders.


## Header object


| Field | Required | Description | Data type| WMS field | From version 
|--|--|--|--|--|--|
| type | T | specifies the type of document **OUT** for input documents **IN** for output documents | varchar(5)  
orderer|T| From the point of view of the client (ERP system) it can be treated as a constant) | nvarchar(25) | `dord_code` 
logistics_center | T | logistics center with WMS | nvarchar(25) | `whc_code`
completion_date |T| requested completion date | smalldatetime | `door_expectedCompletion`
priority | N | value priority from 0 - 89 priority zero is the lowest. Priorities 90-99 reserved as special for internal use | smallint | `door_expectedCompletion`
document_alternative_code|T | alternative order code - order code from customer ERP| nvarchar(50) | `door_alternativeCode`
description|N|Description of the document|nvarchar(500) | `door_description`
firm |T| The object contains the firm data (customer/supplier) depending on the document type| Object
courier|N| The object contains the data needed to issue a waybill from the WMS.
attributes|N| Attributes of the document header|Collection|
products|N| Dictionary data of products used in the order | Collection||1.1

### Return message

Contains the same fields as the input message extended with fields:


| Field | Required | Description | Data type| WMS field | From version 
|--|--|--|--|--|--|
OUT_document_nr| T|[For feedback message only] document number in WMS|nvarchar(25) | `ddoc_code`.
OUT_date_creation| T| [For feedback only] date of creation/import of document|Datetime | `door_dateCreated`
OUT_date_closed |T| [For feedback only] date of document closure  | Datetime | `door_dateClosed`



## Firm object


The object companies (customer/supplier) depending on the document type. If the contractor's data does not exist in the firm dictionary the new firm will be automatically added to the dictionary.


 The field | Required | Description | Data type| WMS field |
--|--|--|--|--|
code|T | company code|nvarchar(50)
name|T | company name|nvarchar(100)
street|T | street name with street number|nvarchar(100) 
postal_code|T | country specific format |varchar(10)
city |T| town name | nvarchar(100) 
country|N | country code if empty inserted **PL** |varchar(2)

JSON example:
```json
"firm": {
                "code": "F00001",
                "name": "Logsoft",
                "street": "Paper 7e",
                "postal_code": "93-400",
                "city": "Łódź",
                "country": "PL".
            },
```

XML example:
```XML
<firm>.
	<code>F00001</code>.
	<name>Logsoft</name>.
	<street>Papiernicza 7e</street>.
	<postal_code>93-400</postal_code>.
	<city>Lodz</city>.
	<country>PL</country>.
<firm>
```

## Courier object


This section is not mandatory, applies only to orders of type ***OUT*** and is used only when a waybill is issued from the system


| Field | Required | Description | Type of data| WMS field |
|--|--|--|--|--|
|Service|T |DHL Standard|varchar(50) |`door_tr_service`.
|COD|N | Casch On Demand |decimal(18,6)| `door_tr_COD`.
| insurance_amount|N | declared amount for insurance (if collection amount must be >= COD)| decimal(18,6)| `door_tr_packageValue` 
|telephone|N | recipient's contact phone number|varchar(50) | `door_tr_contactPhone`
|email |N| recipient's contact email |varchar(50) | `door_tr_contact`
|additional_info |N| additional information for printing on the label |varchar(50) | `door_tr_description`

> to add the fields baselinkerID, Inpost_gabarit, Inpost_paczkomat

### Return message
Contains the same as the input message extended with fields:


| Field | Required | Description | Type of data| WMS field |
|--|--|--|--|--|
|OUT_nr_log_trucking|N |[Only for return message] number of the generated tracking number |nvarchar(50) |`door_tr_truckingNumber`.




JSON example:
```json
    "courier": {
                "service": "DHL Standard",
                "COD": "156.23",
                "insurance_amount": "160",
                "phone": "555-666-777",
                "email": "klient@kontakt.pl",
                "additional_info": "additional_info",
             },
```

XML example:
```XML
<courier>
	<service>DHL Standard</service>.
	<CODE>156.23</COD>.
	<insurance_amount>500</insurance_amount>
	<telefon>555-666-777</telefon>
	<email>klient@kontakt.pl</email>.
	<additional_info>Note glass<additional_info>
</courier>
```

## Document Attribute Collection


Attribute collection can have Max 20 objects. Attributes whose name is inconsistent with the name in the WMS attribute definition will be ignored. Not Required


| Field | Required | Description | Data type| WMS field |
|--|--|--|--|--|
| name | T | attribute code from the WMS attribute definition | varchar(50) |`pdef_code`
| value|T | value inserted in corresponding attribute of document header | varchar(50) | `door_attribXX`



JSON example: 
```json
  "document_attribute": {
    "attribute": [
      {
        "name":"customs_number",
        "value": "123456"
      },
      {
        "name": "order_number",
        "value": "hth/2020/829347"
      }
    ]
  }
```

XML example:

```XML
        <document_attribute>.
            <attribute>.
                <name> customs_number</name>.
                <value>123456</value>.
            </attribute>.
            <attribute>.
                <name> order_number</name>.
                <value>hth/2020/829347</value>
            </attribute>.
        </document_attribute>.
```

## Products

Products dictionary

| Field | Required | Description | Data type| WMS field | From version 
|--|--|--|--|--|--|
|code|T |product code uniquely identifies a product and must be unique within a single ordere |nvarchar(50) |`prd_code`.
|name|N|(if the field is empty the product code will be used as the name when creating a new product). The name does not have to be unique.
|EAN|N|barcode for the basic unit of measurement (e.g. EAN13) - the barcode does not have to be unique. In case of a different code than the one previously added the original entry will not update and a new one will be added with the current code |varchar(25) |`prdb_code`.
packaging_structure|N|This section should be used mainly in case of purchase orders.|collection
werehouse_ group|N | Should correspond to an existing warehouse group in the WMS|varchar(250)||1.1
product_attribute|N|If no quality status attribute is defined (`door_attrib9`) , a default value for quality status is inserted|collection



### Packaging structure 

The packaging structure ***does not update*** is assumed when a product is first added. 

| Field | Required | Description | Data type| WMS field | From version 
|--|--|--|--|--|--|
| unit_of_measure|T | basic unit of measure. The unit of measure code should be consistent with the dictionary in WMS.|varchar(25) |`uom_code` 
weight |N|The gross weight for the primary unit of measure expressed in ***kg***| decimal(18,6) |`pplv_weight`
| volume|N | Volume of the basic measurement unit expressed in ***m3***| decimal(18,6) |`pplv_volume`
units_in_package|N|The number of basic units in a package (carton). A new level of packing structure is created. This packing level is automatically marked as a `pplv_calcAsOpa` packing level | decimal(18,6) | 
| unit_of_package|N| unit of measure for packaging (carton) Required if field "units_in_package" is completed |varchar(25) | `uom_code`|1.1
|units_on_pallet |N | The number of basic units on the pallet. A new packing structure level is created.This packing level is automatically marked as a pallet `pplv_isLoadUnit` | decimal(18,6) | 
| unit_of_pallet|N| unit of measure for pallet  Required if field "units_on_pallet" is completed |varchar(25) | `uom_code`|1.1


## Document item
Represents a document item. 


| Field | Required | Description | Data type | WMS field | From version 
|--|--|--|--|--|--|
|LN|N | Line number - field used when ERP systems in feedback messages require this information e.g. SAP R3| int |`dori_lineNr`.
|code |T | Product code uniquely identifies the product and must be unique within one customer |nvarchar(50) |`prd_code`
|ordered_quantity|T | quantity ordered in basic units of measurement |decimal(18,6) | `dori_basicQuantity`
SSCC|N|Pallet number Can be optionally used only in Purchase order **type = IN** |varchar(25) |`dori_SSCC` 
| pallet_type |N| pallet type only used for delivery advice. Used only when completing the SSCC|varchar(50) |`dori_luType` 
attribute |N | Document Item atributes. If no Quality status attribute is defined a default value for Quality status is inserted.|collection

### Return message
Contains the same information as the input message extended with fields:


| Field | Required | Description | Data type | WMS field |
|--|--|--|--|--|
OUT_quantity_confirmed | N |[Only for return message] Confirmed quantity in basic units |decimal(18,6)|`door_confirmedQuantity`.



Przykład dokumet typu OUT:
```json
{
  "document": {
    "header": {
      "type": "OUT",
      "orderer": "Zlec_1",
      "logistics_center": "CL_Lodz",
      "completion_date": "2021-07-10",
      "priority": "1",
      "document_alternative_code": "ZAM/2021/62934",
      "description": "Sample description for the document",
      "firm": {
        "code": "7811903679",
        "name": "Logsoft",
        "street": "Papiernicza 7e",
        "postal_code": "92-318",
        "city": "Łódź",
        "country": "PL"
      },
      "courier": {
        "service": "DHL Standard",
        "COD": "156.23",
        "insurance_amount": "500",
        "telephone": "555-666-777",
        "email": "klient@kontakt.pl",
        "additional_info": "Sample additional info for courier"
      },
      "document_attributes": {
        "attribute": [
          {
            "name": "Custom_document_No",
            "value": "123456"
          },
          {
            "name": "Order_nr",
            "value": "hth/2020/829347"
          }
        ]
      }
    },
    "products": {
      "product": [
        {
          "code": "PM YOS9",
          "name": "Teddy bear",
          "ean": "5091234567890",
          "warehouse_group": "Toys",
          "packaging_structure": {
            "unit_of_measure": "szt",
            "weight": "12",
            "volume": "0.02",
            "units_in_package": "10",
            "unit_of_package": "KRT",
            "units_on_pallet": "100",
            "unit_of_pallet": "EP"
          },
          "products_attributes": {
            "attribute": [
              {
                "name": "Producer",
                "value": "LOBITO"
              },
              {
                "name": "Colour",
                "value": "White"
              }
            ]
          }
        },
        {
          "code": "GK A314",
          "name": "Hand cream 250ml",
          "ean": "5090987654321",
          "grupa_magazynowa": "Cosmetics",
          "packaging_structure": {
            "unit_of_measure": "szt"
          }
        }
      ]
    },
    "items": {
      "item": [
        {
          "LN": "1",
          "code": "PM YOS9",
          "ordered_quantity": "1"
        },
        {
          "LN": "2",
          "code": "GK A314",
          "ordered_quantity": "4",
          "item_attributes": {
            "attribute": [
              {
                "name": "nr_LOT",
                "value": "ABCD826"
              },
              {
                "name": "Quality_status",
                "value": "OK"
              }
            ]
          }
        }
      ]
    }
  }
}
```
### XML
Przykład skonwertowany do formatu XML

```XML
<?xml version="1.0" encoding="UTF-8"?>
<document>
	<header>
		<type>OUT</type>
		<orderer>Zlec_1</orderer>
		<logistics_center>CL_Lodz</logistics_center>
		<completion_date>2021-07-10</completion_date>
		<priority>1</priority>
		<document_alternative_code>ZAM/2021/62934</document_alternative_code>
		<description>Sample description for the document</description>
		<firm>
			<code>7811903679</code>
			<name>Logsoft</name>
			<street>Papiernicza 7e</street>
			<postal_code>92-318</postal_code>
			<city>Łódź</city>
			<country>PL</country>
		</firm>
		<courier>
			<service>DHL Standard</service>
			<COD>156.23</COD>
			<insurance_amount>500</insurance_amount>
			<telephone>555-666-777</telephone>
			<email>klient@kontakt.pl</email>
			<additional_info>Sample additional info for courier</additional_info>
		</courier>
		<document_attributes>
			<attribute>
				<name>Custom_document_No</name>
				<value>123456</value>
			</attribute>
			<attribute>
				<name>Order_nr</name>
				<value>hth/2020/829347</value>
			</attribute>
		</document_attributes>
	</header>
	<products>
		<product>
			<code>PM YOS9</code>
			<name>Teddy bear</name>
			<ean>5091234567890</ean>
			<warehouse_group>Toys</warehouse_group>
			<packaging_structure>
				<unit_of_measure>szt</unit_of_measure>
				<weight>12</weight>
				<volume>0.02</volume>
				<units_in_package>10</units_in_package>
				<unit_of_package>KRT</unit_of_package>
				<units_on_pallet>100</units_on_pallet>
				<unit_of_pallet>EP</unit_of_pallet>
			</packaging_structure>
			<products_attributes>
				<attribute>
					<name>Producer</name>
					<value>LOBITO</value>
				</attribute>
				<attribute>
					<name>Colour</name>
					<value>White</value>
				</attribute>
			</products_attributes>
		</product>
		<product>
			<code>GK A314</code>
			<name>Hand cream 250ml</name>
			<ean>5090987654321</ean>
			<grupa_magazynowa>Cosmetics</grupa_magazynowa>
			<packaging_structure>
				<unit_of_measure>szt</unit_of_measure>
			</packaging_structure>
		</product>
	</products>
	<items>
		<item>
			<LN>1</LN>
			<code>PM YOS9</code>
			<ordered_quantity>1</ordered_quantity>
		</item>
		<item>
			<LN>2</LN>
			<code>GK A314</code>
			<ordered_quantity>4</ordered_quantity>
			<item_attributes>
				<attribute>
					<name>nr_LOT</name>
					<value>ABCD826</value>
				</attribute>
				<attribute>
					<name>Quality_status</name>
					<value>OK</value>
				</attribute>
			</item_attributes>
		</item>
	</items>
</document>



```

