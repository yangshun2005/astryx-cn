# StyleX CSS 能力参考

> 由 `internal/stylex-capabilities/scan.mjs` 自动生成
> StyleX 版本：**0.19.0**
> 生成时间：2026-09-28

支持 **88/92** 项特性

## At 规则

| 特性                                                 | 状态 | CSS 输出                                   |
| ---------------------------------------------------- | ---- | ------------------------------------------ |
| @media                                               | ✅   | `ltr: ".x1e2nbdu{color:red}",              |
| ltr: "@media (max-width: 768px){.x1v9tpll.x1v9tpll{` |
| @media (hover: hover)                                | ✅   | `ltr: ".x1e2nbdu{color:red}",              |
| ltr: "@media (hover: hover){.x1qg27m4.x1qg27m4{colo` |
| @media (prefers-reduced-motion)                      | ✅   | `ltr: ".x4afe7t{animation-duration:.3s}",  |
| ltr: "@media (prefers-reduced-motion: r`             |
| @media (prefers-color-scheme)                        | ✅   | `ltr: ".x1mqxbix{color:black}",            |
| ltr: "@media (prefers-color-scheme: dark){.xat0y8`   |
| @supports                                            | ✅   | `ltr: ".x78zum5{display:flex}",            |
| ltr: "@supports (display: grid){.xr4m3m1.xr4m3m1{`   |
| @container                                           | ✅   | `ltr: ".xif65rj{font-size:14px}",          |
| ltr: "@container (min-width: 400px){.x5ca7px.x5`     |
| @container（命名）                                   | ✅   | `ltr: ".xif65rj{font-size:14px}",          |
| ltr: "@container sidebar (min-width: 400px){.x1`     |
| @layer                                               | ❌   | 错误                                       |
| @starting-style                                      | ✅   | `ltr: ".x1hc1fzr{opacity:1}",              |
| ltr: "@starting-style{.x4itv7f.x4itv7f{opacity:0}}"` |
| @starting-style（transform）                         | ✅   | `ltr: ".xbryuvx{transform:translateX(0)}", |
| ltr: "@starting-style{.x1yqmsfc.x1yqms`              |

## 伪类

| 特性                                              | 状态 | CSS 输出                                   |
| ------------------------------------------------- | ---- | ------------------------------------------ |
| :hover                                            | ✅   | `ltr: ".x1e2nbdu{color:red}",              |
| ltr: ".x17z2mba:hover{color:blue}",`              |
| :focus                                            | ✅   | `ltr: ".x1e2nbdu{color:red}",              |
| ltr: ".x1cg8fzh:focus{color:blue}",`              |
| :focus-visible                                    | ✅   | `ltr: ".x1a2a7pz{outline:none}",           |
| ltr: ".xp1qbqn:focus-visible{outline:2px solid b` |
| :focus-within                                     | ✅   | `ltr: ".xzppbbw{border-color:gray}",       |
| ltr: ".x1v09i5w:focus-within{border-color:bl`     |
| :active                                           | ✅   | `ltr: ".x1e2nbdu{color:red}",              |
| ltr: ".x4oye9d:active{color:blue}",`              |
| :disabled                                         | ✅   | `ltr: ".x1hc1fzr{opacity:1}",              |
| ltr: ".xijokvz:disabled{opacity:.5}",`            |
| :first-child                                      | ✅   | `ltr: ".x1xmf6yo{margin-top:8px}",         |
| ltr: ".x14l7nz5:first-child{margin-top:0}",`      |
| :last-child                                       | ✅   | `ltr: ".x1e56ztr{margin-bottom:8px}",      |
| ltr: ".xzboxd6:last-child{margin-bottom:0}"`      |
| :nth-child()                                      | ✅   | `ltr: ".x12peec7{background-color:white}", |
| ltr: ".x91ra16:nth-child(even){backgro`           |
| :where()                                          | ✅   | `ltr: ".x1s85apg{display:none}",           |
| ltr: ".xf1xu1x.xf1xu1x:where([open]){display:fle` |
| :is()                                             | ✅   | `ltr: ".x1e2nbdu{color:red}",              |
| ltr: ".xdxic9:is(:hover, :focus){color:blue}",`   |
| :has()                                            | ✅   | `ltr: ".xzppbbw{border-color:gray}",       |
| ltr: ".x1vswxf2:has(:focus){border-color:blu`     |
| :not()                                            | ✅   | `ltr: ".x1lliihq{display:block}",          |
| ltr: ".x41xwj8:not(:last-child){display:block}"`  |
| :placeholder-shown                                | ✅   | `ltr: ".x1mqxbix{color:black}",            |
| ltr: ".x8yzjlm:placeholder-shown{color:gray}",`   |
| :checked                                          | ✅   | `ltr: ".x12peec7{background-color:white}", |
| ltr: ".x5ry8ap:checked{background-colo`           |
| :empty                                            | ✅   | `ltr: ".x1lliihq{display:block}",          |
| ltr: ".xjl7jj:empty{display:none}",`              |

## 伪元素

