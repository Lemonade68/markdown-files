[TOC]

c++ alignas的理解：https://blog.csdn.net/wendyWJGU/article/details/134859784

***

Camera类：lookat类型的cam的view矩阵为什么是先乘rot再乘trans？

***

* glsl中，vec3 * vec3 是将两者对应的分量相乘后作为新的分量，结果仍是vec3；vec3加标量的结果也还是每个分量加上这个标量，仍为vec3

***

commandLineParser.hpp：用于解析输入的命令行语句(Parser)

***

initializers：用于初始化vk结构体（复杂的例如swapchain等就要按正常方法来写）

***

有关系的东西就可以写在一个struct里

***

![image-20240304084445792](Vulkan阅读.assets/image-20240304084445792.png)

***

从staging buffer复制到device local buffer的时候：使用vkBufferCopy来传递copy的信息，包括srcOffset，dstOffset，size三个参数（整个拷贝的话直接默认前两者为0即可）

Image则不同，用vkBufferImageCopy，还要考虑transition问题（见后面）

***

descriptor set layout：用于给不同的shader stages 绑定不同的descriptor types，如uniform buffer/image samplers等；**every shader binding should map to one descriptor set layout binding**

***

## stageMask & Access Mask 相关问题

**关于pWaitDstStageMask和srcStageMask等的理解**：**https://zhuanlan.zhihu.com/p/350483554**；pWaitDstStageMask表示cmdbuffer会先执行到当前的步骤，然后阻塞并等待semaphore，而不是一开始就阻塞；srcStageMask/srcAccessMask/dstStageMask/dstAccessMask：![image-20240304114813597](Vulkan阅读.assets/image-20240304114813597.png)

![image-20240304142105244](Vulkan阅读.assets/image-20240304142105244.png)

![image-20240304142839979](Vulkan阅读.assets/image-20240304142839979.png)

![image-20240304155851751](Vulkan阅读.assets/image-20240304155851751.png)

![image-20240304155910578](Vulkan阅读.assets/image-20240304155910578.png)

![image-20240304161421542](Vulkan阅读.assets/image-20240304161421542.png)

总结：主要问题就是再次从swapchain中拿出一张image的时候其layout是present_KHR，如果使用默认的dependency会出现问题（可能这张图片上次还没被输出完成就开始了layout转换）；使用显式的话写成如上面倒数第二幅图的样子，表示在等待该图片上次output的所有操作结束后（包括semaphore被标记后），再进行layout transition，然后再进行后面的subpass操作，如最后一张图所说

在vkcmddraw和vkcmdendrenderpass之后，framebuffer的color attachment会隐式变成VK_IMAGE_LAYOUT_PRESENT_SRC_KHR格式，以供之后的呈现步骤

***

## descriptorset相关问题

* descriptorsetLayout：将其中的descriptor类型和对应的shader stages绑定，**every shader binding should map to one descriptor set layout binding**，说明一个descriptor的布局是什么样，有几个binding，分别对应什么类型的descriptor**（见descriptorsets.cpp）**

* descriptorPool：声明要使用的descriptor类型以及最大数量（主要就是声明要分配的大小之类的）

* descriptorSets：用于指向uniform buffer或image sampler等，供shaders来获取全局变量，实际的buffer数据存于VkDescriptorBufferInfo结构体中，最后通过VkWriteDescriptorSet结构体（其中制定了要更新的set和对应的VkDescriptorBufferInfo）在vkUpdateDescriptorSets中传递

<img src="Vulkan阅读.assets/image-20240315102500772.png" alt="image-20240315102500772" style="zoom:50%;" />

***

vkMapMemory：将最后一个参数指向的地址与device中分配的内存buffer进行映射，从而使得通过最后参数指针可以修改分配的buffer的内容

***

## onenote上整理的记录

![image-20240312135555866](Vulkan阅读.assets/image-20240312135555866.png)

***

## FLAG_BITS相关问题

### MemoryPropertyFlagBits：

![image-20240314110617540](Vulkan阅读.assets/image-20240314110617540.png)

### BufferUsageFlagBits

![image-20240314110702469](Vulkan阅读.assets/image-20240314110702469.png)

### pipelineStageFlagBits：

https://imgtec.eetrend.com/blog/2020/100048837.html

![image-20240326140654280](Vulkan阅读.assets/image-20240326140654280.png)

![image-20240326140713450](Vulkan阅读.assets/image-20240326140713450.png)

![image-20240326140731106](Vulkan阅读.assets/image-20240326140731106.png)

![image-20240326140745303](Vulkan阅读.assets/image-20240326140745303.png)

