#prim #稠密图
```
  import sys
	import heapq
	
	def main():
	    data = sys.stdin.read().split()
	    if not data:
	        return
	        
	    n = int(data[0])
	    m = int(data[1])
	    
	    # 1. 邻接表建图
	    graph = [[] for _ in range(n + 1)]
	    idx = 2
	    for _ in range(m):
	        u = int(data[idx])
	        v = int(data[idx+1])
	        w = int(data[idx+2])
	        idx += 3
	        # 无向图，双向建边
	        graph[u].append((v, w))
	        graph[v].append((u, w))
	        
	    # 2. 初始化
	    # visited 记录该点是否已经被纳入了生成树（帝国版图）
	    visited = [False] * (n + 1)
	    
	    # pq 存的是 (该点到当前生成树的最短距离, 节点编号)
	    # 随便选一个点作为起点，比如 1 号点，距离为 0
	    pq = [(0, 1)]
	    
	    res = 0  # 记录最小生成树的总权重
	    cnt = 0  # 记录成功纳入版图的节点数量
	    
	    # 3. 开始扩张
	    while pq:
	        d, u = heapq.heappop(pq)
	        
	        # 如果这个城市已经被占领了，直接跳过（处理重边和冗余入队）
	        if visited[u]:
	            continue
	            
	        # 盖章：正式纳入版图
	        visited[u] = True
	        res += d
	        cnt += 1
	        
	        # 如果所有城市都拿下了，可以提前收工
	        if cnt == n:
	            break
	            
	        # 侦察周边城市，更新它们离帝国的距离
	        for v, w in graph[u]:
	            # 只有还没被占领的城市才有价值
	            # 【核心区别】Prim 只关心 v 到树的距离 w，不管别的！
	            if not visited[v]:
	                # 直接把边长 w 扔进堆里
	                heapq.heappush(pq, (w, v))
	                
	    # 4. 判断连通性
	    # 如果扩张结束，版图里的城市数量依然小于 n，说明有孤岛不可达
	    if cnt == n:
	        print(res)
	    else:
	        print("impossible")
	
	if __name__ == '__main__':
	    main()
```