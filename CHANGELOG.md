# Changelog

All notable changes to `qalainau/filament-warp-scheduler` will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.11.0] - 2026-11-13

### Added

- Drag an event from the timeline onto the unscheduled pool to take it off the timeline (`warpSchedulerUnschedule`).
  The same rules as moving decide whether it may go back, `EventSource::unscheduleUsing()` is used when set, and
  repeating events are refused with a reason. On by default whenever the pool is shown, for event sources with
  `unscheduleUsing()` or with nullable start and end columns; `returnToPool(false)` turns it off. The browser dispatches `warp-scheduler:unscheduled`.
- `boardWidth('280px')` sets the width of the board beside the timeline (`BoardLayout::Beside`). Only CSS lengths are
  taken.
- README: "Testing drags, the pool and the board" with feature tests for `warpSchedulerSchedule`,
  `warpSchedulerUnschedule` and `warpSchedulerMoveCard`, and a note on unique columns copied by `replicate()` when one
  occurrence of a recurring event is changed.
- Board swimlanes: `swimlanes('owner_id')` splits the board into a lane per value of a column, or per resource when
  the column is the resource column, with an Unassigned lane. Dragging a card to another lane writes that column (for
  a card on the timeline with resource lanes, it moves the event to that resource and checks the conflict rules).
  Lanes fold and stay folded in the browser. `canMoveCard()` and `afterCardMoved()` also receive `$fromLane` and
  `$toLane`. Off by default.
- Allowed moves between board columns: `transitions(['ready' => ['booked'], ...])`. While a card is dragged, the
  columns it may not move to are dimmed, and the server refuses such moves, including a card dropped on the timeline
  when its column may not move to the `scheduled()` column. Off by default.
- Drafts: with `batchSave()`, dragged and resized events stay drafts with a dashed outline until **Save all** saves
  them in one transaction (through the same checks as `moveEvents()`) or **Undo all** puts them back. If someone else
  changed one of the events in the meantime, nothing is saved and their change is not overwritten. Occurrences of
  repeating events and Alt copies are still saved at once. Off by default.
- A second time scale: `secondaryTimezone('America/New_York')` adds a second column of hours to the week and day views
  and a row under the hours of the timeline. `resourceTimezone('timezone')` shades each resource's night in its own
  time zone. Both only change what is drawn; saved times are unchanged. Off by default.
