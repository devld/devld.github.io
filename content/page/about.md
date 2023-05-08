---
title: 关于
comments: false
---

Hello World

{{< html >}}
<div style="text-align: center; color: #999; letter-spacing: 4px">
    <time id="current-time" style="font-size: 24px"></time>
</div>
<script>
const timeEl = document.getElementById('current-time');
timeEl.innerHTML = (new Date()).toLocaleString();
setInterval(function() {
    timeEl.innerHTML = (new Date()).toLocaleString();
}, 1000)
</script>
{{< /html >}}
