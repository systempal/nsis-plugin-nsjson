# nsJSON NSIS plug-in

Authors: Stuart 'Afrow UK' Welch, Pieter Dewachter

Date: 10th October 2024

Version: 1.1.2

A JSON (JavaScript Object Notation) parser, manipulator and generator plug-in for NSIS.

See `examples/*.nsi`.

## About JSON

See http://www.json.org/ for information about JSON along with the syntax and string escape sequences.

## Important information

All plug-in functions set the NSIS error flag on error. For functions that return a value on the stack, no item will be returned if the error flag has been set. You can use `IfErrors` or `${If} ${Errors}` to check that a call has succeeded before Pop-ping a value, if any.

The plug-in supports all escape sequences in string values:
`\r`, `\n`, `\t`, `\b`, `\f`, `\"`, `\\` and `\uXXXX`

`[NodePath]`, used throughout this readme, is any space-delimited list of mixed node keys and indices (prefixed with /index). For example:

```
{
    "node1": {
        "node2": false,
        "node3": [ "Stuart", 1, { "node4": "Welch", "node5": "" }, 32.5, [ 0, "test", false ] ]
    }
}
```

This is a JSON snippet. "node2" is a child node of "node1". "node2" has a Boolean value of false. "node3" is an array (denoted by square brackets). The 3rd element within the array is another node. The 5th element within the array is another array.
These are examples of node paths within the JSON snippet:

"node1" "node2"
- The path for "node2" (value: false)
- `nsJSON::Get "node1" "node2" /end`

"node1" "node3" /index 0
- The path for the first array element in "node3" (value: "Stuart")
- `nsJSON::Get "node1" "node3" /index 0 /end`

"node1" "node3" /index 2 "node4"
- The path for "node4" (value: "Welch")
- `nsJSON::Get "node1" "node3" /index 3 "node4" /end`

/index 0 "node3" /index 2 /index 0
- Also the path for "node4" (value: "Welch")
- `nsJSON::Get /index 0 /index 1 /index 3 /index 0 /end`

"node1" "node3" /index 4 /index -2
- The path for value "test" in the array; note the negative index
- `nsJSON::Get "node1" "node3" /index 4 /index -2 /end`

Node names containing double quotes are automatically escaped. If no node path is given, the root node is used.

Multiple JSON trees can be manipulated at the same time. Add `/tree Tree` (where Tree is the tree name) before all other plug-in arguments to specify which JSON tree you are manipulating.

## Reading JSON files

`nsJSON::Set [/tree Tree] [NodePath] /file [/unicode] "path\to\input.json"`

Loads the JSON from the given file into the given node. Specify `/unicode` if the input file is Unicode.

---

`nsJSON::Set [/tree Tree] /file [/unicode] "path\to\input.json"`

Loads the JSON from the given file into memory, overwriting any existing JSON tree. Specify `/unicode` if the input file is Unicode.

## JSON from HTTP web requests

`nsJSON::Set [/tree Tree] [NodePath] /http ConfigTree`

Executes an HTTP web request and loads the JSON response into the given node. The ConfigTree JSON tree specifies the web request configuration to execute, described using JSON. Its possible values are as follows...

"**Url**": "http://..."
- The web request URL. This is required.

"**Verb**": "GET"
- The request verb, such as GET or POST. Default is GET.

"**Params**": "..."
- Optional parameters to append to the URL (after ?).

"**ParamsType**": "..."
- The parameters type. Possible values are...

    Empty string/omitted (default)
    - Parameters are given as JSON and will be sent as a standard key/value pair string. Special characters will be URL-encoded automatically. Arrays will be serialized as multiple keys of the same name with a [] suffix.

      For example `{"a": "Value1", "b": true, "c": "Value3"}` becomes `a=Value1&b&c=Value3`, and `{"Array": [1, 2]}` becomes `Array[]=1&Array[]=2`.

    "Raw"
    - Raw parameters are given as a string.

