
# Spec Name

* Status: In Review
* Author(s): [Rob Relyea](https://github.com/rrelyea)
* Issue: [10151](https://github.com/NuGet/Home/issues/10151) INuGetPackageFileService - Fetch Images and embedded licenses for codespaces and normal execution

## Problem Background

With Codespaces support added to NuGet client, our code needs to be able to handle all packages and communication with feeds happening on a server node. This is architected as async method calls to services that are remoted. In most cases, we also have chosen to have 1 code path, for standalone execution and Codespaces-connected execution. These services are still used in the standalone execution, however, the service runs in the same process, as opposed to being remoted across the network, as in the Codespaces-connected execution.

Package Icons can either come from:
- a package feeds search service
- (old style) or a URI to an arbitrary place on the web (specified in the NuSpec via now deprecated [iconUrl](https://docs.microsoft.com/en-us/nuget/reference/nuspec#iconurl) element.)
- embedded in a package (specified in the NuSpec via [icon](https://docs.microsoft.com/en-us/nuget/reference/nuspec#icon) element.)

Licenses can also come from a few places:
- a package feeds search service
- a [licenseUrl](https://docs.microsoft.com/en-us/nuget/reference/nuspec#licenseurl) (deprecated)
- a [license expression](https://docs.microsoft.com/en-us/nuget/reference/nuspec#license)
- an [embedded license file](https://docs.microsoft.com/en-us/nuget/reference/nuspec#license) in the package

## Who are the customers

All PM UI customers.

## Goals

Support Codespaces-connected and standalone fetching of icons and licenses.

## Non-Goals

## Solution Overview

### Fetching Files
Defined a new interface for this service:

```C#
namespace NuGet.VisualStudio.Internal.Contracts
{
    public interface INuGetPackageFileService : IDisposable
    {
        ValueTask<Stream?> GetPackageIconAsync(PackageIdentity packageIdentity, CancellationToken cancellationToken);
        ValueTask<Stream?> GetEmbeddedLicenseAsync(PackageIdentity packageIdentity, CancellationToken cancellationToken);
    }
}
```

On the service side, in [SearchObject.CacheBackgroundData()](https://github.com/NuGet/NuGet.Client/blob/b5b44526dea0379ebd6c8e51e8a041c06d5845ca/src/NuGet.Clients/NuGet.PackageManagement.VisualStudio/Services/SearchObject.cs#L218-L236), beyond the packageSearchMetadata caching that was already done, we call a NuGetPackageFileService instance to AddIconToCache() and to AddLicenseToCache(). These calls cause a PackageIdentity key and a URI value to be stored in a MemoryCache.

On the client side, when trying to fetch the icon, we call PackageFileService.GetPackageIconAsync, passing in the package identity.

On the client side, when trying to fetch an embedded license, we call PackageFileService.GetEmbeddedLicenseAsync.

On the service side, we then retrieve the appropriate file from the appropriate location based on the Uri stored in the MemoryCache and return the stream to the caller.

#### IdentityToUri Memory Cache notes

The settings of the IdentityToUri memory cache that the NuGetPackageFileService class hosts and maintains are identical to the metadata MemoryCache that the search service uses as part of it codespaces rebuilding:
- 4 MB memory limit
- 0 physical memory limit percentage
- 2min polling interval

Given those settings, we don't expect to frequently not find an entry that should be available in the memory cache...we have a Telemetry call to record how often this does happen.

Details of how the memory cache gets populated is overed in the "paragraph above that starts with "On the service side".

#### Issue: should GetEmbeddedLicenseAsync be more generic. (question from zkat)
    zkat - I guess this is fine for now, but I'd really like it if now-or-eventually, this is abstracted away into a set of supported "Well Known Files"--for example, if we want to add Changelog or docs support in the future.

    rrelyea - thought a bit about it... i'd imagine we could pass an enum into a method... packageIdentity, wellKnownEmbeddedFile

    where wellKnownEmbeddedFiles might be readme, license and what else?
    maybe icon...but that isn't always embedded, can also be a url outside of the package.
    i lean towards not doing this in a generic way yet...as we don't know the full set of needs yet...and since the nuspec is on the server, we cannot just take a relative path, etc...

    would love to discuss, hear other thoughts.

### Other Changes

Several classes had a PackageReader property that was of type Func<PackageReader>.
Func<PackageReader> wasn't designed to be remoted nicely. In several places, I replaced the PackageReader property with a PackagePath property, to be used when appropriate for loading files from a package.

## Test Strategy

Added unit tests for NuGetPackageFileService and updated many existing tests for this new implementation of icons/licenses.

## Future Work

None critical.

## Open Questions

Any open questions not specifically called out in the design.

## Considerations

At one point, the design of this interface passed in URIs...we moved to PackageIdentity and storing in the MemoryCache to tighten security of the feature.

We also, chose to only key off of PackageIdentity, instead of PackageIdentity and a package source...staying consistent with our treatment of unique package idenities as the same, regardless of where they come from.

Performance - we've discussed whether the quantity of calls we make for search, icons, licenses is a problem in standalone or Codespaces-connected scenarios. As these calls are async, and in similar quantity to earlier codepaths (for search tab scenarios at least), my impression is that this won't be a bottleneck. I've not done a profiling exercise to confirm.

### References