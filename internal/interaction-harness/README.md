# @astryxdesign/interaction-harness

用于触摸交互与动画验证的 Playwright 工具：真实触摸输入、帧证据、曲线采样与故事板。**内部使用，不发布。** 测试规格与组件放在一起，命名为 `*.interaction.browser.spec.ts`，在 [`playwright.interaction.config.ts`](../../playwright.interaction.config.ts) 下运行于手机尺寸的触摸屏（页面 402 × 714 CSS px，屏幕 402 × 874，每 CSS px 3 个设备像素，触摸，移动视口规则），引擎为 Chromium 和 WebKit。无障碍测试规格（`*.a11y.browser.spec.ts`，在 [`playwright.config.ts`](../../playwright.config.ts) 下）也可以使用动画保持和定时器闸门，以检查动画经过的某个状态。

可参考的范例是 [`BottomSheetDetent.interaction.browser.spec.ts`](../../packages/core/src/BottomSheet/__tests__/BottomSheetDetent.interaction.browser.spec.ts)。工具集在 [`src/harness.interaction.browser.spec.ts`](src/harness.interaction.browser.spec.ts) 中自证其工具。主线程繁忙验证和渲染追踪计数在正向检查旁配有负向对照（一个在阻塞主线程下冻结的 rAF 动画，一个每帧都触发布局的 `top` 动画）；其他自测用已知答案校验工具。

## 运行

```sh
pnpm storybook:build                      # 测试规格从构建后的 Storybook 读取故事
pnpm exec playwright install chromium webkit
pnpm test:interaction --project chromium  # 全部，Chromium
pnpm test:interaction BottomSheetDetent   # 单个规格，两个引擎
pnpm storyboards                          # 仅运行标记为 @storyboard 的测试
```

在系统库版本低于 Playwright 的 WebKit 构建所需版本的 Linux 发行版上，请在官方 Playwright 镜像中运行 WebKit 项目（podman 或 docker；仓库挂载在同一路径，在 rootful docker 守护进程下容器以你的用户身份运行，因此其 `test-results/` 仍归你所有）：

```sh
node internal/interaction-harness/scripts/webkit-container.mjs [playwright test args…]
```

WebKit 在那里通过 Mesa 的软件 GL 进行合成，每个着色器在页面首次绘制时编译（模糊或新动画层的首帧需 450–700 ms）。脚本将 Mesa 的着色器缓存保存在宿主机上（`~/.cache/astryx-webkit-shaders/<image>`，或 `ASTRYX_WEBKIT_SHADER_CACHE`），因此只有机器上的首次运行需要付出编译代价。

设置 `CI` 时，脚本将 CI 的 WebKit 运行器的四个 CPU 作为配额给容器，llvmpipe 也使用相同线程数。`ASTRYX_WEBKIT_CPUS` 可设置其他数量。在 Linux 上容器只在脚本可用的 CPU 上运行，因此 `taskset` 也会限制它。

证据（故事板、曲线追踪）写入 `test-results/interaction-evidence/`（可用 `ASTRYX_INTERACTION_EVIDENCE` 覆盖），并附加到测试报告。它从不提交：Pull Request 链接或附加它。

## 工具

