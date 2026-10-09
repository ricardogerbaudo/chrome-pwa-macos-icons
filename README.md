# chrome-pwa-macos-icons

Adaptive icons for Chrome PWAs on macOS.

Native macOS apps ship an icon with light and dark variants, and the Dock switches between them instantly. Chrome PWAs get a single static icon. When you install the same web app twice (two Teams accounts, for example), there is also no way to tell them apart in the Dock.

The `pwa-icons` script gives a PWA a real adaptive icon designed in [Icon Composer](https://developer.apple.com/icon-composer/), and keeps it in place when Chrome recreates the app.

| Light | Dark |
| --- | --- |
| ![Light](example/preview-light.png) | ![Dark](example/preview-dark.png) |

## Requirements

- macOS 26 or later.
- Xcode, only to compile icons (`add`). It can be uninstalled afterwards.
- A PWA installed by Chrome in `~/Applications/Chrome Apps.localized`.

## Usage

```sh
git clone https://github.com/ricardogerbaudo/chrome-pwa-macos-icons.git
cd chrome-pwa-macos-icons
./pwa-icons add "Microsoft Teams (PWA)" ~/path/to/MyIcon.icon
./pwa-icons list
./pwa-icons remove "Microsoft Teams (PWA)"
```

The app name is the one shown in `~/Applications/Chrome Apps.localized`, without `.app`.

To try it, use the icon in [`example/Example.icon`](example/Example.icon). To design your own, open it in Icon Composer (bundled with Xcode) and use appearance specializations for the dark variant.

## How it works

Chrome creates an app shim for each PWA: a small `.app` bundle signed ad hoc. `add`:

1. Compiles the `.icon` document into an asset catalog (`Assets.car`) with `actool`.
2. Copies it into the shim, sets `CFBundleIconName` and re-signs the bundle ad hoc, the same way Chrome signs it.
3. Installs a LaunchAgent that watches the shims. When Chrome recreates one (after an update, for example), the icon is applied again.

The LaunchAgent only runs when a shim changes. It does not poll and does not restart the Dock on theme changes.

State lives in `~/Library/Application Support/pwa-icons`. `remove` restores the original icon, and removing the last app also removes the LaunchAgent.

## Keeping your icons private

Icons for real apps usually contain logos you can't publish. The `.gitignore` excludes `.icon`, `.car` and `.icns` files, except the example. Keep your own icons somewhere else (iCloud Drive works well) and pass their path to `add`. The compiled copy is stored locally, so the LaunchAgent never depends on that folder.

## Limitations

- Only ad hoc signed bundles can be re-signed like this, so it doesn't work with regular apps signed by their developers.
- Tested with Chrome only. Edge creates similar shims in another folder: set `PWA_ICONS_CHROME_APPS` to point to it. This is untested.
