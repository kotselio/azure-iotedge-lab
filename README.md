# Lab introduction 
This instructional hands on lab is just for education purposes and it is a simple E2E scenario to get the audience familiar with the Azure IoT Edge components. It is an assembly of Azure docs and tutorials and the references are provided throughout the document. By the end of each task we will be providing some pointers for conecpts that can be discussed with the audience, either as a wrap up, or an opening and background information before they complete the task.

## Architecture Overview

A sensor (simulator edge module) is connected to an Edge device. This Edge device is an Ubuntu Virtual Machine that is deployed on Azure. The telemetry of this sensor collects four readings: ambient temperature, ambient pressure, machine temperature and machine pressure. This telemetry is collected by the Edge and it is then sent to IoT Hub, where the Edge device is registered. We can always read messages from IoT Hub using the Device Explorer tool from our workstation. 

Analytics (Cloud): The telemetry messages are being processed in the cloud by Stream Analytics, where we check if the ambient temperature value is over a threshold. If the ambient temperature is too high, the message is pushed to a Service Bus Queue, is being picked up by a Logic App and we send an email notification with a “high ambient temperature” alert. 

Analytics (Edge): We deploy a Stream Analytics Job module, via staging it to a Storage Account and then deploying it on the Edge device. This job is checking if the average machine temperature is above a threshold for more than 30 seconds. If yes, then an alert is sent to IoT Hub and the module resets the simulated machine. 

![alt text](/images/architecture-overview.png "Architecture Overview")

## Task 1: Create an Ubuntu Server 18.04 TLS and install the Azure IoT Edge runtime (approx. 30 mins)
In this task you will create a Virtual Machine to host your IoT Edge deployment. You will connect remotely and install the IoT Edge runtime on your Virtual Machine and connect it to your IoT Hub.

![alt text](/images/azure-iotedge-lab-task1.png "Architecture overview of Task 1")

### Subtask 01: Create virtual machine (Azure Portal)
1. Choose **Create a resource** in the upper left corner of the Azure portal
2. Search for **Ubuntu Server 18.04 LTS** and click **Create**
3. In the **Basics** tab, under **Project details**, make sure the correct subscription is selected and then choose **Create new** under **Resource group** (select your resource group in case you already have one you want to use). Type *myEdgeRG* for the name of the resource group and then choose **OK**
4. Under **Instance details**, type *myEdgeVM* for the **Virtual machine name** and choose *West Europe* for your **Region** (feel free to choose for another region, and select that same region for all your resources). Click on the **Size** option, find and click on **B1ms** and click **Select**
5. Under **Administrator account** select **Password** and type a **username** and **password** (please make a note of the credentials, as we will be using them later).
6. Under **Inbound port rules > Public inbound ports**, choose **Allow selected ports** and then select **SSH (22)** and **HTTP (80)** and **HTTPS (443)** from the drop-down
7. On the bottom of the pane, click **Next: Disks >**
8. Under **Disk options** select **Standard SSD**
9. On the bottom of the pane, click **Next: Networking >** and leave the default configuration
10. On the bottom of the pane, click **Next: Management >** and under **Monitoring** turn the **OS guest diagnostics** to **OFF**
11. On the bottom of the pane, click **Review + create** and finally click on **Create**

### Subtask 02: Connect to the virtual machine (PuTTY / bash)
12. Download PuTTY from here: https://www.putty.org/ (skip if you use Linux-based systems and subsystems or Mac)
13. Navigate to your Resource Group on the Azure Portal and click on the virtual machine
14. On the **Overview** tab click on **Connect > SSH** and copy the **IP Address** of your VM
    * For PuTTY users: paste the IP Address on the Host Name field and click connect (select Yes in the Security Alert screen)
    * For Linux and Mac users: open a terminal and type:
        ``` bash
        ssh [username]@[IP Address]
        ```
15. You will be asked to give the password (you have set the password in step 5)

### Subtask 04: Add a new Edge device to IoT Hub (Azure Portal)
16. Navigate to your IoT Hub in the Azure Portal (we assume you have an IoT Hub instance up and running; if not follow these instructions: https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-create-through-portal)
17. Under **Automatic Device Management** click on **IoT Edge**
18. Click on **Add an IoT Edge Device**
19. In the field **Device ID** type *myUbuntuEdge* and leave the other fields to their default configuration (i.e. Symmetric keys, auto-generate keys, Enable) 
20. The Edge device is now listed. Click on the device’s name and note the **primary connection string**

