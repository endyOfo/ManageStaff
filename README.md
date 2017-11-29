 ManageStaff  Module 
<a id="top"></a>
[![N|magento](https://cldup.com/dTxpPi9lDf.thumb.png)](https://magento.com/)</br>
Author: Endy Ofo </br>


Custom Magneto Module to allow for content management users to be able add/edit and delete staff profiles for the business website.  Customers will be able to view the staff profiles through the front end of the website and contact them via LinkedIn or email links.

# Table of Contents
  * [Prerequisites](#prerequisites)
  * [Test Data Installation Script](#test-data-installation-script)
  * [Test Images](#test-images)
  * [DATA MODEL DESIGN](#data-model-design)
    * [Entity Tables](#entity-tables)
      * [1. ubt_managestaff_staff](#ubt_managestaff_staff)
      * [2. ubt_managestaff_teamcategory](#ubt_managestaff_teamcategory)
      * [3. ubt_managestaff_servicelist](#ubt_managestaff_servicelist)
      * [4. ubt_managestaff_regionlist](#ubt_managestaff_regionlist)
      * [5. ubt_managestaff_storeview](#ubt_managestaff_storeview)
      * [6. ubt_managestaff_viewablelist](#ubt_managestaff_viewablelist)
   * [ BACKEND DESIGN](#backend-design)
        * [Manage Team Members](#manage-team-members)
        * [Team Config](#team-config)
   * [FRONTEND DESIGN](#frontend-design)
   * [FUTURE TASK/ IMPROVEMENTS](#future-tasks)


# Prerequisites <a id="prerequisites"></a>

1. Installed magneto community 1.9.2 + /Enterprise 1.14 +
1. PHP 5.5 +
1. MySQL 5.6 + 

# Installation:-
Manual installation. 
Download and unzip the package, place the file and folders into the corresponding folders

  - app\code\local\Ubt\ManageStaff
  - app\etc\modules\Ubt_ManageStaff.xml 
  - app\design\adminhtml\default\default\layout\ubt_managestaff.xml
  - app\design\frontend\base\default\layout
  - app\design\frontend\base\default\template\ubt\manage_staff\teammembers
  - media\managestaff
  - skin\frontend\base\default\css
  - skin\frontend\base\default\font
  - skin\frontend\base\default\js

### **Activate Module-** 
Once the above step is completed, activate the module by clearing the caches at the location below. 

**NOTE:**  Clearing magneto’s  cache will trigger all newly installed modules to run their installation scripts thereby installing all required data for that module.  

<a id="cache-clear"></a>
### **Cache Clear** 
Login into the admin panel and follow the links below. 

```sh
admin => system => Cache mangement
```
Select all - then click the ‘Flush Magneto Cache' button. 

[ back to top](#top)
<a id="test-data-installation-script"></a>

# Test Data Installation Script
**FOR DEVELOPMENT MODE ONLY -**

The 'data installation script' will install test data if activated.

The script is located here: 

```sh
app\code\local\Ubt\ManageStaff\data\ubt_managestaff_setup\data-install-0.1.0.php
```
The script is commented out by default.

In order to activate it the following procedure should be followed.

- Go to the script page listed above and uncomment it by removing the following characters at the top and bottom of the page respectively:    /**      AND       **/
-  Activating the test script now depends on whether the 'ManageStaff module' has already been activated. If not, then 'the installation script' will automatically install the test data once the actual module is activated. 
-  If however the module has already been activated, then the simplest way to install the data is to force a **pseudo re-activation** of the module.

 **Pseudo Re-activation Of A Module** </br>
 
This procedure does not affect the module or its existing data in any way, all it does is force magneto to run through the module's 'installation scripts' again.  

Go to the "MySQL table" or whichever data storage table is being used, open the table: **'core_resource'**
Then search through the table, find and delete the entire row for the following entry: **ubt_managestaff_setup**

Once the row is deleted run the [Cache Clear](#cache-clear) again and the script will install the data.  

<a id="test-images"></a>
[back to top](#top)</br>
# Test Images

The content of the images folder is for test purposes only and should be deleted prior to installing the module on a production site: 
 
```sh
media\managestaff
```  

**Note:** the media banner folder within the media folder should not be deleted:

> media\managestaff\banner

<a id="data-model-design"></a>

# DATA MODEL DESIGN

The script for the module's data entities creation is stored here:   

```sh
app\code\local\Ubt\ManageStaff\sql\ubt_managestaff_setup\mysql4-install-0.1.0.php
```

The module consists of 6 data entities:

1.  **[ubt_managestaff_staff](#ubt_managestaff_staff)**
1.	**[ubt_managestaff_teamcategory](#ubt_managestaff_teamcategory)**
1. 	**[ubt_managestaff_servicelist](#ubt_managestaff_servicelist)**
1. 	**[ubt_managestaff_regionlist](#ubt_managestaff_regionlist)**
1. 	**[ubt_managestaff_storeview](#ubt_managestaff_storeview)**
1. 	**[ubt_managestaff_viewablelist](#ubt_managestaff_viewablelist)**

The entity models are all located within the following folder: 

```sh
app\code\local\Ubt\ManageStaff\Model
```
[ back to top](#top)</br></br>

 **1 - ubt_managestaff_staff** </br>

This is the main data entity [table](#ubt_managestaff_staff)   for the module and contains all data relevant to a particular staff profile. It is reliant on some of the other entities to the extent that it requires look up for some of its columns. For example, the column **'teamcategory_id'** only contains the Id of a particular team category. However, in order to get the actual category name we need to look up its value in the corresponding column (**teamcategory_id**) of the  **ubt_managestaff_teamcategory**.

An example of how the tables are linked during rendering can be seen
in the staff grid's $this->_prepareCollection() method: 

```php
app\code\local\Ubt\ManageStaff\Block\Adminhtml\Staff\Grid.php

$collection = Mage::getModel('ubt_managestaff/staff')->getCollection(); 
$team       = Mage::getSingleton('core/resource')->getTableName('ubt_managestaff/teamcategory'); 
$service    = Mage::getSingleton('core/resource')->getTableName('ubt_managestaff/servicelist'); 
$region     = Mage::getSingleton('core/resource')->getTableName('ubt_managestaff/regionlist'); 
        
$collection->getSelect()->group('main_table.staff_id')
           ->joinLeft(array('team'=>$team),'`main_table`.`teamcategory_id` = `team`.`teamcategory_id`',array('teamcategory_name'))
          ->joinLeft(array('service'=>$service),'`main_table`.`staffservice_id` = `service`.`service_id`',array('service_name'))
         ->joinLeft(array('region'=>$region),'`main_table`.`staffregion_id` = `region`.`regionlist_id`',array('country_name')); 
         
return $collection;        
```
 
**2 - ubt_managestaff_teamcategory**</br>
This  [table](#ubt_managestaff_teamcategory) contains the names of the different team categories and acts as a look up table for the **teamcategory_id** column in the  ubt_managestaff_staff  table. 

**3 - ubt_managestaff_servicelist**</br>
This [table](#ubt_managestaff_servicelist) contains the names of the different services offered by host company . The table acts as a look up for the  **staffservice_id** column in the  ubt_managestaff_staff  table. 

**4 - ubt_managestaff_regionlist**</br>
This [table](#ubt_managestaff_regionlist) contains the names of the different regions covered by host company. The table acts as a look up for the  **staffregion_id**  column in the  ubt_managestaff_staff  table. 

**6 - ubt_managestaff_viewablelist**</br>
This [table](#ubt_managestaff_viewablelist) contains the different types of viewable settings for staff members . The table acts as a look up for the  **content_viewable**  column in the  ubt_managestaff_staff table. 

The attributes of these models and the relationships between them are shown below.

<a id="entity-tables"></a>
[ back to top](#top)</br>
# Entity Tables


<a id="ubt_managestaff_staff"></a>

### **1. Table: ubt_managestaff_staff** </br>
-: Main Table For The Ubt Staff Module. 

| Column Name     | column Type | Description   | Value Range   |Example Data    |Notes   |
| :------- | ----: | :---: |:---: |:---: |:---: |
| staff_id      | int(10) |     | 5    | Primary key,auto increment |    |
| first_name    | text   | first name   |varchar(250)   |  john    | required    |
| surname       | text   |  surname | varchar(250)  |  doe    | required   |
| url_key       | text    |  234  | varchar(250)   |  Senior Developer    | required |
| jobtitle      | text    | job title  | varchar(250)   |  5    | required    |
| profile       | text    | staff profile  | varchar(250)   |  5    |     |
| profile_image | text   |  staff image  | varchar(250)   |  5    |     |
| emailaddress  | text    | staff email address  | varchar(250)   |  5    |     |
| linkedin      | text    | staff linkedin profile  | varchar(250)  |  5    |     |
|staffregion_id |smallint(5)|region code the member affiliated with-linked to ubt_managestaff_regionlist table  |varchar(250)   |  5    |     |
|staffservice_id|smallint(5)|service code the staff member affiliated with-linked to service table  | 5   |  1    |     |
|status         |smallint(5)	    |  234  | 5   |  1    |     |
|teamcategory_id|smallint(5)	    | service code the staff member affiliated with-linked to service table  | 5   |  1    |     |
|content_viewable|smallint(5)	   |  234  | 5   |  1    |     |
| created_at     |timestamp    |   | timestamp  | 2017-10-12 09:56:19|     |
| update_at     |timestamp   |    | timestamp  | 2017-10-12 09:56:19 |  |




<a id="ubt_managestaff_teamcategory"></a>


### **2. Table : ubt_managestaff_teamcategory**</br>

Look up table for the Ubt team category 


| Column Name     | column Type | Description   | Value Range   |Example Data    |Notes   |
| :------- | ----: | :---: |:---: |:---: |:---: |
| 	service_id      | int(10) |     | 5    | Primary key,auto increment |    |
| service_name     | text   | full name of service | varchar(250)  | service_name| required   |
| status       | smallint(5)    |    | 5   |  1    |  |
| store_id      | smallint(5)  |  | 5 |  1    |     |
| created_at     |timestamp    |   | timestamp | 2017-10-12 09:56:19|     |
| update_at     |timestamp   |    | timestamp   | 2017-10-12 09:56:19 |  |

[ back to top](#top)</br>
<a id="ubt_managestaff_servicelist"></a>


### 3.  **Table : ubt_managestaff_servicelist**</br>

Look up table for the Ubt regions 

| Column Name     | column Type | Description   | Value Range   |Example Data    |Notes   |
| :------- | ----: | :---: |:---: |:---: |:---: |
| 	service_id      | int(10) |     | 5    | Primary key,auto increment |    |
| service_name     | text   | full name of service | varchar(250)  | service_name| required   |
| status       | smallint(5)    |    | 5   |  1    |  |
| store_id      | smallint(5)  |  | 5 |  1    |     |
| created_at     |timestamp    |   |timestamp  | 2017-10-12 09:56:19|     |
| update_at     |timestamp   |    | timestamp   | 2017-10-12 09:56:19 |  |


<a id="ubt_managestaff_regionlist"></a>


### 4. **Table : ubt_managestaff_regionlist**

look up table for the Ubt regions 

| Column Name     | column Type | Description   | Value Range   |Example Data    |Notes   |
| :------- | ----: | :---: |:---: |:---: |:---: |
| regionlist_id      | int(10) |     | 5    | Primary key,auto increment |    |
| country_name    | text   | country abbriviation   |varchar(250)   |  UK   | required    |
| full_name      | text   | full country name | varchar(250)  |  United Kingdom| required   |
| status       | smallint(5)    |    | 5   |  1    |  |
| store_id      | smallint(5)  |  | 5 |  1    |     |
| created_at     |timestamp    |   | timestamp   | 2017-10-12 09:56:19|     |
| update_at     |timestamp   |    | timestamp | 2017-10-12 09:56:19 |  |

<a id="ubt_managestaff_viewablelist"></a>

### **6. Table: ubt_managestaff_viewablelist**

Look up table for the "content_viewable" column

| Column Name     | column Type | Description   | Value Range   |Example Data    |Notes   |
| :------- | ----: | :---: |:---: |:---: |:---: |
|viewablelist_id | int(10) |     | 5    | Primary key,auto increment |    |
|viewable_name| text   | full name of service | varchar(250)  | service_name| required   |
| status       | smallint(5)    |    | 5   |  1    |  |
| store_id      | smallint(5)  |  | 5 |  1    |     |
| created_at     |timestamp    |   | timestamp  | 2017-10-12 09:56:19|     |
| update_at     |timestamp   |    | timestamp | 2017-10-12 09:56:19 |  |



<a id="ubt_managestaff_storeview"></a>



[ back to top](#top)</br>

<a id="backend-design"></a>

# BACKEND DESIGN:-   

The backend user interface is accessed via a “Team” menu. This has been added to the existing CMS menu in the Magneto admin pages.

The 'Team menu' has 2 options:  

**[A. Manage Team Members](#manage-team-members)** </br>
**[B. Team Config](#team-config)**        </br>


<a id="manage-team-members"></a>

## A.  The 'Manage Team Members' Option:-

This option leads to the page responsible for rendering the grid iteration of all team members. There is a button at the top of page that clicks onto the 'add new members forms'.

Each iteration in the grid is clickable and leads to a form for editing that team member's data.

#### RELEVANT LINKS FOR 'MANAGE TEAM MEMBER' OPTIONS:- 
 
The **controller**  for this option  (i.e. the grid and the 'add new member forms') is located here:

```sh
app\code\local\Ubt\ManageStaff\controllers\Adminhtml\Managestaff\Staff\StaffController.php
```

the  **grid page** is located at following: 
```sh
the grid container: 
app\code\local\Ubt\ManageStaff\Block\Adminhtml\Staff.php
```
```sh
the grid page : 
app\code\local\Ubt\ManageStaff\Block\Adminhtml\Staff\Grid.php
```

the **xml page** for the grid page is located at

```sh
app\design\adminhtml\default\default\layout\ubt_managestaff.xml
```
The code below is an example of how the the xml page acts as a links between the  **front end** and the **database logic**..
```html
    <adminhtml_managestaff_staff_staff_index>
        <reference name="content">
            <block type="ubt_managestaff/adminhtml_staff" name="staffmanager" />
        </reference>
    </adminhtml_managestaff_staff_staff_index>
```

#### the 'add new team member' forms 

The container for the forms  are located here: 

```sh
Form container: 
app\code\local\Ubt\ManageStaff\Block\Adminhtml\Staff\Edit.php
```
```sh
Form Tabs buttons (the right sided tab buttons on the form pages): 
app\code\local\Ubt\ManageStaff\Block\Adminhtml\Staff\Edit\Tabs.php
```

The actual forms pages are located here: 

>app\code\local\Ubt\ManageStaff\Block\Adminhtml\Staff\Edit\form.php
>app\code\local\Ubt\ManageStaff\Block\Adminhtml\Staff\Edit\content.php

Once the forms are completed and saved by a user they are submitted to the **save method** of the StaffController.


**Save Method And Form Validation;** 
```php
app\code\local\Ubt\ManageStaff\controllers\Adminhtml\Managestaff\Staff\StaffController.php

public function saveAction() {
   if (($preFilteredData =  $this->_validateParams($this->getRequest()->getPost('team'))) &&    ($this->_validateFormKey())){ 
}

//validate form values prior to saving them. 
protected function _validateParams($params) {
}  
```

<a id="team-config"></a>
[ back to top](#top)</br>

## B.  The 'Team Config' Options:-

This option leads to the page responsible for rendering the tabbed grid iterations of each of the following entities:

1. Team Category Setting
1. Region Setting
1. Service Setting
1. Viewable Setting  

Each entity is hosted within its own tab and each iteration within that tab is clickable and leads to a form for editing the relevant data.

#### RELEVANT LINKS FOR 'Team Config' OPTIONS:- 

The **controller** for this option (i.e. the grids and the 'add new forms') is located here:

```sh
app\code\local\Ubt\ManageStaff\controllers\Adminhtml\Modulesettings\SettingsController.php
```

The  **grid pages** are located here: 
```sh
The Form Container: 
app\code\local\Ubt\ManageStaff\Block\Adminhtml\config\edit.php
```

```sh
The grid pages: 
app\code\local\Ubt\ManageStaff\Block\Adminhtml\config\edit\tabs.php
```

**NOTE:** The grids use a form container rather than the traditional 'grid container', thus enabling access to the **'forms tabs'**. This enables the rendering of several grids on the same page.

The index method of the SettingsController.phpc joins all the grids under a single tab: 


```php
 public function indexAction(){
    $block = $this->getLayout()
                ->createBlock('ubt_managestaff/adminhtml_config_edit')
                .......
                ->_addLeft($this->getLayout()->createBlock('ubt_managestaff/adminhtml_config_edit_tabs'))
}    
```
**Rendering Of The Individual Grids:-**

The individual grids are all stored in the following location:

```sh
app\code\local\Ubt\ManageStaff\Block\Adminhtml\Config\Grids
```
#### Example of how the grids are rendered:

**FIRST- The Edit Tab**    (adds tab buttons to the left side of the grid page.) :

```php
app\code\local\Ubt\ManageStaff\Block\Adminhtml\config\edit\tabs.php
//example of the TEAMCATEGORY tab.
$this->addTab('form_teamcategory', array(
   'label'     => Mage::helper('ubt_managestaff')->__('Team Category Setting'),
     .....
    'url'       => $this->getUrl('*/*/teamcategory', array('_current' => true)),
    'class'     => 'ajax',
));
```
**SECOND - The method and controller for team category tab** (each tab requires its own Action method) :
```php
app\code\local\Ubt\ManageStaff\controllers\Adminhtml\Modulesettings\SettingsController.php
public function teamcategoryAction() {
}    
```

**THIRD - the XML** (each Action method requires xml nodes):
```html
app\design\adminhtml\default\default\layout\ubt_managestaff.xml
<adminhtml_modulesettings_settings_teamcategory>
    <block type="ubt_managestaff/adminhtml_config_edit_tab_teamcategorybutton" name="teamcategory.form"/>
    <block type="ubt_managestaff/adminhtml_config_grids_teamcategory_grid" name="teamcategory.grid"/>
</adminhtml_modulesettings_settings_teamcategory>

```

**NOTE:**   the first call of the xml is to a custom block. In this example, the block is called 'teamcategory.form'. This block adds a button to the top of the grid page which clicks onto the 'add new entity form'.

The button's url also stores the **param variables** of the unique grid button being clicked; 
i.e 
```php
//the grid button for the teamcategory 
'onclick' => "setLocation('{$this->getUrl('*/*/editform', array('configtype' => 'teamcategory'))}')", 
```
These params are used in the **editform method** of the  settingsController.php to determine which model should be called and which form block to call i.e: 

```php
app\code\local\Ubt\ManageStaff\controllers\Adminhtml\Modulesettings\SettingsController.php
public function editformAction() {
    $configtype = $this->getRequest()->getParam('configtype');
    $model = Mage::getModel("ubt_managestaff/$configtype")->load($id);
    $block = $this->getLayout()
                 ->createBlock("ubt_managestaff/adminhtml_config_grids_edit")
                 ................
                ->_addLeft($this->getLayout()->createBlock("ubt_managestaff/adminhtml_config_grids_{$configtype}_edit_tabs"))
}   
```

**FOURTH - The Forms:**

Each entity within the unique grids has its own form button at the top of the page. Each button leads to a separate form.
Location of the forms:  
```sh
app\code\local\Ubt\ManageStaff\Block\Adminhtml\Config\Edit\Tab
//example of the teamcategory form: 
app\code\local\Ubt\ManageStaff\Block\Adminhtml\Config\Edit\Tab\Teamcategory.php
```
[ back to top](#top)</br></br>
**IMPORTANT NOTES:**

**Delete Button At Top Of Edit Form.** 

These buttons are dynamically generated via the **observer pattern** method. The params within the URL of the button are unique to each form; they enable the delete method to determine which model class to load and delete. 


```php
app\code\local\Ubt\ManageStaff\Model\Observer.php

public function addDeletionButton($observer) {
    //these values were set in the editformAction method.
    $configvalues = Mage::registry('moduleConfigValues');
    $data = array('onclick'   => 'setLocation(\'' . $container->getUrl('*/*/delete',  array('configtype' => $configvalues['configtype'],'id'=>$configvalues['id'])) . '\')',
            );
    $container->addButton('my_button_identifier', $data);
}
```

**The delete method:** 

```php
app\code\local\Ubt\ManageStaff\controllers\Adminhtml\Modulesettings\SettingsController.php

public function deleteAction(){
    $configModel = $this->getRequest()->getParam('configtype');
    $model = Mage::getModel("ubt_managestaff/$configModel")->load($id);
   $model->delete();
}
```

**SUBMISSION OF FORMS :**
All submitted forms are sent to the same destination   i.e the **save method** of SettingsController:
```php
app\code\local\Ubt\ManageStaff\controllers\Adminhtml\Modulesettings\SettingsController.php

public function saveAction() {
}
```
The **save method** loops through the names of all the tabs to determine which of them set the form. It then load the corresponding named model and saves the post data. 

<a id="frontend-design"></a>
[ back to top](#top)</br>
# FRONTEND DESIGN :-

The frontend consists of a single page for viewing the profiles associated with the store the customer is on. 

The page has the following functionality;

Search functionality at the top of page; enabling search based on the team member’s first name, surname, services or region. 

The  **front page phtml**  is a composition of several phtml pages; all contained within the folder :

```sh
app\design\frontend\base\default\template\ubt\manage_staff\teammembers
```
The  **controller/method**  for the front page is:

```php
app\code\local\Ubt\ManageStaff\controllers\IndexController.php
public function indexAction() {
}  
```

The  **XML**  for the front page is:

```sh
app\design\frontend\base\default\layout\ubt_managestaff.xml
The node is:  ubt_managestaff_index_index
```
```xml
<ubt_managestaff_index_index>
    <reference name="top.container">
        <block type="ubt_managestaff/teammembers" name="teammemberbanner"
template="ubt/manage_staff/teammembers/banner.phtml" /> 
    </reference>
    <reference name="content">
        <block type="ubt_managestaff/teammembers"name="manage_staff/teammemberview.results" template="ubt/manage_staff/teammembers/list.phtml"/>
    </reference>
    <reference name="manage_staff/teammemberview.results">
        <block type="core/template" name="manage_staff/teammemberview.searchform" as="suppliersearchform"  template="ubt/manage_staff/teammembers/forms/searchform.phtml"/>
    </reference>
    <reference name="content">
        <block type="ubt_managestaff/teammembers" name="teammembersbottompage"      template="ubt/manage_staff/teammembers/jqueryscript.phtml" />
    </reference>
</ubt_managestaff_index_index>
```
- The banner for the page is located here: **app\design\frontend\base\default\template\ubt\manage_staff\teammembers\banner.phtml**
-  The 'banner title' (it’s called and activated in the teammember.phtml page): **app\design\frontend\base\default\template\ubt\manage_staff\teammembers\bannersearch-title.phtml**
-  The 'Free Text Search Bar' form (it’s called and activated in the teammember.phtml page): **app\design\frontend\base\default\template\ubt\manage_staff\teammembers\forms\namesearchform.phtml**

- the array of members values is loaded here: **app\design\frontend\base\default\template\ubt\manage_staff\list.phtml**

- The Search form (it’s called and activated in the list.phtml page): **app\design\frontend\base\default\template\ubt\manage_staff\teammembers\forms\searchform.phtml** 

-  The JQuery functionality for the page: **app\design\frontend\base\default\template\ubt\manage_staff\teammembers\jqueryscript.phtml**
 
**IMPORTANT NOTE;**  

On the team member’s front end- customer facing page:
```sh
app\design\frontend\base\default\template\ubt\manage_staff\list.phtml
```

There is a clickable **mailTo:**  for team members. 

To avoid exposure to emails to spam bots the emails have been wrapped in a  JavaScript method that places  3 characters before the start and end of the email. These pseudo characters are automatically removed when the email tag is clicked or viewed. 

```html
<a href="mailto:xxx<?php echo $this->escapeHtml( $_team['emailaddress']) ;?>xxx"
                             onmouseover="this.href=this.href.replace(/^(mailto\:)x{3}(.*)x{3}$/i, '$1$2');"   
                             target="_top">
                               send email
                             </a>    
```

The link below describes the method. 

[Effective method to hide email from spam bots](https://stackoverflow.com/a/26420534/5677271)

<a id="future-tasks"></a>
[ back to top](#top)</br>
# FUTURE TASK/ IMPROVEMENTS 

- [ ] The front end search should be converted to  ajax form submission 
- [x] detailed commenting of functions 
- [ ] add PHP Unit testing
- [ ] 



