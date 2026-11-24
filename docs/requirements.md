# Demo Server Monitoring — Requirements

Local demo for internal teams who need CPU, memory, disk, and network readings without signing in to the server shell.

The app runs on the Linux machine where Cursor is running. It reads that machine only. A signed-in user can view metrics. Server identity is set in a local config file.

## Problem

L1, L2, and L3 teams in infrastructure and application development currently open a shell on a server to check live resource usage. This demo replaces that check for the local host with a small read-only web UI.

## Users

| User | Need |
| --- | --- |
| L1, L2, and L3 (infrastructure and application development) | See current CPU, memory, disk, and network usage, plus a short trend, from a browser |

The demo has one shared account. Everyone who signs in sees the same view-only screens.

## Goals

- Show live CPU, memory, disk, and network usage for this Linux host.
- Keep about 15 minutes of those readings in memory and chart CPU, memory, and network.
- Refresh on its own about every 30 seconds, and refresh immediately when asked.
- Sign in as `admin` with a password that is stored only as a hash.
- Name the monitored host from a local config file. The UI stays view-only.

## Out of scope

- Remote servers, SSH, and agents on other hosts
- Port usage, process lists, and load average
- Saved history across restarts, alerts, and multiple accounts or roles
- Adding, editing, or deleting servers from the UI

## User flows

### First start

1. Operator starts the app from a terminal in the project directory.
2. The app asks once for the admin password, stores a hash, and discards the typed password.
3. Operator opens the site, signs in as `admin`, and lands on the server view.

### Later starts

1. Operator starts the app. It does not ask for a password again.
2. Operator signs in as `admin` and reviews the live readings and short trends.

### Name the host

1. Operator edits the local config file and sets the display name.
2. Operator restarts the app. The UI shows that name for this machine.

## Functional requirements

### Sign-in

- The username is `admin`.
- On the first start, if no password hash exists, the terminal prompts for the admin password once. The app stores a hash (Werkzeug password hash) in a local file and never writes the plain password.
- The first start needs an interactive terminal. Later starts use the saved hash and do not prompt.
- Sign-in checks `admin` and the password against that hash. A failed attempt stays on the sign-in page with a generic error.
- A signed-in session ends with Sign out, or when the app process stops.
- The app listens on `127.0.0.1:5254` only.

### Server list

- The monitored host is the machine running the app. Metrics come from the local operating system.
- A config file holds one server entry: display name and an optional short description.
- If the config file is missing, the display name is the machine hostname and the description is empty.
- Changing the name or description is done by editing the file and restarting the app.
- The signed-in UI lists that server and opens its readings. It has no create, edit, or delete actions.

### Metrics

Readings are collected on the server about every 30 seconds and kept in process memory for about 15 minutes (about 30 samples). A restart clears the history.

| Reading | What the UI shows |
| --- | --- |
| CPU | Current overall CPU percent, and a trend of that percent over the retained window |
| Memory | Used, total, and percent, and a trend of memory percent |
| Disk | Each real filesystem: mount point, used, total, and percent. Skip pseudo filesystems such as `tmpfs`. Current values only |
| Network | Each interface except loopback: bytes in, bytes out, and the rate since the previous sample. A trend of total bytes-in rate and total bytes-out rate |

- The first network sample shows totals without a rate. The rate appears from the second sample on.
- If one reading fails, that panel shows unavailable and the other panels still render.
- Timestamps use the server's local time. The page shows when the latest sample was taken.

### Refresh

- The server view requests new readings about every 30 seconds.
- A Refresh control fetches immediately and restarts the 30-second interval.
- Refresh updates numbers and charts in place.

## Screens

1. **Sign in** — username, password, and a submit action. Username may be prefilled with `admin`.
2. **Server view** — server name, description, last sample time, Refresh, Sign out, four metric areas, and charts for CPU, memory, and network over the retained window.

The layout is readable on a laptop-width browser. Labels state the unit (percent, bytes, or bytes per second). Chart and style files are served by the app itself so the demo does not call a CDN.

## Tech stack

- Python 3 and Flask
- Werkzeug password hashing for the admin password
- `psutil` for local CPU, memory, disk, and network readings
- Flask templates plus a small browser script for refresh and charts
- Local files only: server config, password hash, and in-memory samples. No database

## Config and files

Server config path: `config/server.yml`

```yaml
name: Demo host
description: Local Linux server
```

Password hash path: `data/admin.password_hash`

Both paths are created under the project directory. The hash file is written only by the first-start prompt.

## Local setup

Run these steps on the machine where Cursor is running:

1. Create a virtual environment and install dependencies from `requirements.txt`.
2. Optionally edit `config/server.yml` with the display name and description.
3. Start the app with `python app.py`. The first start asks for the admin password in that terminal.
4. Open `http://127.0.0.1:5254`, sign in as `admin`, and confirm the server view updates about every 30 seconds.

## Acceptance

- First start stores a password hash and will not start the site until a password has been set.
- Sign-in works for `admin` and the chosen password, and rejects a wrong password.
- After sign-in, the page shows this host's CPU, memory, disk, and network readings.
- CPU, memory, and network charts cover roughly the last 15 minutes while the process stays up.
- Readings refresh about every 30 seconds, and Refresh updates them immediately.
- The page has no controls that add, edit, or delete a server or change a reading.
- With the app bound to `127.0.0.1:5254`, another machine cannot open the UI.
