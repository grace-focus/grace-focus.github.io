<script lang="ts">
  import { base } from '$app/paths';
  import { ArrowLeft } from '@lucide/svelte';
</script>

<svelte:head>
  <link rel="canonical" href="https://grace-focus.github.io/privacy/" />
  <title>privacy policy · grace.</title>
  <meta name="description" content="Privacy policy for the grace. Focus Companion for Chrome and Firefox." />
</svelte:head>

<div class="policy">
  <nav aria-label="Privacy navigation">
    <a class="back-link" href={`${base}/`}><ArrowLeft aria-hidden="true" />back to grace.</a>
  </nav>
  <header>
    <p class="policy-name">grace. Focus Companion · Chrome &amp; Firefox</p>
    <h1>privacy policy</h1>
  </header>
  <article aria-label="grace. Focus Companion Privacy Policy">
    <p><strong>Chrome effective date:</strong> November 11, 2026. <strong>Firefox addition:</strong> November 13, 2026.</p>
    <p><strong>Applies to:</strong> grace. Focus Companion for Google Chrome desktop (Chrome 116 or later; extension 1.1.0 and later) and Firefox desktop (Firefox 115 or later; extension 1.1.1 and later). The current disclosures below describe Chrome extension 1.1.2 and later and Firefox extension 1.1.1 and later; earlier Chrome versions are addressed where their handling differs. Grace desktop has its own local application behavior.</p>
    <h2>Summary</h2>
    <p>The extension connects your browser to the Grace desktop app on the same computer to pair a browser profile and apply or release website blocking rules chosen in Grace. It has no account system, remote service, advertising, analytics, or telemetry. It does not deliberately send browsing history or visited page URLs to Grace or a remote service.</p>
    <h2>Data the extension handles</h2>
    <p>The extension stores the following in browser-managed extension-local storage on your device:</p>
    <ul>
    <li>Your consent decision.</li>
    <li>A pairing token, configured local connection port, and persistent profile identifier. Automatic pairing can create and store these values after consent. Treat these credentials as sensitive.</li>
    <li>Cached appearance colors and animation preference so the options page can match Grace&#x27;s theme.</li>
    </ul>
    <p>After you select <strong>Accept &amp; connect</strong>, the extension and Grace exchange authentication proofs over a local connection to <code>127.0.0.1</code> on the configured port. The extension sends its persistent profile identifier, policy acknowledgements, and recurring connection checks. Grace sends the blocking policy, including domains configured in Grace, over the same authenticated connection. Current versions do not send browser or installation metadata in that authentication exchange. Earlier Chrome versions may send browser kind, installation type and whether the extension may be disabled, and may send browser API error text to Grace; such text can contain sensitive information. Current versions record connection and rule-processing diagnostics locally, including stages and counts, and close the connection on a policy error.</p>
    <p>Both browsers manage blocking rules installed through their Declarative Net Request APIs separately from extension-local storage. When applying active rules, the extension may inspect open tab URLs locally and replace matching tabs with its bundled blocked page. That page&#x27;s query string contains the full original URL, which may remain in browser-managed tab, session, or history data. The page displays the matched hostname locally. The extension does not read browser history through a history API or deliberately include visited URLs in the connection to Grace.</p>
    <p>Before consent, the options page may request Grace&#x27;s local appearance settings through the native host so it can display a matching theme. This local request does not pair the browser or send browsing data.</p>
    <h2>Permissions</h2>
    <ul>
    <li><strong>Storage:</strong> save consent, local pairing settings, profile identifier, and cached appearance.</li>
    <li><strong>Alarms (Chrome):</strong> retry the local connection periodically.</li>
    <li><strong>Declarative Net Request:</strong> add, replace, or remove locally enforced rules for configured blocked domains.</li>
    <li><strong>Tabs:</strong> inspect open tabs locally and redirect tabs matching an active blocking policy to the extension&#x27;s blocked page.</li>
    <li><strong>Native messaging:</strong> communicate with Grace on the same computer for automatic pairing and appearance settings.</li>
    <li><strong>HTTP and HTTPS site access:</strong> apply configured blocking rules across <code>http://</code> and <code>https://</code> origins because you may choose any domain to block. The extension does not read or upload page contents.</li>
    </ul>
    <h2>Consent, withdrawal, and retention</h2>
    <p>The extension remains disconnected until you accept the disclosure in its options page. If you decline, it saves that choice locally and does not pair or exchange focus-session data. Firefox may also present its own data permissions; the extension&#x27;s local consent choice still applies.</p>
    <p>Select <strong>Stop sharing and disconnect</strong> to close the connection and stop future sharing. Withdrawal does not necessarily remove rules already installed in the browser: they can remain until Grace sends an authenticated release or replacement policy, or the extension is uninstalled. The consent decision, pairing values, profile identifier, and cached appearance remain in extension-local storage after withdrawal. Remove the extension or clear its data in the browser to remove that stored data. Removing the extension also removes its blocking rules.</p>
    <p>Grace stores its own focus profiles and settings locally on your computer; manage or delete those in Grace. Chrome and Firefox manage their own extension storage, rule storage, tab and history data, and backup lifecycles.</p>
    <h2>Sharing, updates, and security</h2>
    <p>The extension exchanges data with Grace&#x27;s local service and does not automatically contact this website or a publisher server. The pairing token authenticates the local connection; do not share it. Anyone with access to your computer or browser profile may be able to access local extension data.</p>
    <p>The Firefox companion is distributed unlisted and updated manually through Grace&#x27;s desktop-managed installation; it is not a public AMO listing with automatic AMO updates.</p>
    <p>Opening this policy or another page on the website sends ordinary web requests to its GitHub Pages host. The website does not add analytics or remote fonts. This policy&#x27;s public address is <a href="https://grace-focus.github.io/privacy/">grace-focus.github.io/privacy/</a>.</p>
    <h2>Changes and contact</h2>
    <p>This policy will be updated if the extension&#x27;s data handling changes. For support or privacy questions, open an issue at <a href="https://github.com/samir-scripts/grace/issues">github.com/samir-scripts/grace/issues</a>. Do not include pairing tokens or other sensitive information in an issue.</p>
  </article>
