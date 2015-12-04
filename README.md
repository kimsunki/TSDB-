# TSDB에 현재 온도띄우기


#!/usr/bin/python

import time
import os
import sys
import serial

packet =''
/*
url ="http://127.0.0.1:4242/api/put"

def insert( Temperature, v1, bigEndian( nodeID )):
        data={
                "metric":"foo.bar",
                "timestamp":time.time(),
                "value":value,
                "tags":{
                        "host":"mypc"
                }
        }
        ret = requests.post(url, data=json.dumps(data))
        print ret
        time.sleep(1)




*/
def bigEndian(s):
        res = 0
        while len(s):
                s2 = s[0:2]
                s = s[2:]
                res <<=8
                res += eval('0x' + s2)
        return res

def sese(s):

        head = s[:20]
        type = s[36:40]

        serialID = s[24:36]
        nodeID = s[55:56]
        seq = s[40:44]

        batt = s[60:64]


        if type == "0070" : # TH
                #print s
                print "battery : " , bigEndian(batt)
                temperature = bigEndian( s[64:68] )
                v1 = -39.6 + 0.01 * temperature

                t = int(time.time())
                print "temperature %d %.2f nodeid=%d" % ( t, v1, bigEndian( nodeID ) )
                // TSD에 데이터 를 시각적으로 보기위해 여기에 insert( t, v1, bigEndian( nodeID ) ) 삽입

        else:
                #print >> sys.stderr, "Invalid type : " + type
                pass

if __name__ == '__main__':

        tmpPkt = []
        flag = 0

        test = serial.Serial("/dev/ttyUSB0", 115200)

        while 1:
                Data_in = test.read().encode('hex')

                if(Data_in == '7e'):
                        if(flag == 2) :
                                flag =0
                                tmpPkt.append(Data_in)
                                packet = ''.join(tmpPkt)

                                # send packet
                                sese(packet)

                                tmpPkt = []
                                sys.stdout.flush()
                        else :
                                flag = flag + 1
                                tmpPkt.append(Data_in)
                else :
                        if(flag == 1 and Data_in =='45') :
                                flag =2
                        tmpPkt.append(Data_in)

