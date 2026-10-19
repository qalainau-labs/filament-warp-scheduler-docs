# Warp Scheduler

![A resource timeline of 5,000 people with utilization and a pool of requests](https://raw.githubusercontent.com/qalainau-labs/filament-warp-scheduler-docs/main/art/timeline.png)

**A resource scheduler for Filament that stays fast with thousands of rows: the timeline, calendars and Gantt chart are drawn on a `<canvas>`, while forms, actions, filters, notifications and permissions are Filament's own.**

<a href="https://filament-warp-scheduler.webllsystem.com/"><img src="https://raw.githubusercontent.com/qalainau-labs/filament-warp-scheduler-docs/main/art/live-demo.png" alt="Try the live demo: filament-warp-scheduler.webllsystem.com" width="592"></a>

Scroll through 5,000 people, drag bookings, cover shifts, dispatch jobs and plan a project at **[filament-warp-scheduler.webllsystem.com](https://filament-warp-scheduler.webllsystem.com/)**. No sign-up needed; the data is reset every hour.

Calendar plugins for Filament wrap a JavaScript calendar in a widget. That works for a few dozen resources, but a planning board with hundreds or thousands of people, rooms or machines needs more: a resource timeline that does not build a DOM node per cell, recurring events that are expanded on the server, rules that stop double bookings, and a way to see how busy each resource is.

Warp Scheduler keeps your Eloquent models. Point it at a resource model and one or more event models, and you get a resource timeline, resource day and week views, week, work week, day, N-day, month, month agenda, year and list views and a Gantt chart, with drag and drop that saves through your policies.

```php
use Qalainau\FilamentWarpScheduler\EventSource;
use Qalainau\FilamentWarpScheduler\Pages\SchedulerPage;
use Qalainau\FilamentWarpScheduler\Scheduler;

class StaffSchedule extends SchedulerPage
{
    public function scheduler(Scheduler $scheduler): Scheduler
    {
        return $scheduler
            ->resources(Staff::class)
            ->groupResourcesBy(['region', 'department'])
            ->events([
                EventSource::make(Booking::class)
                    ->resource('staff_id')
                    ->start('starts_at')
                    ->end('ends_at')
                    ->title('title'),
            ]);
    }
}
```

## Why

Measured on the demo app with 5,148 timeline rows (5,000 staff in three levels of groups) and 200,000 bookings, in Chrome:

| | |
| --- | --- |
| Page HTML, including every resource | 44 KB |
| First events on screen after navigation | about 0.2 s |
| Drawing a frame while scrolling (any zoom) | 3 ms or less |
| Utilization for a month of 200,000 bookings | about 0.4 s |

Only the rows and the time range on screen are drawn, and events are loaded per block of 100 resources as you scroll.

## Features

![A shift board with avatars and a list of people to drag onto shifts](https://raw.githubusercontent.com/qalainau-labs/filament-warp-scheduler-docs/main/art/shift-board.png)

- **Views**: resource timeline (day, week, month, quarter and year zoom), resource day and week views (by resource or by date), week, work week, day, N-day, month, month agenda, year and list views, and a Gantt chart. Each view can have its own slot size, visible hours and days.
- **Built for large boards**: canvas rendering with vertical and horizontal virtualization, collapsible nested resource groups, lazy loading of events and placeholders while they load.
- **Drag and drop**: move events in time and across resources, copy with Alt, resize from either edge, select several with Ctrl/Cmd+click and drag them together, copy and paste, rename in place, and create events by selecting a range. Changes appear at once and are saved on the server, which checks read-only mode, locked events and your model policies. Refused changes roll back with a notification.
- **Board (kanban)**: show the same records in columns by stage above or beside the timeline. Drag a card to another column to change its stage, onto a resource to schedule it, or drag an event from the timeline back to the board to take it off the timeline. WIP limits, the days each card has waited, and columns built from your own enum.
- **Drop from anywhere**: drop unscheduled records from a pool, rows from a Filament table or any element on the page, people from a side list onto shifts, or events from another scheduler. Book a request into the first free slot with one click.
- **Your content**: subtitles, icons, badges, avatars and solid, outline or dashed styles on events; server-rendered tooltips and a quick-info popup; resource avatars, descriptions and extra columns; holidays, prices per day and labelled blocked time; shifts as header rows.
- **Filament actions**: each event source with a schema gets create, edit and delete actions in Filament modals. Add your own actions to the event context menu with `eventActions()`, and add, edit or delete resources on the board.
- **Recurring events**: RFC 5545 rules expanded on the server for the visible range, with the wall-clock time kept across daylight saving changes. Edit or delete one occurrence, this and following, or the whole series. `RecurrenceInput` adds a repeat section to your form, including the nth weekday of a month and chosen months.
- **Availability and conflicts**: business hours, per-resource and date exceptions such as holidays, blocked time, and capacity. Rules reject or only warn, are checked on the server before saving, and are previewed in red or amber while dragging.
- **Utilization**: booked time over available time per resource, per group and per hour, day, week or month, shown as meters, group heatmaps and a histogram.
- **Gantt**: progress, baselines, dependency arrows (finish-to-start, start-to-start, finish-to-finish and start-to-finish, with lag), the critical path, and successors that move along when a task is pushed later. Link tasks by dragging between bars.
- **Data**: events shared by several resources (`belongsToMany`), read-only events from any callback, search, reminders as database notifications, CSV and XLSX export and `.ics` import.
- **iCal**: download the visible range as `.ics`, or subscribe from Google Calendar, Apple Calendar or Outlook with a personal link that keeps the user's filters and tenant.
- **Two-way calendar sync**: connect a Google or Microsoft calendar to a resource. Changes on either side are synced through queued jobs, push notifications and delta queries.
- **Print and PDF**: print the current view or a date range, landscape or portrait, in the light palette, page by page, or save it as a PDF from the print dialog.
- **Filament integration**: a page, a dashboard widget, or a trait for any Livewire component. Filter form built from Filament fields, tenancy, dark mode, panel colors, translations (English and Japanese), SPA mode, a compact layout on phones and right-to-left languages. Hooks after every change, and browser events.
- **Accessibility**: keyboard navigation and editing, a screen reader list of the events on screen that also makes them findable with the browser's find, and live announcements of saves.

| Calendar views | Gantt chart |
| --- | --- |
| ![The week view of twelve meeting rooms with prices per day](https://raw.githubusercontent.com/qalainau-labs/filament-warp-scheduler-docs/main/art/calendar-week.png) | ![A Gantt chart with dependencies, progress and baselines](https://raw.githubusercontent.com/qalainau-labs/filament-warp-scheduler-docs/main/art/gantt.png) |

## Requirements

- PHP 8.3+
- Laravel 12 or 13
- Filament 5.x (Livewire 4)
- A queue worker if you use calendar sync

## Installation

> **Warp Scheduler is a commercial plugin.** A license is required. After purchase you receive credentials for the private Composer repository.

Add the repository and authenticate:

```bash
composer config repositories.qalainau composer https://qalainau.privato.pub/composer
composer config --auth http-basic.qalainau.privato.pub "your-email" "your-license-key"
```

Install the package, run the migrations and publish Filament's assets:

```bash
composer require qalainau/filament-warp-scheduler
php artisan migrate
php artisan filament:assets
```

The migrations create the plugin's own tables (`warp_scheduler_*`) for recurrence rules, availability, blocked time, dependencies, baselines, iCal links and calendar connections. Your event and resource tables are not changed. To edit the migrations, publish them with `php artisan vendor:publish --tag=filament-warp-scheduler-migrations`.

> Run `php artisan filament:assets` again after every update of the package.

## Usage

### Where to put a scheduler

Extend `SchedulerPage` for a full page or `SchedulerWidget` for a dashboard widget, and implement `scheduler()`:

```php
use Qalainau\FilamentWarpScheduler\Widgets\SchedulerWidget;

class TodaySchedule extends SchedulerWidget
{
    public function scheduler(Scheduler $scheduler): Scheduler
    {
        return $scheduler->height(360)->initialZoom(TimelineZoom::Day)->resources(...)->events([...]);
    }
}
```

If you give the widget a view of its own, keep `<x-filament-actions::modals />` in it: a widget renders its own
create, edit and delete modals, unlike a page.

In any other Livewire component (for example a resource's relation page), implement `HasScheduler`, `HasSchemas` and `HasActions`, use `InteractsWithScheduler`, `InteractsWithSchemas` and `InteractsWithActions`, and render `{{ $this->getWarpScheduler() }}`. If the component is not a Filament page, also render `<x-filament-actions::modals />` so that the create, edit and delete modals can open. The scheduler's properties and methods are prefixed with `warpScheduler`, so they do not clash with `$record` or your own properties.

### Resources

```php
$scheduler
    ->resources(fn (array $filters) => Staff::query()->where('active', true))   // a model class, a query or a collection
    ->resourceTitle('name')                                                     // or fn (Staff $record): string
    ->resourceLabel('Staff')                                                    // the column heading
    ->groupResourcesBy(['region', 'site'])                                      // nested groups, attributes or closures
    ->resourceColumnWidth(240)
    ->rowHeight(36);
```

A resource model can implement `SchedulerResource` (`getSchedulerTitle()` and `getSchedulerGroups()`) instead.

### Events

```php
EventSource::make(Booking::class)
    ->label('Booking')
    ->resource('staff_id')
    ->start('starts_at')
    ->end('ends_at')               // the end is exclusive
    ->title('title')               // or fn (Booking $record): string
    ->allDay('all_day')            // a boolean column, or true for date-only models
    ->color(fn (Booking $record): string => $record->status === 'tentative' ? 'warning' : 'primary')
    ->modifyQueryUsing(fn (Builder $query, array $filters) => $query->where('status', '!=', 'cancelled'))
    ->schema([...])                // the create and edit form
    ->locked(fn (Booking $record): bool => $record->status === 'done')
    ->editable(true)
    ->resizable(true);
```

An event model can implement `Schedulable` (`getSchedulerColumns()`) to declare its columns once. Pass several sources to `events()` to show several models on the same board.

All-day events are floating dates, like iCalendar: they stay on the same day in every time zone.

Event content, tooltips and the quick-info popup:

```php
EventSource::make(Booking::class)
    ->subtitle('status')                                   // drawn after (or under) the title
    ->icon(fn (Booking $record) => $record->urgent ? Heroicon::OutlinedBolt : null)
    ->badge(fn (Booking $record): ?string => $record->seats)
    ->tooltip(fn (Booking $record) => view('bookings.tooltip', ['booking' => $record]))  // loaded on hover
    ->details(fn (Booking $record): string => $record->notes); // shown in the quick-info popup

$scheduler->quickInfo();  // clicking an event shows its details and actions instead of opening the modal
```

Strings are escaped; return an `Htmlable` (a view or `HtmlString`) for markup.

Avatars, solid or dashed events and shifts:

```php
$scheduler
    ->resourceImage(fn (Staff $record): ?string => $record->avatar_url)  // rows become 52px tall by default
    ->shifts(['Morning shift' => ['06:00', '14:00'], 'Evening shift' => ['14:00', '22:00']]);

EventSource::make(Shift::class)
    ->image(fn (Shift $record): ?string => $record->employee->avatar_url)
    ->variant(fn (Shift $record): string => $record->on_leave ? 'dashed' : 'solid');  // soft, solid, outline, dashed
```

People without an image get their initials. Blocked time shows its reason. Images from other sites must allow
CORS to appear in printouts.

A side list of things to drag onto the board (people who can take a shift, for example), with group filters:

```php
$scheduler
    ->sideList(fn (): array => Employee::available()->get()->map(fn (Employee $employee): array => [
        'id' => $employee->id, 'title' => $employee->name, 'subtitle' => $employee->role,
        'image' => $employee->avatar_url, 'group' => $employee->team,
    ])->all(), label: 'Available')
    ->onExternalDrop(function (array $data, CarbonImmutable $start, ?Model $resource, ?Model $event) {
        // dropped on an event: replace who covers it; dropped on an empty slot: create one
    });
```

Resource column, day cells and holidays:

```php
$scheduler
    ->resourceDescription('department')
    ->resourceColor(fn (Staff $record): string => $record->team_color)
    ->resourceColumns(['capacity' => 'Seats', 'floor' => ['label' => 'Floor', 'width' => 56]])
    ->dayCellContent(fn (CarbonImmutable $date): ?string => $prices->for($date))
    ->holidays(fn (CarbonImmutable $from, CarbonImmutable $to): array => Holiday::between($from, $to)->pluck('name', 'date')->all());
```

Holidays are shaded like weekends and named in the month, week and year views.

Events shared by several resources (a meeting with many rooms) use a `belongsToMany` relationship instead of a column.
The event is drawn on every resource; dragging one row moves that assignment, and the time of all of them:

```php
EventSource::make(Meeting::class)->resourcesRelationship('rooms');
```

Use a multiple `Select::make('rooms')->relationship('rooms', 'name')` in the form. Recurring events need a resource column.

Reminders send a Filament database notification before an event starts:

```php
EventSource::make(Booking::class)
    ->reminders([15, 60 * 24])                              // minutes before
    ->remindTo(fn (Booking $record) => $record->staff->user)
    ->remindUsing(fn (Booking $record, int $minutesBefore) => ...); // or send them yourself
```

List the scheduler components in `filament-warp-scheduler.reminders.components` and run
`Schedule::command('warp-scheduler:send-reminders')->everyMinute()`. Each reminder is sent once; moving an event
sends it again for the new time.

### Import, export and other data

```php
$scheduler
    ->exportable()                 // CSV and XLSX of the visible range (or ->exportable(['csv']))
    ->icsImport()                  // upload an .ics file into a resource
    ->eventsUsing(fn (CarbonImmutable $start, CarbonImmutable $end, array $resources): array => $api->events($start, $end))
    ->resourceSchema([TextInput::make('name')->required()]);  // add, edit and delete resources on the board
```

`eventsUsing()` returns arrays with `id`, `resource`, `start`, `end`, `title` and optionally `allDay`, `color` and
`subtitle`; those events are read-only. With `resourceSchema()`, click a resource name to edit it, right-click it to
delete it, and use the toolbar button to add one. Model policies apply.

### Views and time

```php
$scheduler
    ->views([SchedulerView::ResourceTimeline, SchedulerView::Week, SchedulerView::Month])  // default: all
    ->initialView(SchedulerView::ResourceTimeline)
    ->initialZoom(TimelineZoom::Week)
    ->initialDate('2026-10-01')
    ->timezone('Asia/Tokyo')        // default: FilamentTimezone
    ->firstDayOfWeek(1)
    ->height('70vh')
    ->calendarResourceLimit(300);   // week, day, month and list show all events together
```

Slots, visible days and hours, and header rows:

```php
$scheduler
    ->slotDuration(15)              // time grid slots, minutes (default 30)
    ->snapDuration(5)               // drag, resize and selection rounding, minutes
    ->slotHeight(20)                // time grid slot height, px
    ->slotWidth(64)                 // minimum width of the finest timeline tick, px
    ->scrollTime('08:00')           // where to scroll when today is not in range
    ->workingDays([1, 2, 3, 4, 5])  // default: days with business hours, else Monday to Friday
    ->hideNonWorkingDays()          // or ->hiddenDays([0, 6])
    ->visibleHours('07:00', '21:00') // or ->hideNonWorkingHours() to use the business hours
    ->headerRows([HeaderRow::Month, HeaderRow::Week, HeaderRow::Day])
    ->viewSettings([
        SchedulerView::ResourceTimeline->value => ['hiddenDays' => [0, 6], 'visibleHours' => ['08:00', '20:00']],
        SchedulerView::Week->value => ['slotDuration' => 10],
    ]);
```

More views are available on demand (the default set is `SchedulerView::defaults()`):

| View | Shows |
|---|---|
| `ResourceWeek` / `ResourceWeekByDate` | A week of days per resource, or every resource per day |
| `WorkWeek` | The working days of the week (`workingDays()`) |
| `Days` | Any number of days (`dayCount(3)`) |
| `MonthAgenda` | A compact month with the selected day's events listed below |
| `Year` | Twelve mini months shaded by how busy each day is; click a day to open it |

Hidden days and hours are left out of the timeline (hour zooms squeeze the night away), the week and month
grids and the time grids. Header rows finer than the zoom are skipped.

### Dependent fields in the event form

The create and edit modals are ordinary Filament forms, so dependent ("cascading") fields work as usual:

```php
->schema([
    Select::make('region')->options($regions)->live()->dehydrated(false)
        ->afterStateUpdated(fn (Set $set) => $set('staff_id', null)),
    Select::make('staff_id')->relationship('staff', 'name', fn (Builder $query, Get $get) => $query
        ->when($get('region'), fn (Builder $query, string $region) => $query->where('region', $region))),
    // ...
])
```

On phones (screens under 640px wide) the scheduler switches to `mobileView()` (the list by default; `null` keeps the
current view) and narrows the resource column. When the scheduler itself is narrower than 640px, for example a
widget in half a dashboard, it keeps its view but turns the view tabs into a select and hides the unscheduled pool. Right-to-left pages (`dir="rtl"`)
are mirrored automatically; `rtl()` forces it.

### Filters

```php
$scheduler->filtersForm([
    Select::make('department')->options([...]),
    TextInput::make('search'),
]);
```

The fields update the scheduler as soon as they change (text inputs wait for typing to stop). Their state is passed as `$filters` to `resources()` and `modifyQueryUsing()`.

### Permissions, tenancy and read-only mode

- If a model has a policy, `create`, `update` and `delete` are checked before any change. Without a policy, changes are allowed, as in Filament resources.
- In a panel with tenancy, resources are scoped to the current tenant (`scopeToTenant(false)` turns this off). Events are only returned for the resources shown, so they are scoped too.
- `readOnly()` disables dragging, creating and deleting; clicking an event opens a read-only view.

### Actions

```php
$scheduler->eventActions([
    Action::make('markAsDone')
        ->visible(fn (?Model $record) => $record?->status !== 'done')
        ->action(fn (Booking $record) => $record->update(['status' => 'done'])),
]);
```

Custom actions appear in the event's context menu, and their record is the clicked event.

### Search, drops from outside and other schedulers

```php
$scheduler
    ->searchable()  // a search box in the toolbar; jumps to the event
    ->onExternalDrop(fn (array $data, CarbonImmutable $start, ?Model $resource) => Booking::create([...]))
    ->onAdopt(fn (array $event, CarbonImmutable $start, CarbonImmutable $end, ?Model $resource): bool => ...);
```

Any element can be dragged onto a scheduler that has `onExternalDrop()`:

```html
<tr draggable="true" data-warp-scheduler-draggable='{"id": 5}' data-warp-scheduler-title="Repair" data-warp-scheduler-duration="90">
```

`data-warp-scheduler-draggable` holds a JSON object (passed to `onExternalDrop()` as `$data`) or any other value, such
as an ID (passed as `['value' => ...]`). In Blade, escape the JSON so its quotes cannot end the attribute: `{{ json_encode($data) }}`.

Filament 5 tables have no row attributes, so add the attributes to a column with `extraAttributes()`; the cell becomes
the thing to drag. Filament does not HTML-escape attribute values (it only turns `"` into `\"`), so escape the JSON
yourself with `e()`, or the attribute ends at the first quote and the drop does nothing:

```php
TextColumn::make('title')
    ->extraAttributes(fn (Job $record): array => [
        'draggable' => 'true',
        'data-warp-scheduler-draggable' => e(json_encode(['id' => $record->id])),
        'data-warp-scheduler-title' => e($record->title),
        'data-warp-scheduler-duration' => $record->minutes,
    ])
```

Dragging an event onto another scheduler on the same page calls that scheduler's `onAdopt()`; both reload when it
returns `true`.

### Unscheduled pool

```php
$scheduler->unscheduledPool(label: 'Requests');

EventSource::make(Booking::class)->defaultDuration(fn (Booking $record) => $record->duration_minutes);
```

Records whose start column is empty are listed beside the timeline and can be dropped on a resource.

Each pool item has a button that books it into the first free slot of the resources on screen within two weeks,
respecting business hours, blocked time and the conflict rules.

### Board (kanban)

![Work orders in columns by stage above a timeline of technicians](https://raw.githubusercontent.com/qalainau-labs/filament-warp-scheduler-docs/main/art/board.png)

Give an event source a stage column and list the columns:

```php
use Qalainau\FilamentWarpScheduler\Board\BoardColumn;

$scheduler
    ->events([
        EventSource::make(WorkOrder::class)
            ->resource('technician_id')
            ->start('starts_at')
            ->end('ends_at')
            ->stage('stage'),
    ])
    ->board([
        BoardColumn::make('received'),
        BoardColumn::make('parts')->label('Waiting for parts')->limit(4),
        BoardColumn::make('ready')->label('Ready to book')->limit(5),
        BoardColumn::make('booked')->scheduled()->recent(8),
        BoardColumn::make('done')->color('success')->keepsSchedule()->recent(8),
    ])
    ->agingFrom('created_at', warning: 3, danger: 7);
```

The board replaces the unscheduled pool. There are three kinds of column:

| Column | Holds | Moving a card into it |
| --- | --- | --- |
| Default | Records without a start | Takes a scheduled record off the timeline (its start and end are cleared) |
| `scheduled()` | Records on the timeline (one column) | Drop the card on a resource instead; it then moves here by itself |
| `keepsSchedule()` | Done, invoiced and the like | Changes the stage and keeps the record on the timeline |

Cards in the default columns can be dragged onto the timeline, or booked into the first free slot with their
sparkles button. An event dragged from the timeline onto a column moves there, so dropping a booked job on
"Ready to book" takes it off the timeline. Clicking a card opens the same modal as clicking the event.

- `limit(4)` turns the column heading red above four cards; `limit(4, strict: true)` refuses the fifth.
- `recent(8)` shows only the first eight cards of a column that keeps growing, nearest first, and counts the rest.
- `agingFrom()` shows how many days each card before `scheduled()` has waited.
- `boardLayout(BoardLayout::Beside)` sets the default layout. Users can switch between the board above or beside the
  timeline, the board only and the timeline only; the choice is kept in the browser.
- `cardView('work-orders.card')` renders each card with your own view, which receives `$record`, `$card` and
  `$column`.
- `canMoveCard(fn (Model $record, string $from, string $to): bool => ...)` decides which moves are allowed; your
  model policy's `update` is checked as well.
- `afterCardMoved(fn (Model $record, string $from, string $to) => ...)` runs after each move.
- `EventSource::unscheduleUsing(fn (Model $record) => ...)` replaces how a record is taken off the timeline. By
  default the start and end are cleared and the resource is kept.

Columns can come from an enum. Labels and colors are read from `HasLabel` and `HasColor`, and an enum that implements
`BoardStage` configures its own columns:

```php
enum WorkOrderStage: string implements BoardStage, HasLabel
{
    case Ready = 'ready';
    case Booked = 'booked';
    case Done = 'done';

    public function configureBoardColumn(BoardColumn $column): void
    {
        match ($this) {
            self::Ready => $column->limit(5),
            self::Booked => $column->scheduled(),
            self::Done => $column->keepsSchedule(),
        };
    }

    // getLabel() ...
}

$scheduler->board(WorkOrderStage::class);
```

### Recurring events

```php
EventSource::make(Booking::class)
    ->recurring()
    ->schema([
        // ...
        RecurrenceInput::make(),
    ]);
```

Rules are stored in `warp_scheduler_recurrences`. Changing an occurrence asks whether to apply it to that occurrence, to it and the following ones, or to the whole series.

### Availability and conflicts

```php
$scheduler
    ->businessHours(['mon' => '09:00-18:00', 'tue' => ['09:00-12:00', '13:00-18:00']])
    ->conflictRules([
        NoOverlap::make()->capacity('capacity'),          // a number, a column or a closure
        WithinAvailability::make()->warn(),
        NoBlockedTime::make(),
        CustomRule::using(fn (Placement $placement): ?string => $placement->start->isWeekend() ? 'No weekend bookings.' : null),
    ]);
```

Weekly hours and date exceptions per resource (or for all resources, such as holidays) can also be stored in `warp_scheduler_availabilities`, and blocked time in `warp_scheduler_blocks`. To compute availability yourself, for example with `spatie/opening-hours` or `laraveljutsu/zap`, return intervals from `availabilityUsing(fn (Model $resource, CarbonImmutable $from, CarbonImmutable $to) => [[$start, $end], ...])`.

### Utilization

```php
$scheduler->utilization();

EventSource::make(Absence::class)->countsTowardsUtilization(false);
```

### Hooks

```php
$scheduler
    ->afterEventCreated(fn (Model $record) => activity()->log("Created {$record->title}"))
    ->afterEventMoved(fn (Model $record, EventSource $source) => ...)
    ->afterEventUpdated(fn (Model $record) => ...)
    ->afterEventDeleted(fn (Model $record) => ...);
```

They run after drags, forms, renames, imports and drops. In the browser, the scheduler element dispatches
`warp-scheduler:navigated`, `warp-scheduler:event-clicked`, `warp-scheduler:slot-selected`, `warp-scheduler:moved`,
`warp-scheduler:copied`, `warp-scheduler:renamed`, `warp-scheduler:dropped`, `warp-scheduler:adopted`,
`warp-scheduler:scheduled` and `warp-scheduler:card-moved` with the details in `event.detail`:

```blade
<div x-on:warp-scheduler:moved="console.log($event.detail)">{{ $this->getWarpScheduler() }}</div>
```

To make every scheduler on the page fetch its events and redraw its board after something else changed them,
dispatch `warp-scheduler-refresh`, from Livewire (`$this->dispatch('warp-scheduler-refresh')`) or from the browser
(`window.dispatchEvent(new CustomEvent('warp-scheduler-refresh'))`).

### Gantt

```php
$scheduler
    ->views([SchedulerView::Gantt, SchedulerView::ResourceTimeline])
    ->dependencies()
    ->baselines();

EventSource::make(Task::class)->resource('project_id')->progress('progress');
```

Each resource is a project and its events are the tasks.

### iCal and printing

```php
$scheduler->icalFeed();
```

The toolbar gets a download button and a subscription link. Feeds are served from `/warp-scheduler/feeds/{token}.ics` with the owner's permissions, filters and tenant. The print button is always available.

`printOptions()` asks for the paper orientation and, in the timeline, Gantt and list views, a date range before
printing.

### Calendar sync

```php
$scheduler->calendarSync();
```

Create an OAuth app with Google and/or Microsoft, register these redirect URIs and set the credentials in `.env`:

```
{APP_URL}/warp-scheduler/oauth/google/callback
{APP_URL}/warp-scheduler/oauth/microsoft/callback
```

```dotenv
WARP_SCHEDULER_GOOGLE_CLIENT_ID=
WARP_SCHEDULER_GOOGLE_CLIENT_SECRET=
WARP_SCHEDULER_MICROSOFT_CLIENT_ID=
WARP_SCHEDULER_MICROSOFT_CLIENT_SECRET=
WARP_SCHEDULER_MICROSOFT_TENANT=common
```

Run a queue worker and schedule the fallback sync and the renewal of push notifications:

```php
Schedule::command('warp-scheduler:sync-calendars')->everyFifteenMinutes();
Schedule::command('warp-scheduler:renew-calendar-watches')->daily();
```

Push notifications need a public `https` URL. Without one, the scheduled sync keeps calendars up to date.

## Keyboard

| Key | Action |
| --- | --- |
| Tab / Shift+Tab | Select the next or previous event on screen |
| Enter / Space | Open the selected event |
| Arrow keys | Move the event (time, day or resource, depending on the view) |
| Shift + arrow keys | Change the length |
| Delete / Backspace | Delete the event |
| Escape | Cancel a drag, close a menu or clear the selection |
| Ctrl/Cmd + click | Add an event to the selection; dragging one selected event moves them all |
| Ctrl/Cmd + C, X, V | Copy or cut the selected events and paste them at the pointer |
| F2 / double-click | Rename the event in place |

On touch screens, press and hold an event to drag it.

## Limitations

- Week, day, month, year and list views show every event together and are limited to `calendarResourceLimit()` resources. Use the filters for larger boards.
- Events shared by several resources (`resourcesRelationship()`) cannot be recurring, and reminders are not sent for the occurrences of recurring events.
- Events from `eventsUsing()` are read-only. A date range for printing works in the timeline, Gantt and list views.
- Images from other sites appear in printouts only when those sites allow CORS.
- Recurring events from Google or Outlook are imported as single occurrences. Recurring series created in the scheduler are exported with their rule.
- When both sides of a sync changed the same event, the more recent change wins.
- The drag preview checks conflicts against the events already loaded. The server always checks again before saving.
- Printing renders up to 40 pages of the current view.

## Troubleshooting

**Nothing is drawn.** Run `php artisan filament:assets` after installing or updating, and check that the page shows no JavaScript errors.

**No events appear.** Check the column names in `EventSource` and that the events belong to resources returned by `resources()`.

**Calendar changes do not arrive.** Make sure a queue worker is running and `warp-scheduler:sync-calendars` is scheduled. The last error of each connection is stored in `warp_scheduler_calendar_connections.last_error`.

## Changelog

See [CHANGELOG.md](CHANGELOG.md).

## Support

Ask a question or report a bug at https://warpui.com/support?plugin=scheduler. Do not post your license key in your question.

## License

Warp Scheduler is commercial software. See [LICENSE.md](LICENSE.md) for the license terms.
