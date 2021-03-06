# 进程的虚拟地址空间
-  mm
-  active_mm

linux把虚拟地址空间分为两部分:内核空间和用户空间。
以面向对象的视角来看可以称内核空间实例和用户空间实例，内核空间实例是静态创建的而且就一个；用户空间实例是动态创建的，可以有多个，而且可以创建也可以销毁。
```
            +---------------------------------------------------+
            |                                                   |
            |                  kernel space #0                  |
            |                                                   |
            +------------+------------+------------+------------+
            |            |            |            |            |
            |   user     |   user     |            |   user     |
            |   space    |   space    |  .......   |   space    |
            |   #1       |   #2       |            |   #n       |
            |            |            |            |            |
            +------------+------------+------------+------------+
```
内核空间又称为匿名空间，同样用户空间又称为真实空间。为何这样称了，从用户空间的角度看，用户空间的每个角落即每个字节，它都能访问而且有访问的权限。而内核空间了，用户空间知道内空间的存在，因为知道内核空间的地址，但是也仅仅是知道它的地址而已，当它去访问时会报缺页错误。一句话概括就是用户空间知道内核空间的存在，但无权访问，基于这点称内核空间为匿名空间。
```c
struct task_struct {
  	struct mm_struct		*mm;
	struct mm_struct		*active_mm;
}
```
进程描述的 mm 表示该进程拥有的真实空间；active_mm 表示目前正在使用/激活的真实空间。为什么没有一个字段指向匿名空间了？因为它是全局静态的唯一的存在实例，它一直占据着物理内存，它不会换出也不会销毁，一直处于激活状态，所以不需要对它的引用。而真实的空间是动态创建的实例，它会载入和换出物理内存，也会销毁，所以需要一个索引才能操作它。
```c
//todo
```
真实空间的真实进程 “task->mm” 指向“真实空间"，active_mm 总是指向同一个真实空间。真实空间的使用者是用户进程，它需要真实空间，通过task->mm 来引用。
```c
//todo
```
匿名空间的匿名进程 “task->mm == NULL”，而且当这个匿名进程在运行时 tsk->active_mm 指向“借来”的虚拟空间。当匿名进程调度出去时，借来的空间原样归还。匿名空间的使用者是内核线程，它不关心真实空间，因此它不需要拥有真实空间，所以“task->mm == NULL”。
```c
//todo
```
现在有两个问题，为什么要借？借的是哪个？如果不借导致 task->mm == NULL；如果借的不是上一个导致 task->mm == new；都会导致 task->mm 刷新，这导致 TLB 抖动甚至击穿，为了缓存内核线程不关心的用户空间条目，导致内核线程运行的内核空间条目失效，这是极大的浪费且毫无意义。
```
                                +-----------------------------------+
                                | virtual page number | page offset |     virtual address
                                +-----------------------------------+
                                    |                           |               |
                                    |                           |               |
        TLB                         |                           |               |
        +---------------------------|-----------------+         |               |
        |       |                   |   |             |         |               |
        +-------+-------------------|---+-------------+         |               |
        |       |                   V   |       +     |         |               | translation
        +-------+-----------------------+-------|-----+         |               |
        |       |                       |       |     |         |               |
        +---------------------------------------|-----+         |               |
                                                |               |               |
                                                |               |               |
                                                V               V               V
                                +-----------------------------------+
                                |physical page number | page offset |     physical address
                                +-----------------------------------+

```
```c
//todo
```
