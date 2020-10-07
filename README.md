# Run IoT Edge Runtime in Windows Server Virtual Machine


### Prerequisites

1. Azure Subscription
2. Windows Server Virtual Machine [Azure CLI Commands](https://docs.microsoft.com/en-us/azure/iot-edge/how-to-install-iot-edge-windows-server-vm#deploy-from-azure-cli)
3. Azure IoT Hub [Azure CLI Commands](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-create-using-cli#create-an-iot-hub)
4. Device Provisioning Service [Azure CLI Commands](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-create-using-cli#create-an-iot-hub)

### Setup Azure IoT Edge Runtime in Virtula Machine

*Use PowerShell to run following scripts*

1. Deploy IoT Edge in VM

```

az vm run-command invoke `
    -g <resourcegroup name> `
    -n <vm name> `
    --command-id RunPowerShellScript `
    --script ". {Invoke-WebRequest -useb https://aka.ms/iotedge-win} | Invoke-Expression; Deploy-IoTEdge"


```

2. Dervice device to autoprovision IoT Edge Runtime

```
$KEY='<IoT Hub's SharedAccessKey>'
$REG_ID='<deviceId>'

$hmacsha256 = New-Object System.Security.Cryptography.HMACSHA256
$hmacsha256.key = [Convert]::FromBase64String($KEY)
$sig = $hmacsha256.ComputeHash([Text.Encoding]::ASCII.GetBytes($REG_ID))
$derivedkey = [Convert]::ToBase64String($sig)
echo "`n$derivedkey`n"

```

3. Initialize IoT Edge

```
az vm run-command invoke `
    -g <resourcegroup name> `
    -n <vm name> `
    --command-id RunPowerShellScript `
      --script ". {Invoke-WebRequest -useb https://aka.ms/iotedge-win} | Invoke-Expression; Initialize-IoTEdge -Dps -ScopeId <dpsscopeId> -RegistrationId <deviceid> -SymmetricKey <derived device key>"
```
4. Verification

> Run to add iotedge in environment variable
``` 
$Env:Path += ";C:\Program Files\iotedge"
```
> Command to list modules deployed 

```
iotedge list
```
> Command to run configuration check

```
iotedge Check
```
> Command to get logs from the module

```
iotedge logs <replace with modulename>
```
> Monitor events from the device

```
az iot hub monitor-events -n <iothub name> -d <deviceId>
```
### Deployment

Command to deploy modules on a device

```
az iot edge set-modules --device-id <deviceid> --hub-name <iothubname> --content "<deployment file path>"
```
