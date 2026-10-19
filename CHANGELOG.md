# Changelog

All notable changes to `qalainau/filament-warp-scheduler` will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.1.3] - 2026-10-20

### Changed

- Support moves to https://warpui.com/support?plugin=scheduler: questions before and after buying, bug reports (also
  private ones) and an email when the fix is released. The README, the license agreement and the `support` links in
  `composer.json` point there, and `composer.json` links the public docs instead of the private repository.

## [1.1.2] - 2026-10-20

### Changed

- The license agreement adds a refund policy (section 5): a full refund within seven days of purchase for a defect
  that cannot be fixed or worked around.

## [1.1.1] - 2026-10-20

### Changed

- The license agreement no longer lists a Lifetime tier. The tiers are Single Project and Unlimited Projects, each with one year of updates.

## [1.1.0] - 2026-10-19

### Added

- A board (kanban) above or beside the timeline: `EventSource::stage()` names the stage column and `board()` lists
  the columns, as `BoardColumn`s or an enum. Cards move between columns, onto a resource to be scheduled, and back
  from the timeline to a column, which takes the record off the timeline. Columns can have a WIP limit
  (`limit()`), show only the nearest cards (`recent()`) and keep the schedule (`keepsSchedule()`); cards can show
  how long they have waited (`agingFrom()`). Users switch between the board above or beside the timeline, the board
  only and the timeline only. `canMoveCard()`, `afterCardMoved()`, `cardView()` and
  `EventSource::unscheduleUsing()` customise it, and the browser receives `warp-scheduler:card-moved`.
- `warp-scheduler-refresh` makes every scheduler on the page fetch its events again (and redraw its board).

## [1.0.6] - 2026-10-18

### Changed

- The commercial license agreement is now one text for every Warp plugin (the plugin you purchase is the "Software"); the terms are unchanged.
- The Excel export writes the sheet to a temporary file as it goes instead of building it in memory, so exporting many events needs much less memory.

## [1.0.5] - 2026-10-14

### Fixed

- Creating, editing or deleting an event no longer re-renders the whole page, which showed as the page refreshing
  once (on a page with many resources the response was several hundred kilobytes). Only the closed modal and the
  unscheduled pool are rendered again; the board fetches its events as before. Typing in an event's form while it is
  open renders only that modal, also when the last change is sent together with Save.
- While dragging on the timeline or the Gantt chart, the start and end times (and why a place is refused or flagged)
  are shown in the time header, over a band marking the event's time, instead of on the board, where they covered the
  rows around the drop target.
- Dragging an item from the unscheduled pool or the side list no longer shows the browser's picture of the item under
  the cursor, which covered the drop target. On the board the item is drawn where it would land, and its times are
  shown in the time header.
- A short event can be lengthened again. Events 18 pixels tall (or wide, on the timeline) or less had no resize
  edge, so an event shortened to 15 minutes could only be moved. The end edge now stays grabbable on any event; the
  start edge still needs room for moving.

## [1.0.4] - 2026-10-14

### Fixed

- Placing an event from the unscheduled pool, by dragging it or with the sparkles button, no longer re-renders the
  whole page and fetches every loaded event again, which showed as the page refreshing once. The placed event comes
  back in the response, and the item leaves the pool in the browser.
- After an action (create, edit, delete) or a change to a recurring series, the board keeps showing its events while
  they are fetched again instead of emptying until the response arrives.
- The start and end times shown while dragging no longer cover the dragged event when there is no room above it:
  they go below it, on a translucent background.

## [1.0.3] - 2026-10-14

### Fixed

- Event modals open from dashboard widgets. `SchedulerWidget` did not render the action modals, so clicking, creating,
  editing or deleting an event on a widget showed nothing. If you give a widget a view of your own, keep
  `<x-filament-actions::modals />` in it.
- After clicking an event and closing its quick info or menu with Escape, the arrow keys, Delete and Ctrl/Cmd+C act on
  the selected event again; focus now returns to the board.
- The Resource day view shows one day instead of the same week as Resource week.
- Pulling a locked or read-only event and letting go no longer opens its edit form.
- The list of events for screen readers and the browser's find is updated when an event is renamed or reassigned
  without moving.
- On phones, resource names are shown in the narrow resource column; the extra columns and the per-row utilization
  meter are left out there.

### Changed

- The phone layout (`mobileView()` and the narrow resource column) follows the screen width instead of the
  scheduler's width, as the README described, so a widget in half a desktop dashboard keeps its view. A scheduler
  narrower than 640px still shows its views in a select and hides the unscheduled pool.

