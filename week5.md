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

$ ./a.out|head -n 30 
0.005768
0.0061037
0.00509659
0.00503555
0.00408948
0.0037843
0.0029603
0.00299081
0.00140385
0.00225837
-0.000427259
0.000701926
-0.00149541
-0.0013123
-0.00134281
-0.00192267
-0.000640889
-0.000640889
0.000518815
0.00128178
0.00253304
0.00314341
0.00466933
0.00454726
0.00601215
0.00521867
0.00662252
0.00589007
0.00646992
0.00650044
```
&ensp;&ensp;可以观察到，该开源库实际上是把不同字长的数据都转成了float类型，实际上，这个File::Read()会调用一个它的一个重载方法，声明是:
```cpp
Error File::Read(uint64_t frame_number, void (*decrypt)(char*, size_t),std::vector<float>* output) 
```
&ensp;&ensp;笔者认为，对于所有字节长度都做这样的处理是会失去一定精度的，理想的实现并不多复杂。一个较为方便的实现是将所有的数据用char存储，在需要读取的时候强制转型就可以了，这里给出实现。
