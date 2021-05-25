## Game > GameAnvil > Console User Guide > Monitoring

## Monitoring Dashboard

The machines and instances registered from the management page and the status of each node can be checked.

![monitoring_dashboard_main](https://static.toastoven.net/prod_gameanvil/images/monitoring_dashboard_main_en.png)

- Machine
    - Represents the total number of the machines registered to the management page and the total number of the machines that are normally connected to the Agent installed on machine.
        - Connected machines/total machines
    - If there is a problem with the connection to Agent, its status will become red.
    - Redirects to the '[System monitoring ](https://github.com/TOAST-DOCS/GameAnvil/blob/alpha/en/console-1-monitoring/#_5)' page through the Monitor button.
- Instance
    - The total number of instances and (Running) instances registered to the management page.
        - Active instances/total instances
    - Its status becomes red if there is even one erroneous instance.
    - Redirects to the '[Instance monitoring ](https://github.com/TOAST-DOCS/GameAnvil/blob/alpha/en/console-1-monitoring/#_9)' page through the Monitor button.
- User
    - Number of users connected to GAME node
- Room
    - Number of rooms created in GAME node
- Session
    - Number of sessions confirmed in GATEWAY node
- GATEWAY
    - Displays the total number of GATEWAY nodes and the number of the GATEWAY nodes of which status is READY. 
        - Number of READY GATEWAY nodes/total number of GATEWAY nodes
    - Its status becomes red if there is even one GATEWAY node of which status is DISABLE.
    - Redirects to the '[GATEWAY node monitoring ](https://github.com/TOAST-DOCS/GameAnvil/blob/alpha/en/console-1-monitoring/#gateway)' page through the Monitor button.
- GAME
    - Displays the total number of GAME nodes and the number of the GAME nodes of which status is READY. 
        - Number of READY GAME nodes/total number of GAME nodes
    - Its status becomes red if there is even one GAME node of which status is DISABLE.
    - Redirects to the '[GAME node monitoring ](https://github.com/TOAST-DOCS/GameAnvil/blob/alpha/en/console-1-monitoring/#game)' page through the Monitor button.
- SUPPORT
    - Displays the total number of SUPPORT nodes and the number of the SUPPORT nodes of which status is READY.
        - Number of READY SUPPORT nodes/total number of SUPPORT nodes
    - Its status becomes red if there is even one SUPPORT node of which status is DISABLE.
    - Redirects to the '[SUPPORT node monitoring ](https://github.com/TOAST-DOCS/GameAnvil/blob/alpha/en/console-1-monitoring/#support)' page through the Monitor button.
- MATCH
    - Displays the total number of MATCH nodes and the number of the MATCH nodes of which status is READY.
        - Number of READY MATCH nodes/total number of MATCH nodes
    - Its status becomes red if there is even one MATCH node of which status is DISABLE.
    - Redirects to the '[MATCH node monitoring ](https://github.com/TOAST-DOCS/GameAnvil/blob/alpha/en/console-1-monitoring/#match)' page through the Monitor button.

### Graph showing the change in concurrent users

The number of users who are connected to GAME node can be checked at a glance using a graph.

![monitoring_dashboard_graph](https://static.toastoven.net/prod_gameanvil/images/monitoring_dashboard_graph_en.png)

- The graph is automatically updated every minute, and you can check the graphs for today, yesterday, and the last week.
- If there is no data for yesterday or the last week, no graph will be displayed.
- The graph for concurrent users on a specific GAME node through a filter.



## User distribution monitoring

Can check the number of concurrent users on GAME node, distribution average, and the number of machines, instances, and nodes.

![monitoring_userdistribution_main](https://static.toastoven.net/prod_gameanvil/images/monitoring_userdistribution_main_en.png)

- Concurrent users
    - It means the number of concurrent users aggregated from READY GAME nodes.
    - The number of concurrent users may vary depending on filter.

- Machine
    - It means the number of the machines of which status is READY while GAME node is running.
    - The number of machines may vary depending on the filter.

- Instance
    - It means the number of the instances of which status is READY while GAME node is running.
    - The number of instances may vary depending on the filter.

- GAME node
    - It means the number of the nodes of which status is READY while GAME node is running.
    - The number of nodes may vary depending on the filter.

- User distribution average per node
    - The average of all active GAME nodes divided by concurrent users.
    - The average may vary depending on the filter.
        - Number of concurrent users/total number of active GAME nodes

※ The user monitoring page does not support the automatic refresh feature.<br>
If data needs to be refreshed, click the Refresh button.

### User distribution graph

<span style='color: #EB4927'>①</span>With the graph, the user distribution per machine, instance, and node can be checked through tab<span style='color: #EB4927'>①</span>.

- Machine
    - Appears only the machines of which status is READY while GAME node is running.
    - The label of the vertical axis is displayed as 'Host name.'
- Instance
    - Appears only the instances of which status is READY while GAME node is running.
    - The label of the vertical axis is displayed as 'Instance name@host name.'
- Node
    - Appears only the GAME node of which status is READY while running.
    - The label of the vertical axis is displayed as 'Node Id@host name.'

Up to 50 pieces of data are shown in the graph by default. To display more than 50 items, use 'Show More'<span style='color: #EB4927'>②</span>.

## System monitoring

In System monitoring, the status of machines and instances can be checked with simple information.

![monitoring_system_main](https://static.toastoven.net/prod_gameanvil/images/monitoring_system_main_en.png)

### Tree view

The hierarchy of the machines and instances registered from the management page are displayed in the tree form.

The tree is composed of the following:

![monitoring_system_tree](https://static.toastoven.net/prod_gameanvil/images/monitoring_system_tree_en.png)

Displays machines and the instances belong to each machine. 
By selecting a host name or instance name, the detailed information page of each machine is displayed.

In addition, machines can be easily found by using the host name search feature.

### When machine (host name) is selected,

The information on the selected machine and the instances belong to that machine is displayed.

![monitoring_system_machine](https://static.toastoven.net/prod_gameanvil/images/monitoring_system_machine_en.png)

- Information on machine and instance

1. Machine
    - Host name: The host name of machine
    - Machine status: The connection status with Agent
    - IP address: The IP address of machine
    - GameAnvil Agent Port: The port number of Agent

2. Instance
    - Instance name
    - Instance status
    - Total number of connected nodes: Number of READY nodes/total number of nodes
    - Number of users: The number of users who are connected the GAME node that belongs to an instance
        - Displayed only when the status of GAME node is READY while running
        - In other cases, displays '-'
    - Number of rooms: The number of rooms created by the GAME node that belongs to an instance
        - Displayed only when the status of GAME node is READY while running
        - In other cases, displays '-'

### When an instance is selected,

it displays the detailed information on the selected instance.

![monitoring_system_instance](https://static.toastoven.net/prod_gameanvil/images/monitoring_system_instance_en.png)

- Information on instance
    - Instance name
    - Instance status
    - Instance settings: Check the setting information on an instance through a popup window
    - GATEWAY
        - Number of running GATEWAY nodes/total number of GATEWAY nodes
        - Displays '-/-' if there is no corresponding node
    - GAME
        - Number of running GAME nodes/total number of GAME nodes
        - Displays '-/-' if there is no corresponding node
    - SUPPORT
        - Number of running SUPPORT nodes/total number of MASUPPORTTCH nodes
        - Displays '-/-' if there is no corresponding node
    - MATCH
        - Number of running MATCH nodes/total number of MATCH nodes
        - Number of running MATCH nodes/total number of MATCH nodes
    - Number of users: The number of users who are connected the GAME node that belongs to an instance
        - Displayed only when the status of GAME node is READY while running
        - In other cases, displays '-'
    - Number of rooms: The number of rooms created by the GAME node that belongs to an instance
        - Displayed only when the status of GAME node is READY while running
        - In other cases, displays '-'

## Instance monitoring

The instance information can be checked and each configured node can be started/stopped/deployed from the management page.

![monitoring_instance_main](https://static.toastoven.net/prod_gameanvil/images/monitoring_instance_main_en.png)

1. Search
    - A specific instance can be searched using a host name or instance name.

2. Filter
    - A specific instance can be filtered using the status of an instance or deploy or instance setting type.

    ![monitoring_instance_filter](https://static.toastoven.net/prod_gameanvil/images/monitoring_instance_filter_en.png)

### Instance table

![monitoring_instance_table](https://static.toastoven.net/prod_gameanvil/images/monitoring_instance_table_en.png)

#### Action<span style='color: #EB4927'>①</span>

An action can be performed on the instances selected from the checkbox of the table.

- Start
- Stop
- Deploy


#### Upload files to be deployed<span style='color: #EB4927'>②</span>

Deployment files can be uploaded in the same way as Management > Deployment File page.

![monitoring_instance_upload](https://static.toastoven.net/prod_gameanvil/images/monitoring_instance_upload_en.png)


#### Table

The status of each instance can be checked and take action on start/stop/deploy from an instance table.

- Instance name
- Host name
- Instance settings
- GATEWAY
    - Number of running GATEWAY nodes/total number of GATEWAY nodes
    - Displays '-' if there is no corresponding node
- GAME
    - Number of running GAME nodes/total number of GAME nodes
    - Displays '-' if there is no corresponding node
- SUPPORT
    - Number of running SUPPORT nodes/total number of SUPPORT nodes
    - Displays '-' if there is no corresponding node
- MATCH
    - Number of running MATCH nodes/total number of MATCH nodes
    - Displays '-' if there is no corresponding node
- Instance status
    - In 'Error' status, click 'Error'<span style='color: #EB4927'>③</span> to check the detailed error message.
- Deployment status
    - In 'Error' status, click 'Error'<span style='color: #EB4927'>③</span> to check the detailed error message.
- Action
    - Can perform actions (start/stop/deploy) in a single instance.

## Node monitoring

### ALL

Can monitor all the nodes of which status is READY.

![monitoring_node_all](https://static.toastoven.net/prod_gameanvil/images/monitoring_node_all_en.png)

- Node ID
- Service ID
    - Displays only for GAME and SUPPORT nodes.
- Service name
- Node type
- Instance name
- Host name
- Node status
- Action
    - Resume
        - Can start nodes of which status is Pause.
    - Pause
        - Stops the nodes of which status is Ready.
    - ※ MATCH nodes cannot perform the Resume and Pause action.

#### Filter

![monitoring_node_all_filter](https://static.toastoven.net/prod_gameanvil/images/monitoring_node_all_filter_en.png)

Specific nodes can be filtered using the filters below to all Ready nodes.
- Node type
- Node status
- Service name

### GATEWAY

Can monitor the GATEWAY nodes of which status is READY while running.

![monitoring_node_gateway](https://static.toastoven.net/prod_gameanvil/images/monitoring_node_gateway_en.png)

- Node ID
- Node type
- Instance name
- Host name
- Number of sessions
    - Displays the number of sessions connected to each GATEWAY node.
- Node status
- Action
    - Resume
        - Can start nodes of which status is Pause.
    - Pause
        - Stops the nodes of which status is Ready.

#### Filter

![monitoring_node_gateway_filter](https://static.toastoven.net/prod_gameanvil/images/monitoring_node_gateway_filter_en.png)

Specific nodes can be filtered using the filters below to all running GATEWAY nodes.
- Node status

### GAME

Can monitor the GAME nodes of which status is READY while running.

![monitoring_node_game](https://static.toastoven.net/prod_gameanvil/images/monitoring_node_game_en.png)

- Node ID
- Node type
- Service ID
- Channel ID
- Instance name
- Host name
- Number of users
    - Displays the number of users connected to each GAME node.
- Number of rooms
    - Displays the number of rooms created in each GAME node.
- Node status
- Action
    - Resume
        - Can start nodes of which status is Pause.
    - Pause
        - Stops the nodes of which status is Ready.

#### Filter

![monitoring_node_game_filter](https://static.toastoven.net/prod_gameanvil/images/monitoring_node_game_filter_en.png)

Specific nodes can be filtered using the filters below to all running GAME nodes.
- Node status
- Service name

### SUPPORT

Can monitor the SUPPORT nodes of which status is READY while running.

![monitoring_node_support](https://static.toastoven.net/prod_gameanvil/images/monitoring_node_support_en.png)

- Node ID
- Node type
- Service ID
- Service name
- Instance name
- Host name
- Node status
- Action
    - Resume
        - Can start nodes of which status is Pause.
    - Pause
        - Stops the nodes of which status is Ready.

#### 필터

![monitoring_node_support_filter](https://static.toastoven.net/prod_gameanvil/images/monitoring_node_support_filter_en.png)

Specific nodes can be filtered using the filters below to all running SUPPORT nodes.
- Node status
- Service name

### MATCH

Can monitor the MATCH nodes of which status is READY while running.

![monitoring_node_match](https://static.toastoven.net/prod_gameanvil/images/monitoring_node_match_en.png)

- Node ID
- Node type
- Instance name
- Host name
- Room Match Room Count
- User Match Queue Size
- Node status

#### Filter

![monitoring_node_match_filter](https://static.toastoven.net/prod_gameanvil/images/monitoring_node_match_filter_en.png)

Specific nodes can be filtered using the filters below to all running MATCH nodes.
- Node status
