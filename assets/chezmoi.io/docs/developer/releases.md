# Releases

Releases are managed with [`goreleaser`](https://goreleaser.com/).

## Testing

To build a test release, without publishing, (Ubuntu Linux only) first ensure
that the `musl-tools` and `snapcraft` packages are installed:

```console
$ sudo apt-get install musl-tools snapcraft
```

Then run:

```console
$ make test-release
```

## Publishing

Publish a new release by creating and pushing a tag, for example:

```console
$ git tag v1.2.3
$ git push --tags
```

This triggers a [GitHub Action](https://github.com/twpayne/chezmoi/actions)
that builds and publishes archives, packages, and snaps, creates a new [GitHub
Release](https://github.com/twpayne/chezmoi/releases), and deploys the
[website](https://chezmoi.io).

!!! note

    Publishing [Snaps](https://snapcraft.io/) requires a
    `SNAPCRAFT_STORE_CREDENTIALS` [repository
    secret](https://github.com/twpayne/chezmoi/settings/secrets/actions).

    Snapcraft store credentials periodically expire. Create new snapcraft store
    credentials by running:

    ```console
    $ snapcraft export-login --snaps=chezmoi --channels=stable,candidate,beta,edge --acls=package_upload -
    ```

!!! note

    [brew](https://brew.sh/) automation will automatically detect new releases
    of chezmoi within a few hours and open a pull request in
    [github.com/Homebrew/homebrew-core](https://github.com/Homebrew/homebrew-core)
    to bump the version.

    If needed, the pull request can be created with:

    ```console
    $ brew bump-formula-pr --tag=v1.2.3 chezmoi
    ```

!!! note

    chezmoi is in [Scoop](https://scoop.sh/)'s Main bucket. Scoop's automation
    will automatically detect new releases within a few hours.
