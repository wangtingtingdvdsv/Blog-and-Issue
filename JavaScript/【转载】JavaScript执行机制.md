原文作者：ssssyoki
转载自：[原文出处](https://juejin.im/post/59e85eebf265da430d571f89)
<article data-v-04f218ef="" itemscope="itemscope" itemtype="http://schema.org/Article" class="article" data-v-1be03d82=""><meta itemprop="url" content="https://juejin.im/post/59e85eebf265da430d571f89"><meta itemprop="headline" content="这一次，彻底弄懂 JavaScript 执行机制"><meta itemprop="keywords" content="JavaScript"><meta itemprop="datePublished" content="2017-11-21T08:12:59.282Z"><meta itemprop="image" content="https://user-gold-cdn.xitu.io/2017/11/21/15fdd9dfc3293a5c?w=810&amp;h=389&amp;f=png&amp;s=177029"><div itemprop="author" itemscope="itemscope" itemtype="http://schema.org/Person"><meta itemprop="name" content="ssssyoki"><meta itemprop="url" content="https://juejin.im/user/57a999616be3ff00654e34cc"></div><div itemprop="publisher" itemscope="itemscope" itemtype="http://schema.org/Organization"><meta itemprop="name" content="掘金"><div itemprop="logo" itemscope="itemscope" itemtype="https://schema.org/ImageObject"><meta itemprop="url" content="https://b-gold-cdn.xitu.io/icon/icon-white-180.png"><meta itemprop="width" content="180"><meta itemprop="height" content="180"></div></div><div data-v-04f218ef="" class="author-info-block"><a data-v-04f218ef="" href="/user/57a999616be3ff00654e34cc" target="_blank" rel="" class="avatar-link"><div data-v-0c0b16c6="" data-v-d9afc24c="" data-v-04f218ef="" data-src="https://user-gold-cdn.xitu.io/2018/1/21/161181a099330fe4?imageView2/1/w/100/h/100/q/85/format/webp/interlace/1" class="lazy avatar avatar loaded" style="background-image: url(&quot;https://user-gold-cdn.xitu.io/2018/1/21/161181a099330fe4?imageView2/1/w/100/h/100/q/85/format/webp/interlace/1&quot;);"></div></a><div data-v-04f218ef="" class="author-info-box"><a data-v-04f218ef="" href="/user/57a999616be3ff00654e34cc" target="_blank" rel="" class="username ellipsis"><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">ssssyoki</font></font></a><div data-v-04f218ef="" class="meta-box"><time data-v-04f218ef="" datetime="2017-11-21T08:12:59.282Z" title="Tue Nov 21 2017 16:12:59 GMT+0800 (中国标准时间)" class="time"><font style="vertical-align: inherit;"><font style="vertical-align: inherit;"></font></font></time><span data-v-04f218ef="" class="views-count"><font style="vertical-align: inherit;"><font style="vertical-align: inherit;"></font></font></span><!----></div></div><button data-v-ddc167a0="" data-v-04f218ef="" class="follow-button follow"><font style="vertical-align: inherit;"><font style="vertical-align: inherit;"></font></font></button></div><div data-v-0c0b16c6="" data-v-6e758c9a="" data-v-04f218ef="" data-src="https://user-gold-cdn.xitu.io/2017/11/21/15fdd9dfc3293a5c?imageView2/1/w/1304/h/734/q/85/format/webp/interlace/1" class="lazy thumb article-hero loaded" style="background-image: url(&quot;https://user-gold-cdn.xitu.io/2017/11/21/15fdd9dfc3293a5c?imageView2/1/w/1304/h/734/q/85/format/webp/interlace/1&quot;); background-size: cover;"></div><h1 data-v-04f218ef="" class="article-title"><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">这一次，彻底弄懂JavaScript 执行机制</font></font></h1><div data-v-04f218ef="" data-id="5a13e00bf265da432b4a729f" itemprop="articleBody" class="article-content"><p><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">本文的目的就是要保证你彻底弄懂javascript的执行机制，如果读完本文还不懂，可以揍我。</font></font></p>

<p><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">不论你是javascript新手还是老鸟，不论是面试求职，还是日常开发工作，我们经常会遇到这样的情况：给定的几行代码，我们需要知道其输出内容和顺序。</font><font style="vertical-align: inherit;">因为javascript是一门单线程语言，所以我们可以得出结论：</font></font></p>
<ul>
<li><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">javascript是按照语句出现的顺序执行的</font></font></li>
</ul>
<p><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">看到这里读者要打人了：我难道不知道js是一行一行执行的？</font><font style="vertical-align: inherit;">还用你说？</font><font style="vertical-align: inherit;">稍安勿躁，正因为js是一行一行执行的，所以我们以为js都是这样的：</font></font></p>
<pre><code class="hljs javascript copyable" lang="JavaScript"><span class="hljs-keyword">let</span> a = <span class="hljs-string">'1'</span>;
<span class="hljs-built_in">console</span>.log(a);<font></font>
<font></font>
<span class="hljs-keyword">let</span> b = <span class="hljs-string">'2'</span>;
<span class="hljs-built_in">console</span>.log(b);<span class="copy-code-btn">复制代码</span></code></pre><p></p><figure><img alt="" class="lazyload inited loaded" data-src="https://user-gold-cdn.xitu.io/2017/11/20/15fd87f7221d0dbe?imageView2/0/w/1280/h/960/format/webp/ignore-error/1" data-width="198" data-height="135" src="https://user-gold-cdn.xitu.io/2017/11/20/15fd87f7221d0dbe?imageView2/0/w/1280/h/960/format/webp/ignore-error/1"><figcaption></figcaption></figure><p></p>
<p><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">然而实际上js是这样的：</font></font></p>
<pre><code class="hljs javascript copyable" lang="JavaScript">setTimeout(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>)</span>{
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'定时器开始啦'</span>)<font></font>
});<font></font>
<font></font>
<span class="hljs-keyword">new</span> <span class="hljs-built_in">Promise</span>(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">resolve</span>)</span>{
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'马上执行for循环啦'</span>);
    <span class="hljs-keyword">for</span>(<span class="hljs-keyword">var</span> i = <span class="hljs-number">0</span>; i &lt; <span class="hljs-number">10000</span>; i++){<font></font>
        i == <span class="hljs-number">99</span> &amp;&amp; resolve();<font></font>
    }<font></font>
}).then(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>)</span>{
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'执行then函数啦'</span>)<font></font>
});<font></font>
<font></font>
<span class="hljs-built_in">console</span>.log(<span class="hljs-string">'代码执行结束'</span>);<span class="copy-code-btn">复制代码</span></code></pre><p></p><figure><img alt="" class="lazyload inited" data-src="https://user-gold-cdn.xitu.io/2017/11/20/15fd87d38acc4905?imageView2/0/w/1280/h/960/format/webp/ignore-error/1" data-width="148" data-height="133" src="data:image/svg+xml;utf8,&lt;?xml version=&quot;1.0&quot;?&gt;&lt;svg xmlns=&quot;http://www.w3.org/2000/svg&quot; version=&quot;1.1&quot; width=&quot;148&quot; height=&quot;133&quot;&gt;&lt;/svg&gt;"><figcaption></figcaption></figure><p></p>
<p><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">依照</font></font><strong style="font-size:20px"><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">js是按照语句出现的顺序执行</font></font></strong><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">这个理念，我自信的写下输出结果：</font></font></p>
<pre><code class="hljs javascript copyable" lang="JavaScript"><span class="hljs-comment">//"定时器开始啦"</span>
<span class="hljs-comment">//"马上执行for循环啦"</span>
<span class="hljs-comment">//"执行then函数啦"</span>
<span class="hljs-comment">//"代码执行结束"</span><span class="copy-code-btn">复制代码</span></code></pre><p><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">去chrome上验证下，结果完全不对，瞬间懵了，说好的一行一行执行的呢？</font></font></p>
<p></p><figure><img alt="" class="lazyload inited" data-src="https://user-gold-cdn.xitu.io/2017/11/20/15fd8840f3c3f109?imageView2/0/w/1280/h/960/format/webp/ignore-error/1" data-width="250" data-height="250" src="data:image/svg+xml;utf8,&lt;?xml version=&quot;1.0&quot;?&gt;&lt;svg xmlns=&quot;http://www.w3.org/2000/svg&quot; version=&quot;1.1&quot; width=&quot;250&quot; height=&quot;250&quot;&gt;&lt;/svg&gt;"><figcaption></figcaption></figure><p></p>
<p><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">我们真的要彻底弄明白javascript的执行机制了。</font></font></p>
<h3 class="heading" data-id="heading-0"><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">1.关于javascript</font></font></h3><p><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">javascript是一门</font></font><strong style="font-size:20px"><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">单线程</font></font></strong><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">语言，在最新的HTML5中提出了Web-Worker，但javascript是单线程这一核心仍未改变。</font><font style="vertical-align: inherit;">所以一切javascript版的"多线程"都是用单线程模拟出来的，一切javascript多线程都是纸老虎！</font></font></p>
<h3 class="heading" data-id="heading-1"><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">2.javascript事件循环</font></font></h3><p><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">既然js是单线程，那就像只有一个窗口的银行，客户需要排队一个一个办理业务，同理js任务也要一个一个顺序执行。</font><font style="vertical-align: inherit;">如果一个任务耗时过长，那么后一个任务也必须等着。</font><font style="vertical-align: inherit;">那么问题来了，假如我们想浏览新闻，但是新闻包含的超清图片加载很慢，难道我们的网页要一直卡着直到图片完全显示出来？</font><font style="vertical-align: inherit;">因此聪明的程序员将任务分为两类：</font></font></p>
<ul>
<li><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">同步任务</font></font></li>
<li><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">异步任务</font></font></li>
</ul>
<p>当我们打开网站时，网页的渲染过程就是一大堆同步任务，比如页面骨架和页面元素的渲染。而像加载图片音乐之类占用资源大耗时久的任务，就是异步任务。关于这部分有严格的文字定义，但本文的目的是用最小的学习成本彻底弄懂执行机制，所以我们用导图来说明：</p>
<p></p><figure><img alt="" class="lazyload inited" data-src="https://user-gold-cdn.xitu.io/2017/11/21/15fdd88994142347?imageView2/0/w/1280/h/960/format/webp/ignore-error/1" data-width="1280" data-height="1070" src="data:image/svg+xml;utf8,&lt;?xml version=&quot;1.0&quot;?&gt;&lt;svg xmlns=&quot;http://www.w3.org/2000/svg&quot; version=&quot;1.1&quot; width=&quot;1280&quot; height=&quot;1070&quot;&gt;&lt;/svg&gt;"><figcaption></figcaption></figure><p></p>
<p>导图要表达的内容用文字来表述的话：</p>
<ul>
<li>同步和异步任务分别进入不同的执行"场所"，同步的进入主线程，异步的进入Event Table并注册函数。</li>
<li>当指定的事情完成时，Event Table会将这个函数移入Event Queue。</li>
<li>主线程内的任务执行完毕为空，会去Event Queue读取对应的函数，进入主线程执行。</li>
<li>上述过程会不断重复，也就是常说的Event Loop(事件循环)。</li>
</ul>
<p>我们不禁要问了，那怎么知道主线程执行栈为空啊？js引擎存在monitoring process进程，会持续不断的检查主线程执行栈是否为空，一旦为空，就会去Event Queue那里检查是否有等待被调用的函数。</p>
<p>说了这么多文字，不如直接一段代码更直白：</p>
<pre><code class="hljs javascript copyable" lang="JavaScript"><span class="hljs-keyword">let</span> data = [];<font></font>
$.ajax({<font></font>
    <span class="hljs-attr">url</span>:www.javascript.com,
    <span class="hljs-attr">data</span>:data,
    <span class="hljs-attr">success</span>:<span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
        <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'发送成功!'</span>);<font></font>
    }<font></font>
})<font></font>
<span class="hljs-built_in">console</span>.log(<span class="hljs-string">'代码执行结束'</span>);<span class="copy-code-btn">复制代码</span></code></pre><p>上面是一段简易的<code>ajax</code>请求代码：</p>
<ul>
<li>ajax进入Event Table，注册回调函数<code>success</code>。</li>
<li>执行<code>console.log('代码执行结束')</code>。</li>
<li>ajax事件完成，回调函数<code>success</code>进入Event Queue。</li>
<li>主线程从Event Queue读取回调函数<code>success</code>并执行。</li>
</ul>
<p>相信通过上面的文字和代码，你已经对js的执行顺序有了初步了解。接下来我们来研究进阶话题：setTimeout。</p>
<h3 class="heading" data-id="heading-2">3.又爱又恨的setTimeout</h3><p>大名鼎鼎的<code>setTimeout</code>无需再多言，大家对他的第一印象就是异步可以延时执行，我们经常这么实现延时3秒执行：</p>
<pre><code class="hljs javascript copyable" lang="JavaScript">setTimeout(<span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'延时3秒'</span>);<font></font>
},<span class="hljs-number">3000</span>)<span class="copy-code-btn">复制代码</span></code></pre><p>渐渐的<code>setTimeout</code>用的地方多了，问题也出现了，有时候明明写的延时3秒，实际却5，6秒才执行函数，这又咋回事啊？</p>
<p>先看一个例子：</p>
<pre><code class="hljs javascript copyable" lang="JavaScript">setTimeout(<span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {<font></font>
    task();<font></font>
},<span class="hljs-number">3000</span>)
<span class="hljs-built_in">console</span>.log(<span class="hljs-string">'执行console'</span>);<span class="copy-code-btn">复制代码</span></code></pre><p>根据前面我们的结论，<code>setTimeout</code>是异步的，应该先执行<code>console.log</code>这个同步任务，所以我们的结论是：</p>
<pre><code class="hljs javascript copyable" lang="JavaScript"><span class="hljs-comment">//执行console</span>
<span class="hljs-comment">//task()</span><span class="copy-code-btn">复制代码</span></code></pre><p>去验证一下，结果正确！<br>然后我们修改一下前面的代码：</p>
<pre><code class="hljs javascript copyable" lang="JavaScript">setTimeout(<span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {<font></font>
    task()<font></font>
},<span class="hljs-number">3000</span>)<font></font>
<font></font>
sleep(<span class="hljs-number">10000000</span>)<span class="copy-code-btn">复制代码</span></code></pre><p>乍一看其实差不多嘛，但我们把这段代码在chrome执行一下，却发现控制台执行<code>task()</code>需要的时间远远超过3秒，说好的延时三秒，为啥现在需要这么长时间啊？</p>
<p>这时候我们需要重新理解<code>setTimeout</code>的定义。我们先说上述代码是怎么执行的：</p>
<ul>
<li><code>task()</code>进入Event Table并注册,计时开始。</li>
<li>执行<code>sleep</code>函数，很慢，非常慢，计时仍在继续。</li>
<li>3秒到了，计时事件<code>timeout</code>完成，<code>task()</code>进入Event Queue，但是<code>sleep</code>也太慢了吧，还没执行完，只好等着。</li>
<li><code>sleep</code>终于执行完了，<code>task()</code>终于从Event Queue进入了主线程执行。</li>
</ul>
<p>上述的流程走完，我们知道<code>setTimeout</code>这个函数，是经过指定时间后，把要执行的任务(本例中为<code>task()</code>)加入到Event Queue中，又因为是单线程任务要一个一个执行，如果前面的任务需要的时间太久，那么只能等着，导致真正的延迟时间远远大于3秒。</p>
<p>我们还经常遇到<code>setTimeout(fn,0)</code>这样的代码，0秒后执行又是什么意思呢？是不是可以立即执行呢？</p>
<p>答案是不会的，<code>setTimeout(fn,0)</code>的含义是，指定某个任务在主线程最早可得的空闲时间执行，意思就是不用再等多少秒了，只要主线程执行栈内的同步任务全部执行完成，栈为空就马上执行。举例说明：</p>
<pre><code class="hljs javascript copyable" lang="JavaScript"><span class="hljs-comment">//代码1</span>
<span class="hljs-built_in">console</span>.log(<span class="hljs-string">'先执行这里'</span>);<font></font>
setTimeout(<span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'执行啦'</span>)<font></font>
},<span class="hljs-number">0</span>);<span class="copy-code-btn">复制代码</span></code></pre><pre><code class="hljs javascript copyable" lang="JavaScript"><span class="hljs-comment">//代码2</span>
<span class="hljs-built_in">console</span>.log(<span class="hljs-string">'先执行这里'</span>);<font></font>
setTimeout(<span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'执行啦'</span>)<font></font>
},<span class="hljs-number">3000</span>);<span class="copy-code-btn">复制代码</span></code></pre><p>代码1的输出结果是：</p>
<pre><code class="hljs javascript copyable" lang="JavaScript"><span class="hljs-comment">//先执行这里</span>
<span class="hljs-comment">//执行啦</span><span class="copy-code-btn">复制代码</span></code></pre><p>代码2的输出结果是：</p>
<pre><code class="hljs javascript copyable" lang="JavaScript"><span class="hljs-comment">//先执行这里</span>
<span class="hljs-comment">// ... 3s later</span>
<span class="hljs-comment">// 执行啦</span><span class="copy-code-btn">复制代码</span></code></pre><p>关于<code>setTimeout</code>要补充的是，即便主线程为空，0毫秒实际上也是达不到的。根据HTML的标准，最低是4毫秒。有兴趣的同学可以自行了解。</p>
<h3 class="heading" data-id="heading-3">4.又恨又爱的setInterval</h3><p>上面说完了<code>setTimeout</code>，当然不能错过它的孪生兄弟<code>setInterval</code>。他俩差不多，只不过后者是循环的执行。对于执行顺序来说，<code>setInterval</code>会每隔指定的时间将注册的函数置入Event Queue，如果前面的任务耗时太久，那么同样需要等待。</p>
<p>唯一需要注意的一点是，对于<code>setInterval(fn,ms)</code>来说，我们已经知道不是每过<code>ms</code>秒会执行一次<code>fn</code>，而是每过<code>ms</code>秒，会有<code>fn</code>进入Event Queue。一旦<strong style="font-size:15px"><code>setInterval</code>的回调函数<code>fn</code>执行时间超过了延迟时间<code>ms</code>，那么就完全看不出来有时间间隔了</strong>。这句话请读者仔细品味。</p>
<h3 class="heading" data-id="heading-4">5.Promise与process.nextTick(callback)</h3><p>传统的定时器我们已经研究过了，接着我们探究<code>Promise</code>与<code>process.nextTick(callback)</code>的表现。</p>
<p><code>Promise</code>的定义和功能本文不再赘述，不了解的读者可以学习一下阮一峰老师的<a href="https://link.juejin.im?target=http%3A%2F%2Fes6.ruanyifeng.com%2F%23docs%2Fpromise" target="_blank" rel="nofollow noopener noreferrer">Promise</a>。而<code>process.nextTick(callback)</code>类似node.js版的"setTimeout"，在事件循环的下一次循环中调用 callback 回调函数。</p>
<p>我们进入正题，除了广义的同步任务和异步任务，我们对任务有更精细的定义：</p>
<ul>
<li>macro-task(宏任务)：包括整体代码script，setTimeout，setInterval</li>
<li>micro-task(微任务)：Promise，process.nextTick</li>
</ul>
<p>不同类型的任务会进入对应的Event Queue，比如<code>setTimeout</code>和<code>setInterval</code>会进入相同的Event Queue。</p>
<p>事件循环的顺序，决定js代码的执行顺序。进入整体代码(宏任务)后，开始第一次循环。接着执行所有的微任务。然后再次从宏任务开始，找到其中一个任务队列执行完毕，再执行所有的微任务。听起来有点绕，我们用文章最开始的一段代码说明：</p>
<pre><code class="hljs javascript copyable" lang="JavaScript">setTimeout(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>) </span>{
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'setTimeout'</span>);<font></font>
})<font></font>
<font></font>
<span class="hljs-keyword">new</span> <span class="hljs-built_in">Promise</span>(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">resolve</span>) </span>{
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'promise'</span>);<font></font>
}).then(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>) </span>{
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'then'</span>);<font></font>
})<font></font>
<font></font>
<span class="hljs-built_in">console</span>.log(<span class="hljs-string">'console'</span>);<span class="copy-code-btn">复制代码</span></code></pre><ul>
<li>这段代码作为宏任务，进入主线程。</li>
<li>先遇到<code>setTimeout</code>，那么将其回调函数注册后分发到宏任务Event Queue。(注册过程与上同，下文不再描述)</li>
<li>接下来遇到了<code>Promise</code>，<code>new Promise</code>立即执行，<code>then</code>函数分发到微任务Event Queue。</li>
<li>遇到<code>console.log()</code>，立即执行。</li>
<li>好啦，整体代码script作为第一个宏任务执行结束，看看有哪些微任务？我们发现了<code>then</code>在微任务Event Queue里面，执行。</li>
<li>ok，第一轮事件循环结束了，我们开始第二轮循环，当然要从宏任务Event Queue开始。我们发现了宏任务Event Queue中<code>setTimeout</code>对应的回调函数，立即执行。</li>
<li>结束。</li>
</ul>
<p>事件循环，宏任务，微任务的关系如图所示：</p>
<p></p><figure><img alt="" class="lazyload inited" data-src="https://user-gold-cdn.xitu.io/2017/11/21/15fdcea13361a1ec?imageView2/0/w/1280/h/960/format/webp/ignore-error/1" data-width="1280" data-height="1072" src="data:image/svg+xml;utf8,&lt;?xml version=&quot;1.0&quot;?&gt;&lt;svg xmlns=&quot;http://www.w3.org/2000/svg&quot; version=&quot;1.1&quot; width=&quot;1280&quot; height=&quot;1072&quot;&gt;&lt;/svg&gt;"><figcaption></figcaption></figure><p></p>
<p>我们来分析一段较复杂的代码，看看你是否真的掌握了js的执行机制：</p>
<pre><code class="hljs javascript copyable" lang="JavaScript"><span class="hljs-built_in">console</span>.log(<span class="hljs-string">'1'</span>);<font></font>
<font></font>
setTimeout(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>) </span>{
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'2'</span>);<font></font>
    process.nextTick(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>) </span>{
        <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'3'</span>);<font></font>
    })<font></font>
    <span class="hljs-keyword">new</span> <span class="hljs-built_in">Promise</span>(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">resolve</span>) </span>{
        <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'4'</span>);<font></font>
        resolve();<font></font>
    }).then(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>) </span>{
        <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'5'</span>)<font></font>
    })<font></font>
})<font></font>
process.nextTick(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>) </span>{
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'6'</span>);<font></font>
})<font></font>
<span class="hljs-keyword">new</span> <span class="hljs-built_in">Promise</span>(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">resolve</span>) </span>{
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'7'</span>);<font></font>
    resolve();<font></font>
}).then(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>) </span>{
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'8'</span>)<font></font>
})<font></font>
<font></font>
setTimeout(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>) </span>{
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'9'</span>);<font></font>
    process.nextTick(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>) </span>{
        <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'10'</span>);<font></font>
    })<font></font>
    <span class="hljs-keyword">new</span> <span class="hljs-built_in">Promise</span>(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">resolve</span>) </span>{
        <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'11'</span>);<font></font>
        resolve();<font></font>
    }).then(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>) </span>{
        <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'12'</span>)<font></font>
    })<font></font>
})<span class="copy-code-btn">复制代码</span></code></pre><p>第一轮事件循环流程分析如下：</p>
<ul>
<li>整体script作为第一个宏任务进入主线程，遇到<code>console.log</code>，输出1。</li>
<li>遇到<code>setTimeout</code>，其回调函数被分发到宏任务Event Queue中。我们暂且记为<code>setTimeout1</code>。</li>
<li>遇到<code>process.nextTick()</code>，其回调函数被分发到微任务Event Queue中。我们记为<code>process1</code>。</li>
<li>遇到<code>Promise</code>，<code>new Promise</code>直接执行，输出7。<code>then</code>被分发到微任务Event Queue中。我们记为<code>then1</code>。</li>
<li>又遇到了<code>setTimeout</code>，其回调函数被分发到宏任务Event Queue中，我们记为<code>setTimeout2</code>。</li>
</ul>
<table>
<thead>
<tr>
<th style="text-align:center">宏任务Event Queue</th>
<th style="text-align:center">微任务Event Queue</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:center">setTimeout1</td>
<td style="text-align:center">process1</td>
</tr>
<tr>
<td style="text-align:center">setTimeout2</td>
<td style="text-align:center">then1</td>
</tr>
</tbody>
</table>
<ul>
<li><p>上表是第一轮事件循环宏任务结束时各Event Queue的情况，此时已经输出了1和7。</p>
</li>
<li><p>我们发现了<code>process1</code>和<code>then1</code>两个微任务。</p>
</li>
<li>执行<code>process1</code>,输出6。</li>
<li>执行<code>then1</code>，输出8。</li>
</ul>
<p>好了，第一轮事件循环正式结束，这一轮的结果是输出1，7，6，8。那么第二轮时间循环从<code>setTimeout1</code>宏任务开始：</p>
<ul>
<li>首先输出2。接下来遇到了<code>process.nextTick()</code>，同样将其分发到微任务Event Queue中，记为<code>process2</code>。<code>new Promise</code>立即执行输出4，<code>then</code>也分发到微任务Event Queue中，记为<code>then2</code>。</li>
</ul>
<table>
<thead>
<tr>
<th style="text-align:center">宏任务Event Queue</th>
<th style="text-align:center">微任务Event Queue</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:center">setTimeout2</td>
<td style="text-align:center">process2</td>
</tr>
<tr>
<td style="text-align:center"></td>
<td style="text-align:center">then2</td>
</tr>
</tbody>
</table>
<ul>
<li>第二轮事件循环宏任务结束，我们发现有<code>process2</code>和<code>then2</code>两个微任务可以执行。</li>
<li>输出3。</li>
<li>输出5。</li>
<li>第二轮事件循环结束，第二轮输出2，4，3，5。</li>
<li>第三轮事件循环开始，此时只剩setTimeout2了，执行。</li>
<li>直接输出9。</li>
<li>将<code>process.nextTick()</code>分发到微任务Event Queue中。记为<code>process3</code>。</li>
<li>直接执行<code>new Promise</code>，输出11。</li>
<li>将<code>then</code>分发到微任务Event Queue中，记为<code>then3</code>。</li>
</ul>
<table>
<thead>
<tr>
<th style="text-align:center">宏任务Event Queue</th>
<th style="text-align:center">微任务Event Queue</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:center"></td>
<td style="text-align:center">process3</td>
</tr>
<tr>
<td style="text-align:center"></td>
<td style="text-align:center">then3</td>
</tr>
</tbody>
</table>
<ul>
<li>第三轮事件循环宏任务执行结束，执行两个微任务<code>process3</code>和<code>then3</code>。</li>
<li>输出10。</li>
<li>输出12。</li>
<li>第三轮事件循环结束，第三轮输出9，11，10，12。</li>
</ul>
<p>整段代码，共进行了三次事件循环，完整的输出为1，7，6，8，2，4，3，5，9，11，10，12。<br>(请注意，node环境下的事件监听依赖libuv与前端环境不完全相同，输出顺序可能会有误差)</p>
<h3 class="heading" data-id="heading-5">6.写在最后</h3><h4 class="heading" data-id="heading-6">(1)js的异步</h4><p>我们从最开头就说javascript是一门单线程语言，不管是什么新框架新语法糖实现的所谓异步，其实都是用同步的方法去模拟的，牢牢把握住单线程这点非常重要。</p>
<h4 class="heading" data-id="heading-7">(2)事件循环Event Loop</h4><p>事件循环是js实现异步的一种方法，也是js的执行机制。</p>
<h4 class="heading" data-id="heading-8">(3)javascript的执行和运行</h4><p>执行和运行有很大的区别，javascript在不同的环境下，比如node，浏览器，Ringo等等，执行方式是不同的。而运行大多指javascript解析引擎，是统一的。</p>
<h4 class="heading" data-id="heading-9">(4)setImmediate</h4><p>微任务和宏任务还有很多种类，比如<code>setImmediate</code>等等，执行都是有共同点的，有兴趣的同学可以自行了解。</p>
<h4 class="heading" data-id="heading-10">(5)最后的最后</h4><ul>
<li>javascript是一门单线程语言</li>
<li>Event Loop是javascript的执行机制</li>
</ul>
<p>牢牢把握两个基本点，以认真学习javascript为中心，早日实现成为前端高手的伟大梦想！</p>
<p></p><figure><img alt="" class="lazyload inited" data-src="https://user-gold-cdn.xitu.io/2017/11/21/15fdd96beade6575?imageView2/0/w/1280/h/960/format/webp/ignore-error/1" data-width="314" data-height="286" src="data:image/svg+xml;utf8,&lt;?xml version=&quot;1.0&quot;?&gt;&lt;svg xmlns=&quot;http://www.w3.org/2000/svg&quot; version=&quot;1.1&quot; width=&quot;314&quot; height=&quot;286&quot;&gt;&lt;/svg&gt;"><figcaption></figcaption></figure><p></p>
<blockquote>
<ul>
<li>联系邮箱：ssssyoki@foxmail.com </li>
<li>联系微信：the-UK  </li>
</ul>
</blockquote>
</div></article>
