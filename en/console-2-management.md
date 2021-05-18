## Game > GameAnvil > Console User Guide > Management


## Machine

It is the menu that is used to register the machines on which GameAnvil processes would be run and manage the registered machines. 

GameAnvil Agent must be installed before registering machines. [ [GameAnvil-Agent Download](https://static.toastoven.net/prod_gameanvil/files/gameanvil-agent-1.1.4.tar) ] 

![management_machine_main.png](https://static.toastoven.net/prod_gameanvil/images/management_machine_main.png)

### Register Machine

Registers the machine on which GameAnvil processes would be run. 

Machines must be registered right after the GameAnvil service is activated.

![management_machine_register.png](https://static.toastoven.net/prod_gameanvil/images/management_machine_register.png)
* Input type: Can register by directly enter the machine information on the console window or register multiple machines at once by uploading a .csv file.
* Host name: Enter the host name of the machine to be registered.
* IP address: Enter the Public IP of the machine to be registered. The address is used to communicate with the console, so it must be accurate. 
* GameAnvil Agent Port: Enter the information on the GameAnvil Agent Port installed on the machine. ( Default value is 19080. )
    * File upload: Download a template file, enter the information of the machine to be registered, and upload files to it.  [ [Template File](https://static.toastoven.net/prod_gameanvil/files/GameAnvil_Machine_Template.csv)]


### Machine List
Can check the list of registered machines and provides a feature to be used to search host names, IP addresses, and descriptions.  
![management_machine_list.png](https://static.toastoven.net/prod_gameanvil/images/management_machine_list.png)

### Machine settings 

Sets the master machine and location management machine after the machines are registered. 

Select one or more master machine and location management machine among the registered machines. 

![management_machine_setup.png](https://static.toastoven.net/prod_gameanvil/images/management_machine_setup.png)
* Master machine: Select the machine on which Management Node would be run. Instances and nodes can be controlled only if a master machine is registered. 
* Location management machine: Select the machine on which Location Node would be run. 
* Java version: Select the version of Jana in which GameAnvil server build was used. 

Although both Management Node and Location Node are essential to GameAnvil, they cannot implement or control content.
Management Node and Location Node can be managed by using the set machine and initialize machine features. 

### Reset machine settings

It is a feature that is used to reset the settings of the master machine and location management machine registered using the machine setting feature.

When the settings of existing machine is reset, Management Node and Location Node are closed as well.  
To reset machine settings, all the instances registered to it must not be running.

![management_machine_init.png](https://static.toastoven.net/prod_gameanvil/images/management_machine_init.png)

![management_machine_setup_none.png](https://static.toastoven.net/prod_gameanvil/images/management_machine_setup_none.png)



## Instance

Manages the instance to be displayed on the registered machine through machine management.

An instance is consisted of several [nodes](https://github.com/TOAST-DOCS/GameAnvil/blob/alpha/en/server-2-basic) and instances can be freely configured according to the game environment.

### List

Can check the list of registered instances.

The basic information of an instance and the information of the configured nodes are exposed.

![ManagementInstance-0](https://static.toastoven.net/prod_gameanvil/images/management_instance_0.png)

Provides filters per node type, and the instance name and host name search feature.

Filters and keywords work with the AND condition.

![ManagementInstance-1](https://static.toastoven.net/prod_gameanvil/images/management_instance_1.png)

The detailed information of the nodes configured in an instance can be checked with a popup by clicking the **Confirm** button.

Can check individual node with the tab located on the top of the screen.

![ManagementInstance-2](https://static.toastoven.net/prod_gameanvil/images/management_instance_2.png)

### Register

This is the instance registration screen.

Enter the basic information of an instance first and then set the nodes in the configuration.

For setting nodes, the settings can only be entered in the form of a template. For more information on template, refer to **Instance > Settings** Guide.

The top of the registration screen where the basic information of an instance can be entered.

The entered value per item is checked in real time and displays a notification if invalid information is entered.

Note that **Server build upload paths** and **machines** cannot be edited once they are registered.

![ManagementInstance-3](https://static.toastoven.net/prod_gameanvil/images/management_instance_3.png)

The machine to deploy an instance can be selected by entering the basic information of the instance (instance name and server build upload path).

Machine management displays the list of registered machines with a popup.

Instances that share the same configuration can be selected and registered.

![ManagementInstance-12](https://static.toastoven.net/prod_gameanvil/images/management_instance_12.png)

As the basic information is covered, let's move onto the node settings.

**COMMON, VM_OPTION** must be set among node types and at least one of the following nodes must be set: **GATEWAY, GAME, SUPPORT, and MATCH**.

Nodes can be configured only through a template.

![ManagementInstance-4](https://static.toastoven.net/prod_gameanvil/images/management_instance_4.png)

A new template can be directly added and specified by either selecting a registered template or by **Add Setup Template** among setup selection.

![ManagementInstance-14](https://static.toastoven.net/prod_gameanvil/images/management_instance_14.png)

To configure a node, the port information for **COMMON, GATEWAY, and SUPPORT** must be entered.

Depending on **GATEWAY and SUPPORT** nodes, the user may enter a port.

Port uses a value between **18000-20000** and there should not be a duplicate value in the same machine.

![ManagementInstance-5](http://static.toastoven.net/prod_gameanvil/images/management_instance_5.png)

This is the screen where **GATEWAY** node is configured.

The port input is dynamically configured according to the use of **TCP/WEB SOCKET**.

![ManagementInstance-6](https://static.toastoven.net/prod_gameanvil/images/management_instance_6.png)

This is the screen where **SUPPORT** node is configured.

The port input is dynamically configured according to the **number of services** configured.

![ManagementInstance-7](https://static.toastoven.net/prod_gameanvil/images/management_instance_7.png)

This is the area configured according to the **GATEWAY and SUPPORT** settings.

After the configured port is entered, the user must check any duplicates among the registered instances using **Check for Duplicates**.  

![ManagementInstance-8](https://static.toastoven.net/prod_gameanvil/images/management_instance_8.png)

If a port that is entered using **Check for Duplicates**, a window pops up to notify the information of the duplicate port.

Based on this information, enter the port information again until the input values are filled by repeating the **Check for Duplicates** process.

![ManagementInstance-9](https://static.toastoven.net/prod_gameanvil/images/management_instance_9.png)

When all duplicate ports are resolved, the status of all the input fields becomes **inactive** and it is ready for final registration.

To change some of the settings before saving them, change their status to **active** by clicking **Enter settings again** before changing the settings.

Once a setting becomes **active** again, it must go through the **Check for Duplicates** process again to register it.

![ManagementInstance-11](https://static.toastoven.net/prod_gameanvil/images/management_instance_11.png)

![ManagementInstance-10](https://static.toastoven.net/prod_gameanvil/images/management_instance_10.png)

There is another way to register an instance besides the method explained earlier.

That is **Import Settings**.

![ManagementInstance-13](https://static.toastoven.net/prod_gameanvil/images/management_instance_13.png)

The **Import Settings** feature that copies the settings of an instance that is already registered.

Can check the node settings of the instances that are registered through the Import Settings popup.

When one of the instances in the list is selected, the registration screen is populated except the basic settings.

It is a feature that could be useful when registering instances with the same configuration.

### Change and Deletion

Registered instances can be edited or deleted.

To edit or delete an instance, the status of the instance must be **waiting to be started, stopped, or error**.

From **Instance Monitoring**, the current status of an instance can be checked or edited.

![ManagementInstance-15](https://static.toastoven.net/prod_gameanvil/images/management_instance_15.png)

![ManagementInstance-16](https://static.toastoven.net/prod_gameanvil/images/management_instance_16.png)

## Settings

Manages the templates of each node type of an instance.

The settings of each node can only be configured by selecting a template when registering an instance.

It is convenient if the settings of each frequently used node are pre-registered.

### List

Can check the list of registered templates.

The list is sorted by node type and the current status of the instances that are using the template.

Like instances, templates provide the feature to search for filters for each node type or template names.

Filters and keywords work with the AND condition.

![ManagementTemplate-1](https://static.toastoven.net/prod_gameanvil/images/management_template_1.png)

![ManagementTemplate-2](https://static.toastoven.net/prod_gameanvil/images/management_template_2.png)

### Register

This is the template registration screen.

Register the settings to be used for each node in an instance as a template.

The settings for each node cannot be entered from the instance registration screen. As the settings can only be configured using a template, the necessary settings must be pre-registered.

An available **STANDARD** template is provided when the product becomes active, so use this to register an instance.

Register and utilize templates according to the characteristics of a game.

![ManagementTemplate-3](https://static.toastoven.net/prod_gameanvil/images/management_template_3.png)

Templates are distinguished by node type. Some items have a default value, minimum value, and maximum value applied.

The **Reset Settings** feature returns entire settings of the currently selected type to default values.

If the entered setting value is outside the minimum or maximum value, the corresponding value is automatically changed to the value used when the field was created while FocusOut.

![ManagementTemplate-4](https://static.toastoven.net/prod_gameanvil/images/management_template_4.png)


### Change and Deletion

Registered templates can be edited or deleted.

To edit or delete a template, the status of the instances applied to the template must be **waiting to be started, stopped, error**.

Instances that are using a template can be checked via a popup from **Settings List** and the current status of an instance can be checked or edited from **Instance Monitoring**.

![ManagementTemplate-5](https://static.toastoven.net/prod_gameanvil/images/management_template_5.png)

Templates that are applied to an instance cannot be deleted regardless of the status of the instance.

A template can be deleted after it is changed or deleted from the instance.

![ManagementTemplate-6](https://static.toastoven.net/prod_gameanvil/images/management_template_6.png)



## Deployment file

This is the menu that is used to manage the process file deployed to the GameAnvil instance. 

![management_deployfile_en.png](https://static.toastoven.net/prod_gameanvil/images/management_deployfile.png)

### Upload File

Deployment files can be uploaded through the Upload File menu. Only 1 file can be uploaded at a time.

Only files of which extension is .jar are supported. The allowed size of a file to be uploaded is 1 GB.

Only file upload is done when the upload button is clicked and the deployment is completed only when the **Deploy** feature is run from the instance monitoring menu.

![managment_deployfile_upload_en.png](https://static.toastoven.net/prod_gameanvil/images/management_deployfile_upload.png)

### Deployment history

The deployment history of the deployment file is exposed when the Check Deployment History button is clicked from the list of deployment files.

Can check the information of the deployed instances, machines, and date of deployment and others. 


![managment_deployfile_history_en.png](https://static.toastoven.net/prod_gameanvil/images/management_deployfile_history.png)
