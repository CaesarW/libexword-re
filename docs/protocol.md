# CASIO Ex-Word Protocol Documentation

The protocol used by Casio EX-Word electronic dictionaries is a slightly
modified version obex. This custom version prefixes each request with a
sequence number. The response packet is then preceded by a one byte packet
containing the sequence number of the request.

Example of a request/response exchange

Request: 

```
<seq> <opcode> <packet length> <headers...>
```

Response: 
```
<seq>
<response code> <response length> <headers...>
```

## List of commands:

* Connect

  This command must be the first one you send and is used to make a connection to the obex server on the dictionary.  

  The request/response for the connect string is defined the same as in standard obex except that in the request the version byte is used to specify operating mode, there are also an additional three bytes with the values ```0x40```, ```0x00``` and a third byte specifying the region.  

  * Valid regions:

    | Country  | Value      |
    | -------- | ---------- |
    | Japanese | ```0x20``` |
    | Korean   | ```0x40``` |
    | Chinese  | ```0x60``` |
    | German   | ```0x80``` |
    | Spanish  | ```0xa0``` |
    | French   | ```0xc0``` |
    | Russian  | ```0xe0``` |

  * Modes:
  
     | Mode       | Value                      |
     | ---------- | -------------------------- |
     | TextLoader | ```<region>```             |
     | Library    | ```<region>``` - ```0xf``` |
     | CDLoader   | ```0xf0```                 |
     
  
* Disconnect

  This command sends a disconnect to the dictionary and shuts down the USB device. It uses the standard obex disconnect opcode with no additional headers.

* Setpath

  Changes the current path on the dictionary.  

  Each path name needs to begin with either ```/_INTERNAL_00``` or ```/_SD_00``` which specify which storage device we are accessing either internal memory or the currently inserted SD card.  

  * Request:

    Uses the ```0x85``` (setpath) opcode followed by a name header containing the Unicode path name.
    
  * Response:
    
    Contains no additional headers.

* Capacity

  Retrieves capacity of currently selected storage device.  

  * Request:

    Uses the ```0x83``` (get) opcode followed by a name header containing the Unicode string "_Cap".
    
  * Response:
    
    The response returns a length header and end of body header. The length header contains the length of data sent in the end of body header and should contain the value 8. The end of body header contains two 4 byte numbers in network byte order with the first representing the total capacity and the second one represents amount of space used.

* Model

  Get model information.

  * Request:

    Uses the ```0x83``` (get) opcode followed by a name header containing the Unicode string "_Model".
  * Response:
    The response returns a length header and end of body header. The length header contains the length of data sent in the end of body header. The end of body header contains two null terminated strings containing the model information. So far the first one seems to always be the same for all models tested.

* List

  Return directory information for the currently selected path.

  * Request:

    Uses the ```0x83``` (get) opcode followed by a name header containing the Unicode string "_List".
    
  * Response:
    
    The response returns a length header and end of body header. The length header contains the length of data sent in the end of body header. The end of body header contains the directory information as an array of the following structure.
    
    ```c
    struct directory_entry {
    	uint16_t size;  //size of structure
    	uint8_t  flags; //file = 0, directory = 1, unicode/longname = 2
    	uint8_t  name[]; //name of file/directory
    }
    ```

* Remove
    Removes a file from currently selected path.

  * Request:

    Uses the ```0x82``` (put) opcode followed by a name header containing the Unicode string "_Remove". Following the name header are a length and end of body header. The length gives the length of data in the end of body header and the end of body header contains a null terminated string of the filename to remove.
    
  * Response:
  
    Contains no additional headers.
  

On DataPlus 5+ devices the name of the file is a ```utf16``` string when in Text mode.
    
* SdFormat
    Formats currently inserted SD Card

  * Request:
    
    Uses the ```0x82``` (put) opcode followed by a name header containing the Unicode string "_SdFormat". Following the name header are a length and end of body header. The length gives the length of data in the end of body header and the end of body header contains only a null character.
  
  * Response:
  
    Contains no additional headers.
  
