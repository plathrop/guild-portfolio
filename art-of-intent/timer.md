## What It Is

We are building a timer website for multiple users. It is a green
field project and we will be building it from scratch. It will be
deployed on fly.io with persistent data in their managed Postgres.

It will be written in Typescript. I am unfamiliar with the current
state of web development or the widely used frameworks for this kind
of application. You should walk me through choosing one.

## Who It's For

This will be a multi-user website with identity, logins, and per-user
state.

## What It Should Do

  - allow login via the three most common oauth identity providers
  - allow users to create named timers like "Check Laundry" or
    "Meeting in 30 min" using:
    - a free text input for the name (reasonably limited, perhaps
    64-128 characters?)
    - an input for the numeric part of the duration, in DD:HH:MM:SS
      format which can either be typed, increased/decreased with an
      up/down arrow interface element, or increased/decreased with the
      mouse wheel
  - allow users to set an action when the timer expires, choosing
    between browser-based notification, playing a sound, or visual
    alert. If the site is closed, notifications will not fire
  - allow users to sort timers by name, total duration, or which will
    expire soonest using a UI toggle. Sorting should be persistent
    across logins
  - allow users to pause and resume timers. We will iterate together
    on how this presents in the UI
  - allow users to delete timers, with a confirmation step. No undo
    support necessary
  - expired timers should not auto-delete, users must delete them
  - be usable on a mobile phone - just in the phone's browser. Again,
    I'm unfamiliar with modern web design so we will iterate on this
    together
  - Limit the user to 25 active timers (and up to 25 expired timers
    awaiting deletion)

## What It Should Look Like

  - clean, minimalistic design in cool colors like blue, purple,
    green. Use these colors as a starting point:
    - Midnight Blue: #1C2D42
    - Emerald Green: #1B4D3E
    - Royal Purple: #442B72
    - Cyan Accent: #4FD3C4
    - Light Gray: #E9ECEF
  - default to dark/light mode based on the browser's mode, but allow
    the user to over-ride
  - use a font that clearly distinguishes between letters and numbers.
    Choose a free font and we can iterate from there
  - show a numeric countdown by default, but allow the user to change
    to a progress bar style where the bar progresses from filled to
    empty unless it is paused
  - the progress bar style should use a flat design with a gradient,
    probably using the Emerald Green as a base
  - numeric vs. progress bar style should be a site-wide toggle and
    stored as a user preference

## What It Should Not Do

  - store passwords
  - have any "social" features like sharing
  - store any unnecessary user data; only identity, timers, and
    the user's ui preferences
  - support push notifications
  - real-time cross-device sync; if the user opens the site on another
    device, loading state from the server is sufficient

## Additional Context

### Timer Behavior

Timers are server-authoritative. The database stores the absolute wall
clock time when the timer expires, not a remaining duration. When a
timer is paused, the site should store the remaining duration at the
moment the timer was paused; when the timer is unpaused, a new
absolute wall clock time must be calculated and stored. With all this
said, the consistency guarantees do not need to be super high - losing
a second here or there will not be a big deal for a site that is meant
for such casual usage.

Client-side rendering of the countdown is based on computing the delta
between "now" and the end time stored in the database. Periodically
the client should sync with the server, but not in real-time, the
visual countdown should be client-side between syncs.

If a timer expires when the tab is closed, a browser notification
should list timers that expired since they were last seen. The
notification should be persistent until acknowledged.

### (Rough) Data Model

A rough sketch of the data model:

User:

  - Unique ID;
    - Open question: does this come in via oauth or will we need to
    generate it? If generated, we should use something like UUID
    rather than depending on the data store, so we can be flexible on
    data store.
  - Human-readable identifier via oauth
  - Preferences
    - Open question: json blob or foreign key?
  - Last seen at (timestamp)
  - Open question: anything else coming in from oauth we should store?

Preferences:

  - User ID
  - Color Mode (auto/light/dark)
  - Other preferences we've specified so far
    - In the initial discussion, we can settle on the complete set

Timer:

  - Unique ID: UUID (generated)
  - User ID
  - Name (required, not required to be unique)
  - Creation timestamp
  - Total Duration (as originally set/edited, normalized to seconds)
  - End Time (absolute wall-clock timestamp, null if paused)
  - Time Remaining (normalized to seconds, set only when paused - null
    otherwise)
  - Notification action
  - Expired: bool

### Error Conditions/Handling

Follow the principle of "make invalid states unrepresentable". Perform
client-side validation of timer information before accepting. Provide
clear real-time visual feedback like a visible red highlight and error
message when a field is invalid. Don't defer validation to when the
user clicks "save" (though a final validation is reasonable). A
zero-duration timer is valid (though relatively useless) and should be
displayed as finished immediately after save.

Authentication failures should provide a reasonable and informative
error message. Backend failures should display a message and provide a
retry button.
