[meta]: #meta
- Name: Move metadata store for platform out of labels
- Start Date: 2022-07-27
- Author(s): [@schneems](https://github.com/schneems),
- Status: Draft
- RFC Pull Request: (leave blank)
- CNB Pull Request: (leave blank)
- CNB Issue: (leave blank)
- Supersedes: N/A

# Summary
[summary]: #summary

This RFC proposes that we deprecate and remove the platform label `io.buildpacks.lifecycle.metadata.buildpacks.layers.*` and instead store that data in a file `<layers>/config/<buildpack ID>/<layer>.toml`.

# Definitions
[definitions]: #definitions

- OCI Label: OCI image labels store key-value data on the image. The data is not exposed to the container running the image but can be used for things like annotations. [How and when to use Docker labels / OCI container annotations](https://snyk.io/blog/how-and-when-to-use-docker-labels-oci-container-annotations/)
- [platform specification](https://github.com/buildpacks/spec/blob/edac0f7214646018d6fb78267b1fad6c3d23b5b0/platform.md). A platform orchestrates a lifecycle to make buildpack functionality available to end-users such as application developers. The specification defines the behavior of a platform
- [lifecycle](https://github.com/buildpacks/lifecycle): Reference implementation of the CNB spec including the platform specification. Software that orchestrates a CNB build.

# Motivation
[motivation]: #motivation

- Why should we do this?
    - Labels that become too large may crash K8 nodes [source](https://cloud-native.slack.com/archives/C0333LG1E68/p1658441047456149?thread_ts=1658434382.039529&cid=C0333LG1E68). Moving metadata to a file will fix this.
    - TOML metadata information serialized to the label's JSON format is a lossy. This process affects any buildpacks relying on integers and possibly other data structures not represented in JSON. [Lifecycle issue](https://github.com/buildpacks/lifecycle/issues/884). Moving metadata to a file will fix this.
    - Prevent developers from treating the layer metadata label like a part of the public API for a buildpack. [source](https://cloud-native.slack.com/archives/C0333LG1E68/p1658441047456149?thread_ts=1658434382.039529&cid=C0333LG1E68). Moving metadata to a file will fix this.
- What use cases does it support?
    - Existing behavior will be preserved though the change is breaking.
- What is the expected outcome?
    - The problems above will be remediated.

# What it is
[what-it-is]: #what-it-is

Data present in `io.buildpacks.lifecycle.metadata` [spec](https://github.com/buildpacks/spec/blob/edac0f7214646018d6fb78267b1fad6c3d23b5b0/platform.md#iobuildpackslifecyclemetadata-json) like:

```
$ docker image inspect lol-test | jq -r '.[].Config.Labels["io.buildpacks.lifecycle.metadata"]'
{
  "app": [
    {
      "sha": "sha256:3b2b4f4db73fd7395d7ec1d8f7a07b34b66314d43aacbbb6c2a5b5277573c516"
    }
  ],
  "sbom": {
    "sha": "sha256:36f9de0748c420980a265697f38f76b0f9ae9b7862d3467b76b638706e637425"
  },
  "buildpacks": [
    {
      "key": "heroku/ruby",
      "version": "2.0.0",
      "layers": {
        "counter": {
          "sha": "sha256:28cee3e101143f9121d09b19a4796eb1b61e0aa479f644821d2c13ee5106a7dc",
          "data": {
            "build-count": 1
          },
          "build": true,
          "launch": true,
          "cache": true
        },
      }
    }
  ],
  "config": {
    "sha": "sha256:48c2eae74acdaba0f0abc3b1bde58f49d7b2cdcc0f14a5c8f310c1e638e1b8b5"
  },
  "launcher": {
    "sha": "sha256:9497805c7bd5df192c4ccf7cf9c4496963c1c008141fe134765e23a3800fa4cf"
  },
  "process-types": {
    "sha": "sha256:83d85471d9f8a3834b4e27cf701e3f0aef220cc816d9c173c7d32cd73909a590"
  },
  "runImage": {
    "topLayer": "sha256:e3aab3120cfa85f4ba64f5b2fb7377547f9f1caa82e958830059a0f3d908f345",
    "reference": "34fe0e33bab2f74ef659d8b7bfcc29e6ee6ce491a634677ab34b8e2f3bf55372"
  },
  "stack": {
    "runImage": {
      "image": "heroku/heroku:20-cnb"
    }
  }
}
```

The `"buildpacks"` key contains one buildpack with one layer:

```
  "buildpacks": [
    {
      "key": "heroku/ruby",
      "version": "2.0.0",
      "layers": {
        "counter": {
          "sha": "sha256:28cee3e101143f9121d09b19a4796eb1b61e0aa479f644821d2c13ee5106a7dc",
          "data": {
            "build-count": 5
          },
          "build": true,
          "launch": true,
          "cache": true
        },
      }
    }
  ],
```

With this proposal the `layers` data would be removed from this presentation and stored instead at:

```
<layers>/config/<buildpack ID>/<layer>.toml
```

For example:

```
/layers/config/heroku_ruby/counter.toml
```

The contents of this file would be:

```
build = true
cache = true
launch = true
sha = "sha256:28cee3e101143f9121d09b19a4796eb1b61e0aa479f644821d2c13ee5106a7dc"
[data]
build-count = 5
```

> Note: I'm not sure about moving the sha here then we're mixing buildpack generated content with platform generated content.

# How it Works
[how-it-works]: #how-it-works

TODO

This is the technical portion of the RFC, where you explain the design in sufficient detail.

The section should return to the examples given in the previous section, and explain more fully how the detailed proposal makes those examples work.

# Migration
[migration]: #migration

TODO

This section should document breaks to public API and breaks in compatibility due to this RFC's proposed changes. In addition, it should document the proposed steps that one would need to take to work through these changes. Care should be give to include all applicable personas, such as platform developers, buildpack developers, buildpack users and consumers of buildpack images.

# Drawbacks
[drawbacks]: #drawbacks

TODO Why should we *not* do this?

# Alternatives
[alternatives]: #alternatives

TODO

- What other designs have been considered?
- Why is this proposal the best?
- What is the impact of not doing this?

# Prior Art
[prior-art]: #prior-art

TODO

Discuss prior art, both the good and bad.

# Unresolved Questions
[unresolved-questions]: #unresolved-questions

TODO

- What parts of the design do you expect to be resolved before this gets merged?
- What parts of the design do you expect to be resolved through implementation of the feature?
- What related issues do you consider out of scope for this RFC that could be addressed in the future independently of the solution that comes out of this RFC?

# Spec. Changes (OPTIONAL)

TODO

[spec-changes]: #spec-changes
Does this RFC entail any proposed changes to the core specifications or extensions? If so, please document changes here.
Examples of a spec. change might be new lifecycle flags, new `buildpack.toml` fields, new fields in the buildpackage label, etc.
This section is not intended to be binding, but as discussion of an RFC unfolds, if spec changes are necessary, they should be documented here.
