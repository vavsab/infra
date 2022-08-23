# Pulumiverse Infrastructure setup

The Pulumiverse organization has some infrastructure to support its operation. 
This repository manages as much as possible of this setup using Pulumi IaC.

## Github

**Status:** Partially Automated

The Github provider doesn't contain the functionality to configure all of the organization level settings. The following settings
are controlled manually as a result:

* [Workflow permissions](https://github.com/organizations/pulumiverse/settings/actions): default set to `Read repository contents permission` instead of `Read and write permission`

The following resources within the organization on Github is managed by Pulumi code:

* Teams
* Membership to organization and teams
* Repositories and team access

Each repository which publishes artifacts to package registries will get
separate publishing tokens to the respective package registries.

## Package Registries

Please read the documentation on [Authoring & Publishing](https://www.pulumi.com/docs/guides/pulumi-packages/how-to-author/) before continuing with the specifics below.

**Notes:** 

- to use custom package names for Terraform bridge providers, please ensure you minimally use [`github.com/pulumi/pulumi-terraform-bridge v3.21.0`](https://github.com/pulumi/pulumi-terraform-bridge/releases/tag/v3.21.0).
- when publishing plugin binaries for providers as Github releases, make sure your users are running 
  [Pulumi 3.35.3](https://www.pulumi.com/docs/guides/pulumi-packages/how-to-author/#support-for-github-releases)
  or up. Make sure to set the plugin download URL consistently to `github://api.github.com/pulumiverse`.

  In `schema.json`, add a toplevel entry like this:

  ```json
  "pluginDownloadURL": "github://api.github.com/pulumiverse",
  ```

  When bridging a Terraform provider, add the following to `providers/resources.go`:

  ```go
  func Provider() tfbridge.ProviderInfo {
    ...

    // Create a Pulumi provider mapping
    prov := tfbridge.ProviderInfo{
        ...
        // PluginDownloadURL is an optional URL used to download the Provider
        // for use in Pulumi programs
        PluginDownloadURL: "github://api.github.com/pulumiverse",
        ...
    }
    ...
  }
  ```

### NPM Registry

#### Administration

**Status:** Manual

All members of the Pulumiverse Governance Board receive admin/owner rights on
the `pulumiverse` organization on the [NPM Registry](https://www.npmjs.com).
Having this organization ensures we have the `@pulumiverse/...` package
namespace available and claimed.

#### Publish packages

We encourage all publishable artifacts to use this namespace. For Pulumi
packages, set the package name in the `schema.json` file:

```json
{
    ...
    "language": {
        "nodejs": {
            "packageName": "@pulumiverse/<package>",
            ...
        }
    }
}
```

For a Terraform bridged provider, add this to `resources.go`:

```go
func Provider() tfbridge.ProviderInfo {
    ...

	// Create a Pulumi provider mapping
	prov := tfbridge.ProviderInfo{
        ...
		JavaScript: &tfbridge.JavaScriptInfo{
			PackageName: "@pulumiverse/<package>",
            ...
		},
        ...
	}
    ...
}
```

## Python Package Index

#### Administration

**Status:** Manual

Pypi doesn't have the notion of an organization or team account. A separate
user account `pulumiverse` is the owner of all Python packages published
under the Pulumiverse wings.

The credentials to access this user account are shared securely with the
members of the Pulumi Governance Board.

#### Publish packages

We encourage all publishable artifacts to use the `pulumiverse_` prefix.
For Pulumi packages, set the package name in the `schema.json` file:

```json
{
    ...
    "language": {
        "python": {
            "packageName": "pulumiverse_<package>",
            ...
        }
    }
}
```

For a Terraform bridged provider, add this to `resources.go`:

```go
func Provider() tfbridge.ProviderInfo {
    ...

	// Create a Pulumi provider mapping
	prov := tfbridge.ProviderInfo{
        ...
		Python: &tfbridge.PythonInfo{
			PackageName: "pulumiverse_<package>",
            ...
		},
        ...
	}
    ...
}
```

## Go

#### Administration

Go doesn't have or need a central repository. The generated Go SDK for a
Pulumi package must be committed into the Git repository and properly
tagged.

#### Publish packages

To publish Go packages, the base import path must be set correctly in the
generated `go.mod` file. 
For Pulumi packages, set the package name in the `schema.json` file:

```json
{
    ...
    "language": {
        "go": {
            "importBasePath": "github.com/pulumiverse/pulumi-<package>/sdk/go/<package>"
            ...
        }
    }
}
```

For a Terraform bridged provider, add this to `resources.go`:

```go
func Provider() tfbridge.ProviderInfo {
    ...

	// Create a Pulumi provider mapping
	prov := tfbridge.ProviderInfo{
        ...
		Golang: &tfbridge.GolangInfo{
			ImportBasePath: filepath.Join(
				fmt.Sprintf("github.com/pulumiverse/pulumi-%[1]s/sdk/", mainPkg),
				tfbridge.GetModuleMajorVersion(version.Version),
				"go",
				mainPkg,
			),
            ...
		},
        ...
	}
    ...
}
```

### Nuget

#### Administration

Nuget doesn't have the notion of an organization or team account. A separate
user account `pulumiverse` is the owner of all .NET packages published
under the Pulumiverse wings.

The credentials to access this user account are shared securely with the
members of the Pulumi Governance Board.

#### Publish packages

To publish Nuget packages, the base namespace must be set correctly configured.
For Pulumi packages, set the namespace in the `schema.json` file:

```json
{
    ...
    "language": {
        "csharp": {
            "rootNamespace": "Pulumiverse",
            ...
        }
    }
}
```

For a Terraform bridged provider, add this to `resources.go`:

```go
func Provider() tfbridge.ProviderInfo {
    ...

	// Create a Pulumi provider mapping
	prov := tfbridge.ProviderInfo{
        ...
		CSharp: &tfbridge.CSharpInfo{
			RootNamespace: "Pulumiverse",
            ...
		},
        ...
	}
    ...
}
```

## Cloudflare

**Status:** Manual

* DNS hosting of `pulumiverse.com`

## Pulumi Service

**Status:** Manual

Membership to https://app.pulumi.com/pulumiverse is managed manually, given there
currently is no Pulumi provider to their own platform. This is requested as 
[#18](https://github.com/pulumi/service-requests/issues/18) and `in development`
according the [public roadmap](https://github.com/orgs/pulumi/projects/44).

We will start using the provider once the first release becomes available.
