2026年初，整理一下android app开发者，新windows电脑需要的软件。将

软件列表：一些插件标注AI加粗。

| windows                       | MacOS                                                        | 备注                                                         |
| ----------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| VSCode                        | 同                                                           | 一些非常规项目浏览                                           |
| beyond compare                | 同                                                           | 对比工具                                                     |
| WinHex                        | Hex fiend                                                    | 二进制浏览工具                                               |
| 7zip                          | The unarchiver                                               | 解压软件                                                     |
| jadx/jd-gui                   | 同                                                           | github搜索下载，反编译查看工具                               |
| mat                           | 同                                                           | 独立的内存分析工具<br />http://www.eclipse.org/mat/downloads.php |
| typora                        | 同                                                           | markdown编写工具                                             |
| powertoys微软出品多功能小工具 | Paste 复制粘贴程序<br />LaunchNext mac新系统没有launch<br />Dozer 系统状态栏收纳 | 系统工具                                                     |
| proguard                      | 同                                                           | 混淆工具                                                     |
| 微信输入法                    | 同                                                           | 多端同步                                                     |
| android studio                | 同                                                           | 下对芯片                                                     |
| zulu JDK17                    | 同                                                           | 比oracle方便下载。                                           |
| vlc播放器                     | 同                                                           |                                                              |
| edge浏览器                    | 同                                                           | 国内同步标签比chrome好用，与chrome相似性99%                  |
| IDEA社区版                    | 同                                                           |                                                              |
| notepad++                     | notepadNext/vsCode                                           | 文本编辑，大文件日志分析                                     |
| SpaceSniffer                  | DaisyDisk                                                    | 磁盘工具，小工具大用途                                       |
| `trae`/`cursor`等                   | 同                                                           | AI编程工具                                                   |
| gitBash                       | 自带terminal                                                 |                                                              |
| `warp          `                | 自带terminal                                                 | AI终端                                                       |

git config：

> gitbash 打开后：cat ~/.gitconfig

```shell
[user]
	name = allan
	email = xx@xx.com
[alias]
	st = status
	ss = show --stat --stat-name-width=200 --stat-graph-width=5
	co = checkout
	cp = cherry-pick
	lg = log --graph
	cm = commit
	br = branch
[pull]
	rebase = true
[color]
	ui = auto
[commit]
	template = /Users/allan/.commit.template

```

idea插件：

|                                |                            |
| ------------------------------ | -------------------------- |
| dracula theme<br />xcode theme<br/> relax your eyes green theme <br/> github theme<br/> Codigrate相关主题  | 主题                       |
| APK mover                      | apk安装                    |
| color highlighter              | 检查代码行颜色显示成框框   |
| gitBashOpenHere                | 右键菜单追加打开到terminal |
| builder Generator              | 可选                       |
| rainbow brackets               | 可选，代码彩虹块           |
|  `通义灵码`，推荐`trae`等     | AI编程插件                 |
| koin dependency injection  (official) | 依赖注入 |
|translation yii.Guxing|翻译|

 Idea设置：

1. 搜索hard wrap，修改java，kotlin，xml的长度为160；
2. 字体设置: 搜索font：Appearence和Editor的Font；
3. 禁用Gemini(不仅要科学，而且request老失败)，禁用gitlab才能使用密码方式登录；
4. ai工具推荐trae，比灵码提示准，唯一缺点是无法选择代码片段到框中；
5. 修改背景色：找到"Editor" —> Color Scheme —> General —> Text ----> Default text，点击"Background"所对应的颜色框；
6. Experimental中将Configure all gradle tasks during gradle sync勾上，否则无法找到所有tasks；
7. 添加later：支持，Editor>TODO，点击加号，追加一个`\blater\b.*`的规则，Error stripe mark打开即可。

   > 1、绿豆沙 #C7EDCC RGB(199, 237, 204)
   >
   > 2、银河白 #FFFFFF RGB(255, 255, 255)
   >
   > 3、杏仁黄 #FAF9DE RGB(250, 249, 222)
   >
   > 4、秋叶褐 #FFF2E2 RGB(255, 242, 226)
   >
   > 5、胭脂红 #FDE6E0 RGB(253, 230, 224)
   >
   > 6、海天蓝 #DCE2F1 RGB(220, 226, 241)
   >
   > 7、葛巾紫 #E9EBFE RGB(233, 235, 254)
   >
   > 8、极光灰 #EAEAEF RGB(234, 234, 239)
   >
   > 9、青草绿 #E3EDCD RGB(227, 237, 205)
   >
   > 10、电脑管家 #CCE8CF RGB(204, 232, 207)
   >
   > 11、WPS护眼色 #6E7B6C RGB(110, 123, 108)
