# IncludeReferencedProjects

- Author Name @jzabroski
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

#### Clarification of Corner Cases

## Drawbacks

1. Why should we not do this?

This would break existing csproj definitions.

## Rationale and alternatives

Rob Relyea's spec: https://github.com/NuGet/Home/wiki/Adding-nuget-pack-as-a-msbuild-target
Daniel Kuzzulino's spec: 
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