| 特性                                      | 状态 | CSS 输出                                                        |
| ----------------------------------------- | ---- | --------------------------------------------------------------- |
| ::before                                  | ✅   | `ltr: ".x1cpjm7i::before{content:\"\"}",                        |
| ltr: ".x1fgarty::before{display:block}",` |
| ::after                                   | ✅   | `ltr: ".x1s928wv::after{content:\"\"}",                         |
| ltr: ".xhkezso::after{display:block}",`   |
| ::placeholder                             | ✅   | `ltr: ".x6yu8oj::placeholder{color:gray}",                      |
| ltr: ".xefv9yz::placeholder{font-style`   |
| ::selection                               | ✅   | `ltr: ".x1hr2xla::selection{background-color:blue}",            |
| ltr: ".x50npbo::selection{co`             |
| ::backdrop                                | ✅   | `ltr: ".x17fofgw::backdrop{background-color:rgba(0,0,0,.5)}",`  |
| ::marker                                  | ✅   | `ltr: ".xab66pc::marker{color:blue}",`                          |
| ::file-selector-button                    | ✅   | `ltr: ".x1905k51::file-selector-button{background-color:blue}", |
| ltr: ".xjdbxwa::f`                        |

## 复合选择器

| 特性                                                    | 状态 | CSS 输出                                       |
| ------------------------------------------------------- | ---- | ---------------------------------------------- |
| 伪元素上的伪类（::backdrop + 条件）                     | ✅   | `ltr: ".xph5o2a::backdrop{opacity:0}",         |
| ltr: ".x1pgog5j.x1pgog5j:where([data-open]`             |
| RTL 上下文（:is([dir="rtl"] *)）                        | ✅   | `ltr: ".x5i6ehr{transform:translateX(-100%)}", |
| ltr: ".xttggg:is([dir=\"rtl\"] *){`                     |
| 嵌套 @media + 伪类                                      | ✅   | `ltr: ".x1e2nbdu{color:red}",                  |
| ltr: ".x17z2mba:hover{color:blue}",                     |
| ltr: "@media (h`                                        |

## CSS 值

| 特性                              | 状态 | CSS 输出                                                   |
| --------------------------------- | ---- | ---------------------------------------------------------- |
| CSS 变量（自定义属性）            | ✅   | `ltr: ".xqwr325{color:var(--color-accent)}",`              |
| calc()                            | ✅   | `ltr: ".xkxa9zd{width:calc(100% - 32px)}",`                |
| clamp()                           | ✅   | `ltr: ".xdq3cbq{font-size:clamp(14px,2vw,18px)}",`         |
| light-dark()                      | ✅   | `ltr: ".xarbrzd{color:light-dark(black,white)}",`          |
| color-mix()                       | ✅   | `ltr: ".x10zuzju{color:color-mix(in srgb,red 50%,blue)}",` |
| container-type                    | ✅   | `ltr: ".x12h1iku{container-type:inline-size}",`            |
| container-name                    | ✅   | `ltr: ".x4s5kx9{container-name:sidebar}",`                 |

## 过渡与动画

| 特性                               | 状态 | CSS 输出                                                                           |
| ---------------------------------- | ---- | ---------------------------------------------------------------------------------- |
| transition（简写）                 | ✅   | `ltr: ".xqz5vs7{transition:opacity .3s ease}",`                                    |
| transition（单独属性）             | ✅   | `ltr: ".x19991ni{transition-property:opacity}",`                                   |
| ltr: ".x1d8287x{transition-durati` |
| transitionBehavior: allow-discrete | ✅   | `ltr: ".xd00j3c{transition-behavior:allow-discrete}",`                             |
| animation（简写）                  | ❌   | <repo>/test.tsx: 不支持 animation                                                  |
| stylex.keyframes                   | ✅   | `ltr: "@keyframes x1wc8ddo-B{from{transform:rotate(0deg);}to{transform:rotate(360` |

## 动态样式

| 特性                                       | 状态 | CSS 输出                                |
| ------------------------------------------ | ---- | --------------------------------------- |
| 动态值（stylex.create 中的函数）           | ✅   | `ltr: ".x5lhr3w{width:var(--x-width)}", |
| ltr: "@property --x-width { syntax: \"*\"` |

## 变量

| 特性                                   | 状态 | CSS 输出                                                      |
| -------------------------------------- | ---- | ------------------------------------------------------------- |
| stylex.defineVars                      | ✅   | `ltr: ":root, .x1cptbry{--x15ffsfu:blue;--x1gusifi:green;}",` |
| stylex.createTheme                     | ✅   | `ltr: ":root, .x1cptbry{--x15ffsfu:blue;}",                   |
| ltr: ".x1il70uh, .x1il70uh:root{--x15` |

## 祖先/兄弟选择器

