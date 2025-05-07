+++
date = '2025-05-07T01:23:07+08:00'
draft = true
title = 'Cuda_1'
categories = ["cuda编程"]
tags = ["cuda"]
+++
# cuda

1. **cuda kelnel function**

    1. 核函数在GPU上进行并行执行

    2. 用限定词`__global__`进行修饰

    3. 返回值必须是`void`

    4. 形式

        ~~~c++
        // 形式1
        __global__ void kernal_function(argument arg)
        {
            printf("Hello World form GPU\n");
        }
        // 形式2
        void __global__ kernal_function(argument arg)
        {
        	printf("Hello World form GPU\n");    
        }
        ~~~

    5. 注意事项

        - 核函数只能访问GPU内存
        - 核函数不能使用变长参数
        - 核函数不能使用静态变量
        - 核函数不能使用函数指针
        - 核函数具有异步性 

    6. 具有核函数的cuda程序编写流程

        ~~~c++
        int main(void)
        {
            主机代码
        	核函数调用
            主机代码
            return 0;
        }
        ~~~

    7. 核函数不支持`c++`的`iostream`

2. 线程模型

    1. 线程模型结构
    2. 线程组织管理
    3. 网格和线程块限制

    