***

## glsl的location/set/binding相关问题

glsl中，layout后面的location，set和binding的含义：

<img src="Vulkan阅读.assets/image-20240314201150792.png" alt="image-20240314201150792" style="zoom: 67%;" />

<img src="Vulkan阅读.assets/image-20240315134736229.png" alt="image-20240315134736229" style="zoom: 67%;" />

***

## Vertex Input 的理解

对Vulkan Vertex Input Description的理解：

​		1.	https://zhuanlan.zhihu.com/p/450157594

​		2.	https://blog.csdn.net/Motarookie/article/details/128844604      	没细看，讲的好像更清楚

<img src="Vulkan阅读.assets/v2-97dd2ffbd0fff80f1f8dfc734f6ea073_r.jpg" alt="img" style="zoom: 80%;" />

<img src="Vulkan阅读.assets/image-20240315141045045.png" alt="image-20240315141045045" style="zoom: 80%;" />

<img src="Vulkan阅读.assets/image-20240315141129806.png" alt="image-20240315141129806" style="zoom: 80%;" />

解释：上图中vertex的数据格式是：position, color, position, color, ..., instancePostion, instancePosition,...，这样的。因此对于同一个顶点来说，要取得的是position + color + instancePosition，就需要分成两个binding来取（两边的步长不同，需要分开取），这里就是binding description要干的；而每个步长中的属性怎么分配，对应shader中的location几，就是attribute description来干的。

***

## 两种constant对比

push_constant和specialization_constant的区别：

![image-20240316100941054](Vulkan阅读.assets/image-20240316100941054.png)

还有pc是在pipeline创建时绑定layout信息，但是在draw之前才指定实际数据，作用域是pipeline

而sc是在shader module创建后，pipeline创建前绑定好，作用域是shader域

***

## Texture 相关问题

![image-20240316103644932](Vulkan阅读.assets/image-20240316103644932.png)

获取image时，一般还是先用staging buffer来取得数据，然后copy（通过vkBufferImageCopy这个结构体），注意，当图片含有mipmap时，要考虑每一级的mipmap的offset以及长宽对应的大小变化；

* 与buffer不同的是，image需要考虑transition layout的问题，对应到例子中：第一次是从staging buffer拷贝到Image前，变为transfer_dst layout；第二次是拷贝结束后到读取image前，变为shader_read layout

* **重点:**   具体可以见vulkanTool.cpp 中的 **setImageLayout** 函数，如下：

  <img src="Vulkan阅读.assets/image-20240316164203512.png" alt="image-20240316164203512" style="zoom:67%;" />

  ---- 疑问：newLayout那边dstAccess的含义？是不是指在这个动作前transition必须结束（应该是，看上面stageMask那一部分）

**VkImageMemoryBarrier 用于在合适的阶段更改image layout，其中VkImageSubresourceRange定义了transition时的image对应region，其中参数如下图：**

<img src="Vulkan阅读.assets/image-20240317153512149.png" alt="image-20240317153512149" style="zoom: 50%;" />

之后还需要用到sampler和view这两个东西，使用**VkDescriptorImageInfo**来将view，image，sampler三者结合作为**COMBINED_IMAGE_SAMPLER 的 Descripter Info**

* 关于Texture array：见texturearray.cpp，后续看完instancing之后再看一下

***

## cubemap相关问题

别的大体相同，注意cube faces在vulkan中算作传入的image中的layer（因此数量为6）

<img src="Vulkan阅读.assets/image-20240317194856501.png" alt="image-20240317194856501" style="zoom:67%;" />

然后sampler正常写，view需要将viewType声明为cube

<img src="Vulkan阅读.assets/image-20240317195006723.png" alt="image-20240317195006723" style="zoom: 67%;" />

shader中的相关问题：一个是坐标系问题，二个是拖拽的相机的实现仍然需要理解（为什么shader里面没有model矩阵，以及为什么光源的位置是相机空间中的0，-5，5）



***

## Vulkan坐标系问题

<img src="Vulkan阅读.assets/image-20240317205701621.png" alt="image-20240317205701621" style="zoom:67%;" />

https://zhuanlan.zhihu.com/p/671588535

***

## Vulkan面向对象设计

### 1. vulkanDevice类

* **总结：管理physicalDevice，logicalDevice，内存分配memTypeIndex查找，buffer创建，queueIndex查找，extension检查，commandPool创建、commandBuffer创建，获取depthformat**

![image-20240323162839239](Vulkan阅读.assets/image-20240323162839239.png)

typeBits 和 propertyFlags的区别：

