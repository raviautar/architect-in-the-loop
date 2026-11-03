# Boxes

What exists right now: one row per box. The chief updates this file whenever a box is created, renamed or retired. History lives in the ledger, not here.

Resume a box from its directory, by its full session id, after checking that nothing already runs it:

```sh
pgrep -fl "claude.*<session-id>"
cd ~/workspace && claude --resume <session-id>
```

| window | box | claude session (name · id) | directory | status |
| --- | --- | --- | --- | --- |
| 0 | chief | chief · 00000000-0000-0000-0000-000000000000 | ~/workspace | active: routes work, reads the ledger |
| 1 | adhoc | adhoc · 00000000-0000-0000-0000-000000000001 | ~/workspace | active: whatever has no box of its own yet |
| 2 | search | search · 00000000-0000-0000-0000-000000000002 | ~/workspace | active: the search index and its API |
