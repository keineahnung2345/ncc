## ncc

C/C++ Naming convention checker (ncc)

ncc is a development tool to help programmers write C/C++ code that adheres to
a some naming conventions. It automates the process of checking C/C++ code to
spare humans of this boring (but important) task. This makes it ideal for
projects that want to enforce a coding standard.

## Dependencies

The following packages are pre-requisites for ncc tool. Note that, the python 'clang'
provides libclang python bindings. The clang version should match the libclang.so version on your system

* clang
* yaml
* cppstyle

```python
pip install clang
pip install pyyaml
pip install cppstyle
```

### Windows

On Microsoft Windows install install 32 bit version of LLVM compiler tool chain.

## Usage

```python
ncc.py [--help] [--recurse] [--style STYLE_FILE] [--cdbdir CDBDIR] [--dump]
       [--output OUTPUT] [--filetype FILETYPE] [--exclude EXCLUDE [EXCLUDE ...]]
       [--path PATH [PATH ...]] [--include INCLUDE [INCLUDE ...]]
       [--definition DEFINITION [DEFINITION ...]] [--clang-lib CLANG_LIB]
```

For detailed description of all options:

```python
ncc.py --help
```

To print all rules:
```python
    ncc.py --dump
```

## Style Defintion

Style for c/c++ constructs are defined by regular expresssions. For e.g the below rules say that
a struct can have any character, a class name should begin with 'C', and a class member variable
should begin with m_, and a function name should begin with uppercase alphabet

```yaml
    ClassName: 'C.*'
    CppMethod: '[A-Z].*'
    VariableName:
        ScopePrefix:
            Global: 'g_'
            Static: 's_'
            ClassMember: 'm_'
        DataTypePrefix:
            String: 'str'
            Pointer: 'p'
            Integer: 'n'
            Bool: 'b'
        Pattern: '^.*$'
```

## Example Usage

### Check file test.cpp for naming convention violations:

```bash
    ./ncc.py --style examples/ncc.style --path examples/test.cpp

    examples/test.cpp:16:7: "B" does not match "^C.*$" associated with ClassName
    examples/test.cpp:28:9: "b" does not match "^m_.*$" associated with ClassMemberVariable
    examples/test.cpp:31:5: "main(int, const char **)" does not match "^[A-Z].*$" associated with FunctionName
    Total number of errors = 3
```

### Recursively find all files under 'examples' directory, but exclude '*.cpp' and '*.h' files

```bash
    ./ncc.py --style examples/ncc.style --recurse --exclude *.cpp *.h --path examples

    examples/test.hpp:4:7: "Test" does not match "^C.*$" associated with ClassName
    examples/test.hpp:12:9: "t" does not match "^m_.*$" associated with ClassMemberVariable
    examples/test.hpp:19:9: "_aaa" does not match "^[a-z].*$" associated with StructMemberVariable
    examples/test.c:8:9: "_b" does not match "^[a-z].*$" associated with StructMemberVariable
    examples/test.c:9:9: "_c" does not match "^[a-z].*$" associated with StructMemberVariable
    examples/tmp/test.hpp:4:7: "Test" does not match "^C.*$" associated with ClassName
    examples/tmp/test.hpp:12:9: "t" does not match "^m_.*$" associated with ClassMemberVariable
    examples/tmp/test.hpp:19:9: "_aaa" does not match "^[a-z].*$" associated with StructMemberVariable
    Total number of errors = 8
```

### Specify clang lib path, remove #include, write log into a text file
For example:
```sh
python ncc.py --style geotechnical.style --path <your_project_path> --clang-lib D:\LLVM\bin\libclang.dll --config ".cppstyle" -r -ri --exclude "<your_project_path>/<folder>\*" > mylog.txt 2>&1
```
Note: clanglib is installed on Windows along with LLVM, check [LLVM installation](https://github.com/keineahnung2345/cpp-code-snippets/tree/master/clang-format#on-windows) for more information.

## Troubleshooting
### clang/cindex.py UnicodeDecodeError

If you encounter the following error:
```
Traceback (most recent call last):
  File "ncc.py", line 688, in <module>
    call_checker(valid_paths, op, checker="cppstyle")
  File "ncc.py", line 628, in call_checker
    root = parser.parse(newpath)
  File "D:\anaconda3\lib\site-packages\cppstyle\model\parser.py", line 12, in parse
    return to_node(source.cursor)
  File "D:\anaconda3\lib\site-packages\cppstyle\model\parser.py", line 19, in to_node
    children = get_children(clang_node)
  File "D:\anaconda3\lib\site-packages\cppstyle\model\parser.py", line 68, in get_children
    node = to_node(c)
  File "D:\anaconda3\lib\site-packages\cppstyle\model\parser.py", line 21, in to_node
    comments = get_comments(clang_node)
  File "D:\anaconda3\lib\site-packages\cppstyle\model\parser.py", line 81, in get_comments
    if clang_node.raw_comment:
  File "D:\anaconda3\lib\site-packages\clang\cindex.py", line 1798, in raw_comment
    return conf.lib.clang_Cursor_getRawCommentText(self)
  File "D:\anaconda3\lib\site-packages\clang\cindex.py", line 229, in from_result
    return conf.lib.clang_getCString(res)
  File "D:\anaconda3\lib\site-packages\clang\cindex.py", line 104, in to_python_string
    return x.value
  File "D:\anaconda3\lib\site-packages\clang\cindex.py", line 89, in value
    return super(c_char_p, self).value.decode("utf8")
UnicodeDecodeError: 'utf-8' codec can't decode byte 0xef in position 71: invalid continuation byte
```

Open `D:\anaconda3\Lib\site-packages\clang\cindex.py`, revise:

```python
 @property
 def value(self):
     if super(c_char_p, self).value is None:
         return None
     return super(c_char_p, self).value.decode("utf8")
```

to:

```python
 @property
 def value(self):
     if super(c_char_p, self).value is None:
         return None
     return super(c_char_p, self).value.decode("utf8", "ignore")
```

## License

Copyright © 2018 Nithin Nellikunnu

Distributed under the MIT License (MIT).

## Thank You

Daniel J. Hofmann
