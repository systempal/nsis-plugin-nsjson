**1.1.1.1** - 10th October 2024
- When deleting an element from an array, NSIS no longer crashes (was only working for the last element of an array before)
- Github migration (repository maintained by Pieter Dewachter)

**1.1.1.0** - 21st November 2017
- Fixed JSON with syntax errors still being parsed without setting the error flag.
- Fixed Set function not replacing the root value if the value was an array.
- Fixed Delete function not deleting the root node and tree.

**1.1.0.9** - 9th August 2017
- Fixed access violation and stack overflow crashes in JSON_Delete, which could occur on plug-in unload, Delete function call or when overwriting existing nodes with the Set function.
- Fixed DoHttpWebRequest failing when POST data exceeded 2048 characters.
- Fixed DoCreateProcess failing to read process output when the output exceeded 1024 characters.
- Delete function will now delete a complete node tree if no path is given.

**1.1.0.8** - 8th January 2017
- Added ErrorCode and ErrorMessage for HTTP requests for getting WinINet or Win32 errors.
- Added Sort function.

**1.1.0.7** - 7th November 2016
- Fixed /index for adding array elements with the Set function.
- Fixed file name being left on the stack after Serialize with /file.
- Added access type, proxy, authentication and timeout options for HTTP requests.
- Added POST data encoding options for HTTP requests.
- HTTP request headers can be given as raw headers or as JSON key/value pairs.
- Fixed a memory leak in JSON_SerializeAlloc and EscapePostData.
- POST parameters given as a JSON array was not escaped using EscapePostData.

**1.1.0.6** - 14th October 2016
- Output and ExitCode/StatusCode are always cleared for /exec and /http calls on the Set function.

**1.1.0.5** - 10th August 2016
- Empty value keys ("") were not included in serialized JSON.
- Serialize function popped one extra value from the stack.

**1.1.0.4** - 7th June 2016
- Fixed incorrect flags used for HTTP requests.
- Added support for negative indices.
- Added console application execution for input and output.
- Added support for asynchronous HTTP requests and console application execution.

**1.1.0.3** - 4th March 2016
- Fixed crash for the Delete function when specifying a tree/path that does not exist.
- Fixed Unicode build.
- Added /keys switch to Get function.

**1.1.0.2** - 9th December 2015
- Get function /type switch returns an empty string if the node does not exist.
- Added /always switch to Quote function which surrounds the input string in quotes even if it is already quoted.

**1.1.0.1** - 23rd November 2015
- Added Quote function.
- Added amd64-unicode build.

**1.1.0.0** - 19th April 2015
- Support for multiple JSON trees.
- JSON via HTTP web requests.

**1.0.1.3** - 18th October 2014
- Added UTF-16LE BOM, UTF-16BE BOM and UTF-8 signature detection for input files (with UTF-16BE conversion to UTF-16LE).
- Fixed formatting errors for the Serialize function.
- Fixed closing bracket or curly brace not being included on Serialize to stack when not using /format.
- Moved plug-in DLLs to x86-ansi and x86-unicode respectively for NSIS 3.0.

**1.0.1.2** - 12th July 2014
- Fixed crash on serialization to file for node values larger than 64KB.
- Fixed crash on serialization to stack for JSON larger than NSIS_MAX_STRLEN. The JSON will now be truncated.

**1.0.1.1** - 29th March 2014
- Fixed incorrect handling of escape character (\).

**1.0.1.0** - 28th August 2012
- Added /unicode switch to the Serialize function. Output files for both plug-in builds are now encoded in ANSI by default.
- Removed the Parse function in favour of Set /file [/unicode].
- Added /type, /key, /exists, /count, /isempty to the Get function.
- Added /index switch for referencing nodes by index.

**1.0.0.2** - 15th August 2012
- Fixed Unicode build parsing and serializing.

**1.0.0.1** - 1st July 2012
- Fixed parsing of single digit numbers.
- Fixed Serialize not writing the output file when the stack isn't
  empty.

**1.0.0.0** - 25th June 2012
- First version.