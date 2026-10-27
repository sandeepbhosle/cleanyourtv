# My TV: Amstrad (CloudWalker OS, CVTE board, MediaTek mt5862, Android 9)

Personal setup, not part of the upstream project.

- `launcher/app/src/main/res/values/config.xml`: dock order and `focus_overshoot` false (built into the APK).
- `amstrad-theme/config.json`: the glass look from `themes/glass` without `advanced_effects` (low-RAM TV),
  a three-app dock (JioHotstar, Prime Video, YouTube) in three columns, every other app hidden from Home
  and reachable from the App library pill. Netflix is left out: the installed build is the phone app,
  which cannot be signed in to with a remote and showed "Device failure (500.-1025)".

Push it (copies the glass wallpaper in first):

```sh
cp themes/glass/wallpaper.jpg my-tv/amstrad-theme/
TV_HOST=<tv ip> tools/theme.sh my-tv/amstrad-theme
```

Undo: `TV_HOST=<tv ip> tools/theme.sh --reset`

`ads.txt`: the one package disabled on this TV, the CVTE advert screensaver and analytics app.
Undo with `TV_HOST=<tv ip> tools/restore.sh my-tv/ads.txt`.
