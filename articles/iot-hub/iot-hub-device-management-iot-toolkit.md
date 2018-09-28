---
title: Azure IoT device management with Azure IoT Toolkit extension for Visual Studio Code | Microsoft Docs
description: Use the Azure IoT Toolkit extension for Visual Studio Code for Azure IoT Hub device management, featuring the Direct methods and the Twin's desired properties management options.
author: formulahendry
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 8/3/2018
ms.author: junhan
---

# Use Azure IoT Toolkit extension for Visual Studio Code for Azure IoT Hub device management

![End-to-end diagram](media/iot-hub-get-started-e2e-diagram/2.png)

[Azure IoT Toolkit](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-toolkit) is a useful Visual Studio Code extension that makes IoT Hub management easier. It comes with management options that you can use to perform various tasks.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

| Management option          | Task                                                                                                                            |
|----------------------------|---------------------------------------------------------------------------------------------------------------------------------|
| Direct methods             | Make a device act such as starting or stopping sending messages or rebooting the device.                                        |
| Read device twin           | Get the reported state of a device. For example, the device reports the LED is blinking now.                                    |
| Update device twin         | Put a device into certain states, such as setting an LED to green or setting the telemetry send interval to 30 minutes.         |
| Cloud-to-device messages   | Send notifications to a device. For example, "It is very likely to rain today. Don't forget to bring an umbrella."              |

For more detailed explanation on the differences and guidance on using these options, see [Device-to-cloud communication guidance](iot-hub-devguide-d2c-guidance.md) and [Cloud-to-device communication guidance](iot-hub-devguide-c2d-guidance.md).

Device twins are JSON documents that store device state information (metadata, configurations, and conditions). IoT Hub persists a device twin for each device that connects to it. For more information about device twins, see [Get started with device twins](iot-hub-node-node-twin-getstarted.md).

## What you learn

You learn using Azure IoT Toolkit extension for Visual Studio Code with various management options on your development machine.

## What you do

Run Azure IoT Toolkit extension for Visual Studio Code with various management options.

## What you need

- An active Azure subscription.
- An Azure IoT hub under your subscription.
- [Visual Studio Code](https://code.visualstudio.com/)
- [Azure IoT Toolkit](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-toolkit)

## Sign in to access your IoT hub

1. In **Explorer** view of VS Code, expand **Azure IoT Hub Devices** section in the bottom left corner.
1. Click **Select IoT Hub** in context menu.
1. A pop-up will show in the bottom right corner to let you sign in to Azure for the first time.
1. After you sign in, your Azure Subscription list will be shown, then select Azure Subscription and IoT Hub.
1. The device list will be shown in **Azure IoT Hub Devices** tab in a few seconds.

   > [!Note]
   > You can also complete the set up by choosing **Set IoT Hub Connection String**. Enter the connection string for the IoT hub that your IoT device connects to in the pop-up window.

## Direct methods

1. Right-click your device and select **Invoke Direct Method**. 
1. Enter the method name and payload in input box.
1. Results will be shown in **OUTPUT** > **Azure IoT Toolkit** view.

## Read device twin

1. Right-click your device and select **Edit Device Twin**. 
1. An **azure-iot-device-twin.json** file will be opened with the content of device twin.

## Update device twin

1. Make some edits of **tags** or **properties.desired** field.
1. Right-click on the **azure-iot-device-twin.json** file.
1. Select **Update Device Twin** to update the device twin.

## Send cloud-to-device messages

To send a message from your IoT hub to your device, follow these steps:
 
1. Right-click your device and select **Send C2D Message to Device**. 
1. Enter the message in input box.
1. Results will be shown in **OUTPUT** > **Azure IoT Toolkit** view.

## Next steps

You've learned how to use Azure IoT Toolkit extension for Visual Studio Code with various management options.

[!INCLUDE [iot-hub-get-started-next-steps](../../includes/iot-hub-get-started-next-steps.md)]
