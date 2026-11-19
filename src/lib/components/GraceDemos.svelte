<script lang="ts">
  import { replaceState } from '$app/navigation';
  import { base } from '$app/paths';
  import { page } from '$app/state';
  import { BookOpen, FileText, Pause, Play, RotateCcw, X } from '@lucide/svelte';
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
  const previewAction = $derived(ended ? 'Replay preview' : playing ? 'Pause preview' : 'Play preview');
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

<section id="focus-session" aria-label="Focus demo">
  <!-- Preserve links to the former individual demos. -->
  <span id="blocking"></span>
  <span id="appearance"></span>
  <span id="statistics"></span>
  <span id="watch-demo"></span>

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
      <button class="icon-control" type="button" aria-label={previewAction} title={previewAction} onclick={togglePreview}>
        {#if ended}
          <RotateCcw size={16} aria-hidden="true" />
        {:else if playing}
          <Pause size={16} aria-hidden="true" />
        {:else}
          <Play size={16} aria-hidden="true" />
        {/if}
      </button>
    {/if}
    <details class="preview-transcript">
      <summary class="icon-control" aria-label="Preview transcript" title="Preview transcript"><FileText size={16} aria-hidden="true" /></summary>
      <p>“Just one quick check” appears over Instagram. The view pulls back to reveal Discord and Grace’s idle two-minute Writing timer. The user adds Discord and instagram.com to Writing’s rules, then presses Play. Discord closes and Chrome blocks Instagram. The countdown speeds ahead to the final tick and the start of Short Break. With break access enabled, the same Instagram page loads and Discord reopens. Statistics → This Week shows the completed round and a current streak of one day.</p>
    </details>
    <a class="icon-control" href="https://github.com/samir-scripts/grace/blob/dev/docs/user-guide.md" aria-label="User guide" title="User guide"><BookOpen size={16} aria-hidden="true" /></a>
  </div>

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
  .overview-meta { display: flex; align-items: flex-start; gap: 4px; }
  .icon-control { display: grid; place-items: center; flex: 0 0 auto; width: 44px; height: 44px; color: var(--color-muted); cursor: pointer; }
  .icon-control:hover, .icon-control:focus-visible { color: var(--color-foreground); background: var(--color-row-hover); }
  .icon-control::-webkit-details-marker { display: none; }
  details { margin-top: 8px; color: var(--color-muted); font-size: 0.75rem; line-height: 1.6; }
  .preview-transcript { flex: 1; min-width: 0; margin-top: 0; }
  .preview-transcript p { max-width: 65ch; }
  details p { margin: 4px 0 0; }
  summary { cursor: pointer; }
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
  dialog h2 { margin: 0; font-size: 0.85rem; font-weight: 500; line-height: 1.5; }
  .close { display: inline-flex; gap: 6px; align-items: center; min-height: 44px; padding: 4px 6px; font-size: 0.75rem; white-space: nowrap; cursor: pointer; }
  .close:hover { background: var(--color-row-hover); }
  dialog video { max-height: calc(100dvh - 105px); aspect-ratio: 3 / 2; background: #14191c; }
</style>
