# crono

Log the foods you eat every day to [Cronometer](https://cronometer.com) from the terminal.

```sh
crono log americano celsius --breakfast
crono log bloom -s --date yesterday
crono search cafe latte -b
crono diary
crono undo
```

Cronometer has no public API for personal accounts. This tool logs in the way the website does and replays the "add food" request the website sent when you first added the food by hand. Because it sends the website's own request, each entry keeps the serving size and unit you chose in the website. The meal comes from `crono log`'s flags.

It is a single Python 3 file with no dependencies.

## Setup

1. Clone it and put `crono` on your `PATH`:

   ```sh
   git clone https://github.com/jwhitlow45/cronometer-cli.git ~/repos/cronometer-cli
   ln -s ~/repos/cronometer-cli/crono ~/.local/bin/crono
   ```

2. For zsh tab completion of commands and saved food names, add this line to `~/.zshrc` above `source $ZSH/oh-my-zsh.sh` (or above your own `compinit`), then open a new terminal:

   ```sh
   fpath=(~/repos/cronometer-cli/completions $fpath)
   ```

3. Save your login. Your password goes into the macOS Keychain (service `cronometer-cli`).

   ```sh
   crono login
   ```

4. Save each food once:
   1. Open https://cronometer.com/#diary with DevTools open on the Network tab.
   2. Add the food the normal way, with the serving size you want. Any meal is fine: `crono log` sets the meal each time.
   3. Find the request named `app` whose payload contains `updateDiary`.
   4. Right-click it, then Copy > Copy as cURL (or copy the raw request payload).
   5. Run `crono save americano`. It reads the request from the clipboard, then clears the clipboard, since "Copy as cURL" includes your session cookies.

## Commands

| Command | What it does |
| --- | --- |
| `crono login` | Save your email and password, then log in. |
| `crono save NAME` | Save the add-food request in the clipboard (or on stdin) as `NAME`. Saving an existing name replaces it. |
| `crono list [--names]` | List saved foods. `--names` prints only the names, which tab completion uses. |
| `crono delete NAME...` | Delete saved foods. |
| `crono log NAME... [-b \| -l \| -d \| -s \| -u] [-D DATE]` | Log saved foods to Breakfast (`-b`, `--breakfast`), Lunch (`-l`, `--lunch`), Dinner (`-d`, `--dinner`), Snacks (`-s`, `--snacks`), or Uncategorized (`-u`, `--uncategorized`). With no meal flag, the meal comes from your meal times (see `crono meals`), or is uncategorized when none are set. Logs for today, or for `DATE` (`YYYY-MM-DD`, `today`, `yesterday`). With no `NAME`, shows the diary, like `crono diary`. |
| `crono search WORD... [-b \| -l \| -d \| -s \| -u] [-D DATE]` | Search Cronometer's foods. Each result shows calories, carbs, protein, fat, and caffeine (when it has any) for the website's suggested serving, and the names it is saved under, if any. Arrow keys (or `j`/`k`) move. Enter or space logs the food: pick a serving size, then type an amount (`1`, `1.5`, `1/2`). Tab saves it instead, under a name you choose, for `crono log`. Tab on a food you already saved deletes those saved foods. `q` cancels. |
| `crono diary [-D DATE]` | Show what is logged for today, or for `DATE`, grouped by meal, with the day's total calories and caffeine. |
| `crono unlog [-D DATE]` | Pick entries logged today, or on `DATE`, and remove them. This includes entries added on the website. Arrow keys (or `j`/`k`) move, space selects, enter removes the selected entries or the highlighted one, `q` cancels. |
| `crono undo` | Remove the entries the last `crono log` added. |
| `crono meals [-b TIME] [-l TIME] [-d TIME] [--off]` | Show or set when breakfast, lunch, and dinner start, such as `crono meals -b 7 -l 11:30am -d 17:00`. A food logged for today without a meal flag goes to the meal that started last, and each meal lasts until the next starts (so after dinner it stays Dinner until breakfast). Foods logged for other days without a flag stay uncategorized. `--off` clears the times. |

`unlog` shows each entry's meal and weight in grams. It names meals with Cronometer's defaults (Breakfast, Lunch, Dinner, Snacks), so renamed meals show their default names. The first time a food appears, `unlog` looks up its name with one export, then caches it.

## Files

- `~/.config/crono/config.json`: your email and saved foods. The saved requests hold no session secrets.
- `~/.local/state/crono/`: session cookies, the current web app build, the last log for `undo`, and cached food names.

## When it breaks

This relies on Cronometer's private web API, which can change without notice.

- Each Cronometer deploy changes the web app's build hashes. `crono` looks up the current ones on every run, so deploys alone should not break it.
- If Cronometer changes the shape of a diary entry, logging fails with `IncompatibleRemoteServiceException`. Save the affected foods again.
- `crono diary` reads Cronometer's CSV exports (two per run). The cronometer-mcp author reports that a burst of about ten exports locked out exports for over an hour, so avoid running it in a tight loop.
- The servings export has no nutrition columns, so `crono diary` can show only the day's total calories, not each food's.
- Automated access may be against Cronometer's terms of service. Use at your own risk.

## Credit

The login flow, build hash lookup, CSV exports, and the `authenticate` and `removeServing` requests follow [cphoskins/cronometer-mcp](https://github.com/cphoskins/cronometer-mcp) (MIT license).
