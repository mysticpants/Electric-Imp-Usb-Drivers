# UartOverUsbDriver

The UartOverUsbDriver class creates an interface object that exposes methods similar to the uart object to provide compatability for uart drivers over usb.

### Setup

**To add this library to your project, add** `#require "uartoverusb.device.nut:1.0.0"` **to the top of your device code**

This class requires the UsbHost class. The usb host will handle the connection and instantiation of this class. The class and its identifiers must be registered with the UsbHost and the device driver will be passed to the connection callback on the UsbHost. 

#### Example
The example shows how to use the Brother QL-720NW uart driver to demonstrate how to use UartOverUsb. Please get the QL720NW.device.nut file from [here](https://github.com/electricimp/QL720NW) and paste the class at the top of your code.
```squirrel
#require "usbhost.device.nut:1.0.0"
#require "uartoverusb.device.nut:1.0.0"

class QL720NW {...}

usbHost <- UsbHost(hardware.usb);

// Register the Uart over Usb driver with usb host
usbHost.registerDriver(UartOverUsbDriver, UartOverUsbDriver.getIdentifiers());

// To use a device that is not directly supported by the UartOverUsb you can manually pass in identifiers using the following code
// vid <- [enter device vid];
// pid <- [enter device pid];
// identifier <- {};
// identifier[vid] <- pid;
// identifiers <- [identifier]
// usbHost.registerDriver(UartOverUsbDriver, identifiers);

usbHost.on("connected",function (device) {
    server.log(typeof device + " was connected!");
    switch (typeof device) {
        case ("UartOverUsbDriver"):
            // We pass the uart over usb interface object into the constructor of
            // a class that takes the uart object and we can use the class as per 
            // its original documentation
            printer <- QL720NW(device);

            printer
                .setOrientation(QL720NW.LANDSCAPE)
                .setFont(QL720NW.FONT_SAN_DIEGO)
                .setFontSize(QL720NW.FONT_SIZE_48)
                .write("San Diego 48 ")
                .print();
            break;
    }
});
usbHost.on("disconnected",function (deviceName) {
    server.log(deviceName + " disconnected");
});


```

## Device Class Usage

### Constructor: UartOverUsbDriver(*usb*)

Class instantiation is handled by the UsbHost class.

 
### getIdentifiers()

Returns an array of tables with VID-PID key value pairs respectively. Identifiers are used by UsbHost to instantiate a matching devices driver.


#### Example

```squirrel
#require "uartoverusb.device.nut:1.0.0"

local identifiers = UartOverUsbDriver.getIdentifiers();

foreach (i, identifier in identifiers) {
    foreach (VID, PID in identifier){
        server.log("VID =" + VID + " PID = " + PID);
    }
}

```



### on(*eventName, callback*)

Subscribe a callback function to a specific event.


| Key | Data Type | Required | Description |
| --- | --------- | -------- | ----------- |
| *eventName* | String | Yes | The string name of the event to subscribe to. There is currently 1 event that you can subscibe to: "data"|
| *callback* | Function | Yes | Function to be called on event |

#### Example

```squirrel

// Callback to handle device connection
function onDeviceConnected(device) {
    server.log(typeof device + " was connected!");
    switch (typeof device) {
        case "UartOverUsbDriver":
            device.on("data", function (data){
                 server.log("Received " + data + " via usb");
            });
            break;
    }
}

```

### off(*eventName*)

Clears a subscribed callback function from a specific event.

| Key | Data Type | Required | Description |
| --- | --------- | -------- | ----------- |
| *eventName* | String | Yes | The string name of the event to unsubscribe from.|


```squirrel
#require "usbhost.device.nut:1.0.0"
#require "uartoverusb.device.nut:1.0.0"

usbHost <- UsbHost(hardware.usb);

// Register the Uart over Usb driver with usb host
usbHost.registerDriver(UartOverUsbDriver, UartOverUsbDriver.getIdentifiers());

usbHost.on("connected",function (device) {
    switch (typeof device) {
        case ("UartOverUsbDriver"):
            // Listen for data events
            device.on("data", function (data){
                 server.log("Received " + data + " via usb");
            });
            
            // Stop listening for data after 30 seconds
            imp.wakeup(30, function(){
                device.off("data");
            }.bindenv(this))
            break;
    }
});

```

### write(data)

Writes String or Blob data out to uart over usb.


| Key | Data Type | Required | Description |
| --- | --------- | -------- | ----------- |
| *data* | String/Blob | Yes | The String or Blob to be written to uart over usb.|


#### Example

```squirrel
#require "usbhost.device.nut:1.0.0"
#require "uartoverusb.device.nut:1.0.0"

usbHost <- UsbHost(hardware.usb);

// Register the Uart over Usb driver with usb host
usbHost.registerDriver(UartOverUsbDriver, UartOverUsbDriver.getIdentifiers());

usbHost.on("connected",function (device) {
    switch (typeof device) {
        case ("UartOverUsbDriver"):
            device.write("Testing usb over uart");
            break;
    }
});

```



## License

The UartOverUsbDriver is licensed under [MIT License](./LICENSE).
