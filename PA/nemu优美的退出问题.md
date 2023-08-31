![[Pasted image 20230713151727.png]]
在nemu目录下使用make run命令，编译并执行后进入nemu之后使用q命令退出，会出现这个报错行为。而直接执行build目录下的riscv32-nemu-interpreter键入q命令退出nemu则不会出现报错
![[308c620063bdf7c9ea9cea52c200f897.jpg]]
出现这个错误的原因是程序在结束的时候有非0的返回值，导致报错。

![[ceb3e3ab395300da109c45b381a62619.jpg]]
grep命令查找main函数的位置
![[Pasted image 20230713152851.png]]
![[Pasted image 20230713153114.png]]
![[Pasted image 20230713153126.png]]
所以，看一下mainloop()函数
![[5b46ac1890ae02a28888e00522c34a42.jpg]]
这段代码是一个命令行解析器的主循环。它从标准输入中读取用户输入的命令，并根据命令表中的条目选择相应的处理函数进行处理。

1. 如果是批处理模式（即使用-b选项启动程序），则调用cmd_c函数，并直接返回。
    
2. 否则，进入一个无限循环，通过调用rl_gets函数从标准输入中读取用户输入的命令字符串，直到用户输入了"Ctrl-D"或者发生了错误。
    
3. 对于每一个读取到的命令字符串，先使用strtok函数提取出第一个空格之前的字符串作为命令名称，将剩余部分作为参数字符串。如果命令名为空，则跳过本轮循环。
    
4. 如果支持外设设备（如显示器、键盘等），则在处理命令之前先清空外设事件队列。
    
5. 遍历命令表中的所有条目，查找与当前命令名称匹配的处理函数，并调用该函数处理命令参数。如果未找到匹配的命令，则输出提示信息。
    
6. 如果处理函数返回负值，则退出主循环。

rl_gets()函数，从这里读入用户输入的命令
![[Pasted image 20230713153738.png]]
![[Pasted image 20230713153749.png]]
往下继续看代码，发现cmd_table结构体。保存了命令表，目前的命令很少。
![[223694fe6dab50b0aeef39f49e530137.jpg]]
int (*handler) (char *) , 可以使用cmd_table[i].xxx(cmd_help,cmd_c,cmd_q)执行相应的函数，很妙与在nemu运行时能够使用的命令契合
通过readline()读入命令然后通过strtok()过滤空格，通过strcmp()将键入的命令与cmd_table[i].name进行对比然后执行相应函数，那么在我们键入q时就会调用cmd_q函数
![[Pasted image 20230713154448.png]]
这样就会返回-1
![[Pasted image 20230713154510.png]]
![[Pasted image 20230713154544.png]]
之后就会返回is_exit_status_bad()的值。

结合find和grep可以找到is_exit_status_bad()函数的实现，在/src/utils/state.c下
![[2ea4a51e7e818cb5f8d3efdfea723964.jpg]]
![[5001ef2bab0bebe2481db3d0d1a46e86.jpg]]
使用gdb调试一下程序，good的值是由nemu_state的属性决定的
NEMU跑起来之后键入c，nemu_state.state最终变成了2，也就对应上了NEMU_END。相反，如果键入q的话，nemu_state一直到程序结束都不会发生变化![[Pasted image 20230713163313.png]]
这样修改之后就不会报错。