## [1.0.2] - 2026-10-14

### Fixed

- The Excel export includes the two fills Excel reserves in the workbook's styles, and its temporary file is removed
  even when writing fails.
- On a slow first visit, when Filament's font finished loading after the scheduler was first drawn, text kept the
  fallback font's widths. The scheduler now measures again and redraws once the font has loaded.
- Avatars from SVG images without a width and height are drawn instead of left empty.

### Changed

- `composer.json` declares `ext-zip`, which the Excel export needs.

## [1.0.1] - 2026-10-13

### Fixed

- Dropping an element whose `data-warp-scheduler-draggable` is a number, a string or a JSON array no longer fails on the
  server; the value is passed to `onExternalDrop()` as `['value' => ...]`.
- The README no longer mentions `->recordAttributes()`, which Filament 5 tables do not have, and explains how to escape
  the JSON in `extraAttributes()`.

## [1.0.0] - 2026-10-13

### Added

- Canvas-rendered resource timeline for Filament 5 with day, week, month, quarter and year zoom, nested
  collapsible resource groups, vertical and horizontal virtualization and events loaded per block of resources.
- Resource day, week, day, month and list views and a Gantt chart, with a view switcher (`views()`).
- `SchedulerPage`, `SchedulerWidget` and the `InteractsWithScheduler` trait; `EventSource` for any number of
  event models; the `Schedulable` and `SchedulerResource` contracts.
- Drag and drop to move, copy (Alt) and resize events, range selection to create events, an unscheduled pool,
  Filament create, edit and delete actions, custom `eventActions()` in a context menu, keyboard editing and
  touch dragging. Changes are checked against read-only mode, locks and model policies on the server.
- Filters form, tenant scoping, display time zones with floating all-day dates, dark mode and SPA mode support.
- Recurring events (`recurring()`, `RecurrenceInput`) stored as RFC 5545 rules, expanded on the server, with
  this, following and all edit scopes.
- Business hours, availability and blocked time tables, capacity and conflict rules (`NoOverlap`,
  `WithinAvailability`, `NoBlockedTime`, `CustomRule`) that reject or warn, previewed while dragging.
- Utilization meters, group heatmaps and a histogram (`utilization()`).
- Work week, N-day, year, month agenda and multi-day resource views (grouped by resource or by date).
- Resource and event avatars (`resourceImage()`, `image()`), event variants (`variant()`), labelled blocked time,
  shift header rows (`shifts()`) and a side list to drag people onto slots or events (`sideList()`).
- A compact layout for narrow screens (`mobileView()`) and mirrored drawing for right-to-left languages (`rtl()`).
- Event hooks (`afterEventCreated()`, `afterEventMoved()`, `afterEventUpdated()`, `afterEventDeleted()`), browser
  `warp-scheduler:*` events, print orientation and date range (`printOptions()`) and monthly or yearly rules on the
  nth weekday, a day of the month or chosen months in `RecurrenceInput`.
- CSV and XLSX export (`exportable()`), .ics import (`icsImport()`), read-only events from a callback
  (`eventsUsing()`) and adding, editing and deleting resources on the board (`resourceSchema()`).
- Events assigned to several resources through a `belongsToMany` relationship (`resourcesRelationship()`) and
  reminders sent as database notifications (`reminders()`, `warp-scheduler:send-reminders`).
- Multi-select with Ctrl/Cmd + click and group dragging, copy, cut and paste, in-place renaming, event search,
  drops from any page element (`onExternalDrop()`), moving events between schedulers (`onAdopt()`) and booking
  unscheduled events into the first free slot.
- Event subtitles, icons and badges, server-rendered tooltips, a quick-info popup, resource descriptions, colors and
  extra columns, day cell content, holidays and loading placeholders.
- Slot duration, snap, slot width and height, scroll time, working and hidden days, visible hours (hidden time
  is squeezed out of the timeline), timeline header rows and per-view overrides (`viewSettings()`).
- Gantt dependencies with lag, critical path, baselines and progress; successors move along when a task is
  pushed later.
- iCal export and per-user subscription feeds (`icalFeed()`), printing and PDF.
- Two-way sync with Google Calendar and Microsoft Outlook (`calendarSync()`), with queued jobs, webhooks and
  the `warp-scheduler:sync-calendars` and `warp-scheduler:renew-calendar-watches` commands.
- English and Japanese translations, screen reader support and browser find for the events on screen.
