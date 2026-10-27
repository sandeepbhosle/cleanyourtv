# My TV: Amstrad (CloudWalker OS, CVTE board, MediaTek mt5862, Android 9)

Personal setup on top of this repo, not part of the upstream project.

| | |
|---|---|
| `ro.product.model` / device | `SMART_TV` / `mt5862`, manufacturer `MediaTek` |
| Android / SDK | 9 / 28 (launcher `minSdk` 26) |
| Firmware | `mt5862-userdebug 9 PPR2.180905.006.A1 TVOS-04.16.031.01.12 dev-keys` |
| RAM / storage | ~900 MB, `ro.config.low_ram=true` / 3.2 GB `/data` |
| Stock Home | `tv.cloudwalker.cwnxt.launcher.com/tv.cloudwalker.cwnxt.launcher.MainActivity` (kept enabled: CVTE rule) |
| Network ADB | port 5555, `ro.adb.secure=0` (no "Allow" prompt: keep debugging off when not in use) |

## Files

- `launcher/app/src/main/res/values/config.xml`: dock order and `focus_overshoot` false (built into the APK).
- `my-tv/amstrad-theme/config.json`: the glass look from `themes/glass` without `advanced_effects` (low-RAM TV);
  a three-app dock (JioHotstar, Prime Video, YouTube) in three columns; every other app hidden from Home and
  reachable from the App library pill. Netflix is left out: the installed build is the phone app, which cannot
  be signed in to with a remote and showed "Device failure (500.-1025)".
- `my-tv/ads.txt`: the one package disabled, `com.cvte.tv.pictorial` (advert screensaver, ad SDKs, analytics).

## Install or update

Turn USB debugging (and Network/ADB debugging) on in Developer options first, then from the repo root:

```sh
python3 -m venv tools/venv && tools/venv/bin/pip install -r tools/requirements.txt   # once
cd launcher && ./gradlew --no-daemon assembleRelease && cd ..
shasum -a 256 launcher/app/build/outputs/apk/release/app-release.apk
TV_HOST=<tv ip> tools/venv/bin/python tools/b64push.py launcher/app/build/outputs/apk/release/app-release.apk /data/local/tmp/tvhome.apk
TV_HOST=<tv ip> tools/venv/bin/python tools/tv.py 'pm install -r /data/local/tmp/tvhome.apk'
TV_HOST=<tv ip> tools/venv/bin/python tools/tv.py 'rm -f /data/local/tmp/tvhome.apk'
cp themes/glass/wallpaper.jpg my-tv/amstrad-theme/
TV_HOST=<tv ip> tools/theme.sh my-tv/amstrad-theme
```

`pm install -r` keeps the app's data and its Home default. Do not use `launcher/build-and-install.sh` here
without reading it: it can disable a recognised stock launcher, which this CVTE board must keep.

Make it Home (done once, then rebooted and checked):

```sh
TV_HOST=<tv ip> tools/venv/bin/python tools/tv.py 'cmd package set-home-activity --user 0 com.gathin.tvhome/.MainActivity'
```

Close ADB afterwards: debugging off, reboot, `tools/venv/bin/python tools/adb-closed.py <tv ip>` must say closed.

## Remote buttons

The CVTE hotkey service (`com.cvte.tv.api`) takes the app buttons, but TV Home's `InputKeyService` (an
accessibility key filter) sees them first. Mapped by scan code in `amstrad-theme/config.json` (`app_keys`):

| Button | Key / scan code | Mapped to |
|---|---|---|
| NTX | `KEYCODE_QUICK_START3` / 1128 | JioHotstar (stock: the phone Netflix) |
| APP STORE | `KEYCODE_QUICK_START2` / 1127 | Prime Video (stock: `com.stark.store`) |
| Q.VIEW | `KEYCODE_CHANNEL_RECALL` / 363 | HDMI 1, the Tata Play box (`.../HDMIInputService/HW4`) |
| FAV | `KEYCODE_FAV_LIST` / 364 | HDMI 1 |
| YouTube | `KEYCODE_QUICK_START1` / 1126 | left to the TV (opens YouTube) |

Turn the helper on (once, and again after every `tools/theme.sh` push, which can leave it enabled but not bound):

```sh
TV_HOST=<tv ip> tools/venv/bin/python tools/tv.py 'settings put secure enabled_accessibility_services ""; settings put secure accessibility_enabled 0; sleep 1; settings put secure enabled_accessibility_services com.gathin.tvhome/.InputKeyService; settings put secure accessibility_enabled 1'
```

Off: `settings delete secure enabled_accessibility_services; settings put secure accessibility_enabled 0`
(both were unset/0 before), or `am force-stop com.gathin.tvhome` until Home restarts.

## Recovery

| To undo | Command (debugging on) | Without ADB |
|---|---|---|
| Home back to CloudWalker | `tools/tv.py 'cmd package set-home-activity --user 0 tv.cloudwalker.cwnxt.launcher.com/tv.cloudwalker.cwnxt.launcher.MainActivity'` | Settings > Apps > TV Home > Clear defaults |
| Theme | `tools/theme.sh --reset` | - |
| Advert app | `tools/restore.sh my-tv/ads.txt` or `tools/tv.py 'pm enable com.cvte.tv.pictorial'` | - |
| Screensaver setting, if changed | `tools/tv.py 'settings put secure screensaver_enabled 1'` | Settings > Device Preferences > Screen saver |
| The launcher itself | `tools/tv.py 'pm uninstall com.gathin.tvhome'` (our own app only) | Settings > Apps > TV Home > Uninstall |

(Prefix each with `TV_HOST=<tv ip>` and run `tools/tv.py` through `tools/venv/bin/python`.)