* typeBits可以直接声明为uint，利用的是其二进制的数据，每一位表示一种类型，为1则表示是否有当前类型需要匹配，即比较时是一位位比较，然后右移，改变最低位
* propertyFlags也是每一位表示一个信息，但是是整体与对应的flag进行相与，然后判断结果是否还是flag，是的话表明符合要求
* 在getMemoryType中，先判断typeBits是否为高，为高的话接着判断是否满足该类型所有flag，看下面

<img src="Vulkan阅读.assets/image-20240318104328805.png" alt="image-20240318104328805" style="zoom:80%;" />

* 分开找queueFamily的目的：**尽可能地将不同类型的任务分配到不同的队列族，以实现任务的并行执行，从而提高整体性能**（见VulkanDevice.cpp）—— **理解位与&**：只有设备支持的flag**包含**要求的flag才行，因此只有&后得到的是要求的flag时，才表明能支持要求的操作

***queue**是通过创建logical device时指定需要的queue的type，进而和Logical device一起创建出来的*

***queueFamilyIndex**是在创建device的时候顺便获取的*





### 2. vulkanBuffer类

* **总结：包含buffer，memory，device，descriptorInfo（真正的descriptorSet只在程序中唯一出现，所有ubo的descriptorInfo都会绑定在上面），内存绑定，类型flags等**

<img src="Vulkan阅读.assets/image-20240323162855713.png" alt="image-20240323162855713" style="zoom:67%;" />



### 3. vulkanTexture类

* 总结：包含device，image，imageLayout，view，sampler，memory以及descriptorInfo等，功能函数包括swapchain创建（包含recreate），呈现图片，获取下一张图片等

  <img src="Vulkan阅读.assets/image-20240323162919863.png" alt="image-20240323162919863" style="zoom:67%;" />



### 4. vulkanSwapchain类

surface这个东西只跟swapchain有关，因此放在里面，然后包括instance，swapchain images以及对应的view，还有format，以及swapchain用来present的queue（这里要求支持graphics和present操作）

<img src="Vulkan阅读.assets/image-20240323162953333.png" alt="image-20240323162953333" style="zoom:67%;" />



* **对指针操作的思考：当对指针指向的值进行覆盖写入的时候，原来的值就不需要额外的destory操作来删除了，因为数据被直接覆盖了**



<img src="Vulkan阅读.assets/image-20240323151430939.png" alt="image-20240323151430939" style="zoom: 67%;" />

* 看这里imageSharingMode对是否要设置queueIndex的影响



<img src="Vulkan阅读.assets/image-20240323144323536.png" alt="image-20240323144323536" style="zoom:67%;" />

* 注意这里swapchain中的image不需要自己销毁，销毁swapchain的时候自动会销毁



### 5. vulkanFrameBuffer类

* 总结：包括attachment类，framebuffer本身，renderPass（**创建framebuffer时要制定render pass，其中render pass 制定了对应的attachments等**），sampler等

<img src="Vulkan阅读.assets/image-20240323162640940.png" alt="image-20240323162640940" style="zoom: 67%;" />





### 6. vulkanGLTFModel类

* 为什么下面这个地方还要+vertexStart —— 见图中文字

![image-20240331164837940](Vulkan阅读.assets/image-20240331164837940.png)

![image-20240331164852134](Vulkan阅读.assets/image-20240331164852134.png)

* **为什么第一个example里面用的是两个descriptorset，一个用于martix(set = 0, binding = 0)，一个用于texture(set = 1, binding = 0)，而不是将两个合到一起**：第一个是大家一起共用的，不需要重复创建（所以pool那边数量为1）；第二个是每个image自己独有的，因此要单独创建(每个不同的面对应的texture的image不同)，所以pool那数量为image的数量。分开创建很清晰（应该也可以合起来创建，但是那样的话matrix会多创建不少）

![image-20240331193048021](Vulkan阅读.assets/image-20240331193048021.png)

<img src="Vulkan阅读.assets/image-20240331193141063.png" alt="image-20240331193141063" style="zoom:67%;" />





***

## imgui阅读





***

## 重构renderer

1. device和instance所对应的extension不同，device对应swapchain等，instance对应glfw/validation layer等



***

## 同步问题

http://forenose.com/column/content/427077449.html

https://blog.csdn.net/Navigated/article/details/131553609

https://zhuanlan.zhihu.com/p/632800454?utm_id=0      （这个介绍了srcStageMask之类的问题）

https://gpuopen.com/learn/vulkan-barriers-explained/   （介绍了vulkanBarrier）



