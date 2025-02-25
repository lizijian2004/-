# **C语言重新回顾, 内功提升**

### **函数特性**

****

scanf()函数如果会对输入结果进行一次判断,如果输入成功则返回输入个数,否则为0,且scanf()输入极为强大,可以代替其他输入函数. printf()函数也类似,scanf()输入字符串时不需要&,因为从某种意义上来说数组的名字就代表了数组首地址, scanf()后可以简单的使用getchar()刷新缓冲区, 对于windows平台 scanf_s() 输入字符串时候需要指定大小
if()判断语句可以进行bool掩码判断,不过为了阅读易于理解需要对变量进行优秀的命名

```c
#include<stdio.h>
#if _WIN32
#include<windows.h>
#elif __linux__
#include<unistd.h>
#endif

void print();

int main ()
{
    char str0[30] = {0};
    char str1[30] = {0};
    scanf("%9[a-zA-Z1-9]",str0);
    //scanf("%*[^\n]");scanf("%*c"); // 刷新输入缓存区 scanf中*代表丢弃
    int temp = 0;
    while((temp = getchar())!='\n' && temp != EOF); // 刷新输入缓存区
    scanf("%9[0-9 A-Za-z]",str1);
#if _WIN32
    Sleep(1000);
#elif __linux__
    sleep(1);
#endif
    printf("str0: %s , str1: %s\n", str0, str1);
    return 0;
}
```

### **函数递归**

***

函数递归都需要一个结束条件,如果没有结束条件就会因为调用函数过多而无法从栈中释放而爆栈,因为函数递归需要将剩下的未执行函数执行,执行结束以后才出栈

```c
#include <stdio.h>
#include <string.h>

char *reverse(char *str); // 反转（逆置）字符串 值得注意的是可以使用循环优先使用循环,这里为了方便使用了char类型

int main() {
    char str[15] = "123456789";
    printf("%s\n", reverse(str));
    return 0;
}

char *reverse(char *str) {
    int len = strlen(str);
    if (len > 1) {
        char ctemp = str[0];
        str[0] = str[len - 1];
        str[len - 1] = '\0';  // 最后一个字符在下次递归时不再处理
        reverse(str + 1);
        str[len - 1] = ctemp;
    }

    return str;
}
```

### **宏定义**

***

宏定义本质上其实是文本替换,并没有任何的其他作用,不过有些比较有意思的地方

```c
#include<stdio.h>
#if _WIN32
#include<windown.h>
    #error This maybe cant run in windows
#elif __linux__
#include<unistd.h>
#ifndef __cplusplus
    #define __cplusplus 1
#endif
#endif


#if defined(MAX)

#endif
#define PRINT(type,val,...) printf("DEBUG["#type"] " val "\n", ##__VA_ARGS__)

int main() {
    int age = 100;
    PRINT(ERROR, "age = %d", age);
    PRINT(WARNING, "value = %d", age);
    PRINT(ERROR, "BREAK");
    printf("This line is %d\n",__LINE__);
    printf("This function is %s\n",__FUNCTION__);
    printf("Now time is %s and %s\n", __DATE__, __TIME__);
    return 0;
}
```

* 宏定义只是简单的字符串替换，由预处理器来处理；而 typedef 是在编译阶段由编译器处理的，它并不是简单的字符串替换，而给原有的数据类型起一个新的名字，将它作为一种新的数据类型。
* 定义宏函数则应当将括号表示清楚, 避免语法错误, 且定义宏函数可以将变量名及其数值一起输出 #

```c
#include<stdio.h>
#include<stdlib.h>

#define PRINT(data) printf(#data" is %d",data)

int main(){
    typedef int* INT_Point;
    INT_Point point = NULL;
    int num = 6;
    PRINT(num);
    return 0;
}
```

* 宏定义使用 ## 可以链接文本 这样可以调用一些函数或者输出一些特定的数值

```c
#include<stdio.h>
#include<stdlib.h>

#define PRINT(data) print_ ## data


void print_name()
{
    printf("function name is %s \n",__FUNCTION__);
}

int main(){
    PRINT(name)();
    return 0;
}
```

