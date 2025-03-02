# Release Process

The Kubebuilder Project is released on an as-needed basis. The process is as follows:

1. An issue is proposing a new release with a changelog since the last release. You will need to use the [kubebuilder-release-tools][kubebuilder-release-tools] to generate the notes. See [here][release-notes-generation]
1. All [OWNERS](OWNERS) must LGTM this release
1. An OWNER runs `git tag -s $VERSION` and pushes the tag with `git push $VERSION`. Note that after the OWNER push the tag the CI will automatically add the release notes and the assets.
1. A PR needs to be created to merge `master` branch into `book-v3` to pick up the new docs. 
1. The release issue is closed
1. An announcement email is sent to `kubebuilder@googlegroups.com` with the subject `[ANNOUNCE] kubebuilder $VERSION is released`

**Notes:** This process does not apply to EAP or alpha (pre-)releases which may be cut at any time for development
and testing.

For further information about versioning and update the Kubebuilder binaries check the [versioning][release-process] doc.

## HEAD releases

The binaries releases for HEAD are available here:

- [kubebuilder-release-master-head-darwin-amd64.tar.gz](https://storage.googleapis.com/kubebuilder-release/kubebuilder-release-master-head-darwin-amd64.tar.gz)
- [kubebuilder-release-master-head-linux-amd64.tar.gz](https://storage.googleapis.com/kubebuilder-release/kubebuilder-release-master-head-linux-amd64.tar.gz)

## How the releases are configured

The releases occur in an account in the Google Cloud (See [here](https://console.cloud.google.com/cloud-build/builds?project=kubebuilder)) using Cloud Build.

### To build the Kubebuilder CLI binaries:

A trigger `build-kb-release` is configured to call [build/cloudbuild.yaml](build/cloudbuild.yaml).
This trigger will be executed when any new tag be published. 
The tags must be built from the release branch (Currently, `release-3`).

Also, we have a trigger to generate snapshots builds from the master branch.
This trigger will call [build/cloudbuild_snapshot.yaml](build/cloudbuild_snapshot.yaml)
when any change needs to be performed on master.

### To build the Kubebuilder-tools: (Artifacts required to use ENV TEST)

Kubebuilder projects requires artifacts which are used to do test with ENV TEST (when we call `make test` target)
These artifacts can be checked in the service page: https://storage.googleapis.com/kubebuilder-tools

The build is made from the branch [tools-releases](https://github.com/kubernetes-sigs/kubebuilder/tree/tools-releases) and the trigger will call the `build/cloudbuild_tools.yaml` passing 
as argument the architecture and the SO that should be used, e.g:

<img width="553" alt="Screenshot 2022-04-30 at 10 15 41" src="https://user-images.githubusercontent.com/7708031/166099666-ae9cd2df-73fe-47f6-a987-464f63df9a19.png">

For further information see the [README](https://github.com/kubernetes-sigs/kubebuilder/blob/tools-releases/README.md).

### To build the `kube-rbac-proxy` images:

These images are built from the project [brancz/kube-rbac-proxy](https://github.com/brancz/kube-rbac-proxy).
The projects built with Kubebuilder creates a side container with `kube-rbac-proxy` to protect the Manager.

These images are can be checked in the consolse, see [here](https://console.cloud.google.com/gcr/images/kubebuilder/GLOBAL/kube-rbac-proxy).

The project `kube-rbac-proxy` is in the process to be donated to the k8s org. However, it is going on for a long time and then,
we have no ETA for that to occur. When that occurs we can automate this process. But until there we need to generate these images
by bumping the versions/tags released by `kube-rbac-proxy` on the branch 
[kube-rbac-proxy-releases](https://github.com/kubernetes-sigs/kubebuilder/tree/kube-rbac-proxy-releases)
then the `build/cloudbuild_kube-rbac-proxy.yaml` will generate the images.

To check an example, see the pull request [#2578](https://github.com/kubernetes-sigs/kubebuilder/pull/2578).

**Note**: we cannot use the images produced by the project `kube-rbac-proxy` because we need to ensure
to Kubebuilder users that these images will be available.

### To build the `gcr.io/kubebuilder/pr-verifier` images:

These images are used to verify the PR title and description. They are built from [kubernetes-sigs/kubebuilder-release-tools](https://github.com/kubernetes-sigs/kubebuilder-release-tools/).
In Kubebuilder, we have been using this project via the GitHub action [.github/workflows/verify.yml](.github/workflows/verify.yml)
and not the image, see:

```yaml
  verify:
    name: Verify PR contents
    runs-on: ubuntu-latest
    steps:
    - name: Verifier action
      id: verifier
      uses: kubernetes-sigs/kubebuilder-release-tools@v0.1.1
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
```

However, the image should still be built and maintained since other projects under the org might be using them. 

[kubebuilder-release-tools]: https://github.com/kubernetes-sigs/kubebuilder-release-tools
[release-notes-generation]: https://github.com/kubernetes-sigs/kubebuilder-release-tools/blob/master/README.md#release-notes-generation
[release-process]: https://github.com/kubernetes-sigs/kubebuilder/blob/master/VERSIONING.md#releasing 