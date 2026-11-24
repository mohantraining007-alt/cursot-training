# Implementation TODO

Small steps for the local demo in [requirements.md](requirements.md). Complete one checkbox at a time. Each step includes the code change, a security check, and a documentation update.

Run and open the app only on this machine at `127.0.0.1:5254`. Leave remote deploy out of the workflow.

## 1. Project initialization and basic UI

- [x] Add `.gitignore` for `.venv/`, `__pycache__/`, `.pytest_cache/`, `data/`, and `.env`.
- [x] Add `requirements.txt` with pinned `Flask`.
- [x] Add `app.py` that serves one page on `127.0.0.1:5254`.
- [x] Add a base template, local CSS, and a home screen with the product name and a short description of the demo.
- [x] **Security:** Bind to `127.0.0.1:5254` only. Keep the virtualenv and secret files out of git.
- [x] **Docs:** Add `README.md` with the local setup steps that exist so far (virtualenv, install, `python app.py`, open the home page).

## 2. Sign-in and unit tests

- [ ] Add pinned `pytest` to `requirements.txt`.
- [ ] Hash the admin password with Werkzeug. Username is `admin`.
- [ ] On first start, prompt once in the terminal and write `data/admin.password_hash`. Start the site only after the hash file exists.
- [ ] Add the sign-in screen (username prefilled with `admin`), session login, generic failure text, and Sign out.
- [ ] Send unsigned requests to the sign-in screen. Show a read-only placeholder after sign-in.
- [ ] Generate a new session secret when the process starts and store it only in memory.
- [ ] **Tests:** correct password, wrong password, wrong username, missing hash file, sign-out, and unsigned access to the placeholder.
- [ ] **Security:** Same error text for a bad username and a bad password. Session cookie is `HttpOnly` and `SameSite=Lax`. Never log or render the password. The hash file stays gitignored.
- [ ] **Docs:** Document the first-start password prompt, sign-in, and `pytest` in `README.md`.

## 3. GitHub Actions

- [ ] Add `.github/workflows/demo.yml`, triggered on push and pull request.
- [ ] `test` job: install `requirements.txt` and run `pytest`.
- [ ] `smoke` job: write a hash inside the runner, start the app on `127.0.0.1:5254`, and request the sign-in page.
- [ ] **Security:** Set `permissions: contents: read`. Use current major versions of official actions. Keep secrets out of workflow logs.
- [ ] **Docs:** Note in `README.md` that Actions runs the tests and the localhost smoke check, and that the demo itself is started on this machine.

## 4. Server config

- [ ] Add `config/server.yml` with `name` and `description`.
- [ ] Load that file from a fixed path. If it is missing, use the machine hostname and an empty description.
- [ ] Ignore keys other than `name` and `description`.
- [ ] **Tests:** file present, file missing, and extra keys ignored.
- [ ] **Security:** Read the file in Python. Do not pass its contents to a shell.
- [ ] **Docs:** Document `config/server.yml` in `README.md`.

## 5. Read-only server screen

- [ ] After sign-in, show the server name, description, Sign out, and empty areas for CPU, memory, disk, and network.
- [ ] Leave out add, edit, and delete controls.
- [ ] **Tests:** a signed-in user sees the configured name; a signed-out user is sent to sign-in.
- [ ] **Security:** Render the name and description as escaped text.
- [ ] **Docs:** Describe the server screen in `README.md`.

## 6. In-memory history

- [ ] Store samples in process memory, capped at 30 (about 15 minutes at a 30-second interval).
- [ ] A new process starts with an empty history.
- [ ] **Tests:** append, cap at 30, and read the series back in order.
- [ ] **Security:** Store numeric readings and timestamps only.
- [ ] **Docs:** State in `README.md` that history lasts about 15 minutes and clears when the process stops.

## 7. CPU reading

- [ ] Read overall CPU percent with `psutil`.
- [ ] **Tests:** use a fake collector and check the stored percent.
- [ ] **Security:** Read local counters only.
- [ ] **Docs:** Add CPU to the metrics list in `README.md`.

## 8. Memory reading

- [ ] Read used, total, and percent with `psutil`.
- [ ] **Tests:** use a fake collector for all three values.
- [ ] **Security:** Read local counters only.
- [ ] **Docs:** Add memory to the metrics list in `README.md`.

## 9. Disk reading

- [ ] List real filesystems with mount point, used, total, and percent.
- [ ] Skip pseudo filesystems such as `tmpfs`.
- [ ] **Tests:** a `tmpfs` mount is excluded and a normal mount is kept.
- [ ] **Security:** Read local mounts only. Escape mount points when rendered.
- [ ] **Docs:** Add disk to the metrics list in `README.md`.

## 10. Network reading

- [ ] Read each interface except loopback: bytes in, bytes out, and the rate since the previous sample.
- [ ] Leave the rate empty on the first sample.
- [ ] Keep a trend of total bytes-in rate and total bytes-out rate.
- [ ] **Tests:** loopback excluded, first sample has no rate, second sample has a rate.
- [ ] **Security:** Read local interface counters only. Escape interface names when rendered.
- [ ] **Docs:** Add network to the metrics list in `README.md`.

## 11. Metrics endpoint

- [ ] Add a signed-in endpoint that returns the latest sample and the CPU, memory, and network series.
- [ ] If one collector fails, return an unavailable flag for that reading and still return the others.
- [ ] **Tests:** signed-in payload, signed-out rejection, and one failing collector.
- [ ] **Security:** Require a session. Return a generic error body with no stack trace.
- [ ] **Docs:** Document the endpoint path and auth requirement in `README.md`.

## 12. Metric panels

- [ ] Fill the four areas from the endpoint. Label units (percent, bytes, bytes per second).
- [ ] Show the latest sample time in local time.
- [ ] Show an unavailable state on a failed reading and keep the other panels visible.
- [ ] **Tests:** cover the unavailable flag already added in step 11; confirm the page shell still renders for a signed-in user.
- [ ] **Security:** Keep Jinja autoescape on. Treat mount points and interface names as text.
- [ ] **Docs:** Note the four panels and the unavailable state in `README.md`.

## 13. Local charts

- [ ] Chart CPU percent, memory percent, and network rates over the retained samples.
- [ ] Serve the chart script and styles from the app.
- [ ] **Security:** Load no script or style from a CDN or other host.
- [ ] **Docs:** Note in `README.md` that chart assets are local.

## 14. Refresh

- [ ] Request new readings about every 30 seconds and update the panels and charts in place.
- [ ] Add Refresh, which fetches immediately and restarts the 30-second interval.
- [ ] **Tests:** the metrics endpoint used by the poll stays covered by step 11.
- [ ] **Security:** Fetch same-origin URLs only, and send the session cookie with `SameSite=Lax`.
- [ ] **Docs:** Document the 30-second refresh and the Refresh button in `README.md`.

## 15. Acceptance pass

- [ ] Walk the acceptance list in `requirements.md` on this machine.
- [ ] Confirm the page has no control that adds, edits, or deletes a server.
- [ ] **Security:** Confirm the app binds to `127.0.0.1:5254`, the hash file is gitignored, the sign-in error is generic, the cookie is `HttpOnly` / `SameSite`, server text is escaped, and the workflow uses `contents: read`.
- [ ] **Docs:** Update `README.md` so the setup matches the four local steps in `requirements.md`.