* 联合是使用 # 和 ## 

```c
#include<stdio.h>
#include<stdlib.h>

#define DEBUG_PRINT(type,data,...) printf("DEBUG["#type"]" data "\n",##__VA_ARGS__) //如果没有第三个参数传入则取消第三个参数(...)

int main(){
    int num = 1;
    float num2 = 1.1;
    DEBUG_PRINT(ERROR, "level is %d ",num);
    DEBUG_PRINT(ERROR, "level is %f ",num2);
    DEBUG_PRINT(ERROR, "NOT FOUND");
    return 0;
}
```

* #if #elif 都需要跟着 **#endif**
* **#defined** 可以用于检查宏定义是否定义, 而且有用于简化的**#ifndef和#ifdef**
* 报错信息不需要加引号" "，如果加上，引号会被一起输出

```c
#include<stdio.h>
#include<stdlib.h>

#if _WIN32
    #error cannot run in windows
#elif __linux__
    #define NUM
#endif

int main(){
#ifdef NUM
    printf("NUM is defined\n");
#endif
    return 0;
}
```

### **指针**

***

- **数组指针**：是一个指向数组的指针，其本质是一个指针。

```c
#include <stdio.h>
int main(){
    int a[3][4]={0,1,2,3,4,5,6,7,8,9,10,11};
    int(*p)[4];  // 数组指针, 指向大小为4的数组, 且指向的是二维数组
    int i,j;
    p=a;
    for(i=0; i<3; i++){
        for(j=0; j<4; j++) 
            printf("%2d  ",*(*(p+i)+j));
        printf("\n");
    }

    return 0;
}
```

- **指针数组**：是一个数组，该数组中的每个元素都是一个指针。

```c
int *p[5]; // 指针数组, 一个数组, 其中每个成员都是指针
```

区分以上两个概念的本质是运算符优先级  **[ ]的优先级高于***

* 指针是函数传参的一个重要概念, 函数接受数组的时候一般传入为指针(数组的首地址), 因此不可以在函数中计算数组大小  可以将 int、float、char 等理解为基本类型，将数组理解为由基本类型派生得到的稍微复杂一些的类型。sizeof 就是根据符号的类型来计算长度的。

```c
#include<stdio.h>
#include<stdlib.h>
/* C语言标准规定，作为“类型的数组”的形参应该调整为“类型的指针”。在函数形参定义这个特殊情况下，编译器必须把数组形式改写成指向数组第 0 个元素的指针形式。编译器只向函数传递数组的地址，而不是整个数组的拷贝。*/

int arr_length(int arr[],int len){
    return len;
}

int main(){
    int arr[8] = {0};
    printf("%d", arr_length(arr, sizeof(arr) / sizeof(int)));
    return 0;
}
```

做个小小的测试

```c
#include <stdio.h>

int main(){
    // 小玩意绕饶你, 还是很简单的
    char *lines[5] = {
        "COSC1283/1284",
        "Programming",
        "Techniques",
        "is",
        "great fun"
    };

    char *str1 = lines[1]; // 指针指向第二个数据  "Programming"
    char *str2 = *(lines + 3);  // "is"
    char c1 = *(*(lines + 4) + 6);  // 'f'
    char c2 = (*lines + 5)[5];  // '2'  "COSC1283/1284" 靠后的这个2
    char c3 = *lines[0] + 2;  // 'E'

    printf("str1 = %s\n", str1);
    printf("str2 = %s\n", str2);
    printf("  c1 = %c\n", c1);
    printf("  c2 = %c\n", c2);
    printf("  c3 = %c\n", c3);

    return 0;
}
```

*  函数指针: 指向函数的指针 可以利用函数和结构体指针实现多态, 使用结构体实现继承(结构体嵌套)

```c
#include<stdio.h>

int max(int, int);

int main(){
    int x = 0, y = 0, maxval = 0;
    int (*pmax)(int, int) = max;
    scanf("%d %d", &x, &y);
    maxval = (*pmax)(x, y);
    printf("%d", maxval);
    return 0;
}

int max(int num0, int num1){
    return num0 > num1 ? num0 : num1;
}
```

