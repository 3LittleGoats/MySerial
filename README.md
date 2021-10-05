# MySerial
The story behind MySerial:
    This serial project started I believe about a year ago, when I tried to jump straight into making my own wireless communication protocol. It didn't work reliably with a wired connection at 50 bits per second (bps)! I finally gave up on wireless communication. I ditched the preamble, which seemed to add a lot of complication. I also learned that the data corruption problem I was having was not because the ATMEGAs I was using couldn't handle 50bps. It was because the receiver was receiving a bit exactly when the transmitter sent it, leaving the possibility for the receiver to receive a little early and catch the previous bit instead. I was also using the not-extremely-accurate 8MHz internal oscillator for the transmitter, allowing the transmitter and receiver to slowly drift out of sync.
    I used the 16MHz external oscillator instead. I made the receiver read each bit half a bit-time (the time between each bit) after it was sent in order to give it plenty of leg room for minor deviations. I also programmed the transmitter and receiver to resync after each byte, so that drift does not become a problem when sending large packets (groups of data). With these improvements, MySerial seemed to work perfectly at 10,000bps (although I normally use it at 1000bps), 200 times the speed of the wireless version I previously tried to make.


Technical information:
    Class name: MySerial
    
    Functions in this class:
        begin_t(byte pin, unsigned int bitTime, unsigned int lineUpTime)
            This function is used to set the serial up as a transmitter
            'pin' is the digital pin used to transmit data
            'bitTime' is the time between each bit in microseconds (uS) (1,000,000/bitrate)
            'lineUpTime' is the time that the transmitter initially pulls the data pin high for, in order to "arm" the receiver (let it know that there is incoming data). This is optional, and the default value of 1000 should work just fine
        
        begin_r(byte pin, unsigned int bitTime)
            This function is used to set the serial up as a receiver
            'pin' is the digital pin used to receive data
            'bitTime' is the time between each bit in microseconds (uS) (1,000,000/bitrate). This value must match the bit-time of the transmitter
        
        bool signal()
            This function tells whether or not there is an incoming message ('true' for incoming message, 'false' for no incoming message)
        
        listen()
            This function checks for an incoming message (I may merge 'signal' and 'listen' into one function in the future to make things a little simpler)
        
        int receive(byte _buf[255])
            This function receives incoming data
            '_buf' is where the bytes of data are stored (the array you put in this spot needn't hold 255 bytes. 255 is the absolute maximum number of bytes that can be received at once. If a smaller array is used, use caution, as Atmel chips [at least the ATMEGA] lock up if an array index gets out of range)
            This function returns the number of bytes it received (-1 if it timed out waiting for data)
        
        transmit(byte _buf[255], int byteTotal)
            This function transmits data
            '_buf' is where the bytes of data to be sent are stored (the array you put in this spot needn't hold 255 bytes. 255 is the absolute maximum number of bytes that can be received at once, so no more should be transmitted at once)
            'byteTotal' is the number of bytes to be sent. Telling the transmitter explicitly how many bytes are to be sent allows the use of a large byte array (such as 255) to be used to send varying numbers of bytes without having to change the length of that array to match the number of bytes being sent
    
    
    minimum number of bytes that can be transmitted/received: 0 (-1 has not been tested :) )
    
    maximum number of bytes that can be transmitted/received: 255 (this is because the very first byte of data sent tells the receiver how many bytes to receive)
    
    minimum bitrate: 15
    maximum stable bitrate: 10000 (20000 tested, with some corruption of data)
    maximum bit-time (corresponds to minimum bitrate): 65535
    minimum bit-time (corresponds to maximum bitrate): 100
    
    other notes:
        Internal oscillator does not seem to be accurate enough, and causes enough drift to corrupt transmitted data. An external oscillator is recommended
        This serial protocol can be used for two-way communication over the single data wire (in which case a delay, preferably the same value as the bit-time, between receiving and then transmitting a response is recommended; a pull-down [not pull-up] resistor on the data wire is also highly recommended)
        The general scheme of this serial protocol is:
            "arm" receiver
            send byte of data containing number of bytes in the incoming message
            resync (takes 2 bit-times)
            send first byte of message
            resync (takes 2 bit-times)
            send second byte of message
            resync (takes 2 bit-times)
            ...
            send last byte of message
            resync (takes 2 bit-times) (almost completely unnecessary this time, except that it helps ensure the receiver does not finish ahead of the transmitter)
        Since this protocol essentially sends 2 bits of data every byte solely to resync the transmitter and receiver, totaling 10 bits per byte of data, actual bitrates are 80% of the set bitrate (the receiver "arming" process and very first byte of data must also be factored in)



Feel free to suggest a better name for this protocol! This is the best name I could think of, the first being MyVirtualWire, then MyWire. :)
