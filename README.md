# DanmakuRender-3 —— 一个录制带弹幕直播的小工具（版本3）
结合网络上的代码写的一个能录制带弹幕直播流的小工具，主要用来录制包含弹幕的视频流。     
- 可以录制纯净直播流和弹幕，并且支持在本地预览带弹幕直播流。
- 可以自动渲染弹幕到视频中，并且渲染速度快。
- 支持同时录制多个直播。    

### 新版本已经发布到v4分支，如果录制遇到问题可以试试新版本
旧版本可以在v1分支和v2分支找到。   

**BUG：Python3.10及以上版本可能会导致部分平台录制弹幕出现问题，请使用Python3.7版本录制**               
2022.10.13更新：修改了API逻辑，而且现在可以支持录制不带弹幕的抖音直播        
2022.8.1更新：版本3.1和在线弹幕渲染的新功能（暂时只支持MDY的几个主播）        
2022.5.30版本更新：更新了版本3    
2022.2.9版本更新：主要修复了直播流错误时无法正常重启的情况    
2022.2.22更新：修复了弹幕闪烁的问题    
2022.2.28更新：增加不录制弹幕的功能

## 使用说明
### 前置要求
- Python 3.7+
- Python库 aiohttp,requests,execjs,lxml
- FFmpeg
- 满足条件的NVIDIA或者AMD显卡（也可以不用，但是渲染弹幕会很卡）    

### 安装
- 下载源代码
- 将ffmpeg.exe移动到`tools`文件夹下（或者直接启动程序让他自己安装）    

### 简单实例
- `python main.py -u https://www.huya.com/712416` 录制虎牙712416直播间，其他选项默认，并启动自动渲染弹幕
- `python main.py -u https://www.huya.com/712416 --disable_auto_render` 只录制直播流，不渲染弹幕
- `python main.py -u https://www.huya.com/712416 --gpu amd` 在渲染时使用AMD硬件编码器
- `python main.py --render_only` 只渲染弹幕，这里会渲染录像文件夹里所有没渲染的文件，如果因为程序故障有些视频在录制时没有渲染完成就可以使用这个命令重新渲染    

### 注意事项
- 程序的工作流程是：先录制一小时直播，然后在录制下一小时直播时启动对这一小时直播的渲染。录制完成后可以同时得到直播回放和带弹幕的直播回放（分为两个视频）
- 程序默认使用NVIDIA的硬件编码器渲染，如果用A卡的话需要另外附带参数。如果不渲染弹幕就不用管。
- 程序录制时默认录制原画（直播间最高画质），渲染弹幕时默认使用恒定质量渲染(q=28)，fps游戏直播大概一个小时6-7GB，聊天直播大概一个小时2-3GB，一个小时的视频大约需要渲染20分钟。
- 在关闭程序时，如果选择了自动渲染弹幕，则一定要等录制结束并且渲染完成再关闭（由于程序设定是先录制后渲染），否则带弹幕的录播会出问题。
- 如果因为配置比较差，渲染视频比较慢导致渲染比录制慢很多的，可以选择先不渲染弹幕，在录制结束后手动渲染。（这种情况比较少见，因为渲染的速度很快，我1060的显卡都可以同时录两个直播）
- 在录制的过程中弹幕保存为一个字幕文件，因此使用支持字幕的播放器在本地播放录播可以有弹幕的效果（**就算是没渲染弹幕也可以！**），拿VLC播放器为例，在播放录像时选择字幕-添加字幕文件，然后选择对应的ass文件就可以预览弹幕了。 

### 在线弹幕渲染
现在对MDY的几个主播支持在线弹幕渲染。简单地说就是可以给现有的录像文件加上弹幕，这个录像文件可以是你自己用其他工具录制的，这里只要求录像文件是连续的，不能经过剪辑。程序会自动从我的服务器下载弹幕文件然后渲染。
使用方法：
- `python render_online.py <本地视频地址> <主播名称> <视频开始时间>`，其中本地视频地址就是你需要渲染的视频位置，例如`./TEST.mp4`，主播名称为兼容的主播名称，视频开始时间为视频开始的北京时间，例如`2022.8.1,19:0:0`（注意中间不能有空格！）
- 目前兼容的主播：飞天狙、甜药、卡莎、幽灵
- 更多参数可以参考下面的详细参数        

