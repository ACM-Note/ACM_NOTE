## Tarjan原理详解

Tarjan算法是用来求强连通分量的，它是一种基于DFS（深度优先搜索）的算法，每个强连通分量为搜索树中的一棵子树。并且运用了数据结构栈。

### 算法思路：
1. 首先这个图不一定是一个连通图，所以跑Tarjan时要枚举每个点，若dfn[ ] == 0，进行深搜。
2. 对于搜到的点寻找与其有边相连的点，判断这些点是否已经被搜索过，若没有，则进行搜索。若该点已经入栈，说明形成了环，则更新low.
3. 在不断深搜的过程中如果没有路可走了（出边遍历完了），那么就进行回溯，回溯时不断比较low[ ]，去最小的low值。如果dfn[x]==low[x]则x可以看作是某一强连通分量子树的根，也说明找到了一个强连通分量，然后对栈进行弹出操作，直到x被弹出。


### 两个重要数组：
* dfn[ ]：就是一个时间戳（被搜到的次序），一旦某个点被DFS到后，这个时间戳就不再改变（且每个点只有唯一的时间戳）。所以常根据dfn的值来判断是否需要进行进一步的深搜。
* low[ ]：该子树中，且仍在栈中的最小时间戳，像是确立了一个关系，low[ ]相等的点在同一强连通分量中。
  
（注意初始化时 dfn[ ] = low[ ] = ++cnt.）