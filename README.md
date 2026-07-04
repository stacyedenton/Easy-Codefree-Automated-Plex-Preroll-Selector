# 🎬 Plex Pre-roll Holiday Switcher

Automatically switches your Plex pre-roll video based on the time of year — no manual changes needed. Set it up once, and your pre-rolls update themselves every morning.

---

## What It Does

Plex lets you set a short video clip that plays before movies — called a pre-roll. This tool lets you configure different pre-rolls for different holidays and automatically switches between them on a schedule.

- **Halloween?** Your spooky pre-roll kicks in 7 days before and runs through October 31st
- **Christmas?** Festive pre-roll starts a week before the 25th
- **April Fools?** Plays on the day and sticks around for a few days after
- **Normal days?** Falls back to your default pre-roll automatically

All of this happens silently at 6:00 AM every morning via Windows Task Scheduler — no windows pop up, nothing to click.

---

## Features

- Simple GUI to configure all settings
- Supports **9 holidays** out of the box (see full list below)
- **Two pre-roll slots per holiday** — both play back-to-back (Plex comma-separated format)
- Holiday windows — pre-rolls switch days *before* the holiday, not just on the day
- Fully automatic via **Windows Task Scheduler** — set it and forget it
- **Windows toast notifications** confirm each morning's update
- Last auto-run status displayed in the GUI so you always know it worked
- Supports local files and **UNC network paths** (e.g. `\\192.168.1.0\NAS\prerolls\clip.mp4`)
- Works as a standalone `.exe` 
- No data collection, no internet connection except to your local Plex server

---

## Supported Holidays

| Holiday | Window |
|---|---|
| Valentine's Day | 7 days before → Feb 14 |
| April Fools | Apr 1 → 3 days after |
| July 4th | 7 days before → Jul 4 |
| Halloween | 7 days before → Oct 31 |
| Thanksgiving (US) | 7 days before → Thanksgiving day |
| Christmas | 7 days before → Dec 25 |
| Easter | 7 days before → Easter Sunday |
| Mardi Gras | 7 days before → Mardi Gras day |
| Default | Every other day |

---

## Requirements

