# Edge & IoT Borathon

This borathon will provide you with an opportunity to explore IoT capabilities in AWS. High-level tasks will include:

- Creation of three (3) virtual devices or "things" for communicating temperature and humidity telemetry
- Creation of an AWS IoT Analytics pipeline for aggregating those readings, including filtering and transformation
- Creation of an AWS IoT Events detector model to take action when receiving a high temperature reading
- Creation of an AWS IoT Events detector model to take action when receiving a high humidity reading
- Creation of an AWS IoT Events detector model to take action when receiving a low humidity reading
- Exploration of dashboarding & visualization for graphing temperature and humidity data over time

![image](https://user-images.githubusercontent.com/25084145/113523979-9b0f8880-9579-11eb-943a-2d05e57e3ece.png)

## Device 1

Represents a temperature sensor called "<group name>Sensor01". Telemetry payload for this sensor will be in the following format:

`{
    "device_name": "<group name>Sensor01",
    "readings": {
        "temperature": 72,
        "humidity": 0.55
    },
    "timestamp": "<date/time value>"
}`

This device measures temperature readings in **Fahrenheit** and measures humidity as a %. You will implement code (in a language you choose) to send readings (in the format above) over MQTT to a topic in AWS IoT Core - use a topic name in the format of `<group name>_sensor_readings` to help ensure uniqueness. If you navigate to https://github.com/aws?q=aws-iot-device, you can find available language options. The recommendation is to use the v2 repo (if it exists for your chosen language) and fork the repository into your own GitHub account so you can modify as needed.

Use the following algorithm for randomizing temperature and humidity readings from this device:

- Send a new randomized set of readings **every 10 seconds**
- For temperature:
    - **10% of the time**, send an errant negative temperature value of -999
    - **10% of the time**, send a temperature value of 87
    - Otherwise, send a random temperature value between 50 and 80
- For humidity:
    - **25% of the time**, send a humidity value of 0.95
    - **10% of the time**, send a humidity value of 0.05
    - Otherwise, send a random humidity value between 0.4 and 0.8

## Device 2

Represents a sensor called "<group name>Sensor02". Telemetry payload for this sensor will be in the following format:

`{
    "device_name": "<group name>Sensor02",
    "measurements": {
        "temp": 72,
        "hum": 0.55
    },
    "timestamp": "<date/time value>"
}`

This device measures temperature readings in **Celsius** and measures humidity as a %. Implement code (in the language used for device 1 or in a different language for variety if desired) to send readings (in the format above) over MQTT to a topic in AWS IoT Core - use the same topic name used with device 1.

Use the following algorithm for randomizing temperature and humidity readings from this device:

- Send a new randomized set of readings **every 20 seconds**
- For temperature:
    - **25% of the time**, send a temperature value of 30.5 (~ 87 Fahrenheit)
    - Otherwise, send a random temperature value between 10 and 26.5
- For humiditiy:
    - **20% of the time**, send an errant negative humidity value of -1
    - **15% of the time**, send a humidity value of 0.05
    - Otherwise, send a random humidity value between 0.4 and 0.8

## Device 3

Represents a sensor called "<group name>Sensor03". Telemetry payload for this sensor will be in the following format:

`{
    "device_name": "<group name>Sensor03",
    "tempdata": 72,
    "timestamp": "<date/time value>"
}`

This device measures temperature readings in **Fahrenheit** but does not measure humidity values. Implement code (in the language used for the other devices or in a different language for variety if desired) to send readings (in the format above) over MQTT to a topic in AWS IoT Core - use the same topic name used with the other devices.

Use the following algorithm for randomizing temperature readings from this device:

- Send a new randomized temperature value using current date/time **every 15 seconds**
- **45% of the time**, send an errant negative temperature value of -999
- Otherwise, send a random temperature value between 50 and 80

## AWS IoT Core (https://docs.aws.amazon.com/iot/latest/developerguide/what-is-aws-iot.html)

Verify that data is being received in AWS IoT Core by using the MQTT test client and subscribing to the correct topic (`<group name>_sensor_readings`).

## AWS IoT Analytics (https://docs.aws.amazon.com/iotanalytics/latest/userguide/welcome.html)

Set up a channel, data store, pipeline, and data set for aggregating and processing temperature and humidity readings from the three (3) devices using the defined topic (`<group name>_sensor_readings`). Implement the following pipeline activities for "intelligent" processing of the incoming data:

- Use a Lambda transform activity to normalize data format for incoming telemetry to a common format of `{ "device_name": "<device name>", "temperature": <temperature value>, "humidity": <humidity value (if present)>, "timestamp": "<date/time value>" }`
- Use the same transform operation to convert temperature on all incoming telemetry readings to Fahrenheit
- Use a conditional filter to exclude any errant readings (readings with a temperature value of -999 and/or a humidity value of -1)
- In the Lambda function, route the transformed & filtered data into an S3 bucket for data storage (using `<group name>` in the topic name for distinction from other implementations)

## AWS IoT Events (https://docs.aws.amazon.com/iotevents/latest/developerguide/what-is-iotevents.html)

Set up a detector model that triggers an action when receiving a high temperature reading (above 80 F), when receiving a low humidity reading (less than 0.1), or when receiving a high humidity reading (above 0.9). Create one IoT topic called `<group name>_high_temp`, one IoT topic called `<group name>_low_humidity`, and one IoT topic called `<group name>_high_humidity` (using your group name as part of the topic names for distinction). When one of the thresholds is triggered, publish a message to the appropriate IoT topic. Test the receipt of messages on the IoT topic by adjusting randomization %'s for each case in the logic used to build and send the message from the devices (e.g., use high random % for high temp on a device, use high random % for high humidity on a different or same device, etc.). Alternatively, you can manually post messages in the MQTT test client to create the conditions for triggering the events.

## Dashboard

Extract data from the S3 bucket into a CSV format (or transform as part of Lambda) & import into an Excel spreadsheet for several temperature and humidity readings over a period of time. Create a visualization that charts those readings.

## Optional Exercises

- Experiment with a different programming language for sending MQTT data from 1 of your virtual "things" (if not already completed)
- Expand AWS IoT Events detector models to use "intelligent" thresholds:
    - Take action when receiving 3 high temperature readings (above 80 F) **in sequence**
    - Take action when receiving 3 low humidity readings (less than 0.1) **in sequence**
    - Take action when receiving 3 high humidity readings (above 0.9) **in sequence**
    - Ensure each action accounts for a return to normal (reset of threshold on receipt of reading in normal range) 
- Experiment with one of the other actions available with your AWS IoT event - i.e., instead of publish to an IoT topic, push a new notification to an SNS topic and set up a new Lambda to listen for that notification and write to a different S3 bucket, etc.
- Look for opportunities to integrate your data with a tool like PowerBI for more sophisticated visualizations
