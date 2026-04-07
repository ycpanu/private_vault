1. 处理负权边
	```
	import sys
	from collections import deque
	
	def main():
	    # 快速读取输入
	    data = sys.stdin.read().split()
	    if not data:
	        return
	    
	    n = int(data[0])
	    m = int(data[1])
	    
	    # 使用邻接表存图
	    graph = [[] for _ in range(n + 1)]
	    idx = 2
	    for _ in range(m):
	        u = int(data[idx])
	        v = int(data[idx+1])
	        w = int(data[idx+2])
	        idx += 3
	        graph[u].append((v, w))
	    
	    # 初始化
	    INF = float('inf')
	    dist = [INF] * (n + 1)
	    st = [False] * (n + 1) # 记录点是否在队列中
	    
	    dist[1] = 0
	    q = deque([1])
	    st[1] = True
	    
	    while q:
	        u = q.popleft()
	        st[u] = False # 节点出队后标记为不在队列中
	        
	        for v, w in graph[u]:
	            if dist[u] + w < dist[v]:
	                dist[v] = dist[u] + w
	                # 如果被更新的点不在队列中，则入队
	                if not st[v]:
	                    q.append(v)
	                    st[v] = True
	                    
	    # 输出结果
	    if dist[n] == INF:
	        print("impossible")
	    else:
	        print(dist[n])
	
	if __name__ == '__main__':
	    main()
	```

2. 判定是否存在负权回路
	**判定原理：**

	根据抽屉原理，如果从起点到某个点 $x$ 的最短路径包含 **$n$ 条边**，那么这条路径上一定经过了 **$n+1$ 个点**。由于图中总共只有 $n$ 个点，这说明路径中一定有重复的点，即存在环。又因为我们是在做“最短路”更新时发现的，所以这个环一定是**负权环**（只有绕一圈能让距离变短，算法才会继续更新）。
	
	**具体实现细节：**
	
	1. **多源初始化：** 图可能不是连通的，负环可能躲在某个从 1 号点出发无法到达的角落。因此，初始时我们要把**所有点**都加入队列，并假设它们的初始距离 `dist` 都是 0。
	    
	2. **边数计数：** 维护一个 `cnt[x]` 数组，表示从虚拟起点到 $x$ 的最短路径所包含的边数。
	    
	3. **判定条件：** 每次成功松弛一条边 $u \to v$ 时，令 `cnt[v] = cnt[u] + 1`。一旦发现 `cnt[v] >= n`，立即判定存在负环。
	```
	import sys
	from collections import deque
	
	def main():
	    # 1. 快速读取输入
	    data = sys.stdin.read().split()
	    if not data:
	        return
	    
	    n = int(data[0])
	    m = int(data[1])
	    
	    graph = [[] for _ in range(n + 1)]
	    idx = 2
	    for _ in range(m):
	        u = int(data[idx])
	        v = int(data[idx+1])
	        w = int(data[idx+2])
	        idx += 3
	        graph[u].append((v, w))
	        
	    # 2. 初始化
	    dist = [0] * (n + 1)
	    cnt = [0] * (n + 1)
	    st = [True] * (n + 1) # st 记录点是否在队列中
	    
	    # 注意：为了检测全图负环，需要将所有点初始都放入队列
	    q = deque(range(1, n + 1))
	    
	    # 3. SPFA 判定逻辑
	    while q:
	        u = q.popleft()
	        st[u] = False
	        
	        for v, w in graph[u]:
	            if dist[v] > dist[u] + w:
	                dist[v] = dist[u] + w
	                cnt[v] = cnt[u] + 1
	                
	                # 抽屉原理：边数 >= n 说明存在负环
	                if cnt[v] >= n:
	                    print("Yes")
	                    return
	                
	                if not st[v]:
	                    q.append(v)
	                    st[v] = True
	                    
	    print("No")
	
	if __name__ == '__main__':
	    main()
	```
	