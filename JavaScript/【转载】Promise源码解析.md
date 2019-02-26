转载自：[原文](https://juejin.im/post/5b83cb5ae51d4538cc3ec354)

作者：面条__

<article class="article" data-v-04f218ef="" data-v-1be03d82="" itemtype="http://schema.org/Article" itemscope="itemscope"><meta content="https://juejin.im/post/5b83cb5ae51d4538cc3ec354" itemprop="url"><meta content="Promise实现原理（附源码）" itemprop="headline"><meta content="JavaScript,前端,Promise" itemprop="keywords"><meta content="https://b-gold-cdn.xitu.io/icon/icon-128.png" itemprop="image"><div itemtype="http://schema.org/Person" itemscope="itemscope" itemprop="author"><meta content="面条__" itemprop="name"><meta content="https://juejin.im/user/5a094891f265da4326525623" itemprop="url"></div><div itemtype="http://schema.org/Organization" itemscope="itemscope" itemprop="publisher"><meta content="掘金" itemprop="name"><div itemtype="https://schema.org/ImageObject" itemscope="itemscope" itemprop="logo"><meta content="https://b-gold-cdn.xitu.io/icon/icon-white-180.png" itemprop="url"><meta content="180" itemprop="width"><meta content="180" itemprop="height"></div></div><div class="author-info-block" data-v-04f218ef=""><a class="avatar-link" href="/user/5a094891f265da4326525623" target="_blank" rel="" data-v-04f218ef=""><div class="lazy avatar avatar loaded" style='background-image: url("https://user-gold-cdn.xitu.io/2018/8/23/165641cc247d143e?imageView2/1/w/100/h/100/q/85/interlace/1");' data-v-04f218ef="" data-v-0c0b16c6="" data-v-d9afc24c="" data-src="https://user-gold-cdn.xitu.io/2018/8/23/165641cc247d143e?imageView2/1/w/100/h/100/q/85/interlace/1"></div></a><div class="author-info-box" data-v-04f218ef=""><a class="username ellipsis" href="/user/5a094891f265da4326525623" target="_blank" rel="" data-v-04f218ef=""></a><div class="meta-box" data-v-04f218ef=""><time title="Mon Aug 27 2018 17:59:20 GMT+0800 (中国标准时间)" class="time" data-v-04f218ef="" datetime="2018-08-27T09:59:20.566Z"></time><span class="views-count" data-v-04f218ef=""></span><!----></div></div><button class="follow-button follow" data-v-04f218ef="" data-v-ddc167a0=""></button></div><!----><h1 class="article-title" data-v-04f218ef="">Promise实现原理（附源码）</h1><div class="article-content" data-v-04f218ef="" itemprop="articleBody" data-id="5b83cb786fb9a019fb516237"><blockquote>
<p>本篇文章主要在于探究 <code>Promise</code> 的实现原理，带领大家一步一步实现一个 <code>Promise</code> , 不对其用法做说明，如果读者还对Promise的用法不了解，可以查看阮一峰老师的<a href="https://link.juejin.im?target=http%3A%2F%2Fes6.ruanyifeng.com%2F%23docs%2Fpromise" target="_blank" rel="nofollow noopener noreferrer">ES6 Promise教程</a>。</p>
</blockquote>
<p>接下来，带你一步一步实现一个 <code>Promise</code></p>
<h2 class="heading" data-id="heading-0">1. <code>Promise</code> 基本结构</h2>
<pre><code class="hljs javascript copyable" lang="javascript"><span class="hljs-keyword">new</span> <span class="hljs-built_in">Promise</span>(<span class="hljs-function">(<span class="hljs-params">resolve, reject</span>) =&gt;</span> {
  setTimeout(<span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
    resolve(<span class="hljs-string">'FULFILLED'</span>)
  }, <span class="hljs-number">1000</span>)
})
<span class="copy-code-btn">复制代码</span></code></pre><blockquote>
<p>构造函数<code>Promise</code>必须接受一个函数作为参数，我们称该函数为<code>handle</code>，<code>handle</code>又包含<code>resolve</code>和<code>reject</code>两个参数，它们是两个函数。</p>
</blockquote>
<p>定义一个判断一个变量是否为函数的方法，后面会用到</p>
<pre><code class="hljs javascript copyable" lang="javascript"><span class="hljs-comment">// 判断变量否为function</span>
<span class="hljs-keyword">const</span> isFunction = <span class="hljs-function"><span class="hljs-params">variable</span> =&gt;</span> <span class="hljs-keyword">typeof</span> variable === <span class="hljs-string">'function'</span>
<span class="copy-code-btn">复制代码</span></code></pre><p>首先，我们定义一个名为 <code>MyPromise</code> 的 <code>Class</code>，它接受一个函数 <code>handle</code> 作为参数</p>
<pre><code class="hljs javascript copyable" lang="javascript"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MyPromise</span> </span>{
  <span class="hljs-keyword">constructor</span> (handle) {
    <span class="hljs-keyword">if</span> (!isFunction(handle)) {
      <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> <span class="hljs-built_in">Error</span>(<span class="hljs-string">'MyPromise must accept a function as a parameter'</span>)
    }
  }
}
<span class="copy-code-btn">复制代码</span></code></pre><p>再往下看</p>
<h2 class="heading" data-id="heading
<p><code>Promise</code> 对象存在以下三种状态：</p>
<ul>-1">2. <code>Promise</code> 状态和值</h2>
<li>
<p><code>Pending(进行中)</code></p>
</li>
<li>
<p><code>Fulfilled(已成功)</code></p>
</li>
<li>
<p><code>Rejected(已失败)</code></p>
</li>
</ul>
<blockquote>
<p>状态只能由 <code>Pending</code> 变为 <code>Fulfilled</code> 或由 <code>Pending</code> 变为 <code>Rejected</code> ，且状态改变之后不会在发生变化，会一直保持这个状态。</p>
</blockquote>
<p><code>Promise</code>的值是指状态改变时传递给回调函数的值</p>
<blockquote>
<p>上文中<code>handle</code>函数包含 <code>resolve</code> 和 <code>reject</code> 两个参数，它们是两个函数，可以用于改变 <code>Promise</code> 的状态和传入 <code>Promise</code> 的值</p>
</blockquote>
<pre><code class="hljs javascript copyable" lang="javascript"><span class="hljs-keyword">new</span> <span class="hljs-built_in">Promise</span>(<span class="hljs-function">(<span class="hljs-params">resolve, reject</span>) =&gt;</span> {
  setTimeout(<span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
    resolve(<span class="hljs-string">'FULFILLED'</span>)
  }, <span class="hljs-number">1000</span>)
})
<span class="copy-code-btn">复制代码</span></code></pre><p>这里 <code>resolve</code> 传入的 <code>"FULFILLED"</code> 就是 <code>Promise</code> 的值</p>
<p><code>resolve</code> 和 <code>reject</code></p>
<ul>
<li><code>resolve</code> : 将Promise对象的状态从 <code>Pending(进行中)</code> 变为 <code>Fulfilled(已成功)</code></li>
<li><code>reject</code> : 将Promise对象的状态从 <code>Pending(进行中)</code> 变为 <code>Rejected(已失败)</code></li>
<li><code>resolve</code> 和 <code>reject</code> 都可以传入任意类型的值作为实参，表示 <code>Promise</code> 对象成功<code>（Fulfilled）</code>和失败<code>（Rejected）</code>的值</li>
</ul>
<p>了解了 <code>Promise</code> 的状态和值，接下来，我们为 <code>MyPromise</code> 添加状态属性和值</p>
<blockquote>
<p>首先定义三个常量，用于标记Promise对象的三种状态</p>
</blockquote>
<pre><code class="hljs javascript copyable" lang="javascript"><span class="hljs-comment">// 定义Promise的三种状态常量</span>
<span class="hljs-keyword">const</span> PENDING = <span class="hljs-string">'PENDING'</span>
<span class="hljs-keyword">const</span> FULFILLED = <span class="hljs-string">'FULFILLED'</span>
<span class="hljs-keyword">const</span> REJECTED = <span class="hljs-string">'REJECTED'</span>
<span class="copy-code-btn">复制代码</span></code></pre><blockquote>
<p>再为 <code>MyPromise</code> 添加状态和值，并添加状态改变的执行逻辑</p>
</blockquote>
<pre><code class="hljs javascript copyable" lang="javascript"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MyPromise</span> </span>{
  <span class="hljs-keyword">constructor</span> (handle) {
    <span class="hljs-keyword">if</span> (!isFunction(handle)) {
      <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> <span class="hljs-built_in">Error</span>(<span class="hljs-string">'MyPromise must accept a function as a parameter'</span>)
    }
    <span class="hljs-comment">// 添加状态</span>
    <span class="hljs-keyword">this</span>._status = PENDING
    <span class="hljs-comment">// 添加状态</span>
    <span class="hljs-keyword">this</span>._value = <span class="hljs-literal">undefined</span>
    <span class="hljs-comment">// 执行handle</span>
    <span class="hljs-keyword">try</span> {
      handle(<span class="hljs-keyword">this</span>._resolve.bind(<span class="hljs-keyword">this</span>), <span class="hljs-keyword">this</span>._reject.bind(<span class="hljs-keyword">this</span>)) 
    } <span class="hljs-keyword">catch</span> (err) {
      <span class="hljs-keyword">this</span>._reject(err)
    }
  }
  <span class="hljs-comment">// 添加resovle时执行的函数</span>
  _resolve (val) {
    <span class="hljs-keyword">if</span> (<span class="hljs-keyword">this</span>._status !== PENDING) <span class="hljs-keyword">return</span>
    <span class="hljs-keyword">this</span>._status = FULFILLED
    <span class="hljs-keyword">this</span>._value = val
  }
  <span class="hljs-comment">// 添加reject时执行的函数</span>
  _reject (err) { 
    <span class="hljs-keyword">if</span> (<span class="hljs-keyword">this</span>._status !== PENDING) <span class="hljs-keyword">return</span>
    <span class="hljs-keyword">this</span>._status = REJECTED
    <span class="hljs-keyword">this</span>._value = err
  }
}
<span class="copy-code-btn">复制代码</span></code></pre><p>这样就实现了 <code>Promise</code> 状态和值的改变。下面说一说 <code>Promise</code> 的核心: <code>then</code> 方法</p>
<h2 class="heading" data-id="heading-2">3. <code>Promise</code> 的 <code>then</code> 方法</h2>
<p><code>Promise</code> 对象的 <code>then</code> 方法接受两个参数：</p>
<pre><code class="hljs javascript copyable" lang="javascript">promise.then(onFulfilled, onRejected)
<span class="copy-code-btn">复制代码</span></code></pre><p><strong>参数可选</strong></p>
<p><code>onFulfilled</code> 和 <code>onRejected</code> 都是可选参数。</p>
<ul>
<li>如果 <code>onFulfilled</code> 或 <code>onRejected</code> 不是函数，其必须被忽略</li>
</ul>
<p><strong><code>onFulfilled</code> 特性</strong></p>
<p>    如果 <code>onFulfilled</code> 是函数：</p>
<ul>
<li>当 <code>promise</code> 状态变为成功时必须被调用，其第一个参数为 <code>promise</code> 成功状态传入的值（ <code>resolve</code> 执行时传入的值）</li>
<li>在 <code>promise</code> 状态改变前其不可被调用</li>
<li>其调用次数不可超过一次</li>
</ul>
<p><strong><code>onRejected</code> 特性</strong></p>
<p>    如果 <code>onRejected</code> 是函数：</p>
<ul>
<li>当 <code>promise</code> 状态变为失败时必须被调用，其第一个参数为 <code>promise</code> 失败状态传入的值（ <code>reject</code> 执行时传入的值）</li>
<li>在 <code>promise</code> 状态改变前其不可被调用</li>
<li>其调用次数不可超过一次</li>
</ul>
<p><strong>多次调用</strong></p>
<p>    <code>then</code> 方法可以被同一个 <code>promise</code> 对象调用多次</p>
<ul>
<li>当 <code>promise</code> 成功状态时，所有 <code>onFulfilled</code> 需按照其注册顺序依次回调</li>
<li>当 <code>promise</code> 失败状态时，所有 <code>onRejected</code> 需按照其注册顺序依次回调</li>
</ul>
<p><strong>返回</strong></p>
<p><code>then</code> 方法必须返回一个新的 <code>promise</code> 对象</p>
<pre><code class="hljs javascript copyable" lang="javascript">promise2 = promise1.then(onFulfilled, onRejected);
<span class="copy-code-btn">复制代码</span></code></pre><p>因此 <code>promise</code> 支持链式调用</p>
<pre><code class="hljs javascript copyable" lang="javascript">promise1.then(onFulfilled1, onRejected1).then(onFulfilled2, onRejected2);
<span class="copy-code-btn">复制代码</span></code></pre><p>这里涉及到 <code>Promise</code> 的执行规则，包括“值的传递”和“错误捕获”机制：</p>
<p>1、如果 <code>onFulfilled</code> 或者 <code>onRejected</code> 返回一个值 <code>x</code> ，则运行下面的 <code>Promise</code> 解决过程：<code>[[Resolve]](promise2, x)</code></p>
<ul>
<li>若 <code>x</code> 不为 <code>Promise</code> ，则使 <code>x</code> 直接作为新返回的 <code>Promise</code> 对象的值， 即新的<code>onFulfilled</code> 或者 <code>onRejected</code> 函数的参数.</li>
<li>若 <code>x</code> 为 <code>Promise</code> ，这时后一个回调函数，就会等待该 <code>Promise</code> 对象(即 <code>x</code> )的状态发生变化，才会被调用，并且新的 <code>Promise</code> 状态和 <code>x</code> 的状态相同。</li>
</ul>
<p>下面的例子用于帮助理解：</p>
<pre><code class="hljs javascript copyable" lang="javascript"><span class="hljs-keyword">let</span> promise1 = <span class="hljs-keyword">new</span> <span class="hljs-built_in">Promise</span>(<span class="hljs-function">(<span class="hljs-params">resolve, reject</span>) =&gt;</span> {
  setTimeout(<span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
    resolve()
  }, <span class="hljs-number">1000</span>)
})
promise2 = promise1.then(<span class="hljs-function"><span class="hljs-params">res</span> =&gt;</span> {
  <span class="hljs-comment">// 返回一个普通值</span>
  <span class="hljs-keyword">return</span> <span class="hljs-string">'这里返回一个普通值'</span>
})
promise2.then(<span class="hljs-function"><span class="hljs-params">res</span> =&gt;</span> {
  <span class="hljs-built_in">console</span>.log(res) <span class="hljs-comment">//1秒后打印出：这里返回一个普通值</span>
})
<span class="copy-code-btn">复制代码</span></code></pre><pre><code class="hljs javascript copyable" lang="javascript"><span class="hljs-keyword">let</span> promise1 = <span class="hljs-keyword">new</span> <span class="hljs-built_in">Promise</span>(<span class="hljs-function">(<span class="hljs-params">resolve, reject</span>) =&gt;</span> {
  setTimeout(<span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
    resolve()
  }, <span class="hljs-number">1000</span>)
})
promise2 = promise1.then(<span class="hljs-function"><span class="hljs-params">res</span> =&gt;</span> {
  <span class="hljs-comment">// 返回一个Promise对象</span>
  <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> <span class="hljs-built_in">Promise</span>(<span class="hljs-function">(<span class="hljs-params">resolve, reject</span>) =&gt;</span> {
    setTimeout(<span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
     resolve(<span class="hljs-string">'这里返回一个Promise'</span>)
    }, <span class="hljs-number">2000</span>)
  })
})
promise2.then(<span class="hljs-function"><span class="hljs-params">res</span> =&gt;</span> {
  <span class="hljs-built_in">console</span>.log(res) <span class="hljs-comment">//3秒后打印出：这里返回一个Promise</span>
})
<span class="copy-code-btn">复制代码</span></code></pre><p>2、如果 <code>onFulfilled</code> 或者<code>onRejected</code> 抛出一个异常 <code>e</code> ，则 <code>promise2</code> 必须变为失败<code>（Rejected）</code>，并返回失败的值 <code>e</code>，例如：</p>
<pre><code class="hljs javascript copyable" lang="javascript"><span class="hljs-keyword">let</span> promise1 = <span class="hljs-keyword">new</span> <span class="hljs-built_in">Promise</span>(<span class="hljs-function">(<span class="hljs-params">resolve, reject</span>) =&gt;</span> {
  setTimeout(<span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
    resolve(<span class="hljs-string">'success'</span>)
  }, <span class="hljs-number">1000</span>)
})
promise2 = promise1.then(<span class="hljs-function"><span class="hljs-params">res</span> =&gt;</span> {
  <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> <span class="hljs-built_in">Error</span>(<span class="hljs-string">'这里抛出一个异常e'</span>)
})
promise2.then(<span class="hljs-function"><span class="hljs-params">res</span> =&gt;</span> {
  <span class="hljs-built_in">console</span>.log(res)
}, err =&gt; {
  <span class="hljs-built_in">console</span>.log(err) <span class="hljs-comment">//1秒后打印出：这里抛出一个异常e</span>
})
<span class="copy-code-btn">复制代码</span></code></pre><p>3、如果<code>onFulfilled</code> 不是函数且 <code>promise1</code> 状态为成功<code>（Fulfilled）</code>， <code>promise2</code> 必须变为成功<code>（Fulfilled）</code>并返回 <code>promise1</code> 成功的值，例如：</p>
<pre><code class="hljs javascript copyable" lang="javascript"><span class="hljs-keyword">let</span> promise1 = <span class="hljs-keyword">new</span> <span class="hljs-built_in">Promise</span>(<span class="hljs-function">(<span class="hljs-params">resolve, reject</span>) =&gt;</span> {
  setTimeout(<span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
    resolve(<span class="hljs-string">'success'</span>)
  }, <span class="hljs-number">1000</span>)
})
promise2 = promise1.then(<span class="hljs-string">'这里的onFulfilled本来是一个函数，但现在不是'</span>)
promise2.then(<span class="hljs-function"><span class="hljs-params">res</span> =&gt;</span> {
  <span class="hljs-built_in">console</span>.log(res) <span class="hljs-comment">// 1秒后打印出：success</span>
}, err =&gt; {
  <span class="hljs-built_in">console</span>.log(err)
})
<span class="copy-code-btn">复制代码</span></code></pre><p>4、如果 <code>onRejected</code> 不是函数且 <code>promise1</code> 状态为失败<code>（Rejected）</code>，<code>promise2</code>必须变为失败<code>（Rejected）</code> 并返回 <code>promise1</code> 失败的值，例如：</p>
<pre><code class="hljs javascript copyable" lang="javascript"><span class="hljs-keyword">let</span> promise1 = <span class="hljs-keyword">new</span> <span class="hljs-built_in">Promise</span>(<span class="hljs-function">(<span class="hljs-params">resolve, reject</span>) =&gt;</span> {
  setTimeout(<span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
    reject(<span class="hljs-string">'fail'</span>)
  }, <span class="hljs-number">1000</span>)
})
promise2 = promise1.then(<span class="hljs-function"><span class="hljs-params">res</span> =&gt;</span> res, <span class="hljs-string">'这里的onRejected本来是一个函数，但现在不是'</span>)
promise2.then(<span class="hljs-function"><span class="hljs-params">res</span> =&gt;</span> {
  <span class="hljs-built_in">console</span>.log(res)
}, err =&gt; {
  <span class="hljs-built_in">console</span>.log(err)  <span class="hljs-comment">// 1秒后打印出：fail</span>
})
<span class="copy-code-btn">复制代码</span></code></pre><p>根据上面的规则，我们来为 完善 <code>MyPromise</code></p>
<p>修改 <code>constructor</code> : 增加执行队列</p>
<p>由于 <code>then</code> 方法支持多次调用，我们可以维护两个数组，将每次 <code>then</code> 方法注册时的回调函数添加到数组中，等待执行</p>
<pre><code class="hljs javascript copyable" lang="javascript"><span class="hljs-keyword">constructor</span> (handle) {
  <span class="hljs-keyword">if</span> (!isFunction(handle)) {
    <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> <span class="hljs-built_in">Error</span>(<span class="hljs-string">'MyPromise must accept a function as a parameter'</span>)
  }
  <span class="hljs-comment">// 添加状态</span>
  <span class="hljs-keyword">this</span>._status = PENDING
  <span class="hljs-comment">// 添加状态</span>
  <span class="hljs-keyword">this</span>._value = <span class="hljs-literal">undefined</span>
  <span class="hljs-comment">// 添加成功回调函数队列</span>
  <span class="hljs-keyword">this</span>._fulfilledQueues = []
  <span class="hljs-comment">// 添加失败回调函数队列</span>
  <span class="hljs-keyword">this</span>._rejectedQueues = []
  <span class="hljs-comment">// 执行handle</span>
  <span class="hljs-keyword">try</span> {
    handle(<span class="hljs-keyword">this</span>._resolve.bind(<span class="hljs-keyword">this</span>), <span class="hljs-keyword">this</span>._reject.bind(<span class="hljs-keyword">this</span>)) 
  } <span class="hljs-keyword">catch</span> (err) {
    <span class="hljs-keyword">this</span>._reject(err)
  }
}
<span class="copy-code-btn">复制代码</span></code></pre><p>添加then方法</p>
<p>首先，<code>then</code> 返回一个新的 <code>Promise</code> 对象，并且需要将回调函数加入到执行队列中</p>
<pre><code class="hljs javascript copyable" lang="javascript"><span class="hljs-comment">// 添加then方法</span>
then (onFulfilled, onRejected) {
  <span class="hljs-keyword">const</span> { _value, _status } = <span class="hljs-keyword">this</span>
  <span class="hljs-keyword">switch</span> (_status) {
    <span class="hljs-comment">// 当状态为pending时，将then方法回调函数加入执行队列等待执行</span>
    <span class="hljs-keyword">case</span> PENDING:
      <span class="hljs-keyword">this</span>._fulfilledQueues.push(onFulfilled)
      <span class="hljs-keyword">this</span>._rejectedQueues.push(onRejected)
      <span class="hljs-keyword">break</span>
    <span class="hljs-comment">// 当状态已经改变时，立即执行对应的回调函数</span>
    <span class="hljs-keyword">case</span> FULFILLED:
      onFulfilled(_value)
      <span class="hljs-keyword">break</span>
    <span class="hljs-keyword">case</span> REJECTED:
      onRejected(_value)
      <span class="hljs-keyword">break</span>
  }
  <span class="hljs-comment">// 返回一个新的Promise对象</span>
  <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> MyPromise(<span class="hljs-function">(<span class="hljs-params">onFulfilledNext, onRejectedNext</span>) =&gt;</span> {
  })
}
<span class="copy-code-btn">复制代码</span></code></pre><p>那返回的新的 <code>Promise</code> 对象什么时候改变状态？改变为哪种状态呢？</p>
<p>根据上文中 <code>then</code> 方法的规则，我们知道返回的新的 <code>Promise</code> 对象的状态依赖于当前 <code>then</code> 方法回调函数执行的情况以及返回值，例如 <code>then</code> 的参数是否为一个函数、回调函数执行是否出错、返回值是否为 <code>Promise</code> 对象。</p>
<p>我们来进一步完善 <code>then</code> 方法:</p>
<pre><code class="hljs javascript copyable" lang="javascript"><span class="hljs-comment">// 添加then方法</span>
then (onFulfilled, onRejected) {
  <span class="hljs-keyword">const</span> { _value, _status } = <span class="hljs-keyword">this</span>
  <span class="hljs-comment">// 返回一个新的Promise对象</span>
  <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> MyPromise(<span class="hljs-function">(<span class="hljs-params">onFulfilledNext, onRejectedNext</span>) =&gt;</span> {
    <span class="hljs-comment">// 封装一个成功时执行的函数</span>
    <span class="hljs-keyword">let</span> fulfilled = <span class="hljs-function"><span class="hljs-params">value</span> =&gt;</span> {
      <span class="hljs-keyword">try</span> {
        <span class="hljs-keyword">if</span> (!isFunction(onFulfilled)) {
          onFulfilledNext(value)
        } <span class="hljs-keyword">else</span> {
          <span class="hljs-keyword">let</span> res =  onFulfilled(value);
          <span class="hljs-keyword">if</span> (res <span class="hljs-keyword">instanceof</span> MyPromise) {
            <span class="hljs-comment">// 如果当前回调函数返回MyPromise对象，必须等待其状态改变后在执行下一个回调</span>
            res.then(onFulfilledNext, onRejectedNext)
          } <span class="hljs-keyword">else</span> {
            <span class="hljs-comment">//否则会将返回结果直接作为参数，传入下一个then的回调函数，并立即执行下一个then的回调函数</span>
            onFulfilledNext(res)
          }
        }
      } <span class="hljs-keyword">catch</span> (err) {
        <span class="hljs-comment">// 如果函数执行出错，新的Promise对象的状态为失败</span>
        onRejectedNext(err)
      }
    }
    <span class="hljs-comment">// 封装一个失败时执行的函数</span>
    <span class="hljs-keyword">let</span> rejected = <span class="hljs-function"><span class="hljs-params">error</span> =&gt;</span> {
      <span class="hljs-keyword">try</span> {
        <span class="hljs-keyword">if</span> (!isFunction(onRejected)) {
          onRejectedNext(error)
        } <span class="hljs-keyword">else</span> {
            <span class="hljs-keyword">let</span> res = onRejected(error);
            <span class="hljs-keyword">if</span> (res <span class="hljs-keyword">instanceof</span> MyPromise) {
              <span class="hljs-comment">// 如果当前回调函数返回MyPromise对象，必须等待其状态改变后在执行下一个回调</span>
              res.then(onFulfilledNext, onRejectedNext)
            } <span class="hljs-keyword">else</span> {
              <span class="hljs-comment">//否则会将返回结果直接作为参数，传入下一个then的回调函数，并立即执行下一个then的回调函数</span>
              onFulfilledNext(res)
            }
        }
      } <span class="hljs-keyword">catch</span> (err) {
        <span class="hljs-comment">// 如果函数执行出错，新的Promise对象的状态为失败</span>
        onRejectedNext(err)
      }
    }
    <span class="hljs-keyword">switch</span> (_status) {
      <span class="hljs-comment">// 当状态为pending时，将then方法回调函数加入执行队列等待执行</span>
      <span class="hljs-keyword">case</span> PENDING:
        <span class="hljs-keyword">this</span>._fulfilledQueues.push(fulfilled)
        <span class="hljs-keyword">this</span>._rejectedQueues.push(rejected)
        <span class="hljs-keyword">break</span>
      <span class="hljs-comment">// 当状态已经改变时，立即执行对应的回调函数</span>
      <span class="hljs-keyword">case</span> FULFILLED:
        fulfilled(_value)
        <span class="hljs-keyword">break</span>
      <span class="hljs-keyword">case</span> REJECTED:
        rejected(_value)
        <span class="hljs-keyword">break</span>
    }
  })
}
<span class="copy-code-btn">复制代码</span></code></pre><blockquote>
<p>这一部分可能不太好理解，读者需要结合上文中 <code>then</code> 方法的规则来细细的分析。</p>
</blockquote>
<p>接着修改 <code>_resolve</code> 和 <code>_reject</code> ：依次执行队列中的函数</p>
<p>当 <code>resolve</code> 或  <code>reject</code> 方法执行时，我们依次提取成功或失败任务队列当中的函数开始执行，并清空队列，从而实现 <code>then</code> 方法的多次调用，实现的代码如下：</p>
<pre><code class="hljs javascript copyable" lang="javascript"><span class="hljs-comment">// 添加resovle时执行的函数</span>
_resolve (val) {
  <span class="hljs-keyword">if</span> (<span class="hljs-keyword">this</span>._status !== PENDING) <span class="hljs-keyword">return</span>
  <span class="hljs-comment">// 依次执行成功队列中的函数，并清空队列</span>
  <span class="hljs-keyword">const</span> run = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
    <span class="hljs-keyword">this</span>._status = FULFILLED
    <span class="hljs-keyword">this</span>._value = val
    <span class="hljs-keyword">let</span> cb;
    <span class="hljs-keyword">while</span> (cb = <span class="hljs-keyword">this</span>._fulfilledQueues.shift()) {
      cb(val)
    }
  }
  <span class="hljs-comment">// 为了支持同步的Promise，这里采用异步调用</span>
  setTimeout(<span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> run(), <span class="hljs-number">0</span>)
}
<span class="hljs-comment">// 添加reject时执行的函数</span>
_reject (err) { 
  <span class="hljs-keyword">if</span> (<span class="hljs-keyword">this</span>._status !== PENDING) <span class="hljs-keyword">return</span>
  <span class="hljs-comment">// 依次执行失败队列中的函数，并清空队列</span>
  <span class="hljs-keyword">const</span> run = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
    <span class="hljs-keyword">this</span>._status = REJECTED
    <span class="hljs-keyword">this</span>._value = err
    <span class="hljs-keyword">let</span> cb;
    <span class="hljs-keyword">while</span> (cb = <span class="hljs-keyword">this</span>._rejectedQueues.shift()) {
      cb(err)
    }
  }
  <span class="hljs-comment">// 为了支持同步的Promise，这里采用异步调用</span>
  setTimeout(run, <span class="hljs-number">0</span>)
}
<span class="copy-code-btn">复制代码</span></code></pre><p>这里还有一种特殊的情况，就是当 <code>resolve</code> 方法传入的参数为一个 <code>Promise</code> 对象时，则该 <code>Promise</code> 对象状态决定当前 <code>Promise</code> 对象的状态。</p>
<pre><code class="hljs javascript copyable" lang="javascript"><span class="hljs-keyword">const</span> p1 = <span class="hljs-keyword">new</span> <span class="hljs-built_in">Promise</span>(<span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">resolve, reject</span>) </span>{
  <span class="hljs-comment">// ...</span>
});
<span class="hljs-keyword">const</span> p2 = <span class="hljs-keyword">new</span> <span class="hljs-built_in">Promise</span>(<span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">resolve, reject</span>) </span>{
  <span class="hljs-comment">// ...</span>
  resolve(p1);
})
<span class="copy-code-btn">复制代码</span></code></pre><p>上面代码中，<code>p1</code> 和 <code>p2</code> 都是 <code>Promise</code> 的实例，但是 <code>p2</code> 的<code>resolve</code>方法将 <code>p1</code> 作为参数，即一个异步操作的结果是返回另一个异步操作。</p>
<p>注意，这时 <code>p1</code> 的状态就会传递给 <code>p2</code>，也就是说，<code>p1</code> 的状态决定了 <code>p2</code> 的状态。如果 <code>p1</code> 的状态是<code>Pending</code>，那么 <code>p2</code> 的回调函数就会等待 <code>p1</code> 的状态改变；如果 <code>p1</code> 的状态已经是 <code>Fulfilled</code> 或者 <code>Rejected</code>，那么 <code>p2</code> 的回调函数将会立刻执行。</p>
<p>我们来修改<code>_resolve</code>来支持这样的特性</p>
<pre><code class="hljs javascript copyable" lang="javascript">  <span class="hljs-comment">// 添加resovle时执行的函数</span>
  _resolve (val) {
    <span class="hljs-keyword">const</span> run = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
      <span class="hljs-keyword">if</span> (<span class="hljs-keyword">this</span>._status !== PENDING) <span class="hljs-keyword">return</span>
      <span class="hljs-comment">// 依次执行成功队列中的函数，并清空队列</span>
      <span class="hljs-keyword">const</span> runFulfilled = <span class="hljs-function">(<span class="hljs-params">value</span>) =&gt;</span> {
        <span class="hljs-keyword">let</span> cb;
        <span class="hljs-keyword">while</span> (cb = <span class="hljs-keyword">this</span>._fulfilledQueues.shift()) {
          cb(value)
        }
      }
      <span class="hljs-comment">// 依次执行失败队列中的函数，并清空队列</span>
      <span class="hljs-keyword">const</span> runRejected = <span class="hljs-function">(<span class="hljs-params">error</span>) =&gt;</span> {
        <span class="hljs-keyword">let</span> cb;
        <span class="hljs-keyword">while</span> (cb = <span class="hljs-keyword">this</span>._rejectedQueues.shift()) {
          cb(error)
        }
      }
      <span class="hljs-comment">/* 如果resolve的参数为Promise对象，则必须等待该Promise对象状态改变后,
        当前Promsie的状态才会改变，且状态取决于参数Promsie对象的状态
      */</span>
      <span class="hljs-keyword">if</span> (val <span class="hljs-keyword">instanceof</span> MyPromise) {
        val.then(<span class="hljs-function"><span class="hljs-params">value</span> =&gt;</span> {
          <span class="hljs-keyword">this</span>._value = value
          <span class="hljs-keyword">this</span>._status = FULFILLED
          runFulfilled(value)
        }, err =&gt; {
          <span class="hljs-keyword">this</span>._value = err
          <span class="hljs-keyword">this</span>._status = REJECTED
          runRejected(err)
        })
      } <span class="hljs-keyword">else</span> {
        <span class="hljs-keyword">this</span>._value = val
        <span class="hljs-keyword">this</span>._status = FULFILLED
        runFulfilled(val)
      }
    }
    <span class="hljs-comment">// 为了支持同步的Promise，这里采用异步调用</span>
    setTimeout(run, <span class="hljs-number">0</span>)
  }