### Subtask 05: Install IoT Edge runtime (PuTTY / bash)
21. Go to your terminal and while being logged in to your Edge VM, do the following
22. Follow the instructions of this article for **UBUNTU 18.04**: https://docs.microsoft.com/en-us/azure/iot-edge/how-to-install-iot-edge-linux
    * Install the latest runtime version
    * Install the container runtime
    * Install the Azure IoT Edge Security Daemon
    * Configure the security daemon using **Manual Provisioning** (on Step 20 you can copy the connection string and paste it in the `config.yaml` file as described in the article)
    * Verify successful installation, going to your Edge device in the Azure Portal and verify that you see the following (hit **Refresh**):
    ![alt text](/images/azure-iotedge-lab-step22.png "Verify IoT Edge successful installation")

END OF TASK 1
```
Lessons learned:
Resource Groups, Azure Regions, Virtual Machines, Edge devices VS leaf devices, IoT Edge concept, Edge patterns
```
## Task 2: Deploy a custom module and read telemetry (approx. 15 mins)
You will deploy a simulated sensor module on the Edge, which will generate data (telemetry). You will define the deployment of your IoT Edge via the IoT Hub. Each module is a container. You will use the Device Explorer to read all the telemetry messages sent by your IoT Edge.

![alt text](/images/azure-iotedge-lab-task2.png "Architecrture overview of Task 2")

### Subtask 06: Add a new module (Azure Portal)
23. On the screen of the previous step (22) click on **Set Modules** and under **Deployment modules** click on **Add** and select **IoT Edge Module**
24.	Under the Name field type *SimulatedTemperatureSensor* and under the Image URI type 
    ```
    mcr.microsoft.com/azureiotedge-simulated-temperature-sensor:1.0
    ```
25.	Let the other fields to the default values and click on **Add** and then **Next**, **Next** and **Create**

### Subtask 07: Verify deployment on Edge (PuTTY / bash)
26.	On the Edge virtual machine type: `sudo iotedge list`

    You should be seeing the following:
    ![alt text](/images/azure-iotedge-lab-step26.png "IoT Edge list of modules")
27.	Type: `sudo docker logs -f SimulatedTemperatureSensor`

    You should be seeing the following:
    ![alt text](/images/azure-iotedge-lab-step27.png "IoT Edge telemetry logs")

    **_TIP: you can stop/start/restart the module by typing:_**

    `sudo docker [start OR stop OR restart] SimulatedTemperatureSensor`