- Finding a free slot: `freeSlotSearch()` adds a panel to search up to three free times by length, resources (all
  together or any one), dates, hours and weekdays, with the same rules as dragging, and opens the create form with
  the chosen time and resources. With an Anthropic API key, a sentence ("a free hour next week in the afternoon for
  Tanaka and room A") can be turned into the conditions by Claude; the sentence and the names of the resources on
  screen are sent to Anthropic, events are not. Off by default; `freeSlotSearch(ai: false)` hides the sentence field.

## [1.10.4] - 2026-11-12

### Changed

- Now requires Filament 5.9 or later (was 5.0). Parts of the plugin did not work on older 5.x versions, which were never tested. The tests now also run on the lowest supported versions before each release.

### Fixed

- Calendar sync: pulling a change for an event whose link has no last-synced time no longer fails with "Method Carbon\\CarbonImmutable::minValue does not exist".

## [1.10.3] - 2026-11-10

### Security

- An event color read from a record (`EventSource::color()`) can no longer add CSS to the cards of the unscheduled
  pool and the board. A color that is not a color name or a CSS color (for example one containing `;`) is ignored.
- The README now says that the `$data` of `onExternalDrop()` and the `$event` of `onAdopt()` come from the browser and
  must be checked like request input.

## [1.10.2] - 2026-11-10

### Fixed

- Days off are now shaded the same way in every view, following `workingDays()` (or the working days of the calendar
  with `useWarpDatePicker()`). The month, week and day views shaded only holidays, so Saturday and Sunday looked like
  working days, and the timeline always shaded Saturday and Sunday, even when Saturday was a working day.

## [1.10.1] - 2026-11-10

### Fixed

- The timeline, the Gantt chart and the other scrolling views no longer flicker without end when the board's height is
  not a whole number of pixels, which is common with display scaling such as 125% or 150% on Windows. The browser
  rounds the height up (896.6px to 897px), so the content came out a fraction of a pixel too tall, and the scrollbars
  appeared and disappeared on every frame from the first render. Clicking the zoom again or resizing the window
  stopped it. The board now uses the exact height, and content that is less than a pixel taller than the box no longer
  makes it scroll. 1.9.1 fixed the same flicker for whole-pixel heights only. If you added `scrollbar-gutter: stable`
  to `.fi-warp-sch-scroller` as a workaround, you can remove it.

## [1.10.0] - 2026-11-09

### Fixed

- The menu that asks which events of a repeating event to move now says what it asks ("This event repeats. Change
  which events?") and has a Cancel item. Closing it with Escape or a click outside still cancels the move.
- Long items of the right-click menu wrap onto a second line instead of being cut off with "…".
- A wide event that runs past the right edge of the timeline shows its badge (such as "Urgent") at the visible right
  edge. The badge used to be drawn at the end of the event, out of view.
- Gantt bars that start before the visible part of the chart show their title from the left edge of the chart
  ("Requirements" instead of "ments").
- Utilization counts an all-day event as the available time of its day, not 24 hours. A whole-day job for someone who
  works 08:00–18:00 used to show 240% for the day; it now shows 100%. Without working hours, a day is still 24 hours.

### Added

- `createResourceIcon()`: the icon of the toolbar button that adds a resource. It is a plus by default (it was a
  person with a plus, which looked like "add a person" on boards of rooms or vans). Use `Heroicon::UserPlus` to keep
  the old icon on a board of people.
- `EventSource::reducesAvailability()`: the time of these events (vacation, sick leave) is taken off the available
  time for utilization, so a day off no longer counts as a free working day. Without it nothing changes.

### Documentation

- `useWarpDatePicker()` (added in 1.9.2) needs Warp DatePicker, which is not released yet; the README says so.

## [1.9.2] - 2026-11-07

### Fixed

- Renaming an event in place (double-click on its title) no longer ends when the board is scrolled until the event is
  out of view. The input used to be hidden with `visibility: hidden`, which made the browser take the focus away, so
  the half-typed title was saved and the input closed. It now becomes transparent and keeps the focus, and shows
  again on the event when it is scrolled back; Enter saves and Escape cancels as before.

### Added

- `useWarpDatePicker()`: with Warp DatePicker installed, the board uses its calendar — public holidays and company days off named and shaded, working weekdays and weekly business hours, availability from the business hours of each day (none on days off, without breaks, shorter before holidays, in the calendar's time zone, per resource with a closure) and half days off as named blocked time. Rows of the availability table still come first.

## [1.9.1] - 2026-11-07

### Fixed

- A save callback that returns `false` is refused like a `ChangeRejected` without a message: the change is rolled back,
  the user sees "The change was not saved.", the event jumps back (or the form stays open) and no hooks run. This
  covers `moveUsing()`, `createUsing()`, `updateUsing()`, `deleteUsing()`, `unscheduleUsing()` and `onExternalDrop()`.
  Before, `false` from `moveUsing()` and the others was taken as success, so the board showed a change that was not
  saved. `createUsing()` returning `false` was an error.
- A new record that `moveUsing()` (a copy, an occurrence of a recurring event), `createUsing()` or `onExternalDrop()`
  returns without saving it is refused the same way, and anything the callback saved is rolled back. Before, a copy
  that was never saved appeared on the board until the next reload.
- The timeline, the Gantt chart and the other scrolling views no longer flicker without end when the content is only a
  few pixels taller than the box, with scrollbars that take up space (Windows, or "Always show scrollbars" on macOS).
  The scrollbars used to appear and disappear on every frame. The scrolling area is now always exactly as tall as the
  content. If you added `scrollbar-gutter: stable` to `.fi-warp-sch-scroller` as a workaround, you can remove it.

### Documentation

- `$start` and `$end` in `moveUsing()`, `onExternalDrop()` and `onAdopt()` are in the scheduler's time zone, and
  Eloquent saves them without converting: the `BookingService` example in "Saving through your own code" now converts
  them to `app.timezone` (saving them as they are put a booking 9 hours late on a Tokyo board in a UTC app).
- The times of `warpSchedulerFetch` and `warpSchedulerMove` in tests are Unix timestamps in milliseconds, written in
  the scheduler's time zone (the README and the Boost skill said "wall-clock times written as UTC milliseconds").
- The Boost guidelines and skill: returning `false` or an unsaved record, the time zone of `$start` and `$end`, and
  hiding records that `deleteUsing()` only marks as cancelled with `modifyQueryUsing()`.

## [1.9.0] - 2026-11-06

### Added

- `->height('auto')` makes the resource timeline and the Gantt view as high as their rows, with the utilization band
  right under the last row, so a board of four vans no longer leaves an empty box and the page scrolls instead of the
  board. `->height('auto', max: '80vh')` scrolls inside the box past the max. `height` also goes in `zoomSettings()`
  and `viewSettings()` (`'auto'`, pixels or a CSS length), for example `'auto'` for the day and `640` for the week.
  The other views keep the default height. See [Height of the board](README.md#height-of-the-board).

### Changed

- Stacked layout (`eventLayout: 'stacked'`): while dragging, the label names the day and the place the event goes
  to (`Tue, Nov 3 · 2nd`, translated in the 10 languages) instead of a time range, and sits just under the day's
  header instead of covering it. The timed layout still shows the times on the header. There is no setting for the
  old label; the time the event gets is still passed to `moveUsing()` as before.

## [1.8.0] - 2026-11-06

### Added

- Instructions for AI coding assistants (Claude Code, Cursor, Codex) through [Laravel Boost](https://laravel.com/docs/boost):
  a guideline that Boost adds to `CLAUDE.md` and `AGENTS.md`, and the `warp-scheduler-development` skill for building
  boards and testing them. Select `qalainau/filament-warp-scheduler` in `php artisan boost:install`, or add it to
  `"packages"` in `boost.json`. See [AI coding assistants (Laravel Boost)](README.md#ai-coding-assistants-laravel-boost).

- `php artisan warp:assign-tenant "App\Models\Team" 1` gives the iCal subscription links and connected calendars that
  have no tenant (saved before the app had tenancy) to one tenant. `--dry-run` only counts them. Rows that already have
  a tenant are not touched. The same command also covers the tables of the other Warp plugins in the app.
- `document.querySelector('.fi-warp-sch-stage').warpScheduler.eventRects({ resource, date })` returns the events drawn
  now with their key, record key, resource, title, subtitle, start and end (the wall-clock time in the scheduler's time
  zone, `'2026-11-02T09:00'`) and their box on the screen in viewport pixels, for browser tests and screen recordings.
  `await warpScheduler.whenIdle()` waits until nothing is loading, saving or waiting to be drawn.
- README: "Testing with tenancy" (another tenant's IDs are treated as "not there", not as an exception, and how to
  check what a tenant sees) and "Browser tests and screen readers".

### Changed

- The screen reader list of the events on screen now reads the subtitle, the date and time and the resource after the
  title (before, the time and resource were only in an `aria-label`, which many screen readers skip for list items).
  So `innerText` and Playwright's `getByText()` find them too. Each item also has `data-key`, `data-record-id`,
  `data-resource`, `data-start` and `data-end`.

## [1.7.1] - 2026-11-02

### Fixed

- The work week view is called 「週（平日）」 in Japanese and 「근무 주간」 in Korean, so it is clear it is a week of
  weekdays (it was 「稼働日」 and 「근무일」, "working days").

## [1.7.0] - 2026-11-01

### Added

- A stacked layout for the week and month zooms of the resource timeline:
  `zoomSettings([TimelineZoom::Week->value => ['eventLayout' => EventLayout::Stacked]])` (or `viewSettings()`, or
  `->eventLayout(EventLayout::Stacked)`) puts each day's events of a resource under each other, in time order, as
  full-width blocks of that day, so a week of busy routes fits on one screen and every block is readable. Rows grow
  to the busiest day. Dragging a block to another day, resource or place in the day gives it the start of the event
  that is at that place now (after the last one: when the last one ends; on an empty day: the same time of day), and
  dropping it back on its own place saves nothing. Drops from the side list and the pool work the same way. Without
  it, nothing changes.

### Fixed

- A Week (or Day) zoom with only day header rows (`headerRows([HeaderRow::Day])`, also in `zoomSettings()`) still
  drew a line every 30 minutes when the days were wide. Each day is now one cell.

### Changed

- `slotWidth` (in `slotWidth()`, `viewSettings()` and `zoomSettings()`) is now the width of the finest tick, as the
  README says: one `slotDuration` in the Day and Week zooms (30 minutes unless you change it), one day in the Month
  and Quarter zooms, one week in the Year zoom. Until now it was the width of the lowest header tick: a Week zoom
  with `slotWidth => 100` was 100px per 6 hours, a Day zoom 100px per hour. **If you set `slotWidth`, your timeline
  is now wider.** To keep the old width with 30-minute slots, divide the value by 12 for the Week zoom and by 2 for
  the Day zoom (with `slotDuration => 60`: by 6 and 1), by 7 for the Quarter zoom and by about 4.3 for the Year zoom.
  The Month zoom does not change. Without `slotWidth`, nothing changes.

## [1.6.0] - 2026-10-31

### Added

- `zoomSettings([TimelineZoom::Day->value => ['slotWidth' => 96, 'rowHeight' => 64], ...])` changes the slot width,
  time steps, drag rounding, row height and the other view settings per timeline zoom, on top of `viewSettings()`.
  Switching the zoom switches the settings. Without it, nothing changes.
- `rowHeight` in `viewSettings()` and `zoomSettings()`, for a row height per view or per zoom.
- `viewLabels([...])` and `zoomLabels([...])` rename the view and zoom tabs from PHP, with a text, a translation key or
  a closure, without publishing the language files. Views and zooms that are not listed keep their translated names.

## [1.5.0] - 2026-10-31

### Added

- `timelineZooms([TimelineZoom::Day, TimelineZoom::Week])` chooses which zooms the timeline offers. With one zoom, the
  zoom buttons are hidden. A zoom that is not in the list (the initial zoom, or one set from JavaScript) falls back to
  the first one.
- `EventSource::showTime(false)` leaves the time out of the event boxes, so a box shows only its title and subtitle.
  The tooltip, quick-info popup, drag label and list views still show the time.
- `timeFormat('HH:mm')` writes every time on the scheduler in one format (Carbon `isoFormat` tokens: `H`, `HH`, `h`,
  `hh`, `m`, `mm`, `A`, `a`): event boxes, the hour ticks of the timeline and time grids, tooltips, the quick-info
  popup, the drag label, the list views and the board cards. Without it, times follow the locale as before
  (`en_GB` writes "7:20", not "07:20").

## [1.4.0] - 2026-10-31

### Added

- Translations in German, French, Spanish, Italian, Brazilian Portuguese, Korean, and simplified and traditional
  Chinese, next to English and Japanese.

## [1.3.1] - 2026-10-25

### Fixed

- After publishing the migrations (`php artisan vendor:publish --tag=filament-warp-scheduler-migrations`, as the README
  says for editing them), `php artisan migrate` stopped with "table already exists": Warp Scheduler ran its own copy as
  well. A published migration now replaces the bundled one; migrations you have not published still run as before.
- Printing any other page of the app (Ctrl+P, or another plugin's print) gave blank pages, and every page printed in
  landscape. The scheduler's print styles are loaded on every page; they now hide the rest of the page only while the
  scheduler itself is printing, and the paper orientation is set only for the scheduler's own print.

## [1.3.0] - 2026-10-25

### Upgrading

- Run `php artisan migrate`. The new `update_warp_scheduler_keys_to_strings` migration changes the key columns of the
  package's tables (`owner_id`, `tenant_id`, `schedulable_id`, `predecessor_id`, `successor_id`, `override_id`,
  `resource_id`) from numbers to strings. Existing rows are kept. If you query these tables yourself on PostgreSQL,
  compare them with string keys (`(string) $model->getKey()`).
- If your event model already has a tenant relationship (for example `Booking::team()`) but older rows have an empty
  `team_id`, fill it before updating (see the README), or turn the new filter off with
  `EventSource::make(Booking::class)->scopeToTenant(false)`.

### Added

- Full support for Filament's multi-tenancy (`Panel::tenant()`), documented in the new "Tenancy" section of the README.
  Events are filtered by the event model's own tenant relationship when it has one, and `EventSource::scopeToTenant()`
  changes the relationship name or turns this off for one source.
- Events and resources created in the scheduler (the create modal, Alt-drag copies, `.ics` imports, events pulled from
  Google or Outlook, and the "+" button for resources) are attached to the current tenant.

### Fixed

- In a panel with tenancy, the unscheduled pool and the board listed the cards of every tenant, and their cards could be
  booked or moved from another tenant. They now only show the current tenant's records.
- A Gantt dependency of another tenant could be removed by its number.
- The create and edit modals accepted a resource of another tenant sent from the browser.
- Opening the delete modal of an event or a resource that is not shown (for example another tenant's, by sending its
  number) failed with a server error. The modal now stays closed.
- A resource that belongs to several tenants (for example a user in two teams) showed the events of every tenant, and a
  change in one tenant was pushed to the calendars connected in the other. Events are now filtered by their own tenant
  relationship, and changes only go to calendars connected in the same tenant.
- An iCal feed whose tenant was deleted listed the events of every tenant; it now returns 404. A feed whose user left
  the tenant returns 403, and a calendar connection in the same situation stops syncing.
- A queued sync job, or a feed served by a long-running worker such as Octane, could keep the tenant of the previous
  job or request.
- Teams, users, resources and events with UUID or ULID keys could not be saved on MySQL, MariaDB and PostgreSQL
  (subscription links, connected calendars, recurring events, Gantt dependencies and baselines, reminders, availability
  and blocked time), because the package's tables stored these keys in number columns. They are string columns now.

## [1.2.1] - 2026-10-20

### Added

- A dashboard widget that uses `InteractsWithPageFilters` fetches its events again when the page filters change, and
  closures such as `resources()` can take `array $pageFilters`.
- Changing the dates of a Gantt task in the edit modal moves its successors along, as dragging it does (unless
  `dependencies(reschedule: false)`).

### Fixed

- Events are found in the right range when the app time zone (`app.timezone`), Filament's time zone or the
  scheduler's `timezone()` is not UTC. With the app in a zone ahead of UTC, the last hours of the visible range (in
  the day view, the whole evening) were missing; with the scheduler showing another zone than the app, the conflict
  rules looked at the wrong hours, so the create and edit modals could double-book a resource. Utilization, day
  cells, exports, `.ics` downloads, baselines and blocked time use the same corrected range.
- A save that a model event stops (`saving`, `creating` or `updating` returning `false`) is reported as refused
  instead of as saved: the event goes back, the modal stays open, and no hooks run and nothing is sent to synced
  calendars. An Alt-drag copy the model refused no longer leaves an event without a record on the timeline.
- `ChangeRejected` thrown from `unscheduleUsing()` while moving a board card is shown as a notification instead of
  failing the request.
- Board cards show the new resource, time and title after an event was dragged, resized, moved with the keyboard,
  pasted, renamed, dropped from outside or moved to another scheduler.
- Moving a board card to another column fetches the timeline again, so colours, badges or locks that depend on the
  column are up to date.
- The resource day and resource week views show the new columns after a filter or a resource edit changed the
  resources, instead of the old ones until the window was resized. Settings that a filter changes (such as hidden days
  from a closure) apply at once, and the utilization row no longer shows the numbers of the previous groups while the
  new ones load.
- Tooltips without a `tooltip()` view show the new title and time after an event changed.
- An event moved twice quickly (two drags, or the arrow keys) goes back to its last saved place when the saves fail,
  instead of staying at the first, unsaved, place.
- Pasting cut events that fails no longer brings back an event deleted since it was cut, or puts it back where it was
  before it was moved.
- After an Alt-drag copy, the copy is selected, so the highlight and the arrow keys act on the same event. The same
  holds when an event shared by several resources is moved to another row.
- After the events are fetched again (an action, `warp-scheduler-refresh`), dependency arrows and blocked time that
  were removed elsewhere disappear.
- An `onExternalDrop()` callback that does not return the record it created no longer leaves the drop empty: the
  scheduler fetches the events again.
- The event menu and the quick info no longer open late, after the user scrolled, clicked elsewhere or changed the
  range while they loaded.
- The rename box stays on its event while the scheduler scrolls.
- When several selected events are moved together and one of them is refused, the hooks and synced calendars are no
  longer told about the moves that were rolled back.
- Day cells are fetched again after an action or `warp-scheduler-refresh`, and after a filter change.
- The migrations run on MySQL. Three indexes (on `warp_scheduler_dependencies`, `warp_scheduler_occurrence_exceptions`
  and `warp_scheduler_external_links`) had default names longer than MySQL's 64 characters, so the migration stopped
  with an error. If you published the migrations before and they failed on MySQL, publish them again.
- Searching events by title ignores upper and lower case on PostgreSQL too (it matched only the exact case there).
- Sending reminders no longer fails on PostgreSQL when it runs inside a database transaction and a reminder was
  already sent.

## [1.2.0] - 2026-10-20

### Added

- `createUsing()`, `updateUsing()` and `deleteUsing()` on an event source replace how records are created, changed
  and deleted, so every change can go through your own services. `updateUsing()` also covers renaming in place,
  moving cards on the board and taking records off the timeline; `createUsing()` also covers `.ics` imports.
- Throw `ChangeRejected` from a save callback, `onExternalDrop()` or `onAdopt()` to refuse a change: the database
  transaction is rolled back, the message is shown, the event goes back and a modal stays open.
- `moveUsing()` receives `$copy` and `$original` for Alt-drag copies and pasted events, and can return the record it
  created.
- `dependencies(reschedule: false)` keeps dependency arrows, linking and the critical path without moving successors,
  so dates can be calculated by your application.

### Fixed

- Tooltips show the new title and time after an event was renamed, moved or edited, instead of the ones loaded
  before the change.

### Changed

- Renaming in place, drops from outside and adoptions from another scheduler run in a database transaction, and
  deleting an event removes it from synced calendars only after the record was deleted.

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
