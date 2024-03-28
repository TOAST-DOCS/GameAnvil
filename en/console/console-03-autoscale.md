## Game > GameAnvil > Console User guide > Autoscale

## Auto scale

You can create and manage autoscales from the **Autoscale Group** tab. Autoscale groups automatically scale-out/-in servers within the group based on the conditions you specify.

![Figure](https://static.toastoven.net/prod_gameanvil/images/console/autoscale/autoscale-group.png)

You can create a new autoscale group by clicking **+ Create Autoscale Group **. At this time, it provides a number of settings that affect the behavior of autoscales.

![Figure](https://static.toastoven.net/prod_gameanvil/images/console/autoscale/autoscale-param1.png)

As with server creation, you predefine what configuration information (Config) is used to create servers within an autoscale group. You can also define the instance type and many of the factors on which scaling up and down is based.

![Figure](https://static.toastoven.net/prod_gameanvil/images/console/autoscale/autoscale-param2.png)

Values for these scale-out/-in conditions can be combined as AND/OR operations. At this time, you can set up hardware resources such as CPU and memory, as well as logical resources such as the number of game users.

Once the autoscale group is created, you can check the list as follows. Also, just like creating a server, you can click each item to see the details of the autoscale group created. 
![Figure](https://static.toastoven.net/prod_gameanvil/images/console/autoscale/autoscale-created.png)

The remainder of this article explores the configuration items used in the creation of an autoscale group.

## Create Auto scale group

When creating an autoscale group, you must enter the following information.

* Group Name: Enter a unique name for the autoscale group.
* Memo: Enter a note about the autoscale group.
* Minimum servers: The minimum number of instances that the autoscale group can reduce to the maximum.
* Maximum servers: The maximum number of instances that an autoscale group can grow to.
* Start server: The number of instances that the autoscale group will initially start.

## Scale Out/In Policy

It sets the scale-out/-in policy for autoscale group.

* Scale Unit: Determines how many instances you want to add at a time when you need to scale-out/-in.
* Reuse wait time (in seconds): Guarantees that after a scale-up/down is executed, the next scale-up/down will not occur within the corresponding reuse wait (even if the condition is met).
* Condition operator: Multiple conditions can be combined as AND/OR operations.
* Autoscale condition: Autoscale scale-out/-in conditions can be set. You can combine multiple conditions by clicking **+**.

## Node configuration

The node configuration of an autoscale group is partially different from that of the normal server configuration. The biggest difference is that autoscale groups can only be created with a server composed of a single node. At this time, only two nodes can be selected, GATEWAY and GAME. SUPPORT and MATCH do not yet support autoscale.

When configured as a single node, different nodes cannot be configured as a single server. This means that if a service needs to be configured like a game node or a support node, it needs to be configured as a single service. In the example image described above, three game nodes for the 'RPSGame' service consist of one server. If you select one node, you can no longer select the rest.

When node configuration is completed with an autoscale group, scale-out/-in is automatically run based on the policy set earlier.