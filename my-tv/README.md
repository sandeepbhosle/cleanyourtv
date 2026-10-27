# My TV: Amstrad (CloudWalker OS, CVTE board, MediaTek mt5862, Android 9)

Personal setup, not part of the upstream project.

- `launcher/app/src/main/res/values/config.xml`: dock order and `focus_overshoot` false (built into the APK).
- `amstrad-theme/config.json`: the glass look from `themes/glass` without `advanced_effects` (low-RAM TV),
  Netflix (phone build, no leanback entry) added with `library_apps`, the duplicate Hotstar tile hidden.

Push it (copies the glass wallpaper in first):

```sh
cp themes/glass/wallpaper.jpg my-tv/amstrad-theme/
TV_HOST=<tv ip> tools/theme.sh my-tv/amstrad-theme
```

Undo: `TV_HOST=<tv ip> tools/theme.sh --reset`
