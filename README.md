# DTDatabaseManager

Provide ActionScript SWC to integrate database management functionality with AIR based application developed in AS3.

## Goal

This packages provide an access manager SWC component, which can easily be installed with AIR based applications, being developed using AS3 and/or MXML.

## Configuration 

It has normal basic configuration to integrate it with AIR based app. 

1. [Add SWC](#add-swc)
2. [Create a Local Dtatabase Manager Class](create-a-local-dtatabase-manager-class)
3. [Set Database file constants](set-database-file-constants)
4. [Set Sql Commands](#set-sql-commands)
5. [Initiate Database manager](#initiate-database-manager)
6. [Handle Database Events](#handle-database-events)
7. [Adding Row In Table](#adding-row-in-table)
8. [Updating Row Of Table](#updating-row-of-table)
9. [Deleting Row Of Table](#deleting-row-of-table)


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

#### Set Database file constants

DTDatabaseManager.swc have feature to create and handle n numbers of DataBase table from some basic external configurations.

To Configure DB file, add following constants.

```AS3

	private static const DB_FILE : String = "keygen.db";
	private static const DB_PWD : String = "1234567890987654"; // Optional

```

#### Set Sql Commands

To set sql table and sql command define follwing commands as fallows -

```AS3

	private  static const ADMIN_TABLE_INDEX : int = 0;
	private  static const ADMIN_TABLE_NAME : String = "keygenadmin";
	[Embed(source="sql/admin/createTable.sql", mimeType="application/octet-stream")]
	private static const CreateAdminTableQuerry : Class;
	private static const CREATE_ADMIN_TABLE_SQL:String = new CreateAdminTableQuerry();
	[Embed(source="sql/admin/populateTable.sql", mimeType="application/octet-stream")]
	private static const PopulateAdminTableQuerry : Class;
	private static const POPULATE_ADMIN_TABLE_SQL:String = new PopulateAdminTableQuerry();
	[Embed(source="sql/admin/addRow.sql", mimeType="application/octet-stream")]
	private static const AddRowAdminTableQuerry : Class;
	private static const ADD_ROW_ADMIN_SQL:String = new AddRowAdminTableQuerry();
	[Embed(source="sql/admin/updateRow.sql", mimeType="application/octet-stream")]
	private static const updateRowAdminTableQuerry : Class;
	private static const UPDATE_ROW_ADMIN_SQL:String = new updateRowAdminTableQuerry();
	[Embed(source="sql/admin/deleteRow.sql", mimeType="application/octet-stream")]
	private static const deleteRowAdminTableQuerry : Class;
	private static const DELETE_ROW_ADMIN_SQL:String = new deleteRowAdminTableQuerry();

```

And sql commands in **sql/admin/** package as fallows-

sql/admin/createTable.sql -

```sql
	CREATE TABLE keygenadmin
	(
		id int PRIMARY KEY AUTOINCREMENT,
		adminEmail String NOT NULL,
		adminUName String NOT NULL,
		adminPwd String NOT NULL,
		adminAccess String NOT NULL
	)
```

sql/admin/populateTable.sql -

```sql

	INSERT INTO keygenadmin
	(
		adminEmail,
		adminUName,
		adminPwd,
		adminAccess
	)
	SELECT 'abc@xyz.com', 'admin', '12345678', 'full' UNION
	SELECT 'def@uvw.com', 'admin1', '12345678', 'full'

```

sql/admin/addRow.sql -

```sql

	INSERT INTO keygenadmin
	(
		adminEmail,
		adminUName,
		adminPwd,
		adminAccess
	)
	VALUES
	(
		:adminEmail,
		:adminUName,
		:adminPwd,
		:adminAccess
	)

```
sql/admin/updateRow.sql -

```sql

	UPDATE keygenadmin
	SET adminEmail = :adminEmail,
		adminUName = :adminUName,
		adminPwd = :adminPwd,
		adminAccess = :adminAccess
	WHERE id = :id

```

sql/admin/deleteRow.sql -

```sql

	DELETE FROM keygenadmin
	WHERE id = :id

```


#### Initiate Database manager

Create a public initDB() method with local database manager class as fallows and set internal data as fallows -

```AS3

		public function initDB():void
		{
			dataBaseManager.instance.dbFile = DB_FILE;
			dataBaseManager.instance.dbPwd = DB_PWD; // Optional
			
			var admin : Object = tableFactory.createObject(
				ADMIN_TABLE_INDEX, 
				ADMIN_TABLE_NAME, 
				CREATE_ADMIN_TABLE_SQL, 
				POPULATE_ADMIN_TABLE_SQL, 
				ADD_ROW_ADMIN_SQL, 
				UPDATE_ROW_ADMIN_SQL,
				DELETE_ROW_ADMIN_SQL
			);
			dataBaseManager.instance.dbTable = admin;
			
			DBEventDispatcher.getInstance().addEventListener(DBEvents.DB_POPULATED,handleDBEvents);
			DBEventDispatcher.getInstance().addEventListener(DBEvents.DB_UPDATE_DONE,handleDBEvents);
			dataBaseManager.instance.openDatabase();
			
		}

```

#### Handle Database Events

You need to add **handleDBEvents** as fallows -

```AS3

	private function handleDBEvents(evt:DBEvents):void
	{
		switch(evt.type){
			case DBEvents.DB_POPULATED:
				
				devtripVo.instance.admindbdata = evt.params.dbData[ADMIN_TABLE_INDEX];
				devtripVo.instance.appdbdata = evt.params.dbData[APP_TABLE_INDEX];
				devtripVo.instance.userdbdata = evt.params.dbData[USER_TABLE_INDEX];
				logAdminData(devtripVo.instance.admindbdata);
				logAppData(devtripVo.instance.appdbdata);
				logUserData(devtripVo.instance.userdbdata);
				break;
			case DBEvents.DB_UPDATE_DONE:
				
				if(evt.params.dbTableIndex == ADMIN_TABLE_INDEX)
					devtripVo.instance.admindbdata = evt.params.dbData[ADMIN_TABLE_INDEX];
				else if(evt.params.dbTableIndex == APP_TABLE_INDEX)
					devtripVo.instance.appdbdata = evt.params.dbData[APP_TABLE_INDEX];
				else if(evt.params.dbTableIndex == USER_TABLE_INDEX)
					devtripVo.instance.userdbdata = evt.params.dbData[USER_TABLE_INDEX];
				
				logAppData(devtripVo.instance.appdbdata);
				break;
			default:
				trace("do nothing");
		}
	}

```

#### Adding Row In Table

Nedd to call **insertInfo** method of **dataBaseManager** as fallows -

```AS3
	public function addAdmin(obj:Object):void
	{
		var adminObj : Object = new Object();
		adminObj.adminEmail = obj.Email;
		adminObj.adminUName = obj.UName;
		adminObj.adminPwd = obj.Pwd;
		adminObj.adminAccess = obj.Access;
		
		dataBaseManager.instance.insertInfo(adminObj,ADMIN_TABLE_INDEX);
	}

```

#### Updating Row Of Table

Nedd to call **updateInfo** method of **dataBaseManager** as fallows -

```AS3
	public function updateAdmin(obj:Object):void
	{
		//obj should have all elements of Admin table.
		dataBaseManager.instance.updateInfo(obj,ADMIN_TABLE_INDEX);
	}

```

#### Deleting Row Of Table

Nedd to call **deleteInfo** method of **dataBaseManager** as fallows -

```AS3
	public function deleteAdmin(obj:Object):void
	{
		//obj should have all elements of Admin table.
		dataBaseManager.instance.deleteInfo(obj,ADMIN_TABLE_INDEX);
	}

```


## Event Description

Database manager SWC dispatch following Events, whose description is as fallows -

10. [DB POPULATED](#db-populated)
11. [DB UPDATE DONE](#db-update-done)

#### DB POPULATED

>`DB_POPULATED`

This event will be dispatched when Database is created and populated with default data in **populateTable.sql** . This will have **dbData** Array with the event params, and this arrary include the data(Array collection Object) of DB in sequence of **ADMIN_TABLE_INDEX** i.e. the index passed with populating the DB constants.

#### DB UPDATE DONE

>`DB_UPDATE_DONE`

This event will be dispatched when database update request is done. This will have **dbData** Array with the event params, and this arrary include the data(Array collection Object) of DB in sequence of **ADMIN_TABLE_INDEX** i.e. the index passed with populating the DB constants.

