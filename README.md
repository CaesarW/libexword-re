# Description

_libexword_ is a library that implements the low-level _Apple OBEX_ commands which is used to communicate between PC and _Casio EX-word_ dictionaries. 

This library comes with a program called _exword_ that uses _libexword_ to allow manipulation of files on the dictionary.

Features:
 * list files on device
 * upload/download files
 * delete files
 * install/remove add-on dictionaries

# Installation

You can simple clone the repository and do a compilation.

## Dependencies

On _Debian based distros (*buntu)_: ` build-essential cmake libusb-1.0 libglib2.0-dev libreadline-dev libtool`

On *Arch Linux*: ```base-devel libusb cmake readline libtool glib2```

On _Fedora (RHEL)_: ...

## Compiling

On Terminal:

```bash
git clone https://github.com/CaesarW/libexword-re.git
cd libexword-re
mkdir build
cd build
cmake ..
make
```

# Commands
_Exword_ comes with various functions. Here is the list.

## Basic implementations

* **connect** [mode] [region]  
  This command connects to an attached EX-word dictionary. It accepts two optional parameters the first specifies the connection mode and be bee either library, text, or CD. The second one is the region of your dictionary and is a two letter country code.

	The default values for mode and region are library and _ja_.

* **disconnect**  
  This command simply disconnects from the currently connected dictionary.

* **model**  
  This command displays the raw model information about connected device.

* **capacity**  
  This command displays the capacity of the currently selected storage medium.

* **format**  
  This command will format the currently inserted SD card.

* **list**  
  This command will list all files and directories in your current directory.
  
  
## File Operations

* **delete** <filename>  
  This command deletes deletes the specified file from the currently set directory on the connected device

* **send** <filename>  
  This command will upload a file to the connected device. <filename> should specify the full path to the file on your local filesystem.

* **get** <filename>  
  This command will download a file from the connected device. <filename> should specify the full path on your local filesystem where the file should be saved.

* **setpath** <path>  
  This sets the current path on your dictionary. The path is set using the following format: <sd|mem://path>.

* **set** <option> [value]  
  This command sets various configuration options. If not value is given for the specified option its current value is printed out. 
	Options:
		debug - This option sets the debug level (0-5)
		mkdir - This option tells setpath if it should create non-existent directories (yes|no)

* **dict** <sub-function>  
  This command is used to manage installed add-on dictionaries. It only works when connected in library mode.
  
	* **reset** <username>  
		This will reset your authentication information using the specified username. When finished it will display the new authkey as well as save the username/authkey pair to users.dat file. It also deletes any already installed add-ons.
		
	* **auth** <username> [key]  
		This performs authentication and must be called before any other sub-function except for list and reset. If key is not specified it will try to find the username in the users.dat file.
		
	* **list**  
		This command will list currently installed dictionaries.
		
	* **decrypt** <id>  
		This downloads and decrypts the dictionary with the specified id.
		
	* **remove** <id>  
		This deletes the dictionary with the specified id.
		
	* **install** <id>  
		This will install the dictionary with the specified id.

# Credits

Brian Johnson [@brijohn](https://github.com/brijohn) is the creator of _exword_ power tool.  
Caesar Woo [@CaesarW](https://github.com/CaesarW) does the fork job and tries to bring modern compatibility to the software.  

