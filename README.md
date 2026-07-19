# ProphetSignals — Public Ledger

Every call The Prophet makes lands here: timestamped, hash-chained, graded by code against official market data. Hits and misses, forever. No edits, no deletions — the hash chain makes rewriting history detectable by anyone.

## Integrity notice — 2026-07-19

V1 incorrectly calculated the XLE call at sequence 13 as `+9.625R` by using its latest tightened stop as the risk denominator. Initial risk must use entry 55.86 versus the original stop 53.90, making the correct result `+0.786R`.

The original event has not been edited. Append-only correction `MC-20260719-XLE-RISK-DENOMINATOR` at sequence 16 carries both values and the reason. New official calls, discretionary stop changes, and performance promotion are paused while Methodology 1.1 completes validation.

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

Methodology 1.1: entry = first eligible official session open after publication; R always uses the original stop; stop updates become effective only at a future eligible session; opening gaps fill at the open; stop counts before target on same-day ambiguity; expiry at horizon needs ≥0.5R to classify as a win. Stops only ever tighten.

**Not investment advice.** This is impersonal market commentary of general circulation, identical for every reader.