### Subtask 08: See telemetry in Device Explorer (Azure Portal & Device Explorer)
28.	Use the device explorer download from [here](https://github.com/Azure/azure-iot-sdk-csharp/releases/download/2019-9-11/SetupDeviceExplorer.msi)
29.	If you have a non-Windows workstation, or just like IDEs more, you could also use [VS Code](https://code.visualstudio.com/download) and the [Azure IoT Toolkit](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)’s device explorer. Check step 30 to get the connection string and steps 31 and 32 to use the created consumer groups (*explorer*, *streamanalytics*) and use *Start Monitoring D2C Message*. Alternatively, you could also use the [Azure IoT Explorer](https://github.com/Azure/azure-iot-explorer)
30.	Navigate to your **IoT Hub > Shared access policies** and click on **iothubowner**. Under **Shared access keys** copy the **Connection string—primary key**
31.	Navigate to **IoT Hub > Built-in endpoints**. Under **Events > Consumer Groups** type *explorer* inside the “**Create new consumer group**” text box
32.	Add another consumer group named *streamanalytics*. We will be using this later in Task 3
33.	Go to your Device Explorer and paste the connection string under **IoT Hub Connection String** and click **Update**
34.	On Device Explorer under the **Data** tab, select your device ID as *myUbuntuEdge*, select **Enable** next to **Consumer Group** and type *explorer*. Then click on **Monitor**. You should be seeing the following:
    ![alt text](/images/azure-iotedge-lab-step34.png "Telemetry on Device Explorer")

END OF TASK 2
```
Lessons learned:
Dev tools, Consumer groups, Edge modules (containers), IoT Hub policies
```

## Task 3: Use business logic to capture “interesting events” and take action (approx. 30 mins)
You will create a business rule to process the messages coming from your IoT Hub, using Stream Analytics. If an incoming message fulfills the rule (e.g. temperature value is larger than the specified threshold) then this message is being put to a queue (Service Bus Queue). You will create an integration component (Logic App – an If-This-Then-That workflow designer) to check every minute if there are new messages in the queue. If there are new messages, the Logic App will send an email notification alert to a predefined email address.

![alt text](/images/azure-iotedge-lab-task3.png "Architecture overview of Task 3")

### Subtask 09: Create a Service Bus Queue to create a queue for our “alert” messages (Azure Portal)
35.	Choose **Create a resource** in the upper left corner of the Azure portal
36.	Search for **Service Bus** and click **Create**
37.	Under **Name** type your **_unique_** namespace (namespaces are unique universally): try something like your name and add “bus” like e.g. **mybus** or **iotbus9034c**
38.	Select *Standard* as your **Pricing Tier**. Make sure the correct subscription is selected and then choose *myEdgeRG* under **Resource group**. Select *West Europe* as your **Location** (or the region you have previously selected to deploy your other resources)
39.	Once the deployment succeeded, navigate to your Service Bus and click on **Add Queue**. Under **Name** type *alertsqueue* and leave the rest of the fields to their default values, i.e. **Max queue size** *1GB*, **Message time to live** *14 days*, **Lock duration** *30 seconds*. Leave all the other boxes unchecked. Click **Create**.
40.	You should be able to see the queue on the bottom of the pane:
    ![alt text](/images/azure-iotedge-lab-step40.png "Service Bus queue")
### Subtask 10: Create an integration solution with Logic Apps to act upon an alert (Azure Portal)
41.	Choose **Create a resource** in the upper left corner of the Azure portal
42.	Search for **Logic App** *(Note: not Logic Apps B2B)* and click **Create**
43.	Under **Name** type *alertsapp*. Make sure the correct subscription is selected and then choose *myEdgeRG* under **Resource group**. Select *West Europe* as your **Location** (or the region you have previously selected to deploy your other resources). Set the **Log Analytics** to *Off* and click **Create**
44.	Once the deployment succeeded, navigate to your Logic App and go to the **Logic App Designer** and select **Blanc Logic App**
45.	Search **Service Bus** and click on the Service Bus icon. Under the **Triggers** find and select *When a message is received in a queue (auto-complete)*. Under **Connection name** type *alertsb*. Under the **Service Bus Namespace** select your service bus name (e.g. *mybus*). Click again on the *RootManagerSharedKey* and click **Create**
46.	Under **Queue name** find and select the *alertsqueue*. Leave the **Queue type** to *Main*. Set the **Interval** to *1 minute* and click on **Next Step**
47.	Under **Choose an action** search for *Office 365 Outlook* (if you have an Outlook account) or *Gmail* (if you have a Gmail account, etc.) and click on the icon. Under **Actions** search and select Send an email. You will be asked to login to your account, so please follow the dialog and do so
48.	Fill in the form: **To**: *your email address*; **Subject**: *IoT Edge Alert* [Message Id]; **Body**: *Alert detected! Details:* [Content]. The configuration is depicted below:
    ![alt text](/images/azure-iotedge-lab-step48.png "Logic App notification email setup")
49.	Click on **Save**
### Subtask 11: Create a Stream Analytics Job to define business rules, raise alerts and “glue” everything together (Azure Portal)
50.	Choose **Create a resource** in the upper left corner of the Azure portal
51.	Search for **Stream Analytics job** and click **Create**
52.	Under **Job name** type *myiotjob*. Make sure the correct subscription is selected and then choose *myEdgeRG* under **Resource group**. Select *West Europe* as your **Location** (or the region you have previously selected to deploy your other resources). Leave the rest on the default configuration values: i.e. under **Hosting environment** choose *Cloud* and set the **Streaming units** to *3*. Then click **Create**
53.	Once the deployment succeeded, navigate to your Stream Analytics job and click on **Inputs**. Click on **Add stream input** and select *IoT Hub*. Under the **Input alias** type *myiothub*. Select your subscription and find your IoT Hub. Under **Endpoint** make sure you have selected *Messaging*, under **Shared access policy name** you have selected *iothubowner*, under **Consumer group** you have selected *streamanalytics* (if you don’t see it listed, please visit steps 31 and 32). Leave the rest of the files to their default configuration values (*JSON*, *UTF-8* and *None*) and click **Save**
54.	On the Stream Analytics pane click on the **Outputs**. Click on **Add** and select **Service Bus queue**. Under **Output alias** type *myalertsqueue*. Select your subscription and find your Service Bus. Under the **Queue name** select **Use existing** and find your *alertsqueue*. Leave all the other fields in their default configuration values (*RootManagerSharedKey*, *JSON*, *UTF-8*, *Line separated*) and click **Save**
55.	On the Stream Analytics pane click on the **Query**. Paste the following on the query editor (replace the existing text):
    ``` SQL
    SELECT
        *
    INTO
        [myalertsqueue]
    FROM
        [myiothub]
    WHERE
        ambient.temperature > 21
    ```
56.	Click on **Save query**
57.	On the Stream Analytics pane click on **Overview** and click on **Start**. Select **Now**. Note that it might take a while for the job to start. In them mean time make sure you are receiving messages in your IoT Hub from your IoT Edge device
58.	The emails you should be receiving look like this:
    ![alt text](/images/azure-iotedge-lab-step58.png "Alert email")

**_Tip: To stop the emails from coming click on the apertsapp logic app and click Disable._**

END OF TASK 3
```
Lessons learned:
Stream Analytics job start time – IoT Hub message retention, Logic Apps integration connectors (ITTT/Flow equivalent), Service Bus VS IoT Hub VS Event Hub, Cold / Hot path
```

## Task 4: Use the Intelligent Edge to process data without the cloud (approx. 40 mins)
You will deploy a Stream Analytics Job, but this time as a second IoT Edge module (container). This Job will be calculating the average temperature of a 30 minutes time window; if the average value exceeds the defined threshold, then the module does three things:

a.  Processes all the telemetry messages generated from the simulator module and calculates the average temperature within a 30-minutes window

b.	Sends a “reset” command to the machine (the simulator module will be playing that role and receiving and executing the command) when the threshold is exceeded

c.	Sends a “high average machine temperature alert” event message to your IoT Hub

For deploying the Stream Analytics Job on the IoT Edge, you will need a Storage Account to store the Stream Analytics Job container image, so that the IoT Edge can download it from there. You will define the deployment via your IoT Hub and you will be reading the messages of IoT Hub via the Device Explorer.

![alt text](/images/azure-iotedge-lab-task4.png "Architecture overview of Task 4")
### Subtask 12: Create a Storage Account to store your Stream Analytics container image (Azure Portal)
59.	Choose **Create a resource** in the upper left corner of the Azure portal
60.	Search for **Storage Account** and click **Create**
61.	In the **Basics** tab, under **Project details**, make sure the correct subscription is selected and then choose *myEdgeRG* under **Resource group**
62.	Under **Instance details** type *myiotdata* as your Storage account name, choose *West Europe* for your **Location** (or the region you have previously selected to deploy your other resources), choose *Standard* under **Performance**, choose *StorageV2* for the **Account type**, choose *LRS* for the **Replication**, and finally choose *Hot* for the **Access tier**. Click **Review + create**
63.	Click **Create**

### Subtask 13: Create a Stream Analytics Job deployment for the Edge (Azure Portal)
64.	Choose **Create a resource** in the upper left corner of the Azure portal
65.	Search for **Stream Analytics job** and click **Create**
66.	Under **Job name** type *myedgejob*. Make sure the correct subscription is selected and then choose *myEdgeRG* under **Resource group**. Select *West Europe* as your **Location** (or the region you have previously selected to deploy your other resources). Under **Hosting environment** choose *Edge*. Then click **Create**
67.	Once the deployment succeeded, navigate to your Stream Analytics job and under **Job topology** click on **Inputs**. Click on **Add stream input** and select *Edge Hub*. Under the **Input alias** type *temperature*. Leave all the other fields in their default configuration values (*JSON*, *UTF-8*, *None*) and click **Save**
68.	Under **Job topolog**y click on **Outputs**. Click on **Add** and select *Edge Hub*. Under the **Output** alias type *alert*. Leave all the other fields in their default configuration values (*JSON*, *UTF-8*, *None*) and click **Save**
69.	Under Job topology click on **Query**. Paste the following on the query editor (replace the existing text):
    ```sql
    SELECT
        'reset' AS command
    INTO 
        [alert]
    FROM
        [temperature] TIMESTAMP BY timeCreated 
    GROUP BY TumblingWindow(second,30)
    HAVING Avg(machine.temperature) > 70
    ```
70.	This SQL-like code sends a reset command to the alert output if the average machine temperature in a 30-second window reaches 70 degrees. The reset command has been pre-programmed into the sensor as an action that can be taken
71.	Click **Save query**
72.	To prepare your Stream Analytics job to be deployed on an IoT Edge device, you need to associate the job with a storage container in a storage account. When you go to deploy your job, the job definition is exported to the storage container. On your Stream Analytics job pane, under **Configure**, select **Storage account settings** and click **Add storage account**. Choose **Select storage account from your subscriptions**, make sure you select your subscription, find and select your storage account *myiotdata*. Under **Container** select **Create new** and type *myedgejobdefinition*. Click **Save**
### Subtask 14: Configure and deploy the Stream Analytics Job module to your Edge device (Azure Portal)
In this subtask, you use the **Set Modules** wizard in the Azure portal to create a deployment manifest. A deployment manifest is a JSON file that describes all the modules that will be deployed to a device, the container registries that host the module images, how the modules should be managed, and how the modules can communicate with each other. Your IoT Edge device retrieves its deployment manifest from IoT Hub, then uses the information in it to deploy and configure all its assigned modules.

By the end, you will have two modules deployed. The first is **SimulatedTemperatureSensor** (already deployed in Task 2), which is a module that simulates a temperature and humidity sensor. The second is your Stream Analytics job module. The sensor module provides the stream of data that your job query will analyze.

73.	Follow the instructions documented here: https://docs.microsoft.com/en-us/azure/iot-edge/tutorial-deploy-stream-analytics#deploy-the-job
74.	Navigate to your IoT Hub, select IoT Edge and click on the name of your IoT Edge device. Return to the device details page, and then select **Refresh**.

    You should see the new Stream Analytics module running, along with the IoT Edge agent and IoT Edge hub modules. It may take a few minutes for the information to reach your IoT Edge device, and then for the new modules to start. If you don't see the modules running right away, continue refreshing the page.

    ![alt text](/images/azure-iotedge-lab-step74.png "IoT Edge modules portal")    
### Subtask 15: Verify your deployment and view data and command execution (PuTTY / bash)
75.	Login to your Edge virtual machine again if needed, and check all modules are running, typing:

    ```iotedge list```

    You should be seeing the following:
    ![alt text](/images/azure-iotedge-lab-step75.png "IoT Edge modules list")
76.	View all system logs and metrics data, typing:

    ```sudo iotedge logs -f myedgejob```

    You should be seeing the following:
    ![alt text](/images/azure-iotedge-lab-step76.png "IoT Edge myedgejob module logs")
77.	View the reset command affect the *SimulatedTemperatureSensor* by viewing the sensor logs, typing:

    ```sudo iotedge logs SimulatedTemperatureSensor```
78.	Once the temperature has is gradually increased to 70 degrees for 30 seconds (you would have to wait for a few minutes) you should see the following alert:
    ![alt text](/images/azure-iotedge-lab-step78.png "IoT Edge SimulatedTemperatureSensor module logs reset")
### Subtask 16: View data sent to your IoT Hub (Device Explorer)
79.	Monitoring the messages on your explorer you should be able to see telemetry messages and reset alerts that were generated by the Stream Analytics Job *myedgejob*
    ![alt text](/images/azure-iotedge-lab-step79.png "IoT Edge view data")
    Once the command reset is executed you see that the temperature has dropped again.

END OF TASK 4
```
Lessons learned:
Intelligent Edge VS Gateway, feedback loop, EdgeHub and EdgeAgent, IoT Edge Routes, Time window operations in processing streams
```

## Task 5: Clean up resources
If you wish to permanently delete all the resources:

80.	Navigate to your resource group, where you deployed the Edge VM and the other resources and click **Delete resource group**. Type the name of the resource group and click **Delete**.

If you wish to pause individual resources:

81.	Navigate to your *myEdgeVM* Edge Virtual Machine and click **Stop**.
82.	Navigate to your *myiotjob* Stream Analytics Job (cloud deployment) and click **Stop**.
83.	Navigate to your *alertsapp* Logic App and click **Disable**.
84.	Please note that the storage account and your IoT Hub cannot be paused, so *charges will continue to apply*.

```
Lessons learned:
Cost-driven architecture, Development & Production environments
```

## Resources
### Task 1 Resources
Quickstart: Create a Linux virtual machine in the Azure portal: https://docs.microsoft.com/en-us/azure/virtual-machines/linux/quick-create-portal

Install the Azure IoT Edge runtime on Debian-based Linux systems: https://docs.microsoft.com/en-us/azure/iot-edge/how-to-install-iot-edge-linux

### Task 2 Resources
Deploy Azure IoT Edge modules from the Azure portal: https://docs.microsoft.com/en-us/azure/iot-edge/how-to-deploy-modules-portal

### Task 3 Resources
IoT remote monitoring and notifications with Azure Logic Apps connecting your IoT hub and mailbox: https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-monitoring-notifications-with-azure-logic-apps

Get started with Azure Stream Analytics to process data from IoT devices: https://docs.microsoft.com/en-us/azure/stream-analytics/stream-analytics-get-started-with-azure-stream-analytics-to-process-data-from-iot-devices 

### Task 4 Resources
Tutorial: Deploy Azure Stream Analytics as an IoT Edge module: https://docs.microsoft.com/en-us/azure/iot-edge/tutorial-deploy-stream-analytics