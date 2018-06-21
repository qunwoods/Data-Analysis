# # hyperloglog算法原理

### 算法流程
	1. h:表示原始数据的hash值；
	2. p(s):表示左边起第一个1的位置；
	3. 初始化大小为m(2^b)的M值为负无穷；
	4. 对于h中的每个数，j为前b+1位；
	5. M为b+1位之后第一个1的位置；
	6. 将M[j]设置为max(M[j], w);
![](https://github.com/qunwoods/Data-Analysis/blob/master/CE1D4EAE-B4A7-413F-A2F8-049A8A2A635C.png)
	7. 根据公式计算基数值（其中alpha为偏差偏差矫正）
![](https://github.com/qunwoods/Data-Analysis/blob/master/EC55C098-71AD-41CB-A65E-F249ECE87056.png)

算法为代码（加入偏差矫正）：
[Sketch of the Day: HyperLogLog — Cornerstone of a Big Data Infrastructure – AK Tech Blog](http://content.research.neustar.biz/blog/hll.html
http://algo.inria.fr/flajolet/Publications/FlFuGaMe07.pdf
![](https://github.com/qunwoods/Data-Analysis/blob/master/BB03711F-F629-4F53-BAEB-E221591C1C00.png)
![](https://github.com/qunwoods/Data-Analysis/blob/master/05ADCED5-0751-4D3F-BFBD-021E77DF7384.png)
javascript实现：
```javascript
// b表示hash值的b位用来标识在registers中的位置，所以register的大小为2^b;
m = 2^b #with b in [4...16]
 
if m == 16:
    alpha = 0.673
elif m == 32:
    alpha = 0.697
elif m == 64:
    alpha = 0.709
else:
    alpha = 0.7213/(1 + 1.079/m)
 
// m：将长度为m的registers的元素初始化为0，用来存储每个hash值二进制中第一个1的位置；
registers = [0]*m # initialize m registers to 0，
 
##############################################################################################
# Construct the HLL structure
// 将所有的hash过后的元素转成二进制形式
for h in hashed(data):
	  // 取二进制右边b位 + 1即为register_index，对应存在registers[register_index]中
    register_index = 1 + get_register_index( h,b ) # binary address of the rightmost b bits
    // 从b+1位开始，记下第一个1的位置（从1开始计数），存为run_length
    run_length = run_of_zeros( h,b ) # length of the run of zeroes starting at bit b+1
    // 如果当前run_length大于registers[register_index]，则进行更新；
    registers[ register_index ] = max( registers[ register_index ], run_length )
 
##############################################################################################
# Determine the cardinality
DV_est = alpha * m^2 * 1/sum( 2^ -register )  # the DV estimate
 
if DV_est < 5/2 * m: # small range correction
    V = count_of_zero_registers( registers ) # the number of registers equal to zero
    if V == 0:  # if none of the registers are empty, use the HLL estimate
          DV = DV_est
    else:
          DV = m * log(m/V)  # i.e. balls and bins correction
 
if DV_est <= ( 1/30 * 2^32 ):  # intermediate range, no correction
     DV = DV_est
if DV_est > ( 1/30 * 2^32 ):  # large range correction
     DV = -2^32 * log( 1 - DV_est/2^32)
```

### 遇到的问题
#### druid导入时报错：
部分错误信息：Container [pid=70947,containerID=container_e22_1432969244945_6026_01_000075] is running beyond physical memory limits. Current usage: 4.0 GB of 4 GB physical memory used; 5.8 GB of 8.4 GB virtual memory used. Killing container.
原因分析：
相关解决方案：https://groups.google.com/forum/#!topic/druid-user/tEN_aGL_jR4
