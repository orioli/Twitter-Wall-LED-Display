Twitter-Wall-LED-Display
========================

Connects twitter updates to a bluefield LED display


#!/usr/bin/env python2.7

# author JOSE BERNEGUERES BLUEFIELD (TM) LED DISPLAY 2012 OCT 1 AL AIN UAEU MEDIALAB

import time
import twitter
import serial
import string

def removeNonAscii(s): return "".join(i for i in s if (s in string.ascii_letters or s in string.punctuation))

def tobin(a_char):
    return sum(map(ord,a_char))

def append_XOR( message ):
    mesage = removeNonAscii(message)
    xor = 0x54  # this is the blank message initial xor
    for c in message:
        xor = xor ^ tobin( c )
    return  " " + message + hex(xor)[2:].zfill(2).upper()

def sendtoLED(s,PAGE):

    port = "\\\\.\\COM3"
    try:
        led = serial.Serial(port, 9600,8,serial.PARITY_NONE,serial.STOPBITS_ONE, 1100, False,False,1000,True,None) # N 8 1 9600
        #print led.getSettingsDict()
    except Exception as ex:
        print "Failed to create a serial port on port:", com
        #raise ex

    h1 = '<ID00><L1><P'+str(PAGE)+'><FE><MA><WC><FE>'
    h2 = '<E>'
    packet = h1 + append_XOR( s.created_at + " "+ s.text ) + h2
    print packet
    x = led.write( packet )

    time.sleep(5) # let the LED PANEL DIGEST LAST ORDER
    led.flushOutput() #
    led.flushInput() #

    try:
        led.close()
    except Exception as ex:
        print "Failed to close the port:", com_port
        #raise ex


def main():
    
    api = twitter.Api()
    api = twitter.Api(consumer_key='ZUyo3xCIkxanvMgqCNlI7xA',
                          consumer_secret='TOj6bF6Se2t1nH3ebTAgyOKK9B9Giit1DM', access_token_key='24472870-tC9k4eEZZbvjJGdhW', access_token_secret='XLHgBCSyxhD8gLDy2LHg')
    for i in range(1000):
        statuses = api.GetUserTimeline('medialabAE')
        msgs =  [s for s in statuses ] # if s.favorited
        print "retrieved satuses = " ,len(msgs)
        if len(msgs)> 1:
            sendtoLED(msgs[0],"A")
        else:
            print "error"
        time.sleep(1000) # 20 minutes

    print "finsihed"


if __name__=="__main__":
    start = time.time()
    main()
    print "Elapsed Time: %s" % (time.time() - start)
