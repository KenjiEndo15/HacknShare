# RTO 2 - Code Shenanigans


## C and C&#43;&#43;
### typedef
A `typedef` in C and C&#43;&#43; is used to create an alias for an existing data type.

It allows you to define a new name for a type, making your code more readable and easier to manage.

For example:
```C&#43;&#43;
typedef unsigned long ulong;
```
Now, `ulong` can be used as an alias for `unsigned long`:
```C&#43;&#43;
// Equivalent to unsigned long myVariable.
ulong myVariable;
```
It doesn&#39;t create a new type but provides a shorthand or more meaningful name for an existing type.

### using
`typedef` is more traditional (C and C&#43;&#43;) and can be a bit less intuitive for complex types like function pointers.

`using` (type aliases) is more modern and often clearer, especially with template types.

Both achieve the same goal of creating type aliases, but using is preferred in modern C&#43;&#43; for its simplicity and readability.

### ZeroMemory
Fills a block of memory with zeros.

To avoid any undesired effects of optimizing compilers, use the `SecureZeroMemory` function.

### CloseHandle
This is essential for resource management because failing to close handles can lead to resource leaks, which may eventually cause performance problems or other issues.

### TCHAR
`TCHAR` is a Windows-specific type that represents either a wide-character (`wchar_t`) or a narrow-character (`char`) depending on whether the program is built for Unicode or ANSI.

If the program is compiled with the `UNICODE` flag set, `TCHAR` resolves to `wchar_t`, otherwise, it resolves to `char`.

### NULL terminator
Null terminator (\0) is important in a string array to tell where it ends (when constructing manually the array), otherwise it&#39;s automatic..

In C and C&#43;&#43;, all string literals—whether they are regular (&#34;string&#34;) or wide-character (L&#34;string&#34;) — always include a null terminator (\0) at the end, as mandated by the language standards.

## C#
### extern
`extern` is used in C# to declare a method or function that is defined outside of the current program, typically in an unmanaged (native) library or external assembly.

### Delegates
A delegate is a type that represents references to methods with a particular parameter list and return type. When you instantiate a delegate, you can associate its instance with any method with a compatible signature and return type. You can invoke (or call) the method through the delegate instance.

---

> Author:   
> URL: http://localhost:1313/posts/rto2_code-shenanigans/  

