# ST表

预处理后，O（1）的求出一段的区间最值（或者gcd）偏序关系？

使用st表必须能确定起点和终点，不能通过长度使用st表

```c++
//st[i][j]就表示从i开始的2^j个数的最大值
void init()
{
    for(int j=1;j<21;++j)
        for(int i=1;i+(1<<j)-1<=n;++i)
            st[i][j]=max(st[i][j-1],st[i+(1<<(j-1))][j-1])
    /*
    	i+len-1为第一段最后一个，这里的len=1<<(j-1)
    	而第二段的开头就为i+len-1+1，即i+(1<<(j-1))
    */
}
int ask(int l,int r)
{
    int k=log2(r-l+1);
    return max(st[l][k],st[r-(1<<k)+1][k]);
    /*
    	第一段的易于确定，而第二段则设为x，得
    		x+len-1=r
    	所以
    		x=r-len+1
    	即
    		x=r-(1<<k)+1；
    */
}
```

