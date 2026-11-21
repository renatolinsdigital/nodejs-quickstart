# Understanding Node.js Well

*An Essential compilation by Renato Lins*

![Repo Banner](img/repo_banner.jpg)

Node.js was created by Ryan Dahl in 2009 to run JavaScript **outside the browser**, on servers. He took Google's V8 JavaScript engine, attached an event loop and non-blocking I/O to it, and showed that one process could handle thousands of concurrent connections. Since then Node.js has become one of the most widely used platforms for web APIs, real-time services, command-line tools, build tooling (almost every front-end tool runs on Node), desktop apps (Electron) and serverless functions.

This guide goes from "what is Node.js?" to a tested Express REST API running in Docker. Along the way it explains **the engines underneath** (V8, libuv, the event loop), because knowing those is what lets you reason about performance, debug odd behavior and pick up any new version, or a different language, quickly.

```mermaid
timeline
    title Node.js milestones
    2009 : Ryan Dahl presents Node.js (V8 + event loop + non-blocking I/O)
    2010 : npm is created
    2014 : io.js fork over governance and release speed
    2015 : io.js merges back, Node.js Foundation, Node 4 and the LTS process
    2017 : async/await available natively (Node 7.6 / 8)
    2019 : OpenJS Foundation, ES Modules unflagged (Node 13.2)
    2022 : fetch and the built-in test runner arrive (Node 18)
    2024 : require(esm), stable watch mode, WebSocket client (Node 22)
    2025 : TypeScript type stripping enabled by default (Node 23.6 / 22.18)
    2026 : Node 26 is the last odd/even-era line, yearly releases start with Node 27
```

---

## How to read this guide

