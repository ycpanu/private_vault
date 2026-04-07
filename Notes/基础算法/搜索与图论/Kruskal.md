- #最小生成树 #稀疏图 #Kruskal
- 核心思想：贪心 + 并查集
	1. 把图中所有的边按权重从小到大排好队。
	    
	2. 从最小的边开始挑：
	    
	    - 如果这条边的两个端点**还没有连通**（用并查集判断），就把这条边买下来，把两个点连起来。
	        
	    - 如果这两个端点**已经连通了**（说明如果加上这条边就会形成环），那就果断扔掉这条边，看下一条。
	        
	3. 当你刚好挑够了 $N - 1$ 条边时，一棵最小生成树就拼好了！如果挑完所有的边，选中的边数依然不到 $N - 1$，说明图里有无法到达的孤岛，输出 `impossible`。

- code
	```
	import sys
	
	# 极其精简的并查集查找函数（带路径压缩）
	def find(x, p):
	    if p[x] != x:
	        p[x] = find(p[x], p)
	    return p[x]
	
	def main():
	    # 1. 快速读取
	    data = sys.stdin.read().split()
	    if not data:
	        return
	        
	    n = int(data[0])
	    m = int(data[1])
	    
	    # 2. 存图：不需要复杂的邻接表，直接扔进一个大列表里即可
	    edges = []
	    idx = 2
	    for _ in range(m):
	        u = int(data[idx])
	        v = int(data[idx+1])
	        w = int(data[idx+2])
	        idx += 3
	        edges.append((u, v, w))
	        
	    # 3. Kruskal 核心步骤 1：按边权从小到大排序
	    # lambda x: x[2] 表示按照元组的第 3 个元素（也就是权重 w）进行排序
	    edges.sort(key=lambda x: x[2])
	    
	    # 初始化并查集：每个点起初各自为战，自己是自己的老大
	    p = list(range(n + 1))
	    
	    res = 0  # 记录最小生成树的总权重
	    cnt = 0  # 记录成功选中的边数
	    
	    # 4. Kruskal 核心步骤 2：从小到大贪心挑边
	    for u, v, w in edges:
	        # 寻找两端点各自的集合代表（老大）
	        pu = find(u, p)
	        pv = find(v, p)
	        
	        # 如果老大不同，说明还没连通，可以连！
	        if pu != pv:
	            p[pu] = pv   # 合并集合
	            res += w     # 累加权重
	            cnt += 1     # 选中边数 +1
	            
	            # 提前优化：如果已经选够了 n - 1 条边，直接完工退工
	            if cnt == n - 1:
	                break
	                
	    # 5. 输出结果
	    # 只有一棵包含 N 个点的连通树，才必定有且仅有 N - 1 条边
	    if cnt == n - 1:
	        print(res)
	    else:
	        print("impossible")
	
	if __name__ == '__main__':
	    # 开启递归深度限制，防止并查集极端情况下退化爆栈
	    sys.setrecursionlimit(200000)
	    main()
	```
