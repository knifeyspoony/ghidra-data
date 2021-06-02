# Generating Ghidra Data Type (GDT) Archives
You can create GDT files via the Ghidra Code Browser Window:
1. Select File->Parse C Source
2. Choose an applicable parse configuration, or clear an existing configuration and "Save As" to retain the settings.
3. Use the `+` button to add your header file(s)
4. Select "Parse to File..." to create a _.gdt_ file.

# Windows GDT Files

## Errors
If you're parsing Windows headers, chances are there will be a TON of errors. The default location for the error output is in your user home directory within a file called _CParserPlugin.out_, e.g.:
`%HOMEPATH%/CParserPlugin.out`. With enough time and perseverance, you can likely address all of the errors, but it's very tedious. 

## Preprocessed Files
An alternative to parsing Windows headers directly is to deal with preprocessed files:
1. Create a project using Visual Studio (e.g. VS2019).
2. Create a source file which includes all of the headers you're interested in capturing definitions for.
3. Set the compile options for "Preprocess to a File (/P)" and "Preprocess Suppress Line Numbers (/EP)".
4. Compile your project. Linking will fail, but you'll end up with a single _.i_ file e.g. _main.i_ which will have all of the preprocessed structs, enums, function prototypes, etc.
5. Rename this to _main.h_ or something appropriate for your project.
6. Clean up this file (TODO: automate this part). See [Cleaning up Preprocessed Output](#cleaning-up-preprocessed-output) for an overview of this process.
7. Follow the steps in [Generating Ghidra Data Type (GDT) Archives](#generating-ghidra-data-type-gdt-archives) to create a _.gdt_ file.

## Cleaning up Preprocessed Output
1. Delete all pragmas: `#pragma[^\n]+\n` (Replace with empty string)
2. Remove all inline/forceinline bodies: `(?:__forceinline|__inline)((?:[^{]*\n[^{]*))\{(?:(?>\{(?<LEVEL>)|\}(?<-LEVEL>)|(?!\{|\}).\s*)+(?(LEVEL)(?!)))\}` (Replace with $1;)
3. Remove all __declspecs -> `__declspec\([^\)]+\)+` (Replace with empty string)
4. Add required defines, e.g.:
```C
	#define __pragma(x)
	#define __unaligned
	#define unsigned char _Bool
```
> **NOTE:** You can get feedback from Ghidra at any stage of this process by attempting to create a GDT file from your header(s) and then referring to `%HOMEPATH%/CParserPlugin.out` to see which line causes the parser to fail.

# Repository Contents

## Process Hacker GDT
Process Hacker maintains a collection of Windows Native API headers (unpublished by Microsoft). The GDT based on these headers was generated via the following process:

### Requirements
* [Process Hacker source](https://github.com/processhacker/processhacker)
* [Windows 10 SDK, Version 2004](https://developer.microsoft.com/en-us/windows/downloads/sdk-archive/)
* [VS2019](https://visualstudio.microsoft.com/downloads/)
> **NOTE:** I believe the Community edition of Visual Studio 2019 would work for this, but I didn't try it.

### Creating a Preprocessed Header
1. Clone https://github.com/processhacker/processhacker
2. Create a project in VS2019 that has the 19041 headers (Windows Kits/Include/19041/...) on the include path
3. Add the processhacker/phnt/include directory to the include path
4. Set the compile options for "Preprocess to a File (/P)" and "Preprocess Suppress Line Numbers (/EP)"
5. Add a simple source file:
```C
	#include <phnt_windows.h>
	#include <phnt.h>
	int main() {
		return 1;
	}
```
6. Compile
7. [Clean up the preprocessed output](#cleaning-up-preprocessed-output)
8. [Create a GDT with Ghidra](#generating-ghidra-data-type-gdt-archives)