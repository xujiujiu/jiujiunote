
前阵子因为电脑系统重装，导致nodejs环境需要重新配置，超级麻烦，所以这次把node的环境配置步骤记下来，下回再出现这种情况，只需要配置一下环境变量就搞定，省事。
1.	[node官网](http://nodejs.org/)下载windows版本的nodejs，进行安装`node`程序

2.	安装过程基本直接“NEXT”就可以了。（这边为了避免重装系统等事件影响，安装在非系统版，这里安装在`D:\Program Files\nodejs`）
3.	安装完成后可以使用cmd（win+r然后输入cmd进入）测试下是否安装成功。方法：在cmd下输入`node -v`，出现版本提示就是完成了NodeJS的安装。
![node版本](https://user-gold-cdn.xitu.io/2018/4/1/16281661f97bd97f?w=289&h=45&f=png&s=1069)
4.	由于新版的NodeJS已经集成了npm，所以nodejs安装完成时npm也一并安装好了。同样可以使用cmd命令行输入`npm-v`来测试是否成功安装。
![npm版本](https://user-gold-cdn.xitu.io/2018/4/1/16281685ef553aca?w=361&h=48&f=png&s=1103)
5.	常规NodeJS的搭建到现在为止已经完成了，急不及待的话你可以在”cmd“输入`node`进入node开发模式下，输入你的NodeJS第一句：”hello world“ - 在命令窗口输入：`console.log('hello world')`。

![hello world](https://user-gold-cdn.xitu.io/2018/4/1/162818f08db8d230?w=356&h=81&f=png&s=1633)
6.	自定义npm全局路径

- 我们要先配置npm的全局模块的存放路径以及cache的路径，例如我希望将以上两个文件夹放在NodeJS的主目录下，便在NodeJs下建立`node_global`及`node_cache`两个文件夹。如下图

![npm全局路径](https://user-gold-cdn.xitu.io/2018/4/1/162816e0b48e8928?w=380&h=283&f=png&s=8561)
- 启动cmd，输入以下两条命令，成功后之后通过npm全局安装的包都会存放到`node_global`文件夹下，后续查找包较方便。

>`npm config set prefix "D:\Program Files\nodejs\node_global"`  
>`npm config set cache "D:\Program Files\nodejs\node_cache"`

![npm全局路径配置](https://user-gold-cdn.xitu.io/2018/4/1/162817165dd2685b?w=501&h=83&f=png&s=2949)

- 测试一下，安装一个`vue-cli` 包，输入`npm install -g vue-cli`命令，安装成功后会提示已安装到`D:\Program Files\nodejs\node_cache`文件夹中

![vue-cli安装](https://user-gold-cdn.xitu.io/2018/4/1/162817af1633c8b9?w=570&h=253&f=png&s=8550)

 7. `npm`的全局路径配置完成了，现在开始**配置系统环境变量**
- 打开系统对话框，“我的电脑”右键“属性”-“高级系统设置”-“高级”-“环境变量”。
- 进入环境变量对话框，在 ***系统变量*** 下新建`NODE_PATH`，输入`D:\Program Files\nodejs\node_global\node_modules`
- 由于改变了`modules`的默认地址，所以上面的用户变量都要跟着改变一下（***用户变量*** `PATH`的值修改为`D:\Program Files\nodejs\node_global\`），要不然使用`module`的时候会导致输入命令出现`xxx不是内部或外部命令，也不是可运行的程序或批处理文件`这个错误。
- 因为这里配置的环境变量涉及全局，所以配置完成后需要 **重启计算机**。

妈妈再也不用担心我的电脑更换系统了，只要不更换`D盘`（安装`node`的磁盘），后续更换系统只需按步骤重新配置环境变量即可。[~_~]







