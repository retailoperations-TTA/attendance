# TTA Attendance Tracker

One attendance tracker for all four Travel Agency stores: SoHo, Union Square, Downtown Brooklyn and 5th Ave. It is the SoHo tracker with store tabs, one sign-in, per-store access, and the extras the Union Square tracker had (approved time off, add a budtender by hand, Ctrl+Z undo).

Live at https://retailoperations-tta.github.io/attendance/

## Stores and access

Everything lives in the SoHo Firebase project (`attendance-tracker-c4c8e`), under `stores/soho`, `stores/usq`, `stores/dtbk` and `stores/fifth`. Each store has its own attendance grid, import history, employee list, schedule, time off and settings.

Who sees what is set on the **Admin** page (hamburger menu, admins only):

- **view**: can open the store and read everything. Nothing can be changed, and no alert emails are sent from that screen.
- **edit**: can import, edit cells, record corrective actions, and so on.
- **admin**: every store, plus the Admin page. `retailoperations@thetravelagency.co` and `mark.muniz@thetravelagency.co` are always admins (also in the database rules).

People sign in with Google (their work Google account) or with the email and password from the old SoHo tracker. Admins add people ahead of time on the Admin page under **Add someone**: type the email address, pick view or edit per store, and the access is waiting for them the first time they sign in. Several at once works as one per line, for example `name@thetravelagency.co, soho:edit, usq:view`. Someone who signs in without having been added sees "no store has been opened up to you yet" until an admin adds them; the screen updates on its own when that happens.

Google sign-in needs the Google provider turned on in Firebase Authentication for `attendance-tracker-c4c8e`, with `retailoperations-tta.github.io` under Authorized domains.

## Per-store settings (Admin page)

Company name and work location on corrective-action forms, the direct supervisor, who the alert email goes to, the manager names dropped from imports, supervisor first names, whether the 3-point alert email is on, and the points that call for a 2nd warning, a Final warning and Termination (SoHo used 3 for every step, Union Square used 2, 2 and 1).

## Importing

Open the store, then hamburger menu → Import. One drop zone takes:

- the weekly attendance notices export (points)
- the When I Work **Schedule** export and **Timesheets** export (scheduled vs. worked, no points)
- the When I Work **Time-Off Requests** export: approved NYC-PS&STO hours go on the request's start date, every approved request shades the days it covers (sick = pink with 🤒, other time off = yellow), and an approved sick day that already has a point is changed to Excused with a note
- the employee ID list

A weekly re-import never overwrites a write-up marker, an Excused, or logged sick time with a point. The point goes into the cell's note instead.

## Years

A store opens on the last 90 days. **All of <year>** in the month filter means the current calendar year, so days carried over from an earlier year stay out of the day-to-day grid. Nothing is deleted: earlier years sit under **Earlier years** in the same dropdown, as **All Months (every year)**, **All of 2025**, or a single month of it, and the CSV export still carries every day on record.

**Annual Pts** counts the current calendar year only, the same window the 32 NYC-PS&STO hours reset on. Corrective actions are unaffected: those read points since the last write-up, or the last 6 months when there is no write-up.

## Moving the old trackers in

Each old tracker's data comes in through **Admin → Bring in data from an old tracker**:

1. In the old tracker's Firebase console, open Realtime Database, click the ⋮ menu and choose **Export JSON**.
2. In the new tracker, open the store the data belongs to, then upload that file on the Admin page. The SoHo, Union Square, 5th Ave and Jeff's multi-store layouts are all recognized. For Jeff's file, pick the Brooklyn store in the dropdown.
3. Existing entries in the store are kept; the file only fills in what is missing. People already at 3 points are marked as already alerted, so no corrective-action emails go out for old history.

Brooklyn's values are translated: `late 0.5` / `early out` become `0.5`, `late 1.0` becomes `1`, `NC/NS` stays NC/NS (and now counts as 1 point everywhere), `excused_paid` becomes Excused, `USL` and `excused_unpaid` become NYC-PS&STO entries with the logged minutes (8 hours when none were logged), and the `management` flag keeps managers out of imports and alerts.

**SoHo** is a one-click copy: **Admin → Copy old SoHo data into SoHo**. The old SoHo page keeps working on its own data until you switch over, so copy again right before the switch.

## Setup

1. Firebase console for `attendance-tracker-c4c8e` → Realtime Database → Rules: paste `database.rules.json`. The old SoHo tracker's top-level nodes stay open to any signed-in user until it is retired; remove those lines then.
2. Authentication → Sign-in method: Email/Password is already on. Turn on Google if you want the Google button to work, and make sure `retailoperations-tta.github.io` is under Authorized domains.
3. Repo Settings → Pages: deploy from `main`, root folder.

## What changed from the SoHo tracker

- `NC/NS` counts as 1 point (it was 0 before; SoHo had no NC/NS entries, Brooklyn has many).
- The 32 NYC-PS&STO hours reset each calendar year.
- Undo puts back only the cells the undone change touched, so someone else's edits in between are left alone. Ctrl+Z (Cmd+Z) undoes too.
- The employee list import merges instead of replacing, so IDs typed by hand are kept. Nicknames also match when one first name ends with the other.
- The CSV export is named after the store.
