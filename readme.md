# Lighthouse  [![Build Status](https://travis-ci.org/GoogleChrome/lighthouse.svg?branch=master)](https://travis-ci.org/GoogleChrome/lighthouse) [![Coverage Status](https://coveralls.io/repos/github/GoogleChrome/lighthouse/badge.svg?branch=master)](https://coveralls.io/github/GoogleChrome/lighthouse?branch=master)

> Lighthouse analyzes web apps and web pages, collecting modern performance metrics and insights on developer best practices.

**Lighthouse requires Chrome 56 or later.**

## Installation

### Chrome extension

[Install from the Chrome Web Store](https://chrome.google.com/webstore/detail/lighthouse/blipmdconlkpinefehnmjammfjpmpbjk)

### Node CLI [![NPM lighthouse package](https://img.shields.io/npm/v/lighthouse.svg)](https://npmjs.org/package/lighthouse)

**Requires Node v6+.**

```sh
npm install -g lighthouse
# or use yarn:
# yarn global add lighthouse
```

## Running Lighthouse

### Chrome extension

Check out the quick-start guide: <http://bit.ly/lighthouse-quickstart>

### CLI

Kick off a run by passing `lighthouse` the URL to audit:

```sh
lighthouse https://airhorner.com/
```

By default, Lighthouse writes the report to an HTML file. You can control the output format by passing flags.

#### CLI options

```sh
$ lighthouse --help

lighthouse <url>

Logging:
  --verbose  Displays verbose logging                                                                                                      [boolean]
  --quiet    Displays no progress, debug logs or errors                                                                                    [boolean]

Configuration:
  --save-assets                  Save the trace contents & screenshots to disk                                                             [boolean]
  --save-artifacts               Save all gathered artifacts to disk                                                                       [boolean]
  --list-all-audits              Prints a list of all available audits and exits                                                           [boolean]
  --list-trace-categories        Prints a list of all required trace categories and exits                                                  [boolean]
  --additional-trace-categories  Additional categories to capture with the trace (comma-delimited).
  --config-path                  The path to the config JSON.
  --chrome-flags                 Custom flags to pass to Chrome (space-delimited). For a full list of flags, see
                                 http://peter.sh/experiments/chromium-command-line-switches/.                                          [default: ""]
  --perf                         Use a performance-test-only configuration                                                                 [boolean]
  --port                         The port to use for the debugging protocol. Use 0 for a random port                                 [default: 9222]
  --max-wait-for-load            The timeout (in milliseconds) to wait before the page is considered done loading and the run should continue.
                                 WARNING: Very high values can lead to large traces and instability                                 [default: 25000]

Output:
  --output       Reporter for the results, supports multiple values                           [choices: "json", "html", "domhtml"] [default: "html"]
  --output-path  The file path to output the results. Use 'stdout' to write to stdout.
                 If using JSON output, default is stdout.
                 If using HTML output, default is a file in the working directory with a name based on the test URL and date.
                 If using multiple outputs, --output-path is ignored.
                 Example: --output-path=./lighthouse-results.html
  --view         Open HTML report in your browser                                                                                          [boolean]

Options:
  --help                        Show help                                                                                                  [boolean]
  --version                     Show version number                                                                                        [boolean]
  --disable-storage-reset       Disable clearing the browser cache and other storage APIs before a run                                     [boolean]
  --disable-device-emulation    Disable Nexus 5X emulation                                                                                 [boolean]
  --disable-cpu-throttling      Disable CPU throttling                                                                    [boolean] [default: false]
  --disable-network-throttling  Disable network throttling                                                                                 [boolean]
  --skip-autolaunch             Skip autolaunch of Chrome when already running instance is not found                                       [boolean]
  --select-chrome               Interactively choose version of Chrome to use when multiple installations are found                        [boolean]
  --interactive                 Open Lighthouse in interactive mode                                                                        [boolean]

Examples:
  lighthouse <url> --view                                                   Opens the HTML report in a browser after the run completes
  lighthouse <url> --config-path=./myconfig.js                              Runs Lighthouse with your own configuration: custom audits, report
                                                                            generation, etc.
  lighthouse <url> --output=json --output-path=./report.json --save-assets  Save trace, screenshots, and named JSON report.
  lighthouse <url> --disable-device-emulation --disable-network-throttling  Disable device emulation
  lighthouse <url> --chrome-flags="--window-size=412,732"                   Launch Chrome with a specific window size
  lighthouse <url> --quiet --chrome-flags="--headless"                      Launch Headless Chrome, turn off logging

For more information on Lighthouse, see https://developers.google.com/web/tools/lighthouse/.


```

##### Output Examples
`lighthouse` generates
* `./<HOST>_<DATE>.report.html`

`lighthouse --output json` generates
* json output on `stdout`

`lighthouse --output html --output-path ./report.html` generates
* `./report.html`

NOTE: specifying an output path with multiple formats ignores your specified extension for *ALL* formats

`lighthouse --output json --output html --output-path ./myfile.json` generates
* `./myfile.report.json`
* `./myfile.report.html`

`lighthouse --output json --output html` generates
* `./<HOST>_<DATE>.report.json`
* `./<HOST>_<DATE>.report.html`

`lighthouse --output-path=~/mydir/foo.out --save-assets` generates
* `~/mydir/foo.report.html`
* `~/mydir/foo-0.trace.json`
* `~/mydir/foo-0.screenshots.html`

`lighthouse --output-path=./report.json --output json --save-artifacts` generates
* `./report.json`
* `./report.artifacts.log`

`lighthouse --save-artifacts` generates
* `./<HOST>_<DATE>.report.html`
* `./<HOST>_<DATE>.artifacts.log`

### Testing on a site with authentication

 - `chrome-debug`
 - open and login to your site
 - in a separate terminal tab `lighthouse http://mysite.com`

## Testing on a mobile device

Lighthouse can run against a real mobile device. You can follow the [Remote Debugging on Android (Legacy Workflow)](https://developer.chrome.com/devtools/docs/remote-debugging-legacy) up through step 3.3, but the TL;DR is install & run adb, enable USB debugging, then port forward 9222 from the device to the machine with Lighthouse.

You'll likely want to use the CLI flags `--disable-device-emulation --disable-cpu-throttling` and potentially `--disable-network-throttling`.

```sh
$ adb kill-server

$ adb devices -l
* daemon not running. starting it now on port 5037 *
* daemon started successfully *
00a2fd8b1e631fcb       device usb:335682009X product:bullhead model:Nexus_5X device:bullhead

$ adb forward tcp:9222 localabstract:chrome_devtools_remote

$ lighthouse --disable-device-emulation --disable-cpu-throttling https://mysite.com
```

## Using programmatically

The example below shows how to setup and run Lighthouse programmatically as a Node module. It
assumes you've installed Lighthouse as a dependency (`yarn add --dev lighthouse`).

```javascript
const lighthouse = require('lighthouse');
const chromeLauncher = require('lighthouse/chrome-launcher/chrome-launcher');

function launchChromeAndRunLighthouse(url, flags, config = null) {
  return chromeLauncher.launch().then(chrome => {
    flags.port = chrome.port;
    return lighthouse(url, flags, config).then(results =>
      chrome.kill().then(() => results)
    );
  });
}

// In use:
const flags = {output: 'json'};
launchChromeAndRunLighthouse('https://example.com', flags)
	.then(results => console.log(results));
```

### Recipes
> Helpful for CI integration

  - [gulp](docs/recipes/gulp)

## Viewing a report

Lighthouse can produce a report as JSON, HTML, or stdout CLI output.

HTML report:

![image](https://cloud.githubusercontent.com/assets/238208/21210165/b3c368c0-c22d-11e6-91fb-aa24959e2637.png)

Default CLI output:

![image](https://cloud.githubusercontent.com/assets/39191/19172762/60358d9a-8bd8-11e6-8c22-7fcb119ea0f5.png)

### Online Viewer

Running Lighthouse with the `--output=json` flag generates a json dump of the run.
You can view this report online by visiting <https://googlechrome.github.io/lighthouse/viewer/>
and dragging the file onto the app. You can also use the "Export" button from the
top of any Lighthouse HTML report and open the report in the
[Lighthouse Viewer](https://googlechrome.github.io/lighthouse/viewer/).

In the Viewer, reports can be shared by clicking the share icon in the top
right corner and signing in to GitHub.

> **Note**: shared reports are stashed as a secret Gist in GitHub, under your account.

## Related Projects

* [webpack-lighthouse-plugin](https://github.com/addyosmani/webpack-lighthouse-plugin) -  run Lighthouse from a Webpack build.
* [lighthouse-mocha-example](https://github.com/justinribeiro/lighthouse-mocha-example) -  gathers performance metrics via Lighthouse and tests them in Mocha
* [pwmetrics](https://github.com/paulirish/pwmetrics/) - gather performance metrics
* [lighthouse-hue](https://github.com/ebidel/lighthouse-hue) - Lighthouse score setting the color of Philips Hue lights
* [lighthouse-batch](https://www.npmjs.com/package/lighthouse-batch) - Run Lighthouse over a number of sites in sequence and generating a summary report including all of their scores.
* [lighthouse-cron](https://github.com/thearegee/lighthouse-cron) - Cron multiple batch Lighthouse audits and emit results for sending to remote server.

## Develop

### Setup

```sh
# yarn should be installed, first

git clone https://github.com/GoogleChrome/lighthouse

cd lighthouse
yarn install-all
yarn build-all

# The CLI is authored in TypeScript and requires compilation.
# If you need to make changes to the CLI, run the TS compiler in watch mode:
# cd lighthouse-cli && yarn dev
```

See [Contributing](./CONTRIBUTING.md) for more information.

### Run

```sh
node lighthouse-cli http://example.com
```

> **Getting started tip**: `node --inspect --debug-brk lighthouse-cli http://example.com` to open up Chrome DevTools and step
through the entire app. See [Debugging Node.js with Chrome
DevTools](https://medium.com/@paul_irish/debugging-node-js-nightlies-with-chrome-devtools-7c4a1b95ae27#.59rma3ukm)
for more info.

## Creating custom audits & gatherers

The audits and gatherers checked into the lighthouse repo are available to any configuration. If you're interested in writing your own audits or gatherers, you can use them with Lighthouse without necessarily contributing upstream.

Better docs coming soon, but in the meantime look at [PR #593](https://github.com/GoogleChrome/lighthouse/pull/593), and the tests [valid-custom-audit.js](https://github.com/GoogleChrome/lighthouse/blob/3f5c43f186495a7f3ecc16c012ab423cd2bac79d/lighthouse-core/test/fixtures/valid-custom-audit.js) and [valid-custom-gatherer.js](https://github.com/GoogleChrome/lighthouse/blob/3f5c43f186495a7f3ecc16c012ab423cd2bac79d/lighthouse-core/test/fixtures/valid-custom-gatherer.js). If you have questions, please file an issue and we'll help out!

> **Tip**: see [Lighthouse Architecture](./docs/architecture.md) for more information on Audits and Gatherers.

### Custom configurations for runs

You can supply your own run configuration to customize what audits you want details on. Copy the [default.js](https://github.com/GoogleChrome/lighthouse/blob/master/lighthouse-core/config/default.js) and start customizing. Then provide to the CLI with `lighthouse --config-path=myconfig.js <url>`

If you are simply adding additional audits/gatherers or tweaking flags, you can extend the default configuration without having to copy the default and maintain it. Passes with the same name will be merged together, all other arrays will be concatenated, and primitive values will override the defaults. See the example below that adds a custom gatherer to the default pass and an audit.

```json
{
  "extends": true,
  "passes": [
    {
      "passName": "defaultPass",
      "gatherers": ["path/to/custom/gatherer.js"]
    }
  ],
  "audits": ["path/to/custom/audit.js"],
  "aggregations": [
    {
      "name": "Custom Section",
      "description": "Enter description here.",
      "scored": false,
      "categorizable": false,
      "items": [
        {
          "name": "My Custom Audits",
          "audits": {
            "name-of-custom-audit": {}
          }
        }
      ]
    }
  ]
}
```

## Tests

Some basic unit tests forked are in `/test` and run via mocha. eslint is also checked for style violations.

```sh
# lint and test all files
yarn test

# watch for file changes and run tests
#   Requires http://entrproject.org : brew install entr
yarn watch

## run linting and unit tests separately
yarn lint
yarn unit

## run closure compiler (on whitelisted files)
yarn closure
## import your report renderer into devtools-frontend and run devtools closure compiler
yarn compile-devtools
```

## Lighthouse as trace processor

Lighthouse can be used to analyze trace and performance data collected from other tools (like WebPageTest and ChromeDriver). The `traces` and `devtoolsLogs` artifact items can be provided using a string for the absolute path on disk. The devtoolsLogs array is captured from the Network domain (a la ChromeDriver's [`enableNetwork` option](https://sites.google.com/a/chromium.org/chromedriver/capabilities#TOC-perfLoggingPrefs-object)) and reformatted slightly. As an example, here's a trace-only run that's reporting on user timings and critical request chains:

### `config.json`

```json
{
  "audits": [
    "user-timings",
    "critical-request-chains"
  ],

  "artifacts": {
    "traces": {
      "defaultPass": "/User/me/lighthouse/lighthouse-core/test/fixtures/traces/trace-user-timings.json"
    },
    "devtoolsLogs": {
      "defaultPass": "/User/me/lighthouse/lighthouse-core/test/fixtures/traces/perflog.json"
    }
  },

  "aggregations": [{
    "name": "Performance Metrics",
    "description": "These encapsulate your app's performance.",
    "scored": false,
    "categorizable": false,
    "items": [{
      "audits": {
        "user-timings": { "expectedValue": 0, "weight": 1 },
        "critical-request-chains": { "expectedValue": 0, "weight": 1}
      }
    }]
  }]
}
```

Then, run with: `lighthouse --config-path=config.json http://www.random.url`

The traceviewer-based trace processor from [node-big-rig](https://github.com/GoogleChrome/node-big-rig/tree/master/lib) was forked into Lighthouse. Additionally, the [DevTools' Timeline Model](https://github.com/paulirish/devtools-timeline-model) is available as well. There may be advantages for using one model over another.

## FAQ

### What is the architecture?

See [Lighthouse Architecture](./docs/architecture.md).

### What is "Do Better Web"?

**Do Better Web** is an initiative within Lighthouse to help web developers modernize their existing
web applications. By running a set of tests, developers can discover new web platform APIs, become
aware of performance pitfalls, and learn (newer) best practices. In other words, do better on the web!

DBW is implemented as a set of standalone [gatherers](https://github.com/GoogleChrome/lighthouse/tree/master/lighthouse-core/gather/gatherers/dobetterweb) and [audits](https://github.com/GoogleChrome/lighthouse/tree/master/lighthouse-core/audits/dobetterweb) that are run alongside the core Lighthouse tests. The tests show up under "Best Practices" in the report.

If you'd like to contribute, check the [list of issues](https://github.com/GoogleChrome/lighthouse/issues?q=is%3Aissue+is%3Aopen+label%3ADoBetterWeb) or propose a new audit by filing an issue.

### Are results sent to a remote server?

Nope. Lighthouse runs locally, auditing a page using a local version of the Chrome browser installed the
machine. Report results are never processed or beaconed to a remote server.

### Videos

Our session from Google I/O 2017: architecture, writing custom audits, Github/Travis/CI integration, and more:

[![Lighthouse @ Google I/O](https://img.youtube.com/vi/NoRYn6gOtVo/0.jpg)](https://www.youtube.com/watch?v=NoRYn6gOtVo)

_click to watch the video_

---

<p align="center">
  <img src="https://cloud.githubusercontent.com/assets/39191/22478294/23f662f6-e79e-11e6-8de3-ffd7be7bf628.png" alt="Lighthouse logo" height="150">
  <br>
  <b>Lighthouse</b>, ˈlītˌhous (n): a <s>tower or other structure</s> tool containing a beacon light
  to warn or guide <s>ships at sea</s> developers.
</p>
