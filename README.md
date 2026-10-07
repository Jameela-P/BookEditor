# Wikisource → Wikidata book editor

A userscript for [Wikidata](https://www.wikidata.org) that adds Wikisource books to Wikidata as edition items. You load a CSV/TSV list of ml.wikisource index pages, and the tool takes you through it one book at a time. For each book it finds or creates the Wikidata item, compares it with your data, and adds what is missing. **One click = one Wikidata edit**, so you review every change before it is saved.

Written and maintained by [User:Jameela P.](https://www.wikidata.org/wiki/User:Jameela_P.) as part of the Wiki Librarians Network.

---

## Contents

- [Installation](#installation)
- [Preparing your data](#preparing-your-data)
- [Using the tool](#using-the-tool)
- [What gets added](#what-gets-added)
- [Authors, publishers and places](#authors-publishers-and-places)
- [Checks and warnings](#checks-and-warnings)
- [Saving and exporting progress](#saving-and-exporting-progress)
- [Troubleshooting](#troubleshooting)
- [Technical notes](#technical-notes)

---

## Installation

You need a Wikidata account and must be logged in. Adding statements and creating items both need an account.

**Option A: load the maintained copy (recommended).** You get updates automatically.

1. Open your [common.js](https://www.wikidata.org/wiki/Special:MyPage/common.js) on Wikidata.
2. Add this line and save:

   ```js
   mw.loader.load('//www.wikidata.org/w/index.php?title=User:Jameela_P./BookEditor.js&action=raw&ctype=text/javascript');
   ```

**Option B: your own copy.** Use this if you want to change the script.

1. Create the page `User:<your username>/BookEditor.js` and paste in the contents of `BookEditor.js`.
2. In your `common.js`, add:

   ```js
   mw.loader.load('//www.wikidata.org/w/index.php?title=User:<your username>/BookEditor.js&action=raw&ctype=text/javascript');
   ```

**Open the tool** at:

> https://www.wikidata.org/wiki/Special:BlankPage/BookEditor

The script runs only on that page. Everywhere else on Wikidata it does nothing.

---

## Preparing your data

The tool reads **CSV or TSV** files (it detects the delimiter) encoded as **UTF-8**. The first row must hold the column names.

### Minimum

The only required column is **`index_page_title`**, for example `സൂചിക:Kandamrutham 1906.pdf`.

When a row has only the index page and nothing else (no title, author, publisher or year), the tool **reads that index page on ml.wikisource** and fills in the blanks:

- book title, including the `[[link]]` used for the sitelink;
- author, plus their author page if it is linked;
- translator, editor, publisher, place of publication and year;
- page count and the real file location, read from the scan itself.

Only empty cells are filled, never ones you provided. This happens once per row and is saved with your progress.

### Recognised columns

Column names are matched automatically. Open **Column mapping** in the tool if your file uses other names.

| Column | Used for |
|---|---|
| `index_page_title` | P1957 index URL, P996 file name, and reading the index page |
| `book_title` | P1476 title, and the label of a new item |
| `book_title_raw` | Wikitext title. A `[[page]]` link becomes the mlwikisource sitelink |
| `book_title_url` | Wikisource work page, used to find an already-linked item |
| `author` | P50 author, or P2093 author name string |
| `author_url` | Wikisource author page, used to find the author's item |
| `translator` | P655 translator |
| `editor` | P98 editor |
| `publisher` | P123 publisher |
| `publication_place` | P291 place of publication |
| `publication_year` | P577 publication date (year) |
| `page_count` | P1104 number of pages |
| `source_file_url` | Fallback file name, and detecting files hosted only on Wikisource |
| `cover_thumbnail_url` | Cover preview shown in the tool |
| `wikidata_id` | A known QID. If present, the tool uses that item directly |

Other columns are kept as they are and come back unchanged when you export.

### Several people in one cell

For authors, translators and editors, separate names with `&` or `;`, for example `P K Chomen & P J Kuriyan`. Each name becomes its own statement.

Publishers are **not** split, because names like *Basel Mission Book & Tract Depository* contain `&`.

---

## Using the tool

1. **Load your file** with the file picker at the top.
2. **Check the column mapping** once. It is remembered for the session.
3. The tool opens the first book and does the following automatically:
   - **Finds the book item.** It searches in this order:
     1. the QID in your file;
     2. the Wikisource sitelink;
     3. items that already have this index URL (P1957);
     4. items that already have this Commons file (P996);
     5. a title search.

     If exactly one of the first four finds a match, that item is loaded for you. Otherwise you get a list of candidates showing their *instance of* and year.
   - **Looks up each author, publisher and place** on Wikidata, searching in both Malayalam and English.
   - **Checks** that the Commons file and the Wikisource page exist.
4. **Choose the book item:**
   - **Use this item** for an existing edition.
   - **Create a new edition of this work**, if the match is the literary work rather than this particular printing. This adds P629 (*edition or translation of*).
   - **Create item**, if nothing matches. The form is pre-filled with the Malayalam title as the label and descriptions such as *1906-ൽ പ്രസിദ്ധീകരിച്ച പതിപ്പ്* / *1906 Malayalam edition*.
5. **Go through the statements table.** Each row shows the value from your file, what Wikidata has now, and a status:

   | Status | Meaning |
   |---|---|
   | Already on Wikidata | Nothing to do |
   | Missing | Click **Add** |
   | Different value on Wikidata | Check it. **Add as extra value** keeps both |
   | Choose an item | Pick or create the author, publisher or place first |
   | No usable value | The data is blank, invalid or not possible (e.g. a local-only file) |
   | Unknown in source – skipped | The file says *unknown*, *ലഭ്യമല്ല*, *അജ്ഞാത…* and so on |

   You can edit any value in the table before adding it. **Add all missing** makes one edit per missing statement, one after another.
6. Click **Mark done and go to next**.

---

## What gets added

| Wikidata | Value |
|---|---|
| [P31](https://www.wikidata.org/wiki/Property:P31) instance of | [Q3331189](https://www.wikidata.org/wiki/Q3331189) version, edition or translation *(always)* |
| [P407](https://www.wikidata.org/wiki/Property:P407) language of work or name | [Q36236](https://www.wikidata.org/wiki/Q36236) Malayalam *(always)* |
| mlwikisource sitelink | Page linked in `book_title_raw` |
| [P1957](https://www.wikidata.org/wiki/Property:P1957) Wikisource index page URL | `https://ml.wikisource.org/wiki/` + index page title |
| [P1476](https://www.wikidata.org/wiki/Property:P1476) title | Book title, language **ml** (can be changed per row) |
| [P50](https://www.wikidata.org/wiki/Property:P50) author | Author's item |
| [P2093](https://www.wikidata.org/wiki/Property:P2093) author name string | Author's name as text, when there is no item |
| [P655](https://www.wikidata.org/wiki/Property:P655) translator | Translator's item |
| [P98](https://www.wikidata.org/wiki/Property:P98) editor | Editor's item |
| [P123](https://www.wikidata.org/wiki/Property:P123) publisher | Publisher's item |
| [P291](https://www.wikidata.org/wiki/Property:P291) place of publication | Place's item |
| [P577](https://www.wikidata.org/wiki/Property:P577) publication date | Year, Gregorian calendar, year precision |
| [P996](https://www.wikidata.org/wiki/Property:P996) document file on Wikimedia Commons | Index page title without `സൂചിക:` |
| [P1104](https://www.wikidata.org/wiki/Property:P1104) number of pages | Page count |
| [P629](https://www.wikidata.org/wiki/Property:P629) edition or translation of | The work, only when you chose *Create a new edition of this work* |

### References

By default, the statements that come from the source data (title, people, publisher, place, date, pages) get a reference:

- [P854](https://www.wikidata.org/wiki/Property:P854) reference URL = the index page URL
- [P813](https://www.wikidata.org/wiki/Property:P813) retrieved = today

To turn this off, untick **Add a reference to new statements**. The reference is saved in the same edit as the statement.

### Edit summary

Every edit is tagged:

> Book data from Wikisource via Wikisource → Wikidata book editor (#mlwsBookEditor)

Sitelink edits start with Wikidata's own summary, *Added link to [mlwikisource]: …*.

---

## Authors, publishers and places

Each name gets a dropdown of matching Wikidata items. You can:

- **pick a match**. A single exact match is preselected for you to confirm;
- **type a QID** directly;
- **search again** with different spelling;
- **create a new item** with **+ Create a new item…**;
- for authors only, **add as author name string (P2093)** with **✎ Not on Wikidata – add as author name string**, when the person has no item and you don't want to create one. You can switch to P50 later once an item exists.

**Your choices are remembered.** Once you match *കൊടുങ്ങല്ലൂർ കുഞ്ഞിക്കുട്ടൻ തമ്പുരാൻ* to a QID, or to P2093, every later book with that exact name uses the same choice.

If the file has a Wikisource author page, the tool first tries the item linked to that page.

### Defaults for new items

| Created from | Label | Description | Statements |
|---|---|---|---|
| Author, translator, editor | The name | *(blank)* | P31 = human (Q5) |
| **Publisher** | The publisher's name | *publishing company based in Kerala* | P31 = publishing house (Q2085381), P17 country = India (Q668) |
| Place | The place name | *(blank)* | P31 = human settlement (Q486972) |

The label is saved in Malayalam (`ml`) if it is in Malayalam script, otherwise in English (`en`). You can change every field before clicking **Create item**.

> Some old publishers were outside Kerala (e.g. Basel Mission at Mangalore, presses in Madras). Change the description for those before creating the item.

---

## Checks and warnings

- **Commons file.** The file is checked on Commons before P996 can be added, and redirects are followed. If the name from the index title isn't found, the name in `source_file_url` is tried instead.
- **Local-only files.** Scans uploaded only to ml.wikisource can't be used for P996. These are flagged and skipped.
- **Sitelinks.** The Wikisource page must exist. If another item already links to it, the tool tells you which one, because Wikidata allows only one item per page. If the book item already links to a different page, the button says **Replace link** and asks you to confirm.
- **Work vs edition.** If the matched item isn't an edition (Q3331189), you're warned and offered to create a separate edition linked with P629.
- **Non-Malayalam titles.** Titles not in Malayalam script (e.g. *Grammatica Malabarico-Latina*) are flagged so you can set the right language code.
- **Year.** The first 3–4 digit number in the year cell is used. The original text is shown when it differs.

---

## Saving and exporting progress

- **Progress is saved in your browser**: the file, mapping, current row, done marks, the QIDs found or created, and details filled from index pages. Close the tab and reopen the tool to continue where you left off.
- **Remembered matches** for authors, publishers and places are kept separately and survive **Clear session**.
- **Export TSV with QIDs** downloads your data with `wikidata_id` filled in, an `editor_status` column (`done`), and any details read from the index pages. Export regularly, because browser storage can be cleared, and a very large file may not fit in it.
- **Edits made in this session** lists every edit with a link to its diff.

---

## Troubleshooting

| Problem | What to do |
|---|---|
| The page is blank or normal | Check the address is exactly `Special:BlankPage/BookEditor`, and that the `mw.loader.load` line is in your `common.js`. Reload the page, bypassing the cache. |
| "You are not logged in" | Log in to Wikidata and reload. |
| Malayalam text looks broken | Save the CSV as UTF-8. In Excel, use *CSV UTF-8*. |
| Columns aren't recognised | Open **Column mapping** and pick them by hand. |
| "is already linked from Q…" | The Wikisource page belongs to another item. Remove the link there, or merge the items if they are duplicates. |
| A file shows "not found on Commons" | Type the correct Commons file name in that row, or leave P996 out. |
| Index page details weren't filled | The index page may not use the standard index template, or Wikisource couldn't be reached. Enter the details in the table instead. |
| Wrong item was remembered for a name | Choose the right item in the dropdown. The new choice replaces the old one. |
| Start over with a new file | **Clear session**, then load the new file. |

---

## Technical notes

- Runs entirely in the browser, through the MediaWiki API on wikidata.org, with your logged-in session. No server, OAuth or bot account is needed. Edits count as yours and follow normal Wikidata rate limits.
- Reads ml.wikisource and Commons anonymously through `mw.ForeignApi`.
- Statements are written with `wbsetclaim`, new items with `wbeditentity`, sitelinks with `wbsetsitelink`. Each call is one edit.
- Book items are looked up with `haswbstatement:` searches on P1957 and P996, sitelinks, and `wbsearchentities`.
- Browser storage keys: `mlwsBE.session.v1` (progress) and `mlwsBE.entityCache.v1` (remembered matches).
- Dependencies: MediaWiki modules `mediawiki.api`, `mediawiki.ForeignApi`, `mediawiki.util`. No external libraries.

### Adapting it for another Wikisource

The script is set up for Malayalam Wikisource. To use it with another language, change these in the *Configuration* section of `BookEditor.js`:

- `WS_BASE`;
- `Q_MALAYALAM` (the language QID for P407);
- the `mlwikisource` site ID;
- the default title language `ml`;
- the index-namespace prefix `സൂചിക`.
