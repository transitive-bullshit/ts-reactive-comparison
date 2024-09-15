# TS Reactive Library Comparison <!-- omit from toc -->

- [Intro](#intro)
- [TC39 Signals Proposal](#tc39-signals-proposal)
- [Reactive Library Comparison](#reactive-library-comparison)
- [Benchmark Results](#benchmark-results)
- [Takeaways](#takeaways)
- [What are Signals?](#what-are-signals)
- [License](#license)

## Intro

I fell in love with [MobX](https://mobx.js.org) back in 2015, which then kind of fell out of popularity after React switched from classes to functions and legacy JS decorators took a nosedive. So when I saw the new [TC39 Signals Proposal](https://github.com/tc39/proposal-signals) combined with [TC39 Decorators](https://github.com/tc39/proposal-decorators) hitting Stage 3 and general adoption, I got excited and started wondering what a modern replacement for MobX would look like.

I wanted to start by doing a deep dive on the current state of reactive TypeScript libraries aimed at understanding the most actively maintained, battle-tested, and feature-rich signal libraries that exist which support standalone usage[^standalone-usage] to build upon.

Goals for this deep dive:

- Bring more awareness to the [TC39 Signals Proposal](https://github.com/tc39/proposal-signals).
- Better understand the space of signals design tradeoffs and adoption in order to provide feedback to the [TC39 Signals Proposal](https://github.com/tc39/proposal-signals) and [related standardization efforts](https://github.com/proposal-signals/signal-utils).
- Choose a mature existing signals library to build off of while the Signals Proposal matures with the intent being to switch to the official polyfill once it's ready.

## TC39 Signals Proposal

**[The TC39 Signals Proposal](https://github.com/tc39/proposal-signals) aims to add a standard reactive Signal primitive to JavaScript.** 🔥

This has the potential to greatly change how the majority of state management is handled across JS/TS, just like when `Promise` was introduced as a TC39 standard back in 2015.

[TC39 Signals](https://github.com/tc39/proposal-signals) are currently a [Stage 1 Proposal](https://tc39.es/process-document/). There is a [signal-polyfill](https://github.com/proposal-signals/signal-polyfill) implementation, but it is not recommended for production just yet, and it currently lacks some common features like [batch](https://github.com/tc39/proposal-signals/issues/73) and [effect](https://github.com/tc39/proposal-signals#implementing-effects). A sister project, [signal-utils](https://github.com/proposal-signals/signal-utils), adds some nice functionality like opt-in support for deep, Proxy-based tracking, decorators, an `effect` implementation, and a `batch` implementation. It is unclear, however, how stable / mature these APIs are.

## Reactive Library Comparison

| Library                                                                           | Actively developed? | Supports deep?           | Autotracks deps? | Supports `react`?                                          | Standalone usage?[^standalone-usage] | Bundle size                                      |
| --------------------------------------------------------------------------------- | ------------------- | ------------------------ | ---------------- | ---------------------------------------------------------- | ------------------------------------ | ------------------------------------------------ |
| [Signals Proposal Polyfill](https://github.com/proposal-signals/signal-polyfill)  | **?**               | ❌[^signal-utils]        | ✅               | ✅                                                         | ✅                                   | [9kb](https://pkg-size.dev/signal-polyfill)      |
| [Preact Signals](https://github.com/preactjs/signals)                             | ✅                  | ❌[^preact-deep-signals] | ✅               | ✅                                                         | ✅                                   | [4kb](https://pkg-size.dev/@preact/signals-core) |
| [Vue Reactivity](https://vuejs.org/guide/essentials/reactivity-fundamentals.html) | ✅                  | ✅                       | ✅               | ❌[^vue-reactivity-react]                                  | ✅                                   | [20kb](https://pkg-size.dev/@vue/reactivity)     |
| [MobX](https://mobx.js.org)                                                       | ❌                  | ✅                       | ✅               | ✅                                                         | ✅                                   | [64kb](https://pkg-size.dev/mobx)                |
| [Jotai](https://jotai.org)                                                        | ✅                  | ❌                       | ❌               | ✅                                                         | ❌                                   | [7kb](https://pkg-size.dev/jotai)                |
| [Solid Signals](https://docs.solidjs.com/concepts/intro-to-reactivity)            | ✅                  | ✅[^solid-signals]       | ✅               | [❌](https://github.com/Sawtaytoes/reactjs-solidjs-bridge) | ❌                                   | [21kb](https://pkg-size.dev/solid-js)            |
| [Qwik Signals](https://qwik.dev/docs/components/state/#usesignal)                 | ✅                  | ✅[^qwik-signals]        | ✅               | ❌?                                                        | ❌                                   | [83kb](https://pkg-size.dev/@builder.io/qwik)    |
| [Angular Signals](https://angular.dev/guide/signals)                              | ✅                  | ✅                       | ✅               | ❌                                                         | ❌                                   | [360kb](https://pkg-size.dev/@angular/core)      |
| [Signia](https://signia.tldraw.dev/)                                              | ✅                  | ❌                       | ✅               | ✅                                                         | ✅                                   | [9kb](https://pkg-size.dev/signia)               |
| [Valtio](https://github.com/pmndrs/valtio)                                        | ✅                  | ❌                       | ❌               | ✅                                                         | ❌                                   | [7kb](https://pkg-size.dev/valtio)               |
| [NanoStores](https://github.com/nanostores/nanostores)[^nanostores]               | ✅                  | ❌                       | ❌               | ✅                                                         | ✅                                   | [4kb](https://pkg-size.dev/nanostores)           |

We also considered other reactive libraries such as [cellx](https://github.com/Riim/cellx), [reactively](https://github.com/milomg/reactively), [Glimmer's tracked](https://github.com/tracked-tools/tracked-built-ins), [RxJS](https://rxjs.dev), [Starbeam](https://www.starbeamjs.com), [S.js](https://github.com/adamhaile/S), [compostate](https://github.com/lxsmnsyc/compostate), [uSignal](https://github.com/WebReflection/usignal), [Svelte's reactivity](https://github.com/sveltejs/svelte/blob/main/packages/svelte/src/internal/client/reactivity/sources.js)[^svelte-reactivity], [Pota](https://github.com/potahtml/pota), [Kairo](https://github.com/3Shain/kairo), [Compostate](https://github.com/lxsmnsyc/compostate), [mol wire](https://www.npmjs.com/package/mol_wire_lib), [Oby](https://github.com/vobyjs/oby), [Reactively](https://github.com/milomg/reactively), and more, but didn't include them above because they are either not actively maintained, don't support TypeScript, or are not close enough to the signals proposal to warrant comparison.

Some reactive libraries like [Valtio](https://github.com/pmndrs/valtio), [NanoStores](https://github.com/nanostores/nanostores), and [Jotai](https://jotai.org) don't automatically track dependencies referenced inside of computed properties and effects, which imho makes their relative DX significantly more cumbersome and from my benchmarks, not any faster. The same can also be said for React's own [useEffect](https://react.dev/reference/react/useEffect), which requires the developer to manually declare an effect's dependencies. [Ain't nobody got time for that](https://youtu.be/waEC-8GFTP4?t=25).

[^standalone-usage]: Standalone usage refers to how easy it is to use just the reactivity functionality of the library as an isolated package – without being tied to any specific frontend libraries ala React, Vue, Svelte, etc.
[^signal-utils]: While the official [Signals Proposal](https://github.com/proposal-signals/signal-polyfill) focuses on core dependency tracking of shallow signals, its sister project [signal-utils](https://github.com/proposal-signals/signal-utils) offers support for deep, Proxy-based tracking.
[^preact-deep-signals]: There is a third-party project built on top of [Preact Signals](https://github.com/preactjs/signals) called [DeepSignal](https://github.com/luisherranz/deepsignal). There is also another third-party project called [@preact-signals/utils](https://github.com/XantreDev/preact-signals) which offers [a port of Vue 3's deep tracking API](https://github.com/XantreDev/preact-signals/tree/main/packages/utils#deep-reactivity-port-of-vue-3-deep-tracking-api). Note that neither `DeepSignal` or `@preact-signals/utils` seems very mature, and neither are backed by a robust team of maintainers.
[^vue-reactivity-react]: [Vue's reactivity system](https://vuejs.org/guide/essentials/reactivity-fundamentals.html) (`@vue/reactivity`) doesn't provide out-of-the-box support for `react`, but it should be possible to integrate via [effectScope](https://vuejs.org/api/reactivity-advanced.html#effectscope) as evidenced by the now deprecated [reactivue project](https://github.com/antfu/reactivue).
[^solid-signals]: [Solid](https://www.solidjs.com) supports both [shallow signals](https://docs.solidjs.com/concepts/signals) and [deep stores](https://docs.solidjs.com/concepts/stores).
[^qwik-signals]: [Qwik](https://qwik.dev) supports both [shallow signals](https://qwik.dev/docs/components/state/#usesignal) and [deep stores](https://qwik.dev/docs/components/state/#usestore).
[^svelte-reactivity]: [Svelte](https://svelte.dev) has embraced reactivity perhaps more than any other frontend framework given that it's [reactive engine](https://github.com/sveltejs/svelte/blob/main/packages/svelte/src/internal/client/reactivity) works at compile-time rather than run-time. This is a really interesting and efficient approach, but it's also tightly coupled with Svelte as a whole and not compatible with general JS/TS usage, which is why we're not including it in this comparison.
[^nanostores]: [Nanostores](https://github.com/nanostores/nanostores) is a nice, lean reactivity library, but it's `computed` requires you to explicitly provide a single dependency, making its DX very awkward compared to other libraries which automatically handle any number of dependency subscriptions.

## Benchmark Results

<p align='center'>
	<a href="https://github.com/transitive-bullshit/js-reactivity-benchmark/tree/feature/update"><img src="https://github.com/user-attachments/assets/4621879c-fb20-4056-8fd8-f7daa31a07e3" alt="Framework average benchmark results"></a>
	<a href="https://github.com/user-attachments/files/16992605/reactivity-bench.csv">Raw results CSV</a> (<em>lower times are better</em>)
</p>

These results are from [my fork](https://github.com/transitive-bullshit/js-reactivity-benchmark/tree/feature/update) of [@milomg](https://github.com/milomg)'s excellent [js-reactivity-benchmark](https://github.com/milomg/js-reactivity-benchmark), which aggregates signal benchmarks from several libraries together.

Note that MobX and Valtio are not included in the average results summary because they fail to run some of the benchmark tests. Some of the other reactive libraries are missing from the benchmark because they either don't support standalone usage or because they didn't support the benchmark's abstract signals abstraction.

Note also that none of the benchmark tests use deep reactivity of objects, though many of them do test performance on deep graphs of shallow signals. It would be very interesting to compare performance of deeply reactive objects / arrays / `Map` / `Set` objects.

These results were last updated _September 2024_ on an M3 Macbook Pro.

## Takeaways

1. The ecosystem would seriously benefit from a mature [Signals Standard](https://github.com/proposal-signals/signal-polyfill). There's just so much duplicated work, and complex state management is at the heart of so many apps. Having a standard `Signal` implementation in JavaScript would do wonders for this ecosystem. 💯

2. Several projects have used [Preact Signals](https://github.com/preactjs/signals) as a sort of unofficial standard to build off of because it is lightweight, efficient, and thoroughly tested. **Out of all the shallow, standalone reactive libraries out there, it is imho the best one in terms of DX and maturity**. The only real downside is that it doesn't officially support deep reactivity, which I've found to be extremely valuable as an opt-in feature for real-world usage, especially when using reactivity outside of frontend use cases (like game engines). There are several projects[^preact-deep-signals] which add deep signal support to `@preact/signals-core` via Proxies, but they are less mature and not backed by a large team, meaning that they'd be riskier to depend on in the long run.

3. [Vue's reactivity](https://vuejs.org/guide/essentials/reactivity-fundamentals.html) (`@vue/reactivity`) is imho the most mature and well-maintained TypeScript signals implementation which supports both deep + shallow tracking along with standalone usage. Naming-wise, I don't really like `ref` as a name for `signal` especially because refs already have a well-defined meaning in React land, but this is a minor nit. **Overall, I really like `@vue/reactivity` and will be using it as a base for my work going forwards** until the TC39 Signals Proposal is mature enough to use as a replacement.

4. One important design decision is whether to provide direct access to reactive values (like Vue's [reactive](https://vuejs.org/guide/essentials/reactivity-fundamentals.html#reactive)) or to wrap the reactive value in an accessor (like Vue's [ref](https://vuejs.org/guide/essentials/reactivity-fundamentals.html#ref)). Most reactive libs wrap values in an accessor, even though it makes the DX slightly worse, and [Vue's docs have a great explanation for why](https://vuejs.org/guide/essentials/reactivity-fundamentals.html#limitations-of-reactive). Reactive wrappers generally provide access to the underlying value using one of the following: `wrapper.get()`, `wrapper.value`, `get(wrapper)`, or `wrapper()`, and some support automatic unwrapping when used in JSX or templates and/or at compile-time (like Svelte). ([more info on this pattern from signia](https://signia.tldraw.dev/docs/what-are-signals#how-do-you-access-the-value-of-a-signal))

## What are Signals?

Signals are a declarative way to implement reactivity across an application's state. They simplify the management of potentially complex state graphs by making sure that values are automatically recomputed if and only if their dependencies change.

Many frontend libraries use this reactivity as a way to only re-render components whenever their dependent state changes, as opposed to relying on the developer to explicitly re-render them which becomes significantly more error-prone as apps get more complex (e.g., React, Vue, Svelte vs jQuery).

For a more in-depth guide to signals, check out these awesome resources:

- [TC39 Signals Proposal](https://github.com/tc39/proposal-signals)
- [Vue's Guide to Reactivity in Depth](https://vuejs.org/guide/extras/reactivity-in-depth.html)
- [Signia's Signals Overview and Design Space Exploration](https://signia.tldraw.dev/docs/what-are-signals)
- [Reactive Algorithms Breakdown](https://github.com/milomg/reactively/blob/main/Reactive-algorithms.md)

## License

MIT © [Travis Fischer](https://x.com/transitive_bs)
