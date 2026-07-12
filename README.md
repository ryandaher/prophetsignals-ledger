# ProphetSignals — Public Ledger

Every call The Prophet makes lands here: timestamped, hash-chained, graded by code against official market data. Hits and misses, forever. No edits, no deletions — the hash chain makes rewriting history detectable by anyone.

## Files
- `ledger.jsonl` — the append-only ledger. Each line embeds the SHA-256 of the previous line.
- `stats.json` — derived stats (hit rate, avg R, Brier score) + current chain head hash.
- `latest.json` — the most recent daily run (worldview).
- `head.txt` (+ `.ots`) — chain head, anchored via OpenTimestamps when present.

## Verify the chain yourself
```js
// node verify.mjs
import { createHash } from 'node:crypto';
import { readFileSync } from 'node:fs';
const sha = s => createHash('sha256').update(s).digest('hex');
const canon = o => Array.isArray(o) ? '['+o.map(canon).join(',')+']'
  : o!==null&&typeof o==='object' ? '{'+Object.keys(o).sort().map(k=>JSON.stringify(k)+':'+canon(o[k])).join(',')+'}'
  : JSON.stringify(o);
let prev = sha('prophetsignals-genesis-2026-07-12');
for (const line of readFileSync('ledger.jsonl','utf8').split('\n').filter(Boolean)) {
  const { hash, ...body } = JSON.parse(line);
  if (body.prev_hash !== prev || sha(canon(body)) !== hash) throw new Error('CHAIN BROKEN at seq '+body.seq);
  prev = hash;
}
console.log('chain OK, head', prev);
```

Grading rules: entry = next session's official open after publication; stop counts before target on same-day ambiguity (worst case against us); expiry at horizon needs ≥0.5R to count as a win. Stops only ever tighten.

**Not investment advice.** This is impersonal market commentary of general circulation, identical for every reader.
