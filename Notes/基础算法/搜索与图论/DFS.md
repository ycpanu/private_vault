- 全排列
	```
	def dfs(u,n,p,st):
	    if u==n:
	        print(' '.join(map(str,p)))
	        return
	    for i in range(n):
	        if not st[i]:
	            p[u]=i+1 #下标u表示第u+1层
	            st[i]=True
	            dfs(u+1,n,p,st)
	            st[i]=False
	            
	if __name__=='__main__':
	    n=int(input())
	    p=[0]*n
	    st=[False]*n
	    dfs(0,n,p,st)
	```
	

- N皇后问题
	```
	def dfs(u,n,p,st):
	    if u==n:
	        print(' '.join(map(str,p)))
	        return
	    for i in range(n):
	        if not st[i]:
	            p[u]=i+1 #下标u表示第u+1层
	            st[i]=True
	            dfs(u+1,n,p,st)
	            st[i]=False
	            
	if __name__=='__main__':
	    n=int(input())
	    p=[0]*n
	    st=[False]*n
	    dfs(0,n,p,st)
	```

