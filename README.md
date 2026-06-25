# KeePassOrganizer

**Organize your KeePass vault with AI without exposing your passwords.**

KeePassOrganizer is a single offline HTML file that splits your KeePass CSV export into two files linked by a unique key. You share the safe file with an AI to reorganize your groups, then merge the result back. Passwords never leave your device.

---

## The problem

KeePass vaults accumulate hundreds of entries dumped into a handful of flat folders. Reorganizing them manually is tedious. Asking an AI to help means pasting passwords somewhere you shouldn't.

## How KeePassOrganizer solves it

KeePassOrganizer separates your vault into what's safe to share and what isn't, lets AI do the organization work on the safe half, then rejoins everything automatically.

```
your_vault.csv
      │
      ▼
 KeePassOrganizer
      │
      ├── public_vault.csv   ← key · Group · Title · URL         (share with AI)
      └── private_vault.csv  ← key · Username · Password · TOTP  (stays offline)

      [AI reorganizes public_vault.csv → returns public_reorganized.csv]

      ├── public_reorganized.csv
      └── private_vault.csv
              │
              ▼
        merge_vault.py
              │
              ▼
        merged_output.csv  ← import into KeePassXC
```

---

## Quickstart

### 1. Export from KeePassXC

`Database → Export → Export to CSV`

### 2. Split

Open `keepass_organizer.html` in any browser (no internet needed). Drop your CSV in and download three files:

- `public_vault.csv` (safe to share, no passwords)
- `private_vault.csv` (keep this offline, never share it)
- `merge_vault.py` (the merge script for later)

### 3. Organize with AI

Upload `public_vault.csv` to [Claude](https://claude.ai) and ask:

> "Here's my KeePass vault. Suggest a clean group structure and reorganize these entries."

Download the reorganized CSV the AI returns and save it as `public_reorganized.csv`.

### 4. Merge

Run the merge script locally (requires Python 3, no extra packages):

```bash
python3 merge_vault.py public_reorganized.csv private_vault.csv merged_output.csv
```

### 5. Import

In KeePassXC: `Database → Import → Import CSV File` → select `merged_output.csv`.

---

## Security model

| What | Goes online? |
|---|---|
| Entry titles | ✅ Yes (that's the point) |
| URLs | ✅ Yes |
| Group names | ✅ Yes |
| Usernames | ❌ Never |
| Passwords | ❌ Never |
| TOTP secrets | ❌ Never |
| Notes | ❌ Never |

The `key` column is a random 6-character alphanumeric ID assigned to each entry. It has no meaning on its own. It only becomes useful when both files are present together.

**The private file never touches the internet.** The public file contains no credentials. Even if the public file were intercepted, it reveals nothing sensitive.

---

## Files

```
keepass_organizer.html   the main tool, open in any browser
merge_vault.py           downloaded from the tool after splitting
README.md                this file
```

No build step. No npm install. No server. One HTML file.

---

## Requirements

- Any modern browser (Chrome, Firefox, Safari, Edge) to run the splitter
- Python 3.x (standard library only) to run the merge script
- KeePassXC or KeePass 2.x

---

## FAQ

**Does this work offline?**
Yes. `keepass_organizer.html` runs entirely in your browser with no network requests. The merge script is plain Python with no dependencies.

**Can I use a different AI?**
Yes, any AI assistant works. Just upload `public_vault.csv` and ask it to suggest groups and return a reorganized CSV with the `key` column intact.

**What if the AI changes a title?**
The merge matches on `key`, not title. Titles can be renamed freely by the AI without breaking the merge.

**What if an entry is missing from the merged output?**
The merge script will print a warning for any keys present in the public file but not found in the private file. This usually means an entry was added or the files got out of sync. Re-split and try again.

**Is my data sent anywhere when I use the tool?**
No. The HTML file reads your CSV using the browser's local File API. Nothing is uploaded.

**Can I reorganize without AI?**
Yes, just edit `public_vault.csv` manually in any spreadsheet app (change the Group column), then merge as normal.

---

## Contributing

Issues and PRs welcome. Ideas for future work:

- Drag-and-drop group editor built into the HTML tool
- Direct KeePassXC `.kdbx` support (via a local companion script)
- Browser extension to skip the CSV export step

---

## License

MIT. Do whatever you like with it.

---

## Credits

Built to scratch a real itch: a 300-entry KeePass vault with everything in one folder. Inspired by the principle that you should never need to trust a third party with your passwords, even to get help organizing them.