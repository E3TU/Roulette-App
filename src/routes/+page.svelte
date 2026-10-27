<script lang="ts">
  const items = ['Black', 'Red', 'Green'] as const;

  const itemWidth = 112;
  const gap = 8;
  const visibleCount = 8;
  const step = itemWidth + gap;

  const spinDuration = 1600;
  const extraLoops = 12;

  let spinning = $state(false);
  let winner = $state<number | null>(null);
  let x = $state(0);
  let viewportWidth = $state(visibleCount * step);

  const total = items.length * step;
  const wrap = (v: number) => ((v % total) + total) % total;

  const copies = $derived(Math.max(2, Math.ceil(viewportWidth / total) + 1));
  const track = $derived(Array.from({ length: copies * items.length }, (_, i) => i));
  const activeSlot = $derived(Math.round((x + viewportWidth / 2 - itemWidth / 2) / step));

  const centredX = (i: number) => i * step + itemWidth / 2 - viewportWidth / 2;

  const effects: Record<string, string> = {
    Black: '#15161a',
    Red: '#b0232c',
    Green: '#1f7a4c'
  };

  const easeOutQuad = (t: number) => 1 - (1 - t) ** 2;

  let rafId = 0;

  function spin(target = Math.floor(Math.random() * items.length)) {
    if (spinning) return;

    spinning = true;
    winner = null;

    let d = wrap(centredX(target)) - x;
    if (d < 0) d += total;
    d += extraLoops * total;

    const from = x;
    const t0 = performance.now();

    const finish = () => {
      spinning = false;
      winner = target;
    };

    if (matchMedia('(prefers-reduced-motion: reduce)').matches) {
      x = wrap(from + d);
      finish();
      return;
    }

    const tick = (now: number) => {
      const p = Math.min(1, (now - t0) / spinDuration);
      x = wrap(from + d * easeOutQuad(p));
      if (p < 1) rafId = requestAnimationFrame(tick);
      else finish();
    };

    rafId = requestAnimationFrame(tick);
  }

  $effect(() => () => cancelAnimationFrame(rafId));
</script>

<div class="container">
  <h1>Roulette App</h1>

  <div
    class="roulette"
    bind:clientWidth={viewportWidth}
    style:width="{visibleCount * step - gap}px"
  >
    <div class="pointer"></div>

    <div class="track" style:transform="translate3d({-x}px, 0, 0)">
      {#each track as slot (slot)}
        <div
          class="cell"
          style:width="{itemWidth}px"
          style:margin-right="{gap}px"
          style:background={effects[items[slot % items.length]]}
          class:active={!spinning && slot === activeSlot}
        >
          {items[slot % items.length]}
        </div>
      {/each}
    </div>
  </div>

  <button onclick={() => spin()} disabled={spinning}>
    {spinning ? 'Spinning…' : 'Spin'}
  </button>

  <p class="result">
    {#if winner !== null}
      Result: <strong>{items[winner]}</strong>
    {:else}
      &nbsp;
    {/if}
  </p>
</div>

<style>
  .container {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 1.25rem;
  }
  h1 {
    color: var(--white);
    margin-top: 2rem;
    font-size: 3rem;
  }

  .roulette {
    position: relative;
    overflow: hidden;
    height: 88px;
    border-radius: 12px;
    -webkit-mask-image: linear-gradient(90deg, transparent, #000 8%, #000 92%, transparent);
    mask-image: linear-gradient(90deg, transparent, #000 8%, #000 92%, transparent);
  }

  .track {
    display: flex;
    will-change: transform;
  }

  .cell {
    flex: 0 0 auto;
    height: 88px;
    display: grid;
    place-items: center;
    font: 600 1rem/1 system-ui, sans-serif;
    color: #fff;
    transition: filter 180ms ease;
    border-radius: 15px;
  }

  .cell.active {
    filter: brightness(1.6);
    box-shadow: inset 0 0 0 3px #f5c542;
  }

  .pointer {
    position: absolute;
    inset-block: 0;
    left: 50%;
    width: 3px;
    margin-left: -1.5px;
    background: #f5c542;
    z-index: 1;
  }

  button {
    padding: 0.75rem 2rem;
    border: 0;
    border-radius: 999px;
    font: 600 1rem/1 system-ui, sans-serif;
    background: #f5c542;
    color: #000;
    cursor: pointer;
  }
  button:disabled {
    opacity: 0.5;
    cursor: default;
  }

  .result {
    min-height: 1.5em;
    color: var(--white);
  }
</style>