### 详细说明
程序运行时可以附带以下参数
#### 录制参数
- `-u` 指定录制链接，多个链接用英文逗号隔开
- `-s` 指定视频分块长度（单位：秒），默认为3600，设置为0表示不分块（建议分块，不然一个渲染要等直播结束才开始）
- `--video_dir` 指定视频保存文件夹，默认为此目录下的'直播回放'文件夹
- `--dm_dir` 指定弹幕保存文件夹，默认和视频保存文件夹一样
- `--render_dir` 指定渲染出的带弹幕视频保存文件夹，默认为此目录下的'直播回放（带弹幕）'文件夹
- `--gpu` 指定显卡类型，可以为AMD或者NVIDIA，设置为none表示不使用显卡辅助编码，**默认使用NVIDIA显卡**，如果手动设置了编码器参数则此项无效   
- `--disable_auto_render` 禁用自动渲染，默认false
- `--render_only` 自动渲染录像文件夹里的视频 

特别地，可以使用以下参数单独指定一些录制参数和渲染参数，参数将会覆盖GPU默认选择    
- `--ffmpeg_stream_args` 指定ffmpeg录制时的HTTP参数，默认`-fflags,+discardcorrupt,-reconnect,1,-reconnect_streamed,1` 
- `--hwaccel_args` 指定硬件加速参数，多个参数之间用','分割。N卡默认`-hwaccel,cuda,-noautorotate`，A卡没有
- `--vencoder` 指定视频编码器，NVIDIA显卡默认为H264_NVENC，AMD显卡默认为H264_AMF，不使用硬件加速的话默认为libx264
- `--vencoder_args` 指定视频编码器参数，多个参数之间用','分割，指定此参数必须先指定vencoder参数。h264_nvenc默认为`-cq,27`，libx264默认`-crf,25`，h264_amf默认`-b:v,15M`
- `--aencoder` 指定音频编码器，默认为AAC
- `--aencoder_args` 指定音频编码器参数，多个参数之间用','分割。默认为`-b:a,320K`   

如果程序无法正常判断流的分辨率和帧率（例如部分主播使用了4K120Hz的超高清流）导致录制错误，可以使用以下参数强行指定     
- `--resolution` 指定分辨率，分辨率应该使用x分割，例如1920x1080

#### 弹幕参数
- `--dmrate` 指定弹幕占屏幕的最大比例（即屏幕上半部分有多少可以用来显示弹幕），默认为0.4
- `--margin` 指定弹幕行距，默认6
- `--font` 指定弹幕字体，默认为微软雅黑字体(`Microsoft YaHei`)
- `--fontsize` 指定弹幕字体大小，默认为36
- `--overflow_op` 指定过量弹幕的处理方法，可选ignore（忽略过量弹幕）或者override（强行叠加弹幕），默认override
- `--dmduration` 指定单条弹幕持续时间（秒），默认为16
- `--opacity` 指定弹幕不透明度，默认为0.8
- `--resolution_fixed` 使用自适应分辨率模式，也就是说弹幕大小会随分辨率变化而变化，前面设置的是1080P下的大小，默认true

#### 其他参数
- `--debug` 使用debug模式，将录制信息输出到控制台，新版本不建议使用，错误信息会自动保存为日志文件。
- `--disable_lowspeed_interrupt` 关闭编码过慢自动重启功能（编码速度低说明录制故障或者是资源不足了，新版本估计没啥用）
- `--disable_danmaku_reconnect` 禁用弹幕重连，默认false（如果录制的主播弹幕很少应该禁用） 
- `--flowtype` 选择流类型，可选flv或者是m3u8，默认flv，此选项仅对B站流生效，在flv流故障或者录制超高清直播时应该设置为m3u8
- `-V` 查看版本号

#### 在线弹幕渲染参数
大部分上面的参数可以通用，这里只说一些新参数
- `-o` 输出文件名称，默认为输入文件在后面加一个“带弹幕版”
- `--remote_addr` 远程服务器地址
- `--video_duration` 视频长度，有可能程序无法自动判断视频长度，然后就需要自己指定。

## 更多
感谢 THMonster/danmaku, wbt5/real-url, ForgQi/biliup     
出现问题了可以把日志文件发给我，我会尽量帮忙修复
