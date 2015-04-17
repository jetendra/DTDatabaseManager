# DTDatabaseManager

Provide ActionScript SWC to integrate database management functionality with AIR based application developed in AS3.

## Goal

This packages provide an access manager SWC component, which can easily be installed with AIR based applications, being developed using AS3 and/or MXML.

## Configuration 

It has normal basic configuration to integrate it with AIR based app. 

1. [Add SWC](#add-swc)
2. [Create a Local Dtatabase Manager Class](create-a-local-dtatabase-manager-class)
3. [Set Database constants](Set Database constants)
4. [Add Methods](#add-methods)
5. [Set Sql Commands](#set-sql-commands)

#### Add SWC

>`TBD`

#### Create a Local Dtatabase Manager Class

>`TBD`

#### Set Database constants

>`TBD`

#### Add Methods

>`TBD`

#### Set Sql Commands

>`TBD`

## Event Description

Database manager SWC dispatch following Events, whose description is as fallows -

6. [PRODUCT STATUS](#product-status)
7. [PRODUCT EXPIRED](#product-expired)

#### PRODUCT STATUS

>`PRODUCT_STATUS`

This event will be dispatched when product access validation is done on startup. This will have **_productStatus** objct with the event params, and this object includes following - 

installation_date, product_id, product_key, product_validity, product_status and product_guid.

#### PRODUCT EXPIRED

>`PRODUCT_EXPIRED`

This event will be dispatched on startup, when product free use time is elapsed.

#### Note -

>` A detailed description for class and methods is also available with doc package . The live view URL is https://cdn.rawgit.com/DevtripAccessManager/DTAM/master/doc/index.html`
