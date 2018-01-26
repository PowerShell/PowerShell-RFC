---
RFC: RFC
Author: Tyler Leonhardt
Status: Draft
SupercededBy: N/A
Version: 1.0
Area: External PowerShell Module / PowerShell IoT
Comments Due: Feb 26th, 2018
Plan to implement: Yes
---

# Azure IoT PowerShell Device SDK

This RFC is about the creation of a PowerShell device SDK for Azure IoT to interact with Azure IoT hubs. [Work has already been started](https://github.com/tylerl0706/azure-iot-sdk-powershell) on the barebones PowerShell SDK and your comments are much appriciated.

### Information about Azure IoT

Azure IoT's infrastructure can be (highly) simplified to 3 boxes:

[Your Cloud Service]   <---->   [Azure IoT Hub]   <---->   [Your Device (ex. Raspberry Pi)]

You own the cloud service and the device logic. Azure IoT Hub is the message bus between the two. You can use Azure IoT's SDK's to interact with the Azure IoT Hub.

Messages to the cloud from the device are typically referred to as telemetry events. Messages from the cloud to the device are using "Direct Methods" which are functions defined on the device that get invoked when a message comes in (to the device from the cloud) with a particular key (ex. we could define that "myfunct" calls the "run-myfunct" function on the device).

There are two types of SDK's for Azure IoT: "Device SDK" and "Service SDK". "Service SDK" is typically used in your cloud service that handles and sends events to the device, and the "Device SDK" is used in the device logic to handle and send events to the cloud service. 

Azure IoT has several device SDK's for a [variety of languages](https://github.com/Azure/azure-iot-sdks#microsoft-azure-iot-sdks-1). There is a [.NET device SDK](https://github.com/Azure/azure-iot-sdk-csharp/tree/master/device), so we should have a PowerShell module that uses this.

For more reference, here are the docs for Azure IoT:
* [Azure IoT Hub](https://docs.microsoft.com/en-us/azure/iot-hub/) - The barebones service. Think of it like an event hub in a way
* [Azure IoT Suite](https://docs.microsoft.com/en-us/azure/iot-suite/) - A "preconfigured solution" that includes an instance of Azure IoT Hub and other services with everything set up out of the box.

## Motivation

The PowerShell team is investigating IoT scenarios for PowerShell.

This module will be separate to a PowerShell IoT module (described in a separate RFC - linked to later), but we want to enable scenario that will allow folks to use PowerShell IoT with Azure.

In order to enable that, we need a PowerShell module that will wrap the .NET device SDK.
The SDK caters to C# and we want to give users of PowerShell and PowerShell IoT a "PowerShell-y" experience when interacting with Azure IoT so they can iterate fast with a familiar interface.

## Specification

As mentioned, [Work has already been started](https://github.com/tylerl0706/azure-iot-sdk-powershell). Here are the cmdlet interfaces we want to ship initially:

### Connecting to Azure IoT Hub
```powershell
$ConnectionString = "<your conn string>"
Connect-AzureIoTDevice -ConnectionString $ConnectionString
```

### Setting Azure IoT Reported Properties (the device config/metadata, essentially)
```powershell
$reportedPropertiesObj = @{
    MyData = "Hello World"
}
Set-IoTDeviceReportedProperties -ReportedProperties $reportedPropertiesObj
```

### Sending a message from the device to the Azure IoT Hub

```powershell
Invoke-IoTDeviceEvent -Message "Hello World"
```
NOTE: `Connect-AzureIoTDevice` returns a `DeviceClient` object which can optionally be passed in to `Set-IoTDeviceReportedProperties` or `Invoke-IoTDeviceEvent` to specify the device client you want to use. If it is not passed in, the cmdlets will use the last initialized device client.

### Handling Direct Method invocations (a bit complicated to implement)

The Direct Method portion of the Azure IoT device SDK is asynchronous and, in C#, calls a delegate (that you define on the device) when the name of the function comes in from the server.

To make it more PowerShell-y, we want to be able to run PowerShell functions, rather than having the user define C# delegates. Because of the nature of async Tasks, delegates get run on a separate thread and since PowerShell is single threaded and doesn't quite support Tasks simply, we must wrap user defined PowerShell functions in a delegate that can be run on a separate thread that will create a new runspace to execute the PowerShell in.

To define these functions, a user can create a PowerShell module like so:
```powershell
function Get-AzureIoTManifest {
    return @{
        hello = "Get-HelloWorld"
        echo = "Get-Echo"
    }
}

function Get-HelloWorld ($Request) {
    return @{
        Hello = "World"
    }
}

function Get-Echo ($Request) {
    return @{
        Data = $Request
    }
}

Export-ModuleMember -Function *-*
```

The PowerShell module must implement a function called `Get-AzureIoTManifest` which will return an object that maps the Direct Method name to the function you want to run when that message is received. Under the hood, when a message comes in, it will create a new runspace, import the module and run the desired function.

To tell the Azure IoT device client that you're using a particular module to handle Direct Methods, you run this:

```powershell
Set-AzureIoTDeviceDirectMethod -Module (Resolve-Path ./ExampleDirectMethodModule.psm1).Path
# or if it exists in PSModulePath
Set-AzureIoTDeviceDirectMethod -Module ExampleDirectMethodModule
```

## Alternate Proposals and Considerations

### A module that already existed

Initial search yielded this module in the PowerShell Gallery: https://github.com/markscholman/AzureIoT

However, we decided against it for a few reasons:
* It's been a few months since the last commit
* It didn't work on PowerShell Core
* It didn't implement Direct Methods

### Direct Method implementation

Initial implementation of Direct Methods in PowerShell ran arbitrary script blocks like so:

```powershell
Set-AzureIoTDeviceDirectMethod -Name Foo -ScriptBlock {
    param($Request)
    return "Hello World"
}
```

This is good in simple cases, but does not scale because of scope issues. Since the script would be run on a separate runspace, you must import everything you need inside of the script block and in EVERY script block. This was a huge disadvantage as we expect users will need additional modules, like PowerShell IoT, in each script block.

