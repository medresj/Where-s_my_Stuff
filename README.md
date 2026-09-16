# Where's My Stuff

An offline Android app for logging where your school things are, and moving them
between places as they travel. Built with Kotlin, Jetpack Compose, Room and
WorkManager. No account, no network permission, nothing leaves the phone.

- `compileSdk` / `targetSdk` **36** (Android 16)
- `minSdk` 26 (Android 8.0)
- Package `com.schoolstuff.tracker`

## Getting an APK

1. Install Android Studio (Meerkat or newer — anything that can build API 36).
2. **File → Open** and select this folder. Wait for the first Gradle sync; it
   downloads Gradle 8.14.1 and the libraries. This takes a few minutes once.
3. If Studio offers an AGP or Kotlin upgrade, accept it. The versions here are
   in `gradle/libs.versions.toml` if you'd rather bump them by hand.
4. **Build → Build Bundle(s) / APK(s) → Build APK(s)** for a debug build. The
   file lands in `app/build/outputs/apk/debug/app-debug.apk`.
5. Copy it to the phone and open it. On One UI, Android asks you to allow
   installs from whatever app you used to open the file — allow it once, install,
   then switch the permission back off.

For an APK you can keep and reinstall over the top later, use **Build → Generate
Signed App Bundle / APK → APK**, create a keystore, and pick the `release` build
variant. Keep that keystore file: without it you can't update your own install.

There's no `gradlew` script in here, because the wrapper needs a binary jar that
couldn't be bundled. Android Studio doesn't need it. If you want to build from a
terminal, run `gradle wrapper` once with a local Gradle 8.14+ install and you'll
get `gradlew` afterwards.

## What it does

**The list.** Everything you own, newest movement first. Each card carries a
colour bar down the left edge for its category, so a long list stays scannable.
Search matches names, subjects, exact spots and place names at once.

**Moving something.** Tap the place pill on any card and pick a new one. Two
taps, no form. This is the action you'll use most, so it's the shallowest.

**Places.** Seven come built in: schoolbag, locker, desk at home, shelf at home,
classroom, lent out, missing. Add your own from the edit screen or the Places
screen (the pin icon, top right). Built-in places can't be deleted; yours can,
and items sitting there just lose their place rather than disappearing.

**Due back dates.** Give a library book or a borrowed calculator a date, and
optionally a reminder. The notification fires at 17:00 the evening before —
early enough that you can still put the thing in your bag. Items due within
three days, or already late, group under one filter pill.

**Categories and subjects.** Six categories drive the colours. Subjects are free
text, and previously used ones come back as tappable suggestions so "Maths"
doesn't become "maths" and "Mathe" three weeks later.

## Layout of the code

```
data/       Room entities, DAOs, database, date helpers
reminder/   WorkManager worker + scheduling
ui/         Compose screens, shared components, ViewModel
ui/theme/   Colour palette and type scale
```

`ItemsViewModel` is the only state holder. It exposes one `ListUiState` combining
the items, the places and the active filters; screens read it and call methods
back. Filtering happens in memory, which is the right call for a few hundred
items and keeps the SQL to one join.

Due dates are stored as UTC midnight (`DueDates`), so a calendar day stays the
same day if the phone crosses a time zone. Reminders convert back to local time
only at the moment of scheduling.

## Changing things

- **Reminder time:** `REMINDER_HOUR` in `reminder/Reminders.kt`.
- **Colours:** `ui/theme/Color.kt`. The category colours live on the `Category`
  enum in `data/Model.kt` so the data layer and UI can't drift apart.
- **Built-in places:** `SchoolLocation.PRESETS`. They're seeded on first launch
  only — changing the list later needs a database migration or a reinstall.
- **New fields on an item:** add to the `Item` entity, bump the `version` in
  `AppDatabase` and add a migration, or wipe app data while developing.

The app is locked to dark regardless of the system theme; see the comment on
`SchoolStuffTheme` if you want it to follow the phone instead.
