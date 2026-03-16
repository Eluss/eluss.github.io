---
layout: devlog
title: "yyyy vs YYYY: The Date Format That Breaks Every January"
category: devlog
tags: [ios, swift, bugs, datetime]
---
Two format specifiers that look nearly identical and produce the same output 361 days a year.

`yyyy` — calendar year. The year the date falls in.
`YYYY` — ISO week year. The year that *owns the week* the date falls in.

They diverge at year boundaries, because ISO weeks can straddle two calendar years. ISO week 1 is defined as the week containing the first Thursday of the year. That means late December dates can belong to week 1 of the *next* year.

**December 29, 2019:**

```swift
let formatter = DateFormatter()
formatter.locale = Locale(identifier: "en_US_POSIX")

formatter.dateFormat = "yyyy-MM-dd"
// → "2019-12-29" ✓

formatter.dateFormat = "YYYY-MM-dd"
// → "2020-12-29" ✗
```

December 29, 2019 falls in ISO week 1 of 2020. So `YYYY` returns 2020, but the day and month are still from 2019 — giving you a date that doesn't exist.