"**Data**": "..."`
- Optional request data to send (typically used with POST).

"**DataType**": "..."
- The request data type. Possible values are...

    Empty string/omitted (default)
    - Data is given as JSON and will be sent as a standard POST key/value pair string. Special characters will be URL-encoded automatically. Arrays will be serialized as multiple keys of the
      same name with a [] suffix.

      For example `{"a": "Value1", "b": true, "c": "Value3"}` becomes `a=Value1&b&c=Value3`, and `{"Array": [1, 2]}` becomes
      `Array[]=1&Array[]=2`.

    "JSON"
    - POST data is JSON and will be sent as-is (serialized JSON).

    "Raw"
    - Raw POST data is given as a string.

"**DataEncoding**": "..."
- Specifies the encoding to use for the POST data. Possible values
    are...

    Empty string/omitted (default)
    - Data is encoded as 1 byte per character.

    "Unicode"
    - Data is encoded as 2 bytes per character.

  "Headers": "..."
  - Additional headers to send, which will override these defaults...
    * "Content-Type: application/x-www-form-urlencoded" is sent if "Verb" is "POST" and "DataType" is "Raw".
    * "Content-Type: application/json" is sent if "Verb" is "POST" and "DataType" is "JSON".
    * "Accept-Encoding: gzip,deflate" is sent if "Decoding" is set to true.

    Specify headers as JSON, for example: `{"Content-Type": "text/plain", "a": "b"}`


  "Agent": "nsJSON NSIS plug-in/1.0.x.x"
  - The request agent string. Default is shown.

  "Decoding": false
  - Enables automatic GZIP decompression by sending the "Accept-Encoding: gzip,deflate" request header. If the server responds with a valid "Content-Encoding" header, the compressed data will be decompressed automatically.

  "AccessType": "..."
  - The connection access type. Possible values are...

    Empty string/omitted (default)
    - Direct access (bypasses any proxy).

    "PreConfig"
    - Use the internet configuration defined in the Windows registry.

    "Proxy"
    - Use defined proxy configuration (see below).

"**Proxy**": { "Server": "...", "Bypass": "...", Username": "...", "Password": "..." }
- Specifies the proxy configuration. The "Bypass" value is optional.

"**Username**": "..."
- Username for HTTP authentication.

"**Password**": "..."
- Password for HTTP authentication.

"**ConnectTimeout**": "..."
- The initial connection timeout. A value of `0XFFFFFFFF` will disable the timeout.

"**SendTimeout**": N
- The request send timeout.

"**ReceiveTimeout**": N
- The response receive timeout.

"**UnicodeOutput**": true
- The response must be read as Unicode rather than ASCII.

"**RawOutput**": true
- The response must be read as raw text rather than JSON.

"**Async**": true
- Perform the request asynchronously. You must use the Wait function to wait or check if the request has finished.

"**UIAsync**": true
- For performing the request in a page callback function. This will ensure window messages are processed so that the UI does not become unresponsive. After the request has finished, the node specified by NodePath will contain the following values:

  "Output": ...
  - The HTTP response as JSON.

  "StatusCode": N
  - The HTTP status code.

  "ErrorCode": N
  - The WinINet/Win32 error code on error.

  "ErrorMessage": "..."
  - The WinINet/Win32 error message on error.

## JSON from console application execution

`nsJSON::Set [/tree Tree] [NodePath] /exec ConfigTree`

Executes a console application and loads the output into the given node. The ConfigTree JSON tree specifies the console application to execute, described using JSON. Its possible values are as follows...

  "Path": "$INSTDIR\ConsoleApp.exe"
  - The path to the executable to be executed. This is required.

  "Arguments": ...
  - The command line arguments. The value can be a string or an array of strings.

  "WorkingDir": "..."
  - The working/current directory. If not specified, the installer's working directory will be used, which is $OUTDIR.

  "Input": ...
  - The input to write to STDIN. The value can be a string or an array of strings.

  "UnicodeInput": true
  - The standard input must be written as Unicode rather than ASCII.

  "UnicodeOutput": true
  - The standard output must be read as Unicode rather than ASCII.

  "RawOutput": true
  - The standard output must be read as raw text rather than JSON.

  "Async": true
  - Execute asynchronously. You must use the Wait function to wait or check if the process has finished.

  "UIAsync": true
  - For performing the execution in a page callback function. This will
    ensure window messages are processed so that the UI does not become unresponsive.

  After execution has finished, the node specified by NodePath will
  contain the following values:

  "Output": ...
  - The output of the console application, depending on the RawOutput setting.

  "ExitCode": X
  - The process exit code.

## Modifying JSON

`nsJSON::Set [/tree Tree] [NodePath] /value "Value"`

Sets the value of the given node. The value can be any single value or it can be JSON code. The node will be created if it does not exist.

---

`nsJSON::Delete [/tree Tree] [NodePath] /end`

Deletes the given tree or node. `/end` must be added to the end of the list to prevent stack corruption.

---

```
nsJSON::Quote [/unicode] [/always] Value
Pop $Var
```

Surrounds the given value with quotes if necessary and escapes any characters that require it. The quoted value will be returned on the stack.

Optionally specify `/unicode` to escape non ASCII characters as well although it is perfectly legal to include Unicode characters in JSON.

Specify `/always` to always surround the input string in quotes even if it is already quoted.

---

`nsJSON::Sort [/tree Tree] [NodePath] [/options Options] /end`

Sorts the given node. Add up the following values for the optional sort options:

1. Sort in descending order.
2. Use numeric sort.
4. Sort case sensitively.
8. Sort by keys rather than by values.
16. Sort all nodes in the tree recursively.

## Reading JSON values

```
nsJSON::Get [/tree Tree] [/noexpand] [NodePath] /end
Pop $Var
```

Gets the value of the given node. The value returned will be JSON code if the node is not a value-only node. Specify `/noexpand` when reading quoted string values to stop escape sequences being expanded. `/end` must be added to the end of the list to prevent stack corruption.

---

```
nsJSON::Get [/tree Tree] /type [NodePath] /end
Pop $Var
```

Gets the value-type of the given node. It can be one of "node", "array", "string" (quoted strings), "value" (which is all non-string values such as integers, floats and Booleans) or an empty string if the node does not exist.

---

```
nsJSON::Get [/tree Tree] /key [NodePath] /end
Pop $Var
```

Gets the name (key) of the given node.

---

```
nsJSON::Get [/tree Tree] /keys [NodePath] /end
Pop $VarKeyCount
Pop $VarKey1
Pop $VarKey2
Pop $VarKeyN
```

Gets the keys of the given node. `$VarKeyCount` will be the number of keys pushed onto the stack (to be removed via Pop).

---

```
nsJSON::Get [/tree Tree] /exists [NodePath] /end
Pop $Var
```

Determines whether or not the given node exists. `$Var` will be "yes" or "no".

---

```
nsJSON::Get [/tree Tree] /count [NodePath] /end
Pop $Var
```

Gets the number of child nodes for the given node or the number of elements if the node is an array.

---

```
nsJSON::Get [/tree Tree] /isempty [NodePath] /end
Pop $Var
```

Determines whether or not the given node is empty. `$Var` will be "yes" or "no". An empty node is for example: `"node": { }` or `"array": [ ]`.

## Generating JSON

```
nsJSON::Serialize [/tree Tree] [/format]
Pop $Var
```

Serializes the current JSON tree into $Var. Add `/format`  to apply formatting to the output. The error flag is set if an error occurs.

---

`nsJSON::Serialize [/tree Tree] [/format] /file [/unicode] "path\to\output.json"`

Serializes the current JSON tree into the given file. Add `/format` to apply formatting to the output. Add `/unicode` to generate a Unicode output file (applies to both ANSI and Unicode NSIS).

## Waiting for asynchronous tasks to finish

```
nsJSON::Wait Tree [/timeout TimeoutInMilliseconds]
Pop $Var
```

Checks whether or not the asynchonous task that is being ran under the given JSON tree has finished.

If `/timeout` is specified, the function will return after the specified number of milliseconds; after which `$Var` will be "wait" if the task has not finished; or an empty string otherwise.

When `/timeout` is omitted, the function will wait indefinitely and nothing will be pushed onto the stack when the task finishes.

---

## ⚠️ Differences in the Personal Version

This version is based on the official v1.1.1.1 release from [GitHub](https://github.com/Pieter-Dewachter/nsJSON/releases/tag/v1.1.1.1).

### New supported architecture: x64 (amd64-unicode)

The original v1.1.1.1 supports only:
- x86-ansi
- x86-unicode

The personal version adds support for:
- **amd64-unicode** (x64)

### Visual Studio Project

The original included:
- `Contrib/nsJSON/nsJSON.sln` - VS2012 Solution
- `Contrib/nsJSON/nsJSON.vcxproj` - VS2012 Project (x86 only)
- `Contrib/nsJSON/ConsoleApp/ConsoleApp.vcxproj` - Test application

The personal version uses the same project updated for VS2022 with x64 support.

### Added Files (build infrastructure)

- `build_plugin.py` - Python script to compile the plugin for all architectures
- `.gitignore` - Git exclusion file

### Removed Files

Pre-compiled DLL files have been removed from the distribution (they are generated by the build):

- `Plugins/x86-ansi/nsJSON.dll`
- `Plugins/x86-unicode/nsJSON.dll`

### Functional Changes

No functional changes compared to the original. Modifications are limited to the build infrastructure and x64 support.

### Build

```cmd
cd nsJson
python build_plugin.py
```

DLLs are copied to `plugins/{platform}/nsJSON.dll`.

### Build Options

```powershell
python build_plugin.py --configs x86-unicode      # Single architecture (x86-ansi|x86-unicode|x64-unicode|all)
python build_plugin.py --vs-version 2026          # Specific toolset (auto|2022|2026)
python build_plugin.py --clean                    # Clean dist/ before build
python build_plugin.py --rebuild                  # Force full rebuild
python build_plugin.py --verbosity detailed       # Extended MSBuild output
python build_plugin.py --install-dir "C:\NSIS\Plugins"  # Copy to additional NSIS directory
python build_plugin.py --list                     # List available configurations
```

---

*See [README_IT.md](README_IT.md) for the Italian version.*