* main()函数进行参数记录

```c
#include<stdio.h>

int main(int argc,char *argv[]){ // 可以接受输入的参数和参数名称 可以用于编写shell命令
    return 0;
}
```

### **结构体**

***

结构体(struct)拷贝不如直接使用指针, 因为结构体拷贝是拷贝整个数据而指针直接指向地址
枚举(enum)自动进行初始化, 从0开始, 也可以进行显式初始赋值
联合体(union) 结构体和共用体的区别在于：结构体的各个成员会占用不同的内存，互相之间没有影响；而共用体的所有成员占用同一段内存，修改一个成员会影响其余所有成员
联合体根据大小端存储的不同而导致读取方式不同

大端存储将数据的低位（比如 1234 中的 34 就是低位）放在内存的高地址上, 简单来说就是顺着读

| 内存地址 | 0x4000 | 0x4001 | 0x4002 | 0x4003 |
| -------- | ------ | ------ | ------ | ------ |
| 存放内容 | 0x12   | 0x34   | 0x56   | 0x78   |

小端存储与大端存储相反

| 内存地址 | 0x4000 | 0x4001 | 0x4002 | 0x4003 |
| -------- | ------ | ------ | ------ | ------ |
| 存放内容 | 0x78   | 0x56   | 0x34   | 0x12   |

```c
#include<stdio.h> // 判断大小端存储方法, 很简单的一种分别方法

union data{
    int num;
    char ch;
};

int main(){
    union data temp;
    temp.num = 1;
    if(temp.ch == 1){
        printf("Little-endian\n");
    }else{
        printf("Big-endian\n");
    }    return 0;
}
```

| &      | \|     | ^        | ~      | <<   | >>   |
| ------ | ------ | -------- | ------ | ---- | ---- |
| 按位与 | 按位或 | 按位异或 | 按位非 | 左移 | 右移 |

如果数据较小，被丢弃的高位不包含 1，那么左移 n 位相当于乘以 2 的 n 次方。 如果被丢弃的低位不包含 1，那么右移 n 位相当于除以 2 的 n 次方（但被移除的位中经常会包含 1）。

### **文件操作基础  [FILE结构体详解](https://c.biancheng.net/view/vip_2077.html)**

***

读取文件一般都带有缓存区(刷新缓存区依旧是\n), 不同系统下的换行符不同, windows是\r\n,其余为\n, linux系统写入文本的时候会默认带一个换行, 使用函数写入则需要写入换行符  **函数理解(scanf是从键盘获取数据到内存,printf是从内存输出数据到屏幕, 类似fscanf是从文件获取数据到内存, fprintf则是从内存写入数据到文件), 且写入文件时应该好好思考文件指针如何处理 打开方式为r的不可以创建文件其余均可 w会将源文件删除 a会在源文件最后添加数据** 

| 权限 | 说明                                                         |
| ---- | ------------------------------------------------------------ |
| "r"  | 以“只读”方式打开文件。只允许读取，不允许写入。文件必须存在，否则打开失败。 |
| "w"  | 以“写入”方式打开文件。如果文件不存在，那么创建一个新文件；如果文件存在，那么清空文件内容（相当于删除原文件，再创建一个新文件）。 |
| "a"  | 以“追加”方式打开文件。如果文件不存在，那么创建一个新文件；如果文件存在，那么将写入的数据追加到文件的末尾（文件原有的内容保留）。 |
| "r+" | 以“读写”方式打开文件。既可以读取也可以写入，也就是随意更新文件。文件必须存在，否则打开失败。 |
| "w+" | 以“写入/更新”方式打开文件，相当于`w`和`r+`叠加的效果。既可以读取也可以写入，也就是随意更新文件。如果文件不存在，那么创建一个新文件；如果文件存在，那么清空文件内容（相当于删除原文件，再创建一个新文件） |
| "a+" | 以“追加/更新”方式打开文件，相当于a和r+叠加的效果。既可以读取也可以写入，也就是随意更新文件。如果文件不存在，那么创建一个新文件；如果文件存在，那么将写入的数据追加到文件的末尾（文件原有的内容保留）。 |



