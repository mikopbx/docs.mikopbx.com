# Configuring IDE

## Debug settings

Go to "**Settings**" - "**Preferences**" - "**PHP**" - "**Debug**"

Setup **xDebug**. MikoPBX runs **Xdebug 3**, so the "Debug port" field must contain **9003** (the screenshots below were taken with an older default of 9000):

![xDebug setup PHP Storm](../../.gitbook/assets/sdebug-Preferences.png)

Setup **Advanced settings**

![](../../.gitbook/assets/advanced-Preferences.png)

Go to "**Settings**" - "**Preferences**" - "**PHP**" - "**Debug**" - "**DBGp Proxy**"

![](../../.gitbook/assets/dbgp-Preferences.png)

## Setup Deployment

1. Run "**Tools**" - "**Deployment**" - "**Browse Remote Hosts**" to open the panel "**Remote Host**"
2. Enable "**Tools**" - "**Deployment**" - "**Automatic Upload**"
3. Go to "**Tools**" - "**Deployment**" - "**Configuration...**"
4. Add new SFTP connection
5. Setup mappings

#### Main catalogs MikoPBX

* **`/usr/www`** - [Core](https://github.com/mikopbx/Core) files
* **`/storage/usbdisk1/mikopbx/custom_modules`** - directory for storing modules
* **`/storage/usbdisk1/mikopbx/custom_modules/ModulePhoneBook`** the [**PhoneBook**](https://github.com/mikopbx/ModulePhoneBook) module will be stored in this directory

#### Example of mapping settings:

Let's say a local directory with Core: **`/Users/alexeyportnov/project/pbx_miko/!Core`**

Local directory with modules: **`/Users/alexeyportnov/project/pbx_miko/modules`**,

Then the mapping settings will be as follows:

![PhpStorm Deploiment mappings](../../.gitbook/assets/Deployment_и__Core_–_PbxExtensionSetupBase_php___Core_.png)
