- 图中可能存在重边和自环，边权可能为负数，不存在负权回路
	```
	import sys
	def main():
	    data=sys.stdin.read().split()
	    n=int(data[0])
	    m=int(data[1])
	    q=int(data[2])
	    idx=3
	    INF=10**9
	    dist=[[INF]*(n+1) for _ in range(n+1)]
	    for i in range(1,n+1):
	        dist[i][i]=0
	    for _ in range(m):
	        u=int(data[idx])
	        v=int(data[idx+1])
	        w=int(data[idx+2])
	        idx+=3
	        if dist[u][v]>w:
	            dist[u][v]=w
	    
	    for k in range(1,n+1):
	        for u in range(1,n+1):
	            for v in range(1,n+1):
	                if dist[u][k]+dist[k][v]<dist[u][v]:
	                    dist[u][v]=dist[u][k]+dist[k][v]
	    
	    for _ in range(q):
	        u=int(data[idx])
	        v=int(data[idx+1])
	        idx+=2
	        if dist[u][v]>INF/2:
	            print('impossible')
	        else:
	            print(dist[u][v])
	if __name__=='__main__':
	    main()
	    
	```
