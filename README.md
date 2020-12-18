# azure-mbed-cli-example
Repo for the CLI based [example of connecting an Mbed board to Azure](https://github.com/ARMmbed/mbed-os-example-for-azure).
System: Ubuntu 20.04 

First, let's get the repo with the example code for connecting to Azure using a primary connection string:
```
git clone https://github.com/ARMmbed/mbed-os-example-for-azure.git
cd mbed-os-example-for-azure
mbed deploy
```

Next, we want to install the Azure CLI (https://docs.microsoft.com/en-us/cli/azure/install-azure-cli). Afterwards, we run the following command which will output a link for us to log in from our browser. This is the first and only time that we have to leave the terminal for the rest to work. 
```
az login
```

The default `az` tool does not have the IoT extension out of the box therefore we need to install that next:
```
az extension add --name azure-iot
```

Now, for the next few commands there are a set of named variables that we must pass to the Azure CLI tool. For example, we will call commands such as `az group create -l eastus -n MbedDemoGroup` where `MbedDemoGroup` is the resource group where we wil put the rest of the components. Since I already know in advanced what these variables mean, we could simply define them in the terminal to reference it easily throughout the example, this will also make it easier to automate this process later on with a bash or Python script. For example, we can do `export LOCATION=eastus` so that whenever we need to pass the location to an `az` command we can just reference the `LOCATION` variable like so `az group create -l ${LOCATION} -n MbedDemoGroup`. 

Setting tha value for the region of our server (picked from a fixed list) , the resource group (whatever value we chose to name this group), IoT Hub and edge device (the names of our Hub and unique device).

`export LOCATION=eastus`

`export GROUP=MbedGroup`

`export HUB=MbedHub`

`export DEVICE=mbedEdgeDevice`

Now, we run the following command to find out our subscription ID from our account:

`az account list --output table`

```
Name       CloudName  SubscriptionID       State   IsDefault 
Free Trial AzureCloud axxxxx-xxxxx-xxxx-xx Enabled True
```
So that we can set the variable below as follows:

`export SUBSCRIPTION=axxxxx-xxxxx-xxxx-xx`

With that out of the way, we can now we run the following:

`az group create -l ${LOCATION} -n ${GROUP}`

`az iot hub create --name ${HUB} --resource-group ${GROUP} -l ${LOCATION} --sku F1 --partion-count 2 --subscription ${SUBSCRIPTION}`

The following command outputs some valuable information as a JSON blob. Depending on the application, it might be a good idea to store these values.

`az iot hub device-identity create -n ${HUB} -d ${DEVICE} --edge-enabled --status enabled`

Finally, we run this command to find the primary connection string of our device:

`az iot hub device-identity connection-string show --device-id ${DEVICE} --hub-name ${HUB}`

```
{
  "connectionString": "HostName=xxxxx-xxxx-xxxx-xxx..."
}
```

At this point, we can just follow the steps from the [original example guide](https://github.com/ARMmbed/mbed-os-example-for-azure) starting from the second step. First, copy the string `HostName=xxxxx-xxxx-xxxx-xxx...` and paste it in the [azure cloud credentials file](https://github.com/ARMmbed/mbed-os-example-for-azure/blob/master/azure_cloud_credentials.h). 

Open the mbed_app.json file and enter your Wi-Fi credentials.

Finally, compile the code and flash your deivce (in my case a DISCO_L475VG_IOT01A):

`mbed compile -m DISCO_L475VG_IOT01A -t GCC_ARM -f --sterm --baud 115200`

If you go on the cloud console monitor, you should observe 10 messages received. 



