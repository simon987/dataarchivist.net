---
title: "Mg"
date: 2019-12-14T20:14:25-05:00
draft: true
---

<div style="height: 400px">
<img src="file:///home/drone/test.svg" style="width: 100%;height: 100%">
</div>


<script src="/mg/panzoom.min.js"></script>
<script>
const graph = document.getElementById("graph"); 
const panzoom = Panzoom(graph, {startScale: 1});
graph.addEventListener('wheel', (e) => {
    console.log(e);
    console.log(panzoom.zoomWithWheel(e));
});
</script>
