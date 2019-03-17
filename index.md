## Welcome to GitHub Pages

You can use the [editor on GitHub](https://github.com/anlongstory/anlongstory.github.io/edit/master/index.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/anlongstory/anlongstory.github.io/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.


blob.cpp：  
explicit 作用 —— 30行开始

this 指针  
虚函数，纯虚函数，overrider,overload ，多态

用 type 定义函数指针

typedef int (*BrewFunction)(); （caffe.cpp line66）
ReadProtoFromTextFile() 对比 solver 与 proto 中的定义是否一致，并且将设置赋给对应的 SolverParamter 对象


##  net.cpp

### NetParameter
NetParameter 类含有：
NetParameter param
name: 层的名字
input: 输入
input_shape ： 输入尺寸
input_dim : 输入维度
state : NetState 类，当前有 "phase","level","stage" 三种状态值，根据状态判断和层中的 Include/ exclude 来包含/排除 某些层
force_backward ： 网络是否迫使每一层都反向操作
debug_info : 打印网络调试信息，前向，反向，更新梯度等信息
layer : 网络中的某一层的信息，属于 LayerParamter 类，param.layer(i)，获得第 i 层信息


### LayerParamter 
LayerParamter  类的变量有以下成员变量:
基本上我们在 train.prototxt 和 deploy.prototxt 中的定义和选择的参数都是在这里面定义的
LayerParameter layer\_param；

name： 层的名字 layer\_param.name() 返回
type:  层的类型  "Data","Convolution","ReLu" 等，对应的就是部分层需要额外设置一些参数，例如 convolution_param 设置卷积核大小等

bottom: 此层对应的的 输入 Blob 的名称
top: 此层输出 Blob 的名称
phase: "train" or "test"
propagate_down : 是否进行反向传播

### NetState

当前有 phase， level , stage 三种，主要是用来结合 NetStateRule 来确认是否包含网络层

phase: 只包含 {TRAIN,TEST}
level: 整型数，在 NetStateRule 类中有设置 min_level 和 max_level，当前网络状态在 [min_level，max_level]中即可
stage: 字符串类型的变量，NetStage 中的stage 要包含 NetStateRule 类中列出的所有stage，并不包含任何一个 not_stage

上述主要是通过： StateMeetsRule（） 函数来判断，两者之间是否满足条件的， FilterNet() 则是依据 StateMeetsRule（） 提出来判断是 include 还是 exclude 当前的层



#### AppendTop() 函数中

top_vecs_ ： vector中包含vector 里面存储的是指向输出的 blob 的指针
top_id_vecs_ ： vector中包含vector 里面存储的是 top_vecs_ 中对应的 blob 编号，0，1，2等

其中检测三种情况：
1. 是否是 原位计算，则不需要新建 blob 指针，直接在当前 blob_ 中索引对应的 id 等信息
2. 输出的名字是否重复，直接报错，判断的方法是 blob_name_to_idx 这个map 的指针不为空指针，依次遍历 blob_name 找当前层的名字，如果找到了，说明又不是 inplace计算，就是重复了
3. 正常输出，则新建新的 blob 指针，然后依次加入到 blob_ 中，在从 blob_ 中加入到 top_vecs_ 中，新建一个 available_blobs 插入一个


#### AppendBottom() 函数中

bottom_vecs_ ： vector中包含vector 里面存储的是指向 当前已经使用的 bottom blob 的指针
bottom_id_vecs_ ： vector中包含vector 里面存储的是 bottom_vecs_ 中对应的 blob 编号，0，1，2等
bottom_need_baclward ： 是否反向传播

就是检测当前 avaliable_block 中可用输入 blob，用一个 available_blobs 擦出一个


共有的：  
blobs_ ： vector 包含了所有的 Blob ,以指针形式保存，包含了 输入与输出，top 和 bottom 都是从中取的
blobs_id : vector, 包含blobs_ 中对应的 ID
blob_names : vector，包含了每个 blob 的名字，"data","label"，"conv1" 等
blob_need_baclward : vector ，包含 bool 型，是否要反向传播 

available_blobs ： 已有名字的集合

#### AppendParam() 函数中

params_ ： 参数 blob，从 blob_ 中选取的，例如卷积层的 blob，而 ReLU 层的就没有学习参数，就不会加入进来
param_id_vecs_ ： 存储 net_param_id

包括处理当前层是否是共享的，如果是共享的

## data_layer.cpp

### Datum  

数据类，包含了：

int32 类型： channels,height, width, label
float: float_data 可以保存浮点数
bytes data: 存的像素值


Base_data_layer 中定义了两个类， BaseDataLayer，BasePrefetchingDataLayer

BaseDataLayer  派生于  Layer

BasePrefetchingDataLayer 派生于 BaseDataLayer

DataLayer 派生于  BasePrefetchingDataLayer ，从 lmdb 中读取
ImageDataLayer 派生于  BasePrefetchingDataLayer ， 读取图片
WindowDataLayer 派生于 BasePrefetchingDataLayer，从图像文件的窗口获取数据，需要指定窗口数据文件

以上主要实现的是 LayerSetUp() 函数， loadbatch（）函数， LayerSetUp 主要是根据输入的 blob 预读取数据，loadbatch 则是将数据加入到线程中，参与运算

### ImageDataParameter 

主要是包含：

string source: 数据源路径
uint32 batch_size:  
uint32 rand_skip ： 抽取 data 时， 随机跳跃
uint32 new_height : resize 后的高
uint32 new_width : resize 后的宽
bool is_color ： true 是彩色， false 是灰度 
bool shuffle : 每个 epoch 是否混洗图片数据


包含至少一个纯虚函数的类就是抽象类，属于接口，只是为了给其他类提供一个可以继承的适当的基类，抽象类不能被用于实例化对象，并且纯虚数在派生类中必须要重载