## Game > GameAnvil > Console User Guide > Management


## Machine

It is the menu that is used to register the machines on which GameAnvil processes would be run and manage the registered machines. 

GameAnvil Agent must be installed before registering machines. [ [GameAnvil-Agent Download](https://static.toastoven.net/prod_gameanvil/files/gameanvil-agent-1.1.4.1.tar) ] 

![management_machine_main.png](https://static.toastoven.net/prod_gameanvil/images/management_machine_main_en.png)

### Register Machine

Registers the machine on which GameAnvil processes would be run. 

Machines must be registered right after the GameAnvil service is activated.

![management_machine_register.png](https://static.toastoven.net/prod_gameanvil/images/management_machine_register_en.png)
* Input type: Can register by directly enter the machine information on the console window or register multiple machines at once by uploading a .csv file.
* Host name: Enter the host name of the machine to be registered.
* IP address: Enter the Public IP of the machine to be registered. The address is used to communicate with the console, so it must be accurate. 
* GameAnvil Agent Port: Enter the information on the GameAnvil Agent Port installed on the machine. ( Default value is 19080. )
    * File upload: Download a template file, enter the information of the machine to be registered, and upload files to it.  [ [Template File](https://static.toastoven.net/prod_gameanvil/files/GameAnvil_Machine_Template.csv)]


### Machine List
Can check the list of registered machines and provides a feature to be used to search host names, IP addresses, and descriptions.  
![management_machine_list.png](https://static.toastoven.net/prod_gameanvil/images/management_machine_list_en.png)

### Machine settings 

Sets the master machine and location management machine after the machines are registered. 

Select one or more master machine and location management machine among the registered machines. 

![management_machine_setup.png](https://static.toastoven.net/prod_gameanvil/images/management_machine_setup_en.png)
* Master machine: Select the machine on which Management Node would be run. Instances and nodes can be controlled only if a master machine is registered. 
* Location management machine: Select the machine on which Location Node would be run. 
* Java version: Select the version of Jana in which GameAnvil server build was used. 

Although both Management Node and Location Node are essential to GameAnvil, they cannot implement or control content.
Management Node and Location Node can be managed by using the set machine and initialize machine features. 

### Reset machine settings

It is a feature that is used to reset the settings of the master machine and location management machine registered using the machine setting feature.

When the settings of existing machine is reset, Management Node and Location Node are closed as well.  
Machine settings can be initialized only when no registered instant is currently in operation.

![management_machine_init.png](https://static.toastoven.net/prod_gameanvil/images/management_machine_init_en.png)

![management_machine_setup_none.png](https://static.toastoven.net/prod_gameanvil/images/management_machine_setup_none_en.png)



## Instance

Manages the instance to be displayed on the registered machine through machine management.

An instance is consisted of several [nodes](https://github.com/TOAST-DOCS/GameAnvil/blob/alpha/en/server-2-basic) and instances can be freely configured according to the game environment.

### List

Can check the list of registered instances.

The basic information of an instance and the information of the configured nodes are exposed.

![ManagementInstance-0](https://static.toastoven.net/prod_gameanvil/images/management_instance_list_en.png)

Provides filters per node type, and the instance name and host name search feature.

Filters and keywords work with the AND condition.

![ManagementInstance-1](https://static.toastoven.net/prod_gameanvil/images/management_instance_filter_en.png)

The detailed information of the nodes configured in an instance can be checked with a popup by clicking the **Confirm** button.

Can check individual node with the tab located on the top of the screen.

![ManagementInstance-2](https://static.toastoven.net/prod_gameanvil/images/management_instance_config_popup_en.png)

### Register

This is the instance registration screen.

Enter the basic information of an instance first and then set the nodes in the configuration.

For setting nodes, the settings can only be entered in the form of a template. For more information on template, refer to **Instance > Settings** Guide.

The top of the registration screen where the basic information of an instance can be entered.

The entered value per item is checked in real time and displays a notification if invalid information is entered.

Note that **Server build upload paths** and **machines** cannot be edited once they are registered.

![ManagementInstance-3](https://static.toastoven.net/prod_gameanvil/images/management_instance_register_1_en.png)

The machine to deploy an instance can be selected by entering the basic information of the instance (instance name and server build upload path).

Machine management displays the list of registered machines with a popup.

Instances that share the same configuration can be selected and registered.

![ManagementInstance-12](https://static.toastoven.net/prod_gameanvil/images/management_instance_register_2_en.png)

As the basic information is covered, let's move onto the node settings.

**COMMON, VM_OPTION** must be set among node types and at least one of the following nodes must be set: **GATEWAY, GAME, SUPPORT, and MATCH**.

A node can be set up in a way of selecting a pre-registered template by using **Import Settings**.

![ManagementInstance-4](https://static.toastoven.net/prod_gameanvil/images/management_instance_register_3_en.png)

It is possible to check settings of each node for registered templates by using the popup menu item of Import Settings.

Once a template in the list is selected, the registration screen is filled with all the settings except the default values.

This function is useful when an instance of the same configuration is to be registered or managed.

![ManagementInstance-14](https://static.toastoven.net/prod_gameanvil/images/management_instance_register_4_en.png)

To implement **Import Settings**, the port information of **COMMON, GATEWAY, and SUPPORT** shall be entered.

Depending on **GATEWAY and SUPPORT** nodes, the user may enter a port.

This is the screen where **GATEWAY** node is configured.

The port input is dynamically configured according to the use of **TCP/WEB SOCKET**.

![ManagementInstance-6](https://static.toastoven.net/prod_gameanvil/images/management_instance_register_5_en.png)

This is the screen where **SUPPORT** node is configured.

The port input is dynamically configured according to the **number of services** configured.

![ManagementInstance-7](https://static.toastoven.net/prod_gameanvil/images/management_instance_register_6_en.png)

This is the area configured according to the **GATEWAY and SUPPORT** settings.

Port uses a value between **18000-20000** and there should not be a duplicate value in the same machine.

After the configured port is entered, the user must check any Port duplicates among the registered instances using **Check for Duplicate Ports**. 

![ManagementInstance-11](https://static.toastoven.net/prod_gameanvil/images/management_instance_register_7_en.png)

If it turns out that there is a duplicate port entry as a result of implementing **Check for Duplicate Ports**, a window pops up to notify the information of the duplicate port.

Based on this information, it is required to re-enter a port and repeat the **Check for Duplicate Ports** process until there is no duplicate entry.

![ManagementInstance-14](https://static.toastoven.net/prod_gameanvil/images/management_instance_register_8_en.png)

Once the **Check for Duplicate Ports** process is completed properly, all of the input fields in the instance registration screen becomes **Inactivated** and it is ready for final registration.

To change some of the settings before saving them, change their status to **Active** by clicking **Enter Settings Again** before changing the settings.

Once a setting becomes ***\*Active\****, it must go through the **Check for Duplicate Ports** process again to register it.

![ManagementInstance-14](https://static.toastoven.net/prod_gameanvil/images/management_instance_register_9_en.png)

![ManagementInstance-14](https://static.toastoven.net/prod_gameanvil/images/management_instance_register_10_en.png)

### Change and Deletion

Registered instances can be edited or deleted.

To edit or delete an instance, the status of the instance must be **waiting to be started, stopped, or error**.

From **Monitoring >Instance Monitoring**, the current status of an instance can be checked or edited.

![ManagementInstance-15](https://static.toastoven.net/prod_gameanvil/images/management_instance_modify_delete_en.png)

![ManagementInstance-16](https://static.toastoven.net/prod_gameanvil/images/management_instance_start_en.png)

In the instance correction step, it is unable to change the **Server Build Upload Path** and **Machine** settings.

It is possible to change instance node settings through the **Import Settings** function just as in the registration step.

![ManagementInstance-17](https://static.toastoven.net/prod_gameanvil/images/management_instance_modify_en.png)

## Settings

Manages the templates of each node type of an instance.

The settings of each node can only be configured by selecting a template when registering an instance.

It is convenient if the settings of each frequently used node are pre-registered.

### List

Can check the list of registered templates.

The list is sorted by template name and the current status of the instances that are using the template.

The function to search template names is available.

![ManagementTemplate-1](https://static.toastoven.net/prod_gameanvil/images/management_template_list_en.png)

![ManagementTemplate-2](https://static.toastoven.net/prod_gameanvil/images/management_template_list_popup_en.png)

### Register

This is the template registration screen.

Register the settings to be used for each node in an instance as a template.

The settings for each node cannot be entered from the instance registration screen. As the settings can only be configured using a template, the necessary settings must be pre-registered.

As default settings are provided, it is possible to register templates in reference to them.

Register and utilize templates according to the characteristics of a game.

First of all, enter the basic information of a template, the name and description.

It is possible to enter settings of each node directly. It is also possible to import and register existing template settings by using the **Import Settings** function.

To register a similar template, import settings by using the **Import Settings** function and then enter only certain settings that need to be changed.  

![ManagementTemplate-3](https://static.toastoven.net/prod_gameanvil/images/management_template_register_1_en.png)

**COMMON, VM_OPTION** must be set among node types and at least one of the following nodes must be set: **GATEWAY, GAME, SUPPORT, and MATCH**.

These are settings of the **COMMON** node.

![ManagementTemplate-4](https://static.toastoven.net/prod_gameanvil/images/management_template_register_2_en.png)

These are settings of the **VM OPTION** node.

![ManagementTemplate-5](https://static.toastoven.net/prod_gameanvil/images/management_template_register_3_en.png)

These are settings of the **GATEWAY** node.

Settings may be different depending on whether to use TCP/WEB SOCKET or SSL.

![ManagementTemplate-6](https://static.toastoven.net/prod_gameanvil/images/management_template_register_4_en.png)

These are settings of the **GAME** node.

![ManagementTemplate-7](https://static.toastoven.net/prod_gameanvil/images/management_template_register_5_en.png)

To set up the GAME node, **Service** needs to be selected.

Max. 99 services are available and selectable through the **Select Services** popup. 

Services may be managed by using the **Manage > Instance> Service** menu. 

![ManagementTemplate-8](https://static.toastoven.net/prod_gameanvil/images/management_template_register_6_en.png)

Once a desired service is selected and confirmed, you can enter more settings necessary for each service. 

![ManagementTemplate-9](https://static.toastoven.net/prod_gameanvil/images/management_template_register_7_en.png)  

To set up the SUPPORT node as well, **Service** needs to be selected.

Just as in the case of GAME node, max. 99 services are available and selectable through the **Select Services** popup with the additional settings entered. 

![ManagementTemplate-10](https://static.toastoven.net/prod_gameanvil/images/management_template_register_8_en.png)

If there are services that have yet to be registered in a template, it is possible to register them through the **Select Services** popup.

As the **Register Services** button is clicked, the registration popup will be displayed.

A service is registered once the service ID and service name are entered.

Only registering is possible. For correcting and deleting, the **Manage > Instance > Service** menu should be used.  

![ManagementTemplate-11](https://static.toastoven.net/prod_gameanvil/images/management_template_register_9_en.png)

![ManagementTemplate-12](https://static.toastoven.net/prod_gameanvil/images/management_template_register_10_en.png)

Finally, these are settings of the **MATCH** node.

![ManagementTemplate-13](https://static.toastoven.net/prod_gameanvil/images/management_template_register_11_en.png)

Each input item of templates has a default value, minimum value, and maximum value applied.

The **Reset Settings** feature returns settings of the node to default values.

The **GAME** and **SUPPORT** nodes do not provide the **Reset Settings** function.

If the entered setting value is outside the minimum or maximum value, the corresponding value is automatically changed to the value used when the field was created while FocusOut.


### Change and Deletion

Registered templates can be edited or deleted.

To edit or delete a template, the status of the instances applied to the template must be **waiting to be started, stopped, error**.

Instances that are using a template can be checked via a popup from **Settings List** and the current status of an instance can be checked or edited from **Instance Monitoring**.

![ManagementTemplate-15](https://static.toastoven.net/prod_gameanvil/images/management_template_modify_delete_en.png)

Since the template correction step does not provide the **Import Settings** function, it is required to correct settings directly as necessary.

![ManagementTemplate-16](https://static.toastoven.net/prod_gameanvil/images/management_template_modify_en.png)

Templates that are applied to an instance cannot be deleted regardless of the status of the instance.

A template can be deleted after it is changed or deleted from the instance.

![ManagementTemplate-17](https://static.toastoven.net/prod_gameanvil/images/management_template_delete_en.png)

## Services

Services are identifiers to be used in the **GAME** and **SUPPORT** nodes in the process of the bootstrap of the server code produced by means of **GameAnvil**.

### List

This screen shows the list of registered services.

Services are sorted by type (in the order of GAME and SUPPORT) and then sorted by Service ID.

It is possible to check whether there is a template where the service is used, and if any, which template it is.

It is possible to search services by name.

![ManagementService-1](https://static.toastoven.net/prod_gameanvil/images/management_service_list_en.png)

### Register

The **Service ID**s may be entered with numbers between 1 and 99. Service IDs should not duplicate.

The **Service Name** must be the same with the identifier used in the server code’s bootstrap. Attention must be paid to prevent typing errors since the name is case-sensitive. Just as in the case of service IDs, service names should not duplicate.

There are two ways of service registration: to enter directly and to upload a file.

Click **Enter Directly** and enter the desired value as shown below.

![ManagementService-2](https://static.toastoven.net/prod_gameanvil/images/management_service_add_en.png)

In the second way, click **Upload a File** as below and then click ***Template Example Files***, enter desired data for the corresponding template style, and select or drag and drop a file. It is possible to save multiple services at once.

It should be noted that when a file is uploaded, only one service type is available. Thus, GAME files should be separated from SUPPORT files when saved.

![ManagementService-3](https://static.toastoven.net/prod_gameanvil/images/management_service_fileupload_1_en.png)

Press the Save button, and then the screen below will be displayed. Check if the value on the screen is correct and then press the Save button at the bottom once again. Now the entered value will be transmitted to the server.

![ManagementService-4](https://static.toastoven.net/prod_gameanvil/images/management_service_fileupload_2_en.png)

### Change and Deletion

If a **Service Name** should be corrected or deleted, click the corresponding service in the list. Then the Details screen will be displayed.

It is possible to correct a service name only when there is no instance that is operating by using the template where the service is registered.

It is possible to delete a service only when there is no template where the service is registered.

![ManagementService-5](https://static.toastoven.net/prod_gameanvil/images/management_service_detail_en.png)

## Deployment file

This is the menu that is used to manage the process file deployed to the GameAnvil instance. 

![ManagementDeployFile-1](https://static.toastoven.net/prod_gameanvil/images/management_deployfile_en.png)

### Upload File

Deployment files can be uploaded through the Upload File menu. Only 1 file can be uploaded at a time.

Only files of which extension is .jar are supported. The allowed size of a file to be uploaded is 1 GB.

Only file upload is done when the upload button is clicked and the deployment is completed only when the **Deploy** feature is run from the instance monitoring menu.

![ManagementDeployFile-2](https://static.toastoven.net/prod_gameanvil/images/management_deployfile_upload_en.png)

### Deployment history

The deployment history of the deployment file is exposed when the Check Deployment History button is clicked from the list of deployment files.

Can check the information of the deployed instances, machines, and date of deployment and others. 


![ManagementDeployFile-3](https://static.toastoven.net/prod_gameanvil/images/management_deployfile_history_en.png)