</div>

<style>
  .policy { width: 100%; max-width: 640px; min-width: 0; }
  nav { border-bottom: 1px solid var(--color-rule); }
  .back-link { display: inline-flex; align-items: center; gap: 12px; min-height: 44px; padding: 10px 12px; font-size: 14px; line-height: 1.5; text-decoration: none; }
  .back-link :global(svg) { width: 16px; height: 16px; flex: 0 0 auto; }
  .back-link:hover, .back-link:focus-visible { background: var(--color-row-hover); }
  header { padding: 24px 12px 16px; }
  .policy-name { margin: 0 0 8px; color: var(--color-muted); font-size: 12px; line-height: 1.5; }
  h1 { margin: 0; font-size: 20px; line-height: 1.5; font-weight: 400; }
  article { padding: 0 12px 24px; font-size: 13px; line-height: 1.8; overflow-wrap: anywhere; }
  h2 { margin: 24px 0 8px; color: var(--color-muted); font-size: 12px; line-height: 1.5; font-weight: 400; text-transform: lowercase; }
  p { margin: 0 0 16px; }
  article > :last-child { margin-bottom: 0; }
  ul { margin: 0 0 16px; padding-left: 20px; list-style: disc; }
  li { padding-left: 4px; margin: 0 0 8px; }
  strong { font-weight: 600; }
  code { font: inherit; }
  a { color: inherit; text-decoration: underline; text-underline-offset: 3px; }
  a:hover { color: var(--color-wave-coral); }
  a:focus-visible { outline: 2px solid var(--color-foreground); outline-offset: 4px; }
</style>