```c
#include<stdio.h>
#include<string.h>
#include<assert.h>
// 文件以字符或字符串(格式基本相同)的方式读取

int main(int argc,char*argv[]) {
    FILE *fp = fopen("/home/l/文档/测试的废话","r+"); 
    assert(fp);// fp = NULL; 时会直接终止
    if (fp)
        puts("文件读取成功\n");
    else{
        perror("文件异常");
        return 0;
    }
    
    char ch = 0,carr[BUFSIZ]={0};
    while ((ch = fgetc(fp)) != EOF){
        printf("%c",ch);
    }
    
    while(ch != EOF && (scanf("%c", carr)) != EOF ){
        fputs(carr, fp);
    }
    
    if (feof(fp)) 
        puts("\n文件读取成功且结束");
    if(ferror(fp))
        perror("文件使用中出现错误");
    
    fclose(fp);
    return 0;
}
```

```c
#include<stdio.h>
#include<string.h>
#include<assert.h>
// 文件以数据块的方式读取

int main(int argc,char*argv[]) {
    FILE *fp = fopen("/home/l/文档/测试的废话","r+"); 
    assert(fp);// fp = NULL; 时会直接终止
    if (fp)
        puts("文件读取成功\n");
    else{
        perror("文件异常");
        return 0;
    }
    
    const char temp[] = "随便记录的一点文字,并没有实际意义";
    fwrite(temp, sizeof(char), strlen(temp), fp);
    rewind(fp);
    
    char ch = 0,carr[BUFSIZ]={0};
    fread(carr, sizeof(char), BUFSIZ - 1, fp);
    printf("%s\n", carr);

    const char temp2[] = ", 临时输入一段话用于测试代码";
    fwrite(temp2, sizeof(char), strlen(temp2), fp);
    rewind(fp);

    fread(carr, sizeof(char), BUFSIZ - 1, fp);
    printf("%s", carr);

    if (feof(fp))
        puts("\n文件读取成功且结束");
    if(ferror(fp))
        perror("文件使用中出现错误");
    
    fclose(fp);
    return 0;
}
```

```c
#include<stdio.h>
#include<string.h>
#include<assert.h>
// 文件以初始化的方式读取

int main(int argc,char*argv[]) {
    FILE *fp = fopen("/home/l/文档/测试的废话","r+"); 
    assert(fp);// fp = NULL; 时会直接终止
    if (fp)
        puts("文件读取成功\n");
    else{
        perror("文件异常");
        return 0;
    }

    char temp[BUFSIZ] = {0};
    fscanf(fp, "%[^\n]", temp);
    printf("%s\n\n", temp);
    
    fprintf(fp, "\n%s", temp);

    rewind(fp);
    fread(temp, sizeof(char), BUFSIZ - 1, fp);
    printf("%s\n", temp);

    if (feof(fp))
        puts("\n文件读取成功且结束");
    if(ferror(fp))
        perror("文件使用中出现错误");
    
    fclose(fp);
    return 0;
}
```

| 起始点   | 常量名   | 常量值 |
| -------- | -------- | ------ |
| 文件开头 | SEEK_SET | 0      |
| 当前位置 | SEEK_CUR | 1      |
| 文件末尾 | SEEK_END | 2      |



```c
#include<stdio.h>
#include<string.h>
#include<assert.h>
// 文件任意位置读写

int main(int argc,char*argv[]){
    FILE *fp = fopen("/home/l/文档/测试的废话", "a+");
    if(fp)
        puts("文件正常打开\n");
    else{
        perror("文件异常");
        return 0;
    }
    
    char temp[BUFSIZ] = {0};
    fscanf(fp, "%[^\n]", temp);
    fprintf(stdout, "%s\n", temp);

    fseek(fp, 0, SEEK_SET); // 中间参数为偏移量,正数向后移动,负数向前移动
    
    if (ferror(fp))
        perror("文件使用中出现错误");
    fclose(fp);
    return 0;
}
```

#### 文件操作小实现

***

