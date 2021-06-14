## Game > GameAnvil > Console User Guide > Agent

Agent must be installed to run/stop/deploy the GameAnvil server through the console. 

### Download and Run Agent

* [Download Agent](https://static.toastoven.net/prod_gameanvil/files/gameanvil-agent-1.1.4.1.tar)

* The downloaded tar file contains the following files when it is uncompressed.

  * File configuration

    | File name                           | Description                      | Note |
    | ----------------------------------- | ------------------------- | ---- |
    | gameanvil-agent-*.jar ( *is the name of a version) | gameanvil-agent file      |      |
    | gameanvil-agent.properties          | gameanvil-agent configuration file |      |
    | start.sh                            | Run script             |      |
    | stop.sh                             | Stop script        |      |

    

* Change the GameAnvil-Agent configuration file

  * Check server.address and server.port.

    | Configuration name      | Description                  | Example                          | Note                                                         |
    | -------------- | --------------------- | ----------------------------- | ------------------------------------------------------------ |
    | server.address | IP used by Agent   | server.address=10.160.194.108 | If the setting value is left empty, all the IPs assigned to the machine can be accessed. Therefore, the IP to be used must be specified. |
    | server.port    | Port used by Agent | server.port=19080             | The value must be the same as the GameAnvil Agent Port configured on the console (Default value: 19080). |

    

* Run GameAnvil-Agent using the start.sh script.

  * Use the chmod command to set appropriate permissions if there is no permission for the script.
