迭代最近点算法（Iterative Closests Point，ICP），找到一个可用的变换，使不同的坐标下的点云数据合并到同一个坐标系统中。

ICP算法本质上是基于最小二乘法的最优配准方法。该算法重复进行选择对应关系点对， 计算最优刚体变换，直到满足正确配准的收敛精度要求。

ICP 算法的目的是要找到待配准点云数据与参考云数据之间的旋转参数 R 和平移参数 T，使得两点数据之间满足某种度量准则下的最优匹配。


[三维点云配准 -- ICP 算法 | Yilin's Blog (yilingui.xyz)](https://yilingui.xyz/2019/11/20/191120_point_cloud_registration_icp/)


