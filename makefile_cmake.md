# makefile

## 基础格式

```makefile
target: dependedcies
	commond
# 编译过程识别第一条target，根据dependedcies向下搜索，深搜+回溯
# 增量编译，当dependencies只有一部分有变化（更改时间更新）时，make只会重新编译有变化的项目，因此比较节省时间
```

## 变量

```makefile
TARGET = main # 变量定义，使用的时候通过$(TARGET)进行字符替换
CC = gcc
CFLAG = -g -Wall

$(TARGET): main.c foo.o
	$(CC) $(CFLAG) main.c foo.o -o $(TARGET)
foo.o: foo.c
	$(CC) $(CFLAG) -c foo.c
clean:
	rm -rf $(TARGET) *.o
	
```

## 多个可执行文件

```makefile
# main_1 main_2分别有main函数，作为两个不同的可执行文件，希望通过一次make就可以编译出两个可执行文件
TARGET1 = main_1
TARGET2 = main_2
CC = gcc

all: $(TARGET1) $(TARGET2) # all依赖于TARGET1和TARGET2,make在编译过程就会分别编译出两个TARGET
$(TARGET1): main_1.c foo.o
	$(CC) main_1.c foo.o -o $(TARGET1)
$(TARGET2): main_2.c foo.o
	$(CC) main_2.c foo.o -o $(TARGET2)
foo.o: foo.c
	$(CC) -c foo.c
clean:
	rm -rf $(TARGET1) $(TARGET2) *.o
	
```



# Cmake
