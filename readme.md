# TS Reactive Library Comparison <!-- omit from toc -->

- [Intro](#intro)
- [TC39 Signals Proposal](#tc39-signals-proposal)
- [Reactive Library Comparison](#reactive-library-comparison)
- [Benchmark Results](#benchmark-results)
- [Takeaways](#takeaways)
  - [Signals Standard](#signals-standard)
  - [Performance](#performance)
  - [Preact Signals](#preact-signals)
  - [Vue's Reactivity](#vues-reactivity)
  - [MobX](#mobx)
  - [Wrapping Reactive Values](#wrapping-reactive-values)
  - [Shallow vs Deep Reactivity](#shallow-vs-deep-reactivity)
  - [Autotracking Dependencies](#autotracking-dependencies)
  - [Effect Scoping](#effect-scoping)
- [Related Resources](#related-resources)
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
| [Solid Signals](https://docs.solidjs.com/concepts/intro-to-reactivity)            | ✅                  | ✅[^solid-signals]       | ✅               | [❌](https://github.com/Sawtaytoes/reactjs-solidjs-bridge) | ❌                                   | [21kb](https://pkg-size.dev/solid-js)            |
| [Qwik Signals](https://qwik.dev/docs/components/state/#usesignal)                 | ✅                  | ✅[^qwik-signals]        | ✅               | ❌?                                                        | ❌                                   | [83kb](https://pkg-size.dev/@builder.io/qwik)    |
| [Angular Signals](https://angular.dev/guide/signals)                              | ✅                  | ✅                       | ✅               | ❌                                                         | ❌                                   | [360kb](https://pkg-size.dev/@angular/core)      |
| [Svelte v5](https://svelte.dev/blog/runes)[^svelte-reactivity]                    | ✅                  | ✅                       | ✅               | ❌                                                         | ❌                                   | [35kb](https://pkg-size.dev/@vue/svelte)         |
| [MobX](https://mobx.js.org)                                                       | ❌                  | ✅                       | ✅               | ✅                                                         | ✅                                   | [64kb](https://pkg-size.dev/mobx)                |
| [Reactively](https://github.com/milomg/reactively)                                | ❌                  | ❌                       | ✅               | ✅                                                         | ✅                                   | [2.6kb](https://pkg-size.dev/@reactively/core)   |
| [Signia](https://signia.tldraw.dev/)                                              | ✅                  | ❌                       | ✅               | ✅                                                         | ✅                                   | [9kb](https://pkg-size.dev/signia)               |
| [NanoStores](https://github.com/nanostores/nanostores)[^nanostores]               | ✅                  | ❌                       | ❌               | ✅                                                         | ✅                                   | [4kb](https://pkg-size.dev/nanostores)           |
| [Jotai](https://jotai.org)                                                        | ✅                  | ❌                       | ❌               | ✅                                                         | ✅                                   | [7kb](https://pkg-size.dev/jotai)                |
| [Valtio](https://github.com/pmndrs/valtio)                                        | ✅                  | ❌                       | ❌               | ✅                                                         | ✅                                   | [7kb](https://pkg-size.dev/valtio)               |

I also considered other reactive libraries such as [cellx](https://github.com/Riim/cellx), [Glimmer's tracked](https://github.com/tracked-tools/tracked-built-ins), [RxJS](https://rxjs.dev), [Starbeam](https://www.starbeamjs.com), [S.js](https://github.com/adamhaile/S), [compostate](https://github.com/lxsmnsyc/compostate), [uSignal](https://github.com/WebReflection/usignal), [Pota](https://github.com/potahtml/pota), [Kairo](https://github.com/3Shain/kairo), [mol wire](https://www.npmjs.com/package/mol_wire_lib), [Oby](https://github.com/vobyjs/oby), and more, but didn't include them above because they are either not actively maintained, don't support TypeScript, or are not close enough to the signals proposal to warrant comparison.

> [!NOTE]
> Did I miss your favorite reactive library or get something wrong? Please [create an issue](https://github.com/transitive-bullshit/ts-reactive-comparison/issues/new) to let me know. 🙂

[^standalone-usage]: Standalone usage refers to how easy it is to use just the reactivity functionality of the library as an isolated package – without being tied to any specific frontend libraries ala React, Vue, Svelte, etc.
[^signal-utils]: While the official [Signals Proposal](https://github.com/proposal-signals/signal-polyfill) focuses on core dependency tracking of shallow signals, its sister project [signal-utils](https://github.com/proposal-signals/signal-utils) offers support for deep, Proxy-based tracking.
[^preact-deep-signals]: There is a third-party project built on top of [Preact Signals](https://github.com/preactjs/signals) called [DeepSignal](https://github.com/luisherranz/deepsignal). There is also another third-party project called [@preact-signals/utils](https://github.com/XantreDev/preact-signals) which offers [a port of Vue 3's deep tracking API](https://github.com/XantreDev/preact-signals/tree/main/packages/utils#deep-reactivity-port-of-vue-3-deep-tracking-api). Note that neither `DeepSignal` or `@preact-signals/utils` seems very mature, and neither are backed by a robust team of maintainers.
[^vue-reactivity-react]: [Vue's reactivity system](https://vuejs.org/guide/essentials/reactivity-fundamentals.html) (`@vue/reactivity`) doesn't provide out-of-the-box support for `react`, but it should be possible to integrate via [effectScope](https://vuejs.org/api/reactivity-advanced.html#effectscope) as evidenced by the now deprecated [reactivue project](https://github.com/antfu/reactivue).
[^solid-signals]: [Solid](https://www.solidjs.com) supports both [shallow signals](https://docs.solidjs.com/concepts/signals) and [deep stores](https://docs.solidjs.com/concepts/stores).
[^qwik-signals]: [Qwik](https://qwik.dev) supports both [shallow signals](https://qwik.dev/docs/components/state/#usesignal) and [deep stores](https://qwik.dev/docs/components/state/#usestore).
[^svelte-reactivity]: [Svelte](https://svelte.dev) has embraced reactivity perhaps more than any other frontend framework given that it's [v5 reactive engine based on runes](https://svelte.dev/blog/runes) expects compiler support as opposed to direct, standalone usage. This is a really interesting and efficient approach, but it's also tightly coupled with Svelte as a whole and not currently meant for general JS/TS usage outside of Svelte. It is, however, the most performant TS reactivity framework available according to our benchmark results, which is why we've included it in this comparison.
[^nanostores]: [Nanostores](https://github.com/nanostores/nanostores) is a nice, lean reactivity library, but it's `computed` requires you to explicitly provide a single dependency, making its DX very awkward compared to other libraries which automatically handle any number of dependency subscriptions.

## Benchmark Results

<p align='center'>
  <a href="https://github.com/transitive-bullshit/js-reactivity-benchmark"><img src="https://github.com/user-attachments/assets/f6d041a1-d5da-4b30-8e60-b4c815ac70bc" alt="Average benchmark results across frameworks"></a>
  (<em>lower times are better</em>)
</p>

These results are from [my fork](https://github.com/transitive-bullshit/js-reactivity-benchmark) of [@milomg](https://github.com/milomg)'s excellent [js-reactivity-benchmark](https://github.com/milomg/js-reactivity-benchmark), which aggregates signal benchmarks from several libraries together.

Note that some reactive libraries are missing from this benchmark because they either don't support standalone usage or because they didn't support the benchmark's abstract signals abstraction.

Note also that none of the benchmark tests use deep reactivity of objects, though many of them do test performance on deep graphs of shallow signals. It would be an interesting extension to compare the performance of deeply reactive objects as well.

These results were last updated _September 2024_ on an M3 Macbook Pro using Node.js v22.4.1.

## Takeaways

_(this section is largely subjective)_

### Signals Standard

After putting this comparison together, I feel even more strongly that this ecosystem would seriously benefit from a mature [Signals Standard](https://github.com/proposal-signals/signal-polyfill). Maybe that's just confirmation bias at work, but there's so much duplicated work across these libs that's not interoperable, and complex state management is at the heart of so many apps. Having a standard `Signal` implementation in JavaScript would do wonders for the ecosystem. 💯

### Performance

All of the reactive libs offer approximately similar performance on the [benchmark](https://github.com/transitive-bullshit/js-reactivity-benchmark) with a few outliers.

On the negative side of perf, we have [Angular Signals](https://angular.dev/guide/signals) and [signal-polyfill](https://github.com/proposal-signals/signal-polyfill). I'm not surprised that the Signals standard's polyfill has significantly worse perf since it's more of a proof-of-concept and explicitly not production ready. I was surprised, however, at how poorly the perf was of Angular Signals, which is presumably backed by a team of seasoned Google engineers.

On the positive side of perf, **[Svelte v5's runes](https://svelte.dev/blog/runes) outperform every other framework by a solid margin** ([discussion](https://github.com/sveltejs/svelte/discussions/13277)). The only unfortunate part of this is that Svelte's reactivity APIs (`$state`, `$derived`, `$effect`, etc) are all internal and are not officially supported by the Svelte team for standalone, external usage as opposed to, for instance, `@vue/reactivity` which is exposed specifically for this purpose.

Also on the positive side of perf is [Reactively](https://github.com/milomg/reactively), currently the second fastest signals framework, which makes sense given that the author [@milomg](https://github.com/milomg) also authored most of the [benchmark](https://github.com/transitive-bullshit/js-reactivity-benchmark). It's a shame that `reactively` hasn't been adopted by any larger projects that I'm aware of which could help with maintenance, but maybe the Signals standard can take some inspiration from Reactively's seemingly more efficient implementation going forwards. The author wrote a [detailed description of his approach here](https://github.com/milomg/reactively/blob/main/Reactive-algorithms.md).

I was surprised that several popular libs hung or ran into memory exceptions running some of the benchmark tests (notably [Mobx](https://github.com/mobxjs/mobx/issues/3926) and [Valtio](https://github.com/pmndrs/valtio/discussions/949)), though these benchmark tests do create fairly deep signal graphs of different shapes and usage patterns to try and stress test things.

### Preact Signals

Several projects have used [Preact Signals](https://github.com/preactjs/signals) as a sort of unofficial standard to build off of because it is lightweight, efficient, and thoroughly tested. **Out of all the shallow reactive libraries which support standalone usage, Preact Signals is imho the best one in terms of DX and maturity**.

The only real downside is that it doesn't support deep reactivity, which I've found to be extremely valuable as an opt-in feature for real-world usage, especially when using reactivity outside of frontend use cases (like game engines). There are several projects[^preact-deep-signals] which add deep signal support to `@preact/signals-core` via Proxy-based tracking, but they are less mature and not backed by a large team, which makes me hesitant to adopt them.

### Vue's Reactivity

[Vue's reactivity](https://vuejs.org/guide/essentials/reactivity-fundamentals.html) (`@vue/reactivity`) is imho the most mature and well-maintained TypeScript signals implementation which supports both deep + shallow tracking along with standalone usage.

Naming-wise, I don't really like `ref` as a name for `signal` especially because refs already have a well-defined meaning in React land, but this is a minor nit.

My benchmarking exposed one [perf issue](https://github.com/vuejs/core/issues/11928) with `@vue/reactivity` which was promptly fixed by the Vue team, resulting in a ~5% speedup across all benchmarks.

**Overall, I really like `@vue/reactivity` and am leaning towards using it as a base for my work going forwards** until the TC39 Signals Proposal is mature enough to use as a replacement.

### MobX

[MobX](https://mobx.js.org) was one of the earliest reactive JS libraries to gain popularity, and I shipped several successful apps with it back in the day, but these days it seems to be suffering from its own success and backwards compatibility.

Namely, MobX still [supports non-Proxy-based deep reactivity](https://mobx.js.org/configuration.html#proxy-support) for older browsers. It still [supports legacy decorators](https://mobx.js.org/enabling-decorators.html#using-decorators). It [supports six different patterns for adding reactivity to a class](https://mobx.js.org/observable-state.html). MobX packages don't export ESM. And their list of [caveat limitations](https://mobx.js.org/observable-state.html#limitations) is thoroughly confusing. [MobX is also contains a subtle, but very severe perf footgun](https://github.com/mobxjs/mobx/issues/3926) when used outside of frontend frameworks that was exposed by my benchmarking. This is meant as constructive criticism, fwiw; I know how hard it is to run OSS projects.

I'd love to see a [major MobX v7 release](https://github.com/mobxjs/mobx/issues/3796) which fixes these issues and drops backwards compatibility. I'd also love to see the MobX maintainers [explore using the Signals standard polyfill](https://github.com/mobxjs/mobx/issues/3854) to help push it along.

### Wrapping Reactive Values

One important design decision is whether to provide direct access to reactive values (like Vue's [reactive](https://vuejs.org/guide/essentials/reactivity-fundamentals.html#reactive)) or to wrap the reactive value in an accessor (like Vue's [ref](https://vuejs.org/guide/essentials/reactivity-fundamentals.html#ref)). Most reactive libs wrap values in an accessor, even though it hurts the DX a bit, and [Vue's docs have a great explanation for why this is a useful tradeoff](https://vuejs.org/guide/essentials/reactivity-fundamentals.html#limitations-of-reactive).

Reactive wrappers generally provide access to the underlying value using one of the following: `wrapper.get()`, `wrapper.value`, `get(wrapper)`, or `wrapper()`, and some support automatic unwrapping when used in JSX or templates and/or at compile-time (like Svelte). ([more info on this pattern from signia](https://signia.tldraw.dev/docs/what-are-signals#how-do-you-access-the-value-of-a-signal))

### Shallow vs Deep Reactivity

Another important design decision is **whether reactivity should be shallow or deep**.

Personally, I'd rather have an API that defaults to deep reactivity (with the creation of deep signals being lazy via Proxy-based tracking). You can then expose a shallow API that is opt-in instead of defaulting to shallow and having to opt-in to deep tracking. The reason is simple: less cognitive overhead. As an app developer, reasoning about state in a complex app is already complicated enough. I'd rather default to throwing my app state and models into a reactive system and knowing that mutations will "just work" without having to think about whether the shape of my data works with the shallow reactivity contraint – and then only opting in to shallow reactivity as a perf optimization where it makes sense.

This is, of course, a subjective preference, and one could easily make the opposite argument. E.g., defaulting to shallow means less magic (aka unseen complexity) lurking in the app's state.

It's worth noting that both [Qwik](https://qwik.dev/docs/components/state/#usesignal) and [Solid](https://docs.solidjs.com/concepts/intro-to-reactivity) differentiate between the two approaches by using **shallow signals** and **deep stores**. I think this is a nice compromise, since they're at least separated into different concepts which makes it easier to reason about as long as the shallow signals and deep stores interact in a way that "just works".

Examples of APIs that default to deep reactivity include `@vue/reactivity`'s `ref`, MobX, and Angular. Most of the signal APIs in this comparison default to shallow signals, but that's also because most don't support deep tracking.

### Autotracking Dependencies

Some libraries like [Valtio](https://github.com/pmndrs/valtio), [NanoStores](https://github.com/nanostores/nanostores), and [Jotai](https://jotai.org) don't automatically track dependencies referenced inside of computed properties and effects, which makes their relative DX significantly more cumbersome and from my benchmarks, not any faster. The same can also be said for React's own [useEffect](https://react.dev/reference/react/useEffect), which requires the developer to manually declare an effect's dependencies. [Ain't nobody got time for that](https://youtu.be/waEC-8GFTP4?t=25).

### Effect Scoping

One of the more important but less-visible design considerations is how a framework handles scoping and garbage collection of its reactive dependency graph(s).

Many of these frameworks are optimized for usage in component-based frontend frameworks like React/Vue/Svelte/Angular, and their default effect lifecyles therefore tend to work well out-of-the-box with component lifecycles (mount, render, unmount, etc). When it comes to standalone usage and/or usage outside of a component hierarchy, the ability to define separate root scopes becomes much more important.

[Vue's effectScope RFC](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0041-reactivity-effect-scope.md) provides a great description of the problem and their approach to solving it. Other examples of effect scoping include [Svelte v5's $effect.root](https://svelte-5-preview.vercel.app/docs/runes#$effect-root), [S.js's root](https://github.com/adamhaile/S#computation-roots), and [Solid's createRoot](https://docs.solidjs.com/reference/reactive-utilities/create-root).

Most of these frameworks, however, do not support any sort of effect scoping, meaning that all effects and their respective dependency graphs are shared globally by default.

## Related Resources

For a more in-depth guide to signals, check out these awesome resources:

- [TC39 Signals Proposal](https://github.com/tc39/proposal-signals)
- [Vue's Guide to Reactivity in Depth](https://vuejs.org/guide/extras/reactivity-in-depth.html)
- [Signia's Signals Overview and Design Space Exploration](https://signia.tldraw.dev/docs/what-are-signals)
- [Reactive Algorithms Breakdown](https://github.com/milomg/reactively/blob/main/Reactive-algorithms.md)

## License

MIT © [Travis Fischer](https://x.com/transitive_bs)