| 模块            | 为测试规格提供什么                                                                                                                                                                                                                                                         |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `scripts.ts`    | 触摸脚本即数据：`tap`、`drag`（按住、静止）、`flick`（精确的最终移动速度、速度曲线）、`holdThenDrag`、`pinch`、`rotate`、`path`、`merge`（手势中途第二根手指）、`releaseVelocity`、`toTouchEvents`。                                                |
| `touch.ts`      | `touchDriver(page).play(script)`：脚本作为真实引擎输入。`installInputLog` / `readInputLog`：页面收到了什么。                                                                                                                                               |
| `frames.ts`     | `startScreencast`（带时间戳的合成帧）、`markElement`（标记芯片或轮廓）、`trackMarker`（每帧标记框）、`trackMarkers`（每帧一次解码多种颜色）、`sampleRects`（每 rAF 矩形）、`frameStats`、`requireFrames`。              |
| `busy.ts`       | `blockMainThread(page, ms)`，返回它持有的窗口及其循环测量的跨度，以及 `busyWindowProof(tracked, window)`：呈现的帧、移动的帧、最长停顿、判定；`presented` 添加合成器自身的帧。                           |
| `brightness.ts` | `trackBrightness(page, frames, {x, y})`：每帧一个方形区域内某颜色通道的均值，作为 `box.x` 上的标量，用于淡入或变暗的繁忙窗口验证。                                                                                                                  |
| `webInput.ts`   | `webInputDriver(page)`：`wheel({x, y, dx, dy, ctrl})` 和 `pen(points)` 作为真实引擎输入，用于跨指针验证（`spec:AST-100` FR4）；`wheel` 在页面收到该步后 resolve；通过输入日志读回。                                                 |
| `trace.ts`      | `traceRendering(page, body)`：UpdateLayoutTree / Layout / PrePaint / Paint / Layerize / UpdateLayer / ScrollLayer / HitTest 的主线程计数，`countBetween`，长任务；`installLongTaskLog` / `readLongTasks`；`startCompositorTrace`：合成器何时呈现。 |
| `curves.ts`     | `holdAnimations`、`armHold`、`seekAnimations`、`sampleCurve`（任意 t 处呈现的 transform / opacity）、`waitForAnimation`、`watchAnimations`、`waitForRest`、`releaseAnimations`、`releaseAnimationLog`（在堆或监听器计数前释放记录器保留的内容）。     |
| `math.ts`       | 独立于被测代码的判定基准：`cubicBezier`、`timingFunction`、`springStep`（SwiftUI response / dampingFraction，初始速度）、`projectDecay`、`rubberBand`、`parseTransform`、`compareCurve`、`timeline`。                                                |
| `storyboard.ts` | `animationStoryboard`（跳转到 t = 0, 16.7, 33.3 … ms）、`touchStoryboard`（每个脚本事件一帧）、`castStoryboard`（真实录屏帧）：PNG 联系表 + JSON 追踪。                                                                                           |
| `output.ts`     | `evidenceDir`、`writeEvidence`：证据存放位置及如何附加到报告。                                                                                                                                                                                    |
| `story.ts`      | `phone`、`withStorybook`、`openStory`、`frames`，以及定时器闸门（`installTimerGate`、`freezeTimers`、`thawTimers`）。                                                                                                                                                      |
| `pointer.ts`    | `newFinePointerPage(browser)`：一个真正读取 `(pointer: fine)` 的鼠标页面。`assertFinePointer` / `assertCoarsePointer`：当页面读取另一种指针时，在用例检查前失败。                                                                     |

## 精细指针

每个交互项目都是触摸手机，测试用 `browser.newContext()` 创建的上下文会继承它：除非传入 `hasTouch: false`，否则读取 `(pointer: coarse)`。一个展示触摸工作在鼠标下无成本的用例，用 `newFinePointerPage(browser)` 打开页面，它会关闭触摸和移动，并先检查指针。自行构建上下文的用例在检查前调用 `assertFinePointer(page)`。

## 验证

**真实触摸。** 不可信的 DOM 事件不是输入：它们不启动原生滚动、不认领手势，且读取为 `isTrusted: false`。Chromium 输入通过 CDP `Input.dispatchTouchEvent`，每个事件带显式时间戳：页面的 `event.timeStamp` 是脚本时间，精确到 0.1 ms，无论派发延迟如何，事件经过合成器的手势检测（原生滚动、基于这些时间戳的速度甩动、`touch-action`、`pointercancel`）。WebKit 输入通过引擎自身的输入命令；见下方限制。在移动后一帧读取位置：Chromium 将 `pointermove` 对齐到动画帧。

**故事，而非 Storybook。** `openStory` 在它打开的每个页面上关闭 Storybook 的高亮模块，并检查其根元素不存在。若保持开启，该模块会观察 `#storybook-root`，并在每次插入时，在插入任务内读取页面上每个元素的计算样式（Button 故事中一次退出复制产生 117 次读取和两次强制样式重算），因此验证会同时测量 Storybook 和组件。`openStory(page, origin, id, args, {storybookHighlight: true})` 保持开启。

