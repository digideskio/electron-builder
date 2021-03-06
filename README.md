# electron-builder [![npm version](https://img.shields.io/npm/v/electron-builder.svg)](https://npmjs.org/package/electron-builder)
Complete solution to package and build ready for distribution and "auto update" Electron app for OS X, Windows and Linux.

* NPM packages management:
  * [Native application dependencies](http://electron.atom.io/docs/latest/tutorial/using-native-node-modules/) compilation (only if [two-package.json project structure](#two-packagejson-structure) used).
  * Development dependencies are never included. You don't need to ignore it explicitly.
* [Code Signing](https://github.com/electron-userland/electron-builder/wiki/Code-Signing) on a CI server or development machine.
* [Auto Update](#auto-update) ready application packaging.
* [Build version management](#build-version-management).
* Numerous target formats:
  * All platforms: `7z`, `zip`, `tar.xz`, `tar.lz`, `tar.gz`, `tar.bz2`.
  * [OS X](https://github.com/electron-userland/electron-builder/wiki/Options#OsXBuildOptions-target): `dmg`, `mas`.
  * [Linux](https://github.com/electron-userland/electron-builder/wiki/Options#LinuxBuildOptions-target): `deb`, `rpm`, `freebsd`, `pacman`, `p5p`, `apk`.
  * Windows: Squirrel.Windows. NSIS will be supported soon.
* [Publishing artifacts to GitHub Releases](https://github.com/electron-userland/electron-builder/wiki/Publishing-Artifacts).

[electron-packager](https://github.com/electron-userland/electron-packager) and
[appdmg](https://github.com/LinusU/node-appdmg) are used under the hood.

Real project example — [onshape-desktop-shell](https://github.com/develar/onshape-desktop-shell).

# Two package.json structure

We recommend to use two package.json files (it is not required, you can build project with any structure).

1. For development

   In the root of the project. Here you declare dependencies for your development environment and build scripts.

2. For your application

   In the `app` directory. *Only this directory is distributed with real application.*

Why?

1. Native npm modules (those written in C, not JavaScript) need to be compiled, and here we have two different compilation targets for them. Those used in application need to be compiled against electron runtime, and all `devDependencies` need to be compiled against your locally installed node.js. Thanks to having two files this is trivial (see [#39](https://github.com/electron-userland/electron-builder/issues/39)).
2. No need to specify which [files](https://github.com/electron-userland/electron-builder/wiki/Options#BuildMetadata-files) to include in the app (because development files reside outside the `app` directory).

Please see [Loading App Dependencies Manually](https://github.com/electron-userland/electron-builder/wiki/Loading-App-Dependencies-Manually) and [#379](https://github.com/electron-userland/electron-builder/issues/379#issuecomment-218503881).

# Configuration

See [options](https://github.com/electron-userland/electron-builder/wiki/Options), but consider to follow simple guide outlined below at first.

For a production app you need to sign your application, see [Where to buy code signing certificate](https://github.com/electron-userland/electron-builder/wiki/Code-Signing#where-to-buy-code-signing-certificate).

## In short
1. Specify standard fields in the application `package.json` — [name](https://github.com/electron-userland/electron-builder/wiki/Options#AppMetadata-name), `description`, `version` and [author](https://docs.npmjs.com/files/package.json#people-fields-author-contributors) (for Linux [homepage](https://github.com/electron-userland/electron-builder/wiki/Options#AppMetadata-homepage) and [license](https://github.com/electron-userland/electron-builder/wiki/Options#AppMetadata-license) are also required).

2. Specify [build](https://github.com/electron-userland/electron-builder/wiki/Options#build) field in the development `package.json`:
    ```json
    "build": {
      "app-bundle-id": "your.id",
      "app-category-type": "your.app.category.type",
      "win": {
        "iconUrl": "(windows-only) https link to icon"
      }
    }
    ```
   See [options](https://github.com/electron-userland/electron-builder/wiki/Options). This object will be used as a source of [electron-packager](https://www.npmjs.com/package/electron-packager#packageropts-callback) options. You can specify any other options here.

3. Create directory `build` in the root of the project and put your `background.png` (OS X DMG background), `icon.icns` (OS X app icon) and `icon.ico` (Windows app icon).

   <a id="user-content-linuxIcon" class="anchor" href="#linuxIcon" aria-hidden="true"></a>Linux icon set will be generated automatically on the fly from the OS X `icns` file (or you can put them into the `build/icons` directory — filename must contains size (e.g. `32x32.png`)).

4. Add [scripts](https://docs.npmjs.com/cli/run-script) to the development `package.json`:
    ```json
    "scripts": {
      "postinstall": "install-app-deps",
      "pack": "build --target dir",
      "dist": "build"
    }
    ```
    And then you can run `npm run dist` (to package in a distributable format (e.g. dmg, windows installer, deb package)) or `npm run pack`.

5. Install [required system packages](https://github.com/electron-userland/electron-builder/wiki/Multi-Platform-Build).

Please note — packaged into an asar archive [by default](https://github.com/electron-userland/electron-builder/wiki/Options#BuildMetadata-asar).

# Auto Update
`electron-builder` produces all required artifacts:

* `.dmg`: OS X installer, required for OS X user to initial install.
* `-mac.zip`: required for Squirrel.Mac.
* `.exe` and `-ia32.exe`: Windows installer, required for Windows user to initial install. Please note — [your app must handle Squirrel.Windows events](https://github.com/electronjs/windows-installer#handling-squirrel-events). See [real example](https://github.com/develar/onshape-desktop-shell/blob/master/src/WinSquirrelStartupEventHandler.ts).
* `.full-nupkg`: required for Squirrel.Windows.

For auto updating to work, you must implement and configure Electron's [`autoUpdater`](http://electron.atom.io/docs/latest/api/auto-updater/) module ([example](https://github.com/develar/onshape-desktop-shell/blob/master/src/AppUpdater.ts)).
You also need to deploy your releases to a server.
Consider using [Nuts](https://github.com/GitbookIO/nuts) (GitHub as a backend to store assets) or [Electron Release Server](https://github.com/ArekSredzki/electron-release-server).
See the [Publishing Artifacts](https://github.com/electron-userland/electron-builder/wiki/Publishing-Artifacts) section of the [Wiki](https://github.com/electron-userland/electron-builder/wiki) for information on configuring your CI environment for automatic deployment.

For windows consider only [distributing 64-bit versions](https://github.com/electron-userland/electron-builder/issues/359#issuecomment-214851130).

# Build Version Management
`CFBundleVersion` (OS X) and `FileVersion` (Windows) will be set automatically to `version`.`build_number` on CI server (Travis, AppVeyor and CircleCI supported).

# CLI Usage
Execute `node_modules/.bin/build --help` to get actual CLI usage guide.
In most cases you should not explicitly pass flags, so, we don't want to promote it here ([npm lifecycle](https://docs.npmjs.com/misc/scripts#current-lifecycle-event) is supported and script name is taken in account).
Want more — please file issue.

# Programmatic Usage
See `node_modules/electron-builder/out/electron-builder.d.ts`. [Typings](https://github.com/Microsoft/TypeScript/wiki/Typings-for-npm-packages) is supported.

```js
"use strict"

const builder = require("electron-builder")
const Platform = builder.Platform

// Promise is returned
builder.build({
  targets: Platform.OSX.createTarget(),
  devMetadata: {
    "//": "build and other properties, see https://goo.gl/5jVxoO"
  }
})
  .then(() => {
    // handle result
  })
  .catch((error) => {
    // handle error
  })
```

# Further Reading
See the [Wiki](https://github.com/electron-userland/electron-builder/wiki) for more documentation.
