# Swipe Tracker - Made for NYU's Swipe It Forward Program

Swipe Tracker monitors the donated balance shown in NYU's meal portal and shares updates through Discord. Instead of having to repeatedly opening the portal to check availability, users can run `/balance` or receive an `@everyone` ping when the balance changes from zero to a positive value.

The entire program runs on Node and can be hosted on a local or cloud server. It uses an existing authenticated NYU session URL to access content within the meal portal.

## What it does

- Checks the portal immediately on startup about once a minute.
- Replies to `/balance` with a message displaying the last valid balance, when it was checked, and monitoring status.
- Supports Discord server installs and user installs, allowing users within servers and group DM's to run the command.
- Sends availability alerts to configured server channels.
- Saves balance history and pending alerts across restarts.
- Preserves the last valid reading to display when requests fail, rather than reporting a false zero.

`/balance` only displays the latest saved observation: it does not fetch an immediate NYU request.

## How it works

```text
Authenticated NYU balance page -> Node monitor -> HTML parser -> Saved balance -> /balance reply
                                                                        |
                                                                availability alert
```

The parser selects the Swipe It Forward table and its Current Balance row, removes hidden duplicate accessibility text, and validates the visible amount as a nonnegative integer. Each check has a 20-second timeout and a 2 MB response limit; Each check runs about one minute after the previous check completes.

Alerts follow these rules:

- **No history** to **Any valid balance**: Establish a baseline
- **0** to **Positive**: Send an @everyone ping
- **Positive** to **Positive**: Update the balance, no alert
- **Positive** to **0**: Update the balance, no alert
- **Any** to **Failed check**: Keep the last valid balance
