---
license: cc-by-4.0
task_categories:
  - graph-ml
  - feature-extraction
language:
  - en
tags:
  - steam
  - games
  - network-analysis
  - co-review
  - graph-data
pretty_name: Steam Co-Review Network
---

# Steam Co-Review Network

A filtered graph of **25,996 Steam games and 861,232 weighted links**, supplied
alongside a separate **82,928-game catalog**. Links describe retained shared
reviewer counts between games. The catalog is larger than the graph; its games
are not all graph nodes.

## Available files

Counts and SHA-256 checksums were verified on September 14, 2026. The data bytes
are unchanged by this documentation correction.

| Artifact | GitHub filename | Hugging Face filename | Contents | Bytes |
|---|---|---|---|---:|
| Filtered graph | `steam_network.json` | `steam_network_full.json` | 25,996 nodes; 861,232 links | 42,249,061 |
| Game catalog | `steam_all_2005.json` | `steam_all_2005.json` | 82,928 packed game records, 2005–2025 | 6,482,810 |
| Stored layout | `steam_force_layout.json` | `steam_force_layout.json` | 8,905 positions keyed by game ID | 257,523 |

The Hugging Face filename `steam_network_full.json` is a historical name for the
same filtered graph available on GitHub as `steam_network.json`. It does **not**
contain the previously advertised 48,362-node / 33,041,298-link graph.

The graph's SHA-256 is:

```text
512eafbf312743aa7768a9c1c0fc17b49089bb85640c9bfe8e5265e7d8d9dd2d
```

`SHA256SUMS` covers the available data files using this platform's filenames.
`DATA_PROVENANCE.json` records the cross-platform names, checksums, measured
counts, and the identity of the unavailable historical object.

### Unavailable historical full graph

GitHub's `steam_network_full.json` is only a 135-byte Git LFS pointer, not readable
JSON. The referenced **1,423,482,375-byte** object could not be recovered from the
inspected copies. Its SHA-256 is
`f73197543ac056b56ffdb4ec73bdc94684721b47c46072d42b0cc9479f2d502d`.
The pointer and Git history are retained as provenance. The old 48,362-node /
33,041,298-link figures are historical claims about that missing object, not
measurements of the current download. Do not substitute the smaller graph and
label it as a recovered full graph.

## Read the graph

After downloading the graph, use Python's standard library. Set the filename to
`steam_network_full.json` for the Hugging Face download only:

```python
import hashlib
import json
from pathlib import Path

path = Path("steam_network.json")  # GitHub; see the Hugging Face alias above
raw = path.read_bytes()
assert hashlib.sha256(raw).hexdigest() == "512eafbf312743aa7768a9c1c0fc17b49089bb85640c9bfe8e5265e7d8d9dd2d"
graph = json.loads(raw)
assert len(graph["nodes"]) == 25996
assert len(graph["links"]) == 861232
```

Each node contains a Steam app ID (`id`), title, release year, rating label,
positive-review ratio, review count, price, genre/tag indices, and developer.
Graph genre/tag indices refer to `meta.genres` and `meta.tags`.

Each link has integer `source` and `target` indices into `nodes`, plus an integer
`weight`. Verified weights range from **5 to 694,377**. Node IDs and undirected
pairs are unique, and every endpoint is in range in the distributed graph.

## Filtering and interpretation

The graph's stored metadata records `top_k=50`, `min_shared=5`,
`max_user_games=75`, and `min_game_reviews=50`. The corresponding v2 builder
retains the **union** of each game's top-50 neighbor selections. This is not a
maximum-degree guarantee: a game may also be selected by many other games.

The historical builders cap the games retained per reviewer and prune partial
user/edge accumulators to limit memory. Those operations discard information,
so edge weights represent retained co-review counts rather than guaranteed
complete audience overlap. The graph is a filtered analytical snapshot, not a
complete Steam social graph, ownership network, or measure of all players.

## Catalog and layout

Catalog rows use this nine-field format:

```text
[name, year, ratio, reviews, price, ratingIdx, genreIdxs, tagIdxs, developer]
```

The catalog's `ratings`, `genres`, and `tags` lookup tables are top-level arrays.
Packed catalog rows do not include Steam app IDs; a direct ID join with graph
nodes is unavailable. Rating labels are derived Steam-style categories, not an
independently verified copy of every game's current store rating.

The stored layout contains 8,905 positions. Its metadata records an edge-weight
threshold of 100 and 80 iterations. Current `compute_layout.py` uses different
defaults and outputs additional community information absent from this file;
it is not evidence of an exact rebuild of the stored layout. Match positions
to graph nodes by Steam app ID, not by array index.

## Sources and reproduction limits

- Game metadata: [FronkonGames Steam Games Dataset](https://huggingface.co/datasets/FronkonGames/steam-games-dataset).
  The packaged catalog identifies a January 2026 input snapshot.
- Review source: [artermiloff Steam Games Reviews 2024](https://www.kaggle.com/datasets/artermiloff/steam-games-reviews-2024).
  Historical project notes describe an upstream corpus of 128 million reviews
  across roughly 80,000 games, covering 2012–June 2024. Those are source-corpus
  figures, not counts measured from this filtered graph or proof that every
  upstream review contributed to it.

Exact historical input revision hashes are not recorded in this repository.
The graph and catalog are derived outputs; downloading a newer upstream
snapshot would create a new version, not recover the missing historical object.

The Python files preserve the processing approach. They require external raw
review CSVs and enriched metadata at the paths configured in each script:

- `build_network_v2.py`: filtered co-review construction.
- `enrich_data.py`: nine-field catalog construction and graph enrichment.
- `build_network_full.py`: a less-filtered historical build; its missing output
  has not been reconstructed and its processing still includes caps/pruning.
- `compute_layout.py`: layout and optional graph trimming; review its input and
  output paths before running because it can overwrite a network file.
- `build_all_games.py`: legacy ten-field catalog writer, incompatible with the
  current packed catalog and notebook; do not use it to replace this catalog.

The raw review corpus is not included. No new graph was generated for this
correction. Schema and checksum checks establish the distributed artifact's
identity and internal consistency, not the accuracy of every upstream record.

## Distribution

- [GitHub: data-poems/steam-network-data](https://github.com/data-poems/steam-network-data)
- [Hugging Face: lukeslp/steam-co-review-network](https://huggingface.co/datasets/lukeslp/steam-co-review-network)
- [Kaggle: lucassteuber/steam-universe-network](https://www.kaggle.com/datasets/lucassteuber/steam-universe-network)

## License and author

CC-BY-4.0. Retain source attribution and consult each upstream source's terms
when reusing its original material.

Luke Steuber — [lukesteuber.com](https://lukesteuber.com)
