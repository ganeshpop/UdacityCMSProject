# Microsoft Azure
## Deploy an Article CMS to Azure
| Video Source | Link |
| ------ | ------ |
| YouTube | https://youtu.be/wqOl5fskEB0 |


 Download and install Requirements:
| Requirement | Link |
| ------ | ------ |
| Git |https://git-scm.com/download |
| ODBC Driver | https://www.microsoft.com/en-us/download/details.aspx?id=56567
| SQL CMD Tool | https://go.microsoft.com/fwlink/?linkid=2142258 |

>Set Path Variable(CMD):

```sh
PATH=%PATH%;"C:\Program Files\Microsoft SQL Server\Client SDK\ODBC\170\Tools\Binn"
```

>Download Project Files(CMD): 

```sh
git clone https://github.com/udacity/nd081-c1-provisioning-microsoft-azure-vms-project-starter
```
>Navigate to Downloaded Folder and Rename the Project Folder(CMD):

```sh
ren nd081-c1-provisioning-microsoft-azure-vms-project-starter SourceProject
```


>Login to Azure with CLI:

```sh
az login
```

>Create a Resource Group:
```sh
az group create --name ArticleCmsRg --location <YOUR-LOCATION>
```


>Create an SQL Server:

```sh
az sql server create --name <YOUR-SQL-SERVERNAME> --admin-user <USERNAME> --admin-password <PASSWORD> --resource-group ArticleCmsRg
```

##### In Config-py File:
 #
| Replace | With |
| ------ | ------ |
|SQL_SERVER_NAME | ``<YOUR-SQL-SERVERNAME>``|
|SQL_SERVER_USERNAM | ``<USERNAME>``|
|SQL_SERVER_PASSWORD | ``<PASSWORD>``|

>Create a Database:

```sh
az sql db create --name ArticleCmsDb --resource-group ArticleCmsRg --server <YOUR-SQL-SERVERNAME> 
```


##### In Config-py File:
 #
| Replace | With |
| ------ | ------ |
|SQL_DB_NAME | ArticleCmsDb|


>Locate Your Public IP Address (Copy Address from Name): 

  ```sh
  nslookup myip.opendns.com resolver1.opendns.com
  ```

>Enable Firewall Access for Your IP:

```sh
az sql server firewall-rule create --resource-group ArticleCmsRg --server <YOUR-SQL-SERVERNAME> --name "AllowMyIP" --start-ip-address <IP-FROM-PREVIOUS-STEP>  --end-ip-address  <IP-FROM-PREVIOUS-STEP>
```

#### Navigate to Source Project Folder:

> Create user and article tables and insert data:

```sh
sqlcmd -S <YOUR-SQL-SERVERNAME>.database.windows.net -d ArticleCmsDb -U sqladmin -i .\sql_scripts\users-table-init.sql
```

```sh
sqlcmd -S <YOUR-SQL-SERVERNAME>.database.windows.net -d ArticleCmsDb -U sqladmin -i .\sql_scripts\posts-table-init.sql
```

>Create a Storage Account:

```sh
az storage account create  --name <YOUR-STORAGE-ACCOUNT-NAME>  --resource-group ArticleCmsRg  --sku Standard_ZRS  
```

##### Navigate to your Storage Account and Create an Image Container with Access Level set to Container
#
>Under Settings Copy an Access Key

##### In Config-py File:
#
 
| Replace | With |
| ------ | ------ |
|BLOB_STORAGE_KEY | ``<YOUR-ACCESS-KEY>``|


>In Azure Active Directory Create App Registration Copy <CLIENT_ID> from App Registration Overview

>Create and Copy <CLIENT_SECRET> from Certificates & Secrets 

##### In Config-py File:
#
 
| Replace | With |
| ------ | ------ |
|CLIENT_ID| ``<CLIENT_ID>``|
|CLIENT_SECRET |``<CLIENT_SECRET>``|

## Add Logging Functionality:

>Under FlaskWebProject in __init__.py File add these lines:

```sh
	app.logger.setLevel(logging.INFO)
	streamHandler = logging.StreamHandler()
	streamHandler.setLevel(logging.INFO)
	app.logger.addHandler(streamHandler)
```
	
>In views.py File Add-

- For Invalid Password add this Line:

```sh
app.logger.warning("Logging in failed for user:{}".format(form.username.data))
```

- For Successful Login add this Line:

```sh
app.logger.info("Logging in user:{}".format(form.username.data))
```

- For Logout:

```sh
app.logger.info("Logging out user: {}".format(session.get("user")))
```

## Add functionality to the Sign-In with Microsoft button:

### In Views-py File
> In authorized function add this Line:

```sh
result = _build_msal_app(cache=cache).acquire_token_by_authorization_code(request.args['code'],scopes=Config.SCOPE,redirect_uri=url_for('authorized', _external=True, _scheme='https'))
```
- *Remove* **result = None**



> In _load_cache function add these Lines:

```sh
    cache = msal.SerializableTokenCache()
    if session.get('token_cache'):
        cache.deserialize(session['token_cache'])
    return cache
```
- _Remove_ *cache = None*

>In  _save_cache function add these Lines:

```sh
    if cache.has_state_changed:
        session['token_cache'] = cache.serialize()
```

>In _build_msal_app function add this Line:

```sh
    return msal.ConfidentialClientApplication(
    Config.CLIENT_ID, authority=authority,
    client_credential=Config.CLIENT_SECRET, token_cache=cache)
```

>In _build_auth_url function add this Line:

```sh
    return _build_msal_app(authority=authority).get_authorization_request_url(scopes,state=state,redirect_uri=url_for('authorized', _external=True, _scheme='https'))
```

### Deploy The App:

```sh
az webapp up --name FinalCmsProgApp --sku F1 --resource-group ArticleCmsRg
```

>In the App Overview Page Copy App URL <YOUR-APP-URL> 
>In Azure Active Directory Under your App Registration
>Click Add a Redirect URI
>Click Add a Platform
> Under Web 
Set The following URls-

| Field | Value |
| ------ | ------ |
|Redirect URIs|``<YOUR-APP-URL>/getAToken``|
|Front-channel logout URL|``<YOUR-APP-URL>/login``|


>Get Public IP Address of the app:

```sh
 az webapp show --resource-group ArticleCmsRg --name FinalCmsProgApp  --query outboundIpAddresses --output tsv
```

>Copy the Smallest and Latrgest IP's as <SMALLEST-IP> and <Largest-IP>

>Allow App to access internal sql server:

```sh
az sql server firewall-rule create --resource-group ArticleCmsRg --server finalcmssqlserver  --name "AllowAppIP" --start-ip-address  <SMALLEST-IP-FROM-PREVIOUS-STEP>  --end-ip-address  <LARGEST-IP-FROM-PREVIOUS-STEP>
```



### _Run and Test the Application_

## ✨By G S Sai Ganesh Koundinya✨
[![N|Solid](https://cdn.business2community.com/wp-content/uploads/2016/02/View-my-LinkedIn-profile-image-3-300x140.png.png)](https://www.linkedin.com/in/s-sai-ganesh-koundinya-gollapudi-25285118a/)