**曲线采样。** 暂停所有运行中的动画（CSS 过渡和动画也是 Web Animations），跳转到某个时间，读取 `getComputedStyle`：该精确时间的呈现值，独立于帧时序。已完成的动画、页面暂停的动画以及滚动或视图时间线上的动画不受影响；被保持的动画保留其相对于首次保持最早开始的偏移，释放只恢复它们。从测试发送的保持经过一个往返到达页面，在负载高的运行器上短动画可能先结束：对于由动作启动的动画，在动作前 `await armHold(page, scope, {judges})` 让页面自行武装：它在每帧以及帧间每 4 ms 查看（WebKit 在首次合成层时可能约 70 ms 无帧），一旦作用域下任一动画运行就保持所有运行中的动画（或 `count` 个，用于分开启动的部分；使用 `newOnly` 时，只保持武装后启动的，用于运动中开始的转向），动作返回后 `held()` 返回该保持集。作用域下的其他动画（按压、颜色变化）可能在测试判定的运动开始前就满足该计数，只保持它们的保持对判定运动毫无证明。因此每个武装的保持都命名它判定的运动，`{judges: {target, property}}`（多个用列表；`pseudoElement` 用于 `::before` 或 `::after`），当保持未捕获其任何动画时 `held()` 拒绝。`judges: 'count-only'` 不检查任何东西：它标记一个调用点，其保持尚未捕获其判定的运动。从测试发送的保持（`holdAnimations`，以及不带 `held` 的 `animationStoryboard`）接受相同的 `judges` 并以相同方式拒绝：它捕获到达页面时正在运行的动画，在繁忙运行器上判定的运动可能尚未到来或已经结束。通过 `compareCurve` 与 `math.ts` 的判定基准比较；容差表示误差预算（对于烘焙为 `h` ms 的 `linear()` 停靠点的弹簧，弦误差至多 `h² / 8 · |x''|`）。跳转到动画结束之后会落在其内部半毫秒处，即它呈现的最后一帧，采样会说明（`presentedAt`，判定基准读取处）。关键帧以一段相同值结尾的动画（保留结束值以便后续中断读取）在该段开始处结束：`curveEndMs`、`waitForAnimation` 的间隔和默认故事板时间在那里停止，而 `waitForRest`（判定呈现的 `transform`、`translate`、`rotate`、`scale` 和 `opacity` 而非 `playState`）不会等待保持结束。这适用于在线性或三次贝塞尔效果缓动下正向播放的动画（该段开始时间从曲线解出）；在任何其他效果缓动（steps、`linear()` 停靠点、过冲）或其他方向下，曲线读取到其完整结束，`waitForRest` 等待保持结束。在开始前被反转的动画（对新动画调用 `reverse()`、从一开始负播放速率）由 `waitForAnimation` 从其开始时报告，但 `holdAnimations` 拒绝它：跳转和采样正向运行曲线。记录器每个 `Animation` 对象保留一个条目，因此运行中反转或重播的动画会被误读：反转前取的监视会拒绝，未监视的等待会立即返回一个日期早于它的曲线。为验证读取的每次运行取一个新动画。`sampleCurve(locator, times, {pseudoElement: '::before'})` 改为采样元素的 `::before` 或 `::after`：其呈现值，由该伪元素自身的动画标记。

**等待动画。** 页面从首次调用 `curves.ts` 起记录每个动画（以及之后每个文档）：页面自身 `requestAnimationFrame` 上的帧循环、CSS 动画和过渡事件，以及调用时的 `Element.animate`，包括待定动画。因此 `waitForAnimation` 也会报告在上次 `waitForRest`、`waitForAnimation` 或 `watchAnimations` 之后启动且已完成的动画（设置 `finishedAt`）：短入场不再在冷页面上与首次查看竞争。页面在播放时取消的运行在那里结束，无完成事件。Core 的运动动画器在曲线结束后用 `cancel()` 退役每次运行。记录器在取消时给这样的运行定日期，等待在它呈现 `minProgressMs` 后报告它。当动作可能是页面与工具集的首次接触时，在动作前取 `watchAnimations(locator)`，动作后读取 `watch.started()`。

