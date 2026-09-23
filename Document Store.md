# The Document Store

Campaign material lives in two places. Project files (this document, the rules documents, the campaign document, house rules) are read-only and stable. Everything that changes during play lives in the document store, reached through the localMCP tools.

Categories are dotted paths. `IronCastle` is the parent category and holds seven leaf categories:

- **IronCastle.Oracles.** Every oracle table, one document per table. Read-only in practice.
- **IronCastle.State.** `game-state.md` and `character-bron.md`.
- **IronCastle.Places.** One document per location established in play, such as `keep.md` or `tap-room.md`. A building's rooms are `##` sections of its document. Each summary says where the place is as well as what it is, so the index reads as a map.
- **IronCastle.Promises.** One document per open promise: its terms and its story. Written when a promise is made, when its story moves on, and archived when it is fulfilled or cancelled.
- **IronCastle.Townies.** One document per Townie, plus `unnamed-townies.md`.
- **IronCastle.Sessions.** Narrative session summaries, numbered `session_001.md`, `session_002.md`, and so on.
- **IronCastle.Holidays.** One document per holiday. Dates are in the summaries, only retrieve what you need.

Documents live only in leaves. Every tool that reads, lists or writes a specific document needs the full leaf path, for example `category: "IronCastle.Oracles"`, `filename: "Townie Traits.md"`. The two search tools are the exception: they take a category at any level, and a parent covers everything under it. Search `IronCastle` when you do not know where something lives, and a leaf when you do. Filenames include the `.md` extension and are only unique within their leaf.

## Reading

`document_index` lists a leaf's documents with their summaries. Given `IronCastle` it is refused with the list of leaves, which is a quick way to see them. `get_document` reads one document in full. `get_document_summary` reads only a document's summary, which is useful after a search returns several chunks from one file.

## Searching

There are two search tools, for two different jobs.

`find_text` finds an exact word or phrase, ignoring case, in every document, indexed or not. Use it for anything with a name: a Townie, a place, a promise, an item, a phrase someone said. It returns matching files with short snippets labelled by section, which often answers the question without fetching anything. Pass `wholeWord: true` when a short name could sit inside a longer word. Pass `filename`, with a leaf category, to see where a term appears in one document.

`search_index` matches on meaning, over indexed documents only. Use it for questions like "when did Bron last work the orchard" or "what has anyone said about the castle's spirit". It returns chunk IDs with their leaf and filename. Fetch the text with `retrieve_search_results`, or the whole file with `get_document`. It returns 3 results by default; ask for up to 10 with `topK`.

Semantic search is poor at rare proper nouns. When the question is about a named thing, use `find_text` first.

## Writing

Every write tool refuses to run until `request_write_permission` has been called for that leaf. Permission covers one leaf only, so request it for each leaf you are about to write, and release it with `release_write_permission` when the write-up is done.

- `add_document` creates a document. Always set `indexed: true` unless there is a reason not to. Give it a one-line `summary`, since that is what `document_index` shows.
- `replace_section_text` finds an exact snippet of text within one section and replaces every occurrence of it, leaving the rest of the section alone. Prefer this over `replace_document_section` for a small change: only the changed snippet has to be written out, not the whole section.
- `replace_document_section`, `append_to_document` and `delete_document_section` change one part of a document and leave the rest alone. Reach for `replace_document_section` when more of a section is changing than a snippet-level find-and-replace can cover.
- `update_document` replaces the whole document. Use it only when most of the document changes, and then write the full replacement with nothing dropped.
- `archive_document` removes a document from view entirely: it no longer appears in listings, searches or reads, and cannot be brought back from here. Use it for fulfilled and cancelled promises, once anything worth keeping is in the session summary.
- Editing an indexed document re-indexes it automatically. `index_document` and `deindex_document` are rarely needed.

Every store document opens with a single `#` title, and everything else sits under `##` headers. Sections are what search chunks on, what results are labelled with, and what the section tools address, so anything that will be edited on its own gets its own `##` section. Examples are each Townie's entry in `unnamed-townies.md`, each room in a place document such as `keep.md`, and the favor table in `game-state.md`. Where a header name is ambiguous, qualify it with the headers above it, separated by `>`, for example `Outer Ward > Mill Pond`.

**Every fact has exactly one home.** Do not copy a value into a second document.

- **game-state.md**: the current date, the day's weather, day ticks, whether today's gift has been given, favor and hearts for every Townie, the calendar ahead, and active concerns in the wider castle. Favor and hearts live here and nowhere else.
- **character-bron.md**: Bron's stats, skills, satisfaction, pack, loans, home, promise progress as a list of ticks, backstory, and character notes. Promise ticks live here and nowhere else, with any carryover rates noted on the promise's line.
- **Promise documents**: terms (urgency, complexity, boxes, materials, carryover, standing rulings) and narrative. No ticks. The summary should say what the promise is and where it stands in the fiction in one line.
- **Townie documents**: static facts only. Identity, favored gifts, birthday, what they wear on their sleeve, and what has been established about them in play. No favor, no hearts, no roll history.
- **Place documents**: locations established at the table. The campaign document covers the castle's overall layout; the place documents record what has been seen and fixed in play. Each place lives in one document only; another place's document may mention it by name and point to it, but does not repeat its description.
- **Session summaries**: narrative only. What happened, not where things stand.