# crono

Log the foods you eat every day to [Cronometer](https://cronometer.com) from the terminal.

```sh
crono log americano celsius --breakfast
crono log beef-curry:0.5 -d
crono log beef-curry:300kcal oat-milk:250ml -l
crono log bloom -s --date yesterday
crono search cafe latte -b
crono
crono undo
```

Cronometer has no public API for personal accounts. This tool logs in the way the website does and sends the same "add food" request the website sends. Find foods with `crono search`, and save the ones you eat often so `crono log` can log them by name.

It is a single Python 3 file with no dependencies.

## Setup

1. Clone it and put `crono` on your `PATH`:

   ```sh
   git clone https://github.com/jwhitlow45/cronometer-cli.git ~/repos/cronometer-cli
   ln -s ~/repos/cronometer-cli/crono ~/.local/bin/crono
   ```

2. For zsh tab completion of commands, saved food names, and units after `NAME:` (such as `oat-milk:250m` to `oat-milk:250ml`), add this line to `~/.zshrc` above `source $ZSH/oh-my-zsh.sh` (or above your own `compinit`), then open a new terminal:

   ```sh
   fpath=(~/repos/cronometer-cli/completions $fpath)
   ```

3. Save your login. Your password goes into the macOS Keychain (service `cronometer-cli`).

   ```sh
   crono login
   ```

4. Save the foods you eat often. Run `crono search americano`, highlight the one you want, and press Tab. Pick a serving size, type an amount, and name it. Then `crono log americano` logs it.

## Commands

| Command | What it does |
| --- | --- |
| `crono` | Show today's diary, like `crono diary`. |
| `crono login` | Save your email and password, then log in. |
| `crono list [--names \| --units NAME]` | List saved foods with each one's Cronometer name, serving, and calories. `--names` prints only the names, and `--units NAME` only the units that food's amount can be given in. Tab completion uses both. |
| `crono delete NAME...` | Delete saved foods. |
| `crono log NAME[:AMOUNT]... [-b \| -l \| -d \| -s \| -u] [-D DATE]` | Log saved foods, then show the day's total. `NAME:AMOUNT` logs a multiple of the saved amount, such as `beef-curry:0.5` or `coke-zero-12oz:2`. An amount with a unit logs that much, such as `beef-curry:300kcal`, `oat-milk:250ml`, or `americano:16floz`. The units are the food's own serving sizes (`cliff-bar:2bar`), `kcal`, the mass units `g`, `kg`, `oz`, and `lb` for a food with a mass serving size, and the volume units `ml`, `L`, `tsp`, `tbsp`, `fl oz`, and `cup` for a food with a serving size of known volume. Cronometer shows the entry in the serving size the unit belongs to or converts through, and in the saved serving size for `kcal`. A wrong unit lists the ones the food takes. A name can be shortened to any start of 2 letters or more that only one saved food has, such as `beef` for `beef-curry`. The meal is Breakfast (`-b`, `--breakfast`), Lunch (`-l`, `--lunch`), Dinner (`-d`, `--dinner`), Snacks (`-s`, `--snacks`), or Uncategorized (`-u`, `--uncategorized`). With no meal flag, the meal comes from your meal times (see `crono meals`), or is uncategorized when none are set. With no `NAME`, shows the diary. |
| `crono search WORD... [-b \| -l \| -d \| -s \| -u] [-D DATE]` | Search Cronometer's foods. Each result shows calories, carbs, protein, fat, and caffeine (when it has any) for the website's suggested serving, and the names it is saved under, if any. Arrow keys (or `j`/`k`) move. Enter or space logs the food: pick a serving size, then type an amount (`1`, `1.5`, `1/2`). After the food's serving sizes come `kcal` and the mass and volume units crono can convert, as for `crono log`. Tab saves it instead, under a name you choose, for `crono log`. Tab on a food you already saved deletes those saved foods. "Show more results" at the end of the list searches further. `q` in the serving sizes goes back to the results, and `q` in the results quits. |
| `crono diary [-D DATE]` | Show what is logged, grouped by meal, with each entry's serving and calories, then the day's calories, carbs, protein, fat, and caffeine. |
| `crono unlog [-D DATE]` | Pick entries to remove, including ones added on the website. Arrow keys (or `j`/`k`) move, space selects, enter removes the selected entries or the highlighted one, `q` cancels. |
| `crono undo` | Remove the entries the last `crono log` added. |
| `crono meals [-b TIME] [-l TIME] [-d TIME] [--off]` | Show or set when breakfast, lunch, and dinner start, such as `crono meals -b 7 -l 11:30am -d 17:00`. A food logged for today without a meal flag goes to the meal that started last, and each meal lasts until the next starts (so after dinner it stays Dinner until breakfast). Foods logged for other days without a flag stay uncategorized. `--off` clears the times. |

`DATE` is `today` (the default), `yesterday`, a weekday such as `mon` (the latest one, so `mon` on a Monday is today), `-N` for N days ago (`-D -2`), or `YYYY-MM-DD`. The date flag is a capital `-D`, because `-d` is Dinner.

`diary` and `unlog` name meals with Cronometer's defaults (Breakfast, Lunch, Dinner, Snacks), so renamed meals show their default names.

## Files

- `~/.config/crono/config.json`: your email, saved foods, and meal times. The saved requests hold no session secrets.
- `~/.local/state/crono/`: session cookies, the current web app build, and the last log for `undo`.

## When it breaks

This relies on Cronometer's private web API, which can change without notice.

- Each Cronometer deploy changes the web app's build hashes. `crono` looks up the current ones on every run, so deploys alone should not break it.
- If Cronometer changes the shape of a diary entry, logging fails with a message that says the web app changed. Save the affected foods again with `crono search`.
- Automated access may be against Cronometer's terms of service. Use at your own risk.

## Credit

The login flow, build hash lookup, and the `authenticate` and `removeServing` requests follow [cphoskins/cronometer-mcp](https://github.com/cphoskins/cronometer-mcp) (MIT license).
