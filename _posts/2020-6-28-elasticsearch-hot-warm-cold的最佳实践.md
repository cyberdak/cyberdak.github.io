---
layout : post
title : elasticsearch在日志场景的hot-warm-cold架构的最佳实践
category : "es"
tags : [elasticsearch,es,hot-cold]
---

elasticsearch 一直以来都支持hot-warm架构，但是如何来设置每个阶段的生命周期，却没有一个指导方案。

本文将结合日志业务给出一个hot-warm-cold架构的最佳实践，基于版本6.8.0，该版本自带index lifecycle management功能，中文名叫“索引生命周期策略
”，不带该功能的版本，可以通过搭配curator来实现。

### hot

1. shard 写满30g就启动rollover（滚动更新），因为rollover的索引就已经不在写入了，可以当成只读索引对待了。

### warm

2. 开启 warm阶段，使用默认的“滚动更新时移动到温阶段”
3. 开启强制合并，设置segment数量为1
4. 不迁移节点，在hot节点完成forcemerge，如果hot节点压力过大，可以考虑迁移到cold节点。如果考虑迁移到cold节点，则下面的cold.5阶段可以忽略。

### cold

5. 开启cold阶段，设置迁移到cold阶段的时间阈值。
6. （可选）如果冷节点内存吃紧，酌情开启frozen功能。同时可以考虑升级7.7以上版本，该版本实现了将一部分lucene数据从java堆迁移到硬盘中，释放了es对jvm堆的内存占用。

### delete

7. 开启delete阶段，设置删除时间阈值。
