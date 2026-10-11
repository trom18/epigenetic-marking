# Don't read the whole genome

*Epigenetic switches for an AI that works over a lot of documents.*

Tabak, 2026

[한국어](README_ko.md) | [Prior work](docs/prior-work.md)

Work with an AI for a while and the documents pile up. A code repository, design docs, meeting notes, tickets, old chats. Karpathy's [LLM wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) is one shape this takes: an LLM keeps a markdown wiki on top of raw sources. His note says that reading an index file first and then drilling into pages works surprisingly well at about a hundred sources and a few hundred pages. As the wiki grows, it says, you want proper search, and it names qmd as a good option, or a simple search script you write yourself.

A search engine makes finding faster. But the AI still goes looking every time. It searches, opens a file, reads, searches again. Yesterday it opened the same three files to answer the same kind of question. Today it starts from zero.

This note is an idea for not starting from zero. The hint comes from DNA.

## Search is not the slow part

I looked at my own sessions. The search itself finished quickly. The slow part was the round trip every time the AI called a tool. Each search, open, read, and search again added one, and a single question usually needed several.

So a faster search engine does not buy much. The win is in going to look less often.

## Why Everything is fast

On Windows, the built-in search is slow. Everything, a small free tool, finds files by name almost instantly. Both use an index. So why the difference?

Windows Search crawls folders and builds its own index of contents and properties. It is heavy, and on a big drive it is always catching up.

Everything does not crawl. NTFS already keeps a table of every file, the MFT. Everything reads it once, keeps only the names in memory, and never scans the disk again. It follows the change journal Windows writes anyway, the USN journal, and patches its list. A million files take about 75 MB of RAM.

Three lessons. Reuse a record that already exists. Keep the state small enough to live in memory. Never rescan; follow the changes.

Still, Everything answers "where is it?" Every query still scans the whole list. AI work needs one step more: knowing in advance what this piece of work needs.

## What DNA does

Almost every cell in your body carries the whole genome, copied in full at every division. Yet an eye cell reads eye genes and a heart cell reads heart genes. A cell does not search its genome each time it needs a protein.

Epigenetic marks make this work. The DNA sequence is left alone. Chemical marks on top of it decide which regions are open and which are closed. Closed regions are packed so tightly that the reading machinery can barely reach them. They are not ranked low. They are out of sight.

Five properties of these marks are worth stealing.

1. **They maintain themselves.** Proteins that read a mark write the same mark on neighbouring regions. Open stays open, closed stays closed, and both spread a little.
2. **Use sets them.** Active genes collect opening marks. Idle genes tend to collect closing marks. Nobody keeps a separate list. Living is the marking.
3. **They are inherited.** When a cell divides, the daughter gets the marks along with the genome. A heart cell's daughter is a heart cell from the start.
4. **They differ by cell type.** Same genome, different set of marks.
5. **They can be wiped.** Erase the marks and the cell drifts back toward a blank state. The genome is unharmed.

## Moving it to AI work

- **The genome is all the material.** Code, docs, wiki. The marks never edit it.
- **A cell type is a kind of work.** "Fix the billing bug" and "write the quarterly report" need different material.
- **A mark is how open one piece is for one kind of work.** 0 is closed, 1 is wide open. Marks live next to the material, not inside it.

Who writes the marks is the whole point. Not a person, and not a separate indexing job. The work writes them.

- What the AI opened, quoted, or edited opens up for that kind of work.
- Pieces linked to it (links, imports, same folder) open a little. That is the spreading.
- Pieces left unused close slowly.
- Superseded material, an old version or a reversed decision, is closed hard.

Just as Everything gets the MFT for free, the raw material for these marks already exists: the AI's own tool-call log. Which files it opened, which ones it changed. Nothing new has to be crawled.

When a question comes in:

1. Look only at what is open for this kind of work. It is usually small enough to read whole. No search.
2. If the answer is there, done. One round trip.
3. If not, search everything, and open what you found. Next time the same kind of question ends at step 1.

A new session or a sub-agent inherits the marks instead of starting blank. A kind of work never seen before starts from the closest known one at half strength, the way a stem cell differentiates toward a nearby type.

## How this differs from an index

| | index | epigenetic marks |
|---|---|---|
| records | where things are | what this work needs now |
| written by | a separate indexing job | the work itself |
| how many | one per corpus | one per kind of work |
| at question time | search everything | read what is open |
| over time | stays the same | unused pieces close |
| new session | starts from zero | inherits |

They do not compete. When the open region misses, you need the index. The marks make you call it less often.

## What I don't know yet

- This note is an idea. I have not measured how many round trips it saves on real work.
- Use breeds use. Popular pieces stay open, and a rare but important document can stay closed forever. Step 3, fall back to full search, must never be removed.
- How fast marks should fade, and how to split work into kinds, are open questions.
- Most of the parts already exist: the working-set model and caches in operating systems, Windows SysMain preloading the apps you use often, MemGPT paging context in and out of an LLM's window, Generative Agents ranking memories by recency and importance, ACT-R's memory activation that grows with use and fades with time. See [docs/prior-work.md](docs/prior-work.md). What the DNA picture adds is not a part but a checklist. Put the parts together and it shows what is missing: marks kept per kind of work, marks inherited by new sessions and sub-agents, closed regions that are not visited at all instead of ranked low, and old material closed on purpose instead of left to age out.

MIT license.