**文件复制**: 缓冲区过小会造成读写次数的增加，过大也不能明显提高效率。目前大部分磁盘的扇区都是4K对齐的，如果读写的数据不是4K的整数倍，就会跨扇区读取，降低效率，所以我们开辟4K的缓冲区。 windows写文件的时候如果确定是文本文件则需要使用 t, 其余文件则使用二进制文件写入  b 其余系统则不需要修改文件打开方式

```c
#include<stdio.h>
#include<stdbool.h>
#include<stdlib.h>
#include<assert.h>

bool cp(char *, char *);

int main(int argc,char*argv[]){
    char fileName[256] = {0}, fileWrite[256] = {0}; // 在windows中文件名(文件路径)计数以字节为单位, linux中文件名不超过255字符书即可
    scanf("%s", fileName);
    scanf("%s", fileWrite);
    cp(fileName, fileWrite);
    return 0;
}

bool cp(char *fileName, char *fileWrite){
    FILE *file = fopen(fileName, "r");
    if(!file){
        perror("文件缺失");
        return 0;
    }
    FILE *target = fopen(fileWrite, "w");
    if(!target){
        perror("文件创建失败");
        return 0;
    }

    int bufleng = 1024 * 4,read_count = 0;
    char *bufsize = (char*)malloc(sizeof(char)*bufleng);

    while((read_count = fread(bufsize,1,bufleng,file))>0) {
        fwrite(bufsize, read_count, 1, target);
    }
    free(bufsize);
    fclose(file);
    fclose(target);
    return 1;
}
```

**获取文件大小**: 注意到文件指针, 文件指针应该保存

```c
#include<stdio.h>
#include<assert.h>

long fsize(char *);

int main(int argc,char*argv[]){
    char fileName[256] = {0}, fileWrite[256] = {0}; // 在windows中文件名(文件路径)计数以字节为单位, linux中文件名不超过255字符书即可
    scanf("%s", fileName);
    scanf("%s", fileWrite);
    cp(fileName, fileWrite);
    return 0;
}

long fsize(char* fileName){
    FILE *fp = fopen(fileName, "r");
    assert(fp);
    fpos_t pos;             // fpos_t 结构体用于存储 文件中指针的地址
    fgetpos(fp, &pos);      // 获取文件指针的地址
    fseek(fp, 0, SEEK_END);
    long size = ftell(fp);  // ftell 函数用于获取文件中指针距离文件开头的字节数
    fsetpos(fp, &pos);      // 设置文件中指针的地址
    return size;
}
```

**插入, 删除**:  由于常见的文件都是连续文件(连续文件数据是连续的,改动文件中间数据比较麻烦且耗时较长)而C语言没有提供插入,删除函数, 因此实现过程只能自己实现.  插入函数应该注意写入的数据原本是从一个buffer中存储所以要将buffer指针(最方便的话类型定义为void*)传入 , 删除函数很简单不做介绍