- **Version tags.** Features that did not always exist carry a tag such as `(Node ≥ 22)`. It marks the first release line where the feature works **without a flag**. If you are on an older or newer version, check the docs for *your* version: `https://nodejs.org/docs/latest-v24.x/api/` (replace `24` with your major version).
- **Transferable callouts.** Boxes marked **Transferable** connect a Node.js idea to the same idea in other versions, runtimes or languages (Python, Go, Java, C#, Rust...). The concepts last longer than the APIs, so learning them is the best return on your time.
- **Diagrams** are written in [Mermaid](https://mermaid.js.org/), which GitHub renders natively.
- **Code** uses modern ES Modules (`import`/`export`) unless a section is specifically about CommonJS. All examples run on the latest release of every supported LTS line (22, 24, 26). They were tested on Node 24.

## Table of contents

**[Part I — Foundations](#part-i--foundations)**

- [1. What Node.js is (and isn't)](#1-what-nodejs-is-and-isnt)
- [2. When to use Node.js (and when not to)](#2-when-to-use-nodejs-and-when-not-to)
- [3. Node.js versions and the release lifecycle](#3-nodejs-versions-and-the-release-lifecycle)
- [4. Installing Node.js and managing versions](#4-installing-nodejs-and-managing-versions)
- [5. Running code: the REPL, scripts and essential CLI flags](#5-running-code-the-repl-scripts-and-essential-cli-flags)
- [6. Editor setup: VS Code, Prettier and ESLint](#6-editor-setup-vs-code-prettier-and-eslint)

**[Part II — Under the hood: the engines behind Node.js](#part-ii--under-the-hood-the-engines-behind-nodejs)**

- [7. Architecture overview: the layers of Node.js](#7-architecture-overview-the-layers-of-nodejs)
- [8. The V8 engine: from source code to machine code](#8-the-v8-engine-from-source-code-to-machine-code)
- [9. Memory management and garbage collection](#9-memory-management-and-garbage-collection)
- [10. libuv: the asynchronous I/O engine](#10-libuv-the-asynchronous-io-engine)
- [11. The event loop, phase by phase](#11-the-event-loop-phase-by-phase)
- [12. Microtasks vs macrotasks: execution order](#12-microtasks-vs-macrotasks-execution-order)
- [13. Blocking vs non-blocking: don't block the event loop](#13-blocking-vs-non-blocking-dont-block-the-event-loop)
- [14. Other engines and runtimes](#14-other-engines-and-runtimes)

**[Part III — The core programming model](#part-iii--the-core-programming-model)**

- [15. Asynchronous patterns: callbacks, promises and async/await](#15-asynchronous-patterns-callbacks-promises-and-asyncawait)
- [16. Error handling](#16-error-handling)
- [17. Module systems: CommonJS and ES Modules](#17-module-systems-commonjs-and-es-modules)
- [18. Globals, the process object and web-standard APIs](#18-globals-the-process-object-and-web-standard-apis)

**[Part IV — Core modules](#part-iv--core-modules)**

- [19. Core modules tour and the file system](#19-core-modules-tour-and-the-file-system)
- [20. path, url and os](#20-path-url-and-os)
- [21. Events and EventEmitter](#21-events-and-eventemitter)
- [22. Buffers and binary data](#22-buffers-and-binary-data)
- [23. Streams and backpressure](#23-streams-and-backpressure)
- [24. HTTP from scratch and the fetch client](#24-http-from-scratch-and-the-fetch-client)
- [25. Concurrency for CPU work: child processes, worker threads and cluster](#25-concurrency-for-cpu-work-child-processes-worker-threads-and-cluster)
- [26. The built-in test runner](#26-the-built-in-test-runner)

**[Part V — Packages, tooling and workflow](#part-v--packages-tooling-and-workflow)**

- [27. npm and the anatomy of package.json](#27-npm-and-the-anatomy-of-packagejson)
- [28. Semantic versioning and lockfiles](#28-semantic-versioning-and-lockfiles)
- [29. Installing and managing dependencies](#29-installing-and-managing-dependencies)
- [30. Alternative package managers: Yarn, pnpm and Bun](#30-alternative-package-managers-yarn-pnpm-and-bun)
- [31. Scripts and hot reload](#31-scripts-and-hot-reload)
- [32. Environment variables and configuration](#32-environment-variables-and-configuration)
- [33. TypeScript with Node.js](#33-typescript-with-nodejs)
- [34. Debugging and profiling](#34-debugging-and-profiling)

**[Part VI — Building a real backend API with Express](#part-vi--building-a-real-backend-api-with-express)**

- [35. Web frameworks and the middleware model](#35-web-frameworks-and-the-middleware-model)
- [36. Case study: designing a Tasks API](#36-case-study-designing-a-tasks-api)
- [37. Building the API step by step](#37-building-the-api-step-by-step)
- [38. Testing the API](#38-testing-the-api)
- [39. REST API design and a production checklist](#39-rest-api-design-and-a-production-checklist)

**[Part VII — Docker](#part-vii--docker)**

- [40. What Docker is: containers, images and layers](#40-what-docker-is-containers-images-and-layers)
- [41. Dockerizing the Node.js API](#41-dockerizing-the-nodejs-api)
- [42. Docker Compose: the API with a database](#42-docker-compose-the-api-with-a-database)
- [43. Container best practices for Node.js](#43-container-best-practices-for-nodejs)

**[Part VIII — Production and beyond](#part-viii--production-and-beyond)**

- [44. Security essentials](#44-security-essentials)
- [45. Performance, scaling and deployment](#45-performance-scaling-and-deployment)
- [46. Upgrading Node.js safely](#46-upgrading-nodejs-safely)
- [47. Transferable knowledge: Node.js concepts in other languages](#47-transferable-knowledge-nodejs-concepts-in-other-languages)
- [48. Sharing your project and next steps](#48-sharing-your-project-and-next-steps)

---

## Part I — Foundations

### 1. What Node.js is (and isn't)

Node.js is a **JavaScript runtime**: a program (written mostly in C++ and JavaScript) that reads your `.js` files and executes them, and that gives JavaScript access to things browsers deliberately hide: files, network sockets, processes, environment variables, the operating system.

It helps to separate three things people often mix up:

| Thing | What it is | Examples |
| --- | --- | --- |
| **Language** | The syntax and semantics, standardized as ECMAScript | `const`, classes, promises, `async/await` |
| **Engine** | Parses, compiles and executes the language, manages memory | V8, SpiderMonkey, JavaScriptCore |
| **Runtime** | Engine + an event loop + APIs for a particular environment | Browser (DOM, `window`), **Node.js** (`fs`, `http`, `process`), Deno, Bun |

So `setTimeout`, `fs.readFile` and `http.createServer` are **not JavaScript**. They are APIs that the *runtime* provides. V8 alone cannot read a file or wait for a timer. That is why browser code and Node code share the language but not the APIs (no `document` in Node, no `fs` in the browser).

Node.js is **not** a framework (Express, NestJS and Fastify are frameworks that run *on* Node), it is **not** a language, and it is **not** only for web servers.

> **Transferable:** Every modern platform has the same split. Java is a language, HotSpot is the engine, the JVM plus the JDK libraries form the runtime. Python is a language, CPython is both engine and runtime. Asking "what is the language, the engine and the runtime here?" clears up a lot of confusion when you learn a new stack.

### 2. When to use Node.js (and when not to)

Node.js shines when the work is mostly **waiting**: on the network, on databases, on files, on other services. Its single-threaded event loop handles thousands of concurrent waits cheaply.

**Great fits**

- REST and GraphQL APIs, backends-for-frontends (BFF), API gateways
- Real-time apps: chat, notifications, live dashboards, multiplayer game servers (WebSockets)
- Streaming: proxies, file uploads/downloads, log processing
- CLIs and developer tooling (npm, ESLint, Prettier, Vite, TypeScript's compiler and many more run on Node)
- Server-side rendering of web apps (Next.js, Nuxt, Remix, Astro)
- Serverless functions, desktop apps (Electron), IoT scripts

**Weaker fits**, unless you design around them

- Long CPU-heavy computation in the request path (video encoding, image processing at scale, ML training). The event loop runs on one thread, so CPU work blocks everyone. Offload it to [worker threads](#25-concurrency-for-cpu-work-child-processes-worker-threads-and-cluster), a queue, or a service in another language.
- Problems that need fine-grained shared-memory parallelism, where Go, Rust, Java or C++ are a more natural choice.

**Why it matters for your career:** one language across front end and back end, a huge package ecosystem, and wide adoption (Netflix, PayPal, LinkedIn, Walmart, Uber and NASA have all written publicly about using Node.js) make Node.js skills valuable in web development, microservices, real-time systems and tooling.

### 3. Node.js versions and the release lifecycle

Node.js versions follow **semantic versioning**: `v26.13.1` means major `26`, minor `13`, patch `1`. A new **major** can include breaking changes; minors add features; patches fix bugs and security issues.

**How release lines work**

| Phase | Meaning | Use it for |
| --- | --- | --- |
| **Current** | Newest features, still stabilizing | Trying new features, testing libraries |
| **Active LTS** | Long-Term Support: stable, receives fixes and some backported features | **Production** (the default choice) |
| **Maintenance LTS** | Only critical bug fixes and security patches | Production you have not upgraded yet; plan the upgrade |
| **End-of-Life (EOL)** | No more fixes, including security fixes | Nothing. Upgrade. |

Up to Node 26, a new major came out every April and October. **Even-numbered** lines became LTS and **odd-numbered** lines never did. Starting with **Node 27** the project moved to **one major per year**: an *Alpha* phase (October–March, for early testers), a *Current* phase (April–October), then **every** line becomes LTS for 30 months, for 36 months of support in total. The odd/even rule is gone.

```mermaid
gantt
    title Node.js release lines (official schedule)
    dateFormat YYYY-MM-DD
    axisFormat %Y
    section Node 22 Jod
    Current         : 2024-04-24, 2024-10-29
    Active LTS      : 2024-10-29, 2025-10-21
    Maintenance LTS : 2025-10-21, 2027-04-30
    section Node 24 Krypton
    Current         : 2025-05-06, 2025-10-28
    Active LTS      : 2025-10-28, 2026-10-20
    Maintenance LTS : 2026-10-20, 2028-04-30
    section Node 26 Lithium
    Current         : 2026-05-05, 2026-10-28
    Active LTS      : active, 2026-10-28, 2027-10-20
    Maintenance LTS : 2027-10-20, 2029-04-30
    section Node 27
    Alpha           : 2026-10-28, 2027-04-22
    Current         : 2027-04-22, 2027-10-20
    LTS             : 2027-10-20, 2030-04-30
```

**Status at the time of writing (November 2026)**

| Line | Status | End of life | Recommendation |
| --- | --- | --- | --- |
| **26** (Lithium) | Active LTS | 2029-04-30 | **Use this for new projects** |
| 24 (Krypton) | Maintenance LTS | 2028-04-30 | Fine for production, plan to move to 26 |
| 22 (Jod) | Maintenance LTS | 2027-04-30 | Upgrade soon |
| 27 | Alpha | 2030-04-30 | Testing only |
| 25, 20 and older | EOL | — | Upgrade now |

Dates come from the official [Node.js release schedule](https://github.com/nodejs/Release). Always check [nodejs.org/en/about/previous-releases](https://nodejs.org/en/about/previous-releases) for the current picture, because this table will age.

**Rules that work for any version**

1. **Production runs an LTS line.** Pick the newest Active LTS when you start, and upgrade before your line reaches EOL.
2. **Pin the version in the project** so everyone uses the same one: an `.nvmrc` file, the `engines` field in `package.json`, and the base image tag in your `Dockerfile` ([topic 46](#46-upgrading-nodejs-safely) shows how to keep them in sync).
3. **Know what your version contains:**

   ```bash
   node -v                      # v26.x.y
   node -p process.versions     # versions of V8, libuv, OpenSSL, npm, etc. bundled in this build
   node -p process.release.lts  # LTS codename (e.g. 'Lithium'), or undefined on non-LTS lines
   ```

4. **Read the Stability Index** in the docs. Every API is marked `0 - Deprecated`, `1 - Experimental`, `2 - Stable` or `3 - Legacy`. Build production code on Stable APIs only. Deprecations have codes like `DEP0005`, and running with `--trace-deprecation` shows where your code triggers them.

> **Transferable:** Most platforms use the same "fast line + LTS line" idea. .NET also makes even-numbered releases LTS (3 years). Java ships every 6 months with an LTS every two years (17, 21, 25). Ubuntu has LTS releases every two years. Python releases yearly with about five years of support. Learn to find the support table of any technology you depend on.

### 4. Installing Node.js and managing versions

**Option A — Official installer (simplest).** Download the LTS installer from [nodejs.org](https://nodejs.org/), run it, keep "Add to PATH" checked, then confirm:

```bash
node --version   # e.g. v26.13.1
npm --version    # npm is bundled with Node.js
```

**Option B — A version manager (recommended for developers).** Different projects often need different Node versions. A version manager installs several side by side and switches between them per project, with no reinstalling and no PATH editing.

| Tool | Platforms | Notes |
| --- | --- | --- |
| [nvm](https://github.com/nvm-sh/nvm) | macOS, Linux, WSL | The classic, shell-script based |
| [nvm-windows](https://github.com/coreybutler/nvm-windows) | Windows | A separate project with similar commands (download `nvm-setup.exe` from Releases) |
| [fnm](https://github.com/Schniz/fnm) | All | Fast (Rust), reads `.nvmrc` / `.node-version`, can switch automatically on `cd` |
| [mise](https://mise.jdx.dev/) | All | Polyglot: manages Node, Python, Go, Java... versions with one tool |

Common nvm commands (fnm and nvm-windows are very similar):

```bash
nvm install --lts        # install the latest LTS (nvm-windows: nvm install lts)
nvm install 26           # install the latest 26.x
nvm use 26               # switch the current shell to 26.x
nvm alias default 26     # make 26.x the default for new shells (nvm/fnm)
nvm ls                   # list installed versions (* marks the active one)
nvm uninstall 22.11.0    # remove a specific version
```

Pin the project's version with an `.nvmrc` file at the repository root:

```text
26
```

Then anyone can run `nvm use` (or `fnm use`) inside the project and get the right version.

> [!NOTE]
> On nvm-windows, Node is not active until you run `nvm use <version>` at least once, even if Node is installed.

> **Transferable:** Version managers exist for every ecosystem: `pyenv`/`uv` (Python), `rustup` (Rust), SDKMAN (Java), `rbenv` (Ruby). Polyglot tools such as mise or asdf cover all of them with one config file. The pattern is always "install many, pin one per project in a file committed to Git."

### 5. Running code: the REPL, scripts and essential CLI flags

**The REPL** (Read-Eval-Print Loop) is an interactive prompt, handy for quick experiments:

```text
$ node
> 1 + 1
2
> [1, 2, 3].map((n) => n * 2)
[ 2, 4, 6 ]
> .help    (list REPL commands)
> .exit    (or press Ctrl+C twice)
```

**Running a file:** create `scripts.js`:

```javascript
const num1 = 10;
const num2 = 20;
console.log(`The sum of ${num1} and ${num2} is: ${num1 + num2}`);
```

```bash
node scripts.js
# The sum of 10 and 20 is: 30
```

**Flags worth knowing.** Put flags *before* the file name:

| Command | What it does |
| --- | --- |
| `node --watch app.js` | Restart automatically when files change `(Node ≥ 22)` |
| `node --env-file=.env app.js` | Load environment variables from a file `(Node ≥ 20.6)` |
| `node --run dev` | Run a `package.json` script, faster than `npm run` `(Node ≥ 22)` |
| `node --test` | Run the built-in test runner ([topic 26](#26-the-built-in-test-runner)) |
| `node --inspect app.js` | Enable the debugger ([topic 34](#34-debugging-and-profiling)) |
| `node -e "console.log(1+1)"` | Evaluate a string of code |
| `node -p "process.versions"` | Evaluate and **p**rint the result |
| `node app.ts` | Run TypeScript directly by stripping types `(Node ≥ 22.18 / 23.6)`, see [topic 33](#33-typescript-with-nodejs) |
| `node --help` | Everything else |

Flags can also go in the `NODE_OPTIONS` environment variable (for example `NODE_OPTIONS="--max-old-space-size=2048"`), which helps when you do not control the command line, such as in Docker or CI.

### 6. Editor setup: VS Code, Prettier and ESLint

[Visual Studio Code](https://code.visualstudio.com/) is a free, lightweight editor with first-class JavaScript and Node.js support, including a built-in debugger. Download and install it for your OS.

**Useful extensions**

| Extension | Why |
| --- | --- |
| **Prettier** (`esbenp.prettier-vscode`) | Formats code automatically, so style is never debated in code review |
| **ESLint** (`dbaeumer.vscode-eslint`) | Finds bugs and bad patterns as you type |
| **Path Intellisense** (`christian-kohler.path-intellisense`) | Autocompletes file paths in imports |
| **Code Spell Checker** (`streetsidesoftware.code-spell-checker`) | Catches typos in identifiers, strings and comments |
| **REST Client** (`humao.rest-client`) | Send HTTP requests from `.http` files, great for testing APIs ([topic 37](#37-building-the-api-step-by-step)) |
| **Docker / Container Tools** | Dockerfile and Compose support ([Part VII](#part-vii--docker)) |
| **JavaScript (ES6) code snippets** | Shortcuts, e.g. type `imp` + Tab for an import |
| **Live Server** | Local server with live reload for *front-end* HTML/JS pages (not needed for Node APIs) |

**Format on save.** Create `.vscode/settings.json` in the project:

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": "explicit"
  }
}
```

**Prettier rules.** Create `.prettierrc` at the project root:

```json
{
  "singleQuote": true,
  "semi": true,
  "tabWidth": 2,
  "printWidth": 100,
  "trailingComma": "all"
}
```

`singleQuote` uses `'` instead of `"`, `semi` adds semicolons, `tabWidth` sets indentation, `printWidth` sets where lines wrap, and `trailingComma` adds trailing commas (cleaner Git diffs). See [prettier.io](https://prettier.io/docs/options) for all options.

**ESLint.** Prettier handles *style*; ESLint handles *correctness* (unused variables, accidental globals, unreachable code...). Set it up interactively with `npm init @eslint/config@latest`, or write a minimal "flat config" by hand:

```bash
npm install -D eslint @eslint/js globals
```

```javascript
// eslint.config.js
import js from '@eslint/js';
import globals from 'globals';

export default [
  js.configs.recommended,
  { languageOptions: { globals: globals.node } },
];
```

Run it with `npx eslint .`.

> **Transferable:** Every ecosystem separates the *formatter* (opinionated, no debates: Prettier, Black/Ruff for Python, `gofmt`, `rustfmt`) from the *linter* (finds bugs: ESLint, Ruff/Pylint, `go vet`, Clippy). Set both up on day one of any project.

---

## Part II — Under the hood: the engines behind Node.js

### 7. Architecture overview: the layers of Node.js

Knowing the layers explains most of Node's behavior: why I/O does not block, why CPU work does, and where performance problems come from.

```mermaid
%%{init: {"htmlLabels": false, "flowchart": {"htmlLabels": false, "wrappingWidth": 400, "padding": 24}}}%%
flowchart TB
    APP["Your code<br/>+ npm packages<br/>JS or TypeScript"]
    subgraph NODE["Node.js runtime"]
        LIB["Node.js standard library<br/>written in JavaScript<br/>fs, http, net, crypto,<br/>stream, events..."]
        BIND["C++ bindings<br/>bridge between JS<br/>and native code"]
        V8["V8<br/>runs JavaScript<br/>manages memory"]
        UV["libuv<br/>event loop<br/>async I/O<br/>thread pool"]
        DEPS["Other native libraries<br/>llhttp, OpenSSL, zlib,<br/>c-ares, ICU, nghttp2,<br/>simdutf, ada"]
    end
    OS["Operating system<br/>sockets, files, timers,<br/>processes<br/>epoll on Linux<br/>kqueue on macOS<br/>IOCP on Windows"]

    APP --> LIB --> BIND
    BIND --> V8
    BIND --> UV
    BIND --> DEPS
    UV --> OS
    DEPS --> OS
```

| Component | Role |
| --- | --- |
| **V8** | Google's JavaScript engine (also used in Chrome). Compiles and runs your JS, manages the heap and garbage collection. [Topic 8](#8-the-v8-engine-from-source-code-to-machine-code) |
| **libuv** | A C library that provides the event loop, cross-platform async I/O and a thread pool. [Topic 10](#10-libuv-the-asynchronous-io-engine) |
| **Bindings** | C++ glue that exposes native functionality to JavaScript (e.g. `fs.readFile` in JS calls into C++, which calls libuv) |
| **llhttp** | Very fast HTTP/1.1 parser (written in TypeScript, compiled to C) |
| **OpenSSL** | TLS/HTTPS and the `crypto` module |
| **zlib** | Compression (`zlib` module, gzip/deflate/brotli) |
| **c-ares** | Asynchronous DNS resolution (`dns.resolve*`) |
| **ICU** | Internationalization (`Intl`, dates, number formatting) |
| **undici** | The HTTP client behind the global `fetch` (written in JavaScript) |
| **npm** | The bundled package manager (a separate project, shipped with Node) |

**What happens when you call `fs.readFile()`:** your JS calls Node's `fs` library, which goes through a C++ binding, which asks libuv to read the file on a thread-pool thread. The JavaScript thread moves on immediately. When the read completes, libuv puts a "done" event in a queue, and the event loop later runs your callback (or resolves your promise) back on the JavaScript thread.

> **Transferable:** "A high-level language on top of a native core" is a common design. Python's standard library wraps C, the JVM is C++ underneath Java, and Deno uses V8 + Rust (tokio) instead of V8 + C (libuv). When something is slow or crashes "inside the runtime", it is usually at this boundary.

### 8. The V8 engine: from source code to machine code

V8 does **not** simply interpret JavaScript, and it does not compile everything ahead of time either. It uses a **multi-tier JIT (Just-In-Time) pipeline**: start running quickly, watch what the code actually does, then spend compile time only on the code that runs often ("hot" code).

```mermaid
%%{init: {"htmlLabels": false, "flowchart": {"htmlLabels": false, "wrappingWidth": 400, "padding": 24}}}%%
flowchart LR
    SRC["JavaScript source"] --> PARSE["Parser<br/>source to AST"]
    PARSE --> IGN["Ignition<br/>interpreter<br/>AST to bytecode<br/>collects type feedback"]
    IGN -- "warm" --> SP["Sparkplug<br/>baseline compiler<br/>fast, no optimizations"]
    SP -- "hot" --> MAG["Maglev<br/>mid-tier optimizing<br/>compiler"]
    MAG -- "very hot" --> TF["TurboFan<br/>top-tier optimizing<br/>compiler"]
    MAG -. "deoptimize when an<br/>assumption breaks" .-> IGN
    TF -. "deoptimize" .-> IGN
```

1. **Parsing.** Source code becomes an Abstract Syntax Tree (AST). V8 parses lazily: functions that are never called are only pre-parsed.
2. **Ignition** turns the AST into compact **bytecode** and interprets it. While running, it records **type feedback**: "this `+` always adds two small integers", "this object always has the same shape".
3. **Sparkplug** quickly turns bytecode into machine code without optimizing, which removes interpreter overhead.
4. **Maglev** and **TurboFan** use the type feedback to produce **speculatively optimized** machine code ("assume `x` is always an integer"). TurboFan takes longer to compile but produces the fastest code.
5. **Deoptimization.** If an assumption becomes false (suddenly `x` is a string), V8 throws the optimized code away and falls back to bytecode. Code that keeps flipping types can get stuck being slow.

**Hidden classes (shapes) and inline caches.** JavaScript objects are dynamic, but V8 secretly assigns each object a *hidden class* describing its layout. Objects with the same properties added in the same order share a hidden class, so property access becomes a fixed memory offset, cached at each call site (an *inline cache*).

```javascript
// Same shape: V8 can optimize every access to p.x / p.y
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
}
const a = new Point(1, 2);
const b = new Point(3, 4);

// Different shapes for "the same kind" of object: slower, polymorphic access
const c = { x: 1, y: 2 };
const d = { y: 2, x: 1 }; // different property order = different hidden class
delete c.x;               // deleting can push an object into slow "dictionary mode"
```

**Practical takeaways** (these apply to any JIT-compiled language):

- Initialize all of an object's properties up front (in the constructor), in the same order.
- Avoid `delete` on hot objects. Set the property to `undefined` or build a new object.
- Keep the types passed to a hot function consistent ("monomorphic" code is fastest).
- Keep arrays homogeneous (all numbers, or all objects of the same shape).
- **Measure before optimizing.** Readable code first, then profile ([topic 34](#34-debugging-and-profiling)).

Other things V8 provides: **isolates** (independent V8 instances with separate heaps; each [worker thread](#25-concurrency-for-cpu-work-child-processes-worker-threads-and-cluster) gets its own), the **call stack**, **garbage collection** (next topic) and **startup snapshots** (pre-initialized heaps that make Node start faster). Check your V8 version with `node -p process.versions.v8` and list its tuning flags with `node --v8-options`.

> **Transferable:** The JVM (HotSpot's C1/C2 compilers), .NET (tiered compilation in RyuJIT) and PyPy use the same idea: interpret or quickly compile first, profile, then optimize hot paths speculatively and deoptimize when assumptions break. The "keep types and shapes stable" advice carries over directly.

### 9. Memory management and garbage collection

**Stack vs heap.** Each function call pushes a *frame* onto the **call stack** (local variables, arguments, return address). It is small, fast and cleaned up automatically when the function returns. Objects, arrays, closures and strings live on the **heap**, a large memory area managed by V8's **garbage collector (GC)**.

**Generational GC.** Most objects die young (a temporary array inside a function, a request object). V8 exploits this with two main areas:

```mermaid
%%{init: {"htmlLabels": false, "flowchart": {"htmlLabels": false, "wrappingWidth": 400, "padding": 24}}}%%
flowchart LR
    NEW(["new object"]) --> YOUNG["Young generation<br/>small, collected often<br/>by the Scavenger<br/>(fast copying GC)"]
    YOUNG -- "survives two<br/>collections" --> OLD["Old generation<br/>large, collected<br/>less often by<br/>Mark-Sweep-Compact"]
    YOUNG -- "unreachable" --> FREE1(["memory reclaimed"])
    OLD -- "unreachable" --> FREE2(["memory reclaimed"])
```

- **Minor GC (Scavenger):** copies live young objects to another space. Very fast, runs often, briefly pauses JS.
- **Major GC (Mark-Sweep-Compact):** marks everything reachable from "roots" (globals, the stack), frees the rest, and compacts memory to reduce fragmentation. V8's Orinoco project made most of this work **concurrent and parallel** (done on helper threads while JS keeps running), so pauses are short.

An object is garbage when nothing reachable references it. A **memory leak** in JavaScript is therefore almost always "something still references data you no longer need":

| Common leak | Fix |
| --- | --- |
| A module-level `Map`/array used as a cache that only grows | Limit its size (LRU), add expiry, or use a `WeakMap` |
| Event listeners added on every request, never removed | `emitter.off()`, `once()`, or an `AbortSignal` |
| `setInterval` that is never cleared | `clearInterval` on shutdown |
| Closures capturing large objects | Capture only what you need |

**Inspecting memory:**

```javascript
const { rss, heapTotal, heapUsed, external } = process.memoryUsage();
const mb = (n) => `${(n / 1024 / 1024).toFixed(1)} MB`;
console.log({ rss: mb(rss), heapTotal: mb(heapTotal), heapUsed: mb(heapUsed), external: mb(external) });
```

- `rss`: total memory the process holds; `heapUsed`: live JS objects; `external`: memory outside the V8 heap (e.g. `Buffer`s).
- Take **heap snapshots** with `node --inspect` + Chrome DevTools, and compare two snapshots to find what is growing.
- The old-generation limit is set by `--max-old-space-size=<MB>` (e.g. `node --max-old-space-size=4096 app.js`). If the heap reaches the limit you get `FATAL ERROR: ... JavaScript heap out of memory`. Recent versions size the default from available (and container) memory, but setting it explicitly in containers makes behavior predictable.

> **Transferable:** Java (G1/ZGC) and .NET (gen0/gen1/gen2) also use generational collectors with the same "most objects die young" idea. Go uses a concurrent, non-generational collector. Python uses reference counting plus a cycle collector. Rust has no GC at all; ownership rules free memory at compile-time-known points. "A leak is an unwanted reference" holds in every GC language.

### 10. libuv: the asynchronous I/O engine

libuv is the C library that gives Node its **event loop** and **cross-platform async I/O**. It was written for Node.js and is now also used by other projects (Julia, Luvit, pyuv and others).

The key idea: **the operating system can do many kinds of I/O without a thread waiting for it.** libuv uses the best mechanism each OS offers and falls back to a thread pool when no async API exists.

```mermaid
%%{init: {"htmlLabels": false, "flowchart": {"htmlLabels": false, "wrappingWidth": 400, "padding": 24}}}%%
flowchart TB
    JS["Your JavaScript<br/>(main thread)"] -- "socket.write, http.get" --> LOOP["libuv event loop<br/>(same main thread)"]
    JS -- "fs.readFile, dns.lookup,<br/>crypto.pbkdf2, zlib.gzip" --> LOOP
    LOOP -- "network I/O" --> KERNEL["OS async notification<br/>epoll on Linux<br/>kqueue on macOS/BSD<br/>IOCP on Windows"]
    LOOP -- "work with no<br/>async OS API" --> POOL["libuv thread pool<br/>4 threads by default"]
    KERNEL -- "socket ready" --> LOOP
    POOL -- "task finished" --> LOOP
    LOOP -- "run callback or<br/>resolve promise" --> JS
```

| Work | Handled by |
| --- | --- |
| TCP/UDP sockets, HTTP servers and clients, pipes | **OS kernel** async notification (no extra threads) |
| File system operations (`fs.*`) | **Thread pool** |
| `dns.lookup()` (used by `http.get`, `fetch`, DB drivers to resolve hostnames) | **Thread pool** |
| CPU-heavy `crypto` (`pbkdf2`, `scrypt`, `randomBytes` async), `zlib` async | **Thread pool** |
| Timers (`setTimeout`, `setInterval`) | The event loop itself (a min-heap of deadlines) |

**Consequences you can observe:**

- Node handles 10,000 idle WebSocket connections easily: they cost no threads.
- Only **4** `fs`/`crypto`/`dns.lookup` operations run truly in parallel by default. If you hash many passwords with `crypto.scrypt` at once, they queue. You can raise the pool size with the environment variable `UV_THREADPOOL_SIZE` (e.g. `UV_THREADPOOL_SIZE=16`), set before the process starts.
- Your JavaScript itself still runs on **one** thread. libuv's threads never run your JS.

> **Transferable:** The same "readiness notification" APIs (epoll/kqueue/IOCP) power nginx, Redis, Python's `asyncio`, Rust's `tokio`, Java's Netty and Go's network poller. Learn epoll/kqueue once and you understand how all of them scale to many connections.

### 11. The event loop, phase by phase

The event loop is a loop in libuv that keeps asking: *"Is there anything ready to run?"* Each iteration (a "tick") goes through fixed **phases**, and each phase has its own queue of callbacks:

```mermaid
%%{init: {"htmlLabels": false, "flowchart": {"htmlLabels": false, "wrappingWidth": 400, "padding": 24}}}%%
flowchart TB
    START(["node app.js<br/>runs the main script<br/>synchronously"]) --> TIMERS
    TIMERS["1. timers<br/>expired setTimeout and<br/>setInterval callbacks"] --> PENDING
    PENDING["2. pending callbacks<br/>some deferred<br/>system I/O errors"] --> IDLE
    IDLE["3. idle, prepare<br/>internal use"] --> POLL
    POLL["4. poll<br/>get new I/O events,<br/>run their callbacks<br/>(waits here when idle)"] --> CHECK
    CHECK["5. check<br/>setImmediate callbacks"] --> CLOSE
    CLOSE["6. close callbacks<br/>e.g. socket.on('close')"] --> ALIVE{"anything still<br/>pending?"}
    ALIVE -- "yes" --> TIMERS
    ALIVE -- "no" --> EXIT(["process exits"])
```

Between **every single callback** in every phase, Node empties the **microtask queues** (`process.nextTick`, then promises). See the next topic.

What each phase means in practice:

- **timers:** runs `setTimeout`/`setInterval` callbacks whose time has passed. A timer of `100` ms means "*no earlier than* 100 ms", not "exactly 100 ms".
- **poll:** the heart of I/O. Incoming HTTP requests, finished file reads and socket data are processed here. If nothing is scheduled, the loop **sleeps** here until the OS reports an event or the next timer is due. That is why an idle Node server uses almost no CPU.
- **check:** `setImmediate` callbacks run right after poll. Use it to say "run this after the current I/O is processed".
- **close:** cleanup callbacks for closed handles.

**Why a process exits (or doesn't).** Node keeps running while there are *active handles* (a listening server, an open socket, a pending timer). When none are left, the process exits. `timer.unref()` tells Node "don't stay alive just for this timer", which the API's graceful-shutdown timeout uses in [topic 37](#37-building-the-api-step-by-step).

### 12. Microtasks vs macrotasks: execution order

There are two kinds of queues:

- **Macrotasks** (just "tasks"): callbacks in the event-loop phases: timers, I/O, `setImmediate`, close.
- **Microtasks:** high-priority jobs that run **as soon as the current JavaScript finishes**, before the loop moves on:
  1. the `process.nextTick` queue (Node-specific), then
  2. the promise microtask queue (`.then`, `await` continuations, `queueMicrotask`).

```javascript
// order.cjs  (CommonJS, see the note below about ES Modules)
const fs = require('node:fs');

console.log('1. synchronous');

setTimeout(() => console.log('timeout'), 0);
setImmediate(() => console.log('immediate'));
Promise.resolve().then(() => console.log('promise'));
process.nextTick(() => console.log('nextTick'));

fs.readFile(__filename, () => {
  // Inside an I/O callback the order of setTimeout vs setImmediate IS deterministic
  setTimeout(() => console.log('timeout (from I/O)'), 0);
  setImmediate(() => console.log('immediate (from I/O)'));
});

console.log('2. synchronous');
```

Typical output:

```text
1. synchronous
2. synchronous
nextTick
promise
timeout                 <- these two can swap places when called from the main script,
immediate               <- because it depends on how fast the loop starts (non-deterministic)
immediate (from I/O)    <- after poll comes check, so setImmediate always wins here
timeout (from I/O)
```

Things worth remembering:

- Synchronous code always finishes first. Nothing interrupts running JavaScript.
- `process.nextTick` runs before promises. In an **ES Module**, the top-level code itself runs as part of a promise job, so there you will see `promise` printed *before* `nextTick`. Don't write code that depends on the order between the two.
- Microtasks can **starve** the loop: a promise chain or `nextTick` that keeps scheduling itself never lets timers or I/O run.
- Prefer `setImmediate` or promises over `process.nextTick` in new code. `nextTick` is mostly for library internals.

> **Transferable:** Browsers have the same task/microtask split (promise callbacks run before the next `setTimeout`), so this knowledge also explains front-end behavior. Python's `asyncio` runs callbacks with `loop.call_soon`, and C# continuations run on the `SynchronizationContext` or thread pool. Every async runtime has some "run this next" queue with priority rules, and it is worth learning them.

### 13. Blocking vs non-blocking: don't block the event loop

Because all your JavaScript runs on one thread, **while one callback runs, nothing else does**: no other request, no timer, no I/O callback. Non-blocking I/O lets one thread serve many clients *as long as each callback finishes quickly*.

```mermaid
sequenceDiagram
    participant A as Client A
    participant B as Client B
    participant N as Node main thread
    participant DB as Database
    A->>N: GET /users
    N->>DB: query (non-blocking, thread is free)
    B->>N: GET /health
    N-->>B: 200 OK, served while the DB works
    DB-->>N: rows ready (poll phase)
    N-->>A: 200 with the users
```

Now the bad case. One CPU-heavy request freezes the whole server:

```javascript
import { createServer } from 'node:http';

createServer((req, res) => {
  if (req.url === '/block') {
    const end = Date.now() + 5000;
    while (Date.now() < end) {} // burns CPU for 5 seconds: EVERY other request waits
    return res.end('finished blocking\n');
  }
  res.end('fast\n');
}).listen(3000, () => console.log('http://localhost:3000'));
```

Open `/block` in one tab and `/` in another: the "fast" route waits 5 seconds too.

**Typical event-loop blockers**

| Blocker | Better approach |
| --- | --- |
| `fs.readFileSync`, `execSync` and other `*Sync` APIs inside request handlers | The async versions (`fs/promises`). `*Sync` is fine at startup and in CLI scripts |
| Large `JSON.parse` / `JSON.stringify` (many MB) | Limit body sizes, paginate, stream |
| CPU work: image resizing, big sorts, compression, hashing with `*Sync` | [Worker threads](#25-concurrency-for-cpu-work-child-processes-worker-threads-and-cluster), async crypto, a job queue |
| Regular expressions with catastrophic backtracking (ReDoS) | Simpler patterns, input length limits, safe-regex linting |
| Long loops over huge arrays | Process in chunks, yielding with `setImmediate`, or stream |

**Measure it:** Node can report how late the event loop is running:

```javascript
import { monitorEventLoopDelay } from 'node:perf_hooks';

const histogram = monitorEventLoopDelay({ resolution: 20 });
histogram.enable();
setInterval(() => {
  console.log(`event loop delay p99: ${(histogram.percentile(99) / 1e6).toFixed(1)} ms`);
  histogram.reset();
}, 5000);
```

A healthy server shows a few milliseconds. Hundreds of milliseconds means something is blocking.

> **Transferable:** "Don't block the event loop" is the same rule as "don't block the UI thread" in Android, iOS, WPF or the browser, and "don't call blocking code inside `async def`" in Python's asyncio. Any single-threaded event loop has this weakness and the same fixes: go async, chunk the work, or move it to another thread or process.

### 14. Other engines and runtimes

Node.js is not the only way to run JavaScript outside the browser. The concepts in this guide (engine + event loop + APIs) apply to all of them.

**JavaScript engines**

| Engine | Made by | Used in |
| --- | --- | --- |
| **V8** | Google | Chrome, Edge, **Node.js**, Deno, Cloudflare Workers, Electron |
| **SpiderMonkey** | Mozilla | Firefox (the first JS engine ever, 1995) |
| **JavaScriptCore** | Apple | Safari, **Bun** |
| **Hermes** | Meta | React Native (compiles JS to bytecode ahead of time for fast mobile startup) |
| **QuickJS** | Fabrice Bellard | Small embedded devices and apps |

**Server-side runtimes**

| Runtime | Engine + core | Highlights |
| --- | --- | --- |
| **Node.js** | V8 + libuv (C/C++) | The most mature, largest ecosystem, most hosting support |
| **Deno** | V8 + tokio (Rust) | Created by Ryan Dahl (Node's creator) in 2018. Secure by default (explicit permissions), TypeScript built in, web-standard APIs, npm-compatible since Deno 2 |
| **Bun** | JavaScriptCore + Zig | Focus on speed. Runtime, package manager, bundler and test runner in one binary, largely Node-compatible |
| **Cloudflare Workers** (workerd) | V8 isolates | Edge/serverless. Web-standard APIs only, millisecond cold starts |

**Portability tip:** the more you use **web-standard APIs** (`fetch`, `Request`/`Response`, `URL`, `AbortController`, Web Streams, `crypto.subtle`, `structuredClone`), the more portable your code is across Node, Deno, Bun, edge runtimes and browsers. The WinterTC committee (Ecma TC55) works on standardizing this common server-side API surface.

> **Transferable:** Python works the same way. CPython is the reference runtime, PyPy uses a JIT, MicroPython targets microcontrollers. Same language, different engines and trade-offs. When a new runtime appears, ask: which engine, which event loop or concurrency model, which APIs, and how compatible is it?

---

## Part III — The core programming model

### 15. Asynchronous patterns: callbacks, promises and async/await

Node's async style evolved in three stages. You will meet all three in real code.

**1. Callbacks (error-first).** The original Node style: the last argument is a function called as `(error, result)`.

```javascript
import { readFile } from 'node:fs';

readFile('config.json', 'utf8', (err, data) => {
  if (err) return console.error('Could not read file:', err.message);
  console.log(JSON.parse(data));
});
```

Nesting many callbacks leads to "callback hell" (deep indentation, error handling repeated at every level).

**2. Promises.** A promise is an object representing a value that will be available later.

```mermaid
%%{init: {"htmlLabels": false, "flowchart": {"htmlLabels": false, "wrappingWidth": 400, "padding": 24}}}%%
stateDiagram-v2
    [*] --> Pending
    Pending --> Fulfilled : resolve(value)
    Pending --> Rejected : reject(error) or throw
    Fulfilled --> [*] : .then() handlers run
    Rejected --> [*] : .catch() handlers run
```

```javascript
import { readFile } from 'node:fs/promises';

readFile('config.json', 'utf8')
  .then((data) => JSON.parse(data))
  .then((config) => console.log(config))
  .catch((err) => console.error('Failed:', err.message));
```

**3. async/await.** Syntax on top of promises that reads like synchronous code. `await` pauses *this function* (not the thread) until the promise settles. The event loop keeps serving other work meanwhile.

```javascript
import { readFile } from 'node:fs/promises';

async function loadConfig(path) {
  try {
    const data = await readFile(path, 'utf8');
    return JSON.parse(data);
  } catch (err) {
    throw new Error(`Could not load config from ${path}`, { cause: err });
  }
}

const config = await loadConfig('config.json'); // top-level await works in ES Modules
```

**Converting old callback APIs:** `util.promisify` wraps any error-first callback function:

```javascript
import { promisify } from 'node:util';
import { exec } from 'node:child_process';

const execAsync = promisify(exec);
const { stdout } = await execAsync('git --version');
```

Most core modules also have promise versions built in: `node:fs/promises`, `node:timers/promises`, `node:dns/promises`, `node:stream/promises`, `node:readline/promises`.

**Sequential vs parallel.** The most common async performance bug is awaiting independent work one item at a time:

```javascript
// Slow: about 3 seconds if each call takes 1 second
const user = await getUser(id);
const orders = await getOrders(id);
const recommendations = await getRecommendations(id);

// Fast: about 1 second, all three run concurrently
const [user2, orders2, recommendations2] = await Promise.all([
  getUser(id),
  getOrders(id),
  getRecommendations(id),
]);
```

```mermaid
gantt
    title Three independent 1-second calls
    dateFormat X
    axisFormat %Ss
    section Sequential awaits
    getUser            : 0, 1
    getOrders          : 1, 2
    getRecommendations : 2, 3
    section Promise.all
    getUser            : 0, 1
    getOrders          : 0, 1
    getRecommendations : 0, 1
```

| Combinator | Resolves when | Rejects when | Use for |
| --- | --- | --- | --- |
| `Promise.all` | all fulfill (array of results) | **any** rejects (fails fast) | Everything is required |
| `Promise.allSettled` | all settle (array of `{status, value/reason}`) | never | Best effort, report each result |
| `Promise.race` | the first one settles | the first one rejects | Timeouts, first response wins |
| `Promise.any` | the first one fulfills | all reject (`AggregateError`) | Redundant sources, fallbacks |

**Cancellation and timeouts** use `AbortController` / `AbortSignal`, a web standard supported across Node APIs:

```javascript
// Give up on a slow request after 5 seconds
const res = await fetch('https://api.github.com/repos/nodejs/node', {
  signal: AbortSignal.timeout(5000),
});

// Cancel manually
const controller = new AbortController();
setTimeout(() => controller.abort(), 1000);
await someLongOperation({ signal: controller.signal });
```

> **Transferable:** `async/await` exists with nearly the same meaning in C#, Python, Rust, Kotlin (`suspend`), Swift and Dart. Promise is called `Task` (C#), `Future` (Rust, Dart, Java's `CompletableFuture`) or a coroutine/awaitable (Python). `Promise.all` is `Task.WhenAll` in C# and `asyncio.gather` in Python. Go takes a different route, with goroutines and channels and no colored async functions, but the sequential-vs-parallel lesson still applies.

### 16. Error handling

Errors in Node come in different flavors, and each needs its own handling style:

| Where the error happens | How to catch it |
| --- | --- |
| Synchronous code | `try { ... } catch (err) { ... }` |
| `async` function / `await` | `try/catch` around `await`, or `.catch()` on the promise |
| Error-first callback | Check the first argument: `if (err) ...` |
| EventEmitter / stream | Listen for the `'error'` event. **An unhandled `'error'` event crashes the process.** |

**Operational vs programmer errors.** This distinction decides what you do with an error:

- **Operational errors** are expected in a correct program: file not found, invalid user input, network timeout, database down. **Handle them**: retry, return a 4xx/5xx, show a message.
- **Programmer errors** are bugs: `undefined is not a function`, wrong argument types. **Don't try to recover.** Log them, let the process crash, and let a supervisor (Docker, systemd, Kubernetes, PM2) restart it in a clean state.

**Custom error classes** carry meaning (and HTTP status codes, in an API):

```javascript
export class NotFoundError extends Error {
  constructor(resource, id) {
    super(`${resource} '${id}' not found`);
    this.name = 'NotFoundError';
    this.status = 404;
  }
}

try {
  await db.query('...');
} catch (err) {
  // `cause` keeps the original error and stack trace for debugging
  throw new Error('Failed to load orders', { cause: err });
}
```

**Process-level safety nets.** Last-resort handlers. Log the error, then exit; never just carry on:

```javascript
process.on('uncaughtException', (err) => {
  console.error('Uncaught exception:', err);
  process.exit(1); // state may be corrupted: restart cleanly
});

process.on('unhandledRejection', (reason) => {
  console.error('Unhandled promise rejection:', reason);
  process.exit(1);
});
```

Since Node 15, an unhandled promise rejection **crashes the process by default**. A forgotten `await` or a missing `.catch()` is a real bug, not a warning.

**Error codes:** system errors have a `code` property (`ENOENT` file not found, `EACCES` permission denied, `EADDRINUSE` port in use, `ECONNREFUSED` nothing listening). Check `err.code`, not the message text, which can change between versions.

> **Transferable:** Java, C# and Python use exceptions much like JavaScript. Go returns errors as values (`if err != nil`), and Rust uses `Result<T, E>`, which forces you to handle them. The "operational vs programmer error" split and "crash and let a supervisor restart" (the Erlang/Elixir "let it crash" philosophy) apply everywhere.

### 17. Module systems: CommonJS and ES Modules

A module is a file with its own scope that **exports** some values and **imports** others. Node supports two systems.

**CommonJS (CJS)**, the original Node system:

```javascript
// math.cjs
function add(a, b) {
  return a + b;
}
module.exports = { add };
```

```javascript
// main.cjs
const { add } = require('./math.cjs');
const fs = require('node:fs');
console.log(add(2, 3)); // 5
```

**ES Modules (ESM)**, the official JavaScript standard (ES2015), used by browsers, Deno, Bun and modern Node code:

```javascript
// math.js
export const add = (a, b) => a + b;
export default function multiply(a, b) {
  return a * b;
}
```

```javascript
// main.js
import multiply, { add } from './math.js'; // the file extension is REQUIRED in ESM
import { readFile } from 'node:fs/promises';

console.log(add(2, 3), multiply(2, 3)); // 5 6

const { default: lodash } = await import('lodash'); // dynamic import: loads on demand, returns a promise
```

**How Node decides which system a file uses**

| File | Treated as |
| --- | --- |
| `.mjs` | Always ESM |
| `.cjs` | Always CommonJS |
| `.js` with `"type": "module"` in the nearest `package.json` | ESM |
| `.js` with `"type": "commonjs"` or no `type` field | CommonJS (recent versions also detect ESM syntax automatically, but be explicit) |

**Recommendation for new projects:** add `"type": "module"` to `package.json` and use `import`/`export` everywhere. (Recent npm versions write `"type": "commonjs"` when you run `npm init`; change it.)

**Key differences**

| | CommonJS | ES Modules |
| --- | --- | --- |
| Syntax | `require()` / `module.exports` | `import` / `export` |
| Loading | Synchronous, at runtime, can be called anywhere (even in an `if`) | Imports are static (analyzed before running); `import()` for dynamic loading |
| Top-level `await` | No | Yes |
| File extensions in relative paths | Optional (`./math`) | Required (`./math.js`) |
| `__dirname`, `__filename` | Available | Use `import.meta.dirname` / `import.meta.filename` `(Node ≥ 20.11)` |
| Tree shaking by bundlers | Hard | Easy (static structure) |
| Interop | `require()` of ESM works `(Node ≥ 22.12 / 20.19)` if the module has no top-level `await` | Can `import` CommonJS packages (`module.exports` becomes the default export) |

**The `node:` prefix.** Import core modules as `node:fs`, `node:path` and so on. It makes clear the module is built in and cannot be confused with an npm package of the same name.

**JSON modules:** `import data from './data.json' with { type: 'json' };`. In CommonJS, `require('./data.json')` just works.

**How a module name is resolved:**

```mermaid
%%{init: {"htmlLabels": false, "flowchart": {"htmlLabels": false, "wrappingWidth": 400, "padding": 24}}}%%
flowchart TD
    REQ["import x from<br/>'specifier'"] --> CORE{"Built-in?<br/>'node:fs', 'fs',<br/>'http'..."}
    CORE -- "yes" --> LOADCORE["Load the core module"]
    CORE -- "no" --> REL{"Starts with<br/>'./', '../' or '/'?"}
    REL -- "yes" --> FILE["Resolve relative to<br/>the current file<br/>(ESM needs the exact<br/>file name)"]
    REL -- "no, a bare name<br/>like 'express'" --> NM["Look in<br/>./node_modules/express<br/>then ../node_modules<br/>up to the root"]
    NM --> EXPORTS["Read its package.json<br/>'exports' (or 'main')<br/>points to the entry file"]
```

**Modules are cached.** A module runs **once**, the first time it is imported, and every later import gets the same instance. That makes module-level variables effectively singletons (useful for a shared DB pool or config; dangerous for per-request state).

> **Transferable:** The same concepts exist everywhere under different names: Python's `import` and packages (`sys.path` is the equivalent of `node_modules` resolution), Java packages and the classpath, Go modules, Rust crates and `use`. A module system always answers three questions: how do I expose code, how do I consume it, and where does the runtime look for it?

### 18. Globals, the process object and web-standard APIs

Some values are available everywhere without importing.

**`process`**: information about, and control over, the running process:

```javascript
process.argv;           // command-line arguments: ['node path', 'script path', ...args]
process.env.PORT;       // environment variables (always strings, or undefined)
process.cwd();          // directory the process was started from
process.pid;            // process id
process.platform;       // 'linux' | 'darwin' | 'win32'
process.uptime();       // seconds since start
process.exitCode = 1;   // set the exit code and let Node exit naturally (preferred)
process.exit(1);        // exit immediately (pending async work is lost)
process.on('SIGTERM', () => { /* graceful shutdown */ });
```

**Parsing CLI arguments** without a library, using `util.parseArgs` `(Node ≥ 20)`:

```javascript
// greet.js  →  node greet.js --name Ada --shout
import { parseArgs } from 'node:util';

const { values } = parseArgs({
  options: {
    name: { type: 'string', short: 'n', default: 'world' },
    shout: { type: 'boolean', default: false },
  },
});

const message = `Hello, ${values.name}!`;
console.log(values.shout ? message.toUpperCase() : message);
```

**Exit codes:** `0` means success and anything else means failure. Shells, CI pipelines, Docker and Kubernetes all rely on them.

**Web-standard globals** available in modern Node (the same APIs as in browsers):

| Global | Purpose |
| --- | --- |
| `fetch`, `Request`, `Response`, `Headers`, `FormData` | HTTP client `(stable since Node 21)` |
| `URL`, `URLSearchParams` | Parse and build URLs safely |
| `AbortController`, `AbortSignal` | Cancellation and timeouts |
| `TextEncoder`, `TextDecoder` | Convert strings to and from bytes |
| `structuredClone()` | Deep-copy objects |
| `crypto.randomUUID()`, `crypto.subtle` | Web Crypto |
| `ReadableStream`, `WritableStream` | Web Streams |
| `WebSocket` | WebSocket client `(Node ≥ 22)` |
| `EventTarget`, `Event`, `BroadcastChannel` | Events and messaging |
| `performance` | High-resolution timing |
| `setTimeout`, `setInterval`, `queueMicrotask` | Scheduling |

**Not available in Node:** `window`, `document`, `localStorage` and other browser-only DOM APIs. `globalThis` refers to the global object in every environment.

---

## Part IV — Core modules

### 19. Core modules tour and the file system

Core modules ship with Node, so there is nothing to install. The full list is in the [API docs](https://nodejs.org/api/).

| Module | Purpose |
| --- | --- |
| `node:fs`, `node:fs/promises` | Files and directories |
| `node:path` | Build and parse file paths portably |
| `node:url` | URL utilities (`fileURLToPath`, `pathToFileURL`) |
| `node:os` | OS info: CPUs, memory, hostname, temp dir |
| `node:events` | `EventEmitter`, the base of much of Node |
| `node:stream` | Streaming data (files, HTTP bodies, compression) |
| `node:buffer` | Binary data |
| `node:http`, `node:https`, `node:http2` | HTTP servers and clients |
| `node:net`, `node:dgram` | Raw TCP and UDP |
| `node:crypto` | Hashing, encryption, random values, UUIDs |
| `node:zlib` | gzip, deflate, brotli compression |
| `node:child_process` | Run other programs |
| `node:worker_threads` | Run JavaScript in parallel threads |
| `node:cluster` | Run several Node processes sharing a port |
| `node:util` | `promisify`, `parseArgs`, `inspect`, `styleText`... |
| `node:test`, `node:assert` | Testing |
| `node:readline` | Read input line by line (files, terminal) |
| `node:perf_hooks` | Performance measurement |
| `node:timers/promises` | `await setTimeout(1000)` |

**The file system module comes in three styles.** Prefer the promise API.

```javascript
// 1. Promises (recommended)
import { mkdir, readFile, writeFile, readdir, rm } from 'node:fs/promises';

await mkdir('output', { recursive: true });
await writeFile('output/hello.txt', 'Hello, Node.js!\n');
await writeFile('output/hello.txt', 'Another line\n', { flag: 'a' }); // append
const text = await readFile('output/hello.txt', 'utf8');
console.log(text);
console.log(await readdir('output'));
await rm('output', { recursive: true, force: true });

// 2. Callbacks (older code)
import { readFile as readFileCb } from 'node:fs';
readFileCb('package.json', 'utf8', (err, data) => { /* ... */ });

// 3. Synchronous: blocks the event loop. OK for scripts and startup, not for request handlers.
import { readFileSync, existsSync } from 'node:fs';
if (existsSync('package.json')) console.log(readFileSync('package.json', 'utf8').length);
```

For files larger than a few MB, use **streams** ([topic 23](#23-streams-and-backpressure)) instead of reading everything into memory.

### 20. path, url and os

**`node:path`** builds paths that work on every OS. Windows uses `\`, POSIX uses `/`, and string concatenation breaks across platforms:

```javascript
import path from 'node:path';

path.join('data', 'users', '42.json');      // 'data/users/42.json' (or 'data\\users\\42.json' on Windows)
path.resolve('data', 'file.txt');           // absolute path from the current working directory
path.basename('/tmp/report.pdf');           // 'report.pdf'
path.extname('photo.final.jpg');            // '.jpg'
path.dirname('/var/log/app.log');           // '/var/log'
path.parse('/home/ada/notes.md');           // { root, dir, base, ext, name }
```

**Paths relative to the current file, not the working directory**, a classic bug source:

```javascript
import path from 'node:path';

// ESM (Node ≥ 20.11)
const templatePath = path.join(import.meta.dirname, 'templates', 'email.html');

// Older ESM equivalent:
// import { fileURLToPath } from 'node:url';
// const __dirname = path.dirname(fileURLToPath(import.meta.url));
```

`process.cwd()` is where the user *ran* the command; `import.meta.dirname` is where the *file* lives. They are often different.

**`node:os`**: information about the machine:

```javascript
import os from 'node:os';

console.log({
  platform: os.platform(),                  // 'linux', 'darwin', 'win32'
  cpus: os.availableParallelism(),          // how many things can truly run in parallel
  totalMemGB: (os.totalmem() / 1024 ** 3).toFixed(1),
  freeMemGB: (os.freemem() / 1024 ** 3).toFixed(1),
  home: os.homedir(),
  tmp: os.tmpdir(),
  eol: JSON.stringify(os.EOL),              // '\n' or '\r\n'
});
```

### 21. Events and EventEmitter

Much of Node is built on the **observer pattern**: an object emits named events and listeners react. HTTP servers emit `'request'`, streams emit `'data'` and `'end'`, processes emit `'exit'`.

```javascript
import { EventEmitter, once } from 'node:events';

class OrderService extends EventEmitter {
  place(order) {
    // ...save the order...
    this.emit('order:placed', order);
  }
}

const orders = new OrderService();

orders.on('order:placed', (order) => console.log(`Email receipt for order ${order.id}`));
orders.on('order:placed', (order) => console.log(`Update stock for ${order.items.length} items`));
orders.once('order:placed', () => console.log('First order of the day!')); // runs only once

orders.place({ id: 1, items: ['book', 'pen'] });
orders.place({ id: 2, items: ['lamp'] });

// Wait for a single event with a promise
setTimeout(() => orders.place({ id: 3, items: [] }), 100);
const [order] = await once(orders, 'order:placed');
console.log('Awaited order', order.id);
```

Important details:

- Listeners run **synchronously**, in registration order, when `emit()` is called. `emit` is not async.
- Emitting `'error'` with no `'error'` listener **throws** and crashes the process. Always listen for `'error'` on emitters and streams.
- Node warns about a possible leak when more than 10 listeners are attached to one event (`MaxListenersExceededWarning`). It usually means listeners are added on every request and never removed.
- Remove listeners with `emitter.off(name, fn)`.

> **Transferable:** The same pattern appears as C# `event`s and delegates, Java listeners and `PropertyChangeSupport`, Python's `blinker` or Django signals, and the browser's `addEventListener`. At the architecture level it becomes pub/sub and message brokers (Redis Pub/Sub, Kafka, RabbitMQ).

### 22. Buffers and binary data

JavaScript strings are text. Files, sockets and images are **bytes**. A `Buffer` is Node's byte array (a subclass of the standard `Uint8Array`).

```javascript
const buf = Buffer.from('héllo', 'utf8');
console.log(buf);                       // <Buffer 68 c3 a9 6c 6c 6f>
console.log('héllo'.length, buf.length); // 5 characters, but 6 bytes: 'é' takes 2 bytes in UTF-8

console.log(buf.toString('base64'));    // 'aMOpbGxv'
console.log(buf.toString('hex'));       // '68c3a96c6c6f'
console.log(Buffer.from('aMOpbGxv', 'base64').toString('utf8')); // 'héllo'

const zeros = Buffer.alloc(16);          // 16 zero-filled bytes
const joined = Buffer.concat([Buffer.from('ab'), Buffer.from('cd')]); // 'abcd'
```

Things to know:

- **Character count is not byte count.** Validate upload sizes and `Content-Length` in bytes.
- **Encodings:** `utf8` (default), `base64`, `base64url`, `hex`, `latin1`, `utf16le`.
- `Buffer.allocUnsafe(n)` is faster but may contain old memory contents. Only use it if you overwrite every byte.
- When you read a file without an encoding (`readFile(path)`), you get a `Buffer`; with `'utf8'` you get a string.

> **Transferable:** Every language separates text from bytes: Python `str` vs `bytes`, Go `string` vs `[]byte`, Java `String` vs `byte[]`, Rust `String` vs `Vec<u8>`. Most "weird character" bugs anywhere come from mixing up encodings at this boundary.

### 23. Streams and backpressure

Streams process data **piece by piece** (in chunks) instead of loading everything into memory. They let a small server compress a 10 GB file or proxy a video with constant, low memory.

| Type | Role | Examples |
| --- | --- | --- |
| **Readable** | Source of data | `fs.createReadStream`, an HTTP request on the server, `process.stdin` |
| **Writable** | Destination | `fs.createWriteStream`, an HTTP response, `process.stdout` |
| **Duplex** | Both, independently | A TCP socket |
| **Transform** | Duplex that modifies data passing through | `zlib.createGzip()`, encryption, CSV parsing |

```mermaid
%%{init: {"htmlLabels": false, "flowchart": {"htmlLabels": false, "wrappingWidth": 400, "padding": 24}}}%%
flowchart LR
    R["Readable<br/>fs.createReadStream()<br/>access.log"] -- "chunk" --> T["Transform<br/>zlib.createGzip()"]
    T -- "chunk" --> W["Writable<br/>fs.createWriteStream()<br/>access.log.gz"]
    W -. "buffer full: pause<br/>(backpressure)" .-> R
    W -. "'drain': resume" .-> R
```

**Backpressure** is the mechanism that slows a fast producer down to the speed of a slow consumer (e.g. a fast disk feeding a slow network client). Without it, data piles up in memory. `pipeline()` handles backpressure, errors and cleanup for you:

```javascript
import { createReadStream, createWriteStream } from 'node:fs';
import { pipeline } from 'node:stream/promises';
import { createGzip } from 'node:zlib';

await pipeline(
  createReadStream('access.log'),
  createGzip(),
  createWriteStream('access.log.gz'),
);
console.log('Compressed with constant memory, whatever the file size.');
```

**Reading a huge file line by line** with async iteration:

```javascript
import { createReadStream } from 'node:fs';
import { createInterface } from 'node:readline';

const lines = createInterface({ input: createReadStream('big.csv'), crlfDelay: Infinity });

let count = 0;
for await (const line of lines) {
  if (line.includes('ERROR')) count++;
}
console.log(`${count} error lines`);
```

**Streaming an HTTP response** (memory stays flat even for huge files):

```javascript
import { createServer } from 'node:http';
import { createReadStream } from 'node:fs';
import { pipeline } from 'node:stream/promises';

createServer(async (req, res) => {
  res.setHeader('content-type', 'video/mp4');
  await pipeline(createReadStream('movie.mp4'), res).catch(() => res.destroy());
}).listen(3000);
```

Prefer `pipeline()` over the older `.pipe()`, which does not forward errors or clean up on failure.

> **Transferable:** Streams are the same idea as Unix pipes (`cat access.log | gzip > access.log.gz`), Java's `InputStream`/`OutputStream`, Go's `io.Reader`/`io.Writer`, and Python generators or file iteration. Reactive frameworks (RxJS, Reactor) and message systems (Kafka) apply backpressure at larger scales.

### 24. HTTP from scratch and the fetch client

Before using a framework, it helps to see what one does for you. Here is a tiny JSON API using only `node:http`:

```javascript
import { createServer } from 'node:http';

const server = createServer(async (req, res) => {
  const url = new URL(req.url, `http://${req.headers.host}`);

  if (req.method === 'GET' && url.pathname === '/hello') {
    const name = url.searchParams.get('name') ?? 'world';
    res.writeHead(200, { 'content-type': 'application/json' });
    return res.end(JSON.stringify({ message: `Hello, ${name}!` }));
  }

  if (req.method === 'POST' && url.pathname === '/echo') {
    let body = '';
    for await (const chunk of req) body += chunk; // the request body is a readable stream
    res.writeHead(200, { 'content-type': 'application/json' });
    return res.end(body);
  }

  res.writeHead(404, { 'content-type': 'application/json' });
  res.end(JSON.stringify({ error: 'Not found' }));
});

server.listen(3000, () => console.log('Listening on http://localhost:3000'));
```

```bash
curl "http://localhost:3000/hello?name=Ada"
curl -X POST http://localhost:3000/echo -H "Content-Type: application/json" -d '{"ok":true}'
```

**The life of a request:**

```mermaid
sequenceDiagram
    participant C as Client
    participant K as OS kernel + libuv
    participant P as llhttp parser
    participant H as Your handler (JS)
    C->>K: TCP connection + request bytes
    K->>P: data ready (poll phase)
    P->>H: emits 'request' with req and res objects
    H->>H: route, read body, do work (DB, files...)
    H->>K: res.end(body)
    K->>C: response bytes
```

This works, but notice what is missing: routing with parameters (`/users/:id`), body size limits, JSON parsing with error handling, validation, middleware for logging and auth, centralized error handling. **That is exactly what frameworks like Express add** ([Part VI](#part-vi--building-a-real-backend-api-with-express)).

**HTTP client: `fetch`** is built in (no `axios` needed for most cases):

```javascript
const res = await fetch('https://api.github.com/repos/nodejs/node', {
  headers: { 'user-agent': 'understanding-nodejs-well' },
  signal: AbortSignal.timeout(5000),
});
if (!res.ok) throw new Error(`HTTP ${res.status}`); // fetch does NOT throw on 4xx/5xx
const repo = await res.json();
console.log(`${repo.full_name} has ${repo.stargazers_count} stars`);

// POST JSON
await fetch('https://httpbin.org/post', {
  method: 'POST',
  headers: { 'content-type': 'application/json' },
  body: JSON.stringify({ hello: 'world' }),
});
```

> **Transferable:** Python's `http.server`, Go's `net/http` and Java's `HttpServer` sit at the same "bare" level, with frameworks (Flask/FastAPI, Gin, Spring) built on top. Go's standard library is good enough that many Go services skip frameworks entirely. Knowing the raw HTTP layer makes every framework easier to understand.

### 25. Concurrency for CPU work: child processes, worker threads and cluster

The event loop handles *waiting* well, but CPU work blocks it. Node offers three ways to use more CPU cores:

```mermaid
%%{init: {"htmlLabels": false, "flowchart": {"htmlLabels": false, "wrappingWidth": 400, "padding": 24}}}%%
flowchart TB
    subgraph WT["Worker threads"]
        MAIN["Main thread<br/>event loop + your server"] <-- "postMessage" --> W1["Worker 1<br/>own V8 isolate<br/>+ event loop"]
        MAIN <-- "postMessage" --> W2["Worker 2"]
    end
    subgraph CL["Cluster (many processes)"]
        PRIMARY["Primary process"] --> P1["Worker process 1<br/>full copy of your app"]
        PRIMARY --> P2["Worker process 2"]
        PRIMARY --> P3["Worker process N"]
    end
    subgraph CP["Child process"]
        NODEP["Node process"] -- "spawn / execFile" --> EXT["ffmpeg, git, python,<br/>shell scripts..."]
    end
```

| Tool | Use when | Communication | Memory |
| --- | --- | --- | --- |
| `worker_threads` | CPU-heavy **JavaScript** (parsing, hashing, image processing in JS) | `postMessage`, `SharedArrayBuffer` | Lower (same process) |
| `cluster` | Using every core for an HTTP server on **one machine** | IPC messages | One full app copy per process |
| `child_process` | Running **another program** or script | stdin/stdout/stderr, IPC | Separate process |

**Worker threads.** Two CPU-heavy jobs run in parallel while the main thread stays responsive:

```javascript
// fib.js
import { Worker, isMainThread, parentPort, workerData } from 'node:worker_threads';

const fib = (n) => (n < 2 ? n : fib(n - 1) + fib(n - 2)); // deliberately slow (CPU-bound)

if (isMainThread) {
  const runInWorker = (n) =>
    new Promise((resolve, reject) => {
      const worker = new Worker(new URL(import.meta.url), { workerData: n });
      worker.once('message', resolve);
      worker.once('error', reject);
    });

  const ticker = setInterval(() => console.log('main thread is still responsive...'), 250);
  console.time('two fib(40) in parallel');
  const results = await Promise.all([runInWorker(40), runInWorker(40)]);
  console.timeEnd('two fib(40) in parallel'); // about the time of ONE fib(40)
  clearInterval(ticker);
  console.log(results); // [ 102334155, 102334155 ]
} else {
  parentPort.postMessage(fib(workerData));
}
```

Creating a worker costs some milliseconds and memory, so real apps keep a **pool** of workers (e.g. the `piscina` package) instead of one per task.

**Cluster.** One process per CPU core, all listening on the same port:

```javascript
import cluster from 'node:cluster';
import { createServer } from 'node:http';
import { availableParallelism } from 'node:os';

if (cluster.isPrimary) {
  for (let i = 0; i < availableParallelism(); i++) cluster.fork();
  cluster.on('exit', (worker) => {
    console.log(`Worker ${worker.process.pid} died, starting a new one`);
    cluster.fork();
  });
} else {
  createServer((req, res) => res.end(`Handled by process ${process.pid}\n`)).listen(3000);
}
```

In containers and Kubernetes, the usual approach is **one Node process per container**, with more containers (replicas) instead of `cluster` ([topic 45](#45-performance-scaling-and-deployment)). Tools like PM2 provide cluster mode plus restarts on plain servers.

**Child processes.** Prefer `execFile`/`spawn` with an argument array. `exec` runs a shell, and putting user input into a shell command is a command-injection vulnerability.

```javascript
import { execFile, spawn } from 'node:child_process';
import { promisify } from 'node:util';

const execFileAsync = promisify(execFile);
const { stdout } = await execFileAsync('git', ['log', '--oneline', '-5']); // buffered output
console.log(stdout);

const child = spawn('ping', ['-c', '3', 'nodejs.org']); // streamed output (use '-n' on Windows)
child.stdout.pipe(process.stdout);
child.on('close', (code) => console.log(`exited with code ${code}`));
```

> **Transferable:** Python has the same split for the same kind of reason (the GIL): `threading`, `multiprocessing` and `subprocess`. Java and C# use real shared-memory threads and thread pools. Go multiplexes goroutines over all cores automatically. Whatever the language, decide first whether the work is **I/O-bound** (async is enough) or **CPU-bound** (you need parallelism).

### 26. The built-in test runner

Node ships a test runner `(stable since Node 20)` and an assertion library, so small and medium projects need no testing dependencies.

```javascript
// src/price.js
export function applyDiscount(price, percent) {
  if (percent < 0 || percent > 100) throw new RangeError('percent must be between 0 and 100');
  return Math.round(price * (100 - percent)) / 100;
}
```

```javascript
// test/price.test.js
import assert from 'node:assert/strict';
import { describe, it, mock } from 'node:test';
import { applyDiscount } from '../src/price.js';

describe('applyDiscount', () => {
  it('applies a percentage discount', () => {
    assert.equal(applyDiscount(200, 25), 150);
  });

  it('rejects invalid percentages', () => {
    assert.throws(() => applyDiscount(100, 120), RangeError);
  });

  it('can mock functions', () => {
    const sendEmail = mock.fn();
    sendEmail('ada@example.com');
    assert.equal(sendEmail.mock.callCount(), 1);
  });
});
```

```bash
node --test                 # finds **/*.test.js, **/test/**/*.js and similar patterns
node --test --watch         # re-run on changes
node --test --experimental-test-coverage   # coverage report (flag name may change in future versions)
```

It also supports `before`/`after`/`beforeEach` hooks, `t.skip()`/`t.todo()`, mocking timers (`mock.timers`), snapshots and parallel test files. Popular alternatives with more features are **Vitest** and **Jest**.

**What to test:** pure logic with unit tests, HTTP endpoints with integration tests ([topic 38](#38-testing-the-api)), and a few end-to-end tests for critical flows. That is the "testing pyramid": many fast tests at the bottom, few slow ones at the top.

> **Transferable:** `describe/it/assert` maps directly to `pytest` (Python), JUnit (Java), xUnit/NUnit (C#), `go test` and `cargo test`. Test structure (arrange, act, assert), the testing pyramid and mocking at boundaries are the same in every language.

---

## Part V — Packages, tooling and workflow

### 27. npm and the anatomy of package.json

**npm** (Node Package Manager) is three things: the **CLI** bundled with Node, the **registry** at [npmjs.com](https://www.npmjs.com/) (millions of public packages), and the **`package.json` format** that describes a project.

Create a project:

```bash
mkdir my-app && cd my-app
npm init -y           # creates package.json with defaults
npm pkg set type=module
```

A typical `package.json`:

```json
{
  "name": "tasks-api",
  "version": "1.0.0",
  "private": true,
  "type": "module",
  "main": "src/server.js",
  "engines": { "node": ">=22.12" },
  "scripts": {
    "dev": "node --watch --env-file-if-exists=.env src/server.js",
    "start": "node src/server.js",
    "test": "node --test",
    "lint": "eslint ."
  },
  "dependencies": {
    "express": "^5.1.0",
    "zod": "^4.0.0"
  },
  "devDependencies": {
    "eslint": "^9.0.0",
    "supertest": "^7.0.0"
  }
}
```

| Field | Meaning |
| --- | --- |
| `name`, `version` | Package identity. Required if you publish |
| `private: true` | Prevents accidental `npm publish` of an application |
| `type` | `"module"` for ES Modules, `"commonjs"` otherwise ([topic 17](#17-module-systems-commonjs-and-es-modules)) |
| `main` / `exports` | Entry point when someone imports your package. `exports` can also hide internal files |
| `engines` | Supported Node versions. Documents the requirement, and hosts and tools can enforce it |
| `scripts` | Named commands: `npm run <name>` ([topic 31](#31-scripts-and-hot-reload)) |
| `dependencies` | Needed at **runtime** (Express, a DB driver) |
| `devDependencies` | Needed only for **development** (test tools, linters, TypeScript) |
| `peerDependencies` | For plugins: "the host app must provide this" (e.g. a React component library) |
| `optionalDependencies` | Install if possible, ignore failure |
| `overrides` | Force a specific version of a transitive dependency (e.g. a security fix) |
| `workspaces` | Several packages in one repository (monorepo) |
| `bin` | Commands your package installs (for CLIs) |
| `files` | What gets included when publishing |
| `packageManager` | Which package manager and version the project uses (e.g. `"pnpm@10.0.0"`) |

> **Transferable:** Every ecosystem has a manifest: `pyproject.toml` (Python), `Cargo.toml` (Rust), `go.mod` (Go), `pom.xml`/`build.gradle` (Java), `*.csproj` (.NET), `composer.json` (PHP), `Gemfile` (Ruby). They all declare name, version, dependencies, scripts or tasks, and runtime version.

### 28. Semantic versioning and lockfiles

**Semantic versioning (SemVer):** `MAJOR.MINOR.PATCH`

- **MAJOR**: breaking changes (code that used it may need to change)
- **MINOR**: new features, backwards compatible
- **PATCH**: bug fixes, backwards compatible

**Version ranges in `package.json`:**

| Range | Means | Example matches for `5.1.0` |
| --- | --- | --- |
| `^5.1.0` (npm's default) | Same major: `>=5.1.0 <6.0.0` | 5.1.1, 5.3.0, but not 6.0.0 |
| `~5.1.0` | Same minor: `>=5.1.0 <5.2.0` | 5.1.9, but not 5.2.0 |
| `5.1.0` | Exactly this version | only 5.1.0 |
| `>=5.1.0` | Anything newer | includes 6.x, 7.x... (risky) |
| `^0.4.2` | **Special case for 0.x:** `>=0.4.2 <0.5.0` | 0.x versions treat MINOR as breaking |

**Ranges are flexible, so you need a lockfile for reproducibility:**

```mermaid
%%{init: {"htmlLabels": false, "flowchart": {"htmlLabels": false, "wrappingWidth": 400, "padding": 24}}}%%
flowchart LR
    PJ["package.json<br/>express: ^5.1.0<br/>(the range you allow)"] -- "npm install<br/>picks the newest match" --> LOCK["package-lock.json<br/>express 5.3.0 + every<br/>sub-dependency, with<br/>exact versions and<br/>integrity hashes"]
    LOCK -- "npm ci<br/>installs exactly<br/>this tree" --> NM["node_modules/<br/>identical on<br/>every machine"]
```

- `package-lock.json` records the **exact** version of every package in the tree, including dependencies of dependencies, plus integrity hashes.
- **Commit the lockfile** to Git. Never commit `node_modules/`.
- Use **`npm install`** while developing (it may update the lockfile) and **`npm ci`** in CI and Docker (it installs exactly what the lockfile says, fails if `package.json` and the lockfile disagree, and starts from a clean `node_modules`).

> **Transferable:** SemVer is the universal language of versioning (Cargo, Composer, Go modules, NuGet and most others use it). Lockfiles exist everywhere: `Cargo.lock`, `poetry.lock`/`uv.lock`, `go.sum`, `Gemfile.lock`, `composer.lock`, `packages.lock.json`. The rule is the same: ranges in the manifest, exact versions in the lockfile, commit both.

### 29. Installing and managing dependencies

```bash
npm install express            # add a runtime dependency (shorthand: npm i express)
npm install -D supertest       # add a dev dependency (--save-dev)
npm install express@4          # a specific major version
npm install                    # install everything listed in package.json
npm ci                         # clean, exact install from the lockfile (CI/Docker)
npm uninstall lodash           # remove a package (from node_modules AND package.json)

npm ls                         # dependency tree (npm ls --depth=0 for the top level)
npm outdated                   # what has newer versions
npm update                     # update within the allowed ranges
npm explain ms                 # why is this package installed?
npm view express version       # latest published version
npm audit                      # known vulnerabilities in your tree
npm audit fix                  # apply compatible fixes
```

**What each file and folder is**

- **`package.json`**: your declared dependencies (ranges). You edit this, or let npm commands do it.
- **`package-lock.json`**: the exact resolved tree. Generated by npm, committed to Git.
- **`node_modules/`**: the installed code. Generated, **never committed**. Add it to `.gitignore`.

**Using a package:**

```javascript
import _ from 'lodash';

const numbers = [1, 2, 3, 4, 5];
console.log(_.chunk(numbers, 2)); // [[1, 2], [3, 4], [5]]
```

**Check the platform before adding a dependency.** Modern JavaScript and Node already cover many old library use cases: `structuredClone` (deep clone), `Object.groupBy` (grouping), `Array.prototype.toSorted`/`findLast`, `fetch` (HTTP), `crypto.randomUUID` (UUIDs), `util.parseArgs` (CLI args), `node:test` (testing), `--env-file` (dotenv). Every dependency is code you trust and must keep updated.

**Global installs and `npx`.** `npm install -g <pkg>` installs a CLI system-wide. Prefer `npx <pkg>`, which runs a package's command without a global install, so the version can be pinned per project:

```bash
npx eslint .                       # uses the project's local eslint
npx create-vite@latest my-frontend # run a generator once without installing it
```

### 30. Alternative package managers: Yarn, pnpm and Bun

All of them read `package.json` and install from the npm registry. They differ in speed, disk usage and strictness.

| | **npm** | **Yarn** | **pnpm** | **Bun** |
| --- | --- | --- | --- | --- |
| Comes with Node | Yes | No | No | No (separate runtime) |
| Install strategy | Flat `node_modules` | Flat `node_modules` or Plug'n'Play (no `node_modules`) | Global content-addressable store, hard-linked into a **strict** `node_modules` | Flat `node_modules`, very fast |
| Disk usage across projects | Each project has its own copy | Similar (PnP is smaller) | **Each version stored once on disk** | Global cache |
| Strictness | Can import undeclared ("phantom") dependencies | PnP is strict | **Strict**: you can only import what you declare | Like npm |
| Workspaces / monorepos | Yes | Yes (a long-time strength) | Yes (excellent) | Yes |
| Lockfile | `package-lock.json` | `yarn.lock` | `pnpm-lock.yaml` | `bun.lock` |

**Command equivalents**

| Task | npm | Yarn | pnpm | Bun |
| --- | --- | --- | --- | --- |
| Install all | `npm install` | `yarn` | `pnpm install` | `bun install` |
| Add | `npm i express` | `yarn add express` | `pnpm add express` | `bun add express` |
| Add dev | `npm i -D vitest` | `yarn add -D vitest` | `pnpm add -D vitest` | `bun add -d vitest` |
| Remove | `npm uninstall x` | `yarn remove x` | `pnpm remove x` | `bun remove x` |
| Run script | `npm run dev` | `yarn dev` | `pnpm dev` | `bun run dev` |
| CI install | `npm ci` | `yarn install --immutable` | `pnpm install --frozen-lockfile` | `bun install --frozen-lockfile` |
| One-off command | `npx` | `yarn dlx` | `pnpm dlx` | `bunx` |

**Installing them:** `npm install -g pnpm` (or `yarn`). **Corepack** is a tool that reads the `packageManager` field in `package.json` and runs exactly that manager and version. It shipped with Node 14.19 through 24. **Since Node 25 it is no longer bundled**; install it with `npm install -g corepack` if you want it.

**Which one?** npm is always available and perfectly fine. pnpm is a great choice for monorepos and large dependency trees (speed, disk, strictness). Use whatever the project already uses: **never mix lockfiles** in one repository.

> **Transferable:** Python went through the same evolution: pip, then pipenv/poetry, now `uv` (fast, written in Rust). Each generation made installs faster, more reproducible and stricter. Expect the JavaScript tooling landscape to keep changing too; the concepts (manifest, lockfile, registry, cache, workspace) stay the same.

### 31. Scripts and hot reload

**npm scripts** turn long commands into short, shared names. They also put `node_modules/.bin` on the PATH, so locally installed tools just work:

```json
{
  "scripts": {
    "dev": "node --watch --env-file-if-exists=.env src/server.js",
    "start": "node src/server.js",
    "test": "node --test",
    "lint": "eslint .",
    "format": "prettier --write .",
    "check": "npm run lint && npm test"
  }
}
```

```bash
npm run dev        # any script
npm start          # shortcuts exist for start and test
npm test
node --run dev     # Node ≥ 22: runs the script directly, without npm's startup overhead
```

`pre<name>` and `post<name>` scripts run automatically before and after `<name>` (e.g. `pretest` runs before `test`).

**Hot reload during development.** Restart the app whenever you save:

- **Built in:** `node --watch src/server.js` `(stable since Node 22)`. It watches the files your app imports. Add `--watch-path=./src` to watch a folder explicitly.
- **nodemon:** the classic tool, still useful for older Node versions or custom setups (watching non-JS files, running other commands):

  ```bash
  npm install -D nodemon
  ```

  ```json
  { "scripts": { "dev": "nodemon src/server.js" } }
  ```

Stop either one with `Ctrl + C`.

### 32. Environment variables and configuration

**Configuration that varies between environments** (ports, database URLs, API keys, log levels) belongs in **environment variables**, not in code. This is one of the "Twelve-Factor App" principles, and every hosting platform, Docker and Kubernetes supports it.

```javascript
const port = Number(process.env.PORT ?? 3000); // env values are ALWAYS strings (or undefined)
```

**Local development with a `.env` file:**

```bash
# .env  (NEVER commit this file)
PORT=3000
DATABASE_URL=postgres://user:secret@localhost:5432/app
```

```bash
node --env-file=.env src/server.js            # Node ≥ 20.6, errors if the file is missing
node --env-file-if-exists=.env src/server.js  # Node ≥ 22.9, fine if it is missing (e.g. in production)
```

Or from code: `process.loadEnvFile('.env')`. On older versions, use the `dotenv` package.

**Good practices:**

1. Commit a **`.env.example`** with every variable name and safe placeholder values. Add `.env` to `.gitignore`.
2. **Validate configuration at startup** and fail fast with a clear message, rather than crashing later on the first request that needs a value. The API in [topic 37](#37-building-the-api-step-by-step) does this with a schema.
3. Read `process.env` in **one** config module and import it elsewhere. Don't scatter `process.env.X` across the codebase.
4. In production, use the platform's secret storage (Docker/Kubernetes secrets, AWS Secrets Manager, Vault...), not files in the image.

**`NODE_ENV`** is a convention: `production` makes Express and many libraries skip debug work (and `npm ci --omit=dev` skips dev dependencies). Set `NODE_ENV=production` in production.

**Setting variables per command:**

```bash
PORT=4000 node src/server.js                      # macOS / Linux / Git Bash
```

```powershell
$env:PORT = 4000; node src/server.js              # Windows PowerShell
```

> **Transferable:** Every stack reads environment variables the same way (`os.environ` in Python, `os.Getenv` in Go, `System.getenv` in Java, `Environment.GetEnvironmentVariable` in .NET). "Config in the environment, validated at startup, secrets never in Git" applies everywhere.

### 33. TypeScript with Node.js

**TypeScript** adds static types to JavaScript. It catches whole classes of bugs before running the code, and makes editors far more helpful (autocomplete, safe renames). Most professional Node codebases use it.

**Running TypeScript directly** `(Node ≥ 22.18 / 23.6)`: Node can execute `.ts` files by **stripping the type annotations**, with no build step:

```typescript
// greet.ts
interface User {
  name: string;
  age: number;
}

export function greet(user: User): string {
  return `Hello, ${user.name} (${user.age})`;
}

console.log(greet({ name: 'Ada', age: 36 }));
```

```bash
node greet.ts
```

Limits of type stripping, which are worth knowing:

- Node **does not type-check**. It only removes types. Run `tsc --noEmit` (in CI and in your editor) to check them.
- Only "erasable" syntax works by default. Features that generate runtime code (`enum`, `namespace` with values, constructor parameter properties) need `--experimental-transform-types`, or avoid them (prefer union types and `as const` objects).
- Relative imports must use the real `.ts` extension: `import { greet } from './greet.ts'`.

A `tsconfig.json` that matches this setup:

```json
{
  "compilerOptions": {
    "target": "esnext",
    "module": "nodenext",
    "noEmit": true,
    "strict": true,
    "allowImportingTsExtensions": true,
    "erasableSyntaxOnly": true,
    "verbatimModuleSyntax": true,
    "skipLibCheck": true
  }
}
```

```bash
npm install -D typescript @types/node
npx tsc --noEmit        # type-check without producing files
```

**Other ways to run TypeScript:** compile with `tsc` to JavaScript in a `dist/` folder (the classic approach, still common for libraries and Docker images), or use `tsx` for fast development runs on older Node versions.

> **Transferable:** Python went the same way with type hints plus a separate checker (mypy, Pyright): the runtime ignores the types, the tooling checks them. Statically typed languages (Go, Java, C#, Rust) check types in the compiler. Either way, types are documentation the machine verifies.

### 34. Debugging and profiling

**Stop using only `console.log`.** Node has a full debugger built in.

**VS Code:** open a **JavaScript Debug Terminal** (Command Palette → "Debug: JavaScript Debug Terminal") and run any command (`npm run dev`, `node app.js`, `node --test`). Breakpoints you set in the editor just work. You can also add a `debugger;` statement in code.

**Chrome DevTools:**

```bash
node --inspect src/server.js       # start and accept a debugger
node --inspect-brk src/server.js   # also pause on the first line (debug startup code)
```

Open `chrome://inspect` in Chrome and click **inspect**. You get breakpoints, step-through, the call stack, variable inspection, a CPU profiler and heap snapshots.

> [!WARNING]
> Never expose the inspector port to a network in production. It gives full code execution on the machine.

**Profiling:**

| Problem | Tool |
| --- | --- |
| "Which function uses the CPU?" | `node --cpu-prof app.js` writes a `.cpuprofile` file; open it in Chrome DevTools (Performance tab) or VS Code |
| "Memory keeps growing" | Heap snapshots via `--inspect` (take two, compare), `--heapsnapshot-near-heap-limit=2`, `--heap-prof` |
| "The server feels laggy" | Event-loop delay with `perf_hooks.monitorEventLoopDelay` ([topic 13](#13-blocking-vs-non-blocking-dont-block-the-event-loop)) |
| "What is Node doing internally?" | `NODE_DEBUG=http,net node app.js` prints core-module debug logs |
| "Where does this warning come from?" | `--trace-warnings`, `--trace-deprecation`, `--trace-uncaught` |
| "How fast is this endpoint under load?" | Load testers such as `autocannon` (Node), k6, wrk |

Simple timing in code:

```javascript
console.time('db query');
await db.query('SELECT ...');
console.timeEnd('db query'); // db query: 12.345ms

const start = performance.now();
// ...work...
console.log(`took ${(performance.now() - start).toFixed(1)} ms`);
```

> **Transferable:** Every serious runtime offers a debugger protocol, a CPU sampler and a heap analyzer: Java (JDWP, JFR, VisualVM), .NET (dotnet-trace, dotnet-dump), Python (`pdb`, `py-spy`, `tracemalloc`), Go (`pprof`, Delve). The workflow is the same everywhere: reproduce, measure, find the hot spot, fix it, measure again.

---

## Part VI — Building a real backend API with Express

### 35. Web frameworks and the middleware model

**Express** is the most widely used Node.js web framework: minimal, stable and backed by a huge ecosystem of middleware. **Express 5** (stable since late 2024 and the npm default since 2025) brought native promise support: errors thrown in `async` route handlers go to your error handler automatically.

**The framework landscape**

| Framework | Style | Choose it when |
| --- | --- | --- |
| **Express** | Minimal, unopinionated, middleware-based | You want the most documentation, examples and middleware. A great learning base |
| **Fastify** | Fast, schema-based validation and serialization, plugin system | Performance matters, you like JSON Schema |
| **NestJS** | Opinionated, TypeScript-first, dependency injection, modules (inspired by Angular and Spring) | Large teams and enterprise codebases that want a fixed structure |
| **Hono** | Tiny, web-standard `Request`/`Response`, runs on Node, Bun, Deno and edge runtimes | Portability across runtimes, edge deployments |
| **Koa** | From the original Express team, `async` middleware, very small core | You want minimalism with a cleaner async model |

**The middleware model** is the key idea behind Express (and Koa, Fastify hooks, ASP.NET Core, Django middleware...). A request passes through a **chain of functions**. Each one can read or modify `req`/`res`, end the response, or call `next()` to pass control on:

```mermaid
%%{init: {"htmlLabels": false, "flowchart": {"htmlLabels": false, "wrappingWidth": 400, "padding": 24}}}%%
flowchart LR
    REQ(["HTTP request"]) --> J["express.json()<br/>parse the body"]
    J --> L["requestLogger<br/>start a timer"]
    L --> R{"router:<br/>does a route match?"}
    R -- "/api/v1/tasks/..." --> V["validate()<br/>check input"]
    V --> H["route handler<br/>call the service"]
    H --> RES(["HTTP response"])
    R -- "no match" --> NF["notFoundHandler<br/>404"]
    NF --> RES
    V -- "throws" --> EH["errorHandler<br/>(4 arguments)"]
    H -- "throws / rejects" --> EH
    EH --> RES
```

```javascript
// The shape of every middleware
function middleware(req, res, next) {
  // do something with req / res
  next(); // pass to the next function, or end with res.send()/res.json()
}

// Error-handling middleware: Express recognizes it by its FOUR parameters
function errorHandler(err, req, res, next) {
  res.status(500).json({ error: 'Something went wrong' });
}
```

**Order matters:** middleware runs in the order it is registered with `app.use()`, so body parsing comes before routes, and the 404 and error handlers come last.

### 36. Case study: designing a Tasks API

We will build a small but realistic **Tasks REST API** with the practices you would expect in a professional codebase:

- Layered architecture (routes, services, repositories) with dependency injection
- Input validation with clear error messages
- Centralized error handling with consistent JSON errors
- Configuration from environment variables, validated at startup
- Request logging, a health-check endpoint and graceful shutdown
- Automated tests with `node:test` + `supertest`
- A Docker image ([Part VII](#part-vii--docker))

**Endpoints**

| Method | Path | Description | Success |
| --- | --- | --- | --- |
| `GET` | `/health` | Liveness check for load balancers and Docker | `200` |
| `GET` | `/api/v1/tasks` | List tasks, optional filter `?done=true\|false` | `200` |
| `GET` | `/api/v1/tasks/:id` | Get one task | `200` |
| `POST` | `/api/v1/tasks` | Create a task `{ "title", "description"? }` | `201` + `Location` header |
| `PATCH` | `/api/v1/tasks/:id` | Partial update `{ "title"?, "description"?, "done"? }` | `200` |
| `DELETE` | `/api/v1/tasks/:id` | Delete a task | `204` |

Errors always look the same:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "details": [{ "field": "body.title", "message": "Too small: expected string to have >=1 characters" }]
  }
}
```

**Architecture**

```mermaid
%%{init: {"htmlLabels": false, "flowchart": {"htmlLabels": false, "wrappingWidth": 400, "padding": 24}}}%%
flowchart TB
    CLIENT(["Client<br/>curl, browser, app,<br/>another service"]) -- "HTTP + JSON" --> MW
    subgraph API["Tasks API"]
        MW["Middleware<br/>JSON parsing, logging"] --> ROUTES["Routes (HTTP layer)<br/>validate input,<br/>call the service"]
        ROUTES --> SERVICE["Service (business layer)<br/>rules such as<br/>'task must exist'"]
        SERVICE --> REPO["Repository (data layer)<br/>the only code that<br/>knows the storage"]
        ROUTES -. "errors" .-> ERR["Error handler<br/>consistent JSON errors"]
        SERVICE -. "errors" .-> ERR
    end
    REPO --> STORE[("In-memory Map today,<br/>PostgreSQL tomorrow")]
```

**Why layers?** Each layer has one reason to change. Switching from in-memory storage to PostgreSQL only touches the repository. Moving from REST to GraphQL only touches the routes. Business rules can be tested without HTTP. This is the "separation of concerns" principle, and you will find it in Spring, ASP.NET, Django and NestJS projects alike.

**Project structure**

```text
tasks-api/
├── src/
│   ├── tasks/                     # one folder per feature ("feature-first")
│   │   ├── tasks.routes.js        # HTTP layer
│   │   ├── tasks.service.js       # business layer
│   │   ├── tasks.repository.js    # data layer
│   │   └── tasks.schemas.js       # input validation schemas
│   ├── middlewares/
│   │   ├── error-handler.js
│   │   ├── request-logger.js
│   │   └── validate.js
│   ├── app.js                     # builds the Express app (no network)
│   ├── config.js                  # environment config, validated
│   ├── errors.js                  # error classes
│   └── server.js                  # starts the HTTP server + graceful shutdown
├── test/
│   └── tasks.test.js
├── .env.example
├── .gitignore
├── Dockerfile
├── .dockerignore
├── compose.yaml
└── package.json
```

### 37. Building the API step by step

**Step 1: create the project**

```bash
mkdir tasks-api && cd tasks-api
npm init -y
npm pkg set type=module engines.node=">=22.12"
npm pkg set private=true --json        # --json stores a real boolean, not the string "true"
npm pkg set scripts.dev="node --watch --env-file-if-exists=.env src/server.js"
npm pkg set scripts.start="node src/server.js" scripts.test="node --test"
npm install express zod
npm install -D supertest
```

`zod` validates data against schemas; `supertest` sends HTTP requests to the app in tests.

Create `.gitignore`:

```text
node_modules/
.env
coverage/
```

And `.env.example` (copy it to `.env` for local development):

```bash
NODE_ENV=development
PORT=3000
```

**Step 2: configuration (`src/config.js`)**: read and validate environment variables in one place, and fail fast:

```javascript
import { z } from 'zod';

// Validate environment variables once, at startup. Fail fast if something is wrong.
const EnvSchema = z.object({
  NODE_ENV: z.enum(['development', 'test', 'production']).default('development'),
  PORT: z.coerce.number().int().min(1).max(65535).default(3000),
});

const parsed = EnvSchema.safeParse(process.env);

if (!parsed.success) {
  console.error('Invalid environment variables:', z.flattenError(parsed.error).fieldErrors);
  process.exit(1);
}

export const config = Object.freeze({
  env: parsed.data.NODE_ENV,
  port: parsed.data.PORT,
});
```

Running with `PORT=abc` now stops immediately with `Invalid environment variables: { PORT: [ 'Invalid input: expected number, received NaN' ] }` instead of failing mysteriously later.

**Step 3: error classes (`src/errors.js`)**: errors that know their HTTP status:

```javascript
// Errors that carry an HTTP meaning. Anything else is treated as a 500.
export class AppError extends Error {
  constructor(message, { status = 500, code = 'INTERNAL_ERROR', details, cause } = {}) {
    super(message, { cause });
    this.name = new.target.name;
    this.status = status;
    this.code = code;
    this.details = details;
  }
}

export class NotFoundError extends AppError {
  constructor(resource, id) {
    super(`${resource} '${id}' not found`, { status: 404, code: 'NOT_FOUND' });
  }
}

export class ValidationError extends AppError {
  constructor(details) {
    super('Request validation failed', { status: 400, code: 'VALIDATION_ERROR', details });
  }
}
```

**Step 4: the repository (`src/tasks/tasks.repository.js`)**: the data layer. It is in-memory for now, but its methods are `async` and shaped like database calls, so callers will not change when a real database arrives ([topic 42](#42-docker-compose-the-api-with-a-database)):

```javascript
import { randomUUID } from 'node:crypto';

// Data-access layer. In-memory for now; the methods are async on purpose,
// so swapping in a real database later does not change any caller.
export function createInMemoryTaskRepository() {
  const tasks = new Map();

  return {
    async findAll({ done } = {}) {
      const all = [...tasks.values()];
      return done === undefined ? all : all.filter((task) => task.done === done);
    },

    async findById(id) {
      return tasks.get(id) ?? null;
    },

    async create({ title, description }) {
      const now = new Date().toISOString();
      const task = { id: randomUUID(), title, description, done: false, createdAt: now, updatedAt: now };
      tasks.set(task.id, task);
      return task;
    },

    async update(id, changes) {
      const current = tasks.get(id);
      if (!current) return null;
      const updated = { ...current, ...changes, updatedAt: new Date().toISOString() };
      tasks.set(id, updated);
      return updated;
    },

    async remove(id) {
      return tasks.delete(id);
    },
  };
}
```

**Step 5: the service (`src/tasks/tasks.service.js`)**: business rules. It receives the repository as a parameter (**dependency injection**) instead of importing it, which makes it easy to test and to swap implementations:

```javascript
import { NotFoundError } from '../errors.js';

// Business layer: rules live here, not in routes and not in the database code.
export function createTaskService(repository) {
  return {
    list(filters) {
      return repository.findAll(filters);
    },

    async get(id) {
      const task = await repository.findById(id);
      if (!task) throw new NotFoundError('Task', id);
      return task;
    },

    create(data) {
      return repository.create(data);
    },

    async update(id, changes) {
      const task = await repository.update(id, changes);
      if (!task) throw new NotFoundError('Task', id);
      return task;
    },

    async remove(id) {
      const removed = await repository.remove(id);
      if (!removed) throw new NotFoundError('Task', id);
    },
  };
}
```

**Step 6: validation schemas (`src/tasks/tasks.schemas.js`)**: *never trust client input*. Schemas describe valid input once; zod checks it, trims strings, converts types and **strips unknown fields** (so a client cannot sneak in `"isAdmin": true`):

```javascript
import { z } from 'zod';

const title = z.string().trim().min(1).max(200);
const description = z.string().trim().max(2000);

export const createTaskSchema = z.object({
  title,
  description: description.optional(),
});

export const updateTaskSchema = z
  .object({
    title: title.optional(),
    description: description.optional(),
    done: z.boolean().optional(),
  })
  .refine((body) => Object.keys(body).length > 0, { message: 'Provide at least one field' });

export const listQuerySchema = z.object({
  done: z
    .enum(['true', 'false'])
    .transform((value) => value === 'true')
    .optional(),
});

export const idParamsSchema = z.object({ id: z.uuid() });
```

**Step 7: middlewares**

`src/middlewares/validate.js`, a reusable validation middleware:

```javascript
import { ValidationError } from '../errors.js';

// Usage: validate({ body: schema, query: schema, params: schema })
// Parsed (trimmed, coerced, unknown-keys-stripped) values end up in req.validated.
export const validate = (schemas) => (req, res, next) => {
  req.validated = {};

  for (const [part, schema] of Object.entries(schemas)) {
    const result = schema.safeParse(req[part]);

    if (!result.success) {
      throw new ValidationError(
        result.error.issues.map((issue) => ({
          field: [part, ...issue.path].join('.'),
          message: issue.message,
        })),
      );
    }

    req.validated[part] = result.data;
  }

  next();
};
```

> [!NOTE]
> Why `req.validated` instead of overwriting `req.query`? In Express 5, `req.query` is a read-only getter, so assigning to it throws. Storing parsed values separately also makes it obvious which data has been validated.

`src/middlewares/request-logger.js`:

```javascript
// Logs one line per request once the response has been sent.
export function requestLogger(req, res, next) {
  const start = performance.now();

  res.on('finish', () => {
    const ms = (performance.now() - start).toFixed(1);
    console.log(`${req.method} ${req.originalUrl} -> ${res.statusCode} (${ms} ms)`);
  });

  next();
}
```

`src/middlewares/error-handler.js`: one place that turns any error into a consistent JSON response, and never leaks internals for 500s:

```javascript
import { AppError } from '../errors.js';

export function notFoundHandler(req, res) {
  res.status(404).json({
    error: { code: 'ROUTE_NOT_FOUND', message: `Cannot ${req.method} ${req.path}` },
  });
}

// Express recognizes an error handler by its FOUR parameters.
export function errorHandler(err, req, res, next) {
  if (res.headersSent) return next(err); // too late to send JSON; let Express close the connection

  // AppError -> its own status; body-parser errors (bad JSON) carry err.status; anything else -> 500
  const status = err instanceof AppError ? err.status : (err.status ?? 500);
  const isServerError = status >= 500;

  if (isServerError) console.error(err); // log the details, never send them to the client

  res.status(status).json({
    error: {
      code: err instanceof AppError ? err.code : isServerError ? 'INTERNAL_ERROR' : 'BAD_REQUEST',
      message: isServerError ? 'Internal server error' : err.message,
      ...(err.details && { details: err.details }),
    },
  });
}
```

**Step 8: routes (`src/tasks/tasks.routes.js`)**: the HTTP layer. Thanks to Express 5, `async` handlers need no `try/catch`: a thrown `NotFoundError` reaches `errorHandler` automatically:

```javascript
import { Router } from 'express';
import { validate } from '../middlewares/validate.js';
import { createTaskSchema, idParamsSchema, listQuerySchema, updateTaskSchema } from './tasks.schemas.js';

// HTTP layer: translate HTTP <-> service calls. No business rules here.
// Express 5 forwards rejected promises to the error handler, so no try/catch is needed.
export function createTasksRouter(service) {
  const router = Router();

  router.get('/', validate({ query: listQuerySchema }), async (req, res) => {
    res.json(await service.list(req.validated.query));
  });

  router.get('/:id', validate({ params: idParamsSchema }), async (req, res) => {
    res.json(await service.get(req.validated.params.id));
  });

  router.post('/', validate({ body: createTaskSchema }), async (req, res) => {
    const task = await service.create(req.validated.body);
    res.status(201).location(`${req.baseUrl}/${task.id}`).json(task);
  });

  router.patch('/:id', validate({ params: idParamsSchema, body: updateTaskSchema }), async (req, res) => {
    res.json(await service.update(req.validated.params.id, req.validated.body));
  });

  router.delete('/:id', validate({ params: idParamsSchema }), async (req, res) => {
    await service.remove(req.validated.params.id);
    res.status(204).end();
  });

  return router;
}
```

**Step 9: the app (`src/app.js`)**: the **composition root**, where all the pieces are wired together. It builds the app but does **not** start listening, so tests can use it without opening a port:

```javascript
import express from 'express';
import { errorHandler, notFoundHandler } from './middlewares/error-handler.js';
import { requestLogger } from './middlewares/request-logger.js';
import { createInMemoryTaskRepository } from './tasks/tasks.repository.js';
import { createTasksRouter } from './tasks/tasks.routes.js';
import { createTaskService } from './tasks/tasks.service.js';

// Builds the app WITHOUT starting a server, so tests can use it directly.
// Dependencies are injected: pass a different repository to use a real database.
export function createApp({ taskRepository = createInMemoryTaskRepository(), logRequests = true } = {}) {
  const app = express();

  app.disable('x-powered-by'); // don't advertise the framework
  app.use(express.json({ limit: '100kb' })); // parse JSON bodies, reject huge payloads
  if (logRequests) app.use(requestLogger);

  app.get('/health', (req, res) => {
    res.json({ status: 'ok', uptime: Math.round(process.uptime()) });
  });

  const taskService = createTaskService(taskRepository);
  app.use('/api/v1/tasks', createTasksRouter(taskService));

  // Order matters: these two must be registered LAST.
  app.use(notFoundHandler);
  app.use(errorHandler);

  return app;
}
```

**Step 10: the server (`src/server.js`)**: start listening and **shut down gracefully**. When Docker or Kubernetes stops a container, it sends `SIGTERM`; the server stops accepting new connections and lets in-flight requests finish:

```javascript
import { createApp } from './app.js';
import { config } from './config.js';

const app = createApp();

const server = app.listen(config.port, (error) => {
  if (error) throw error; // e.g. EADDRINUSE: the port is already taken
  console.log(`Tasks API on http://localhost:${config.port} (${config.env})`);
});

// Graceful shutdown: stop accepting new connections, let in-flight requests finish.
// Docker/Kubernetes send SIGTERM; Ctrl+C sends SIGINT.
function shutdown(signal) {
  console.log(`${signal} received, closing server...`);
  server.close(() => process.exit(0));
  setTimeout(() => process.exit(1), 10_000).unref(); // hard limit if something hangs
}

process.on('SIGTERM', shutdown);
process.on('SIGINT', shutdown);
```

**Step 11: run it and try it**

```bash
cp .env.example .env
npm run dev
```

```bash
# Create
curl -i -X POST http://localhost:3000/api/v1/tasks \
  -H "Content-Type: application/json" \
  -d '{"title": "Learn the event loop", "description": "phases and microtasks"}'

# List (all, or filtered)
curl http://localhost:3000/api/v1/tasks
curl "http://localhost:3000/api/v1/tasks?done=false"

# Update (replace <id> with the id returned above)
curl -X PATCH http://localhost:3000/api/v1/tasks/<id> \
  -H "Content-Type: application/json" -d '{"done": true}'

# Delete
curl -i -X DELETE http://localhost:3000/api/v1/tasks/<id>

# Invalid input -> 400 with details
curl -X POST http://localhost:3000/api/v1/tasks -H "Content-Type: application/json" -d '{"title": ""}'
```

On **Windows PowerShell**, use `Invoke-RestMethod` (no quoting headaches):

```powershell
Invoke-RestMethod -Method Post -Uri http://localhost:3000/api/v1/tasks `
  -ContentType 'application/json' -Body '{"title": "Learn the event loop"}'
Invoke-RestMethod http://localhost:3000/api/v1/tasks
```

Or, with the VS Code **REST Client** extension, save this as `requests.http` and click "Send Request" above each block:

```http
@base = http://localhost:3000/api/v1

### Create a task
POST {{base}}/tasks
Content-Type: application/json

{ "title": "Learn the event loop" }

### List tasks
GET {{base}}/tasks

### Only finished tasks
GET {{base}}/tasks?done=true
```

The server logs one line per request:

```text
Tasks API on http://localhost:3000 (development)
POST /api/v1/tasks -> 201 (0.9 ms)
GET /api/v1/tasks?done=true -> 200 (0.5 ms)
POST /api/v1/tasks -> 400 (0.7 ms)
```

**The full journey of `POST /api/v1/tasks`:**

```mermaid
sequenceDiagram
    autonumber
    participant C as Client
    participant E as Express middleware
    participant V as validate()
    participant H as Route handler
    participant S as TaskService
    participant R as Repository
    C->>E: POST /api/v1/tasks with a JSON body
    E->>E: express.json() parses the body, logger starts a timer
    E->>V: router matches POST /
    V->>V: createTaskSchema.safeParse(req.body)
    alt invalid input
        V-->>C: 400 VALIDATION_ERROR via errorHandler
    else valid
        V->>H: next() with req.validated.body
        H->>S: create(data)
        S->>R: create(data)
        R-->>S: new task with id and timestamps
        S-->>H: task
        H-->>C: 201 Created, Location header, task JSON
    end
```

### 38. Testing the API

Because `createApp()` does not open a port, `supertest` can send requests straight to it. Each test gets a fresh app, so tests never share data.

`test/tasks.test.js`:

```javascript
import assert from 'node:assert/strict';
import { randomUUID } from 'node:crypto';
import { beforeEach, describe, it } from 'node:test';
import request from 'supertest';
import { createApp } from '../src/app.js';

describe('Tasks API', () => {
  let app;

  beforeEach(() => {
    app = createApp({ logRequests: false }); // fresh in-memory data for every test
  });

  it('creates a task and fetches it by id', async () => {
    const created = await request(app)
      .post('/api/v1/tasks')
      .send({ title: '  Learn the event loop  ' })
      .expect(201);

    assert.equal(created.body.title, 'Learn the event loop'); // trimmed by the schema
    assert.equal(created.body.done, false);
    assert.equal(created.headers.location, `/api/v1/tasks/${created.body.id}`);

    const fetched = await request(app).get(`/api/v1/tasks/${created.body.id}`).expect(200);
    assert.deepEqual(fetched.body, created.body);
  });

  it('updates, filters and deletes tasks', async () => {
    const { body: task } = await request(app).post('/api/v1/tasks').send({ title: 'Ship it' });
    await request(app).post('/api/v1/tasks').send({ title: 'Write tests' });

    await request(app).patch(`/api/v1/tasks/${task.id}`).send({ done: true }).expect(200);

    const done = await request(app).get('/api/v1/tasks?done=true').expect(200);
    assert.deepEqual(done.body.map((t) => t.title), ['Ship it']);

    await request(app).delete(`/api/v1/tasks/${task.id}`).expect(204);
    await request(app).get(`/api/v1/tasks/${task.id}`).expect(404);
  });

  it('rejects invalid input with 400 and field details', async () => {
    const res = await request(app).post('/api/v1/tasks').send({ title: '' }).expect(400);
    assert.equal(res.body.error.code, 'VALIDATION_ERROR');
    assert.equal(res.body.error.details[0].field, 'body.title');
  });

  it('rejects malformed JSON with 400', async () => {
    await request(app)
      .post('/api/v1/tasks')
      .set('Content-Type', 'application/json')
      .send('{"title": ')
      .expect(400);
  });

  it('returns 404 for unknown tasks and unknown routes', async () => {
    await request(app).get(`/api/v1/tasks/${randomUUID()}`).expect(404);
    await request(app).get('/nope').expect(404);
  });
});
```

```text
$ npm test

▶ Tasks API
  ✔ creates a task and fetches it by id (33.3ms)
  ✔ updates, filters and deletes tasks (12.4ms)
  ✔ rejects invalid input with 400 and field details (3.0ms)
  ✔ rejects malformed JSON with 400 (2.6ms)
  ✔ returns 404 for unknown tasks and unknown routes (3.7ms)
✔ Tasks API (55.8ms)
ℹ tests 5
ℹ pass 5
ℹ fail 0
```

These are **integration tests**: they exercise routing, validation, the service, the repository and error handling together. Because the service takes its repository as a parameter, you can also unit-test business rules with a fake repository, no HTTP involved.

### 39. REST API design and a production checklist

**REST conventions that carry over to any stack**

| Rule | Example |
| --- | --- |
| Resources are **nouns**, plural | `/tasks`, `/tasks/42`, `/users/7/tasks` |
| HTTP **methods** are the verbs | `GET` read, `POST` create, `PUT` replace, `PATCH` partial update, `DELETE` remove |
| `GET`, `PUT` and `DELETE` are **idempotent** (repeating them has the same effect) | Clients and proxies can safely retry them |
| Use **status codes** precisely | see below |
| **Version** the API | `/api/v1/...` so you can ship `/api/v2` without breaking clients |
| **Paginate** collections | `?limit=20&cursor=...` (cursor-based scales better than `?page=`) |
| Consistent **error format** | `{ "error": { "code", "message", "details" } }` |
| Document it | OpenAPI (Swagger) spec, generated or hand-written |

**Status codes you will use most**

| Code | Meaning |
| --- | --- |
| `200 OK` | Success with a body |
| `201 Created` | Resource created (plus a `Location` header) |
| `204 No Content` | Success, nothing to return (e.g. delete) |
| `400 Bad Request` | Malformed or invalid input |
| `401 Unauthorized` | Not authenticated (no or invalid credentials) |
| `403 Forbidden` | Authenticated, but not allowed |
| `404 Not Found` | Resource does not exist |
| `409 Conflict` | State conflict (duplicate email, version mismatch) |
| `422 Unprocessable Content` | Valid syntax but fails business rules (some APIs use 400 for this too) |
| `429 Too Many Requests` | Rate limit hit |
| `500 Internal Server Error` | A bug or unexpected failure (never leak details) |
| `503 Service Unavailable` | Temporarily down or overloaded |

**From tutorial to production: what to add next**

| Concern | Typical solution in the Node ecosystem |
| --- | --- |
| Persistence | PostgreSQL with `pg`, or an ORM/query builder (Prisma, Drizzle, Kysely, Knex); migrations ([topic 42](#42-docker-compose-the-api-with-a-database)) |
| Security headers | `helmet` middleware |
| CORS (browser clients on another origin) | `cors` middleware with an explicit origin allowlist |
| Rate limiting | `express-rate-limit` (backed by Redis when running several instances) |
| Authentication | JWT or session cookies; OAuth/OIDC via a provider (Auth0, Keycloak, Cognito...); hash passwords with `argon2`, `bcrypt` or the built-in `crypto.scrypt` |
| Structured logging | `pino` / `pino-http` (JSON logs with request ids, so log tools can search them) |
| Observability | Metrics (Prometheus), tracing (OpenTelemetry), error tracking (Sentry) |
| API docs | OpenAPI spec + Swagger UI or Scalar |
| Readiness checks | A `/ready` endpoint that checks the database, separate from `/health` |
| Background jobs | A queue such as BullMQ (Redis), or a managed queue service |
| Types | TypeScript ([topic 33](#33-typescript-with-nodejs)) |

Adding security headers and CORS, for example, takes two lines in `createApp`:

```javascript
import cors from 'cors';
import helmet from 'helmet';

app.use(helmet());
app.use(cors({ origin: ['https://my-frontend.example.com'] }));
```

> **Transferable:** This Tasks API maps one-to-one onto other stacks. Routes become controllers (Spring `@RestController`, ASP.NET controllers, FastAPI routers, Gin handlers), zod schemas become Pydantic models, Bean Validation or FluentValidation, and the error middleware becomes `@ControllerAdvice`, exception filters or FastAPI exception handlers. The repository and service layers, dependency injection and REST rules are the same everywhere. Build this once in Node and you can rebuild it in any language.

---

## Part VII — Docker

### 40. What Docker is: containers, images and layers

**The problem Docker solves:** "It works on my machine." Your laptop has Node 26 on Windows; the server has Node 22 on Linux with different system libraries; a colleague is on macOS. Docker packages your app **together with everything it needs to run** (the exact Node version, OS libraries, `node_modules`, your code) into one artifact that runs the same way everywhere.

**A container is an isolated process, not a virtual machine.** It runs directly on the host's kernel, but the kernel gives it its own view of the system:

- **Namespaces** (isolation): its own process tree (it sees itself as PID 1), network stack and IP, filesystem, hostname and users.
- **cgroups** (limits): caps on CPU, memory and I/O.
- **A layered filesystem** (OverlayFS): the image's read-only layers stacked together, plus a thin writable layer on top.

```mermaid
%%{init: {"htmlLabels": false, "flowchart": {"htmlLabels": false, "wrappingWidth": 400, "padding": 24}}}%%
flowchart TB
    subgraph VMS["Virtual machines"]
        direction TB
        VA["App A"] --> GA["Guest OS A<br/>(full kernel, GBs)"]
        VB["App B"] --> GB["Guest OS B<br/>(full kernel, GBs)"]
        GA --> HYP["Hypervisor"]
        GB --> HYP
        HYP --> HW1["Host hardware"]
    end
    subgraph CTS["Containers"]
        direction TB
        CA["App A + its libraries"] --> RT["Container runtime<br/>(Docker Engine)"]
        CB["App B + its libraries"] --> RT
        RT --> KER["Host OS kernel (shared)"]
        KER --> HW2["Host hardware"]
    end
```

| | Virtual machine | Container |
| --- | --- | --- |
| Isolates | A whole computer (own kernel) | A process (shared kernel) |
| Size | Gigabytes | Megabytes to a few hundred MB |
| Startup | Tens of seconds to minutes | Milliseconds to seconds |
| Density | A handful per host | Dozens to hundreds per host |
| Isolation strength | Stronger | Good, but the kernel is shared |

**Vocabulary**

| Term | Meaning |
| --- | --- |
| **Dockerfile** | A text recipe describing how to build an image |
| **Image** | A read-only, versioned package (filesystem layers + metadata such as the start command). Think "class" |
| **Layer** | One cached filesystem diff, produced by one Dockerfile instruction. Layers are shared between images |
| **Container** | A running (or stopped) instance of an image, plus a writable layer. Think "object" |
| **Tag** | A human-readable version label: `node:26-alpine`, `tasks-api:1.0.0` |
| **Registry** | A server that stores images: Docker Hub, GitHub Container Registry (GHCR), AWS ECR, Google Artifact Registry |
| **Volume** | Persistent storage that outlives containers (databases need this) |
| **Port mapping** | `-p 8080:3000` forwards host port 8080 to container port 3000 |
| **Docker Compose** | A YAML file that defines and runs several containers together ([topic 42](#42-docker-compose-the-api-with-a-database)) |

**The workflow:**

```mermaid
%%{init: {"htmlLabels": false, "flowchart": {"htmlLabels": false, "wrappingWidth": 400, "padding": 24}}}%%
flowchart LR
    DF["Dockerfile<br/>(recipe)"] -- "docker build" --> IMG["Image<br/>(read-only layers)"]
    IMG -- "docker run" --> CON["Container<br/>(running process)"]
    IMG -- "docker push" --> REG[("Registry<br/>Docker Hub, GHCR, ECR")]
    REG -- "docker pull" --> PROD["Same image running on<br/>CI, staging, production"]
```

**The engines behind Docker.** Like Node.js, Docker is a stack of components:

```mermaid
%%{init: {"htmlLabels": false, "flowchart": {"htmlLabels": false, "wrappingWidth": 400, "padding": 24}}}%%
flowchart LR
    CLI["docker CLI<br/>(what you type)"] -- "REST API" --> D["dockerd<br/>Docker Engine daemon"]
    D --> CTR["containerd<br/>manages image and<br/>container lifecycles"]
    CTR --> RUNC["runc<br/>creates the container"]
    RUNC --> K["Linux kernel<br/>namespaces + cgroups"]
```

Images and runtimes follow the **OCI** (Open Container Initiative) standards, so an image built with Docker also runs on Podman, containerd or Kubernetes. On **Windows and macOS**, Docker Desktop runs a small Linux virtual machine (WSL 2 on Windows) because Linux containers need a Linux kernel.

**Essential commands**

| Command | What it does |
| --- | --- |
| `docker build -t name:tag .` | Build an image from the Dockerfile in the current folder |
| `docker images` | List local images |
| `docker run --rm -p 3000:3000 name:tag` | Run a container (`--rm` deletes it on exit, `-d` runs it in the background) |
| `docker ps` / `docker ps -a` | Running containers / all containers |
| `docker logs -f <container>` | Follow a container's output |
| `docker exec -it <container> sh` | Open a shell inside a running container |
| `docker stop <container>` | Graceful stop (SIGTERM, then SIGKILL after 10 seconds) |
| `docker rm <container>` / `docker rmi <image>` | Delete a container / an image |
| `docker compose up --build` / `docker compose down` | Start / stop a multi-container setup |
| `docker system prune` | Clean up unused containers, networks and dangling images |

> **Transferable:** Docker is language-agnostic: the same concepts and commands package a Python, Go, Java or .NET app. Only the base image and the build steps change. Containers are also the unit Kubernetes, AWS ECS and Google Cloud Run run, so this knowledge carries straight into cloud deployment.

### 41. Dockerizing the Node.js API

**"Dockerizing" a Node app** means writing a `Dockerfile` that turns the project into an image containing a specific Node version, production dependencies only, your source code, a non-root user and a start command, so the exact same artifact runs on your laptop, in CI and in production.

**Step 1: `.dockerignore`** keeps the build context small and secrets out of the image. Without it, `COPY` would send your local `node_modules` (built for your OS) and your `.env` into the image:

```text
node_modules
npm-debug.log*
.git
.gitignore
.env
.env.*
!.env.example
Dockerfile
.dockerignore
compose.yaml
coverage
test
.vscode
*.md
```

**Step 2: `Dockerfile`** (multi-stage):

```dockerfile
# syntax=docker/dockerfile:1

# Change the Node version in ONE place (keep it in sync with .nvmrc and "engines")
ARG NODE_VERSION=26

# ---------- Stage 1: install production dependencies ----------
FROM node:${NODE_VERSION}-alpine AS deps
WORKDIR /app
# Copy only the manifests first: this layer (and the slow npm ci) stays cached
# until package.json or package-lock.json changes.
COPY package.json package-lock.json ./
RUN npm ci --omit=dev

# ---------- Stage 2: the runtime image ----------
FROM node:${NODE_VERSION}-alpine AS runtime
ENV NODE_ENV=production
WORKDIR /app

COPY --from=deps /app/node_modules ./node_modules
COPY package.json ./
COPY src ./src

# Official Node images include an unprivileged "node" user: don't run as root
USER node

EXPOSE 3000

# Docker marks the container healthy/unhealthy based on this command.
# (busybox wget ships with alpine; on slim/distroless images use a `node -e "fetch(...)"` one-liner)
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
  CMD wget -qO- http://127.0.0.1:3000/health || exit 1

# Exec form (JSON array): node runs as PID 1 and receives SIGTERM directly (see topic 43)
CMD ["node", "src/server.js"]
```

What each part does:

- **`ARG NODE_VERSION`** keeps the Node version in one place, so upgrading is a one-line change.
- **`FROM node:26-alpine`** starts from the official Node image on Alpine Linux (small).
- **Multi-stage build:** the `deps` stage installs packages. The final `runtime` stage copies only the resulting `node_modules`, so npm's cache and any build tools never reach the final image.
- **`COPY package*.json` before `COPY src`** is the most important caching trick (see below).
- **`npm ci --omit=dev`** gives an exact, reproducible install without test tools and linters.
- **`USER node`** means a compromised app does not have root inside the container.
- **`HEALTHCHECK`** lets Docker (and orchestrators) know whether the app actually responds, not just whether the process exists.

**Why the order of instructions matters (layer caching):**

```mermaid
%%{init: {"htmlLabels": false, "flowchart": {"htmlLabels": false, "wrappingWidth": 400, "padding": 24}}}%%
flowchart BT
    L1["FROM node:26-alpine<br/>base OS + Node<br/>(downloaded once)"] --> L2["WORKDIR /app"]
    L2 --> L3["COPY package.json<br/>package-lock.json<br/>changes rarely"]
    L3 --> L4["RUN npm ci --omit=dev<br/>slow, but CACHED while<br/>manifests are unchanged"]
    L4 --> L5["COPY src<br/>changes on every<br/>commit, but cheap"]
```

Docker reuses a cached layer until one of its inputs changes; after that, **every later layer is rebuilt**. If you copied all the source code before running `npm ci`, every one-line code change would reinstall every dependency.

**Step 3: build and run**

```bash
docker build -t tasks-api:1.0.0 .

docker run --rm -p 3000:3000 --name tasks-api tasks-api:1.0.0
# Tasks API on http://localhost:3000 (production)

curl http://localhost:3000/health          # from another terminal
docker ps                                  # STATUS shows "(healthy)" after the first check
docker stop tasks-api                      # SIGTERM -> "SIGTERM received, closing server..."
```

Map a different host port or override configuration without rebuilding:

```bash
docker run -d -p 8080:3000 -e PORT=3000 --name tasks-api tasks-api:1.0.0
```

**With a build step (TypeScript, bundlers)** add a stage that has dev dependencies, compile there, and copy only the output:

```dockerfile
FROM node:${NODE_VERSION}-alpine AS build
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci                      # includes devDependencies such as typescript
COPY . .
RUN npm run build               # e.g. tsc -> dist/

# ...then in the runtime stage:
# COPY --from=build /app/dist ./dist
# CMD ["node", "dist/server.js"]
```

### 42. Docker Compose: the API with a database

Real apps need more than one container: the API, a database, maybe Redis. **Docker Compose** describes them all in one file and starts them with one command. Containers in the same Compose project share a network and reach each other **by service name** (the API connects to the host `db`).

```mermaid
%%{init: {"htmlLabels": false, "flowchart": {"htmlLabels": false, "wrappingWidth": 400, "padding": 24}}}%%
flowchart LR
    YOU(["You<br/>localhost:3000"]) --> API["api container<br/>node src/server.js"]
    API -- "postgres://db:5432" --> DB[("db container<br/>PostgreSQL")]
    DB --- VOL[("named volume db-data<br/>survives restarts")]
    INIT["db/init.sql"] -. "runs on first start" .-> DB
```

**`compose.yaml`:**

```yaml
services:
  api:
    build: .
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: production
      PORT: 3000
      DATABASE_URL: postgres://tasks:tasks@db:5432/tasks
    depends_on:
      db:
        condition: service_healthy # wait until Postgres accepts connections
    restart: unless-stopped

  db:
    image: postgres:17-alpine
    environment:
      POSTGRES_USER: tasks
      POSTGRES_PASSWORD: tasks # local development only: use secrets for real deployments
      POSTGRES_DB: tasks
    volumes:
      - db-data:/var/lib/postgresql/data
      - ./db/init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U tasks -d tasks"]
      interval: 5s
      timeout: 3s
      retries: 10
    ports:
      - "5432:5432" # optional: lets database tools on your machine connect

volumes:
  db-data:
```

**`db/init.sql`** (the Postgres image runs it on the first start, when the volume is empty):

```sql
CREATE TABLE IF NOT EXISTS tasks (
  id          uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  title       text NOT NULL,
  description text,
  done        boolean NOT NULL DEFAULT false,
  created_at  timestamptz NOT NULL DEFAULT now(),
  updated_at  timestamptz NOT NULL DEFAULT now()
);
```

**A PostgreSQL repository.** Thanks to the layered design, this is the **only** new data-access code. It has the same methods as the in-memory one, so the service and routes don't change. Install the driver with `npm install pg`, then create `src/tasks/tasks.repository.pg.js`:

```javascript
const COLUMNS = `id, title, description, done, created_at AS "createdAt", updated_at AS "updatedAt"`;

// Same interface as createInMemoryTaskRepository(), backed by PostgreSQL.
// Always use parameters ($1, $2...) for values: never concatenate input into SQL.
export function createPostgresTaskRepository(pool) {
  return {
    async findAll({ done } = {}) {
      const { rows } = await pool.query(
        `SELECT ${COLUMNS} FROM tasks WHERE ($1::boolean IS NULL OR done = $1) ORDER BY created_at`,
        [done ?? null],
      );
      return rows;
    },

    async findById(id) {
      const { rows } = await pool.query(`SELECT ${COLUMNS} FROM tasks WHERE id = $1`, [id]);
      return rows[0] ?? null;
    },

    async create({ title, description }) {
      const { rows } = await pool.query(
        `INSERT INTO tasks (title, description) VALUES ($1, $2) RETURNING ${COLUMNS}`,
        [title, description ?? null],
      );
      return rows[0];
    },

    async update(id, changes) {
      const { rows } = await pool.query(
        `UPDATE tasks
            SET title = COALESCE($2, title),
                description = COALESCE($3, description),
                done = COALESCE($4, done),
                updated_at = now()
          WHERE id = $1
      RETURNING ${COLUMNS}`,
        [id, changes.title ?? null, changes.description ?? null, changes.done ?? null],
      );
      return rows[0] ?? null;
    },

    async remove(id) {
      const { rowCount } = await pool.query('DELETE FROM tasks WHERE id = $1', [id]);
      return rowCount > 0;
    },
  };
}
```

Wire it up: add `DATABASE_URL: z.url().optional()` to the schema in `src/config.js` (and `databaseUrl: parsed.data.DATABASE_URL` to the exported config), then choose the repository in `src/server.js`:

```javascript
import pg from 'pg';
import { createApp } from './app.js';
import { config } from './config.js';
import { createInMemoryTaskRepository } from './tasks/tasks.repository.js';
import { createPostgresTaskRepository } from './tasks/tasks.repository.pg.js';

const pool = config.databaseUrl ? new pg.Pool({ connectionString: config.databaseUrl }) : null;
const taskRepository = pool ? createPostgresTaskRepository(pool) : createInMemoryTaskRepository();

const app = createApp({ taskRepository });
// ...listen as before, and in shutdown(): server.close(async () => { await pool?.end(); process.exit(0); });
```

A connection **pool** keeps a few database connections open and lends them to requests, which is much faster than connecting on every query. Create one per process and close it on shutdown.

**Run everything:**

```bash
docker compose up --build        # build the API image and start both containers
docker compose logs -f api       # follow the API logs
docker compose ps                # status and health
docker compose down              # stop and remove containers (data stays in the volume)
docker compose down -v           # ...and delete the volume (fresh database next time)
```

**Development tips**

- A popular setup is to run **only the database in Docker** (`docker compose up db`) and run Node locally with `npm run dev`, which gives fast reloads and easy debugging. Point `DATABASE_URL` in `.env` to `localhost:5432`.
- `docker compose watch` (with a `develop.watch` section in `compose.yaml`) can sync code changes into running containers.
- For real projects, manage schema changes with **migrations** (e.g. `node-pg-migrate`, Prisma Migrate, Drizzle Kit, Knex) instead of a single init script.

### 43. Container best practices for Node.js

**1. Choose the base image deliberately**

| Image | Based on | Pros | Cons |
| --- | --- | --- | --- |
| `node:26` | Debian (full) | Has compilers and tools, so native modules build easily | Large (around 1 GB) |
| `node:26-slim` | Debian (minimal) | glibc, so the best compatibility with native addons; small | No build tools |
| `node:26-alpine` | Alpine Linux | Very small | Uses musl libc, so some native addons need extra work or behave differently |
| Distroless Node images | Debian, nothing else | Smallest attack surface: no shell, no package manager | Harder to debug (no shell to `exec` into) |

Pin at least the major version (`node:26-alpine`). For fully reproducible builds, pin the exact version or the image digest, and rebuild regularly to pick up security patches. Check sizes with `docker images`.

**2. Handle signals: PID 1 and graceful shutdown**

```mermaid
sequenceDiagram
    participant D as docker stop / Kubernetes
    participant N as node (PID 1)
    D->>N: SIGTERM
    N->>N: server.close(): stop accepting, finish in-flight requests
    N->>N: close DB pool
    N-->>D: exit code 0
    Note over D,N: if the process is still alive after the grace period (10 s by default)
    D->>N: SIGKILL (cannot be caught, requests are cut off)
```

- Use the **exec form** `CMD ["node", "src/server.js"]`. The shell form (`CMD node src/server.js`) starts a shell as PID 1, which may not forward signals to Node.
- Prefer `node` over `npm start` as the container command. npm adds another process layer between the signal and your app.
- A process running as PID 1 gets **no default signal handlers** from the kernel, so Node must handle `SIGTERM` itself (our `server.js` does). Alternatively, run with `docker run --init` (or `init: true` in Compose), which adds a tiny init process (tini) that forwards signals and reaps zombie processes.

**3. Security**

- Run as a **non-root** user (`USER node`).
- **Never bake secrets into the image.** Don't `COPY .env`, and don't put secrets in `ARG`/`ENV`: they are visible in `docker history`. Inject them at runtime (environment variables, Docker/Kubernetes secrets).
- Install production dependencies only (`npm ci --omit=dev`) and keep `.dockerignore` strict.
- Scan images for known vulnerabilities: `docker scout cves tasks-api:1.0.0` or Trivy.

**4. Operations**

- **One process per container.** Scale by running more containers (replicas), not with `cluster` inside a container.
- **Log to stdout/stderr**, not to files. Docker and every log platform collect stdout.
- **Set memory limits** and tell V8 about them: for example, with a 512 MB container limit, set `NODE_OPTIONS=--max-old-space-size=384` (about 75%, leaving room for buffers and native memory).
- Provide **health checks**: `HEALTHCHECK` in Docker, liveness and readiness probes in Kubernetes.
- Set `NODE_ENV=production`.
- Speed up builds with BuildKit cache mounts: `RUN --mount=type=cache,target=/root/.npm npm ci --omit=dev`.

> **Transferable:** These rules apply to containerizing any language: minimal base image, multi-stage builds (compile in a "builder" image, ship only the output: this is how Go and Rust services end up with images of just a few MB), non-root user, exec-form entrypoint, signal handling, config through the environment, logs to stdout, one process per container.

---

## Part VIII — Production and beyond

### 44. Security essentials

**Keep Node itself updated.** The Node.js project ships security releases regularly. Running an EOL line means known vulnerabilities stay unpatched forever.

**Your dependencies are your attack surface (supply chain).** In 2025, several extremely popular npm packages (including `chalk` and `debug`) were briefly published with malicious code after maintainer accounts were phished, and self-spreading malware such as the "Shai-Hulud" worm hit hundreds of packages. Defenses:

- Commit lockfiles and install with `npm ci`. Review lockfile changes in pull requests.
- Run `npm audit` in CI, and automate updates with Dependabot or Renovate (reviewing what changes).
- Think twice before adding a dependency: is it maintained, widely used, and really needed?
- Be careful with install scripts. `npm install --ignore-scripts` disables them, and pnpm (v10+) does not run dependencies' lifecycle scripts unless you allow them.
- If you publish packages: enable 2FA and use trusted publishing with provenance.

**Application security checklist**

| Threat | Defense |
| --- | --- |
| Invalid or malicious input | Validate everything at the boundary (schemas, as in the API), limit body sizes |
| SQL / NoSQL injection | Parameterized queries (`$1`), or an ORM. Never build queries by string concatenation |
| Command injection | `execFile`/`spawn` with argument arrays, never `exec` with user input; never `eval` or `new Function` on input |
| Path traversal (`../../etc/passwd`) | Resolve the path, then check it is still inside the allowed folder |
| Prototype pollution | Don't deep-merge untrusted JSON into objects; use `Map` or `Object.create(null)` for dictionaries |
| ReDoS | Avoid nested quantifiers in regexes, limit input length |
| Leaked secrets | Environment variables or a secret manager, `.env` in `.gitignore`, rotate keys |
| Weak authentication | Hash passwords with argon2, bcrypt or `crypto.scrypt`; compare secrets with `crypto.timingSafeEqual`; short-lived tokens |
| Broken authorization (IDOR) | Check that the current user may access *this specific* resource, on every request |
| Missing HTTP protections | `helmet`, a strict CORS allowlist, rate limiting, HTTPS everywhere, cookies with `HttpOnly`, `Secure` and `SameSite` |
| Leaking internals | Generic 500 messages (as in our error handler), `app.disable('x-powered-by')` |

**Node's permission model** `(Node ≥ 22.13 / 23.5)` restricts what a process may access, like a sandbox:

```javascript
// sandbox.js
import { readFileSync, writeFileSync } from 'node:fs';

readFileSync('package.json');
console.log('read ok');
writeFileSync('hacked.txt', 'oops'); // not allowed: throws ERR_ACCESS_DENIED
```

```text
$ node --permission --allow-fs-read=./ sandbox.js
read ok
Error: Access to this API has been restricted. Use --allow-fs-write to manage permissions.
  code: 'ERR_ACCESS_DENIED',
  permission: 'FileSystemWrite',
```

With `--permission`, file writes, child processes, worker threads and native addons are denied unless explicitly allowed (`--allow-fs-write`, `--allow-child-process`, `--allow-worker`, `--allow-addons`). It is defense in depth against a compromised dependency, not a replacement for the practices above.

Further reading: the official [Node.js security best practices](https://nodejs.org/en/learn/getting-started/security-best-practices) and the [OWASP Top 10](https://owasp.org/www-project-top-ten/).

> **Transferable:** The OWASP Top 10, injection prevention, supply-chain hygiene (lockfiles, audits, minimal dependencies) and least privilege apply to every language. Only the tool names change: `pip-audit`, `cargo audit`, `govulncheck`, OWASP Dependency-Check.

### 45. Performance, scaling and deployment

**Measure first.** Load test (`npx autocannon -c 100 -d 10 http://localhost:3000/api/v1/tasks`), profile ([topic 34](#34-debugging-and-profiling)), and fix the actual bottleneck. It is usually the database or the network, rarely JavaScript itself.

**Common wins**

- Never block the event loop ([topic 13](#13-blocking-vs-non-blocking-dont-block-the-event-loop)). Move CPU work to worker threads or queues.
- Run independent I/O concurrently with `Promise.all` ([topic 15](#15-asynchronous-patterns-callbacks-promises-and-asyncawait)).
- Use database **connection pools**, indexes, and avoid N+1 queries (one query per item in a loop).
- **Cache**: in-memory LRU for hot data, Redis for shared caches, HTTP caching headers (`Cache-Control`, `ETag`) for clients and CDNs.
- **Stream** large payloads instead of buffering them ([topic 23](#23-streams-and-backpressure)). Paginate big collections.
- Let a reverse proxy or CDN handle TLS, compression and static files.
- Upgrade Node: each major version brings V8 and runtime performance improvements.

**Scaling**

- **Vertical:** a bigger machine. Simple, but limited, and a single process uses about one core for JavaScript.
- **Horizontal:** more instances behind a load balancer. This requires **stateless** instances: sessions in Redis or signed tokens, files in object storage (S3), never in process memory or on the local disk.

```mermaid
%%{init: {"htmlLabels": false, "flowchart": {"htmlLabels": false, "wrappingWidth": 400, "padding": 24}}}%%
flowchart LR
    USERS(["Users"]) --> LB["Load balancer<br/>or reverse proxy<br/>nginx, cloud LB,<br/>Kubernetes Service"]
    LB --> A1["API container 1"]
    LB --> A2["API container 2"]
    LB --> A3["API container 3"]
    A1 & A2 & A3 --> DB[("PostgreSQL")]
    A1 & A2 & A3 --> CACHE[("Redis<br/>cache, sessions,<br/>rate limits")]
    A1 & A2 & A3 --> Q[["Job queue"]] --> WORKER["Worker containers<br/>emails, reports,<br/>image processing"]
```

**Deployment options**

| Option | Examples | Good for |
| --- | --- | --- |
| **PaaS** | Render, Railway, Fly.io, Heroku, Azure App Service | Fastest path from Git push to a running app |
| **Containers** | Google Cloud Run, AWS ECS/Fargate, Azure Container Apps, **Kubernetes** | Full control, any stack, run the image from [topic 41](#41-dockerizing-the-nodejs-api) |
| **Serverless functions** | AWS Lambda, Azure Functions, Vercel/Netlify functions | Spiky or low traffic, pay per request; watch out for cold starts and statelessness |
| **Edge runtimes** | Cloudflare Workers, Vercel Edge | Very low latency worldwide; web-standard APIs, not full Node |
| **Virtual machines** | EC2, DigitalOcean droplets + PM2 or systemd + nginx | Maximum control, maximum maintenance |

**A typical CI/CD pipeline:**

```mermaid
%%{init: {"htmlLabels": false, "flowchart": {"htmlLabels": false, "wrappingWidth": 400, "padding": 24}}}%%
flowchart LR
    PUSH["git push /<br/>pull request"] --> CI["CI<br/>npm ci, lint, test"]
    CI --> BUILD["docker build<br/>+ vulnerability scan"]
    BUILD --> REGISTRY[("Container registry")]
    REGISTRY --> STAGING["Deploy to staging<br/>smoke tests"]
    STAGING --> PRODUCTION["Deploy to production<br/>rolling update,<br/>health checks"]
```

### 46. Upgrading Node.js safely

Node majors arrive yearly (twice a year before Node 27), and each LTS line has a fixed end date, so upgrading is a routine skill. The same process works for any version jump and any runtime.

**1. Keep the version in one place per tool, and keep them in sync**

| Where | What to change |
| --- | --- |
| `.nvmrc` | `26` |
| `package.json` | `"engines": { "node": ">=26" }` |
| `Dockerfile` | `ARG NODE_VERSION=26` |
| CI | Test matrix (below) |
| Hosting | The platform's runtime setting, if not using Docker |

**2. Test on both versions in CI before switching**

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [24, 26]
    steps:
      - uses: actions/checkout@v5
      - uses: actions/setup-node@v5
        with:
          node-version: ${{ matrix.node-version }}
          cache: npm
      - run: npm ci
      - run: npm test
```

**3. The upgrade checklist**

1. Read the release notes and the **"Notable changes" / "semver-major"** sections of every major version you skip.
2. Run your tests on the **current** version with `--trace-deprecation` (or `--throw-deprecation`) and fix the deprecations first. They often become removals in the next major.
3. Update dependencies (`npm outdated`). Check that important packages support the new version (their `engines` field and changelogs).
4. **Native addons** (packages with C/C++ code, e.g. `bcrypt`, `sqlite3`) must be rebuilt for each major (`npm rebuild`, or a fresh `npm ci`). Addons built on **Node-API** stay compatible across versions; older-style addons do not.
5. Watch for behavior changes in: OpenSSL (TLS settings, legacy algorithms), default `highWaterMark`/stream behavior, module loading (ESM/CJS interop), and removed experimental flags.
6. Deploy gradually (staging, then a canary or a rolling deploy) and watch error rates, memory and latency.

> **Transferable:** "Pin, test on a matrix, fix deprecations first, read the changelog of every skipped major, roll out gradually" is exactly how teams upgrade Python, Java, .NET, Go or a framework like Spring or React.

### 47. Transferable knowledge: Node.js concepts in other languages

The ideas in this guide exist in every backend stack. Use this table as a map when you learn your next language:

| Concept | Node.js | Python | Go | Java | C# / .NET | Rust |
| --- | --- | --- | --- | --- | --- | --- |
| Runtime / execution | V8 JIT + libuv | CPython interpreter (PyPy JIT) | Compiled to native + Go runtime | JVM (JIT) | CLR (JIT) | Compiled to native, no runtime GC |
| Concurrency model | Event loop + async/await, worker threads | asyncio + async/await, threads, multiprocessing | Goroutines + channels | Threads, virtual threads (21+), CompletableFuture | async/await (Task), thread pool | async/await + tokio, threads |
| Package manager | npm, pnpm, Yarn | pip, uv, Poetry | go modules | Maven, Gradle | NuGet | Cargo |
| Manifest | `package.json` | `pyproject.toml` | `go.mod` | `pom.xml`, `build.gradle` | `*.csproj` | `Cargo.toml` |
| Lockfile | `package-lock.json` | `uv.lock`, `poetry.lock` | `go.sum` | Gradle lockfiles | `packages.lock.json` | `Cargo.lock` |
| Version manager | nvm, fnm | pyenv, uv | `toolchain` in go.mod | SDKMAN | `global.json` | rustup |
| Web framework | Express, Fastify, NestJS | FastAPI, Django, Flask | net/http, Gin, Echo | Spring Boot | ASP.NET Core | Axum, Actix Web |
| Middleware | `app.use()` | ASGI/WSGI middleware | `http.Handler` wrappers | Servlet filters, interceptors | Middleware pipeline | Tower layers |
| Validation | zod | Pydantic | validator | Bean Validation | FluentValidation, DataAnnotations | serde + validator |
| Tests | `node:test`, Vitest, Jest | pytest | `go test` | JUnit | xUnit, NUnit | `cargo test` |
| Env config | `process.env` | `os.environ` | `os.Getenv` | `System.getenv` | `IConfiguration` | `std::env` |
| Typical Docker base | `node:26-alpine` | `python:3.x-slim` | Build with `golang`, ship `scratch`/distroless | `eclipse-temurin` JRE | `mcr.microsoft.com/dotnet/aspnet` | Build with `rust`, ship slim/distroless |

**Questions to ask when you learn any new runtime or language**

1. **Engine:** is it interpreted, JIT-compiled or compiled ahead of time? How is memory managed (GC, reference counting, ownership)?
2. **Concurrency:** event loop, OS threads, green threads or goroutines? What blocks, and how do I run CPU work in parallel?
3. **Async style:** callbacks, futures/promises, async/await, channels?
4. **Modules and packages:** how is code split, how are dependencies declared, where is the lockfile?
5. **Versions:** what is the release cadence, which version is LTS, how do I pin it per project?
6. **Errors:** exceptions or error values? What happens to an unhandled error?
7. **Standard library:** what is built in (HTTP, testing, JSON, crypto) before I add dependencies?
8. **Tooling:** formatter, linter, test runner, debugger, profiler?
9. **Shipping:** how do I build a container image for it, and what is the idiomatic base image?

If you can answer these for Node.js (and after this guide you can), you can answer them for anything else.

### 48. Sharing your project and next steps

**Make your project easy to run for others.** A good README contains:

1. What the project does (one paragraph).
2. **Prerequisites:** the Node version (point to `.nvmrc`) and the package manager if it is not npm.
3. **Setup:** `git clone ...`, `cd ...`, `npm install` (or `pnpm install` / `yarn`), `cp .env.example .env`.
4. **Run:** `npm run dev` for development, `npm start` for production, or `docker compose up`.
5. **Test:** `npm test`.
6. **Configuration:** a table of environment variables and what they mean.
7. License and how to contribute.

**Files every Node repository should have:** `package.json`, the lockfile, `.gitignore` (at least `node_modules/` and `.env`), `.nvmrc`, `.env.example`, a `README.md`, a `LICENSE`, and, if you deploy with containers, a `Dockerfile` + `.dockerignore`.

**A learning path from here**

1. Rebuild the Tasks API from memory, then add PostgreSQL ([topic 42](#42-docker-compose-the-api-with-a-database)), authentication (JWT) and pagination.
2. Convert it to TypeScript ([topic 33](#33-typescript-with-nodejs)).
3. Add structured logging with `pino` and API docs with OpenAPI.
4. Deploy the Docker image to a container platform with a CI pipeline ([topic 45](#45-performance-scaling-and-deployment)).
5. Build something real-time with WebSockets (the `ws` package or Socket.IO).
6. Try the same API in a second framework (Fastify or NestJS), then in a second language (FastAPI or Go). [Topic 47](#47-transferable-knowledge-nodejs-concepts-in-other-languages) is your map.

**References**

- Node.js API docs: [nodejs.org/api](https://nodejs.org/api/) (versioned docs: `https://nodejs.org/docs/latest-v26.x/api/`)
- Node.js guides: [nodejs.org/en/learn](https://nodejs.org/en/learn), including [the event loop explained](https://nodejs.org/en/learn/asynchronous-work/event-loop-timers-and-nexttick) and [don't block the event loop](https://nodejs.org/en/learn/asynchronous-work/dont-block-the-event-loop)
- Release schedule: [github.com/nodejs/Release](https://github.com/nodejs/Release)
- V8 blog (engine internals): [v8.dev/blog](https://v8.dev/blog)
- libuv design overview: [docs.libuv.org](https://docs.libuv.org/en/v1.x/design.html)
- Express: [expressjs.com](https://expressjs.com/) and the [Express 5 migration guide](https://expressjs.com/en/guide/migrating-5.html)
- Zod: [zod.dev](https://zod.dev/)
- Docker docs: [docs.docker.com](https://docs.docker.com/) and the [official Node.js Docker best practices](https://github.com/nodejs/docker-node/blob/main/docs/BestPractices.md)
- Node.js best practices (community): [github.com/goldbergyoni/nodebestpractices](https://github.com/goldbergyoni/nodebestpractices)
- The Twelve-Factor App: [12factor.net](https://12factor.net/)
