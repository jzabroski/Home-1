# PackProjectReferences

- Author Name [@jzabroski](https://github.com/jzabroski)
- Start Date (YYYY-MM-DD)
- GitHub Issue https://github.com/NuGet/Home/issues/3891
- GitHub PR (GitHub PR link)

## Summary

There is a TODO in the existing NuGet code 

## Motivation 

<!-- Why are we doing this? What pain points does this solve? What is the expected outcome? -->

## Explanation

### Functional explanation

<!-- Explain the proposal as if it were already implemented and you're teaching it to another person. -->
<!-- Introduce new concepts, functional designs with real life examples, and low-fidelity mockups or  pseudocode to show how this proposal would look. -->

### Technical explanation

#### Implementation Details

#### Interaction Models
One interaction NuGet has is with MSBuild.  MSBuild already has facilities for copying build output like Content items into the output directory.
See: https://docs.microsoft.com/en-us/nuget/reference/msbuild-targets#including-content-in-a-package

Another interaction is with the [`dotnet pack --IncludeReferencedProjects`](https://docs.microsoft.com/en-us/nuget/reference/cli-reference/cli-ref-pack) CLI.

Another is with PackAsTool.

#### Clarification of Corner Cases

1. Create a nuget package that depends on private libraries, i.e. libraries that are needed at runtime, but should not be exposed as a nuget package.

[Is there a good way for the NuGet package to import the dependencies that the Project had?](https://github.com/NuGet/Home/issues/3891#issuecomment-382867751)
Say Project A depends on Project B, but Project B depends on NuGet Package 1 and Nuget Package 2.
```
Solution 'NuGet.CornerCase1' (2 of 2 projects)
|
|
 \ Class Libraries (solution folder)
 |-- Project B
 |
  \ Dependencies
   |
    \ Packages
    |
     \ NuGet Package 1
     \ NuGet Package 2
|
 \ Executables (solution folder)
 |-- Project A
 |
  \ Dependencies
  |
   \ Projects
    |
     \ Project B
```
How can I make Project A list the dependences of Project B as its own dependencies?

Note: This use case is used by tools like OctoPack, which require a NuGet package to be self-extracting archive without network calls to NuGet.org or some other package provider.  In this sense, once an OctoPack is created, its internal contents is a consistent snapshot of all transitive dependencies.

## Drawbacks

1. Why should we not do this?

This would break existing csproj definitions.

## Rationale and alternatives

Rob Relyea's spec: https://github.com/NuGet/Home/wiki/Adding-nuget-pack-as-a-msbuild-target
Daniel Kuzzulino's spec: https://github.com/NuGet/NuGet.Build.Packaging aka NuGetizer
Rohit Agarwal's workaround: https://github.com/NuGet/Home/issues/3891#issuecomment-377319939
  * This solution does _not_ also copy any `PackageReference`s from `ProjectReference`s into the resulting `.nuspec` file?
Joseph Musser's workaround: "If dotnet pack isn't working for you, why don't you go back to using nuget pack?"
  * This solution has a contra argument here: https://github.com/NuGet/Home/issues/3891#issuecomment-512713719
  * nuget.exe doesn't understand multi targeting, and the issue about it on the github tracker says something along the lines of "nuget support is being built into dotnet, so we're not going to continue supporting this, as there's no point in duplicating effort".

1. Why is this the best design compared to other designs?
2. What other designs have been considered and why weren't they chosen?
3. What is the impact of not doing this?

## Prior Art

<!-- What prior art, both good and bad are related to this proposal? -->
<!-- Do other features exist in other ecosystems and what experience have their community had? -->
<!-- What lessons from other communities can we learn from? -->
<!-- Are there any resources that are relevent to this proposal? -->

## Unresolved Questions

<!-- What parts of the proposal do you expect to resolve before this gets accepted? -->
<!-- What parts of the proposal need to be resolved before the proposal is stabilized? -->
<!-- What related issues would you consider out of scope for this proposal but can be addressed in the future? -->

## Future Possibilities

<!-- What future possibilities can you think of that this proposal would help with? -->
