# VMware

## 虚拟网络编辑 - nat模式
当虚拟机不能和宿主机相连的时候，可以打开虚拟网络编辑器，还原默认设置

然后再手动把指定的固定ip更改回来，要注意以下几个地方，都需要统一网段才行
![](/assets/image/container/snipaste_20180821_104912.png)

对于linux中的固定ip来说，一定记得 dns也需要手动，否则会外网链接不上
![](/assets/image/container/snipaste_20180821_110938.png)
