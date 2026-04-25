# 最大公约数：gcd(a,b)

- **代码模板** ：
	```
	long long gcd(long long a,long long b){
		if(b==0){
		return a;
		}
		else{
		return gcd(b,a%b);
		}
	}
	```

# 最小公倍数：lcm(a,b)

- **代码模板** ：
	```
	long long lcm(long a,long b){
		return (a/gcd(a,b))*b;
	}
	```