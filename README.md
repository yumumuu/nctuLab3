# Route Configuration

This repository is a lab for NCTU course "Introduction to Computer Networks 2018".

---
## Abstract

In this lab, we are going to write a Python program with Ryu SDN framework to build a simple software-defined network and compare the different between two forwarding rules.

---
## Objectives

1. Learn how to build a simple software-defined networking with Ryu SDN framework
2. Learn how to add forwarding rule into each OpenFlow switch

---
## Execution

> TODO:
> * How to run your program?
    使用mininet還有Ryu Controller
> * What is the meaning of the executing command (both Mininet and Ryu controller)?


`mn --custom SimpleTopo.py --topo topo--link tc --controller         remote`
    custom:設定路徑
    topo:設定拓墣名稱
    link:對連線條件進行設定 這裡使用tc
    
    
`sudo ryu-manager SimpleController.py --observe-links`
    用ryu-manager執行`SimpleController.py`
    observe-link:開啟終端機
`h1 iperf -s -u -i 1 –p 5566 > ./out/result1 &`
    -s:server -u:UDP -i:參數 1:設定得參數 -p:port 5566:設定的port >./out/result1 &:設定的路徑位置 
    
`h2 iperf -c 10.0.0.1 -u –i 1 –p 5566`
    -c:client 10.0.0.1:位置 -u:UDP -i:參數 1:設定得參數 -p:port 5566:設定的port
    
