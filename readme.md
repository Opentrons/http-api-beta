# Opentrons HTTP Platform API Beta

> ### Curious to access the early HTTP API?
> [Use this form to apply for the beta program](https://opentrons-ux.typeform.com/to/WxFIa8vs)

- [Opentrons HTTP Platform API Beta](#opentrons-http-platform-api-beta)
  * [New Features](#new-features)
  * [Protocol upload](#protocol-upload)
  * [Protocol sessions](#protocol-sessions)
  * [Support files](#support-files)
  * [Importing user modules in Python protocols](#importing-user-modules-in-python-protocols)
- [Audience](#audience)
- [Sharing Feedback](#sharing-feedback)
- [Setting Up](#setting-up)
  * [Update your Opentrons App and OT-2](#update-your-opentrons-app-and-ot-2)
  * [Enable the beta features](#enable-the-beta-features)
- [Examples](#examples)
  * [Requirements for these examples](#requirements-for-these-examples)
  * [Uploading a protocol](#uploading-a-protocol)
  * [POST the file to OT-2](#post-the-file-to-ot-2)
  * [Extract the uploaded protocol id from the response](#extract-the-uploaded-protocol-id-from-the-response)
  * [Uploading a protocol with support files](#uploading-a-protocol-with-support-files)
  * [Extract the uploaded protocol id from the response](#extract-the-uploaded-protocol-id-from-the-response-1)
  * [Creating a protocol session](#creating-a-protocol-session)
  * [Extract the session id from the response](#extract-the-session-id-from-the-response)
  * [Sending protocol session commands](#sending-protocol-session-commands)
  * [Run](#run)
  * [Pause](#pause)
  * [Resume](#resume)
  * [Cancel](#cancel)
  * [Getting protocol session’s status](#getting-protocol-session-s-status)
  * [Deleting a protocol session](#deleting-a-protocol-session)
  * [Deleting an uploaded protocol](#deleting-an-uploaded-protocol)
  * [Protocol](#protocol)
  * [Configuration File](#configuration-file)
  * [Python helpers](#python-helpers)
  * [Replace with actual OT2 address](#replace-with-actual-ot2-address)
- [Future Plans](#future-plans)
  * [Analysis upon upload](#analysis-upon-upload)
  * [Notifications](#notifications)
  * [Custom Labware](#custom-labware)



Version 4.0 of the OT-2 robot software introduces a number of features that provide greater flexibility in protocol execution. 

Prior to this version, to run a protocol on the OT-2, you needed to use:

* The Opentrons App GUI.
* The opentrons-execute tool on the OT-2’s command line.
* The OT-2’s Jupyter Notebook.

Now, you can additionally upload, run, and interact with protocols entirely through a documented RESTful HTTP API.

Please note: These features are in beta stage and therefore details might change. 

# New Features

## Protocol upload

Python and JSON protocols can be uploaded to the OT-2 using an HTTP API. Protocol files will reside on the OT-2’s file system until deleted or the OT-2 is turned off.

## Protocol sessions

A protocol session type has been added to the sessions endpoints. A protocol session is associated with an uploaded protocol. Protocols can be simulated and executed using session commands.

## Support files

Protocols uploaded using the HTTP API can include support files. These can be data files or other python files. Python protocols can use _open_ to access these files. 

## Importing user modules in Python protocols

A python file uploaded as a support file can be imported in a Python protocol. This enables breaking up Python protocols into multiple files for easy code reuse. 


# Audience

We assume that you’re either already familiar with these things, or you can figure them out on your own.

* How HTTP and JSON APIs generally work.
* How to use your chosen programming language, libraries, and tools (like  [Postman](https://www.postman.com/)  or  [cURL](https://en.wikipedia.org/wiki/CURL) ) to interact with HTTP and JSON APIs.

If you’re not already familiar with these things, let us know. We won’t be able to provide detailed support, but we might be able to point you to some learning resources.

# Sharing Feedback

Share feedback by emailing  [beta@opentrons.com](applewebdata://2A956D69-64A2-432B-880B-381DED88BF33/beta@opentrons.com) .

If you have any thoughts on the current API, or our  [future plans](https://coda.io/d/Opentrons-HTTP-Platform-API-Beta_di2AUgtv1iy/Future-plans_suIju)  for it, please share them! **We need your help to make this API great. ❤️**

We’d love to hear about:
* Bugs that you run into.
* Features that seem missing.
* Parts of the API that you found confusing, hard to use, or inconvenient.
* Difficulties following the documentation, or things that you wish were explained better.
* Anything else on your mind!

# Setting Up

## Update your Opentrons App and OT-2

Make sure both your Opentrons App and OT-2 are on the latest software version.

Perform the update  [as normal](https://support.opentrons.com/en/articles/1795303-get-started-update-your-ot-2) .

You can check what the latest version is at  [https://github.com/Opentrons/opentrons/releases/latest](https://github.com/Opentrons/opentrons/releases/latest) .

## Enable the beta features

Since these features are experimental, they’re disabled by default. Here’s how to enable them:

1. From the **Robot** tab, go to your OT-2’s page.
2. Scroll down to the **Advanced Settings**section.
3. Turn on **Enable Experimental HTTP Protocol Sessions**.

**Note:** While these beta features are enabled, you won’t be able to upload protocols the normal way, through the Opentrons App.

To restore the ability to upload protocols through the Opentrons App, turn off **Enable Experimental HTTP Protocol Sessions.** Feel free to toggle the setting whenever you need to.

# Examples

## Requirements for these examples

The software requirement for the examples is cURL or Python 3.7 with  [requests](https://requests.readthedocs.io/en/master/)  package. 

Replace _{_**robot_ip_address**_}_ with the IP address of the OT-2 which is found in the Opentrons application’s **Connectivity** section. 

## Uploading a protocol

To upload a protocol file called “my_protocol.py”. The response will contain a unique identifier for the protocol (_protocol_id)._

### cURL

```shell
curl -X POST “http://{*robot_ip_address*}:31950/protocols” -H “Opentrons-Version: 2” -H  “accept: application/json” -H “Content-Type: multipart/form-data” -F “protocolFile=@my_protocol.py”
```

### Python

POST the file to OT-2:

```python
response = requests.post(
    url=f”http://{robot_ip_address}:31950/protocols”,
    files=[(“protocolFile”, open(“my_protocol.py”, ‘rb’))],
    headers={“Opentrons-Version”: “2”},
)
```

Extract the uploaded protocol id from the response:

```python
protocol_id = response.json()[‘data’][‘id’]
```

## Uploading a protocol with support files

Uploading a protocol file called “my_protocol.py” with two data files called “my_data.csv” and “my_data.json”. 

### cURL

```shell
curl -X POST “http://{*robot_ip_address*}:31950/protocols” -H “Opentrons-Version: 2” -H  “accept: application/json” -H  “Content-Type: multipart/form-data” -F “supportFiles=@my_data.csv” -F “supportFiles=@my_data.json” -F “protocolFile=@my_protocol.py”
```

### Python

POST the files to OT-2:

```python
response = requests.post(
    url=f”http://{robot_ip_address}:31950/protocols”,
    files=[(“protocolFile”, open(“my_protocol.py”, ‘rb’)),
           (“supportFiles”, open(“my_data.csv”, ‘rb’)),
           (“supportFiles”, open(“my_data.json”, ‘rb’))],
    headers={“Opentrons-Version”: “2”},
)
```

Extract the uploaded protocol id from the response:

```python
protocol_id = response.json()[‘data’][‘id’]
```

## Creating a protocol session

Once a protocol is uploaded, a session must be created to run the protocol. The response will contain a unique identifier for the session (_session_id_).

### cURL

```shell
curl -X POST “http://{*robot_ip_address*}:31950/sessions” -H “Opentrons-Version: 2” -H “accept: application/json” -H “Content-Type: application/json” -d “{\”data\”:{\”sessionType\”:\”protocol\”,\”createParams\”:{\”protocolId\”:\”{*protocol_id*}\”}}}”
```

### Python

```python
response = requests.post(
    url=f”http://{robot_ip_address}:31950/sessions”,
    json={
        “data”: {
            “sessionType”: “protocol”,
            “createParams”: {
                “protocolId”: protocol_id
            }
        }
    },
    headers={“Opentrons-Version”: “2”},
)
```

Extract the session id from the response:

```python
session_id = response.json()[‘data’][‘id’]
```

Then, use `session_id` to send commands to the protocol sessions.

## Run

### cURL

```shell
curl -X POST “http://{*robot_ip_address*}:31950/sessions/{*session_id*}/commands/execute” -H “Opentrons-Version: 2” -H “accept: application/json” -H “Content-Type: application/json” -d “{\”data\”:{\”command\”:\”protocol.startRun\”,\”data\”:{}}}”
```

### Python

```python
requests.post(
    url=f”http://{robot_ip_address}:31950/sessions/{session_id}/commands/execute”,
    headers={“Opentrons-Version”: “2”},
    json={“data”: {“command”: “protocol.startRun”, “data”: {}}}
)
```

## Pause

### cURL

```shell
curl -X POST “http://{*robot_ip_address*}:31950/sessions/{*session_id*}/commands/execute” -H “Opentrons-Version: 2” -H “accept: application/json” -H “Content-Type: application/json” -d “{\”data\”:{\”command\”:\”protocol.pause\”,\”data\”:{}}}”
```

### Python

```python
requests.post(
    url=f”http://{robot_ip_address}:31950/sessions/{session_id}/commands/execute”,
    headers={“Opentrons-Version”: “2”},
    json={“data”: {“command”: “protocol.pause”, “data”: {}}}
)
```

## Resume

### cURL

```shell
curl -X POST “http://{*robot_ip_address*}:31950/sessions/{*session_id*}/commands/execute” -H “Opentrons-Version: 2” -H “accept: application/json” -H “Content-Type: application/json” -d “{\”data\”:{\”command\”:\”protocol.resume\”,\”data\”:{}}}”
```

### Python

```shell
requests.post(
    url=f”http://{robot_ip_address}:31950/sessions/{session_id}/commands/execute”,
    headers={“Opentrons-Version”: “2”},
    json={“data”: {“command”: “protocol.resume”, “data”: {}}}
)
```
  
## Cancel

### cURL

```shell
curl -X POST “http://{*robot_ip_address*}:31950/sessions/{*session_id*}/commands/execute” -H “Opentrons-Version: 2” -H “accept: application/json” -H “Content-Type: application/json” -d “{\”data\”:{\”command\”:\”protocol.cancel\”,\”data\”:{}}}”
```

### Python

```python
requests.post(
    url=f”http://{robot_ip_address}:31950/sessions/{session_id}/commands/execute”,
    headers={“Opentrons-Version”: “2”},
    json={“data”: {“command”: “protocol.cancel”, “data”: {}}}
)
```

## Getting protocol session’s status

### cURL

```shell
curl “http://{*robot_ip_address*}:31950/sessions/{*session_id*}” -H “Opentrons-Version: 2”
```

### Python

```python
response = requests.get(
    url=f”http://{robot_ip_address}:31950/sessions/{session_id}”,
    headers={“Opentrons-Version”: “2”},
)
```

## Deleting a protocol session

### cURL

```shell
curl -X DELETE “http://{*robot_ip_address*}:31950/sessions/{*session_id*}” -H “Opentrons-Version: 2”
```

### Python

```python
requests.delete(
    url=f”http://{robot_ip_address}:31950/sessions/{session_id}”,
    headers={“Opentrons-Version”: “2”},
)
```

## Deleting an uploaded protocol

### cURL

```shell
curl -X DELETE “http://{*robot_ip_address*}:31950/protocols/{*protocol_id*}” -H “Opentrons-Version: 2”
```

### Python

```python
requests.delete(
    url=f”http://{robot_ip_address}:31950/protocols/{protocol_id}”,
    headers={“Opentrons-Version”: “2”},
)

# Complete example program

This example uses the [basic transfer](https://docs.opentrons.com/v2/new_examples.html#basic-transfer) sample protocol as the foundation, but utilizing new features.

## Protocol

Create a file called “basic_transfer.py” containing this data:

```python
from opentrons import protocol_api
from helpers import load_config, pick_up_then_drop

metadata = {‘apiLevel’: ‘2.6’}


def run(protocol: protocol_api.ProtocolContext):
    configuration = load_config(“basic_transfer_config.json”)

    plate = protocol.load_labware(configuration[‘plate’], 1)
    tiprack_1 = protocol.load_labware(configuration[‘tiprack’], 2)
    instrument = protocol.load_instrument(configuration[‘instrument’][‘model’],
                                          configuration[‘instrument’][‘mount’],
                                          tip_racks=[tiprack_1])

    transfers = configuration[‘transfers’]
    for transfer in transfers:
        with pick_up_then_drop(instrument):
            ml = transfer[‘ml’]
            instrument.aspirate(ml, plate[transfer[‘source_well’]])
            instrument.dispense(ml, plate[transfer[‘target_well’]])
```

## Configuration File

Create a file called “basic_transfer_config.json” containing

```json
{
  “plate”: “corning_96_wellplate_360ul_flat”,
  “tiprack”: “opentrons_96_tiprack_300ul”,
  “instrument”: {
    “model”: “p300_single”,
    “mount”: “right”
  },
  “transfers”: [
    {
      “source_well”: “A1”,
      “target_well”: “B1”,
      “ml”: 100
    }]
}
```

## Python helpers

Create a file called “helpers.py” containing this code.

```python
import contextlib
import json

def load_config(name: str):
    “””Load a configuration file”””
with open(name, ‘rb’) as f:
        return json.load(f)

@contextlib.contextmanager
def pick_up_then_drop(instrument):
    “””Pick up then automatically drop the tip in a context manager”””
    instrument.pick_up_tip()
    yield
    instrument.drop_tip()
```

# Python Run Script

This script will upload the basic_transfer protocol along with the configuration and helper python module. It will then create protocol session and run the protocol. It will print out the final status of the protocol session when the run is completed. Finally, it will delete the session and the protocol.

```python
import requests
import time


# Replace with actual OT2 address
ROBOT_IP_ADDRESS = “127.0.0.1”


def run():
    # POST the protocol and support files to OT2
    response = requests.post(
        url=f”http://{ROBOT_IP_ADDRESS}:31950/protocols”,
        headers={“Opentrons-Version”: “2”},
        files=[(“protocolFile”, open(“basic_transfer.py”, ‘rb’)),
               (“supportFiles”, open(“helpers.py”, ‘rb’)),
               (“supportFiles”, open(“basic_transfer_config.json”, ‘rb’)),
               ]
    )
    print(f”Create Protocol result: {response.json()}”)

    # Extract the uploaded protocol id from the response
    protocol_id = response.json()[‘data’][‘id’]

    try:
        errors = response.json()[‘data’].get(‘errors’)
        if errors:
            raise RuntimeError(f”Errors in protocol: {errors}”)

        run_protocol(protocol_id)

    finally:
        # Use the protocol_id to DELETE the protocol
        requests.delete(
            url=f”http://{ROBOT_IP_ADDRESS}:31950/protocols/{protocol_id}”,
            headers={“Opentrons-Version”: “2”},
        )


def run_protocol(protocol_id: str):
    # Create a protocol session
    response = requests.post(
        url=f”http://{ROBOT_IP_ADDRESS}:31950/sessions”,
        headers={“Opentrons-Version”: “2”},
        json={
            “data”: {
                “sessionType”: “protocol”,
                “createParams”: {
                    “protocolId”: protocol_id
                }
            }
        }
    )
    print(f”Create Session result: {response.json()}”)
    # Extract the session id from the response
    session_id = response.json()[‘data’][‘id’]

    try:
        # Creating the protocol session kicks off a full simulation which can
        # take some time. Wait until session is in the ‘loaded’ state before running
        while True:
            # Sleep for 1/2 a second
            time.sleep(.5)

            response = requests.get(
                url=f”http://{ROBOT_IP_ADDRESS}:31950/sessions/{session_id}”,
                headers={“Opentrons-Version”: “2”},
            )

            current_state = response.json()[‘data’][‘details’][‘currentState’]
            if current_state == ‘loaded’:
                break
            elif current_state == ‘error’:
                raise RuntimeError(f”Error encountered {response.json()}”)

        # Send a command to begin a protocol run
        requests.post(
            url=f”http://{ROBOT_IP_ADDRESS}:31950/sessions/{session_id}/commands/execute”,
            headers={“Opentrons-Version”: “2”},
            json={“data”: {“command”: “protocol.startRun”, “data”: {}}}
        )

        # Wait until session is in the ‘finished’ state
        while True:
            # Sleep for 1/2 a second
            time.sleep(.5)

            response = requests.get(
                url=f”http://{ROBOT_IP_ADDRESS}:31950/sessions/{session_id}”,
                headers={“Opentrons-Version”: “2”},
            )

            current_state = response.json()[‘data’][‘details’][‘currentState’]
            if current_state == ‘finished’:
                print(“Run is complete:”)
                print(response.json())
                break
            elif current_state == ‘error’:
                raise RuntimeError(f”Error encountered {response.json()}”)

    finally:
        # Use the session_id to DELETE the session
        requests.delete(
            url=f”http://{ROBOT_IP_ADDRESS}:31950/sessions/{session_id}”,
            headers={“Opentrons-Version”: “2”},
        )


if __name__ == ‘__main__’:
    run()
```

# Future Plans

## Analysis upon upload

Protocols uploaded to /protocols endpoint will be immediately analyzed and validated. The labware, instruments, and modules required by the protocol will be returned in the JSON response.

Example upload response:

```json
{
  "links": {
    "self": {
      "href": "/protocols/protocol.ot2"
    },
    "protocols": {
      "href": "/protocols"
    },
    "protocolById": {
      "href": "/protocols/{protocolId}"
    }
  },
  "data": {
    "id": "protocol.ot2",
    "protocolFile": {
      "basename": "protocol.ot2.py"
    },
    "supportFiles": [],
    "lastModifiedAt": "2021-01-06T14:14:36.337610+00:00",
    "createdAt": "2021-01-06T14:14:36.337615+00:00",
    "requiredEquipment": {
      "pipettes": [
        {
          "mount": "right",
          "requestedAs": "p300_single",
          "pipetteName": "p300_single",
          "channels": 1
        }
      ],
      "labware": [
        {
          "label": "nest_96_wellplate_200ul_flat",
          "uri": "opentrons/nest_96_wellplate_200ul_flat/1",
          "slot": 1
        },
        {
          "label": "opentrons_96_tiprack_20ul",
          "uri": "opentrons/opentrons_96_tiprack_20ul/1",
          "slot": 2
        },
        {
          "label": "opentrons_24_tuberack_eppendorf_1.5ml_safelock_snapcap",
          "uri": "opentrons/opentrons_24_tuberack_eppendorf_1.5ml_safelock_snapcap/1",
          "slot": 4
        },
        {
          "label": "opentrons_1_trash_1100ml_fixed",
          "uri": "opentrons/opentrons_1_trash_1100ml_fixed/1",
          "slot": 12
        }
      ],
      "modules": []
    },
    "metadata": {
      "name": "Agar test v1.0",
      "author": null,
      "apiLevel": "2.2"
    }
  }
}
```

## Notifications

The OT2 now has a pub/sub service: the notification server. Users can receive protocol events via a websocket or the python subscriber client from our notification-server package.

## Custom Labware

Uploading custom labware is not currently supported in the HTTP API, but will be in the near future.
