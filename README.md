# love-actions-linux

## About

Github Action for building & deploying Linux `.AppImage` packages of a [LÖVE](https://love2d.org/) framework based game.

### Related actions

See related actions below:

[Love actions bare package](https://github.com/marketplace/actions/love-actions-bare-package)

[Love actions for testing](https://github.com/marketplace/actions/love-actions-for-testing)

[Love actions for android](https://github.com/marketplace/actions/love-actions-for-android)

[Love actions for iOS](https://github.com/marketplace/actions/love-actions-for-ios)

[Love actions for macOS portable](https://github.com/marketplace/actions/love-actions-for-macos-portable)

[Love actions for macOS AppStore](https://github.com/marketplace/actions/love-actions-for-macos-appstore)

[Love actions for Windows](https://github.com/marketplace/actions/love-actions-for-windows)

## Quick example

```yaml
- name: Build Linux packages
  id: build-packages
  uses: love-action/love-actions-linux@v1
  with:
    app-name: Love App
    bundle-id: com.example.loveapp
    description: My awesome love app
    version-string: "1.0.0"
    icon-path: ./.github/build/linux/dev/icon.png
    love-package: ./game.love
    lib-path: ./lib
    share-path: ./share
    build-deb: true
    product-name: love_app
    output-folder: ./dist
```

### Notice

If you want to load dynamic libraries in love runtime, you should place them in the `{lib-path}/lua/5.1/` dir.

Your library folder should look like this:

```
 - lib
    |- a.so
    |- b.so
    |- lua
        |-5.1
           |- load_in_lua.so
```

## With [Love actions bare package](https://github.com/marketplace/actions/love-actions-bare-package) and [Love actions for testing](https://github.com/marketplace/actions/love-actions-for-testing)

```yml
env:
  BUILD_TYPE: ${{ fromJSON('["dev", "release"]')[startsWith(github.ref, 'refs/tags/v')] }}
  CORE_LOVE_PACKAGE_PATH: ./core.love
  CORE_LOVE_ARTIFACT_NAME: core_love_package
  PRODUCT_NAME: my_love_app
  BUNDLE_ID: com.example.myloveapp

jobs:
  build-core:
    runs-on: ubuntu-latest
    env:
      OUTPUT_FOLDER: ./build
    steps:
      - uses: actions/checkout@v3
        with:
          submodules: recursive
      - name: Build core love package
        uses: love-actions/love-actions-core@v1
        with:
          build-list: ./media/ ./parts/ ./Zframework/ ./conf.lua ./main.lua ./version.lua
          package-path: ${{ env.CORE_LOVE_PACKAGE_PATH }}
      - name: Upload core love package
        uses: actions/upload-artifact@v3
        with:
          name: ${{ env.CORE_LOVE_ARTIFACT_NAME }}
          path: ${{ env.CORE_LOVE_PACKAGE_PATH }}
  auto-test:
    runs-on: ubuntu-latest
    needs: build-core
    steps:
      - uses: actions/checkout@v3
        with:
          submodules: recursive
      - name: Love actions for testing
        uses: love-actions/love-actions-test@v1
        with:
          font-path: ./parts/fonts/proportional.otf
          language-folder: ./parts/language
  build-linux:
    runs-on: ubuntu-latest
    needs: [build-core, auto-test]
    env:
      OUTPUT_FOLDER: ./build
    steps:
      - uses: actions/checkout@v3
        with:
          submodules: recursive
      # Download your core love package here
      - name: Download core love package
        uses: actions/download-artifact@v3
        with:
          name: ${{ env.CORE_LOVE_ARTIFACT_NAME }}
      # This is an example dynamic library
      - name: Download ColdClear
        uses: ./.github/actions/get-cc
        with:
          platform: Linux
          dir: ./ColdClear
      # Notice the subdir /lua/5.1
      - name: Process ColdClear
        shell: bash
        run: |
          cd ./ColdClear
          mkdir -p ./lib/lua/5.1
          mv ./x64/CCloader.so ./lib/lua/5.1
      - name: Build Linux packages
        id: build-packages
        uses: love-actions/love-actions-linux@v1
        with:
          app-name: Love App
          bundle-id: com.example.loveapp
          description: My awesome love app
          version-string: "1.0.0"
          icon-path: ./.github/build/linux/${{ env.BUILD_TYPE }}/icon.png
          love-package: ${{ env.CORE_LOVE_PACKAGE_PATH }}
          lib-path: ./ColdClear/lib
          product-name: ${{ env.PRODUCT_NAME }}
          output-folder: ${{ env.OUTPUT_FOLDER }}
      - name: Upload AppImage artifact
        uses: actions/upload-artifact@v3
        with:
          name: ${{ needs.get-info.outputs.base-name }}_Linux_AppImage
          path: ${{ env.OUTPUT_FOLDER }}/${{ env.PRODUCT_NAME }}.AppImage
```

## All inputs

| Name               | Required  | Default                    | Description                                                               |
| :----------------- | --------- | -------------------------- | ------------------------------------------------------------------------- |
| `app-name`       | `false` | `"Love App"`             | App display name. Would be used in desktop file                           |
| `bundle-id`      | `false` | `"org.loveactions.love"` | App bundle ID. Would be used to rename debian package's desktop file      |
| `description`    | `false` | `"love"`                 | App description. Would be used in control file and desktop file           |
| `version-string` | `false` | `"love"`                 | App version string. Use 3 numbers seperated by dots                       |
| `icon-path`      | `false` | `""`                     | Path to the png icon. Would be used in desktop file                      |
| `love-package`   | `false` | `"./game.love"`          | Love package. Used to assemble the executable                             |
| `lib-path`       | `false` | `""`                     | Path to the library folder. Would copy all contents excluding top folder |
| `share-path`     | `false` | `""`                     | Path to the share folder. Would copy all contents excluding top folder    |
| `build-deb`      | `false` | `"true"`                 | Switch to control build debian package or not                             |
| `product-name`   | `false` | `"love_app"`             | Base name of the package. Used to rename products                         |
| `output-folder`  | `false` | `"./build"`              | Packages output folder. All packages would be placed here                 |

## All outputs

| Name              | Example                       | Description                                                                                     |
| :---------------- | ----------------------------- | ----------------------------------------------------------------------------------------------- |
| `package-paths` | `./build/love_app.AppImage` | built packages' paths in a bash-style list relative to the repository root, separated by spaces |
