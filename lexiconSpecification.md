# 1 Introduction

This is the version of this lexicon specification.
0.2.0

Changelog:
0.2.0

* changed to json format
* reworked structure and content

0.1.0

* first version



## 1.1 Purpose



This document defines the format and required data fields for lexicons. To support that, it provides a general overview of the roll lexicons play within Project Mycelium, how they interact with plugins, and the version control procedures.



## 1.2 Project Mycelium Terminology

Project Mycelium: a long term project at InterMet to reduce redundant code bases and replace legacy software. This includes a wide variety of work and code.



The upper level program: a program that loads and uses plugins.



Plugin: a file containing Java code that is based on a common interface (defined in this specification).



pluginBase: the base code that is used to ensure consistency across plugins, and the central point for version control of the lexicon and plugin. This implements a Java interface.



Java interface: this is a feature of the Java language. It can be thought of as a template that does nothing on its own, but can be *implemented* by a plugin to make the place holders in the template functional. It is called an interface because that is exactly what it is - it establishes a common interface for implementations.



Device: referring to an external device such as a printer, sensor, or anything that requires inter-process communication.



Lexicon: a json file that defines a variety of information about how an upper level program should use a plugin. It includes identifying information about the plugin and a specification of the plugin's interface.



## 1.3 Structure of a Project Mycelium Program

A common plugin interface is defined in pluginBase, as well as the lexiconSpecification (this file). The lexiconSpecificationVersion originates from here (this repository).



Plugins are written per device, such as for a sensor or printer. They include pluginBase as a git submodule in order to access the interface and make an implementation (public class plugin implements pluginBase). Plugins have an their own version, and reference the version of the lexicon/pluginBase.



An upper level program is written that is able to load plugins, and is made to require a lexiconSpecificationVersion (respecting semantic versioning rules for compatibility). It follows a multiple step operation.

