# makefile

## 基础格式

```makefile
target: dependedcies
	command # command必须以tab开头，不可以是四个空格
# 编译过程识别第一条target，根据dependedcies向下搜索，深搜+回溯
# 增量编译，当dependencies只有一部分有变化（更改时间更新）时，make只会重新编译有变化的项目，因此比较节省时间
# command中如果命令不写在同一行，由于make是多线程执行，多行命令不会在同一个bash中执行。如果希望同一个bash里执行，可以写在同一行，用分号隔开，或是类似于C语言中宏定义用饭斜杠连接
```

## 变量赋值

- := 简单赋值，可以理解成C语言的赋值=
- = 递归赋值，赋值之后所有与之相关的变量值都会改变
- ?= 条件赋值，如果变量未定义就赋值，否则维持原来的值不变
- += 追加赋值，在原变量的基础上加一个空格，之后加上赋值的内容

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
	
	
# 简单赋值
X := foo # foo
Y := $(X)bar # 此时Y是foobar
X := bar # X被改变成bar，由于是简单赋值，Y此时不受影响，还是foobar

# 递归赋值
X = foo # foo
Y = $(X)bar # 此时Y是foobar
X = bar # X被改变成bar，但是因为是递归赋值，此时X的改变会影响Y的值，此时Y的值会变更为barbar
```

## 自动化变量

```makefile
$@ # 表示目标文件
$^ # 表示依赖的文件列表，即所有的依赖文件
$< # 表示依赖的第一个文件，一般依赖文件有.c .h时，把.c放在第一个，使用$<表示.c文件
VPATH # 可以理解成环境变量PATH，用于多文件夹路径时提供给make搜索的路径
```



## 多个可执行文件

```makefile
# main_1 main_2分别有main函数，作为两个不同的可执行文件，希望通过一次make就可以编译出两个可执行文件
TARGET1 = main_1
TARGET2 = main_2
CC = gcc

.PHONY: all
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

## 通配符

makefile中支持使用通配符，例如 * % ? [...]

```makefile
test: *.c # 使用*通配符，匹配任意多个字符
    gcc -o $@ $^
    
OBJ = $(wildcard *.c) # 当通配符要用在依赖中的时候，需要用wildcard函数提取
test: $(OBJ)
	gcc -o $@ $^
```

## 条件变量

- ifeq

- ifneq

- ifdef

- ifndef

  ```makefile
  libs_for_gcc= -lgnu
  normal_libs=
  ifeq($CC, gcc)
  	lib = $(libs_for_gcc)
  else
  	lib = $(normal_libs)
  endif
  
  
  foo =
  bar = $(foo)
  ifdef foo
  	xxx # 此时虽然把foo赋值给bar，而foo是空，但是ifdef认为bar仍然被定义，条件为true，执行此处命令
  else
  	xxx
  endif
  ```

## 伪目标

```makefile
.PHONY: clean # 声明clean是一个伪目标，这样在make clean时，make会直接执行rm -rf的命令
# 如果不声明为伪目标，则当工程文件夹下有名字为clean的文件时，make会认为这个规则没有依赖文件，所以目标被认为是最新的而不去执行规则所定义的命令，也就是rm命令不会被执行
clean:
	rm -rf *.o
```

## make嵌套执行

```makefile
SUBDIR = ./subdir
$(SUBDIR):
	cd $(SUBDIR) && $(MAKE) # cd到./subdir下然后进行make，前提是subdir下也有一个makefile文件
	
SUBDIR = ./subdir
$(SUBDIR):
	$(MAKE) -C $@ # 和上述写法类似，只不过不需要显示地指定cd到subdir中，而是通过-C选项改变make的变量CURDIR
```

## 常用函数

```makefile
SRC = $(wildcard ./src/*.c) # wildcard函数列举出./src目录中所有的.c文件，foo.c bar.c
OBJ = $(patsubst %.c, %.o, $(SRC)) # patsubst把SRC中的所有.c的后缀改成.o，foo.o bar.o
$(TARGET): OBJ
	gcc -o $(TARGET) $(OBJ)
%.o: %.c
	gcc -o $@ -c $< #编译生成OBJ中的所有.o文件
```



# Cmake
