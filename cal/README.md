# Calendar

A single-page checkmark calendar, live at https://www.hoai.net/stuff/cal/.

`index.html` is the whole front end (inline CSS and JS). It keeps a log of
checkmarks in the browser's `localStorage` and syncs it every 15 seconds with
`./api.php`.

## Only on the server

- `api.php`, the sync backend. The server runs it, so it can't be downloaded
  and isn't in this repository.
- The log file `api.php` reads and writes. The deploy never deletes server
  files and `.deployignore` keeps `cal/*.json`, `*.log`, `*.txt` and `data/`
  from being uploaded, so the log is never overwritten.

## Checks

- Open `index.html` in a browser: the months render and clicking a day cycles
  its mark. Without `api.php` (opened locally) marks stay in that browser only.
- After a deploy, https://www.hoai.net/stuff/cal/ still shows the existing
  checkmarks.