**主线程繁忙。** 标记元素，启动录屏，启动动画，等待它运行（`waitForAnimation`：线程被持有时动画无法启动），阻塞主线程 1 s（`spec:AST-094` FR5 的验证行），停止。始终先启动手势或动画并在飞行中阻塞，绝不在之前：在无头 Chromium 中，主线程繁忙时开始的触摸即使只有被动监听器也不会滚动或启动手势（输入等待主线程）。因此在主线程繁忙下跟踪手指在 iOS 模拟器上验证，不在此处。录屏在主线程外捕获，因此合成器动画在阻塞的每一帧都显示新位置，而主线程动画一帧都不显示（无变化，因此根本不发送帧：`busyWindowProof` 将整个窗口计为一次停顿）。`settledWithin`（2 px）在缓动尾部停止判定：录屏舍入到整像素，因此仍在移动的尾部显示距静止一两像素内的平台。`settledWithinSteps` 以录屏自身的位置步长计数该界限，从录屏读取（`castStep`，来自 `trackMarker` 为默认标记每帧给出的 `step`；由其他颜色跟踪的对照不带步长，`castStep` 拒绝它）：Chromium 的 PNG 录屏为 1 px，WebKit 的 JPEG 录屏为 2 px，其 4:2:0 色度将标记颜色保持在 2×2 像素块中，使尾部平台更长。两个步长在两个引擎中都在同一尾部停止，而距静止更远的冻结仍被判定。

每帧慢于两个整像素的运动需要更精细的读取。在 Chromium 中，`trackMarker(page, frames, {subpixel: true})` 从其抗锯齿边缘读取亚像素标记：60 帧内 10 px 滑动每帧读取 0.164 到 0.169 px，实际移动 0.167 px，而像素间静止的框每帧读取相同值。每帧随后携带 `SUBPIXEL_STEP_PX`（0.05 px）作为其步长，验证从它的一半计数变化。标记必须在纯色背景上不透明。使用默认芯片标记：轮廓标记的窗口包含元素自身内容。WebKit 的 JPEG 录屏将颜色保持在 2×2 像素块中，因此读取在那里抛出。

判定部分短于录屏间隔的四倍则失败：被测动画在阻塞期间未移动。判定部分开始时已稳定的（窗口内无可判定）抛出 `BusyWindowTooLateError`，带数字：阻塞运行太晚，这不是对组件的读取。使用 `during` 时，`blockMainThread` 从看到动画的那一帧中排队的任务启动循环，在该帧渲染更新之后：WebKit 在那里将新启动的动画交给合成器，而 rAF 回调内的循环会冻结甚至合成器动画。在启动稳定的释放前调用它，之后 await，这样稳定开始时页面已在等待。动画在不再待定且已运行至少一帧后计数：低于 16 的 `minProgressMs` 提升为 16，因为页面查看的帧中创建的动画尚未在合成器上。它只在其曲线的保持尾部之前计数（以相同值段结尾的关键帧，按 `waitForRest` 读取方式读取）：当那里唯一运行的动画在页面等待时启动，或等待超时且只有此类动画时，页面查看太晚，等待以 `BusyWindowTooLateError` 拒绝。使用 `pseudoElement`（`'::before'` 或 `'::after'`）时，只有 `during` 自身上的该伪元素动画计数，按 `sampleCurve` 读取方式：子元素的动画（如开关拇指的按压）不启动阻塞。`verdict` 判定录屏交付的帧，无论其速率（负载机器可能五六帧才交付一帧），且不容忍冻结动画的任何表现：判定部分至少一帧，无长于录屏间隔三倍的停顿（两帧丢失），且判定部分交付的帧中至少 90% 呈现新位置（总允许一次未命中；未交付的帧是间隙，用于停顿检查，除非那里交付少于四帧：则判定部分的跨度计每一帧，因此一帧迟到不是"1 of 1 moved"）。仅在声称显示速率的验证中传入 `maxMedianFrameMs`（60 Hz 下为 20）。`verdict.failures` 命名每次未命中及其数字，`summary` 携带原始数字供报告。帧按呈现顺序判定：负载机器可能乱序交付。`castStoryboard` 将阻塞的帧转为胶片条。

刚启动的动画可能在阻塞开始后到达屏幕（负载机器上 WebKit 中晚 50–100 ms）。当录屏在阻塞前看到元素静止时（在启动动画前等待 `cast.frameCount()` 超过 0），判定从其首次呈现变化开始：判定部分仍以阻塞结束且仍需四个间隔，`onScreenFromMs` 和摘要报告它何时到来。移动过、然后从阻塞前保持到其