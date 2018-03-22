# DocXml - C# XML documentation reader 

[![Build status](https://ci.appveyor.com/api/projects/status/5dvuk9y8eo6q85n4?svg=true)](https://ci.appveyor.com/project/loxsmoke/docxml-nbx6b) [![AppVeyor tests](https://img.shields.io/appveyor/tests/loxsmoke/docxml-nbx6b.svg)](https://ci.appveyor.com/project/loxsmoke/docxml-nbx6b) [![codecov](https://codecov.io/gh/loxsmoke/docxml/branch/master/graph/badge.svg)](https://codecov.io/gh/loxsmoke/docxml) [![NuGet version](https://badge.fury.io/nu/LoxSmoke.DocXml.svg)](https://badge.fury.io/nu/LoxSmoke.DocXml) [![NuGet](https://img.shields.io/nuget/dt/LoxSmoke.DocXml.svg)](https://www.nuget.org/packages/LoxSmoke.DocXml) 

This component is a set of helper classes and methods for compiler-generated XML documentation retrieval.
XML documentation as described here https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/xmldoc/xml-documentation-comments
is generated by compiler from source code comments and saved in a XML file where each
documented item is identified by its unique name. These names sometimes are quite complicated especially
when templates and generic types are involved. DocXml makes retrieval of XML documentation easy. For example
here is the class with generic method and the piece of code that retrieves method documentation.

```csharp
// Source code for XML documentation
namespace LoxSmoke.DocXmlUnitTests
{
    class MyClass2
    {
        /// <summary>
        /// TemplateMethod description
        /// </summary>
        /// <returns>Return value description</returns>
        public List<T> TemplateMethod<T>()
        { return null; }
    }
}
```

Fragment from **DocXmlUnitTests.xml** file generated by compiler.
```xml
<member name="M:LoxSmoke.DocXmlUnitTests.MyClass2.TemplateMethod``1">
    <summary>
    TemplateMethod description
    </summary>
    <returns>Return value description</returns>
</member>
```

Some code that retrieves XML documentation from the file and type reflection information.
```csharp
// Get MethodInfo of MyClass2.TemplateMethod
var minfo = typeof(MyClass2).GetMethod("TemplateMethod");
// Load XML documentation file
DocXmlReader reader = new DocXmlReader("DocXmlUnitTests.xml");
// Get method documentation from XML file
var comments = reader.GetMethodComments(minfo);
// and print it
Console.WriteLine(comments.Summary);
Console.WriteLine(comments.Returns);
```

A bit more interesting example uses **JsonObjectContract** from popular **Newtonsoft.Json** nuget package. Here code retrieves summary comments for each property.
```csharp
// Just for the sake of this example create JsonObjectContract
var jsonContract = new DefaultContractResolver().ResolveContract(typeof(MyClass2)) as JsonObjectContract;
// Load XML documentation file
DocXmlReader reader = new DocXmlReader("DocXmlUnitTests.xml");
foreach (var jsonContractProperty in jsonContract.Properties)
{
    var minfo = jsonContractProperty.DeclaringType.GetMember(jsonContractProperty.UnderlyingName)[0];
    // Get documentation from XML file
    var comments = reader.GetMemberComments(minfo);
    // and print it
    Console.WriteLine(comments.Summary);
}
```


## How to use

DocXml is available as LoxSmoke.DocXml nuget package. It is built as .net standard 2.0 class library.

## Classes and methods

The main class is **DocXmlReader**. It reads XML file and returns documentation/comments objects.

Comment classes represent one or more comments associated with each item in the source code. Simple summary 
comments are returned as strings and more complex comments are returned as comments objects. Here is the list of 
comments classes:
* **CommonComments** is the base class of all comments classes.
* **TypeComments** is documentation of the class or struct. 
* **MethodComments** is documentation of the method, constructor, operator or property with parameter descriptions.
* **EnumComments** is documentation of the Enum type. Comments of enum values are included here as well.
* **EnumValueComment** is documentation of one Enum value: name, integer value and relevant documentation comments.

Static **XmlDocId** class is a set of static methods that generates IDs used for retrieval of documentation from 
XML file. This class is used by **DocXmlReader**.   

## Documentation comments examples

Simple summary comment can be located next to class definition, property, method, field, etc.
```csharp
/// <summary>
/// This is class comment
/// </summary>
```
**DocXmlReader** method **GetMemberComment** returns this comment as string.
This comment is also available as **Summary** property in all comments classes.

Delegate type may have comments for each parameter. 
```csharp
/// <summary>
/// Delegate type description 
/// </summary>
/// <param name="parameter">Parameter description</param>
public delegate void DelegateType(int parameter);
```
**DocXmlReader** method **GetTypeComment** returns this comment as **TypeComments** object with **Summary** property and the list of parameters as **Parameters** property. **Parameters** property is the list of tuples where **Item1** is the name of the parameter and **item2** is the parameter description comment.

Enum comment with documented values.
```csharp
/// <summary>
/// Enum type description
/// </summary>
public enum TestEnum
{
    /// <summary>
    /// Enum value one
    /// </summary>
    Value1,
     
    /// <summary>
    /// Enum value two
    /// </summary>
    Value2
};
```
**DocXmlReader** method **GetEnumComments** returns this comment as **EnumComments** object with **Summary** property and the list of **ValueComments**. Each element in **ValueComments** has the name of the value e.g. **Value1**, actual integer value e.g. **10** and relevant comments such as summary comment of the value. If none of values has any summary comment then **ValuesComments** list is empty by default.

Method comment with parameters and return value.
```csharp
/// <summary>
/// Member function description.
/// </summary>
/// <param name="one">Parameter one</param>
/// <param name="two">Parameter two</param>
/// <returns>Return value description</returns>
public int MemberFunction2(string one, ref int two)
```
**DocXmlReader** method **GetMethodComments** returns this comment as **MethodComments** object with **Summary** and **Returns** properties and the list of **Parameters** tuples. Each element in **Parameters** is the tuple where **Item1** is the name of the parameter e.g. **one** and **Item2** is the comment of the parameter value.


Web controller method comment with response codes.
```csharp
/// <summary>
/// Member function description
/// </summary>
/// <returns>Return value description</returns>
/// <response code="200">OK</response>
```
**DocXmlReader** method **GetMethodComments** returns this comment as **MethodComments** object with **Summary** and **Returns** properties and the list of **Responses** tuples. Each element in **Responses** is the tuple where **Item1** is the code of the response e.g. **200** and **Item2** is the comment of the value e.g. **OK**.






