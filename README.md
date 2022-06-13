# Tool Description Library (TDL)

## Introduction
The goal of this library is to export tool description formats like [CTD](https://github.com/WorkflowConversion/CTDSchema) or [CWL](https://www.commonwl.org/).

The current CLI interfaces of OpenMS and SeqAn3 are very different. Merging command line parsing into a library is currently (2022) not feasible for these projects. While OpenMS already has support for CTD both are missing support for CWL. The idea is to replace the CTD exporter of OpenMS with the TDL implementation and integrating TDL into SeqAn3. As a next step TDL will gain support for CWL and with (hopefully) minimal code adjustments this
will give OpenMS and SeqAn3 based tools easy access to CWL tool description support.

## Usage (C++20)
At the core of TDL is the `ToolInfo` structure. It consist of three values:
```
struct ToolInfo {
    MetaInfo                meta{};
    std::vector<Node>       params{};
    std::vector<CLIMapping> cliMapping{};
};
```

Where `meta` carries meta information of the
  - meta data about the tool
  - a tree of parameters `struct Node`
  - a tree of mapping from parameters to CLI.

 The `Node` is defined as:
 ```
struct Node {
    using Children = std::vector<Node>;
    using Value = std::variant<BoolValue,          // just a single bool value
                               IntValue,           // single int, double or string value
                               DoubleValue,
                               StringValue,
                               IntValueList,       // list of int, double or string values
                               DoubleValueList,
                               StringValueList,
                               Children>;          // not a value, but a node with children

    std::string name{};           //!< Name of the entry.
    std::string description{};    //!< Entry description.
    std::set<std::string> tags{}; //!< List of tags, e.g.: advanced parameter tag.
    Value value{Children{}};      //!< Current value of this entry
};

```

A CTD file is generated by calling `convertToCTD`
```
auto toolInfo = ToolInfo {
    ...
};
auto ctdAsString = convertToCTD(toolInfo);
std::cout << ctdAsString;
```

## Examples
- [Detailed Example](Example01.cpp.md)
- [Complete Example](Example00.cpp.md)

## Special cases
### CTD
  - string values with tags `input file`, `output file` or `output prefix`
    are exported as CTD typed values `input-file`, `output-file` or output-prefix`.
    The list valid string values are interpereted as `supported_formats`
  - list of string values with tags `input file` or `output file`
    are exported as CTD typed values `input-file` or `output-file`.
    The list valid string values are interpereted as `supported_formats`
