# -Java-Mitch-
SpringBoot+Uni-app前后端分离架构，实现常见的数十种文档格式的转换
# Mitch文档转换
#### Description
文档格式转换，支持常见10种文件格式的上传，可转换的类型有数十种；
保存的方式有两种：本地保存、二进制流返回
部分文档可以看到转换进度（websocket）
示例网站：http://www.mitchconvert.top/
本项目做了简单的用户登录、注册和保存转换记录的逻辑；如果只想看转换文档的代码，直接看utils包下microsoft包里面的代码。


不管什么格式的文档，核心的文档转换代码只有一句：ConvertFileHandler.buildChainByNoSave(fileBo);其中fileBo是ConvertFileBo实体类的实例对象。

# 网址示例

### 登录页：

登录页提供两种登录方式：密码登录和邮箱验证码登录；新用户在首次访问页面时，系统会提示使用邮箱验证码方式进行注册性登录；当用户登录后主动设置了初始密码即可使用密码进行登录：

 
![输入图片说明](md-img/clip_image002.jpg)
当用户密码试错达到5次时，锁定账号15分钟：

![输入图片说明](md-img/clip_image004.jpg)

### 转换页：

登录成功后，系统自动跳转到转换页。页面由两部分构成，上部分显示用户的转换纪录，下部分则是主要的转换功能区域。用户根据上传的文档种类选择对应的文档区，通过点击上传区域或将文件拖到上传区域即可上传文件，然后选择要转换类型，再点击确定即可（部分有转换进度显示）：

![输入图片说明](md-img/clip_image006.jpg)

当转换完成后，会跳出下载后文档的弹框，此时可以进行保存到本地：

![输入图片说明](md-img/clip_image008.jpg)

 

  以下通过一个示例演示将一个excel文件按xlsx->docx->pdf->pptx->png的顺序来转换得到每个文件的展示：

1）excel源文件图片示例（有两个sheet页）：

![输入图片说明](md-img/clip_image010.jpg)

2）转成docx示例：

![输入图片说明](md-img/clip_image012.jpg)

3）再将转换得到的docx文件转成pdf：

![输入图片说明](md-img/clip_image014.jpg)

4）再将转换得到的pdf转成pptx：

![输入图片说明](md-img/clip_image016.jpg)

5）最后将转换得到的pptx转成png图片(压缩包形式)：

![输入图片说明](md-img/clip_image018.jpg)

 

### 个人中心页：

个人中心页是提供给用户修改用户信息、密码，提交问题反馈或建议的页面；当登录的角色是管理员时，在该页面还会多出几个管理员按钮，是用来管理用户、查看项目的统计信息的：

![输入图片说明](md-img/clip_image020.jpg)

修改个人信息和登录密码，提交问题反馈示例图：

![输入图片说明](md-img/clip_image022.jpg)

 

提交反馈后管理员和用户双方可见的示例图：

![输入图片说明](md-img/clip_image024.jpg)

用户管理模块则是分页显示系统用户的信息，可对用户进行单个禁用或修改权：

![输入图片说明](md-img/image-20230530144535480.png)

访问量统计是用来简要统计项目的使用情况，统计的种类包括所有用户转换统计、用户在线数量、用户注册量按日期统计等统计信息：

![输入图片说明](md-img/clip_image028.jpg)

# 工程简介
一、本项目整体的文档转换流程介绍（伪代码）：  
    1、可上传的文档有四大类：words、pdf、excel、ppt（注意这里每一大类可上传的文档后缀可能有多种），
将这四大类构建成一个责任链的调用形式，调用的入口即为抽象类ConvertFileHandler，入参是ConvertFileBo
的实体类，里面包含转换的文档convertFile、要转换的类型chooseType、选择的四大类型之一convertType，
ConvertFileHandler有两个抽象方法：一个是转换完保存到指定的本地路径，另一个是转换为二进制流交给前端（
本项目是选择后一种，因为部署在安装手机上的缘故，手机内存不多，所以才选择这种方式）。第一个链是校验文件
格式、大小，通过校验后才交给所对应的链去处理。
    2、每一个链都会调不通的文档转换工具类，比如ExcelToChain链回去调ExcelToOther工具类进行转换，
以此类推..
    3、每个工具类里面都有四个静态的集合：
    1）allowedConvertType：允许转换的类型-->指的是chooseType
    2）isImageType：转换的类型是否是图片类型（原因：转换后文件数量非单一个（所以需要打成压缩包），转换方式可能跟其他文档不太一样）
    3）isZipType：转换后需要打成压缩包返回的 “允许转换的类型”，不同的工具类转完了以后不单单只有图片类型
可能会产生多个文件，所以用这个来标识。
    4）allowedSourceType：允许上传的文件的格式，比如说words支持上传doc、txt、md等等..
    3、转换步骤：加载配置，去除水印和页数限制；创建文档对象；创建临时保存的目录；设置保存选项（里面有转换进度回调，通过webSocket返回给前端）；
调用文档对象的保存方法执行转换；

二、注意事项：若部署在Linux下时，需要注意该环境下会缺失一些window的字体，导致转换时
报错或乱码，需要做如下操作：
（1）在/usr/share/fonts/下创建一个目录存放Windows字体
```
# mkdir /usr/share/fonts/winfonts/
```
（2）将字体上传到创建好的目录并解压
```
# cd /usr/share/fonts/winfonts/
# tar xf Windows_fonts.tar.gz
```
（3）建立字体索引信息，更新字体缓存
```
#安装sudo apt-get install ttf-mscorefonts-installer
mkfontscale
mkfontdir
#安装sudo apt-get install fontconfig
fc-cache -fv
fc-cache -rv 是强制刷新
```
（4）让字体生效
```
# source /etc/profile
```
（5）再次查看字体,如fc-list nofound 则执行sudo apt-get install fontconfig
```
# fc-list  |wc -l
```

# 延伸阅读
如果对您有帮助，希望您能给我一个小心心starts，感激不尽！
