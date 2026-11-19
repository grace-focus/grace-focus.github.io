<script lang="ts">
  import { replaceState } from '$app/navigation';
  import { base } from '$app/paths';
  import { page } from '$app/state';
  import { Pause, Play, RotateCcw, X } from '@lucide/svelte';
  import { onMount } from 'svelte';

  const recording = `${base}/demos/focus-session.mp4`;
  const preview = `${base}/demos/focus-preview.mp4`;
  const poster = `${base}/demos/focus-session.webp`;
  const previewPoster = `${base}/demos/focus-preview.webp`;
  const fullRuntime = '1:14';
  const transcript = 'In Writing’s settings, add Discord and Chess to Blocked Applications and add youtube.com, instagram.com, and reddit.com to Blocked Websites. Save, return to the timer, and press Play. Grace stops Discord and Chess when you try to open them. The paired Chrome browser blocks all three sites. Fast-forward through the same round, then watch the timer enter Short Break. In Statistics, Today shows one completed round and two minutes of focus.';

  let trigger: HTMLAnchorElement;
  let previewPlayer: HTMLVideoElement;
  let dialog: HTMLDialogElement;
  let fullPlayer: HTMLVideoElement;
  let ready = $state(false);
  let playing = $state(false);
  let ended = $state(false);
  let previewFailed = $state(false);
  let inView = false;
  let reduceMotion = $state(false);
  let requestedPlay = false;
  let pausedByViewer = false;

  function syncPreview() {
    const shouldPlay = inView && !document.hidden && !dialog.open && !pausedByViewer &&
      !ended && !previewFailed && (!reduceMotion || requestedPlay);
    if (!shouldPlay) {
      previewPlayer.pause();
      return;
    }

    if (!previewPlayer.hasAttribute('src')) previewPlayer.src = preview;
    void previewPlayer.play().catch(() => {});
  }

  function togglePreview() {
    if (!previewPlayer.paused) {
      pausedByViewer = true;
      previewPlayer.pause();
      return;
    }

    if (ended) previewPlayer.currentTime = 0;
    ended = false;
    requestedPlay = true;
    pausedByViewer = false;
    if (!inView) previewPlayer.scrollIntoView({ block: 'nearest' });
    syncPreview();
  }

  function openDemo(playImmediately: boolean) {
    if (dialog.open) return;
    dialog.showModal();
    syncPreview();
    if (!fullPlayer.hasAttribute('src')) fullPlayer.src = recording;
    if (playImmediately) void fullPlayer.play().catch(() => {});
  }

  function activateDemo(event: MouseEvent) {
    // Preserve the ordinary video link for modified clicks and static output.
    if (event.altKey || event.ctrlKey || event.metaKey || event.shiftKey) return;
    event.preventDefault();
    openDemo(true);
  }

  function closeOnBackdrop(event: MouseEvent) {
    if (event.target !== dialog) return;
    const bounds = dialog.getBoundingClientRect();
    if (event.clientX < bounds.left || event.clientX > bounds.right ||
        event.clientY < bounds.top || event.clientY > bounds.bottom) {
      dialog.close();
    }
  }

  function demoClosed() {
    fullPlayer.pause();
    if (fullPlayer.readyState) fullPlayer.currentTime = 0;
    if (location.hash === '#watch-demo') {
      replaceState('#focus-session', page.state);
    }
    trigger.focus();
    syncPreview();
  }

  onMount(() => {
    const motion = window.matchMedia('(prefers-reduced-motion: reduce)');
    reduceMotion = motion.matches;
    previewPlayer.muted = true;

    const motionChanged = () => {
      reduceMotion = motion.matches;
      requestedPlay = false;
      syncPreview();
    };
    const openFromLink = () => {
      if (location.hash === '#watch-demo') openDemo(false);
    };
    const observer = new IntersectionObserver(([entry]) => {
      inView = entry.isIntersecting && entry.intersectionRatio >= 0.5;
      syncPreview();
    }, { threshold: 0.5 });

    observer.observe(previewPlayer);
    document.addEventListener('visibilitychange', syncPreview);
    motion.addEventListener('change', motionChanged);
    window.addEventListener('hashchange', openFromLink);
    ready = true;
    openFromLink();

    return () => {
      observer.disconnect();
      document.removeEventListener('visibilitychange', syncPreview);
      motion.removeEventListener('change', motionChanged);
      window.removeEventListener('hashchange', openFromLink);
      previewPlayer.pause();
      fullPlayer.pause();
    };
  });
</script>

