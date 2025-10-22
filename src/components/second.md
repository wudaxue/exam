2.

Promise
```js
  let sequence = Promise.resolve(); // 初始空任务链
  const results = []; // 按顺序保存结果

  async function fetchData(id) {
    // 模拟异步请求（0~2秒随机延迟）
    return new Promise((resolve) => {
      const delay = Math.random() * 2000;
      setTimeout(() => resolve(`数据-${id}`), delay);
    });
  }

  function handleClick(id) {
    const p = fetchData(id);

    // 将处理逻辑串联在 sequence 上
    sequence = sequence
      .then(() => p)
      .then((data) => {
        results.push(data);
        console.log('✅ 按顺序更新 UI:', data);
        updateUI(results);
      })
      .catch(console.error);
  }

  function updateUI(data) {
    console.log('当前 UI 数据:', data);
  }

  // 模拟点击按钮
  handleClick(1);
  handleClick(2);
  handleClick(3);

```

非promise
```js
  const queue = [];
  const results = [];
  let processing = false;

  function fetchData(id, callback) {
    const delay = Math.random() * 2000;
    setTimeout(() => callback(`数据-${id}`), delay);
  }

  function handleClick(id) {
    const task = { id, done: false, data: null };
    queue.push(task);

    fetchData(id, (data) => {
      task.done = true;
      task.data = data;
      processQueue();
    });
  }

  function processQueue() {
    if (processing) return;
    processing = true;

    (function next() {
      const first = queue[0];
      if (!first || !first.done) {
        processing = false;
        return;
      }

      // 处理队头
      results.push(first.data);
      console.log('✅ 按顺序更新 UI:', first.data);
      updateUI(results);

      // 移除已处理任务
      queue.shift();

      // 递归处理下一个
      next();
    })();
  }

  function updateUI(data) {
    console.log('当前 UI 数据:', data);
  }

  // 模拟点击
  handleClick(1);
  handleClick(2);
  handleClick(3);
```


3.
```html

<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <title>子页面</title>
</head>
<body>
  <h3 id="title"></h3>
  <button id="btn">调用另一个 iframe</button>

  <script>
  const params = new URLSearchParams(location.search);
  const id = params.get('id') || 'unknown';
  document.getElementById('title').textContent = `我是 ${id}`;

  // base64 工具
  const encode = (obj) => btoa(JSON.stringify(obj));
  const decode = (str) => { try { return JSON.parse(atob(str)); } catch { return {}; } };
  const genKey = () => 'k_' + Date.now() + '_' + Math.random().toString(36).slice(2,8);

  // 注册自己
  window.parent.postMessage({ type: 'register', id }, '*');

  // 监听消息
  window.addEventListener('message', (e) => {
    const msg = e.data;
    if (!msg || !msg.type) return;

    if (msg.type === 'register_ack') {
      console.log(`✅ ${id} 注册成功`);
    }

    if (msg.type === 'invoke') {
      const payload = decode(msg.payload);
      if (msg.from === 'parent') {
        console.log(`🟢 ${id} 收到来自主页面的消息：`, payload.text);
      } else {
        console.log(`🟢 ${id} 收到来自 ${msg.from} 的消息：${payload.text}，从 ${msg.via} 中转`);
      }

      // 回复执行结果
      window.parent.postMessage({
        type: 'response',
        from: id,
        to: msg.from,
        key: msg.key,
        payload: encode({ result: 'ok', key: msg.key })
      }, '*');
    }
  });

  // 点击按钮 → iframe2 调用 iframe3（通过主页面中转）
  document.getElementById('btn').addEventListener('click', () => {
    if (id !== 'iframe2') {
      alert('只有 iframe2 可以点击这个按钮发消息哦～');
      return;
    }

    const key = genKey();
    const payload = encode({ text: 'iframe2 发来的消息～' });
    console.log('📤 iframe2 → iframe3 发起调用');
    window.parent.postMessage({
      type: 'invoke',
      from: 'iframe2',
      to: 'iframe3',
      key,
      payload
    }, '*');
  });
  </script>
</body>
</html>

```

