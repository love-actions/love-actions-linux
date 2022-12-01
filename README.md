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
    desktop-file-path: ./.github/build/linux/dev/template.desktop
    executable-name: app
    icon-path: ./.github/build/linux/dev/icon.png
    love-package: ./game.love
    lib-path: ./lib
    share-path: ./share
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

## All inputs

| Name                  | Required  | Default           | Description                                                                                                                                             |
| :-------------------- | --------- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `desktop-file-path` | `false` | `""`            | Path to the `.desktop` file, see [Desktop integration](https://docs.appimage.org/reference/desktop-integration.html). Use LÖVE default if not specified |
| `executable-name`   | `false` | `"love"`        | Executable name. Used as appImage's internal executable filename                                                                                        |
| `icon-path`         | `false` | `""`            | Path to the appImage's icon. Use LÖVE default if not specified                                                                                         |
| `love-package`      | `false` | `"./game.love"` | Love package. Used to assemble the executable                                                                                                           |
| `lib-path`          | `false` | `""`            | Path to the library folder. Would copy all contents to `squashfs-root/lib` excluding top folder                                                      |
| `share-path`        | `false` | `""`            | Path to the share folder. Would copy all contents to `squashfs-root/share` excluding top folder                                                       |
| `product-name`      | `false` | `"love_app"`    | Base name of the package. Used to rename products                                                                                                       |
| `output-folder`     | `false` | `"./build"`     | Packages output folder. All packages would be placed here                                                                                               |

## All outputs

| Name              | Example                       | Description                                                                                     |
| :---------------- | ----------------------------- | ----------------------------------------------------------------------------------------------- |
| `package-paths` | `./build/love_app.AppImage` | built packages' paths in a bash-style list relative to the repository root, separated by spaces |
