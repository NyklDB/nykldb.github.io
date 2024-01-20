# <img src="nykldb-logo.png" alt="N" id="nykldb-logo">yklDB Documentation

#### NyklDB Version 0.9.0

##### August 27, 2023

NyklDB is a document-oriented database that stores data as JSON documents. Data in a NyklDB has a [table-like structure](#database-structure), with columns and rows, therefore, any data that you can visualize as a table or spreadsheet can be stored in a NyklDB. 

It is designed to be fast, scaleable and easy to use. NyklDB contains a [basic search engine](#search) that allows you to perform complex searches, and it can execute mathimatical, boolean, and text based [formulas](#formulas) to preform transformations and calculations on your data.

NyklDB data is also [sortable, filterable](#sort-shuffle-and-filter), [syncable and shareable](#sync-and-share). All data inputs are [validated](#data-validation) according to [data type](#types), and data can be [imported](#importing-data) or exported as JSON documents.

## Table of Contents

* [Overview](#overview)
  * [Database Structure](#database-structure)
* [Features](#features)
  * [Sort, Shuffle and Filter](#sort-shuffle-and-filter)
  * [Search](#search)
  * [Search Suggestions](#search-suggestions)
  * [Sync and Share](#sync-and-share)
  * [Data Validation](#data-validation)
  * [Formulas](#formulas)
* [Types](#types)
  * [String Types](#string-types)
  * [Numeric Types](#numeric-types)
  * [Boolean Types](#boolean-types)
* [Getting Started](#getting-started)
  * [JavaScript Installation](#javascript-installation)
  * [Setting up a new NyklDB](#setting-up-a-new-nykldb)
  * [The Options Parameter](#the-options-parameter)
    * [Importing Data](#importing-data)
    * [Custom Properties](#custom-properties)
* [Dependencies](#dependencies)

# <a id="overview"></a> Overview

Nykl (pronounced *nickle*) comes from a Swedish word, *nyckel*, which means *key*.

NyklDB is a client-side document-oriented data store with data type validation.

What does that mean?

- Client-side: NyklDB runs in a web browser, not on a server.

- Document-oriented: Rather than having one "single source of truth" on a server, with NyklDB you may have a copy of your data on your phone, on your PC, and another copy on your friend's PC, etc. When you want to update one copy from another copy, you simply [*synchronize*](#sync-and-share) them. This enables you to use your data offline, and synchronize it when you connect again. NyklDB uses the JSON file format to exchange data.

- Data type validation: NyklDB ensures that data that you write to it is correct before it gets written, according
 to the [*type*](#types) of data that you specified. This helps to eliminate or correct many common input mistakes, and takes care of much of your form validation work for you, such as for names, email addresses and phone numbers.

### <a id="database-structure"></a> Database Structure

Unlike many document-oriented databases, NyklDB follows a very strict data structure. NyklDB can be visualized as having a table-like structure (Figure 1), with table headers, columns and rows, although 
the actual data is written in JSON (Figure 2).

*Figure 1: Imagine NyklDB as having a table-like structure*

|id  |Name (string)|Age (number)|Is Awesome (boolean)|
|:--:|-------------|------------|--------------------|
|AAA |Bob          |42          |false               |
|AAB |Jill         |21          |true                |
|AAC |Anne         |88          |true                |

*Figure 2: What a NyklDB table really looks like*

```json
{
    "title": "Table_Name",
    "created": 3064452,
    "lastModified": 3064455,
    "deleted": false,
    "version": "0.9.0",
    "ids": { 
        "AAA": [/* lastModified info */],
        "AAB": [/* lastModified info */],
        "AAC": [/* lastModified info */]
    },
    "columns": {
        "headers": ["id", "Name", "Age", "Is_Awesome"],
        "meta": {/* column metadata */
            "Name": {
                "type": ["string", 0],
                "timestamp": [0, 0],
                "searchable": [true, 0],
                "initialValue": ["", 0]
            },
            "Age": {
                "type": ["number", 0],
                "timestamp": [0, 0],
                "initialValue": [0, 0]
            },
            "Is_Awesome": {
                "type": ["boolean", 0],
                "timestamp": [0, 0],
                "initialValue": [false, 0],
                "exportAs": ["Is Awesome", 0]
            }
        }
    },
    "table": [
        ["AAA", "Bob", 42, false],
        ["AAB", "Jill", 21, true],
        ["AAC", "Anne", 88, true]
    ],
    "properties": {/* table properties */}
}

``` 

NyklDB stores data temporarily in the browser using indexed-db. 
For more persistant storage, you can export the data from NyklDB as a JSON file and send it to your own cloud 
solution. Synchronisation is handled client-side when you import that JSON file back into NyklDB.

LZString compression is used so you can fit many MB of data in the very limited indexed-db space that some 
browsers allow, and reduce bandwidth if you are uploading it to a cloud service provider.

# <a id="features"></a> Features

Other than the basic read/write functionality that would be expected in any type of data storage solution, NyklDB 
also contains the following features:

### <a id="sort-shuffle-and-filter"></a> Sort, Shuffle and Filter

Like a table, data can be sorted alphabetically/numerically by any column in the table, or conversely, shuffled to 
be completely random &ndash; useful if you want to create a playlist, or stack of flashcards.

The table can also be filtered by column, based on matching a given regular expression.

### <a id="search"></a> Search

NyklDB contains a basic search engine. Data is search indexed for fast search results based on whole, and multiple 
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

NyklDB can aid, or hint to a user what they might be able to search for by returning search suggestions based on 
partial word matches. This is surprisingly fast and can be used to create an as-you-type experience to fill in a 
search box and/or the dropdown list under the search box. Recent successful searches are cached and returned at the 
top of the next search suggestion, giving you search history as well (though this cache isn't saved anywhere so it will 
be gone the next time you access the app or refresh the webpage).

### <a id="sync-and-share"></a>Sync and Share

Data can be exported as a JSON file which can be saved or transferred somewhere and reimported without worrying 
about overwriting more recent data because all data (in every single cell) is stored with accompanying LastModified 
attributes.

This means that data can be synchronized between local storage and cloud storage, devices, or even people.
Conflict resolution follows the assumption that the most recent data in each cell is the data that you want to keep.

Sharing data securely based on a Diffie-Hellman key exchange system coming soon...

### <a id="data-validation"></a> Data Validation

Data that you input into a NyklDB object is validated before it is saved, at a minimum, by verifying that it is 
either a String, Number, or Boolean value. All data *must* fall into one of these three categories. Other types of 
data, such as JavaScript Objects, Arrays or the values `null` and `undefined` are not allowed in a NyklDB and will 
throw an error.

>#### Why no `null` or `undefined`?
>Trying to save the values `null` or `undefined` likely indicates an error somewhere in your code. If you would like to clear a string value, save the value `''` (an empty string). If it's a number, save `0`, or a boolean, save `false`. These are the default values that a new row will contain when it is first initialized (unless you specify a different initial value for a column).

Within these 3 general types of data are many more subtypes. Additional data validation, and sometimes error 
correction can be done by specifying a subtype, i.e. streetAddress is a subtype of String. Some subtypes fall into 
more than one category such as *postalZipCode* which can be a String or a Number. [See Types below](#types) for a 
complete list of all the different subtypes.

If you want the most flexibility you can specify *"any"*, which will allow any type of data to be saved to that column 
in the table, as long as it is a String, Number or Boolean value (though there is no additional validation).

The data type for each column in the table is best specified when the table is created, but in some cases can be 
changed afterwards, if it doesn't cause a conflict with the existing data in the table.

### <a id="formulas"></a> Formulas

You can set a formula on a particular column to pre-calculate its values. Formulas are one line expressions that 
follow the following rules:

* Start with a function name, followed by parentheses 
* Put function parameters inside the parentheses, separated by commas
* Parameters inside "double" quotes are literal text values
* Parameters without quotes, and starting with a $ (dollar sign) refer to a column name
* Parameters without quotes, and starting with a # (pound sign) refer to a property name
* Dots join functions together
* Functions can be nested inside other functions

Available functions include:

| Function Name		| Alternate Names	| Description						| Parameters                            | Returns
|-------------------|-------------------|-----------------------------------|---------------------------------------|--------
| ADD				| PLUS				| Add 2 or more numbers together    | Num1, Num2...                         | Number
| AND               | ALL_TRUE          | Check whether 2 or more conditions are true | Test1, Test2...             | Boolean
| CONVERT_UNITS     |                   | Multiply by common unit conversions | Num, FromUnit, ToUnit               | Number
| DIVIDE			|					| Divide Num1 by Num2				| Num1, Num2                            | Number
| GET_VALUE			| VALUE_OF, VALUE   | Retrieve a value                  | $ColName or #PropName                 | Value
| HAS_VALUE			| IS_NOT_EMPTY		| Check whether a value has been assigned | $ColName or #PropName           | Boolean
| IF				|					| Check whether a condition is true	| Test, ValueIfTrue, ValueIfFalse       | Value
| IS_BOOLEAN		|					| Check if a value is a true or false (boolean) value | Value               | Boolean
| IS_EQUAL_TO		| EQUALS			| Check if 2 values are the same	| Value1, Value2                        | Boolean
| IS_GREATER_THAN	|       			| Check if Num1 is greater than Num2 | Num1, Num2                           | Boolean
| IS_LESS_THAN		|   				| Check if Num1 is less than Num2	| Num1, Num2                            | Boolean
| IS_NUMBER			|       			| Check if a value is a number		| $ColName or #PropName                 | Boolean
| IS_STRING			| IS_TEXT           | Check if a value is a text value	| $ColName or #PropName                 | Boolean
| JOIN				|					| Join text values together 		| Any # of $ColNames, #PropNames or "text" | String
| MATCHES           |                   | Check whether any of the values equals Value1 | Value1, Value2...         | Boolean 
| MINUS				| SUBTRACT			| Subtract Num2 from Num1			| Num1, Num2                            | Number
| MULTIPLY			| TIMES				| Multply Num1 by Num2				| Num1, Num2                            | Number
| OR                | ANY_TRUE          | Check whether any condition is true | Test1, Test2...                     | Boolean
| ROUND             |                   | Round a Num to Decimals length    | Num, Decimals                         | Number 
| TO_FRACTION       | FRACTION          | Convert a decimal to a fraction   | Num                                   | String
| TO_NUMBER         |                   | Convert value to number           |                                       | Number
| TO_STRING         | TO_TEXT           | Convert value to text             |                                       | String
| TO_LOWERCASE		|                   | Convert text to lowercase letters	|                                       | String
| TO_TITLECASE		|                   | Capitalise text					|                                       | String
| TO_UPPERCASE		|                   | Convert text to all uppercase letters |                                   | String

| TRIM				|					| Remove extra spaces from text		|                                       | String
| More to come....

> ###### Formula example
> `GET_VALUE($LastName).TO_UPPERCASE().JOIN(", ",$FirstName," ",$MiddleName).TRIM()`
> gets the value in column 'LastName', converts it to ALL CAPS, joins it with a ", " (comma space), the value in column
> 'FirstName', " " (space), the value in column 'MiddleName', and then trims out any extra spaces to build something like
> `SMITH, William John`

| A note on literal text values     |
|:---------|
| If a literal value (in double quotes) contains a double quote symbol, you must escape it (preceed it with a backslash), otherwise it will be interpreted as the end of the literal value and not as part of the text. <br><br>For example: to create `William "The Boss" Smith`, you would do: <br><br>`JOIN($FirstName," \"",$NickName,"\" ",$LastName)`<br><br>or in JavaScript:

```JavaScript
// in single quotes
var formula = 'JOIN($FirstName," \"",$NickName,"\" ",$LastName)';

// in double quotes (note the triple backslash because you must escape the escape symbol too)
var formula2 = "JOIN($FirstName,\" \\\"\",$NickName,\"\\\" \",$LastName)"
```

# <a id="types"></a> Types 

The type of data that a column expects can be specified when creating a new table or adding a new column to a table. 
Any data that you try to set on a cell will be first validated according to what type of data the column accepts. 
In some cases the data will be coerced into fitting into the column type (for example converting the string value 
"4" into the number 4), or in other cases an error will be returned.

Some of the types are actually validated exactly the same, for example **string** and **multilineString**, but you 
can use a specific type to indicate what sort of input field to provide to the user for that particular value. A 
**string** value might use a basic `input` of `type="text"`, whereas a **multilineString** may work better as a 
`textarea` element. Likewise a **color** type would work good with a `input` of `type="color"` and **date** with a 
`type="date"` input.

The following types can be specified:

### <a id="string-types"></a>String Types

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

### <a id="numeric-types"></a> Numeric Types

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

### <a id="boolean-types"></a> Boolean Types

* **any** see [String types: any](#string-types)
* **boolean** accepts only true or false
* **checkbox** accepts only true or false

> Note: not all types listed are currently fully implementated in this Beta version, specifically uniqueString, password, country, integer, posInteger, negInteger, url, range, and date

# <a id="getting-started"></a> Getting Started

The data store can run along-side your other JavaScript code by adding a \<script\> tag directly to your HTML 
document, include it as a Javascript module, or run it in a background task in a Web Worker (which is recommended).

### <a id="javascript-installation"></a> JavaScript Installation

To start using NyklDB using JavaSript, download a copy of `nyklDB.min.js`, `base64.min.js`, `storage.js`, `Lawnchair.js`, and the Lawnchair adaptors that you want to use (`dom.js`, `indexed-db.js` for starters), and insert them into your html file with script tags, loading them in this order:
```html
    <script type="text/javascript" src="scripts/base64.min.js"></script>
    <script type="text/javascript" src="scripts/Lawnchair.js"></script>
    <script type="text/javascript" src="scripts/adapters/indexed-db.js"></script>
    <script type="text/javascript" src="scripts/adapters/dom.js"></script>
    <script type="text/javascript" src="scripts/storage.js"></script>
    <script type="text/javascript" src="scripts/nyklDB.min.js"></script>
```

### <a id="setting-up-a-new-nykldb"></a> Setting up a new NyklDB

Setting up a new NyklDB is as simple as calling the constructor using the "new" keyword and giving it a 
title. The title identifies this particular NyklDB data store. After the NyklDB data store is created, initialize 
it by calling the "init" function and passing it the required table "headers" parameter. Optional "customProperties" 
and "importData" can also be passed to the table on setup. A callback function can return a boolean value indicating 
whether or not the NyklDB data store was successfully created and initiated, and any error messages. Inside the 
callback function, 'this' refers to the NyklDB instance.

> [See more details about the NyklDB class](NyklDB.html)

```javascript
var myTable = new APP.NyklDB("Accounting Spreadsheet");

var headers = [
        { name: "Month", type: "string"}, 
        { name: "Monthly Income", type: "posInteger" }, 
        { name: "Monthly Expenses", type: "negInteger" },
        { name: "Balance", type: "integer"}
    ];
var options = {
        //see 'The Options Parameter' below
    };
var callback = function (success, errors) { 
        if (success) {
            console.log(this.getTitle()); //returns "Accounting Spreadsheet"
            //update UI to show newly created NyklDB here
        } else console.log(errors);
};

myTable.init(headers, options, callback);
myTable.getLength(); //returns 0
```

For each column header, column "name" and data "type" are required, as shown above, but additional metadata can also be included such as whether a column should be indexed for searching, what it's initial value should be if nothing is specified, or even a [formula](#formulas), like this:

```javascript
var headers = [
    {
        name: "Month",
        type: "string", 
        search: true,
        initialValue: "January"
    },
    {
        name: "Monthly Income",
        type: "posInteger",
        search: false,
        initialValue: 2000
    },
    {
        name: "Monthly Expenses",
        type: "negInteger",
        search: false
    },
    {
        name: "Balance",
        type: "integer",
        formula: "SUBTRACT($Monthly_Income,$Monthly_Expenses)"
    }
]
```

### <a id="the-options-parameter"></a> The Options Parameter

The 'options' parameter accepts an Object. It may contain data to import immediately after the database is created 
in the form of a CSV file (string), or JSON object, and custom properties that you would like to add to your table.

```javascript
var options = {
    "importData": nyklDBTable,
    "importJSON": json,
    "importCSV": string,
    "customProperties": {
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
//table data that was generated by NyklDB sync or exportJSON functions
"importData": nyklDBTable, 

//json data that needs to be parsed
"importJSON": json, 

//or CSV data that needs to be converted to JSON and then parsed
"importCSV": string, 
```

##### <a id="custom-properties"></a> Custom Properties

Custom properties can be used for data that doesn't quite fit into the "table" model, such as maybe some metadata 
that applies to the entire database. The existence of a custom property must be initiated when the NyklDB object 
is created, and must be given an initial value. Just like table columns, custom properties can be given a data type 
as well, or they will default to the type `"any"`.

```javascript
customProperties:{
    //name the custom property whatever you want
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

# <a id="dependencies"></a> Dependencies

* base64.js
* storage.js
* Lawnchair.js
