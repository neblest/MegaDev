<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { causeOf, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  let { ticket }: { ticket: number } = $props();

  // One id, one report. Then each report goes to exactly one problem:
  // a one-day move (the desk's own reading, desk-notes.json) is its own problem,
  // the rest are split by cause: parser or hint.
  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const problemOf = (r: Report) => (notes.oneDayMove.includes(r.id) ? 'one-day-move' : (causeOf(r, tenantOf(r)) ?? 'unplaced'));
  const rows = $derived(
    ['parser', 'hint', 'one-day-move', 'unplaced'].map((p) => ({ problem: p, n: merged.filter((r) => problemOf(r) === p).length })).filter((x) => x.n > 0),
  );
  const exported = (reports as Report[]).length;
</script>

<p>
  The export has {exported} rows. {exported - merged.length} are repeats of the same id, so
  <strong>{merged.length}</strong> reports are really merged into #{ticket}.
</p>

<table>
  <thead><tr><th>problem</th><th>reports</th></tr></thead>
  <tbody>
    {#each rows as x (x.problem)}
      <tr><td><code>{x.problem}</code></td><td>{x.n}</td></tr>
    {/each}
    <tr><td><strong>total</strong></td><td><strong>{rows.reduce((s, x) => s + x.n, 0)}</strong></td></tr>
  </tbody>
</table>
