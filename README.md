<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <script src="./console-capture.js">
    </script>
    <script src="./coi.js">
    </script>
    <!-- Resolves to the directory this page is served from, so the app works from a domain
             root and from a GitHub Pages project subpath alike. It lives here rather than being
             injected at deploy time because publish hashes index.html into
             service-worker-assets.js, and a later edit would leave the shipped file failing the
             integrity check the service worker installs it with. -->
    <base href="./" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no, viewport-fit=cover" />
    <title>
      Cut the Rope: Community Package
    </title>
    <meta name="description" content="Play Cut the Rope: Community Package directly in your browser. A fan-made collection of community made levels, available online with no installation required." />
    <meta name="keywords" content="Cut the Rope, Cut the Rope DX, Cut the Rope web, Cut the Rope browser, Om Nom, Cut the Rope fan game, Community Package, Cut the Rope Level Editor, Level Editor, Editor, editor, community, package" />
    <meta name="author" content="Cut the Rope Home" />
    <meta name="robots" content="index, follow" />
    <link rel="canonical" href="https://ctrhome.github.io/play/community-package/" />
    <link rel="icon" type="image/x-icon" href="./favicon.ico" />
    <link rel="apple-touch-icon" href="./icon-512.png" />
    <link rel="manifest" href="./manifest.webmanifest" />
    <meta name="theme-color" content="#202329" />
    <meta name="mobile-web-app-capable" content="yes" />
    <!-- Open Graph -->
    <meta property="og:type" content="website" />
    <meta property="og:title" content="Cut the Rope: Community Package - Play Online" />
    <meta property="og:description" content="Play Cut the Rope: Community Package directly in your browser. No installation required." />
    <meta property="og:url" content="https://ctrhome.github.io/play/community-package/" />
    <meta property="og:image" content="https://community-package.ctrhome.workers.dev/og-image.png" />
    <meta property="og:image:type" content="image/png" />
    <meta property="og:image:width" content="1200" />
    <meta property="og:image:height" content="600" />
    <meta property="og:image:alt" content="Cut the Rope: Community Package product card image" />
    <meta property="og:site_name" content="Cut the Rope: Community Package" />
    <!-- Twitter / X -->
    <meta name="twitter:card" content="summary_large_image" />
    <meta name="twitter:title" content="Cut the Rope: Community Package - Play Online" />
    <meta name="twitter:description" content="Play Cut the Rope: Community Package directly in your browser. No installation required." />
    <meta name="twitter:image" content="https://community-package.ctrhome.workers.dev/og-image.png" />
    <script type="application/ld+json">
      {
                &#34;@context&#34;: &#34;https://schema.org&#34;,
                &#34;@type&#34;: &#34;SoftwareApplication&#34;,
                &#34;name&#34;: &#34;Cut the Rope: Community Package&#34;,
                &#34;alternateName&#34;: &#34;Community Package&#34;,
                &#34;url&#34;: &#34;https://ctrhome.github.io/play/community-package/&#34;,
                &#34;image&#34;: &#34;https://community-package.ctrhome.workers.dev/og-image.png&#34;,
                &#34;description&#34;: &#34;A fan-made collection of Cut the Rope levels that can be played directly in a web browser.&#34;,
                &#34;applicationCategory&#34;: &#34;GameApplication&#34;,
                &#34;operatingSystem&#34;: &#34;Web browser&#34;,
                &#34;isAccessibleForFree&#34;: true
            }
    </script>
    <style>
      html,
            body {
                height: 100%;
                margin: 0;
                background: #000;
                overflow: hidden;
                -webkit-user-select: none;
                user-select: none;
                -webkit-tap-highlight-color: transparent;
                touch-action: none;
                overscroll-behavior: none;
            }
            /* The canvas is the viewport. The game reads this box back and lays itself out to
               whatever shape it is, so constraining it here would be constraining the game.
               Fixed positioning rather than viewport units because a mobile browser collapsing
               its URL bar changes the visible area without changing 100vh. */
            #game {
                display: block;
                position: fixed;
                inset: 0;
                width: 100%;
                height: 100%;
            }
            /* Shares the canvas&#39;s box. A cutscene is genuinely fixed-aspect, so it keeps
               object-fit rather than stretching. z-index puts it over the game but under the
               splash, so a cutscene can never show through the loading screen. */
            #movie {
                display: block;
                position: fixed;
                inset: 0;
                width: 100%;
                height: 100%;
                z-index: 5;
                background: #000;
                object-fit: contain;
            }
            #movie[hidden] {
                display: none;
            }
            #splash {
                position: fixed;
                inset: 0;
                display: flex;
                flex-direction: column;
                align-items: center;
                justify-content: flex-start;
                gap: 20px;
                overflow-y: auto;
                touch-action: pan-y;
                box-sizing: border-box;
                padding: calc(24px + env(safe-area-inset-top, 0px))
                    calc(24px + env(safe-area-inset-right, 0px))
                    calc(24px + env(safe-area-inset-bottom, 0px))
                    calc(24px + env(safe-area-inset-left, 0px));
                background: #202329;
                color: #e6e6e6;
                font-family:
                    system-ui,
                    -apple-system,
                    &#34;Segoe UI&#34;,
                    Roboto,
                    sans-serif;
                z-index: 10;
                transition: opacity 0.25s ease-out;
            }
            /* Faded rather than removed, so the canvas underneath is already painted when the
               splash goes; pointer-events keeps the invisible overlay from eating taps. */
            #splash.hidden {
                opacity: 0;
                pointer-events: none;
            }
            #splash &gt; * {
                flex-shrink: 0;
            }
            #splash::before,
            #splash::after {
                content: &#34;&#34;;
                margin: auto;
            }
            #splash img {
                width: 96px;
                height: 96px;
            }
            #splash p,
            #splash h1 {
                margin: 0;
                max-width: min(100%, 40rem);
                overflow-wrap: anywhere;
                text-align: center;
            }
            #splash h1 {
                font-size: 1.25rem;
                font-weight: 600;
            }
            #splash #splash-hint {
                max-width: min(100%, 320px);
                font-size: 0.8rem;
                line-height: 1.45;
                opacity: 0.6;
            }
            #splash .spinner {
                width: 36px;
                height: 36px;
                border-radius: 50%;
                border: 4px solid #3a3f47;
                border-top-color: #e6e6e6;
                animation: splash-spin 0.8s linear infinite;
            }
            #splash .spinner[hidden] {
                display: none;
            }
            @keyframes splash-spin {
                to {
                    transform: rotate(360deg);
                }
            }
            #start {
                padding: 12px 28px;
                font-size: 1rem;
                border: 0;
                border-radius: 8px;
                cursor: pointer;
            }
            #start[hidden],
            #resume[hidden] {
                display: none;
            }
            #resume {
                padding: 12px 28px;
                font-size: 1rem;
                border: 0;
                border-radius: 8px;
                cursor: pointer;
            }
            #export-log {
                padding: 8px 18px;
                font-size: 0.85rem;
                border: 1px solid #3a3f47;
                border-radius: 8px;
                background: transparent;
                color: #e6e6e6;
                cursor: pointer;
                opacity: 0.75;
            }
            #export-log:hover {
                opacity: 1;
            }
            #export-log[hidden],
            #export-log-status[hidden] {
                display: none;
            }
            #export-log-status {
                font-size: 0.8rem;
                opacity: 0.6;
            }
            #update {
                padding: 20px 24px;
                border: 0;
                border-radius: 10px;
                background: #202329;
                color: #e6e6e6;
                font-family:
                    system-ui,
                    -apple-system,
                    &#34;Segoe UI&#34;,
                    Roboto,
                    sans-serif;
                box-shadow: 0 8px 32px rgb(0 0 0 / 50%);
            }
            #update::backdrop {
                background: rgb(0 0 0 / 50%);
            }
            #update p {
                margin: 0 0 16px;
            }
            #update-actions {
                display: flex;
                gap: 8px;
                justify-content: flex-end;
            }
            #update button {
                padding: 8px 18px;
                font-size: 0.95rem;
                border: 0;
                border-radius: 6px;
                cursor: pointer;
            }
    </style>
  </head>
  <body>
    <canvas id="game">
    </canvas>
    <video id="movie" playsinline="" hidden="">
    </video>
    <div id="splash">
      <img src="./icon-512.png" width="96" height="96" alt="Cut the Rope: Community Package icon" decoding="async" />
      <div class="spinner" id="splash-spinner">
      </div>
      <p id="splash-progress">
        Loading…
      </p>
      <h1>
        Cut the Rope: Community Package
      </h1>
      <button id="start" hidden="">
        Play
      </button>
      <p id="splash-hint" role="status" hidden="">
        Taking too long? You might want to check your internet speed, or
                reload the page.
      </p>
      <p id="unsupported-error" role="alert" hidden="">
        This browser is too old to run the game. iOS and iPadOS need
                version 15 or newer.
      </p>
      <p id="boot-error" role="alert" hidden="">
        The game could not start. Check your connection and reload the
                page.
      </p>
      <p id="context-lost-error" role="alert" hidden="">
        The browser reset the game&#39;s graphics. Reload to continue -
                previously saved progress should still be there, but the current
                level will restart.
      </p>
      <button id="resume" hidden="">
        Resume
      </button>
      <button id="export-log">
        Export log
      </button>
      <p id="export-log-status" role="status" hidden="">
      </p>
    </div>
    <dialog id="update">
      <p>
        A new version of the game is ready.
      </p>
      <div id="update-actions">
        <button id="update-later">
          Later
        </button>
        <button id="update-now">
          Update
        </button>
      </div>
    </dialog>
    <script type="module" src="./pwa.js">
    </script>
    <script type="module">
      // Loading has two stretches with no reporting of their own — compiling what was
            // downloaded, and decoding the audio — so a slow device and a wedged one look alike
            // from here. The advice waits out a comfortably slow start rather than accusing
            // every one of them of being broken.
            const HINT_DELAY_MS = 30000;

            const start = document.getElementById(&#34;start&#34;);
            const splash = document.getElementById(&#34;splash&#34;);
            const spinner = document.getElementById(&#34;splash-spinner&#34;);
            const hint = document.getElementById(&#34;splash-hint&#34;);
            const progress = document.getElementById(&#34;splash-progress&#34;);
            const hintTimer = setTimeout(() =&gt; {
                hint.hidden = false;
            }, HINT_DELAY_MS);

            // The advice is for a slow start, not a stopped one. A boot that has already
            // failed says so on the splash, and this arriving half a minute later would talk
            // over it with a suggestion that no longer applies.
            globalThis.ctrdxStopHint = () =&gt; {
                clearTimeout(hintTimer);
                hint.hidden = true;
            };
            globalThis.ctrdxReady = () =&gt; {
                globalThis.ctrdxStopHint();
                spinner.hidden = true;
                progress.remove();
                start.hidden = false;
            };
            // The failure seam lives here, not in main.js, because main.js reaches its own
            // first statement only after its static imports have been fetched and evaluated.
            // An import that fails takes main.js down before anything it installs exists.
            // This module imports nothing and runs first, so it is the earliest point that
            // can still speak. The first failure wins: a failed boot knocks over what follows
            // it, and the cascade is noise once the cause is on screen.
            let reported = false;
            globalThis.ctrdxFail = (id, detail) =&gt; {
                if (reported) {
                    return;
                }
                reported = true;

                console.error(`ctrdx-boot-failure: ${id}`, detail ?? &#34;&#34;);
                globalThis.ctrdxStopHint();
                spinner.hidden = true;
                progress.hidden = true;
                start.hidden = true;
                splash.classList.remove(&#34;hidden&#34;);
                document.getElementById(id)?.removeAttribute(&#34;hidden&#34;);
            };

            const onBootRejection = (event) =&gt;
                globalThis.ctrdxFail(&#34;boot-error&#34;, event.reason);
            const onBootError = (event) =&gt;
                globalThis.ctrdxFail(
                    &#34;boot-error&#34;,
                    event.error ?? event.message,
                );
            globalThis.addEventListener(&#34;unhandledrejection&#34;, onBootRejection);
            globalThis.addEventListener(&#34;error&#34;, onBootError);

            // Called once the game is running. After boot a rejection belongs to the running
            // game, and covering a playable session with the boot error screen would cost the
            // player more than the rejection did.
            globalThis.ctrdxBootComplete = () =&gt; {
                globalThis.removeEventListener(
                    &#34;unhandledrejection&#34;,
                    onBootRejection,
                );
                globalThis.removeEventListener(&#34;error&#34;, onBootError);
                // Only now, on a page that was already isolated without it: the worker claims
                // the page when it activates, and a controller swap during startup lands in
                // the middle of the runtime fetching its own worker scripts.
                globalThis.ctrdxInstallWorker?.();
            };

            start.addEventListener(&#34;click&#34;, async () =&gt; {
                await globalThis.ctrdxStart();
                splash.classList.add(&#34;hidden&#34;);
            });

            // The page owns this rather than the game: a run that never reached the game is
            // exactly the run whose log is worth having, and the worker that would have served
            // the request is the thing that did not start. Reading the database directly leaves
            // the button working whatever happened behind it.
            const exportLog = document.getElementById(&#34;export-log&#34;);
            const exportStatus = document.getElementById(&#34;export-log-status&#34;);

            // The splash is what a player looks at while waiting, so an outcome that stayed there
            // would still be sitting under the Play button on the next launch.
            const STATUS_LINGER_MS = 5000;
            let statusTimer = null;

            function showExportStatus(text, linger) {
                clearTimeout(statusTimer);
                statusTimer = null;
                exportStatus.textContent = text;
                exportStatus.hidden = false;
                if (linger) {
                    statusTimer = setTimeout(() =&gt; {
                        exportStatus.hidden = true;
                        statusTimer = null;
                    }, STATUS_LINGER_MS);
                }
            }

            exportLog.addEventListener(&#34;click&#34;, async () =&gt; {
                exportLog.disabled = true;
                // Not timed: the export replaces it as soon as it knows the outcome, and a
                // progress note that cleared itself mid-export would read as a failure.
                showExportStatus(&#34;Collecting…&#34;, false);
                try {
                    const { exportZip } = await import(&#34;./log.js&#34;);
                    // console-capture.js may still be handing its first entries to the store.
                    await globalThis.ctrdxCaptureSettled?.();
                    const saved = await exportZip();
                    showExportStatus(
                        saved === 0
                            ? &#34;No logs recorded yet.&#34;
                            : `Exported ${saved} ${saved === 1 ? &#34;log&#34; : &#34;logs&#34;}.`,
                        true,
                    );
                } catch (error) {
                    showExportStatus(
                        `Could not export the log: ${error}`,
                        true,
                    );
                } finally {
                    exportLog.disabled = false;
                }
            });

            // The application is imported here rather than through its own script tag so that
            // this bootstrap owns the promise for its whole module graph. A module that fails
            // to *download* raises an error event at its script element, and that event does
            // not bubble - so a plain window listener never sees it, and the page would sit on
            // its spinner with nothing reported. Verified in Chromium: with a separate script
            // tag a 404 on one of main.js&#39;s static imports reached no handler at all, while
            // this form reports it. Listening in the capture phase would also catch it, but it
            // catches every failed resource with it - a broken splash icon would then report a
            // boot failure over a game that started perfectly well, which was likewise
            // confirmed. A rejection here is precise about what actually failed.
            import(&#34;./main.js&#34;).catch((error) =&gt;
                globalThis.ctrdxFail(&#34;boot-error&#34;, error),
            );
    </script>
  </body>
</html>