- Windows 10 or Windows 11
- Plex Media Server (local network, any version with pre-roll support)
- Your Plex authentication token ([how to find it](https://support.plex.tv/articles/204059436-finding-an-authentication-token-x-plex-token/))
- PowerShell (built into Windows — no install needed)

---

## Installation

### Option A — Run from source (requires AutoHotkey v2)

1. Download and install [AutoHotkey v2](https://www.autohotkey.com/)
2. Download `HolidaySwitcher.ahk` from this repo
3. Double-click to run

### Option B — Compiled .exe (no AutoHotkey needed)

1. Download `HolidaySwitcher.exe` from the [Releases](../../releases) page
2. Double-click to run — nothing else to install

---

## Setup

1. **Open the app** — the GUI will appear
2. **Enter your Plex Server URL** — usually `http://localhost:32400` if Plex is on the same machine, or `http://192.168.1.X:32400` if on another device on your network
3. **Enter your Plex Token** — click the link in the app if you need help finding it
4. **Fill in your pre-roll file paths** for each holiday you want to customize
   - File 1 is required, File 2 is optional (plays after File 1)
   - Leave a holiday blank to skip it and fall back to Default
   - Use full paths: `C:\Videos\halloween.mp4` or `\\NAS\media\halloween.mp4`
5. **Click Save & Apply**
   - A UAC (admin) prompt will appear — this is required to create the scheduled task
   - Click Yes
   - The app will update Plex immediately and schedule future updates

That's it. You can close the app — the scheduled task runs independently every morning.

---

## How It Works

```
┌─────────────────────┐        Save & Apply
│   HolidaySwitcher   │ ──────────────────────────────────────────┐
│   (GUI / .exe)      │                                           │
└─────────────────────┘                                           │
          │                                              ┌────────▼────────┐
          │ Writes settings to                           │  PlexSwitch.ps1  │
          └──────────────────────────────────────────►  │  (in AppData)    │
                                                         └────────┬────────┘
                                                                  │
                                              ┌───────────────────▼──────────────────┐
                                              │       Windows Task Scheduler          │
                                              │   Runs PlexSwitch.ps1 at 6:00 AM     │
                                              │   every day via powershell.exe        │
                                              └───────────────────┬──────────────────┘
                                                                  │
                                                         ┌────────▼────────┐
                                                         │   Plex Server   │
                                                         │  HTTP PUT API   │
                                                         └─────────────────┘
```

When you click **Save & Apply**:
1. Your settings are saved to `%AppData%\PlexPrerollSwitcher\settings.ini`
2. A PowerShell script (`PlexSwitch.ps1`) is generated with your paths and all holiday logic baked in
3. A Windows Task Scheduler task (`PlexPreroll_Switcher`) is created/updated to run that script daily at 6 AM
4. The PowerShell script runs immediately to update Plex right now

Every morning at **6:00 AM**, Windows silently runs `PlexSwitch.ps1` which:
- Checks today's date against all holiday windows
- Picks the matching pre-roll (or Default)
- Sends a PUT request to your Plex server's API
- Verifies Plex saved the value correctly
- Shows a Windows toast notification with the result
- Writes a status entry that appears in the GUI next time you open it

---

## Files Created

All files are stored in `%AppData%\PlexPrerollSwitcher\`:

| File | Purpose |
|---|---|
| `settings.ini` | Your saved GUI settings |
| `PlexSwitch.ps1` | Generated PowerShell script (what Task Scheduler runs) |
| `RegisterTask.ps1` | Temporary script used to register the scheduled task |
| `run.log` | Full log of every run — useful for troubleshooting |
| `last_run.ini` | Status of the most recent scheduled run |

---

## Troubleshooting

**The GUI opens fine but Plex doesn't update**
- Check your Plex URL — make sure you can open `http://YOUR_IP:32400` in a browser
- Verify your token is correct — it should be a string of letters and numbers
- Open `run.log` (linked in the GUI on failure) to see the exact error

**I don't see the scheduled task in Task Scheduler**
- Open Task Scheduler (`Win + R` → `taskschd.msc`) and look in **Task Scheduler Library**
- Look for `PlexPreroll_Switcher`
- If it's missing, click Save & Apply again and accept the UAC prompt

**The pre-roll isn't switching on the right day**
- The script runs at 6:00 AM — if your PC is off or asleep at that time, the task won't run until the next day
- In Task Scheduler, the task is set to **Start When Available** — so if your PC was off, it will run the next time it starts up

**I changed my NAS IP address**
- Open the GUI, update the file paths with the new IP, and click Save & Apply
- This regenerates `PlexSwitch.ps1` with the new paths automatically

---

## Updating Settings

Just open the app, make your changes, and click **Save & Apply** again. It will:
- Overwrite the saved settings
- Regenerate `PlexSwitch.ps1` with the updated paths
- Update the scheduled task
- Apply the change to Plex immediately

---

## Compiling from Source

If you want to build the `.exe` yourself:

1. Install [AutoHotkey v2](https://www.autohotkey.com/)
2. Right-click `HolidaySwitcher.ahk` → **Compile Script**
3. Or use Ahk2Exe directly: `Ahk2Exe.exe /in HolidaySwitcher.ahk /out HolidaySwitcher.exe`

The compiled `.exe` is fully standalone — no AHK installation needed to run it.

---

## Privacy & Security

- **No data leaves your network.** The app only communicates with your local Plex server.
- **Your Plex token is stored in plain text** in `%AppData%\PlexPrerollSwitcher\settings.ini`. This is consistent with how other Plex tools handle tokens, but be aware that any app running on your PC could read this file.
- **No ads, no telemetry, no phoning home.**

---

## Credits

Created by [Stacy Edenton](https://stacyedenton.com) With help by Claude / ChatGPT

---

## Support
If this tool saves you time, consider donating!
[💵 Donate via Cash App](https://cash.app/$StacyEdenton)

## License

MIT License — free to use, modify, and distribute. See `LICENSE` for details.
