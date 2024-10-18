# threadpoll
实现一个自带扩容功能的线程池
***创建线程池思路：**

1. 创建线程池初始化函数，传入最大线程数、最开始的线程数以及任务队列最大容量，包含：
   1. 初始化成员变量（互斥锁、条件变量以及线程容量相关）
   2. 初始化任务队列（头尾指针，当前任务数以及最大容量。完成后释放queue_ready信号量）
   3. 初始化初始线程队列，pthread_create，调用线程回调函数。
   4. 启动管理者线程
   5. do-while环节出错后跳出并抛出异常
2. 将任务（相关函数）添加到任务队列中：
   1. 加锁。
   2. 等待信号量queue_reaady，判断任务队列是否为空
   3. 将结构体task（包含回调函数名以及参数）加入到队列中
   4. 更新队列参数。
   5. 释放queue_not_empty参数
   6. 释放锁。
3. 线程回调函数callback（每个新建立的线程都要执行）：
   1. 加锁。
   2. 等待信号量queue_not_empty。
   3. 接收队列中的一个task，获取函数名以及返回参数。
   4. 修改队列参数。
   5. 释放锁，释放一个queue_ready信号零
   6. 互斥修改忙线程数、空闲线程数
   7. 执行task封装的回调函数。
   8. 互斥修改忙线程数、空闲线程数
   9. while循环，防止线程终止。
4. 管理者线程adjust_thrd:
   1. 获取线程池指针。
   2. 设置while循环以及每次检测的间隔时间。
   3. 互斥访问忙线程数以及空闲线程数和总线程数。
   4. 判断三者的大小关系，选择进行扩容还是瘦身
   5. 加锁，改变线程池大小，cur_thrd_num并且新建立若干线程。
   6. 互斥修改忙线程数以及空闲线程数和总线程数
   
