# [先上github地址](https://github.com/Pryriat/BandwagongVPS_controller "先上github地址")

# 序


- 其实做这个软件的动机很简单...没人做
- Bandwagongvps是一家国外的VPS商家，价格便宜量又足，匿名购买还不怕查水表。把这个博客和扶墙梯子放上VPS后，自然要不时地查看VPS的负载情况~~苦逼运维~~。在手机端上有很方便的VPS管理软件叫做[Bandwagon 控制台](https://play.google.com/store/apps/details?id=com.sukaiyi.bandwagonvps "Bandwagon 控制台")，可以查看服务器实时负载信息，对服务器进行基本的管理，如重装系统、重启、shell等。就像一把瑞士军刀，麻雀虽小五脏俱全。
  ![](http://ozhtfx691.bkt.clouddn.com/Bwh_controller/ad_bwh.jpg)
- 手机上都有这么方便的玩意，电脑上就更不用说了吧，github走你
  ![](https://tinytracer.com/wp-content/uploads/2018/04/e22208bbd846fbdde60b3fb9d61b18bc.png)
  ....一个安卓的，一个网页端


# ~~为毛没有PC的~~


- 秉承自动动手丰衣足食的原则~~还不是因为没人做~~，我陷入了沉思，然后打开了VPS的网页端控制界面
  ![](http://ozhtfx691.bkt.clouddn.com/Bwh_controller/api.png)
  无论是获取信息，还是重装系统，亦或是basic shell，都可以通过一个简单的get请求搞定。用python+pyqt，requests库处理请求，json库解析返回的json格式数据，pyqt将数据转化成图像和界面，perfect。思路有了，开搞！


# 起


- 写这个客户端给自己用和给别人用最大的区别在于给自己用时可以将VPS的VEID和KEY写死在代码里，用的时候简单方便；但若要分享出去，则需要一个登陆窗口获取相应的信息，然后进行加密储存在本地以避免明文泄露。因此，首要的部分是一个登陆窗口。
  ![](http://ozhtfx691.bkt.clouddn.com/FSQ$U%7B%5DEQ@5_CF%7BY%288%7DTTLC.png)
- 登陆完成后需要关闭登陆界面，关闭时要注意资源释放
- 如果本地有储存的帐号信息，则跳过登陆界面

# 承


- 有了请求API所需的必要信息就可以对服务器进行请求获取数据了。服务器返回值为JSON格式，需要用requests库的json方法将其转化为dict类型进行解析
  ![](http://ozhtfx691.bkt.clouddn.com/1522210984%281%29.png)
- 在计算百分比时要注意整型与浮点型的转换
- 关键字段封装成list对返回的dict进行迭代，简化代码。遇到特殊字段特殊处理
- 存、磁盘等占用过高时需要改变进度条的颜色


# 转


- IWIVM的请求API不仅可以查看信息，还能对VPS进行修改。服务器返回依旧为JSON格式，通过'error'字段查看修改操作是否成功
  ![](http://ozhtfx691.bkt.clouddn.com/1522210984%281%29.png)
- 个shell指令都是从默认目录开始执行，即先执行cd后不影响下一条指令的目录（复杂的操作通过ssh客户端执行）
- shell窗口的作用为简单的信息查看（如free、ps等)


# 合


- 上三个窗口分别封装为三个类，通过一个主类联系在一起（松耦合），因此主类的主要任务是确定程序的执行流程（是否显示登陆界面）、信息窗口和控制窗口的初始化、托盘的创建及退出后的资源释放。

```python
claass mainwindow(QWidget):
    def __init__(self,TAR,head,web_payload,parent=None):
        super().__init__()
        self.tray = QSystemTrayIcon(self)
        #self.timer = QTimer(self)
        #self.timer.timeout.connect(self.warn)
        #self.timer.start(20*1000)
        self.quitAction = QAction("&Quit",self,triggered=qApp.quit)
        self.showAction = QAction("&Show",self,triggered=self.showNormal)
        self.Tray_icon_menu = QMenu(self)
        self.Tray_icon_menu.addAction(self.showAction)
        self.Tray_icon_menu.addAction(self.quitAction)
        self.tray.setContextMenu(self.Tray_icon_menu)
        self.tray.setIcon(QIcon('final.ico' ))
        self.tray.activated.connect(self.db_cilcked)
        self.tray.messageClicked.connect(self.showNormal)
        self.tray.show()
        self.bwh_stat = bwh_stat(TAR,head,web_payload)
        self.bwh_control =bwh_controls(TAR,head,web_payload)
        self.tray.setToolTip("IP : %s\nRAM : %s%%\nSwap : %s%% \nBandwidth : %s%% "%(self.bwh_stat.info_data['ip_addresses'][0],str(self.bwh_stat.ram_stat_value),\
            str(self.bwh_stat.swap_stat_value),str(self.bwh_stat.data_usage_value)))
        self.setWindowTitle(self.tr("bwh_contorler"))
        self.setWindowFlags(Qt.MSWindowsFixedSizeDialogHint|Qt.Window)
        self.tabwidget = QTabWidget(self)
        self.tabwidget.addTab(self.bwh_stat,"bwh_stat")
        self.tabwidget.addTab(self.bwh_control,"bwh_control")
        self.vbox = QVBoxLayout()
        self.vbox.addWidget(self.tabwidget)
        self.setLayout(self.vbox)
        self.resize(640,480)
        self.setFixedSize(640,480)

    def hideEvent(self, event):
        self.hide()
    def closeEvent(self, QCloseEvent):
        self.tray.hide()
        return super().closeEvent(QCloseEvent)
    def db_cilcked(self,reason):
        if reason == 2:
            self.showNormal()
```
- 将不同窗口抽出并封装成类的另一好处在于增加了代码的灵活性，通过修改'web_payload'的值可以创建不同的窗口以获取不同服务器的信息，为日后的拓展（如多用户）留下空间


# 感想

- ~~似乎拿了github上基于pyqt构建的跨平台bwh客户端一血~~
- 欲望是第一生产力