1. At runtime, the upper level program loads plugins and the lexicon.
2. The lexicon is parsed per the lexiconSpecificationVersion that the upper level program requires. This is used to determine what plugin to load, and can be used to determine if a plugin offers the messages that the upper level program requires (if it isn't locked to a device).
3. Plugins are run with Java's dynamic compilation feature. Plugins also load the lexicon.
4. The upper level program uses the information in the lexicon to determine how to properly call interface functions in the plugin, and how to interpret returned values. Likewise with the plugins. Plugins may also get information from the lexicon that is only used by the plugin.



## 1.4 Version Control

**Nathan needs to write this section**



# 2 Interface

## 2.1 Interface Functions

Implementations support calls to the following functions, per the implementation of pluginBase. This is not a specification for code, only for reference. Refer to the pluginBase code for the code.

* messageType\_q: query. Data is transferred to the upper level program once.
* messageType\_c: command. Data is transferred to the plugin once.
* messageType\_p: parameterized query. Data is transferred to the plugin, then to the upper level program.
* startComm: starts communication with the device and opens/creates all system functions (such as ports).
* stopComm: stops communication with the device and closes/removes all system functions (such as ports).
* generic: a placeholder function template that is defined on a per plugin basis to support functionality beyond the lexiconSpecification. Based on messageType\_p.
* Additional functions supporting implementation of plugins within Java. These are part of the API and are subject to version control like everything else, but are not part of the lexicon concept. Consequently, no additional detail will be provided in this document.



Every plugin shall support calls to all standard interface functions without stopping program execution. Refer to the errors section for more information.



## 2.2 Interface Data

All data transferred to or from a plugin by interface functions is passed as a byte array, even if it has a length of 1 (refer to pluginBase for the implementation). The plugin and upper level program must then type cast according to the type specified in the lexicon. The valid types are defined as part of the messageType Json name (keep reading).



# 3 Format

Lexicons are written in json format. The data is principally represented as a single object containing multiple members (name-value pairs). Depending on the name used, other data may follow. It is the purpose of this specification to define what the expected names are, and what values should be paired with them. The structure of the document herein mirrors the json format. Subsections (1.1) correspond to a name-value pair, and a further subsections (1.1.1) correspond to arrays that are used as member values.



Refer to lexiconExample for reference.



Json naming conventions are generic, such as "name", and consequently do not blend well with common English when specificity is required. To address this issue, json terms will be indicated as belonging to json terminology with "json", as in "json name" from herein.



## 3.1 lexiconSpecificationVersion

Required:	yes
Json type:	string



This refers to the last lexiconSpecification.md version that the lexicon was verified to be compliant to. This will typically be the version of the lexiconSpecification that the lexicon was written under.



The data in the string shall be in the format of major.minor.patch, such as "1.2.3". Numbers may not be omitted, and no additional text is allowed.



## 3.2 lexiconVersion

Required:	yes
Json type:	string



This refers to the version of this particular lexicon file.

The data in the string shall be in the format of major.minor.patch, such as "1.2.3". Numbers may not be omitted, and no additional text is allowed.



## 3.3 deviceName

Required:	yes
Json type:	string



The deviceName is a human readable name for the device, such as "Dymo Label Writer 450". It is controlled at the lexiconVersion patch level. Do not use the deviceName to identify devices in code.



## 3.4 deviceID

Required:	yes
Json type:	number (restricted to positive integers)



The deviceID is number that is unique to that device's model (or the highest level of organization for which the plugin is compatible with). It is controlled at the lexiconVersion major version level and therefore should not change often. Use the deviceID to identify devices in code, such as specifying that a particular device be used with an upper level program.



## 3.5 protocol

Required:	no

Json type:	string



A string representing the protocol used to communicate with the sensor. Valid values are:

* "usb" for any USB protocol (requires protocolTokens USB\_vendorID and USB\_productID)
* "serial" for devices communicating via COM ports (requires protocol tokens serial\_port and serial\_baud)
* "IP" for TCP and UDP



For devices that do not require a protocol connection, such as a printer that is using an OS utility to communicate, 



**Travis: I have no intention of implementing IP for the current project. I have not decided on the required json members, either.**



## 3.6 protocolTokens

Required:	yes, if protocol=usb, serial, or IP

Json type:	array



An array of tokens used to supply information about the device's communication. This may be used for multiple purposes. Example A: an upper level program may use them to get information from the user to establish a "serial" connection. Example B: they may act as hard coded data that is utilized by the plugin.



### 3.6.1 USB\_vendorID

Required:	yes, if protocol=usb

Json type:	string



The USB vendor ID of the device expressed as a 2 byte hexadecimal number.



### 3.6.2 USB\_productID

Required:	yes, if protocol=usb

Json type:	string



The USB product ID of the device expressed as a 2 byte hexadecimal number.



### 3.6.3 serial\_port

Required:	yes, if protocol=serial

Json type:	number



The number of the com port to connect to, expressed as the number only. IE COM1 is expressed as 1. Set this to -1 to indicate that the upper level program should determine the serial port (in which case a message needs to be provided to set the port number).



### 3.6.4 serial\_baud

Required:	yes, if protocol=serial

Json type:	number



The baud rate to open the serial port at.



Note: all serial ports will open with 8N1 encoding and no flow control for the time being. When those get added, additional serial protocolTokens will need to be added.



## 3.7 description

Required:	no

Json type:	string



This is an optional human readable description of the device. The maximum length is 100 characters.



## 3.8 messages

Required:	yes

Json type:	array of objects



"messages" is a json array of objects, each with required members. Each json object represents a command that is valid to send to the plugin via messageType\_x interface functions. The json object can contain any number of entries, including none.



All messages require the same json members. A brief summary is:

* messageName is used to identify a functional group of messages
* messageType identifies which messageType\_x functions are supported
* messageDescription is a description of what the message does
* messageTokens contains data type and length information for using with messageType\_x functions
* messagePanelClass is the name of the Java class to use



All message entries follow standard API semantic versioning rules, as they are an API.



### 3.8.1 messageName

Required:	yes

Json type:	string



The messageName is a character string that identifies a function. This is processed by messageType\_x functions to determine the required task.



Changes to existing messageNames are controlled with the lexiconVersion major version. Adding new names is controlled with the lexiconVersion minor version.



### 3.8.2 messageType and messageType\_x

Required:	yes

Json type:	object



The messageType json member is an object containing 1 boolean member for each possible message type. The names match the messageType\_x interface functions.



### 3.8.3 messageDescription

Required:	yes

Json type:	string



This is a description of what the message does that is controlled with lexiconVersion patches. The maximum length is 100 characters.



### 3.8.4 messageTokens

Required:	yes

Json type:	object



Recall that the messageType\_x interface functions only accept or return byte arrays. The data in message tokens informs the plugin and the upper level program how to type cast the bytes into useful numbers.



The possible json members are a matrix of messageType and function (return, argument, length). Combinations that are not applicable are omitted:

* return\_q (type)
* return\_p (type)
* argument\_c (type)
* argument\_p (type)
* return\_q\_len (number, positive integer)
* return\_p\_len (number, positive integer)
* argument\_c\_len (number, positive integer)
* argument\_p\_len (number, positive integer)



Json members are only required if their matching messageType\_x entry is true.



The json names ending with \_len specify the *minimum* expected length of the array in multiples of "type". The valid types are:

* float, 32 bit
* double, 64 bit
* byte, 8 bit
* short, 16 bit
* int, 32 bit
* long, 64 bit
* ascii, wrapper for byte that notifies the upper level program that it should be interpreted as ASCII characters without escape sequences (raw ascii). **Travis: the point here is to differentiate between UTF-8 and ascii encoding with typing. Bad idea? My main motivation here is to explicitly support normal ascii (encoding + character set). It seems to be more common now to support UTF-8 (encoding) and unicode (character set). I think the Json standard requires the latter, but I want to support ascii because it is comparatively simple to decode and the tool of choice for things like the XQ2.**
* time, wrapper for long that contains a unix timestamp in milliseconds



Note that ASCII types may have a minimum length, but it is likely and acceptable that implementations of plugins will expect the upper level program to check the length of the byte array.



Some commands may not require an input (such as triggers), in which case define an argument of byte and length of 0.



### 3.8.5 messagePanelClass

Required:	yes

Json type:	string



This is the name of the Java class to call in order to use this message. (**Travis: can you elaborate in a technically accurate way? The extent of elaboration is up to you, but how to get this string would be good to document.**)



# 4 Plugin Errors

If a plugin receives erroneous data from the upper level program, then it shall \_\_\_\_\_\_.
**Travis: this needs to be decided on. I don't think this can be enforced by pluginBase because each function will be implemented for each plugin. Thus a standard behavior should be defined here. It does need to be defined because each messageType interface function will need to return something even passed invalid tokens (mainly applicable to parameterized queries).**

# 5 Standard Messages

**Nathan: fill this in soon.**