> * Show the screenshot of using iPerf command in Mininet (both `SimpleController.py` and `controller.py`)
result of `contriller.py`
![](https://i.imgur.com/Zdq9YnU.jpg)
result of `SimpleContriller.py`
![](https://i.imgur.com/kX0LKFC.jpg)


---
## Description

### Tasks

> TODO:
> * Describe how you finish this work in detail

1. Environment Setup
    先加入github Classroom
    之後用SSH登入我的container 再clone我的GitHub repository
    之後再test mininet
2. Example of Ryu SDN
    開兩個terminal跑 一個用 
    `sudo mn --custom SimpleTopo.py --topo topo--link tc --controller remote`跑`SimpleTopo.py`
    另一個用 `sudo ryu-manager SimpleController.py --observe-links`
    跑 `SimpleController`
    就只是跑跑看

3. Mininet Topology
    依照topo.png寫一個`topo.py` 接著開兩個terminal跑 一個用 
    `sudo mn --custom SimpleTopo.py --topo topo--link tc --controller remote`跑`topo.py`
    另一個用 `sudo ryu-manager SimpleController.py --observe-links`
    跑 `SimpleController`
    測試看看`topo.py`


4. Ryu Controller
    依照topo.png寫一個`controller.py` 
5. Measurement
    接著開兩個terminal跑 一個用 
    `sudo mn --custom SimpleTopo.py --topo topo--link tc --        controller remote`
    跑`topo.py`
    另一個用 `sudo ryu-manager controller.py --observe-links`
    跑 `controller.py`
    之後再用
    `h1 iperf -s -u -i 1 –p 5566 > ./out/result1 &`
    `h2 iperf -c 10.0.0.1 -u –i 1 –p 5566`
    測試結果 result1放topo跟SimpleController的結果
    result2放topo跟controller的結果

### Discussion

> TODO:
> * Answer the following questions

1. Describe the difference between packet-in and packet-out in detail.
   對於接收到的封包進行轉送到 Controller 的動作（ Packet-In ）。
    對於接收到來自 Controller 的封包轉送到指定的連接埠（ Packet-Out ）。
2. What is “table-miss” in SDN?
    一個Flow Table會包含多個Flow Entry，而一個Flow Entry事實上包含比對欄位、優先序、計數器、執行動作、逾時時間、Cookie等資料。其中最重要的是前面兩個資料，也就是比對欄位以及優先序。 優先序數值從0開始，數字越小代表優先序越高，所以如果設定某個Flow Entry是優先序為零，而且比對欄位是什麼都通過的話，等於就是把這筆Flow Entry設定為Table-Miss Flow Entry
   
3. Why is "`(app_manager.RyuApp)`" adding after the declaration of class in `controller.py`?
    因為app_manager是Ryu 最主要的一個部分，Ryu 都是需要繼承它，當作一切的開端
   
4. Explain the following code in `controller.py`.
    ```python
    @set_ev_cls(ofp_event.EventOFPPacketIn, MAIN_DISPATCHER)
    ```
    負責 Packet-In 事件
    在 Switch 與 Ryu 完成交握的狀況下執行
    也代表著，只要更換參數，就能管理 OpenFlow 所提供的事件，也因此可以將管理邏輯，由事件帶入整體管理環境中。

    接下來，我們看看事件內部的運作該怎麼規劃。首先，介紹透過事件提取出來的參數：

    ev.msg：代表 Packet-In 傳入的資訊（包含 switch、in_port_number         等...）
    msg.datapath：在 OpenFlow 中，datapath 代表的就是 Switch
    datapath.ofproto、datapath.ofproto_parser：取出此 Switch 中，使用的 OpenFlow 協定及 Switch 與 Ryu 之間的溝通管道。
    接下來，介紹執行的部分。在接收到 Packet-In 的事件時，我們規劃的動作        （Action）是將此封包進行 Flooding，也就是傳往除了 in_port 外的所有 port     上。我們透過ofp_parser建立此動作：

    `actions = [datapath.ofproto_parser.OFPActionOutput(out_port)]`
    這裡使用到的OFPActionOutput，會依給定的參數而將封包傳往指定 port 上。例    如：ofproto_parser.OFPActionOutput(1)，則代表將封包傳往 port 1。因此在這裡    我們使用out_port，規劃封包轉送到除了 in_port 以外的所有 port 上。

    再透過產生一個 Packet-Out 事件，將我們要傳出的訊息建立好：

    `out = ofp_parser.OFPacketOut(datapath=dp,                           buffer_id=msg.buffer_id, in_port=msg.in_port, actions=actions)`
    最後使用datapath.send_msg(out)將此封包傳送至 Switch，Switch 會依此訊息， 進行封包轉送、分配
5. What is the meaning of “datapath” in `controller.py`?
    datapath : the switch in the topology using OpenFlow
   
6. Why need to set "`ip_proto=17`" in the flow entry?
        代表將IP通訊協定設為UDP
   
7. Compare the differences between the iPerf results of `SimpleController.py` and `controller.py` in detail.
   多收到6個ＡＣＫ
   
8. Which forwarding rule is better? Why?
    controller的rule比較好，因為收到比較多ack。

---
## References

> TODO: 
> * Please add your references in the following

* **Ryu SDN**
    * [Ryubook Documentation](https://osrg.github.io/ryu-book/en/html/)
    * [Ryubook [PDF]](https://osrg.github.io/ryu-book/en/Ryubook.pdf)
    * [Ryu 4.30 Documentation](https://github.com/mininet/mininet/wiki/Introduction-to-Mininet)
    * [Ryu Controller Tutorial](http://sdnhub.org/tutorials/ryu/)
    * [OpenFlow 1.3 Switch Specification](https://www.opennetworking.org/wp-content/uploads/2014/10/openflow-spec-v1.3.0.pdf)
    * [Ryubook 說明文件](https://osrg.github.io/ryu-book/zh_tw/html/)
    * [GitHub - Ryu Controller 教學專案](https://github.com/OSE-Lab/Learning-SDN/blob/master/Controller/Ryu/README.md)
    * [Ryu SDN 指南 – Pengfei Ni](https://feisky.gitbooks.io/sdn/sdn/ryu.html)
    * [OpenFlow 通訊協定](https://osrg.github.io/ryu-book/zh_tw/html/openflow_protocol.html)
* **Python**
    * [Python 2.7.15 Standard Library](https://docs.python.org/2/library/index.html)
    * [Python Tutorial - Tutorialspoint](https://www.tutorialspoint.com/python/)
* **Others**
    * [Cheat Sheet of Markdown Syntax](https://www.markdownguide.org/cheat-sheet)
    * [Vim Tutorial – Tutorialspoint](https://www.tutorialspoint.com/vim/index.htm)
    * [鳥哥的 Linux 私房菜 – 第九章、vim 程式編輯器](http://linux.vbird.org/linux_basic/0310vi.php)

---
## Contributors

> TODO:
> * Please replace "`YOUR_NAME`" and "`YOUR_GITHUB_LINK`" into yours

* [Yumumuu 莊于萱](https://github.com/nctucn/lab3-yumumuu)
* [David Lu](https://github.com/yungshenglu)

---
## License

GNU GENERAL PUBLIC LICENSE Version 3