* Send file
  
  Sends a file to the dictionary.
  * Request:
    
      Uses the ```0x82``` (put) opcode followed by a name, length, and body/end of body headers. The name header is a Unicode string containing the name of the file being sent, the length header should contain the total size of the file and the body/end of body headers contain the actual file data.  
      
      If the file is greater then max packet size it will be split across multiple put requests only the first one will include the name and length headers.
      
  * Response:
    
    Contains no additional headers.
  
* Get file
  
  Sends a file to the dictionary.
  
  * Request:
  
    Uses the ```0x83``` (get) opcode followed by a name header. The name header is a Unicode string containing the name of the file being retrieved.
  
  * Response:
    
    The response returns a length header and body/end of body headers. The length header contains the length of data sent in the body/end of body headers. The body/end of body headers contain the file data retrieved.  
    
    If the file is large enough(more then about 4k), it will be split across multiple response packets. In this case only the first response contains the length header.

* AuthChallenge

  Sends an authentication request to the dictionary. 

  This command must be issued in Library mode before many commands such as delete, upload, capacity, etc will work. Somewhat interestingly the List command does not require authentication and will if issued cause the other commands requiring authentication to work as if you had issued a successful AuthChallenge command.
  * Request:

    Uses the ```0x82``` (put) opcode followed by a name header containing the Unicode string "_AuthChallenge". Following the name header are a length and end of body header. The length should always be 20 and the end of body header should contain the the 20 byte authentication key.

  * Response: 

      Contains no additional headers.

* AuthInfo
  Resets dictionary authentication info stored in ```authinfo.inf```. 

  When run this command will also delete all dictionaries currently installed on device. 

  * Request:
    
      Uses the ```0x83``` (get) opcode followed by a name header containing the Unicode string "_AuthInfo". Following the name header is a 40 byte non-standard header (```0x70```) cantoning data used to generate a new authkey.

  * Response:
    
      The response returns a length header that and end of body header. The length header will always be 20 and the end of body header will contain the new 20 byte authkey.

* UserId
  Sets user id of dictionary.

  * Request:

    Uses the ```0x82``` (put) opcode followed by a name header containing the Unicode string "_UserId". Following the name header are a length header and end of body header. The length should always be 17 and the end of body header contains the new user name, this should be zero padded to a length of 17.
    
  * Response:
    
      Contains no additional headers.

* Unlock
  Unlock device for adding/removing add-ons.

  * Request: 

    Uses the ```0x82``` (put) opcode followed by a name header containing the Unicode string "_Unlock". Following the name header are a length and end of body header. The length should always be 1 and the end of body always contains a single null byte.
    
  * Response:
    
      Contains no additional headers.

* Lock
  Lock device after removing/adding an add-on.

  * Request:

    Uses the ```0x82``` (put) opcode followed by a name header containing the Unicode string "_Lock". Following the name header are a length and end of body header. The length should always be 1 and the end of body always contains a single null byte.

  * Response:
    
    Contains no additional headers.

* CName

  Specifies name of dictionary.

  This command is used to set the name of the add-on dictionary you wish to access. It should be sent immediately after then Unlock command.
    * Request:

      Uses the ```0x82``` (put) opcode followed by a name header containing the Unicode string "_CName". Following the name header are a length and end of body header. The length header contains the length of the end of body header and the end of body contains two null terminated sjis encoded strings. The first should be the add-on's five character ID. The second is the name of the add-on dictionary.

    * Response:

      Contains no additional headers.

* CryptKey
  
  Generates a new CryptKey. 
  
  The key returned is not used to directly encrypt the dictionary. Instead it is used to generate a secondary key that does the actual encryption.
  
  * Request:
  
    Uses the ```0x83``` (get) opcode followed by a name header containing the Unicode string "_CryptKey". Following the name header is a non-standard header (```0x71```) containing a 28 byte block used to generate the encryption key.
  
    The algorithm used to generate the 16 byte key looks like the following:
    
    ```
       b[0] b[1] b[2]  b[3]  b[4]  b[5]  b[6]  b[7]  b[8]  b[9]  b[10] b[11]
    +    0    0  b[16] b[17] b[18] b[19] b[20] b[21] b[22] b[23]    0     0
    ```
    
    The last 4 bytes are b[24] - b[27], however only the first 12 are returned in the response header.
    
  * Response:
    
      The response returns a length header that and end of body header. The length header will always be 12 and the end of body header will contain the first 12 bytes of the generated key.
