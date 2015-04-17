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

Just need to copy the SWC and pest in you Flex lib package.

#### Create a Local Dtatabase Manager Class

To configure DTDatabaseManager swc and handle its result create a new (local to app) database manager singleton class as fallows.

````AS3
package com.devtrip
{
	import com.devtrip.utils.db.dataBaseManager;
	import com.devtrip.utils.db.events.DBEventDispatcher;
	import com.devtrip.utils.db.events.DBEvents;
	import com.devtrip.utils.db.tableFactory;
	
	public class KeyGenDBManager
	{
	  
	  private static var _instance : KeyGenDBManager = null;
	  
		public function KeyGenDBManager(enforcer:SingletonEnforcer)
		{
			if (_instance != null) throw Error('Singelton error');
			_instance = this;
		}
		
		/**
		 * @Public [access point for class]
		 * @param - [NA] 
		 * @return - [available instance of the class
		 * */
		public static function get instance() : KeyGenDBManager {
			if(_instance == null){
				_instance = new KeyGenDBManager(new SingletonEnforcer());
			}
			return _instance;
		}
		
	}
}

class SingletonEnforcer
{
}
````

#### Set Database constants

DTDatabaseManager.swc have feature to create and handle n numbers of DataBase table from some basic external configurations.



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
