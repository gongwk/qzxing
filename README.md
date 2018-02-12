# qzxing
Qt/QML wrapper library for the [ZXing](https://github.com/zxing/zxing) barcode image processing library. 

Supports barcode decoding for the following types: 

 * UPC-A 	
 * UPC-E 	
 * EAN-8 	
 * EAN-13 	
 * ITF 	
 * Code 39 
 * Code 93 	
 * Code 128 (GS1) 	
 * Codabar 	
 * QR Code
 * Data Matrix
 * Aztec (beta)
 * PDF 417
 
 Supports barcode encoding for the following types:
 
 * QR Code
 
# Table of contents
1. [How to include](#howToInclude)
    1. [Embed the source code](#embedInSourceCode)
    1. [Compile the project as an external library](#externalLibrary)
    1. [Control dependencies](#controlDependencies)
        1. [QZXing (core)](#controlDependenciesCore)
        1. [QZXing (core + QML)](#controlDependenciesCoreQML)
        1. [QZXing + QZXingFilter](#controlDependenciesCoreQMLQZXingFilter)
1. [How to use](#howTo)
    1. [Decoding operation](#howToDecoding)
        1. [C++/Qt](#howToDecodingCPP)
        1. [Qt Quick](#howToDecodingQtQuick)
    1. [Encoding operation](#howToEncoding)
        1. [C++/Qt](#howToEncodingCPP)
        1. [Qt Quick](#howToEncodingQtQuick)
1. [Contact](#contact)

<a name="howToInclude"></a>
# How to include

The project can be used in two ways:
<a name="embedInSourceCode"></a>
## Embed the source code. 
Copy source code folder of QZXing to the root of your project. Add the following line to your .pro file. For more information see [here](https://github.com/ftylitak/qzxing/wiki/Using-the-QZXing-through-the-source-code).

```qmake
include(QZXing/QZXing.pri)
```
<a name="externalLibrary"></a>
## Compile the project as an external library
Open QZXing project (QZXing.pro) and compile. If it is needed to compile as static library, uncomment the following line in the .pro file.

```qmake
CONFIG += staticlib
```
<a name="controlDependencies"></a>
## Control dependencies 
Project file config tags are now introduced to be able to control the dependencies of the library accoring to the needs.
The core part requires only "core" and "gui" Qt modules. Though for backward compatibility "quick" Qt module is also required. 
The 3 level of dependencies are:

<a name="controlDependenciesCore"></a>
### QZXing (core) 
By including QZXing.pri or by building QZXing.pro you get the core functionality of QZXing which requires only QtCore and QtGui (because of QImage).

Warning! The initial default configuration till 20/03/2017 was including qzxing_qml. This tag could not be removed once added, so it was needed to be removed from the defaults. 

<a name="controlDependenciesCoreQML"></a>
### QZXing (core + QML) 
If an application is going to use QML functionality, it is now possible to add the dependency to it. This can be done by adding the folloing line to the .pro file of its project:
	
```qmake
CONFIG += qzxing_qml
```

<a name="controlDependenciesCoreQMLQZXingFilter"></a>
### QZXing + QZXingFilter 
QZXing includes QZXingFilter, a QAbstractVideoFilter implementation to provide a mean of providing live feed to the decoding library. It automatically includes QML implementation as well.
This option requires "multimedia" Qt module this is why it is considered as a separate configuration. It can be used by adding the folloing line to the .pro file of a project:

```qmake
CONFIG += qzxing_multimedia
```
<a name="howTo"></a>	
# How to use 

Follows simple code snippets that brefly show the use of the library. For more details advise the examples included in the repository and the [wiki](https://github.com/ftylitak/qzxing/wiki).

<a name="howToDecoding"></a>
## Decoding operation 

<a name="howToDecodingCPP"></a>
### C++/Qt 

```cpp
#include <QZXing.h>

int main() 
{
	QImage imageToDecode("file.png");
	QZXing decoder;
	decoder.setDecoder( DecoderFormat_QR_CODE | DecoderFormat_EAN_13 );
	QString result = decoder.decodeImage(imageToDecode);
}
```
	
<a name="howToDecodingQtQuick"></a>
### Qt Quick 

First register QZXing type to the QML engine.

```cpp
#include <QZXing.h>

int main() 
{
	...
	QZXing::registerQMLTypes();
	...
}
```
	
The in the QML file 

```qml
import QZXing 2.3

function decode(preview) {
	imageToDecode.source = preview
	decoder.decodeImageQML(imageToDecode);
}

Image{
	id:imageToDecode
}

QZXing{
	id: decoder

	enabledDecoders: QZXing.DecoderFormat_QR_CODE

	onDecodingStarted: console.log("Decoding of image started...")

	onTagFound: console.log("Barcode data: " + tag)

	onDecodingFinished: console.log("Decoding finished " + (succeeded==true ? "successfully" :    "unsuccessfully") )
}
```
 
<a name="howToEncoding"></a>
## Encoding operation 

<a name="howToEncodingCPP"></a>
### C++/Qt 

The encoding function has been written as static as it does not have any dependencies to data other than the ones provided by the arguments. 

Use the encoding function with its default settings: 

* Format: QR Code
* Size: 240x240
* Error Correction Level: Low (L)

```cpp
QString data = "text to be encoded";
QImage barcode = QZXing::encodeData(data);
```

Or use the encoding function with custom settings:

```cpp
QString data = "text to be encoded";
QImage barcode = QZXing::encodeData(data, QZXing::EncoderFormat_QR_CODE,
								QSize(width.toInt(), height.toInt()), QZXing::EncodeErrorCorrectionLevel_H);
```

<a name="howToEncodingQtQuick"></a>
### Qt Quick 

The encoding function can be easily used in QML through QZXing's Image Provider: "image://QZXing/encode/<data_to_be_encoded>". As with the C++ example, it can either be used with the default settings or with custom settings. 

Default settings:

```qml
TextField {
	id: inputField
	text: "Hello world!"
}

Image{
	source: "image://QZXing/encode/" + inputField.text;
	cache: false;
}
```

Or use the encoding function with custom settings that are passed like URL query parameters:

| attribute name | value      | description                                   |
| -------------- | ---------- | --------------------------------------------- |
| width          | int        | width in pixels                               |
| height         | int        | height in pixels                              |
| corretionLevel | L, M, Q, H | the error correction level                    |
| format         | qrcode     | the encode formatter. Currently only QR Code. |


```qml
TextField {
	id: inputField
	text: "Hello world!"
}

Image{
	source: "image://QZXing/encode/" + inputField.text +
					"?width=250" +
					"&height=250" +
					"&corretionLevel=M" +
					"&format=qrcode"
	cache: false;
}
```
 
<a name="contact"></a>
# Contact 
In case of bug reports or feature requests feel free to open an [issue](https://github.com/ftylitak/qzxing/issues). 
