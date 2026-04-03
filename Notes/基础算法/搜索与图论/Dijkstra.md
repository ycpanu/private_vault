- 单源最短路，存在自环、重边
```
	import sys
	import heapq
	
	def main():
	    # 1. 快速读取所有数据
	    data = sys.stdin.read().split()
	    if not data:
	        return
	        
	    n = int(data[0])
	    m = int(data[1])
	    
	    # 2. 邻接表建图
	    # graph[u] 存放元组 (目标节点 v, 边长 w)
	    graph = [[] for _ in range(n + 1)]
	    
	    idx = 2
	    for _ in range(m):
	        x = int(data[idx])
	        y = int(data[idx+1])
	        z = int(data[idx+2])
	        idx += 3
	        graph[x].append((y, z))
	        
	    # 3. Dijkstra 初始化
	    # dist 数组记录从起点 1 到各个点的目前最短距离，初始为无穷大
	    dist = [float('inf')] * (n + 1)
	    dist[1] = 0
	    
	    # visited 数组记录哪些点已经确定了最终的最短距离
	    visited = [False] * (n + 1)
	    
	    # 优先队列（最小堆），存放元组 (当前离起点的距离, 节点编号)
	    # Python 的 heapq 默认按元组的第一个元素（即距离）进行升序排序
	    pq = [(0, 1)]
	    
	    # 4. 开始核心的“贪心 + 松弛”过程
	    while pq:
	        # 每次都从堆顶拿出目前离起点最近的、且未确定最终距离的点
	        d, u = heapq.heappop(pq)
	        
	        # 【核心防线】懒惰删除：如果这个点之前已经确定过了，直接跳过
	        # 这完美防御了“重边”带来的冗余数据
	        if visited[u]:
	            continue
	            
	        # 盖章确认：点 u 的最短距离已经锁死，以后不再更改
	        visited[u] = True
	        
	        # 遍历点 u 的所有出边，尝试“松弛”它的邻居们
	        for v, w in graph[u]:
	            # 如果邻居 v 还没确定最终距离，且通过 u 走过去比以前记录的路线更近
	            if not visited[v] and dist[u] + w < dist[v]:
	                dist[v] = dist[u] + w
	                # 将更新后更优秀的距离压入堆中
	                heapq.heappush(pq, (dist[v], v))
	                
	    # 5. 输出结果
	    if dist[n] == float('inf'):
	        print("-1")
	    else:
	        print(dist[n])
	
	if __name__ == '__main__':
	    main()
```

