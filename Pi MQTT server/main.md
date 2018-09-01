/*
 * Copyright 2010-2017 Amazon.com, Inc. or its affiliates. All Rights Reserved.
 *
 * Licensed under the Apache License, Version 2.0 (the "License").
 * You may not use this file except in compliance with the License.
 * A copy of the License is located at
 *
 *  http://aws.amazon.com/apache2.0
 *
 * or in the "license" file accompanying this file. This file is distributed
 * on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either
 * express or implied. See the License for the specific language governing
 * permissions and limitations under the License.
 */
 '''

from AWSIoTPythonSDK.MQTTLib import AWSIoTMQTTShadowClient
import logging
import time
import json
import argparse
import serial

ser = serial.Serial('/dev/ttyACM0', 9600)



# Shadow JSON schema:
#
# Name: Bot
# {
#	"state": {
#		"desired":{
#			"property":<INT VALUE>
#		}
#	}
# }

# Custom Shadow callback
def customShadowCallback_Update(payload, responseStatus, token):
    # payload is a JSON string ready to be parsed using json.loads(...)
    # in both Py2.x and Py3.x
    if responseStatus == "timeout":
        print("Update request " + token + " time out!")
    if responseStatus == "accepted":
        payloadDict = json.loads(payload)
        print("~~~~~~~~~~~~~~~~~~~~~~~")
        print("Update request with token: " + token + " accepted!")
        #print("property: " + str(payloadDict["state"]["desired"]["property"]))
        print("~~~~~~~~~~~~~~~~~~~~~~~\n\n")
    if responseStatus == "rejected":
        print("Update request " + token + " rejected!")

# Custom Shadow callback
def customShadowCallback_Delta(payload, responseStatus, token):
    # payload is a JSON string ready to be parsed using json.loads(...)
    # in both Py2.x and Py3.x
    print(responseStatus)
    payloadDict = json.loads(payload)
    print("++++++++DELTA++++++++++")
    print("Attic Light: " + str(payloadDict["state"]["AtticLight"]))
    print("Kitchen Light: " + str(payloadDict["state"]["KitchenLight"]))
    print("Bed Room Light: " + str(payloadDict["state"]["BedroomLight"]))
    print("Bath Room Light: " + str(payloadDict["state"]["BathroomLight"]))
    print("Living Room Light: " + str(payloadDict["state"]["LivingroomLight"]))
    print("Kitchen Temp: " + str(payloadDict["state"]["KitchenTemp"]))
    print("Living Room Temp: " + str(payloadDict["state"]["LivingroomTemp"]))
    print("Bed Room Temp: " + str(payloadDict["state"]["BedroomTemp"]))
    print("Window: " + str(payloadDict["state"]["Window"]))
    print("Door: " + str(payloadDict["state"]["Door"]))

    print("version: " + str(payloadDict["version"]))
    print("+++++++++++++++++++++++\n\n")
    
    
    if str(payloadDict["state"]["AtticLight"]) == "1":
        ser.write(bytes(b'aaaa'))
        #time.sleep(3)
    else:
        ser.write(bytes(b'bbbb'))
        #time.sleep(3)
        
    if str(payloadDict["state"]["KitchenLight"]) == "1":
        ser.write(bytes(b'cccc'))
        #time.sleep(3)
    else:
        ser.write(bytes(b'dddd'))
        #time.sleep(3)
        
    if str(payloadDict["state"]["BedroomLight"]) == "1":
        ser.write(bytes(b'eeee'))
        #time.sleep(3)
    else:
        ser.write(bytes(b'ffff'))
        #time.sleep(3)
        
    if str(payloadDict["state"]["BathroomLight"]) == "1":
        ser.write(bytes(b'gggg'))
        #time.sleep(3)
    else:
        ser.write(bytes(b'hhhh'))
        #time.sleep(3)
        
    if str(payloadDict["state"]["LivingroomLight"]) == "1":
        ser.write(bytes(b'iiii'))
        #time.sleep(3)
    else:
        ser.write(bytes(b'jjjj'))
        #time.sleep(3)
    
    TL = 'tl' + str(payloadDict["state"]["LivingroomTemp"])
    
    ser.write(bytes(TL, 'utf-8'))
    
    TB = 'tb' + str(payloadDict["state"]["BedroomTemp"])
    
    ser.write(bytes(TB, 'utf-8'))
    
    TA = 'ta' + str(payloadDict["state"]["KitchenTemp"])
    
    ser.write(bytes(TA, 'utf-8'))
    
    
# Read in command-line parameters
parser = argparse.ArgumentParser()
parser.add_argument("-e", "--endpoint", action="store", required=True, dest="host", help="Your AWS IoT custom endpoint")
parser.add_argument("-r", "--rootCA", action="store", required=True, dest="rootCAPath", help="Root CA file path")
parser.add_argument("-c", "--cert", action="store", dest="certificatePath", help="Certificate file path")
parser.add_argument("-k", "--key", action="store", dest="privateKeyPath", help="Private key file path")
parser.add_argument("-w", "--websocket", action="store_true", dest="useWebsocket", default=False,
                    help="Use MQTT over WebSocket")
parser.add_argument("-n", "--thingName", action="store", dest="thingName", default="Bot", help="Targeted thing name")
parser.add_argument("-id", "--clientId", action="store", dest="clientId", default="basicShadowDeltaListener",
                    help="Targeted client id")

args = parser.parse_args()
host = args.host
rootCAPath = args.rootCAPath
certificatePath = args.certificatePath
privateKeyPath = args.privateKeyPath
useWebsocket = args.useWebsocket
thingName = args.thingName
clientId = args.clientId

if args.useWebsocket and args.certificatePath and args.privateKeyPath:
    parser.error("X.509 cert authentication and WebSocket are mutual exclusive. Please pick one.")
    exit(2)

if not args.useWebsocket and (not args.certificatePath or not args.privateKeyPath):
    parser.error("Missing credentials for authentication.")
    exit(2)

# Configure logging
logger = logging.getLogger("AWSIoTPythonSDK.core")
logger.setLevel(logging.DEBUG)
streamHandler = logging.StreamHandler()
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
streamHandler.setFormatter(formatter)
logger.addHandler(streamHandler)

# Init AWSIoTMQTTShadowClient
myAWSIoTMQTTShadowClient = None
if useWebsocket:
    myAWSIoTMQTTShadowClient = AWSIoTMQTTShadowClient(clientId, useWebsocket=True)
    myAWSIoTMQTTShadowClient.configureEndpoint(host, 443)
    myAWSIoTMQTTShadowClient.configureCredentials(rootCAPath)
else:
    myAWSIoTMQTTShadowClient = AWSIoTMQTTShadowClient(clientId)
    myAWSIoTMQTTShadowClient.configureEndpoint(host, 8883)
    myAWSIoTMQTTShadowClient.configureCredentials(rootCAPath, privateKeyPath, certificatePath)

# AWSIoTMQTTShadowClient configuration
myAWSIoTMQTTShadowClient.configureAutoReconnectBackoffTime(1, 32, 20)
myAWSIoTMQTTShadowClient.configureConnectDisconnectTimeout(10)  # 10 sec
myAWSIoTMQTTShadowClient.configureMQTTOperationTimeout(5)  # 5 sec

# Connect to AWS IoT
myAWSIoTMQTTShadowClient.connect()

# Create a deviceShadow with persistent subscription
deviceShadowHandler = myAWSIoTMQTTShadowClient.createShadowHandlerWithName(thingName, True)

# Listen on deltas
deviceShadowHandler.shadowRegisterDeltaCallback(customShadowCallback_Delta)


# Loop forever
while True :
    while ser.in_waiting:
        mytest = ser.read()
        dump = ser.read()
        dump = ser.read()
        print(mytest)
        if mytest == b'a' :
            JSONPayload = '{"state":{"desired":{"AtticLight":' + str(1) + '}}}'
            deviceShadowHandler.shadowUpdate(JSONPayload, customShadowCallback_Update, 5)
        elif mytest == b'b' :
            JSONPayload = '{"state":{"desired":{"AtticLight":' + str(0) + '}}}'
            deviceShadowHandler.shadowUpdate(JSONPayload, customShadowCallback_Update, 5)
        elif mytest == b'c' :
            JSONPayload = '{"state":{"desired":{"KitchenLight":' + str(1) + '}}}'
            deviceShadowHandler.shadowUpdate(JSONPayload, customShadowCallback_Update, 5)
        elif mytest == b'd' :
            JSONPayload = '{"state":{"desired":{"KitchenLight":' + str(0) + '}}}'
            deviceShadowHandler.shadowUpdate(JSONPayload, customShadowCallback_Update, 5)
        elif mytest == b'e' :
            JSONPayload = '{"state":{"desired":{"BedroomLight":' + str(1) + '}}}'
            deviceShadowHandler.shadowUpdate(JSONPayload, customShadowCallback_Update, 5)
        elif mytest == b'f' :
            JSONPayload = '{"state":{"desired":{"BedroomLight":' + str(0) + '}}}'
            deviceShadowHandler.shadowUpdate(JSONPayload, customShadowCallback_Update, 5)
        elif mytest == b'g' :
            JSONPayload = '{"state":{"desired":{"BathroomLight":' + str(1) + '}}}'
            deviceShadowHandler.shadowUpdate(JSONPayload, customShadowCallback_Update, 5)
        elif mytest == b'h' :
            JSONPayload = '{"state":{"desired":{"BathroomLight":' + str(0) + '}}}'
            deviceShadowHandler.shadowUpdate(JSONPayload, customShadowCallback_Update, 5)
        elif mytest == b'i' :
            JSONPayload = '{"state":{"desired":{"LivingroomLight":' + str(1) + '}}}'
            deviceShadowHandler.shadowUpdate(JSONPayload, customShadowCallback_Update, 5)
        elif mytest == b'j' :
            JSONPayload = '{"state":{"desired":{"LivingroomLight":' + str(0) + '}}}'
            deviceShadowHandler.shadowUpdate(JSONPayload, customShadowCallback_Update, 5)
