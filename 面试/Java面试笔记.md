# 平衡二叉树

1. 空树，或者他的左右两个子树的高度的绝对值不会超过1

## 红黑树

### 关于NIL节点

所有数据都是节点而不是叶节点 所有的叶节点都是虚构的用NIL代替并且都是黑色的

### 关于平衡的两种考虑情况

对于一个节点子树来说 如果左右两个子节点都是黑色则就是平衡的 如果出现一个黑一个红，那么红色的那个节点向下一层作为平衡考虑 比如

![image-20200318082319775](/Users/bohanxiao/Library/Application Support/typora-user-images/image-20200318082319775.png)    这里P就是一颗局部平衡的子树 因为左右两个子节点都是黑色的 所以对于P来说 局部是平衡的 而下面是否平衡 则是S的事情



![image-20200318082425384](/Users/bohanxiao/Library/Application Support/typora-user-images/image-20200318082425384.png)     但是对于这棵树 因为S是红色的 所以考虑平衡的时候 需要向下一层 左图整体需要作为一颗子树来进行局部平衡考虑

### 插入

1. 根节点必须是黑色
2. 父子不能同为红色
3. 从任何一个节点出发，到达叶子节点所经过的黑色节点数量一致

左旋

<img src="/Users/bohanxiao/Library/Application Support/typora-user-images/image-20200316100620850.png" alt="image-20200316100620850"  />

左旋其实核心就是因为将右子节点变到父节点但是可能会发现如下情况

1. 右子节点本身有左子节点---这个点本身是比原来的父节点（黄色）大的 所以必须并到黄色的右边

右旋同理

![image-20200316113401002](/Users/bohanxiao/Library/Application Support/typora-user-images/image-20200316113401002.png)

总结：

​	父节点为黑直接插入

​	父节点如果为红色：

叔叔：U 

父节点：P 

祖父：PP 

插入：I

1. 叔叔节点存在并且为红
   1. U 和 P 都变成黑色 PP变成红色， 把PP设置成插入点（为了向上检查）
   2. 如果插入点是根节点 则变成黑色（唯一一种情况可能出现 黑黑树）

2. 叔叔节点不存在 或者为黑节点 

   1. P 为 PP的左子节点

      1. I是P的左子节点
         1. 将P设为黑色
         2. 将PP设为红色
         3. 对PP进行右旋![image-20200316114357007](/Users/bohanxiao/Library/Application Support/typora-user-images/image-20200316114357007.png)
         4. 因为最后P是黑色的 所以不会改变上层结构

      2. I 为P的右子节点

         1. 对P进行左旋（其实也就是左右旋）
         2. 把P设置为插入结点，得到情景4.2.1
         3. 进行情景4.2.1的处理

         ![image-20200316201045807](/Users/bohanxiao/Library/Application Support/typora-user-images/image-20200316201045807.png)

   2. P为PP的右节点

      1. I是P的右子节点
         1. 将P设为黑色
         2. 将PP设为红色
         3. 对PP进行左旋

![image-20200316201611876](/Users/bohanxiao/Library/Application Support/typora-user-images/image-20200316201611876.png)



​				2. I是P的左子节点 

​				右左旋

![image-20200316202151183](/Users/bohanxiao/Library/Application Support/typora-user-images/image-20200316202151183.png)





### 删除

其实每次删除的都是可以看做有一个替换节点 这个节点是 下一个比这个节点小的点

#### 找到替换节点

1. 情景1：若删除结点无子结点，直接删除
2. 情景2：若删除结点只有一个子结点，用子结点替换删除结点
3. 情景3：若删除结点有两个子结点，用后继结点（大于删除结点的最小结点）替换删除结点

#### 替换方法 

到达这一步的前提是 已经是来到了情况上述的情况1， 因为情况3会递归的转换到情况1 或者情况2，情况2又会转换到情况1，也就是说将会没有子节点 是一个单独的

下面的情况是

1. 如果替换节点是红色节点 则直接替换过去并且设置成和原来删除点的颜色（因为替换节点是红色不会影响平衡）

