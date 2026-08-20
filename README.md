# WebGal-CustomIframeEngine
基于WebGal开发版https://github.com/xiaoxustudio/WebGAL/tree/feat/createFrame-command 打包的WebGal引擎
关于 **`WebGAL` 的 `createIframe`（即你所提到的自定义网页/Iframe 引擎功能）**，它是 WebGAL 用来嵌入网页小游戏、HTML 交互页面、UI 扩展等核心的高级指令。

由于它允许 WebGAL 与外部网页（HTML/JS）进行双向通信，因此在制作像“三棱镜色散小游戏”这类独立玩法时非常实用。以下是该功能的完整使用教程和核心参数解析：

---

### 一、 核心指令与语法格式

在 WebGAL 脚本中，调用和关闭网页的指令主要有两个：

1. **创建并嵌入 iframe：**
```webgal
createIframe:页面路径 -id=窗口ID -width=宽度 -height=高度 -wait -returnValue=变量名 -allowSameOrigin -allowScripts -allowModals;

```


2. **移除/关闭 iframe：**
```webgal
removeIframe:窗口ID;

```



---

### 二、 核心参数详解（以你的小游戏为例）

我们在前面代码中用的这一句：

```webgal
createIframe:1.html -id=prismGame -width=100% -height=100% -wait -returnValue=result -allowSameOrigin -allowScripts -allowModals;

```

每一个参数的作用如下：

* **`1.html`**：你要嵌入的网页文件路径。通常放在项目根目录的 `public` 或资源文件夹下。
* **`-id=prismGame`**：给这个 iframe 窗口起一个唯一标识。之后用 `removeIframe:prismGame;` 关闭它时，全靠这个 ID 来精准定位。
* **`-width=100% -height=100%`**：设置窗口的尺寸。如果做全屏小游戏，直接设为 `100%` 即可。
* **`-wait`（极为重要）**：**阻塞标志**。加上它之后，WebGAL 主剧本会暂停运行，等待 iframe 内部主动触发“关闭并返回值”的信号后，主剧本才会继续往下走。
* **`-returnValue=result`**：接收来自 iframe 网页内部传回的数据。如果小游戏通关了，它可以把分数或状态（如 `result=success`）传回给 WebGAL 的全局变量。
* **权限参数**（保证网页正常运行）：
* `-allowSameOrigin`：允许同源策略。
* `-allowScripts`：允许 iframe 内部运行 JavaScript 脚本（小游戏逻辑必备）。
* `-allowModals`：允许 iframe 弹出提示框等。



---

### 三、 WebGAL 与网页（iframe）的双向通信

要想让你的 `1.html` 小游戏和 WebGAL 完美互动，单靠脚本里的 `createIframe` 还不够，**网页内部的 HTML/JS 也需要配合**。

#### 1. 小游戏如何把结果传回 WebGAL？

在你的 `1.html` 的 JavaScript 代码中，当玩家通关或点击退出时，需要调用 WebGAL 提供的桥接接口（Bridge）把数据传回来并关闭窗口：

```javascript
// 假设这是 1.html 里的通关触发函数
function onGameWin() {
    // 检查并调用 WebGAL 的 iframe 关闭和返回值接口
    if (window.parent && window.parent.WebGAL) {
        // 将结果返回给 WebGAL，并关闭 iframe
        window.parent.WebGAL.closeIframe('prismGame', 'success'); 
    }
}

```

#### 2. 主剧本如何接收这个返回值？

当 iframe 关闭后，`createIframe` 收集到的返回值会存入你指定的变量中（例如上面设置的 `result`）。你可以在剧本的下一行直接进行条件判断：

```webgal
createIframe:1.html -id=prismGame -width=100% -height=100% -wait -returnValue=result;

; 根据小游戏传回的返回值走不同分支
if:result=='success';
系统: 恭喜你通关了小游戏！
jumpLabel:next_chapter;
endif;

if:result!='success';
系统: 你似乎中途退出了游戏。
endif;

```

---

### 四、 开发与调试的避坑指南

1. **路径问题**：
`1.html` 必须放置在项目可以被正确索引的路径下。如果报错 404，通常是路径没有对准项目根目录的静态资源文件夹。
2. **防残留（扫雷机制）**：
每次进入加载 iframe 的场景前，**务必在第一行写上 `removeIframe:prismGame;**`。因为如果玩家中途强行读档或跳过，极易导致浏览器的 DOM 里残留一个隐形的 iframe 遮挡鼠标，造成后续剧情“无法点击”。
3. **样式撑满**：
如果你做的是全屏小游戏（如三棱镜、拼图等），记得在 `1.html` 的 CSS 里面把 `body` 和根节点的宽高设为 `margin: 0; width: 100vw; height: 100vh; overflow: hidden;`，否则可能会带有一圈白色的默认边框。
### 五、 如何将自定义的 WebGAL 引擎导入编辑器

如果你已经将自定义引擎（例如 `WebGal-CustomIframeEngine`）打包为了完整的文件夹，可以通过以下步骤将其导入到 WebGAL Terre 编辑器中：

1. **一键打开用户数据目录**：
* 打开 WebGAL Terre 编辑器右上角的 **“设置”** 弹窗。
* 在下方的“用户数据”区域，直接点击 **“打开用户数据目录”** 按钮（正如你截图中所展示的那样）。


2. **创建并放入引擎文件夹**：
* 在弹出的文件夹窗口中（即 `C:\Users\FengM\.webgal_terre` 目录），检查是否存在名为 **`derivative-engines`** 的文件夹；如果没有，请手动创建一个。
* 进入 `derivative-engines` 文件夹，将你打包好的 `WebGal-CustomIframeEngine` 完整文件夹放进去。


3. **在编辑器中应用**：
* 彻底关闭并**重新启动** WebGAL Terre 编辑器。
* 打开你的游戏项目，在左侧或顶部的模板/项目配置中，选择你的 **`WebGal-CustomIframeEngine`** 并点击应用即可生效。