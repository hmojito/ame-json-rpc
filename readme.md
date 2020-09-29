#AME JSON RPC API V2.0.0

## mongoose-os

[mongoose-os] (https://mongoose-os.com/docs/mongoose-os/quickstart/setup.md) is used as a base OS, it offers functionnalities to manage configuration and interact easily with the tool with a standard `remote procedure call (RPC)` mecanism throught a variety of channels.

### JSON-RPC

The RPC mechanism is based on [JSON-RPC v2] (https://www.jsonrpc.org/specification).

#### Request frame format

    {"id": 1, "src": "src", "method": "method", "params": params}

> [required] `id`: `number`, the identifier associated with the request, the tool will reuse this id in the reply (id)

> [required] `src`: `string`,  the name of the sender, the tool will reuse this name in the reply `dst`

> [required] `method`: `string`, Name of the method to call on the tool

> [optionnal] `params`: `number`, `string`, `Object`, Arguments passed to the method

#### Reply frame format

##### Reply with result

When the request is done wihout error, a reply is sent containing the result of the remote procedure call. 

    {"id": 1, "dst": "dst", "result": result }
    
> [required] `id`: `number`, the identifier used in the request frame.

> [required] `dst`: `string`, the destination of the reply, corresponding to the source of the request. 

> [required] `result`: `number`, `string`, `Object` or `null`, The result of the method call if no error where thrown.

##### Reply with error

When the request throw an error, a reply is sent containing the error code and the error message. 

    {"id": 1, "dst": "dst", "error": {"code":1234, "message":"Invalid parameters" }
    
> [required] `id`: `number`, the identifier used in the request frame

> [required] `dst`: `string`, the destination of the reply, corresponding to the source of the request 

> [required] `error.code`: `number`, Code representing the error.

> [required] `error.message`: `string`, Message describing the error.

#### Notification frame format

Depending on the channel used to communicate, the tool can send notifications when some events are triggered remotely.

    {"method": "method", "params": params}

> [required] `method`: `string`, Name of the event triggered

> [optionnal] `params`: `number`, `string`, `Object`, Parameters associated with this event


### RPC Channels

#### TCP Socket

The host open a TCP Socket Server listening on a port, the tool will connect automatically to this server. 

##### Sending requests

The host needs to write the JSON RPC Request frame ending with a linefeed.

##### Receiving replies and notification

The host needs to read line by line on the corresponding socket. 
    
##### Enable and configure RPC TCP Socket channel using mos command line :  

    mos config-set rpc.tcp.enable=true rpc.tcp.server_addr=192.168.1.1:8000

#### MQTT

Needs a MQTT broker installed and configured accessible by the tool and the host.

##### Sending requests

The host needs first to suscribe to the reply topic, this topic should be unique, for example you can use `/device.id/method/id/rpc`

The host can now publish on the topic `/device.id/rpc` a JSON RPC Request frame : 

    {"id": 1, "src": "/device.id/method/id", "method": "method", "params": params}

When the RPC call ended with result or error the corresponding JSON RPC Reply frame is sent on the topic. 

##### Receiving notifications

To receive notifications the host needs to suscribe to the topic `/device.id/method`, where `method` represent the notification name.

##### Enable and configure RPC MQTT channel using mos command line :  

    mos config-set rpc.mqtt.enable=true mqtt.enable=true mqtt.server=192.168.1.1:1883

#### Websocket

Websocket channel can only be used to send requests, notifications are not supported.

##### Sending requests

The host connect to the device http server at the address `device_ip:80`  

#### HTTP Rest

#### UDP Socket

#### Google Cloud Platform

#### Azure

#### GATTS

## mos

[mos] (https://mongoose-os.com/docs/mongoose-os/quickstart/setup.md) is a command line utility, it can be used to configure and manage tool, and to call RPC methods on the tool.

### Selecting a channel

To select the channel to communicate with the tool, mos take `--port` argument with one of the folowing option :

* websocket : ws://192.168.4.1/rpc
* mqtt : mqtt://my.mqtt.server:1883/device.id

> You can set MOS_PORT to one of this value in your environement path to use mos utility with this port as default value

### Reading configuration 

    mos config-get
    
    mos config-get wifi
    
    mos config-get wifi.sta
    
### Writing configuration 

    mos config-set wifi.sta.ssid="WIFI_TEST" wifi.sta.enable=true
    
### Calling RPC Methods 

    mos call Sys.GetInfo
    
    mos call AME.Tool.Bip
    
    mos call AME.Program.Get '1'
    
### Manage files 

    mos ls
    
    mos get conf9.json
    
    mos put conf9.json
    
### Firmware update

    mos ota firmware.zip
    
    mos call OTA.commit

## AME Data Structures

### AMEDate 
    
    "YYYY/MM/DD-HH:MM:SS"

### AMELedColor

    enum AMELedColor {
        GREEN   = "GREEN",
        ORANGE  = "ORANGE",
        RED     = "RED",
        BLUE    = "BLUE"
    };

### AMETriggerStatus

    enum AMETriggerStatus {
        ON      = "ON"
        OFF     = "OFF"    
    };

### AMEDirection

    enum AMEDirection {
        CW  = "CW",                 // Clocwise
        CCW = "CCW"                 // Counter Clockwise
    };

### AMETorqueUnit

    enum AMETorqueUnit {
        NM  = "N.m",
        DNM = "dN.m"
    };

		
### AMEStatus
    
    enum AMEStatus {
        BATTERY                 = "BATTERY",
        INVALID_HALL_STATE      = "INVALID_HALL_STATE",
        I2T                     = "I2T",
        MOTOR_STALL             = "MOTOR_STALL",
        OVER_CURRENT            = "OVER_CURRENT",
        OVER_TEMPERATURE        = "OVER_TEMPERATURE",
        CURRENT_OFFSET          = "CURRENT_OFFSET",
        SHUNT_CALIBRATION       = "SHUNT_CALIBRATION",
        TORQUE_OFFSET           = "TORQUE_OFFSET",
        TRANSDUCER              = "TRANSDUCER",
        STEP_TIMEOUT            = "STEP_TIMEOUT",
        PROGRAM_TIMEOUT         = "PROGRAM_TIMEOUT",
        OVER_TORQUE             = "OVER_TORQUE",
        OVER_ANGLE              = OVER_ANGLE",
        UNDER_TORQUE            = "UNDER_TORQUE",
        UNDER_ANGLE             = "UNDER_ANGLE",
        EARLY_TRIGGER_RELEASE   = "EARLY_TRIGGER_RELEASE",
        WATCH_DOG               = "WATCH_DOG",
        STEPS_INCOMPLETE        = "STEPS_INCOMPLETE",
    };

### AMEStep

	class AMEStep {
	
        enum Type {
            ANGLE = "ANGLE",
            TORQUE = "TORQUE"
        };
        
        num:                        number;                 // Step number [1-8]
        type:                       Type;                   // Step type 
        direction:                  AMEDirection;           // Step type 
        timeout:                    number;                 // Step timeout (ms)
        counting_angle_threshold:   number;
        shiftdown_threshold:        number;
        joint_type:                 AMEJointType;
        target:                     number;
        max_torque:                 number;
        min_torque:                 number;
        max_angle:                  number;
        min_angle:                  number;
        shifdown_speed:             number;
        free_speed:                 number;
        acceleration:               number;
	};

### AMEProgram

    class AMEProgram {
        num:                number;
        gang_count:         number;
        auto_increment:     number;
        reset_to:           number;
        assembly_complete:  boolean;
        steps:              AMEStep[];
        reverse_step:       AMEStep;
    };

### AMEResult

    class AMEResult {
        tool_id:            number;
        life_cycle:         number;
        user_cycle:         number;
        result_index:       number;
        prog_num:           number;
        step_num:           number;
        current_gang_count: number;
        total_gang_count:   number;
        type:               AMEStep.Type;
        torque_unit:        AMETorqueUnit;
        peak_torque:        number;
        peak_angle:         number;
        peak_current:       number;
        target:             number;
        max_torque:         number;
        min_torque:         number;
        max_angle:          number;
        min_angle:          number;
        status:             AMEStatus[];
        date:               AMEDate;
    };


### AMEToolInfo

    class AMEToolInfo {
        tool_id:        number;
        max_torque:     number;
        motor_speed:    number;
        nb_results:     number;
        life_cycle:     number;
        user_cycle:     number;
    };

### AMECalibration

    class AMECalibration {
        
        class Value {
            date: AMEDate;
            factor: number;
        };
        
        torque: Value;
        angle: Value;
    };

## AME RPC Commands
### AME.Tool.Bip

Emit a short bip

    { "id": 1, "src":"src", method": "AME.Tool.Bip" }

### AME.Tool.Led

Switch on tool leds

    { "id": 1, "src":"src", method": "AME.Tool.Led", "params": leds }

> [required] `leds` : `AMELed[]` : Array of leds to switch on 

> return `null`

### AME.Program.Get

Retrieve program (num) from the tool

    { "id": 1, "src":"src", method": "AME.Program.Get" "params": num }

> [required] `num` : `number` : Program number

> return `AMEProgram`

### AME.Program.Set

Store program (program) to the tool
    
    { "id": 1, "src":"src", method": "AME.Program.Set" "params": program }

> [required] `program` : `AMEProgram` : Program to store

> return `null`

### AME.Calibration.User.Get

Retrieve user tool calibration 

    { "id": 1, "src":"src", method": "AME.Calibration.User.Get" }

> return `AMECalibration`

### AME.Calibration.User.Set

Store user tool calibration

    { "id": 1, "src":"src", method": "AME.Calibration.User.Set", "params": user_calibration }
    
> [required] `user_calibration` : `AMECalibration` : User calibration to set

### AME.Calibration.Factory.Get

Retrieve factory tool calibration 

    { "id": 1, "src":"src", method": "AME.Calibration.Factory.Get" }

> return `AMECalibration`

### AME.Result.Get

Retrieve stored result

    { "id": 1, "src":"src", method": "AME.Result.Get", "params": idx }

> [required] `idx` : `number` : Index of the result to retrieve [0,nb_results - 1]

> return `AMEResult`

## AME Events

### AME.Result.Received

Emited when a new result is done

    { "method": "AME.Result.Received", "params": result }
    
> `result` : AMEResult

### AME.Trigger.Changed

Emited when the tool trigger changed (ON/OFF)

    { "method": "AME.Trigger.Changed", "params": trigger_status }
    
> `trigger_status` : AMETriggerStatus

### AME.Direction.Changed

Emited when the tool direction changed (CW/CCW)

    { "method": "AME.Direction.Changed", "params": direction }
    
> `direction` : AMEDirection

### AME.Program.Changed

Emited when the current selected program changed (manually or programatically), or when the stored program updated.

    { "method": "AME.Program.Changed", "params": num }
    
> `num` : number