2. 如果替换节点是黑色节点 2

   1. 替换结点是其父结点的左子结点 **2.1**

      1. 替换结点的兄弟结点是红结点 （父节点则一定是黑色）**2.1.1**

         - **将S设为黑色**

         - **将P设为红色**

         - **对P进行左旋，得到情景2.1.2.3**

         - **进行情景2.1.2.3的处理**

         - ![image-20200317094958917](/Users/bohanxiao/Library/Application Support/typora-user-images/image-20200317094958917.png)

           

      2. 替换结点的兄弟结点是黑结点（父节点不确定什么颜色）**2.1.2**

         1. 替换结点的兄弟结点的右子结点是红结点，左子结点任意颜色 **2.1.2.1**（出口）

            1. **将S的颜色设为P的颜色**
            2. **将P设为黑色**
            3. **将SR设为黑色**
            4. **对P进行左旋**
            5. ![image-20200317095156005](/Users/bohanxiao/Library/Application Support/typora-user-images/image-20200317095156005.png)
            6. P为任意颜色无所谓， SL一定是红色或者nil否则无法平衡 那么开始的时候 左右是各一个黑色结束的时候也是各一个黑色 因为R会被删掉 SL是nil或者红色
            7. 这里默认R最后是要删除掉的 所以树仍然是平衡的

         2. 替换结点的兄弟结点的右子结点为黑结点，左子结点为红结点 **2.1.2.2**

            1. **将S设为红色**
            2. **将SL设为黑色**
            3. **对S进行右旋，得到情景2.1.2.1**
            4. **进行情景2.1.2.1的处理**
            5. ![image-20200317110505981](/Users/bohanxiao/Library/Application Support/typora-user-images/image-20200317110505981.png)
            6. 这里因为SL会有子节点或者NIL 所以情况对于R来说 局部上和 2.1.2.1是一样的

         3. 替换结点的兄弟结点的子结点都为黑结点**2.1.2.3**

            备注：

            1. **将S设为红色**
            2. **把P作为新的替换结点**
            3. **重新进行删除结点情景处理**
            4. ![image-20200317111023965](/Users/bohanxiao/Library/Application Support/typora-user-images/image-20200317111023965.png)
            5. 这里的两SL 和 SR 一定是NIL节点。因为R是没有子节点的，那么为了平衡 右子树只可能是两个NIL节点

   2. 替换结点是其父结点的右子结点 **2.2**

      1. 替换结点的兄弟结点是红结点 **2.2.1**
         1. **将S设为黑色**
         2. **将P设为红色**
         3. **对P进行右旋，得到情景2.2.2.3**
         4. **进行情景2.2.2.3的处理**
         5. ![image-20200317114402163](/Users/bohanxiao/Library/Application Support/typora-user-images/image-20200317114402163.png)

      2. 替换结点的兄弟结点是黑结点**2.2.2**

         1. 替换结点的兄弟结点的左子结点是红结点，右子结点任意颜色**2.2.2.1**
            1. **将S的颜色设为P的颜色**
            2. **将P设为黑色**
            3. **将SL设为黑色**
            4. **对P进行右旋**
            5. ![image-20200317114806646](/Users/bohanxiao/Library/Application Support/typora-user-images/image-20200317114806646.png)
         2. 替换结点的兄弟结点的左子结点为黑结点，右子结点为红结点
            1. **将S设为红色**
            2. **将SR设为黑色**
            3. **对S进行左旋，得到情景2.2.2.1**
            4. **进行情景2.2.2.1的处理**
            5. ![image-20200317114840166](/Users/bohanxiao/Library/Application Support/typora-user-images/image-20200317114840166.png)
         3. 替换结点的兄弟结点的子结点都为黑结点**2.2.2.3**
            1. **将S设为红色**
            2. **把P作为新的替换结点**
            3. **重新进行删除结点情景处理**
            4. ![image-20200317114911421](/Users/bohanxiao/Library/Application Support/typora-user-images/image-20200317114911421.png)

         