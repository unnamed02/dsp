# 第五周工作小结

&ensp;&ensp;.wav文件直接保存存储音频的时域信息。在本实验中，笔者更关心采样频率，采样位数等信息，值得注意的是，由于.wav文件一般会有1，2，3，4字节长，而一般编程语言中并无3字节的基本类型，这将会是工程实现中的一个难点。   
&ensp;&ensp;在GNU(Linux)/C++平台上，目前并没有较为成熟又轻量级的语音识别库，笔者在Github考察了几个.wav文件读取库，并尝试下载到本地编译运行，读了几个本地的.wav文件，基本可以读出连续的采样序列并打印到控制台上。  
&ensp;&ensp;以下为文件及运行结果：

```cpp
#include <iostream>
#include <system_error>
#include "wave/file.h"

int main() {
  wave::File read_file;
  wave::Error err = read_file.Open("wave/test_resource/Untitled3.wav", wave::kIn);
  if (err) {
    std::cout << "Something went wrong in in open" << std::endl;
    return 1;
  }
  std::vector<float> content;
  err = read_file.Read(&content);
  if (err) {
    std::cout << "Something went wrong in read" << std::endl;
    return 2;
  }
  for(auto iter = content.begin();iter!= content.end();iter++){
    std::cout << *iter <<  std::endl;
  }
  return 0;
}

```
```shell
$ g++ test.cpp -I wave/build/install/include wave/build/install/lib/libwave.a

$ ./a.out|head -n 10
0.005768
0.0061037
0.00509659
0.00503555
0.00408948
0.0037843
0.0029603
0.00299081

```
&ensp;&ensp;可以观察到，该开源库实际上是把不同字长的数据都转成了float类型，实际上，这个File::Read()会调用一个它的一个重载方法，声明是:
```cpp
Error File::Read(uint64_t frame_number, void (*decrypt)(char*, size_t),std::vector<float>* output) 
```
&ensp;&ensp;该函数实际是将该采样点数据除以该字长长所能表示的最大值得到的float存入vector，因为最大值是二的整幂次，因而整个数制变换可以描述为二进制取原码存入float尾数，取对应的指数和符号位。因为float有32位，其中有23位尾数，那么在表示32位数据时是有一定损失的(24位时恰好不会有损失，因为float有单独的符号位)，在处理采样字节为4的.wav文件时，应当使用double。  
&ensp;&ensp;原开源工程并没有做double对应的实现，笔者自己实现了一份，详见对应的commit :
[add double](https://github.com/unnamed02/dsp/commit/0f32d5e3432d2b878e840d5d76425e2513b8e75f)   
&ensp;&ensp;这里比较下16位长时存float和doube的读取结果，据前面的分析，结果应当是相同的:
```shell
$ ./test_float|head -10 
running File::Read(vector<float>*)
data is 16 bites long
0.005768
0.0061037
0.00509659
0.00503555
0.00408948
0.0037843
0.0029603
0.00299081
```
```shell
$ ./test_double|head -10
running File::Read(vector<double>*)
data is 16 bytes long
0.005768
0.0061037
0.00509659
0.00503555
0.00408948
0.0037843
0.0029603
0.00299081
```