# <img src="nyckeldb-logo.png" alt="N" id="nyckeldb-logo">yckelDB Documentation

#### NyckelDB Version 0.8.0

##### November 16, 2022

NyckelDB is a highly structured JavaScript data store. Any data that you can visualize as a table or spreadsheet can 
be stored in a NyckelDB object. NyckelDB data is [sortable, filterable](#sort-shuffle-and-filter), 
[searchable](#search), [syncable and shareable](#sync-and-share). All data inputs are validated according to data 
type, and data can be imported or exported as JSON or CSV (coming soon).

## Table of Contents

* [Overview](#overview)
* [Features](#features)
  * [Sort, Shuffle and Filter](#sort-shuffle-and-filter)
  * [Search](#search)
  * [Search Suggestions](#search-suggestions)
  * [Sync and Share](#sync-and-share)
  * [Data Validation](#data-validation)
  * [Formulas](#formulas)
* [Types](#types)
  * [String types](#string-types)
  * [Numeric types](#numeric-types)
  * [Boolean types](#boolean-types)
* [Getting Started](#getting-started)
  * [Setting up a new NyckelDB Object](#setting-up-a-new-nyckeldb-object)
  * [The Options Parameter](#the-options-parameter)
    * [Importing Data](#importing-data)
    * [Custom Properties](#custom-properties)
* [Full API Documentation](#full-api-documentation)
* [Dependencies](#dependencies)

# <a id="overview"></a> Overview

Nyckel (nick-ale) is a Swedish word that means "key".

The basic concept of NyckelDB is to create a portable JavaScript client-side data store with data type validation.

What does that mean?

- Client-side: Because NyckelDB is written in JavaScript (Typescript actually, and transpiled to JavaScript), it 
runs on your local device in a web browser, not on a server.

- Portable: A database typically means a single 'base' for your data, usually in the cloud. All API calls reference
 that <i>single source of truth</i>. NyckelDB aims to break that model. You may have a copy of your NyckelDB on your
  phone, one on your PC, and another on your friend's PC. When you want to update one copy from another copy, you 
  simply <i>synchronize</i> them. This enables you to use your data offline, and synchronize it when you connect 
  again. [See Syncing and sharing your data](#sync-and-share).

- Data type validation: NyckelDB ensures that data that you write to it is correct before it gets written, according
 to the <i>type</i> of data that you specified. This helps to elliminate or correct many common input mistakes. 
 [See data types](#types).

NyckelDB can be visualized as having a table-like structure (Figure 1), with table headers, and table rows, although 
the actual structure is JSON (Figure 2).

|           |COLUMN 0        |COLUMN 1  |COLUMN 2  |COLUMN 3      |
|:---------:|:--------------:|----------|----------|--------------|
|**HEADERS**|**id**          |**Name**  |**Age**   |**Is Awesome**|
|**TYPES**  |**uniqueString**|**string**|**number**|**boolean**   |
|**ROW 0**  |**aaa**         |Bob       |42        |false         |
|**ROW 1**  |**aab**         |Jill      |21        |true          |
|**ROW 2**  |**aac**         |Anne      |88        |true          |

*Figure 1: Imagine NyckelDB as having a table-like structure*

```javascript
{
    title: "Table Name",
    created: 1234567,
    lastModified: 1234567,
    version: "0.8.0_1.2",
    ids: {
        aaa: [/* lastModified info */],
        aab: [/* lastModified info */],
        aac: [/* lastModified info */]
    },
    columns: {
        headers: ["id", "Name", "Age", "Is_Awesome"],
        meta: {/* column metadata */}
    },
    table: [
        ["aaa", "Bob", 42, false],
        ["aab", "Jill", 21, true],
        ["aac", "Anne", 88, true]
    ],
    properties: {/* custom properties */}
}

``` 
*Figure 2: What a NyckelDB table really looks like*

The data store can run along side your other JavaScript code by adding a \<script\> tag directly to your HTML 
document, include it as a Javascript module, or run it in a background task in a Web Worker (which is recommended).

NyckelDB stores data temporarily in the browser using indexed-db. 
For more persistant storage, you can export the data from NyckelDB as a JSON file and send it to your own cloud 
solution. Synchronisation is handled client-side when you import that JSON file back into the NyckelDB.

LZString compression is used so you can fit many MB of data in the very limited localStorage space that some 
browsers allow, and reduce bandwidth if uploading to a cloud service provider.

All data written to the data store is automatically validated before being written, taking care of much of your form 
validation work for you such as names, email addresses and phone numbers. 

# <a id="features"></a> Features

Other than the basic read/write functionality that would be expected in any type of data storage solution, NyckelDB 
also contains the following features:

### <a id="sort-shuffle-and-filter"></a> Sort, Shuffle and Filter

Like a table, data can be sorted alphabetically/numerically by any column in the table, or conversely, shuffled to 
be completely random &ndash; useful if you want to create a playlist, or stack of flashcards.

The table can also be filtered by column, based on matching a given regular expression.

### <a id="search"></a> Search

NyckelDB contains a basic search engine. Data is search indexed for fast search results based on whole, and multiple 
words, returning results in order of best match. 
Complex searches can be created with the + and - operators, which correspond to AND and NOT respectively.
Fuzzy matching can be used in some cases to return near matches, for example, common names or spelling mistakes.

You can add your own "dictionary" JSON file to refine your search experience. For example:

> ```javascript
> { 
>   "searchterm": ["equivelent searchterm", "another one"],
>   "something": ["something else"]
>
> }
> ```
> In this case if you searched for `searchterm`, you would instead get the search results for `equivelent searchterm` and `another one`. If you searched for `something`, you would get search results for `something else`.

All searches are case insensitive; searches are done in lowercase only, therefore the dictionary file need only be 
in lowercase.

### <a id="search-suggestions"></a> Search Suggestions

NyckelDB can aid, or hint to a user what they might be able to search for by returning search suggestions based on 
partial word matches. This is surprisingly fast and can be used to create an as-you-type experience to fill in a 
search box and/or the dropdown list under the search box. Recent successful searches are cached and returned at the 
top of the next search suggestion, giving you search history as well (though this cache isn't saved anywhere so will 
be gone the next time you access the app or webpage).

### <a id="sync-and-share"></a>Sync and Share

Data can be exported as a JSON file which can be saved or transferred somewhere and reimported without worrying 
about overwriting more recent data because all data (in every single cell) is stored with accompanying LastModified 
attributes.

This means that data can be synchronized between local storage and cloud storage, devices, or even people.
Conflict resolution follows the assumption that the most recent data in each cell is the data that you want to keep.

Sharing data securely based on a Diffie-Hellman key exchange system coming soon...

### <a id="data-validation"></a> Data Validation

Data that you input into a NyckelDB object is validated before it is saved, at a minimum, by verifying that it is 
either a String, Number, or Boolean value. All data *must* fall into one of these three categories. Other types of 
data, such as JavaScript Objects, Arrays or the values 'null' and 'undefined' are not allowed in a NyckelDB and will 
throw an error.

Within these 3 general types of data are many more subtypes. Additional data validation, and sometimes error 
correction can be done by specifying a subtype, i.e. streetAddress is a subtype of String. Some subtypes fall into 
more than one category such as *postalZipCode* which can be a String or a Number. [See Types below](#types) for a 
complete list of all the different subtypes.

If you want the most flexibility you can specify *any*, which will allow any type of data to be saved to that column 
in the table, as long as it is a String, Number or Boolean value (though there is no additional validation).

The data type for each column in the table is best specified when the table is created, but in some cases can be 
changed afterwards, if it doesn't cause a conflict with the existing data in the table.

### <a id="formulas"></a> Formulas

You can set a formula on a particular column to pre-calculate its values. Formulas are one line expressions that 
follow the following rules:

* Start with a function name, followed by parentheses 
* Put function parameters inside the parentheses, separated by commas
* Parameters inside (single or double) quotes are literal values
* Parameters without quotes starting with a $ refer to a custom property name
* Parameters without quotes with no $ refer to a column name
* Dots join functions together
* Literal values (in quotes) which contain a quote symbol (a ' or a ") can be 'escaped' by placing a backslash before it, so this is OK in single quotes: `'He said, "How\'s it goin\'?"'` or like this in double `"He said, \"How's it goin'?\""`

> Example: `GET_VALUE(LastName).TO_UPPERCASE().JOIN(", ", FirstName, " ", MiddleName).TRIM()`
> gets the value in column 'LastName', converts it to ALL CAPS, joins it with a ", " (comma space), the value in column
> 'FirstName', " " (space), the value in column 'MiddleName', and then trims out any extra spaces to build something like
> `SMITH, William John`

Available functions include:

| Function Name		| Alternate Names	| Description						| Parameters
|-------------------|-------------------|-----------------------------------|------------
| ADD				| PLUS				| Add 2 numbers together			| Num1, Num2
| DIVIDE			|					| Divide Num1 by Num2				| Num1, Num2
| GET_VALUE			| VALUE, VAL		| Retrieve a value					| ColName or $PropName
| HAS_VALUE			| IS_NOT_EMPTY		| Check whether a value has been assigned| ColName or $PropName
| IF				|					| Check whether a condition is true	| Condition, if_True, if_False
| IS_BOOLEAN		|					| Check if a value is a true or false (boolean) value | Value
| IS_EQUAL			|					| Check if 2 values are the same	| Value1, Value2
| IS_GREATER_THAN	| IS_GT				| Check if Num1 is greater than Num2 | Num1, Num2
| IS_LESS_THAN		| IS_LT				| Check if Num1 is less than Num2	| Num1, Num2
| IS_NUMBER			| IS_NUM			| Check if a value is a number		| ColName or $PropName
| IS_STRING			| IS_STR, IS_TEXT	| Check if a value is a text value	| ColName or $PropName
| JOIN				|					| Join text values together 		| Any # of ColNames, $PropNames or "text"
| MINUS				| SUBTRACT			| Subtract Num2 from Num1			| Num1, Num2
| MULTIPLY			| TIMES				| Multply Num1 by Num2				| Num1, Num2
| TO_LOWERCASE		| TO_LOWER, TO_LC	| Convert text to lowercase letters	| 
| TO_TITLECASE		| TO_TITLE, TO_TC	| Capitalise text					|
| TO_UPPERCASE		| TO_UPPER, TO_UC	| Convert text to all uppercase letters|
| TRIM				|					| Remove extra spaces from text		|
| More to come....

# <a id="types"></a> Types 

The type of data that a column expects can be specified when creating a new table or adding a new column to a table. 
Any data that you try to set on a cell will be first validated according to what type of data the column accepts. 
In some cases the data will be coerced into fitting into the column type, for example converting the string value 
"4" into the number 4, or an error will be returned.

Some of the types are actually validated exactly the same, for example **string** and **multilineString**, but you 
can use a specific type to indicate what sort of input field to provide to the user for that particular value. A 
**string** value might use a basic `input` of `type="text"`, whereas a **multilineString** may work better as a 
`textarea` element. Likewise a **color** type would work good with a `input` of `type="color"` and **date** with a 
`type="date"` input.

The following types can be specified:

### <a id="string-types"></a>String types

* **any** is the default type if no type is specified. Use it if you want to accept input data that could be String, Number, or Boolean values
* **cityCounty** for a county, city or town name
* **color** a hex or rgba color string
* **country** for a country or nation
* **email** verifies that the value could be a valid email address
* **familyName** for a last name
* **formattedAddress** for full home/mail address
* **geoLocation** for complete formatted GPS position in decimal coordinates
* **givenName** for a first or middle name
* **icon** an icon class name
* **mailAddress** checks that a string contains both letters and numbers, and corrects some common mailing address input errors
* **multilineString** accepts text with carriage returns
* **password** checks for password strength and that it follows character use requirements, then saves a salted SHA256 hash of the password. Retrieval of the password requires that the original password be given and only returns true or false depending on whether they match.
* **phoneNumber** verifies that the value could be a valid local or long distance phone number, and formats it accordingly
* **postalZipCode** verifies that the value could be a Canadian postalCode, or American zipCode
* **provinceStateRegion** for a province, state or region
* **streetAddress** checks that a string contains both letters and numbers, and corrects some common address input errors
* **string** accepts any text value
* **textSelect** use with textOptions column option to set the specific text to accept
* **textSuggest** use with textOptions column option to set the specific text to suggest
* **uniqueString** can be used for things like ids or usernames. It wont accept a value if it already exists in that column of the table.
* **url** for a web address

### <a id="numeric-types"></a> Numeric types

* **any** see [String types: any](#string-types)
* **date** for brevity, stores dates as the number of milliseconds since the database creation. Retrieved values are returned in UNIX time - the number of milliseconds since 00:00:00 Jan 1, 1970 UTC
* **integer** checks that it is a number, and rounds numbers to the nearest integer value
* **latitude** checks that it is a valid latitudinal decimal coordinate
* **longitude** checks that it is a valid longitudinal decimal coordinate
* **negInteger** checks that it is a negative number, including 0, and rounds to the nearest integer value
* **number** accepts any numeric value, whether positive, negative, or decimal, up to the JavaScript physical size limit
* **password** see [String types: password](#string-types)
* **phoneNumber** see [String types: phoneNumber](#string-types)
* **posInteger** checks that is a positive number, including 0, and rounds to the nearest integer value
* **postalZipCode** see [String types: postalZipCode](#string-types)
* **range** accepts a number between 2 given values. Set rangeMin and rangeMax on the column options to use range

### <a id="boolean-types"></a> Boolean types

* **any** see [String types: any](#string-types)
* **boolean** accepts only true or false
* **checkbox** accepts only true or false

> Note: not all types listed are currently fully implementated in this Beta version, specifically uniqueString, password, country, integer, posInteger, negInteger, url, range, and date

# <a id="getting-started"></a> Getting started

To start using NyckelDB, download a copy of `nyckelDB.min.js`, `base64.min.js`, `storage.js`, `Lawnchair.js`, and the Lawnchair adaptors that you want to use (`dom.js`, `indexed-db.js` for starters), and insert them into your html file with script tags, loading them in this order:
```html
    <script type="text/javascript" src="scripts/base64.min.js"></script>
    <script type="text/javascript" src="scripts/Lawnchair.js"></script>
    <script type="text/javascript" src="scripts/adapters/indexed-db.js"></script>
    <script type="text/javascript" src="scripts/adapters/dom.js"></script>
    <script type="text/javascript" src="scripts/storage.js"></script>
    <script type="text/javascript" src="scripts/nyckelDB.min.js"></script>
```

### <a id="setting-up-a-new-nyckeldb-object"></a> Setting up a new NyckelDB Object

Setting up a new NyckelDB object is as simple as calling the constructor using the "new" keyword and giving it a 
unique app ID, and a title. The app ID can be the name of the app or any other identifier to distinguish between 
other web apps that could be distributed from the same web domain. The title identifies this particular NyckelDB 
data store. After the NyckelDB data store is created, initialize it by calling the "init" function and passing it 
the required table parameters: "headers", and "types". Optional "customProperties" and "importData" can also be 
passed to the table on setup. A callback function can return a boolean value indicating whether or not the NyckelDB 
data store was successfully created and initiated, and any error messages. Inside the callback function, 'this' 
refers to the NyckelDB instance.

> [See more details about the NyckelDB API below](#full-api-documentation)

```javascript
var appId = "Company App", //new in version 0.7.0
    title = "Accounting Spreadsheet",
    headers = ["Month", "Monthly Income", "Monthly Expenses", "Balance"],
    types = ["date", "posInteger", "negInteger", "integer"],
    options = {
        //see 'The Options Parameter' below
    },
    callback = function (success, errors) { 
        if (success) {
            console.log(this.getTitle()); //returns "Accounting Spreadsheet"
            //update UI to show newly created NyckelDB here
        } else console.log(errors);
};
var myTable = new APP.NyckelDB(appId, title);
myTable.init(headers, types, options, callback);
myTable.getLength(); //returns 0
```

There is quite a bit of flexibility in how you want to go about setting up a new NyckelDB object. Headers can be 
passed as an Array, and types as a corresponding Array (as shown above), or you can pass types as an Object in the 
form:

```javascript
var types = { 
    "Month": "date",
    "Monthly Income": "posInteger",
    "Monthly Expenses": "negInteger",
    "Balance": "integer"
}
```

which might be an easier way to manage your code, especially if you have a large number of columns. 

If the order of the columns in your table is unimportant, you can pass this Object directly as the headers 
themselves in the first parameter instead of an Array, and the second parameter as null.

> __Q: When should you use an Array for your column headers? When would order matter?__
> 
> A: If you want to display the data from an entire row all at once on the screen in a specific order, 
or if you receive your data as an Array and want to pass it directly into a newly created row in your 
table at once. You can work around this by reordering the output afterwards of course, but better to be aware of 
this in the beginning.

Finally, additional metadata can be set such as whether a column should be indexed for searching, what it's initial 
value should be if nothing is specified, or even a [formula](#formulas), by passing Types in like this:

```javascript
var types = { 
    "Month": { 
        "type": "string", 
        "search": true,
        "initialValue": "January"
    },
    "Monthly Income": {
       "type": "posInteger",
        "search": false,
        "initialValue": 2000
    },
    "Monthly Expenses": {
        "type": "negInteger",
        "search": false
    },
    "Balance": {
        "type": "integer",
        "formula": "SUBTRACT('Monthly Income','Monthly Expenses')"
    }
}
```

### <a id="the-options-parameter"></a> The Options Parameter

The 'options' parameter accepts an Object. It may contain data to import immediately after the database is created 
in the form of a CSV file (string), or JSON object, and custom properties that you would like to add to your table.

```javascript
var options = {
    "importData": nyckelDBTable,
    "importJSON": json,
    "importCSV": string,
    "customProperties":{
        "FinalBalance": {
            "type": "number",
            "initialValue": 10000
        },
        "ICanBuyMyGroceriesThisMonth": {
            "type": "boolean",
            "initialValue": true
        }
    }	
};
```

##### <a id="importing-data"></a> Importing Data

CSV, or JSON data can be imported into the table in the 'options' parameter in one of 3 forms:

```javascript
//table data that was generated by NyckelDB sync or exportJSON functions
"importData": nyckelDBTable, 

//json data that needs to be parsed
"importJSON": json, 

//or CSV data that needs to be converted to JSON and then parsed
"importCSV": string, 
```

##### <a id="custom-properties"></a> Custom Properties

Custom properties can be used for data that doesn't quite fit into the "table" model, such as maybe some metadata 
that applies to the entire database. The existence of a custom property must be initiated when the NyckelDB object 
is created, and must be given an initial value. Just like table columns, custom properties can be given a data type 
as well, or they will default to the type `"any"`.

```javascript
customProperties:{
    //name the custom property what ever you want
    "FinalBalance": {
        "type": "number",
        "initialValue": 10000
    },
    "ICanBuyMyGroceriesThisMonth": {
        "type": "boolean",
        "initialValue": true
    }
}	
```

# <a id="full-api-documentation"></a> Full API Documentation

[https://ggoodkey.github.io/nyckeldb/0.8.0/NyckelDB.html](https://ggoodkey.github.io/nyckeldb/0.8.0/NyckelDB.html)

# <a id="dependencies"></a> Dependencies

* base64.js
* storage.js
* Lawnchair.js
