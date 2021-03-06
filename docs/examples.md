---
Order: 30
Title: Examples
Description: Examples for using the Cake.Issues.DupFinder addin.
---
The following example will call [JetBrains dupFinder] and output the number of warnings.

To call [JetBrains dupFinder] from a Cake script you need to add the `JetBrains.ReSharper.CommandLineTools`:

```csharp
#tool "nuget:?package=JetBrains.ReSharper.CommandLineTools"
```

To read issues from dupFinder log files you need to import the core addin and the dupFinder support:

```csharp
#addin "Cake.Issues"
#addin "Cake.Issues.DupFinder"
```

:::{.alert .alert-warning}
Please note that you always should pin addins and tools to a specific version to make sure your builds are deterministic and
won't break due to updates to one of the packages.

See [pinning addin versions](https://cakebuild.net/docs/tutorials/pinning-cake-version#pinning-addin-version) for details.
:::

We need some global variables:

```csharp
var logPath = @"c:\build\dupfinder.xml";
var repoRootPath = @"c:\repo";
```

The following task will run [JetBrains dupFinder] and write a log file:

```csharp
Task("Analyze-Project").Does(() =>
{
    // Run DupFinder.
    var settings = new DupFinderSettings() {
        OutputFile = logPath
    };

    DupFinder(repoRootPath.CombineWithFilePath("MySolution.sln"), settings);
});
```

Finally you can define a task where you call the core addin with the desired issue provider.

```csharp
Task("Read-Issues")
    .IsDependentOn("Analyze-Project")
    .Does(() =>
    {
        // Read Issues.
        var issues =
            ReadIssues(
                DupFinderIssuesFromFilePath(logPath),
                repoRootPath);

        Information("{0} issues are found.", issues.Count());
    });
```

[JetBrains dupFinder]: https://www.jetbrains.com/help/resharper/dupFinder.html
