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
10. [Drop Table](#drop-table)


#### Add SWC

Just need to copy the SWC and paste in you Flex lib package.

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
	CREATE TABLE IF NOT EXISTS keygenadmin
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

And call initDB() method of local data base manager with creation complete handler of you flex app as fallows -

```AS3

KeyGenDBManager.instance.initDB();

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

#### Drop Table

Nedd to call **dropTableData** method of **dataBaseManager** as fallows -

```AS3
	public function dropAdminTable():void
	{
		dataBaseManager.instance.dropTableData(ADMIN_TABLE_INDEX);
	}

```


## Event Description

Database manager SWC dispatch following Events, whose description is as fallows -

11. [DB POPULATED](#db-populated)
12. [DB UPDATE DONE](#db-update-done)

#### DB POPULATED

>`DB_POPULATED`

This event will be dispatched when Database is created and populated with default data in **populateTable.sql** . This will have **dbData** Array with the event params, and this arrary include the data(Array collection Object) of DB in sequence of **ADMIN_TABLE_INDEX** i.e. the index passed with populating the DB constants.

#### DB UPDATE DONE

>`DB_UPDATE_DONE`

This event will be dispatched when database update request is done. This will have **dbData** Array with the event params, and this arrary include the data(Array collection Object) of DB in sequence of **ADMIN_TABLE_INDEX** i.e. the index passed with populating the DB constants.

## Sample Class

A consolidated implementation of above steps is as fallows - 

```AS3

package com.devtrip
{
	import com.devtrip.utils.db.dataBaseManager;
	import com.devtrip.utils.db.events.DBEventDispatcher;
	import com.devtrip.utils.db.events.DBEvents;
	import com.devtrip.utils.db.tableFactory;
	import com.devtrip.vo.devtripVo;
	
	import mx.collections.ArrayCollection;

	public class DTDMKeyGenDBManager
	{
		private static var _instance : DTDMKeyGenDBManager = null;
		
		private static const DB_FILE : String = "dtdmkeygen.db";
		private static const DB_PWD : String = "1234567891234567";
		
		private  static const USER_TABLE_INDEX : int = 0;
		private  static const USER_TABLE_NAME : String = "dtdmkeygenuser";
		[Embed(source="sql/user/createTable.sql", mimeType="application/octet-stream")]
		private static const CreateUserTableQuerry : Class;
		private static const CREATE_USER_TABLE_SQL:String = new CreateUserTableQuerry();
		[Embed(source="sql/user/populateTable.sql", mimeType="application/octet-stream")]
		private static const PopulateUserTableQuerry : Class;
		private static const POPULATE_USER_TABLE_SQL:String = new PopulateUserTableQuerry();
		[Embed(source="sql/user/addRow.sql", mimeType="application/octet-stream")]
		private static const AddRowUserTableQuerry : Class;
		private static const ADD_ROW_USER_SQL:String = new AddRowUserTableQuerry();
		[Embed(source="sql/user/updateRow.sql", mimeType="application/octet-stream")]
		private static const updateRowUserTableQuerry : Class;
		private static const UPDATE_ROW_USER_SQL:String = new updateRowUserTableQuerry();
		[Embed(source="sql/user/deleteRow.sql", mimeType="application/octet-stream")]
		private static const deleteRowUserTableQuerry : Class;
		private static const DELETE_ROW_USER_SQL:String = new deleteRowUserTableQuerry();
		
		private  static const APP_TABLE_INDEX : int = 1;
		private  static const APP_TABLE_NAME : String = "dtdmkeygenapp";
		[Embed(source="sql/app/createTable.sql", mimeType="application/octet-stream")]
		private static const CreateAppTableQuerry : Class;
		private static const CREATE_APP_TABLE_SQL:String = new CreateAppTableQuerry();
		[Embed(source="sql/app/populateTable.sql", mimeType="application/octet-stream")]
		private static const PopulateAppTableQuerry : Class;
		private static const POPULATE_APP_TABLE_SQL:String = new PopulateAppTableQuerry();
		[Embed(source="sql/app/addRow.sql", mimeType="application/octet-stream")]
		private static const AddRowAppTableQuerry : Class;
		private static const ADD_ROW_APP_SQL:String = new AddRowAppTableQuerry();
		[Embed(source="sql/app/updateRow.sql", mimeType="application/octet-stream")]
		private static const updateRowAppTableQuerry : Class;
		private static const UPDATE_ROW_APP_SQL:String = new updateRowAppTableQuerry();
		[Embed(source="sql/app/deleteRow.sql", mimeType="application/octet-stream")]
		private static const deleteRowAppTableQuerry : Class;
		private static const DELETE_ROW_APP_SQL:String = new deleteRowAppTableQuerry();
		
		private  static const KEY_TABLE_INDEX : int = 2;
		private  static const KEY_TABLE_NAME : String = "dtdmkeygenkey";
		[Embed(source="sql/key/createTable.sql", mimeType="application/octet-stream")]
		private static const CreateKeyTableQuerry : Class;
		private static const CREATE_KEY_TABLE_SQL:String = new CreateKeyTableQuerry();
		[Embed(source="sql/key/populateTable.sql", mimeType="application/octet-stream")]
		private static const PopulateKeyTableQuerry : Class;
		private static const POPULATE_KEY_TABLE_SQL:String = new PopulateKeyTableQuerry();
		[Embed(source="sql/key/addRow.sql", mimeType="application/octet-stream")]
		private static const AddRowKeyTableQuerry : Class;
		private static const ADD_ROW_KEY_SQL:String = new AddRowKeyTableQuerry();
		[Embed(source="sql/key/updateRow.sql", mimeType="application/octet-stream")]
		private static const updateRowKeyTableQuerry : Class;
		private static const UPDATE_ROW_KEY_SQL:String = new updateRowKeyTableQuerry();
		[Embed(source="sql/key/deleteRow.sql", mimeType="application/octet-stream")]
		private static const deleteRowKeyTableQuerry : Class;
		private static const DELETE_ROW_KEY_SQL:String = new deleteRowKeyTableQuerry();
		
		public function DTDMKeyGenDBManager(enforcer:SingletonEnforcer)
		{
			if (_instance != null) throw Error('Singelton error');
			_instance = this;
		}
		
		/**
		 * @Public [access point for class]
		 * @param - [NA] 
		 * @return - [available instance of the class
		 * */
		public static function get instance() : DTDMKeyGenDBManager {
			if(_instance == null){
				_instance = new DTDMKeyGenDBManager(new SingletonEnforcer());
			}
			return _instance;
		}
		
		/**
		 * 
		 * 
		 **/
		
		public function initDB():void
		{
			dataBaseManager.instance.dbFile = DB_FILE;
			dataBaseManager.instance.dbPwd = DB_PWD;
			
			var user : Object = tableFactory.createObject(
				USER_TABLE_INDEX, 
				USER_TABLE_NAME, 
				CREATE_USER_TABLE_SQL, 
				POPULATE_USER_TABLE_SQL, 
				ADD_ROW_USER_SQL, 
				UPDATE_ROW_USER_SQL,
				DELETE_ROW_USER_SQL
			);
			dataBaseManager.instance.dbTable = user;
			
			var app : Object = tableFactory.createObject(
				APP_TABLE_INDEX, 
				APP_TABLE_NAME, 
				CREATE_APP_TABLE_SQL, 
				POPULATE_APP_TABLE_SQL, 
				ADD_ROW_APP_SQL, 
				UPDATE_ROW_APP_SQL,
				DELETE_ROW_APP_SQL
			);
			dataBaseManager.instance.dbTable = app;
			
			var key : Object = tableFactory.createObject(
				KEY_TABLE_INDEX, 
				KEY_TABLE_NAME, 
				CREATE_KEY_TABLE_SQL, 
				POPULATE_KEY_TABLE_SQL, 
				ADD_ROW_KEY_SQL, 
				UPDATE_ROW_KEY_SQL,
				DELETE_ROW_KEY_SQL
			);
			dataBaseManager.instance.dbTable = key;
						
			DBEventDispatcher.getInstance().addEventListener(DBEvents.DB_POPULATED,handleDBEvents);
			DBEventDispatcher.getInstance().addEventListener(DBEvents.DB_UPDATE_DONE,handleDBEvents);
			dataBaseManager.instance.openDatabase();
			
		}
		
		/**
		 * 
		 * 
		 **/
		
		private function handleDBEvents(evt:DBEvents):void
		{
			switch(evt.type){
				case DBEvents.DB_POPULATED:
					
					devtripVo.instance.appdbdata = evt.params.dbData[APP_TABLE_INDEX];
					devtripVo.instance.userdbdata = evt.params.dbData[USER_TABLE_INDEX];
					devtripVo.instance.keydbdata = evt.params.dbData[KEY_TABLE_INDEX];
					logAppData(devtripVo.instance.appdbdata);
					logUserData(devtripVo.instance.userdbdata);
					logKeyData(devtripVo.instance.keydbdata);
					
					break;
				case DBEvents.DB_UPDATE_DONE:
					
					if(evt.params.dbTableIndex == APP_TABLE_INDEX)
						devtripVo.instance.appdbdata = evt.params.dbData[APP_TABLE_INDEX];
					else if(evt.params.dbTableIndex == USER_TABLE_INDEX)
						devtripVo.instance.userdbdata = evt.params.dbData[USER_TABLE_INDEX];
					else if(evt.params.dbTableIndex == KEY_TABLE_INDEX)
						devtripVo.instance.keydbdata = evt.params.dbData[KEY_TABLE_INDEX];
					
					logAppData(devtripVo.instance.appdbdata);
					logUserData(devtripVo.instance.userdbdata);
					logKeyData(devtripVo.instance.keydbdata);
					
					break;
				default:
					trace("do nothing");
			}
		}
		
		/**
		 * 
		 * 
		 **/
		
		private function logUserData(arr : ArrayCollection):void{
			for(var i:String in arr){
				trace("id : " + arr[i].id +
					" , " + "userEmail : " + arr[i].userEmail +
					" , " + "userUName : " + arr[i].userUName +
					" , " + "userPwd : " + arr[i].userPwd + 
					" , " + "userApp : " + arr[i].userApp + 
					" , " + "userAccess : " + arr[i].userAccess);
			}
		}
		
		private function logAppData(arr : ArrayCollection):void{
			for(var i:String in arr){
				trace("id : " + arr[i].id +
					" , " + "appId : " + arr[i].appId +
					" , " + "appName : " + arr[i].appName +
					" , " + "appVesrion : " + arr[i].appVesrion + 
					" , " + "appAdmin : " + arr[i].appAdmin + 
					" , " + "appUser : " + arr[i].appUser + 
					" , " + "appStatus : " + arr[i].appStatus);
			}
		}
		
		private function logKeyData(arr : ArrayCollection):void{
			for(var i:String in arr){
				trace("id : " + arr[i].id +
					" , " + "appId : " + arr[i].appId +
					" , " + "appName : " + arr[i].appName +
					" , " + "appVesrion : " + arr[i].appVesrion + 
					" , " + "appGuid : " + arr[i].appGuid + 
					" , " + "appUserEmail : " + arr[i].appUserEmail + 
					" , " + "appKeyDate : " + arr[i].appKeyDate);
			}
		}
		
		/**
		 * 
		 * 
		 **/
		
		public function addKey(obj:Object):void
		{
			var keyObj : Object = new Object();
			keyObj.appId = obj.id;
			keyObj.appName = obj.name;
			keyObj.appVesrion = obj.vesrion;
			keyObj.appGuid = obj.guid;
			keyObj.appUserEmail = obj.userEmail;
			keyObj.appKeyDate = obj.keyDate;
			
			dataBaseManager.instance.insertInfo(keyObj,KEY_TABLE_INDEX);
		}
		
		public function addApp(obj:Object):void
		{
			var appObj : Object = new Object();
			appObj.appId = obj.id;
			appObj.appName = obj.name;
			appObj.appVesrion = obj.version;
			appObj.appAdmin = obj.admin;
			appObj.appUser = obj.user;
			appObj.appStatus = obj.status;
			
			dataBaseManager.instance.insertInfo(appObj,APP_TABLE_INDEX);
		}
		
		public function addUser(obj:Object):void
		{
			var userObj : Object = new Object();
			userObj.userEmail = obj.Email;
			userObj.userUName = obj.UName;
			userObj.userPwd = obj.Pwd;
			userObj.userApp = obj.App;
			userObj.userAccess = obj.Access;
			
			dataBaseManager.instance.insertInfo(userObj,USER_TABLE_INDEX);
		}
		
		/**
		 * 
		 * 
		 **/
		
		public function updateKey(obj:Object):void
		{
			dataBaseManager.instance.updateInfo(obj,KEY_TABLE_INDEX);
		}
		
		public function updateApp(obj:Object):void
		{
			dataBaseManager.instance.updateInfo(obj,APP_TABLE_INDEX);
		}
		
		public function updateUser(obj:Object):void
		{
			dataBaseManager.instance.updateInfo(obj,USER_TABLE_INDEX);
		}
		
		/**
		 * 
		 * 
		 **/
		
		public function deleteKey(obj:Object):void
		{
			dataBaseManager.instance.deleteInfo(obj,KEY_TABLE_INDEX);
		}
		
		public function deleteApp(obj:Object):void
		{
			dataBaseManager.instance.deleteInfo(obj,APP_TABLE_INDEX);
		}
		
		public function deleteUser(obj:Object):void
		{
			dataBaseManager.instance.deleteInfo(obj,USER_TABLE_INDEX);
		}
		
		/**
		 * Cross Table Reference updats 
		 * (Similar to Foreign key concept which isnotavailable with AIR SQL Lite )
		 **/
		
		public function updateUserForNewApp(obj:Object,appId:String):void{
			var userIdArr : Array = (obj.user).split(",");
			var userDb : ArrayCollection = devtripVo.instance.userdbdata;
			for (var i:String in userIdArr){
				for(var j:String in userDb){
					if(userDb[j].id == userIdArr[i]){
						var userDataObject : Object = userDb[j];
						userDataObject.userApp = userDataObject.userApp + ","+appId;
						dataBaseManager.instance.updateInfo(userDataObject,USER_TABLE_INDEX);
					}
				}	
			}	
		}
		
		public function updateAppForNewUser(obj:Object,userId:String):void{
			var appIdArr : Array = (obj.App).split(",");
			var appDb : ArrayCollection = devtripVo.instance.appdbdata;
			for (var i:String in appIdArr){
				for(var j:String in appDb){
					if(appDb[j].id == appIdArr[i]){
						var appDataObject : Object = appDb[j];
						appDataObject.appUser = appDataObject.appUser + "," + userId;
						dataBaseManager.instance.updateInfo(appDataObject,APP_TABLE_INDEX);
					}
				}	
			}	
		}
		
		/**
		*
		*
		**/
		
		public function dropKeyTable():void
		{
			dataBaseManager.instance.dropTableData(KEY_TABLE_INDEX);
		}
		
		public function dropAppTable():void
		{
			dataBaseManager.instance.dropTableData(APP_TABLE_INDEX);
		}
		
		public function dropUserTable():void
		{
			dataBaseManager.instance.dropTableData(USER_TABLE_INDEX);
		}
	}
}

class SingletonEnforcer
{
}


```
