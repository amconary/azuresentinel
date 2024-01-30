<h1>Azure Sentinel Lab</h1>

 <h2>Description</h2>
Project consists of configuring an Azure lab environment with Sentinel using a custom log to map failed logon attempts based on geolocation.
<br />


<h2>Languages and Utilities Used</h2>

- <b>Event Viewer</b>
- <b>Log Analytics</b> 
- <b>Sentinel</b>
- <b>Defender</b>
- <b>Powershell</b>
- <b>KQL Query</b>
- <b>Geolocation API</b>

<h2>Environments Used </h2>

- <b>Windows 10 VM</b>
- <b>Azure</b>

<h2>Project walk-through:</h2>

<p align="center">
Sentinel Lab 
<br />
<br />
Sign into the Azure portal, then we will create a virtual machine to start. Create a new resource group using the settings below as a guide for free tier. <br />
<img src="https://i.imgur.com/yr3rNKA.png" height="60%" width="60%" alt="Sentinellab"/>
<br />
<br />
Next go to the Networking tab and change network security group to advanced and create a new security group. Remove the default inbound rule so we can create our own for testing purposes. <br />
Set the rules as follows to allow the VM to be in a vulnerable state:  <br/>
<img src="https://i.imgur.com/wyXzbuD.png" height="60%" width="60%" alt="Sentinellab"/> <br />
Click on review and create, then create to finish the setup of the VM. <br />
<br />
<br />
Next we will move on to a log analytics workspace. We will create this workspace to help ingest the logs so we can discover where attacks are coming from. <br />
Connect it to the resource group we setup previously:  <br/>
<img src="https://i.imgur.com/tD8uTxN.png" height="60%" width="60%" alt="Sentinellab"/>
<br />
<br />
Now we will go to Microsoft Defender for Cloud > Environment settings and then choose the Log Analytics workspace we just created to enable the Defender server: <br/>
<img src="https://i.imgur.com/ctsuMt0.png" height="60%" width="60%" alt="Sentinellab"/>
<br />
<br />
Once the Defender server is complete, under Data Collection select All Events. <br />
Swap back over to Log Analytics and we are going to connect it to our VM we created earlier. <br /> 
<img src="https://i.imgur.com/U6whEuY.png" height="60%" width="60%" alt="Sentinellab"/>
<br />
<br />
Next we will setup Microsoft Sentinel. Move to the Sentinel Page > Create > choose are Analytics workspace to add it to Sentinel. <br />
From here we will remote into the Azure VM using RDP to log in with the public IP. <br />
Once logged into the Azure VM we will start up event viewer so we can view event log info, gather the IP address of any attack attempts, which we will use to help build a custom log that we can feed to the SIEM. <br />
<img src="https://i.imgur.com/Lm2etoB.png" height="60%" width="60%" alt="Sentinellab"/>
<br />
<br />
For this lab we are going to use the custom Powershell log exporter script created by Josh Makador and a geolocation API to help build the custom logs for the SIEM. It will look through the logs and pull all the events for 4625 - failed logon attempts and grab their geolocation info. <br />
<img src="https://i.imgur.com/3oWGcmU.png" height="60%" width="60%" alt="Sentinellab"/>
<br />
<br />
Save and run the following Powershell script to start the collection process: https://github.com/joshmadakor1/Sentinel-Lab/blob/main/Custom_Security_Log_Exporter.ps1 <br />
We will now return back to our Log Analytics workspace and we are going to build the custom log. Go to Log analytics > Tables > Create> New Custom Log ( MMA-Based). <br />
We are going to need the log info from our VM since its not located on our host. Swap over to the VM and open our log file that was created by our Powershell script, copy all the contents of the log file. <br />
<img src="https://i.imgur.com/imU28yQ.png" height="60%" width="60%" alt="Sentinellab"/>
<br />
<br />
Create a text file on your host machine with this data which we will upload to Azure to help train our SIEM. Once its uploaded, move to Collection Path, here you will set the path to the log file that matches the location on our VM. The custom log collector is now complete. <br />
We can now move back to the Log Analytics workspace, click Logs and run our query for failed events. It may take some time to populate the data. Once it pupulates you should see all the results for failed logon attempts: <br />
<img src="https://i.imgur.com/VEpN8aO.png" height="60%" width="60%" alt="Sentinellab"/>
<br />
<br />
Now that we have results in our logs we are going to extract info from the raw data to create custom columns based on the fields we want to see. To do that we are going to modify our query using a KQL regex to split the data up into the proper fields: <br />
<img src="https://i.imgur.com/YRzNUzh.png" height="60%" width="60%" alt="Sentinellab"/>
<br />
<br />
Next will we go over to Sentinel and use the data to create an attack map to visualize where these attacks are coming from. Sentinel > Workbook > Add workbook: <br />
<img src="https://i.imgur.com/SRsqlyc.png" height="60%" width="60%" alt="Sentinellab"/>
<br />
<br />
Here we will utilize the KQL query again followed by a summarize query which will help us group together the like events based on the column fields that are being extracted from the data: <br />
<img src="https://i.imgur.com/vqTT69p.png" height="60%" width="60%" alt="Sentinellab"/>
<br />
<br />
Set you Visualization to “Map” so it can plot out the points using the latitude and longitude fields: <br />
<img src="https://i.imgur.com/K1Li3Uh.png" height="60%" width="60%" alt="Sentinellab"/>
<br />
<br />
Save the Workbook and watch as the attack attempts get plotted to the map based on the data we are ingesting from the event logs. <br />
<br />
<br />
</p>

<!--
 ```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```
--!>