<span class="copy-code-btn">复制代码</span></code></pre><p>这样一个Promise就基本实现了，现在我们来加一些其它的方法</p>
<p><code>catch</code> 方法</p>
<blockquote>
<p>相当于调用 <code>then</code> 方法, 但只传入 <code>Rejected</code> 状态的回调函数</p>
</blockquote>
<pre><code class="hljs javascript copyable" lang="javascript"><span class="hljs-comment">// 添加catch方法</span>
<span class="hljs-keyword">catch</span> (onRejected) {
  <span class="hljs-keyword">return</span> <span class="hljs-keyword">this</span>.then(<span class="hljs-literal">undefined</span>, onRejected)
}
<span class="copy-code-btn">复制代码</span></code></pre><p>静态 <code>resolve</code> 方法</p>
<pre><code class="hljs javascript copyable" lang="javascript"><span class="hljs-comment">// 添加静态resolve方法</span>
<span class="hljs-keyword">static</span> resolve (value) {
  <span class="hljs-comment">// 如果参数是MyPromise实例，直接返回这个实例</span>
  <span class="hljs-keyword">if</span> (value <span class="hljs-keyword">instanceof</span> MyPromise) <span class="hljs-keyword">return</span> value
  <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> MyPromise(<span class="hljs-function"><span class="hljs-params">resolve</span> =&gt;</span> resolve(value))
}
<span class="copy-code-btn">复制代码</span></code></pre><p>静态 <code>reject</code> 方法</p>
<pre><code class="hljs javascript copyable" lang="javascript"><span class="hljs-comment">// 添加静态reject方法</span>
<span class="hljs-keyword">static</span> reject (value) {
  <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> MyPromise(<span class="hljs-function">(<span class="hljs-params">resolve ,reject</span>) =&gt;</span> reject(value))
}
<span class="copy-code-btn">复制代码</span></code></pre><p>静态 <code>all</code> 方法</p>
<pre><code class="hljs javascript copyable" lang="javascript"><span class="hljs-comment">// 添加静态all方法</span>
<span class="hljs-keyword">static</span> all (list) {
  <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> MyPromise(<span class="hljs-function">(<span class="hljs-params">resolve, reject</span>) =&gt;</span> {
    <span class="hljs-comment">/**
     * 返回值的集合
     */</span>
    <span class="hljs-keyword">let</span> values = []
    <span class="hljs-keyword">let</span> count = <span class="hljs-number">0</span>
    <span class="hljs-keyword">for</span> (<span class="hljs-keyword">let</span> [i, p] <span class="hljs-keyword">of</span> list.entries()) {
      <span class="hljs-comment">// 数组参数如果不是MyPromise实例，先调用MyPromise.resolve</span>
      <span class="hljs-keyword">this</span>.resolve(p).then(<span class="hljs-function"><span class="hljs-params">res</span> =&gt;</span> {
        values[i] = res
        count++
        <span class="hljs-comment">// 所有状态都变成fulfilled时返回的MyPromise状态就变成fulfilled</span>
        <span class="hljs-keyword">if</span> (count === list.length) resolve(values)
      }, err =&gt; {
        <span class="hljs-comment">// 有一个被rejected时返回的MyPromise状态就变成rejected</span>
        reject(err)
      })
    }
  })
}
<span class="copy-code-btn">复制代码</span></code></pre><p>静态 <code>race</code> 方法</p>
<pre><code class="hljs javascript copyable" lang="javascript"><span class="hljs-comment">// 添加静态race方法</span>
<span class="hljs-keyword">static</span> race (list) {
  <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> MyPromise(<span class="hljs-function">(<span class="hljs-params">resolve, reject</span>) =&gt;</span> {
    <span class="hljs-keyword">for</span> (<span class="hljs-keyword">let</span> p <span class="hljs-keyword">of</span> list) {
      <span class="hljs-comment">// 只要有一个实例率先改变状态，新的MyPromise的状态就跟着改变</span>
      <span class="hljs-keyword">this</span>.resolve(p).then(<span class="hljs-function"><span class="hljs-params">res</span> =&gt;</span> {
        resolve(res)
      }, err =&gt; {
        reject(err)
      })
    }
  })
}
<span class="copy-code-btn">复制代码</span></code></pre><p><code>finally</code> 方法</p>
<blockquote>
<p><code>finally</code> 方法用于指定不管 <code>Promise</code> 对象最后状态如何，都会执行的操作</p>
</blockquote>
<pre><code class="hljs javascript copyable" lang="javascript"><span class="hljs-keyword">finally</span> (cb) {
  <span class="hljs-keyword">return</span> <span class="hljs-keyword">this</span>.then(
    <span class="hljs-function"><span class="hljs-params">value</span>  =&gt;</span> MyPromise.resolve(cb()).then(<span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> value),
    reason =&gt; MyPromise.resolve(cb()).then(<span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> { <span class="hljs-keyword">throw</span> reason })
  );
};
<span class="copy-code-btn">复制代码</span></code></pre><p>这样一个完整的 <code>Promsie</code> 就实现了，大家对 <code>Promise</code> 的原理也有了解，可以让我们在使用Promise的时候更加清晰明了。</p>
<p>完整代码如下</p>
<pre><code class="hljs javascript copyable" lang="javascript">  <span class="hljs-comment">// 判断变量否为function</span>
  <span class="hljs-keyword">const</span> isFunction = <span class="hljs-function"><span class="hljs-params">variable</span> =&gt;</span> <span class="hljs-keyword">typeof</span> variable === <span class="hljs-string">'function'</span>
  <span class="hljs-comment">// 定义Promise的三种状态常量</span>
  <span class="hljs-keyword">const</span> PENDING = <span class="hljs-string">'PENDING'</span>
  <span class="hljs-keyword">const</span> FULFILLED = <span class="hljs-string">'FULFILLED'</span>
  <span class="hljs-keyword">const</span> REJECTED = <span class="hljs-string">'REJECTED'</span>
  <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MyPromise</span> </span>{
    <span class="hljs-keyword">constructor</span> (handle) {
      <span class="hljs-keyword">if</span> (!isFunction(handle)) {
        <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> <span class="hljs-built_in">Error</span>(<span class="hljs-string">'MyPromise must accept a function as a parameter'</span>)
      }
      <span class="hljs-comment">// 添加状态</span>
      <span class="hljs-keyword">this</span>._status = PENDING
      <span class="hljs-comment">// 添加状态</span>
      <span class="hljs-keyword">this</span>._value = <span class="hljs-literal">undefined</span>
      <span class="hljs-comment">// 添加成功回调函数队列</span>
      <span class="hljs-keyword">this</span>._fulfilledQueues = []
      <span class="hljs-comment">// 添加失败回调函数队列</span>
      <span class="hljs-keyword">this</span>._rejectedQueues = []
      <span class="hljs-comment">// 执行handle</span>
      <span class="hljs-keyword">try</span> {
        handle(<span class="hljs-keyword">this</span>._resolve.bind(<span class="hljs-keyword">this</span>), <span class="hljs-keyword">this</span>._reject.bind(<span class="hljs-keyword">this</span>)) 
      } <span class="hljs-keyword">catch</span> (err) {
        <span class="hljs-keyword">this</span>._reject(err)
      }
    }
    <span class="hljs-comment">// 添加resovle时执行的函数</span>
    _resolve (val) {
      <span class="hljs-keyword">const</span> run = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
        <span class="hljs-keyword">if</span> (<span class="hljs-keyword">this</span>._status !== PENDING) <span class="hljs-keyword">return</span>
        <span class="hljs-comment">// 依次执行成功队列中的函数，并清空队列</span>
        <span class="hljs-keyword">const</span> runFulfilled = <span class="hljs-function">(<span class="hljs-params">value</span>) =&gt;</span> {
          <span class="hljs-keyword">let</span> cb;
          <span class="hljs-keyword">while</span> (cb = <span class="hljs-keyword">this</span>._fulfilledQueues.shift()) {
            cb(value)
          }
        }
        <span class="hljs-comment">// 依次执行失败队列中的函数，并清空队列</span>
        <span class="hljs-keyword">const</span> runRejected = <span class="hljs-function">(<span class="hljs-params">error</span>) =&gt;</span> {
          <span class="hljs-keyword">let</span> cb;
          <span class="hljs-keyword">while</span> (cb = <span class="hljs-keyword">this</span>._rejectedQueues.shift()) {
            cb(error)
          }
        }
        <span class="hljs-comment">/* 如果resolve的参数为Promise对象，则必须等待该Promise对象状态改变后,
          当前Promsie的状态才会改变，且状态取决于参数Promsie对象的状态
        */</span>
        <span class="hljs-keyword">if</span> (val <span class="hljs-keyword">instanceof</span> MyPromise) {
          val.then(<span class="hljs-function"><span class="hljs-params">value</span> =&gt;</span> {
            <span class="hljs-keyword">this</span>._value = value
            <span class="hljs-keyword">this</span>._status = FULFILLED
            runFulfilled(value)
          }, err =&gt; {
            <span class="hljs-keyword">this</span>._value = err
            <span class="hljs-keyword">this</span>._status = REJECTED
            runRejected(err)
          })
        } <span class="hljs-keyword">else</span> {
          <span class="hljs-keyword">this</span>._value = val
          <span class="hljs-keyword">this</span>._status = FULFILLED
          runFulfilled(val)
        }
      }
      <span class="hljs-comment">// 为了支持同步的Promise，这里采用异步调用</span>
      setTimeout(run, <span class="hljs-number">0</span>)
    }
    <span class="hljs-comment">// 添加reject时执行的函数</span>
    _reject (err) { 
      <span class="hljs-keyword">if</span> (<span class="hljs-keyword">this</span>._status !== PENDING) <span class="hljs-keyword">return</span>
      <span class="hljs-comment">// 依次执行失败队列中的函数，并清空队列</span>
      <span class="hljs-keyword">const</span> run = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
        <span class="hljs-keyword">this</span>._status = REJECTED
        <span class="hljs-keyword">this</span>._value = err
        <span class="hljs-keyword">let</span> cb;
        <span class="hljs-keyword">while</span> (cb = <span class="hljs-keyword">this</span>._rejectedQueues.shift()) {
          cb(err)
        }
      }
      <span class="hljs-comment">// 为了支持同步的Promise，这里采用异步调用</span>
      setTimeout(run, <span class="hljs-number">0</span>)
    }
    <span class="hljs-comment">// 添加then方法</span>
    then (onFulfilled, onRejected) {
      <span class="hljs-keyword">const</span> { _value, _status } = <span class="hljs-keyword">this</span>
      <span class="hljs-comment">// 返回一个新的Promise对象</span>
      <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> MyPromise(<span class="hljs-function">(<span class="hljs-params">onFulfilledNext, onRejectedNext</span>) =&gt;</span> {
        <span class="hljs-comment">// 封装一个成功时执行的函数</span>
        <span class="hljs-keyword">let</span> fulfilled = <span class="hljs-function"><span class="hljs-params">value</span> =&gt;</span> {
          <span class="hljs-keyword">try</span> {
            <span class="hljs-keyword">if</span> (!isFunction(onFulfilled)) {
              onFulfilledNext(value)
            } <span class="hljs-keyword">else</span> {
              <span class="hljs-keyword">let</span> res =  onFulfilled(value);
              <span class="hljs-keyword">if</span> (res <span class="hljs-keyword">instanceof</span> MyPromise) {
                <span class="hljs-comment">// 如果当前回调函数返回MyPromise对象，必须等待其状态改变后在执行下一个回调</span>
                res.then(onFulfilledNext, onRejectedNext)
              } <span class="hljs-keyword">else</span> {
                <span class="hljs-comment">//否则会将返回结果直接作为参数，传入下一个then的回调函数，并立即执行下一个then的回调函数</span>
                onFulfilledNext(res)
              }
            }
          } <span class="hljs-keyword">catch</span> (err) {
            <span class="hljs-comment">// 如果函数执行出错，新的Promise对象的状态为失败</span>
            onRejectedNext(err)
          }
        }
        <span class="hljs-comment">// 封装一个失败时执行的函数</span>
        <span class="hljs-keyword">let</span> rejected = <span class="hljs-function"><span class="hljs-params">error</span> =&gt;</span> {
          <span class="hljs-keyword">try</span> {
            <span class="hljs-keyword">if</span> (!isFunction(onRejected)) {
              onRejectedNext(error)
            } <span class="hljs-keyword">else</span> {
                <span class="hljs-keyword">let</span> res = onRejected(error);
                <span class="hljs-keyword">if</span> (res <span class="hljs-keyword">instanceof</span> MyPromise) {
                  <span class="hljs-comment">// 如果当前回调函数返回MyPromise对象，必须等待其状态改变后在执行下一个回调</span>
                  res.then(onFulfilledNext, onRejectedNext)
                } <span class="hljs-keyword">else</span> {
                  <span class="hljs-comment">//否则会将返回结果直接作为参数，传入下一个then的回调函数，并立即执行下一个then的回调函数</span>
                  onFulfilledNext(res)
                }
            }
          } <span class="hljs-keyword">catch</span> (err) {
            <span class="hljs-comment">// 如果函数执行出错，新的Promise对象的状态为失败</span>
            onRejectedNext(err)
          }
        }
        <span class="hljs-keyword">switch</span> (_status) {
          <span class="hljs-comment">// 当状态为pending时，将then方法回调函数加入执行队列等待执行</span>
          <span class="hljs-keyword">case</span> PENDING:
            <span class="hljs-keyword">this</span>._fulfilledQueues.push(fulfilled)
            <span class="hljs-keyword">this</span>._rejectedQueues.push(rejected)
            <span class="hljs-keyword">break</span>
          <span class="hljs-comment">// 当状态已经改变时，立即执行对应的回调函数</span>
          <span class="hljs-keyword">case</span> FULFILLED:
            fulfilled(_value)
            <span class="hljs-keyword">break</span>
          <span class="hljs-keyword">case</span> REJECTED:
            rejected(_value)
            <span class="hljs-keyword">break</span>
        }
      })
    }
    <span class="hljs-comment">// 添加catch方法</span>
    <span class="hljs-keyword">catch</span> (onRejected) {
      <span class="hljs-keyword">return</span> <span class="hljs-keyword">this</span>.then(<span class="hljs-literal">undefined</span>, onRejected)
    }
    <span class="hljs-comment">// 添加静态resolve方法</span>
    <span class="hljs-keyword">static</span> resolve (value) {
      <span class="hljs-comment">// 如果参数是MyPromise实例，直接返回这个实例</span>
      <span class="hljs-keyword">if</span> (value <span class="hljs-keyword">instanceof</span> MyPromise) <span class="hljs-keyword">return</span> value
      <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> MyPromise(<span class="hljs-function"><span class="hljs-params">resolve</span> =&gt;</span> resolve(value))
    }
    <span class="hljs-comment">// 添加静态reject方法</span>
    <span class="hljs-keyword">static</span> reject (value) {
      <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> MyPromise(<span class="hljs-function">(<span class="hljs-params">resolve ,reject</span>) =&gt;</span> reject(value))
    }
    <span class="hljs-comment">// 添加静态all方法</span>
    <span class="hljs-keyword">static</span> all (list) {
      <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> MyPromise(<span class="hljs-function">(<span class="hljs-params">resolve, reject</span>) =&gt;</span> {
        <span class="hljs-comment">/**
         * 返回值的集合
         */</span>
        <span class="hljs-keyword">let</span> values = []
        <span class="hljs-keyword">let</span> count = <span class="hljs-number">0</span>
        <span class="hljs-keyword">for</span> (<span class="hljs-keyword">let</span> [i, p] <span class="hljs-keyword">of</span> list.entries()) {
          <span class="hljs-comment">// 数组参数如果不是MyPromise实例，先调用MyPromise.resolve</span>
          <span class="hljs-keyword">this</span>.resolve(p).then(<span class="hljs-function"><span class="hljs-params">res</span> =&gt;</span> {
            values[i] = res
            count++
            <span class="hljs-comment">// 所有状态都变成fulfilled时返回的MyPromise状态就变成fulfilled</span>
            <span class="hljs-keyword">if</span> (count === list.length) resolve(values)
          }, err =&gt; {
            <span class="hljs-comment">// 有一个被rejected时返回的MyPromise状态就变成rejected</span>
            reject(err)
          })
        }
      })
    }
    <span class="hljs-comment">// 添加静态race方法</span>
    <span class="hljs-keyword">static</span> race (list) {
      <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> MyPromise(<span class="hljs-function">(<span class="hljs-params">resolve, reject</span>) =&gt;</span> {
        <span class="hljs-keyword">for</span> (<span class="hljs-keyword">let</span> p <span class="hljs-keyword">of</span> list) {
          <span class="hljs-comment">// 只要有一个实例率先改变状态，新的MyPromise的状态就跟着改变</span>
          <span class="hljs-keyword">this</span>.resolve(p).then(<span class="hljs-function"><span class="hljs-params">res</span> =&gt;</span> {
            resolve(res)
          }, err =&gt; {
            reject(err)
          })
        }
      })
    }
    <span class="hljs-keyword">finally</span> (cb) {
      <span class="hljs-keyword">return</span> <span class="hljs-keyword">this</span>.then(
        <span class="hljs-function"><span class="hljs-params">value</span>  =&gt;</span> MyPromise.resolve(cb()).then(<span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> value),
        reason =&gt; MyPromise.resolve(cb()).then(<span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> { <span class="hljs-keyword">throw</span> reason })
      );
    }
  }
<span class="copy-code-btn">复制代码</span></code></pre><p>如果觉得还行的话，点个赞、收藏一下再走吧。</p>
</div></article>
