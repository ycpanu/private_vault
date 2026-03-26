- 搜索最短路径
	```
	from collections import deque
	n,m=map(int,input().split())
	grid=[list(map(int,input().split())) for _ in range(n)]
	dis=[[-1]*m for _ in range(n)]
	
	def bfs():
	    q=deque()
	    q.append((0,0))
	    dis[0][0]=0
	    dx=[-1,0,1,0]
	    dy=[0,1,0,-1]
	    while q:
	        x,y=q.popleft()
	        for i in range(4):
	            nx,ny=x+dx[i],y+dy[i]
	            if nx>=0 and nx<n and ny>=0 and ny<m and dis[nx][ny]==-1 and grid[nx][ny]==0:
	                dis[nx][ny]=dis[x][y]+1
	                q.append((nx,ny))
	
	bfs()
	print(dis[n-1][m-1])
	```

- 八数码
	记录状态
	```
	from collections import deque
	data=list(map(str,input().split()))
	start=''.join(data)
	end='12345678x'
	dis={}
	
	def bfs():
		q=deque()
		dis[start]=0
		q.append(start)
		dx=[-1,0,1,0]
		dy=[0,1,0,-1]
		while q:
			t=q.popleft()
			distance=dis[t]
			if t==end:
				return distance
			pos=t.find('x')
			x,y=pos//3,pos%3
			for i in range(4):
				a=x+dx[i]
				b=y+dy[i]
				if a>=0 and a<3 and b>=0 and b<3:
					# 交换 x 和目标位置
					new_t = list(t)
					new_t[pos], new_t[a * 3 + b] = new_t[a * 3 + b], new_t[pos]
					new_t = ''.join(new_t)
					if new_t not in dis:
						dis[new_t] = distance + 1
						q.append(new_t)
		return -1
	
	print(bfs())

	```