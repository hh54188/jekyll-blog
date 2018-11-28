# Mobx 与 Redux 的性能对比

我无意挑起战争

在本文中你将看到我最终得出的结论是 Mobx 的性能优于 Redux。但很明显这样的结论是片面的，甚至是有失偏颇的，因为我只选取了有限的场景对两者进行测试。可能真实的情况恰恰相反，Mobx 仅仅在我测试的这几个场景中优于 Redux，但是在我所有没有测试到的场景中都劣于 Redux，这都是有可能的。性能跑分这类东西从来都不要放在心上，「鲁大师」不也是被戏称为「娱乐大师」嘛。

本文的重点不在于让两者拼个你死我活，而是在对比性能的过程中探索优劣可能是由什么原因造成的，并且我们能从中学习到什么

退一万步说，即使 Redux 性能确实略逊一筹，也无伤大雅。当我们在评价一个框架，或者在为产品做技术选型时，性能只是其中的一个方面。比如 Redux 天生的 event sourcing 机制能够帮助我们方便的回溯状态，如果你的产品里需要这样的业务场景，那么 Redux 当然是不二之选。通常在低于某个阈值下性能不会出现大的差别。

## 和谁比，怎么比

让我们从一个 stackoverflow 上关于 Mobx 的[有趣的性能问题](https://stackoverflow.com/questions/38460113/mobx-performance)开始

提问者做了一个测试，往`observable.array`装饰过的数组（Mobx 自己的数据结构）中`push`200个元素，计算总共花费的时间，并且和原生的操作进行能比较。结果是使用 Mobx 的方式一共花费了 120ms, 而原生的操作只花费了不到 1ms。这是不是说明了 Mobx 性能非常糟糕？

理论上来说提问者的测试方法没有错，测试的结果也是正确的。但问题在于单纯数值上的对比是有失公允的，虽然原生数组`push`方法更快，但是它无法提供单向数据流、无法提供状态管理不是？同时 Mobx 也能与React 进行配合优化组件的渲染。所以我们不能仅仅考量数值上的大小，还要考虑整体利益的得失

在我做优化工作的早期，我习惯于使用工程上的指标，比如 DOMContentLoaded 时间，onLoad 时间，软性一点的是 Speed Index。但目前我更倾向于使用业务性质的指标，因为你要想清除一个问题是，工程的指标真的和业务指标正相关吗？如果 onLoad 时间边长，页面的 PV 就真的会下降吗？理论上是，但并不一定，相反如果你顽皮一点，你完全能够做到让 onLoad 的时间边长，但是 PV 上升，只要保证 above fold content 足够快和可用就好了

做优化的目的


[https://www.exp-platform.com/Documents/2014%20experimentersRulesOfThumb.pdf](https://www.exp-platform.com/Documents/2014%20experimentersRulesOfThumb.pdf)

>At Bing, we use multiple performance metrics for diagnostics, but
our key time-related metric is Time-To-Success (TTS) [24], which
side-steps the measurement issues. For a search engine, our goal is
to allow users to complete a task faster. For clickable elements, a
user clicking faster on a result from which they do not come back
for at least 30 seconds is considered a successful click. TTS as a
metric captures perceived performance well: if it improves, then
important areas of the pages are rendering faster so that users can
interpret the page and click faster. This relatively simple metric
does not suffer from heuristics needed for many performance
metrics. It is highly robust to changes, and very sensitive. Its main
deficiency is that it only works for clickable elements. For queries
where the SERP has the answer (e.g., for “time” query), users can
be satisfied and abandon the page without clicking.