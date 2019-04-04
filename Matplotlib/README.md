# Matplotlib常用操作
---
## 一、介绍

[官网参考链接](https://matplotlib.org)

![img](https://www.matplotlib.org.cn/static/images/tutorials/anatomy.png '图示说明')



## 二、绘制简单的图

1. 线性与非线性图

    ```python
    x = np.linspace(0, 2, 100)  # 生成0到2的等间距100个数
    
    plt.plot(x, x, label='linear')  
    plt.plot(x, x**2, label='quadratic')
    plt.plot(x, x**3, label='cubic')
    plt.xlabel('x label')  # 设置x，y轴标签
    plt.ylabel('y label')
    plt.title("Simple Plot")  # 设置标题
    plt.legend()  # 显示图例
    
    plt.show()  # 显示出上述图片，jupyter里面不需要这句话，但是IDE里需要，不然没有图片。。。
    ```





