```
import sys

def main():
    # 1. 快速读取输入
    data = sys.stdin.read().split()
    if not data:
        return
        
    n = int(data[0])
    m = int(data[1])
    k = int(data[2])
    
    # 2. 存图：不需要邻接表，直接存所有的边即可
    edges = []
    idx = 3
    for _ in range(m):
        u = int(data[idx])
        v = int(data[idx+1])
        w = int(data[idx+2])
        idx += 3
        edges.append((u, v, w))
        
    # 定义一个足够大、且不会溢出的 INF
    # 题目最多 500 个点，边权最小 -10000，最差负距离是 -5000000
    # 我们设定 INF 为 50000000 (5千万) 绰绰有余
    INF = 50000000
    dist = [INF] * (n + 1)
    dist[1] = 0
    
    # 3. 核心逻辑：执行 k 次，代表最多走 k 条边
    for _ in range(k):
        # 【防御坑点 1】必须备份！防止发生串联更新
        backup = dist.copy()
        
        # 遍历所有的边尝试松弛
        for u, v, w in edges:
            # 使用“上一次的距离(backup)”来更新“当前的距离(dist)”
            if backup[u] + w < dist[v]:
                dist[v] = backup[u] + w
                
    # 4. 输出判断
    # 【防御坑点 2】不可达节点的 INF 可能会因为负权边变小，所以用 INF // 2 拦截
    if dist[n] > INF // 2:
        print("impossible")
    else:
        print(dist[n])

if __name__ == '__main__':
    main()
```

