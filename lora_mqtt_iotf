import json
import paho.mqtt.client as mqtt
import ssl
import serial
import time
import serial.tools.list_ports

ser = serial.Serial(serial.tools.list_ports.comports()[0].device, 9600, timeout=0, parity=serial.PARITY_EVEN, rtscts=1)
ser.flushInput() #clear Serialbuffer
ser.flush() #clear Serialbuffer

org="apx28c"
username = "use-token-auth"
password = "w+MBybkaMuB3K&iTH?" #auth-token
deviceType="lora"
deviceID="m0"

topic = "iot-2/evt/status/fmt/json"

def on_connect(client, userdata, rc):
    print("Connected with result code "+str(rc))
    client.subscribe("iot-2/cmd/acommand/fmt/json")

def on_message(client, userdata, msg):
    print(msg.topic+" "+str(msg.payload))

rootCert = "certs/messaging.pem"

clientID = "d:" + org + ":" + deviceType + ":" + deviceID
client = mqtt.Client(clientID)
client.on_connect = on_connect
client.on_message = on_message

client.username_pw_set(username, password=password)

client.tls_set(ca_certs=rootCert, certfile=None, keyfile=None, cert_reqs=ssl.CERT_REQUIRED, tls_version=ssl.PROTOCOL_SSLv23)

client.connect(org+".messaging.internetofthings.ibmcloud.com", 8883, 60)
client.loop_start()

while True :
    ser.flush()
    s = ser.readline()
    if len(s) > 0:
      s=s.replace("\r\n","")
      l=s.split(",")
    if len(l) == 10:

        if l[1] == "A":

  	  lat= round(float(l[2][2:])/60+float(l[2][:2]),4)
          lng= round(float(l[4][3:])/60+float(l[4][:3]),4)
          if l[3] == "S":
            lat=lat*-1
          if l[5] == "W":
            lng=lng*-1
          spd=float(l[6])
          dir=float(l[7])

          tmd=l[8][4:]+"-"+l[8][2:-4]+"-"+l[8][:2]+"T"+l[0][:2]+":"+l[0][2:-5]+":"+l[0][4:-3]+"Z"
          print "%f,%f,%f,%f,%s" % (lat,lng,dir,spd,tmd)

  while 1:
    time.sleep(2)
    msg = {"d":{
        'location': "%s%s" %(lat,lng),
        'deviceLocation':{
          'lat': decimal.Decimal("%f" %(lat)),
          'lng': decimal.Decimal("%f" %(lng))
        }
     }}
    payload = json.dumps(msg)
    client.publish(topic, payload, qos=0, retain=False)
    time.sleep(300)
