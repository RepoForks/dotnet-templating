# .NET Templating

## Getting Started

1. Add a `.template.config\template.json`. Place this in the root directory for each template (the same folder that contains a .sln or .csproj). This file is what we will be using to take our apps and make them templates by providing some metadata.
2. Install the template. Execute `dotnet new --install PATH` (example: `dotnet new --install ~/Desktop/iOS/SingleViewApp
`) where path is the path to the folder containing the .template.config. This will discover any template files in this path and populate the list of available templates.
3. Create the template using `dotnet new [shortName] -n NameOfTemplate -o FolderNameToCreateTemplate`. This replaces the namespace elements to use `NameOfTemplate` instead of whatever the template app itself is using.


$ dotnet new sayedweb -n Contoso.Web -o Contoso.Web

Uninstalling Templates: There currently is no `--uninstall` option. Instead, run `dotnet new --debug:reinit` to reset the templates to default settings.

# Symbols
The ability to alter content within a template depending on a particular user configuration is an important feature for any templating engine. Not only do we need to be able to replace namespaces. .NET templating does this with a feature called **symbols**. Define a symbol in the `symbols` collection, describe some metadata, and have it replaced in the template, without modifying the template project.

Example:

```
  "symbols" : { // [Optional] Defines variables and their values. When a defined symbol name is encountered anywhere in the template, it is replaced by the value defined in this configuration. This is a collection of key-value paris, where the keys are symbol names, and the value contains key-value-pair configuration information on assigning the symbol encountered a value.
      "appIdentifier" : {
          "type" : "parameter", // [Required] Valid values of parameter, computed, generated.
          "description": "Overrides the Info.plist's CFBundleIdentifier", // [Optional] Human-readable description of the symbol.
          "replaces": "com.xamarin.SingleViewApp", // [Optional]
          "datatype": "string", // [Optional] Indicates data type for parameter.
          "defaultValue": "com.companyname" // [Optional] Indicates default value if none is provided.
      }
  }
```

Run `dotnet new xamios -o HelloThere -n HelloThere --appIdentifier com.companyname.hellothere`, and the Info.plist will be updated with the user's defined Info.plist value automatically.

## Adding Optional Content (with Symbols!)

Often, we may want to selectively add or remove parts of the template. In this case, let's assume we want to delete ViewController.cs/ViewController.designer.cs and remove some methods in the `AppDelegate` class.

Add a symbol to your definition named `EnableAwesomeStuff`.

```
      "enableAwesomeStuff" : {
        "type": "parameter",
        "dataType":"bool", // Indicate this parameter supports true/false values. Used to determine if content is added to project.
        "defaultValue": "false"
      }
```

To exclude a file from being processed during creation, you will need to add a new element into the template.json file:

```
  "sources": [
    {
        "modifiers": [
            {
                "condition": "(!enableAwesomeStuff)",
                "exclude": [ "ViewController.cs", "ViewController.designer.cs" ]
            }
        ]
    }
]
```

This modifies our sources element which will exclude the ViewController and ViewController.designer files if enableAwesomeStuff is false. 

We can also use this value within our code to determine if stuff is excluded, such as:

```
#if (enableAwesomeStuff)
public void MyAwesomeMethod()
{
    Console.WriteLine ("So awesome!");
}
#endif
```

## References

* [template.json](https://github.com/dotnet/templating/wiki/%22Runnable-Project%22-Templates)
* [samples](https://github.com/dotnet/dotnet-template-samples)