```c
#include<stdio.h>
#include<math.h>
#include<stdbool.h>
#include<stdlib.h>
#include<assert.h>


long fcopy(FILE *fSource, long offsetSource, long len, FILE *fTarget, long offsetTarget);
long fsize(FILE *);
long finsert(FILE *fp, long offset, void *buffer, int len);
int fdelete(FILE *fp, long offset, int len);

int main(int argc, char *argv[])
{
    char fileName[256] = {0}, fileWrite[256] = {0}; // 在windows中文件名(文件路径)计数以字节为单位, linux中文件名不超过255字符书即可
    FILE *source = fopen(fileName, "r+");
    FILE *target = fopen(fileWrite, "a+");
    return 0;
}


/**
 * 文件复制函数
 * @param  fSource       要复制的原文件
 * @param  offsetSource  原文件的位置偏移（相对文件开头），也就是从哪里开始复制
 * @param  len           要复制的内容长度，小于0表示复制offsetSource后边的所有内容
 * @param  fTarget       目标文件，也就是将文件复制到哪里
 * @param  offsetTarget  目标文件的位置偏移，也就是复制到目标文件的什么位置
 * @return 是否成功复制
**/
long fcopy(FILE *fSource, long offsetSource, long len, FILE *fTarget, long offsetTarget){
    int bufferLen = 1024*4;
    char *buffer = (char*)malloc(bufferLen);  
    int readCount;
    long nBytes = 0;  //总共复制了多少个字节

    fseek(fSource, offsetSource, SEEK_SET);
    fseek(fTarget, offsetTarget, SEEK_SET);
    if(len<0){  //复制所有内容
        while( (readCount=fread(buffer, 1, bufferLen, fSource)) > 0 ){
            nBytes += readCount;
            fwrite(buffer, readCount, 1, fTarget);
        }
    }else{  //复制len个字节的内容
        int loopNum = (int)ceil((double)((double)len/bufferLen));
        for(int i=1; i<=loopNum; i++){
            if(len-nBytes < bufferLen){ bufferLen = len-nBytes; }
            readCount = fread(buffer, 1, bufferLen, fSource);
            fwrite(buffer, readCount, 1, fTarget);
            nBytes += readCount;
        }
    }
    fflush(fTarget);
    free(buffer);
    return nBytes;
}

long fsize(FILE* fileName){
    fpos_t pos;             // fpos_t 结构体用于存储 文件中指针的地址
    fgetpos(fileName, &pos);      // 获取文件指针的地址
    fseek(fileName, 0, SEEK_END);
    long size = ftell(fileName);  // ftell 函数用于获取文件中指针距离文件开头的字节数
    fsetpos(fileName, &pos);      // 设置文件中指针的地址
    return size;
}


/**
 * 向文件中插入内容
 * @param  fp      要插入内容的文件
 * @param  buffer  缓冲区，也就是要插入的内容
 * @param  offset  偏移量（相对文件开头），也就是从哪里开始插入
 * @param  len     要插入的内容长度
 * @return  成功插入的字节数
**/
long finsert(FILE *fp, long offset, void *buffer, int len){
    long fileSize = fsize(fp);
    if(offset < 0 || len < 0)
    {
        perror("参数错误");
        return -1;
    }

    if(offset == fileSize){  //在文件末尾插入
        fseek(fp, offset, SEEK_SET);
        fwrite(buffer, len, 1, fp);
        if(ferror(fp))
            return -1;
    }

    if(offset < fileSize){  //从开头或者中间位置插入
        FILE *fpTemp = tmpfile();
        fcopy(fp, 0, offset, fpTemp, 0);
        fwrite(buffer, len, 1, fpTemp);
        fcopy(fp, offset, -1, fpTemp, offset+len);
        freopen("/home/l/文档/reopenFile", "w+", fp );
        fcopy(fpTemp, 0, -1, fp, 0);
        fclose(fpTemp);
    }
    return 0;
}


int fdelete(FILE *fp, long offset, int len){
    long fileSize = fsize(fp);
    if(offset>fileSize || offset < 0 || len < 0)
        return -1;
    FILE *fpTemp = tmpfile();
    fcopy(fp, 0, offset, fpTemp, 0);   // 前数据已经保存
    fcopy(fp, offset + len, -1, fpTemp, offset);
    freopen("/home/l/文档/newFile", "w+", fp);
    fcopy(fpTemp, 0, -1, fp, 0);

    fclose(fpTemp);
    return 0;
}
```

### 动态链接库

****

**动态链接库**: 除了复杂之外就是快且省内存, 一般用于大型项目, 由于比较复杂所以之后的代码具体实现省略

```shell
gcc -fpic -shared one.c two.c -o libtemp.so
winodws平台使用dll, linux使用so后缀
如此使用Gcc创建了一个动态链接库, 静态链接库使用static, 此处省略这种写法
```

```c
#include<stdio.h>
#include<dlfcn.h>

int main(int argc,char* argv[]){
    void *handler = dlopen("动态链接库路径",RTLD_LAZY); // 使用指针打开库文件 RTLD_LAZY表示调用时载入内存, 之后释放 RTLD_NOW 表示现在加载进入内存
    if(dlerror()!=NULL){
        printf("%s",dlerror());
    }
    void (*pointer)(参数列表) = dlsym(handler,"函数名"); // 函数指针记录函数
    if(dlerror()!=NULL){
        printf("%s",dlerror());
    }
    dlclose(handler);
    return 0;
}
```

***

