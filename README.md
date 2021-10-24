# Docker Image for Unity3d

CI specialised docker images for Unity3d.

<br><br><br>

## :wrench: Use the images on Github Actions

[Unity - Builder](https://github.com/marketplace/actions/unity-builder) and [Unity - Test runner](https://github.com/marketplace/actions/unity-test-runner) actions support `customImage` parameter:

```yml
- uses: game-ci/unity-test-runner@v2
  with:
    customImage: mobsakai/unity3d:2020.3.0f1-webgl
    ...
```

```yml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        unityVersion:
          [
            2019.3.15f1,
            2019.4.16f1,
            2020.1.16f1,
          ]
        include:
          - targetPlatform: StandaloneLinux64
            module: -base
          - targetPlatform: StandaloneOSX
            module: -mac
          - targetPlatform: StandaloneWindows64
            module: -windows
          - targetPlatform: iOS
            module: -ios
          - targetPlatform: Android
            module: -android
          - targetPlatform: WebGL
            module: -webgl
    steps:
      - uses: actions/checkout@v2

      - uses: game-ci/unity-test-runner@v2
        env:
          UNITY_LICENSE: ${{ secrets.UNITY_LICENSE }
        with:
          customImage: mobsakai/unity3d:${{ matrix.unityVersion }}${{ matrix.module }}
          customParameters: -nographics
          targetPlatform: ${{ matrix.targetPlatform }}
          githubToken: ${{ github.token }}

      - uses: game-ci/unity-builder@v2
        env:
          UNITY_LICENSE: ${{ secrets.UNITY_LICENSE }}
        with:
          customImage: mobsakai/unity3d:${{ matrix.unityVersion }}${{ matrix.module }}
          targetPlatform: ${{ matrix.targetPlatform }}

      - uses: actions/upload-artifact@v2
        with:
          name: Build
          path: build
```

For details, see https://game.ci/docs/github/getting-started

<br><br><br><br>
<br><br><br><br>

# :eight_spoked_asterisk: Changes from [game-ci/docker](https://github.com/game-ci/docker)

* Remove original workflows
  * Cats
  * Docs
  * New Versions (Base/Hub/Editor/Retry)
  * PR
* Remove backend-versioning server (Use `workflow_dispatch` and [unity-changeset](https://www.npmjs.com/package/unity-changeset) instead)
* Release automatically with [semantic-release](https://github.com/semantic-release/semantic-release)
  * The release is [based on a committed message](https://www.conventionalcommits.org/)
  * Tagging based on [Semantic Versioning 2.0.0](https://semver.org/)
    * Use `v1.0.0` insted of `v1.0`
* Run workflow manually or automatically
  * Build all images on tag pushed
  * Check/Build all new editor images every day
  * Run workflow manually from [Actions page](../../actions)
* Fast skip earlier builds of images that already exist
* Add build configurations file (`.github/workflow/.env`)
* Support for alpha/beta versions of Unity (e.g. 2022.1.0a, 2021.2.0a)
  * :warning: **NOTE: The versions removed from [Unity beta](https://unity3d.com/beta) will not be updated**
* Grouping workflows in a module (base, ios, android, ...)
  * Improve the visibility of actions page
  * Easy to retry
* Support short image tags (eg. `mobsakai/unity3d:2020.3.0f1` for Linux (Mono), `mobsakai/unity3d:2020.3.0f1-webgl` for WebGL, etc.)
* Image tags that fail to build twice will automatically be ignored
  * [This locked issue](https://github.com/mob-sakai/docker/issues/19) is used for failure log

<br><br>

## :hammer: How to build images

### 1. :pencil2: Setup build configurations (.env)

See [.env](https://github.com/mob-sakai/docker/blob/main/.env)

```sh
# ================= Registry settings =================
DOCKER_REGISTRY=docker.io
# DOCKER_REGISTRY=ghcr.io
# DOCKER_REGISTRY=gcr.io
# ================ Repository settings ================
BASE_IMAGE_REPOSITORY=mobsakai/unity3d_base
HUB_IMAGE_REPOSITORY=mobsakai/unity3d_hub
EDITOR_IMAGE_REPOSITORY=mobsakai/unity3d
# =================== Build settings ==================
UBUNTU_IMAGE=ubuntu:18.04
MINIMUM_UNITY_VERSION=2018.3
INCLUDE_BETA_VERSIONS=true
# Excluded image tags (Regular expressions)
EXCLUDE_IMAGE_TAGS="
2018.*-linux-il2cpp
2019.1.*-linux-il2cpp
2019.2.*-linux-il2cpp
<<AUTO_IGNORED_IMAGE_TAGS>>
"
```

<br><br>

### 2. :key: Setup repository secrets

| Name                | Description                                                    |
| ------------------- | -------------------------------------------------------------- |
| `DOCKER_USERNAME`   | Docker username to login.                                      |
| `DOCKER_PASSWORD`   | Docker password or access token to login.                      |
| `GH_WORKFLOW_TOKEN` | A [Github parsonal access token][] with `workflow` premission. |

[Github parsonal access token]: https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/creating-a-personal-access-token

<br><br>

### 3. :arrow_forward: Run workflows (automatically)

All workflows will be run automatically.

| Workflow       | Description                                                  | Trigger                                                                         |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------------------------- |
| `Release`      | Release new tag.                                             | - Pushed commits (include feat or fix) on `main` branch                         |
| `Build All`    | Build base/hub images and dispatch `Build Editor` workflows. | - Released a new version<br>- Scheduled (daily)<br>- New Unity version released |
| `Build Editor` | Build editor images with a specific Unity module             | - Dispatched from `Build All`                                                   |

<br><br>

### 4. :arrow_forward: Run workflows (manually)

You can run them manually from the [Actions page](../../actions).

**NOTE: You need permissions to run the workflow.**

<br><br><br>

## :mag: FAQ

### :exclamation: Error on time limit or API limit

Because the combination of the editor build is so large, the builds may fail due to the time limit of github actions (<6 hours) or API limitations.

Re-run `Build All` workflow manually after all jobs are done.

### :exclamation: Missing library for editor

If a missing library is found, fix the `editor/Dockerfile` or `base/Dockerfile`.

<br><br><br>

## :bulb: Next plans

* Test the build for each patch versions (2018.3.0, 2018.3.1, ...)
  * May be unnecessary for stable versions (2018.x, 2019.x)
  * Build a simple project for all platforms
  * Inspect the missing library
* Notify the error summary to mail, Slack or Discord


<br><br><br><br>
<br><br><br><br>

---
:books: << The following is the original Readme >> :books:

# Docker images for Unity

(Not affiliated with Unity Technologies)

Source of CI specialised docker images for Unity, free to use for everyone.

Please find our
[website](https://game.ci)
for any related
[documentation](https://game.ci/docs).

## Base

See the [base readme](./base/README.md) for base image usage.

## Hub

See the [hub readme](./hub/README.md) for hub image usage. 

## Editor

See the [editor readme](./editor/README.md) for editor image usage.

## Community

Feel free to join us on
<a href="http://game.ci/discord"><img height="30" src="media/Discord-Logo.svg" alt="Discord" /></a>
and engage with the community.

## Contributing

To contribute, please see the [development readme](./DEVELOPMENT.md) 
after you agree with our [code of conduct](./CODE_OF_CONDUCT.md) 
and have read the [contribution guide](./CONTRIBUTING.md).

## Support us

GameCI is free for everyone forever.

You can support us at [OpenCollective](https://opencollective.com/game-ci).

## Licence

This repository is [MIT](./LICENSE) licensed.

This includes all contributions from the community.