<section id="focus-session" aria-labelledby="focus-session-title">
  <!-- Preserve links to the former individual demos. -->
  <span id="blocking"></span>
  <span id="appearance"></span>
  <span id="statistics"></span>
  <span id="watch-demo"></span>
  <h2 id="focus-session-title">Watch Grace protect a Writing session</h2>

  <a class="demo-trigger" bind:this={trigger} href={recording} aria-haspopup={ready ? 'dialog' : undefined} onclick={activateDemo}>
    <div class="frame">
      <!-- The visible preview is decorative here. Its action has a text label below. -->
      <!-- svelte-ignore a11y_media_has_caption -->
      <video
        bind:this={previewPlayer}
        aria-hidden="true"
        poster={previewPoster}
        muted
        playsinline
        loop={!reduceMotion}
        disablepictureinpicture
        disableremoteplayback
        preload="none"
        onplay={() => { playing = true; }}
        onpause={() => { playing = false; }}
        onended={() => { playing = false; ended = true; }}
        onerror={() => { previewFailed = true; }}
      ></video>
    </div>
    <span class="watch"><Play size={16} fill="currentColor" aria-hidden="true" /> Watch full demo <span class="runtime">{fullRuntime}</span></span>
  </a>

  <div class="overview-meta">
    {#if ready && !previewFailed}
      <button class="preview-control" type="button" onclick={togglePreview}>
        {#if ended}
          <RotateCcw size={14} aria-hidden="true" /> Replay preview
        {:else if playing}
          <Pause size={14} aria-hidden="true" /> Pause preview
        {:else}
          <Play size={14} aria-hidden="true" /> Play preview
        {/if}
      </button>
    {/if}
    <p class="build">macOS beta (1.7.1) · recorded in Tart</p>
    <nav aria-label="Get Grace">
      <a href="https://github.com/samir-scripts/grace/releases">Download</a>
      <a href="https://github.com/samir-scripts/grace/blob/dev/docs/user-guide.md">User guide</a>
    </nav>
  </div>

  <details>
    <summary>Preview transcript · 24 seconds</summary>
    <p>“Just one quick check” appears over Instagram. The view pulls back to reveal Discord and Grace’s idle two-minute Writing timer. The user adds Discord and instagram.com to Writing’s rules, then presses Play. Discord closes and Chrome blocks Instagram. The countdown speeds ahead to the final tick and the start of Short Break. With break access enabled, the same Instagram page loads and Discord reopens. Statistics → This Week shows the completed round and a current streak of one day.</p>
  </details>

  <!-- Native modal supplies Escape, the focus trap and inert background. -->
  <!-- svelte-ignore a11y_click_events_have_key_events a11y_no_noninteractive_element_interactions -->
  <dialog bind:this={dialog} aria-labelledby="watch-demo-title" onclick={closeOnBackdrop} onclose={demoClosed}>
    <header>
      <h2 id="watch-demo-title">A protected Writing session</h2>
      <button class="close" type="button" onclick={() => dialog.close()}>
        <X size={17} aria-hidden="true" /> Close
      </button>
    </header>
    <!-- This silent recording uses the descriptive transcript as its text alternative. -->
    <!-- svelte-ignore a11y_media_has_caption -->
    <video bind:this={fullPlayer} aria-labelledby="watch-demo-title" aria-describedby="watch-demo-transcript" controls playsinline {poster} preload="none"></video>
    <details>
      <summary>Transcript</summary>
      <p id="watch-demo-transcript">{transcript}</p>
    </details>
  </dialog>
</section>

<style>
  section { position: relative; min-width: 0; margin-bottom: 32px; scroll-margin-top: 16px; }
  section > span { position: absolute; top: 0; scroll-margin-top: 16px; }
  h2 { margin: 0 0 8px; font-size: 0.85rem; font-weight: 500; line-height: 1.5; }
  .demo-trigger { display: block; border-radius: 4px; text-decoration: none; }
  .frame {
    overflow: hidden;
    aspect-ratio: 3 / 2;
    background: var(--color-row-hover);
    border: 1px solid var(--color-rule);
    border-radius: 4px;
  }
  video { display: block; width: 100%; }
  .frame video { height: 100%; }
  .watch {
    display: flex;
    align-items: center;
    gap: 8px;
    min-height: 44px;
    padding: 0 4px;
    font-size: 0.85rem;
  }
  .demo-trigger:hover .watch { text-decoration: underline; text-underline-offset: 3px; }
  .runtime { margin-left: auto; color: var(--color-muted); font-variant-numeric: tabular-nums; }
  .overview-meta { display: flex; flex-wrap: wrap; gap: 0 14px; align-items: center; }
  button { display: inline-flex; gap: 6px; align-items: center; min-height: 36px; cursor: pointer; }
  button:hover { background: var(--color-row-hover); }
  .preview-control, .close { padding: 4px 6px; font-size: 0.75rem; }
  .build, details { color: var(--color-muted); font-size: 0.75rem; line-height: 1.6; }
  .build { margin: 0; }
  details { margin-top: 8px; }
  details p { margin: 4px 0 0; }
  summary { cursor: pointer; }
  nav { display: flex; gap: 14px; margin: 2px 0 2px auto; font-size: 0.75rem; }
  nav a { text-underline-offset: 3px; }
  :is(a, button, summary):focus-visible { outline: 2px solid var(--color-foreground); outline-offset: 3px; }

  dialog {
    width: min(100% - 24px, 980px);
    max-height: calc(100dvh - 24px);
    margin: auto;
    padding: 12px;
    border: 1px solid var(--color-rule);
    border-radius: 8px;
    background: var(--color-background);
    color: var(--color-foreground);
  }
  dialog::backdrop { background: rgba(14, 18, 25, 0.84); }
  dialog header { display: flex; align-items: center; justify-content: space-between; gap: 12px; margin-bottom: 8px; }
  dialog h2 { margin: 0; }
  .close { min-height: 44px; white-space: nowrap; }
  dialog video { max-height: calc(100dvh - 105px); aspect-ratio: 3 / 2; background: #14191c; }
</style>