| 特性                                                 | 状态 | CSS 输出                             |
| ---------------------------------------------------- | ---- | ------------------------------------ |
| stylex.when.ancestor(:hover)                         | ✅   | `ltr: ".x1e2nbdu{color:red}",        |
| ltr: ".xobp4yc.xobp4yc:where(.x-default-marker:hove` |
| stylex.when.ancestor(:hover) + @media                | ✅   | `ltr: ".x1e2nbdu{color:red}",        |
| ltr: "@media (hover: hover){.x1mm45h8.x1mm45h8.x1mm` |
| stylex.when.ancestor(:focus-within)                  | ✅   | `ltr: ".xzppbbw{border-color:gray}", |
| ltr: ".x1y7zxk6.x1y7zxk6:where(.x-default-ma`        |
| stylex.when.ancestor(:active)                        | ✅   | `ltr: ".x1c071of{transform:none}",   |
| ltr: ".x1nyz7lo.x1nyz7lo:where(.x-default-mark`      |
| stylex.when.ancestor(:disabled)                      | ✅   | `ltr: ".x1hc1fzr{opacity:1}",        |
| ltr: ".x1ymx0df.x1ymx0df:where(.x-default-marker:di` |
| stylex.when.ancestor([data-attr])                    | ✅   | `ltr: ".x1e2nbdu{color:red}",        |
| ltr: ".x1vt6d5i.x1vt6d5i:where(.x-default-marker[da` |
| stylex.when.descendant(:hover)                       | ✅   | `ltr: ".x1e2nbdu{color:red}",        |
| ltr: ".xepvv76.xepvv76:where(:has(.x-default-marker` |
| stylex.when.siblingBefore(:checked)                  | ✅   | `ltr: ".x1e2nbdu{color:red}",        |
| ltr: ".x72zv5i.x72zv5i:where(.x-default-marker:chec` |
| stylex.when.siblingAfter(:checked)                   | ✅   | `ltr: ".x1e2nbdu{color:red}",        |
| ltr: ".x1e2z1t3.x1e2z1t3:where(:has(~ .x-default-ma` |
| stylex.when.anySibling(:hover)                       | ✅   | `ltr: ".x1hc1fzr{opacity:1}",        |
| ltr: ".x1tq5f2g.x1tq5f2g:where(.x-default-marker:ho` |

## StyleX API

| 特性                         | 状态 | CSS 输出                                                                           |
| ---------------------------- | ---- | ---------------------------------------------------------------------------------- |
| stylex.firstThatWorks()      | ✅   | `ltr: ".x1fb8ovd{display:flex;display:grid}",`                                     |
| stylex.positionTry()         | ✅   | `ltr: "@position-try --x19yd6gm {top:top;top:anchor(end);}",                       |
| ltr: ".xvc0jx8{posit`        |
| stylex.viewTransitionClass() | ✅   | `ltr: "::view-transition-old(*.x8n7qjs){opacity:0;}::view-transition-new(*.x8n7qj` |
| stylex.defineConsts()        | ✅   | `ltr: "",                                                                          |
| ltr: "",`                    |
| stylex.types.color()         | ✅   | `ltr: "@property --x15ffsfu { syntax: \"<color>\"; inherits: true; initial-value:` |
| stylex.types.length()        | ✅   | `ltr: "@property --xdwkrp { syntax: \"<length>\"; inherits: true; initial-value: ` |

## 前沿特性

| 特性                                                 | 状态 | CSS 输出                                                               |
| ---------------------------------------------------- | ---- | ---------------------------------------------------------------------- |
| @scope                                               | ✅   | `ltr: ".x1e2nbdu{color:red}",                                          |
| ltr: "@scope (.card){.x1ph799q.x1ph799q{color:blue}` |
| @property（显式）                                    | ❌   | 错误                                                                   |
| ::view-transition-old                                | ✅   | `ltr: ".x1bb6q1l::view-transition-old(root){animation-duration:.3s}",` |
| ::view-transition-new                                | ✅   | `ltr: ".x1wpo8qz::view-transition-new(root){animation-duration:.3s}",` |
| :modal                                               | ✅   | `ltr: ".x12peec7{background-color:white}",                             |
| ltr: ".x1mt27nc:modal{background-color`              |
| anchor() 定位                                        | ✅   | `ltr: ".xnj0yav{top:anchor(--my-anchor bottom)}",                      |
| ltr: ".x1os70c6{position-anchor`                     |
| CSS 嵌套（&）                                        | ❌   | <repo>/test.tsx: 无效的伪类或 at 规则。                                |
| @media (scripting: none)                             | ✅   | `ltr: ".x1lliihq{display:block}",                                      |
| ltr: "@media (scripting: none){.x1cfwndu.x1cfwn`     |
| :user-valid / :user-invalid                          | ✅   | `ltr: ".xzppbbw{border-color:gray}",                                   |
| ltr: ".xwdq3mt:user-invalid{border-color:red`        |
| color-scheme 属性                                    | ✅   | `ltr: ".x108lcm5{color-scheme:light dark}",`                           |
| text-wrap: balance                                   | ✅   | `ltr: ".x1w2vvpw{text-wrap:balance}",`                                 |
| text-wrap: pretty                                    | ✅   | `ltr: ".x1fzhlzt{text-wrap: