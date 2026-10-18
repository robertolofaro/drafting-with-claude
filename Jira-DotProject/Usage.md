I'll build this as a small Python project: a config-driven extractor for Jira and dotProject, a SQLite3 "transit" database builder, and a Tk wizard GUI that walks through your four steps. Let me lay out the files.

Now let's write the files.

Let me fix a rough edge: reopening the DB in step 3 should use a clean method rather than a hacky inline import.

Now let's prefill the author field with the user's email, and write `main.py`.

Good, `default_author` is set before `_build_notebook` (which builds tab3). Now `run()` and `main.py`:

Now the README:

Now let's smoke-test: install deps, byte-compile, and run logic tests without network/DB.

All core logic checks out. Let's also sanity-check the GUI module parses correctly as a module (tkinter isn't installed in this sandbox, so I can't exercise the window itself, but I'll verify there are no obvious reference bugs by inspecting it once more).

Now let's package it up and send it to you.

**What's in pm-extractor.zip** — all logic smoke-tested (config parsing, dynamic table creation, Jira/dotProject write paths, ZIP packaging all pass; only the Tk window itself couldn't be exercised here since this sandbox has no tkinter/display, but it's plain stdlib `ttk` and will run fine on your machine):

- **`main.py`** — launches the GUI, optional `config.yaml` + `--author` args.
- **`app/config.py`** — loads `scopes.yaml`: one entry per project/program/initiative, each either `type: jira` or `type: dotproject`, with its own credentials and retrieval parameters.
- **`app/jira_source.py`** — REST client supporting both Cloud (email+API token) and Server/DC (PAT or basic auth), paginated JQL search, attachment download.
- **`app/dotproject_source.py`** — MySQL/MariaDB client that runs *your* arbitrary SQL (since your fork's schema is custom, I made no assumptions about table names) plus an attachments query that copies files from a configured base path.
- **`app/db_builder.py`** — builds `transit.sqlite3`: fixed tables for `meta`, `selection_criteria`, `retrieval_stats`, `commentary`, `jira_issues(_raw)`, `jira_attachments`, `dotproject_attachments`, plus one dynamic `dp_<query_name>` table per dotProject query, with columns inferred from the result set.
- **`app/packager.py`** — zips the DB + `attachments/` + a manifest.
- **`app/gui.py`** — the 4-tab wizard: Parameters → Build DB → Commentary → Package, each step manually triggered and only unlocked once the previous one succeeds, with a live log panel and "Test connection" buttons for both backends.
- **`config/scopes.example.yaml`** — fully commented template for both a Jira and a dotProject scope.
- **`README.md`** — install, config, and step-by-step usage.

To run it: `pip install -r requirements.txt`, copy `config/scopes.example.yaml` to your own file (and `chmod 600` it, since credentials are in clear text as you asked), then `python main.py config/scopes.yaml`.

One thing worth flagging: dotProject attachments assume files sit on a filesystem path you can point to (local or mounted share) — if your fork serves them differently (e.g. a download script or BLOB column), that's an easy tweak in `fetch_attachments()`. Let me know if that's actually how yours works and I'll adjust it.
