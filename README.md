# **Bài Lab Network xây dựng và cấu hình một Topology theo mô hình 3 lớp**
 
Bài lab này là đề tài mà các anh Mentor tại VNNIC(Trung Tâm Internet Việt Nam) đã lên ý tưởng và đưa cho tụi mình thực hiện. 

Topology của bài lab này là một mô hình mạng điển hình gồm 3 lớp chính là CORE, DISTRIBUTION và ACCESS. Mục tiêu của mô hình này là chú trọng đến khả năng dự phòng khi xảy ra sự cố mạng vì có thể thấy mỗi thiết bị luôn có một Active và Backup của nó, ngoài ra mô hình này cần sự bảo mật cơ bản như setup các policy cơ bản trên Firewall và thiết lập một VPN để kết nối vào mạng nội bộ, và cũng như thực hành về giao thức định tuyến động BGP giữa các nhà mạng.

Dưới đây là hình ảnh về Topology cũng như đề bài

![image](https://user-images.githubusercontent.com/72087963/231234786-17652c2e-9955-4fb5-9340-0c6b950559d5.png)

   Trước tiên để vào bài Lab, ta cần các kiến thức nền cơ bản của CCNA như:
* Routing và switching
* Các lệnh cấu hình cơ bản của Firewall Cisco ASA    
* Cấu hình Routing BGP(Định tuyến BGP có thể xem qua tại https://academy.vnnic.vn/bgp-nguyen-ly-hoat-dong-va-cau-hinh / https://academy.vnnic.vn/huong-dan-thuc-hanh-giao-thuc-bgp . Đây là một giao thức định tuyến khá phức tạp, phải mất khoảng 3-6 tháng để có thể học được giao thức định tuyến này, nên bài này mình chỉ giải thích những cái mình hiểu nhé).
* Các dịch vụ DHCP(Giao thức cấu hình IP động cho client), STP(Giao thức chống Loop cho switch ở Layer 2), VRRP(giao thức Gateway định tuyến dự phòng ở Layer 3), NAT( giao thức giao tiếp của IP mạng local ra ngoài bằng 1 IP khác,"mình cũng không biết giải thích như thế nào ^^"), LACP(giao thức liên kết các đường mạng vật lý thành 1 đường mạng ảo),... 
* Kiến thức về Linux quản trị Server(ở đây ta có 2 server là DNS và Web), quy hoạch IP,... và một số kiến thức khác. 

  Có thể thấy kiến thức áp dụng bài lab khá nhiều nhưng chỉ cần có kiến thức tốt về Routing và Switching cùng với khả năng research siêu cấp vippro thì ta có thể hoàn thành được bài lab :V.

  Để hoàn thành được bài lab, mình sẽ chia ra làm 4 chính(thật ra mình đã chia theo các anh Mentor đã phân công cho 4 người nên chia ra 4 phần :v):
    - Cấu hình lớp Layer 2 về user.
    - Cấu hình Core đổ các Vlan về Layer 2.
    - Cấu hình Firewall(đây là phần mình được giao trong bài Lab này).
    - Cấu hình Router routing ra các nhà mạng.
 
Dải IP tụi mình được cấp là **103.95.196.0/22** (vì IPv6 tụi mình chưa làm xong[nói thẳng ra là lười vkl =)))] nên mình sẽ không cấu hình phần IPv6 nhé). Ai cần giải thích cách chia mạng con của tụi mình thì mình sẽ để ở đây nhé(https://github.com/Kaint2051/Lab_Network-at-VNNIC/wiki/Chia-IP-theo-d%E1%BA%A3i-103.95.196.0-Subnet-22)

Vậy là xong phần quy hoạch IP(Cái này rối vkl, thề 🥴), ta sẽ chia các Dải mạng con vào các Vlan(mạng lan ảo) như sau:
 * Vlan 10: 103.95.197.128/28  | IP dành cho thiết bị kết nối
 * Vlan 20: 103.95.197.64/28   | IP dành cho thiết bị server
 * Vlan 30: 103.95.197.144/28  | IP dành cho Kế toán(quên là cạp đất mà ăn đó nghe ae 🤡)
 * Vlan 40: 103.95.196.0/24    | IP dành cho Văn phòng
 * Vlan 50: 103.95.197.0/26    | IP dành cho Kỹ thuật
 * Vlan 60: 103.95.197.160/28  | IP dành cho Lãnh đạo
 * Vlan 80: 103.95.197.96/28   | IP dành cho quản trị thiết bị (Server)


Tiếp theo là cấu hình từng phần của bài lab. Lệnh cấu hình khá dài và nhiều mà mình thì lười vkl nên mình sẽ config những cái chính còn lại mình sẽ để toàn bộ trong file cấu hình nha :v.

Đầu Tiên, ta cần phân tích xem mô hình 3 lớp trong sơ đồ logic này. Quan trọng nhất là lớp CORE sẽ là 2 Firewall ASA, nhiệm vụ của nó là Gateway chính của các VLAN trong Topology đưa gói tin tới các Gateway và là 1 DHCP Server cho toàn bộ mạng. Tiếp đến lớp Distribute sẽ là 2 Switch dưới Switch CORE có nhiệm vụ phân tán xuống các Switch lớp ACCESS để chia ra nhiều phân vùng. Và cuối cùng là lớp Access, nơi các Switch kết nối trực tiếp tới Enduser.
  
Qua phân tích trên thì mình sẽ cấu hình 2 Firewall trước, vì yêu cầu của bài Lab trọng tâm khá nhiều vào 2 con firewall này và đây cũng là lớp CORE của cả Topology, vậy nên bắt đầu vào thôi:
 - Trước hết ta cần Sub-interface các vlan để làm gateway định tuyến tới Layer3. Và để cấu hình Sub-interface vlan các bạn nên đọc qua cấu hình ASA cơ bản để hiểu rõ các câu lệnh và cách hoạt động nhé.
  
![image](https://user-images.githubusercontent.com/72087963/231340435-d7d88a85-ee88-44d5-ab4b-4123cf1e8f1e.png)

 Ở hình trên tụi mình thấy FW-ASA-1 và FW-ASA-2 đều cùng đổ các vlan về 2 đường Gi0/1 và Gi0/2 vì vậy mình sẽ vào bật 2 cổng này trước. Và trên FW-ASA-2 làm tương tự
***
  * FW-ASA-1(config)# interface range G0/1-2 (lệnh range để cấu hình 2 cổng cùng 1 lúc)
  * FW-ASA-1(config-if)# no shutdown (lệnh bật cổng)
  * FW-ASA-1(config-if)# exit
***
 
  Sau khi bật xong các cổng mình sẽ vào các Sub-interface Vlan trên 2 cổng vừa bật. Ở đây mình sẽ chia ra G0/1(30,40) G0/2(50,60,80)
***
  * FW-ASA-1(config)# interface G0/1.30 (vào Sub-interface Vlan 30 trên cổng G0/1)
  * FW-ASA-1(config-subif)# vlan 30 (Access vlan 30 vào sub-interface)
  * FW-ASA-1(config-subif)# nameif Inside_vlan30 (đăt tên cho Sub-interface)
  * FW-ASA-1(config-subif)# security-level 100 (đặt độ bảo mật cho sub-interface, mình sẽ đặt là 100 để có thể trust mạng local và test ping trước)
  * FW-ASA-1(config-subif)# ip address 103.95.197.145 255.255.255.240 (đặt IP gateway cho sub-interface)
  * FW-ASA-1(config-subif)# exit
***
  Làm tương tự với các vlan khác

 Tiếp đến là cấu hình mạng Outside đến Router. Mình sẽ để địa chỉ 103.95.197.129/28 cho ASA-1 và 103.95.197.132/28 cho ASA-2
   Tại FW-ASA-1:
***
  * FW-ASA-1(config)# interface G0/0
  * FW-ASA-1(config-if)# nameif Outside
  * FW-ASA-1(config-if)# security-level 0 (vì là mạng Outside nên ta cần để độ bảo mật là 0)
  * FW-ASA-1(config-if)# ip address 103.95.197.129 255.255.255.240
  * FW-ASA-1(config-if)# no shutdown
  * FW-ASA-1(config-if)# exit
*** 
  Làm tương tự với ASA-2 với IP .132 nhé

  Đến đây ta đã hoàn thành gần như 50% công việc trên Firewall rồi, để tăng tính dự phòng ta nên cấu hình Failover trên cả 2 Firewall để tránh trường hợp team Network tự tạo OT để fix khi 2 con ngỏm cùng lúc =)). 
      
Về bản chất thì Failover là để tăng tính dự phòng khi ASA-1 sẽ là `Active`(ở mode Active, các lưu lượng mạng đi ra ngoài sẽ hoạt động chính trên con này) và ASA-2 sẽ là `Standby`(ở mode Standby, nó sẽ hoạt động như 1 bản copy của Active và sẽ ko nhận bất kì hoạt động định tuyến mạng nào trừ khi con Active ngỏm), nếu có sự cố thì Standby sẽ lên làm Active để lưu lượng mạng trong quá trình gặp sự cố vẫn diễn ra bình thường (Đây là theo cách hiểu của mình và nếu có sai sót mong được góp ý bên dưới :> )
     
 Để hiểu rõ hơn về Failover và dự phòng trong Firewall Cisco ASA thì đây là link mà mình thấy giải thích ok nhất: https://cnttshop.vn/blogs/cisco/cau-hinh-ha-failover-active-standby-tren-firewall-cisco-asa

Còn bây giờ thì vào cấu hình thôi 😁

À trước khi cấu hình port nào thì nhớ bật port nhé(mình đã quên bật port và phải ngồi suy nghĩ cả buổi chiều để tự hỏi vì sao cả 2 con đều ko hoạt động 😂), và vì failover cần 1 dải IP mới nên mình đã + thêm 1bit vào phần Net để có mạng 103.95.198.0/29(có thể sử dụng /30)
***
  * FW-ASA-1(config)# failover lan unit primary (Primary để chỉ định đây là Active và nên cấu hình Primary trước nhé)
  * FW-ASA-1(config)# failover lan interface int_g0/7 G0/7 (Đặt tên cho failover link trên cổng nối giữa 2 con)
  * FW-ASA-1(config)# failover interface ip int_g0/7 103.95.198.1 255.255.255.248 standby 103.95.198.2 (cấu hình IP cho Active và standby trên cổng failover)
  * FW-ASA-1(config)# failover (bật mode failover)
***
Làm tương tự với ASA-2 và đợi khoảng 1-2 phút thì cả 2 Firewall sẽ tự động chuyển mode.

Tiếp đến là cấu hình Default Route để toàn bộ mạng Local có thể ra được bên ngoài.
***
  * FW-ASA-1(config)# Route Outside 0.0.0.0 0.0.0.0 103.95.197.142 (để tên Outside đúng với nameif mình đã đặt ở trên cổng G0/0 nhé, và 103.95.197.142(VRRP) sẽ là IP nexthop tiếp theo để các vlan ra được tới các Router) 
***
Giải thích câu lệnh trên 1 xíu với cách hiểu của mình thì đây là định tuyến sẽ đưa các gói tin ra ngoài mạng Internet mà không biết một tuyến đường cụ thể và nó sẽ phải tự học toàn bộ route trong bảng định tuyến ngoài Internet.

Bước kế tiếp là triển khai dịch vụ DHCP.  
***
  * FW-ASA-1(config)# dhcpd address 103.95.197.146-103.95.197.158 Inside_vlan30 (dải IP mình cấp cho Vlan 30)
  * FW-ASA-1(config)# dhcpd enable Inside_vlan30 (bật dịch vụ DHCP cho Vlan 30)
***
 Làm tương tự với các vlan còn lại nha.

Cuối cùng là cấu hình MDF(modular policy framework) cho phép ping các gói ICMP qua firewall nhá(mặc định trong ASA thì sẽ ko được bật sẵn)
***
  * FW-ASA-1(config)# policy-map global_policy 
  * FW-ASA-1(config-pmap)# class inspection_default 
  * FW-ASA-1(config-pmap-c)# inspect icmp 
***
Đây là lần đầu mình cấu hình ASA nên về phần policy mình vẫn chưa có kiến thức nhiều vì vậy mình sẽ không giải thích đoạn này nhé.


 Ở phần CORE thì mọi việc dễ hơn những sẽ dài hơn và khó chịu mà 1 đặc sản ở layer đem đến đó chính là Loop gói tin (thề luôn, nhiều khi mình phải cúng ông bà để phù hộ cho ko bị loop ở layer 2😂😂)
  
  Đầu tiên thì chúng ta cần lên trunking và Allowed vlan toàn bộ và đường nối(trừ 2 đường e1/2-3 sẽ config LACP) tới CORE-1 và CORE-2 nhé.(Ở đây nếu mọi người thấy dài quá thì dùng VTP cho khoẻ, còn đề bài yêu cầu ko dùng VTP thì mình đổ chay🙂)
***
  * SW-CORE-01(config)# interface range e1/0-1
  * SW-CORE-01(config-if)# switchport trunk encapsulation dot1q (cho biết Core cần chuyển Encap trên port là đường trunk)
  * SW-CORE-01(config-if)# switchport mode trunk (chuyển đổi mode trunk)
  * SW-CORE-01(config-if)# switchport trunk allowed vlan 30,40,50,60,80 (cho phép các vlan được allowed trên các port này)
***
 Trên Core 2 làm tương tự

 Tiếp theo ta cần tạo vlan ở 2 Core:
***
  * SW-CORE-01(config)# vlan 30
  * SW-CORE-01(config-vlan)#name VanPhong
  * SW-CORE-01(config-vlan)#exit
***
  Làm tương tự với các vlan khác và cấu hình Core 2 cũng như vậy

 Tạo LACP cho 2 port e1/2-3
***
  * SW-CORE-01(config)# interface range e1/2-3
  * SW-CORE-01(config-if)# channel-group 1 mode on (cho 2 port vào group 1 và dùng mode on để tự thương lượng với nhau)
  * SW-CORE-01(config-if)# exit
  * SW-CORE-01(config)# interface port-channel 1
  * SW-CORE-01(config-if)# switchport mode trunk
  * SW-CORE-01(config-if)# switchport trunk allowed vlan 30,40,50,60
***
Cấu hình ở Core 2 tương tự như trên
 Đối với LACP ta phải lựa chọn 2 Core có port với thông số tên port và cấu hình của port phải giống nhau 100%. Là 1 giải pháp để tăng tính Redundacy và sẵn sàng cao.

Cuối cùng là cấu hình STP, đây là bước cấu hình tuy dễ nhưng để chính xác thì rất hên xui 😂. Trong đề bài yêu cầu Core 1 sẽ là Root chính cho các vlan nên ta sẽ cấu hình trên Core 1
***
  * SW-CORE-01(config)# spanning-tree mode rapid-pvst (bật mode rapid trong STP để hội tụ nhanh hơn)
  * SW-CORE-01(config)# spanning-tree vlan 30 root primary (đặt vlan 30 thành Root Bridge)
***
Khuyến cáo bước này nên show spanning-tree lại để check xem vlan trên Core 1 đã thành Root chưa nhé. Xong rồi thì làm tương tự với các vlan còn lại.

Tới đây thì hầu như 2 lớp Distribution và Access ở layer 2 đều có cấu hình tương tự nhau, đều lên trunk và allowed các vlan trên các cổng. Tuy nhiên ở Switch Access, các port đổ về các user ta phải cấu hình mode access như sau:
***
  * ASW-01(config)# interface e0/0 
  * ASW-01(config-if)# switchport mode access
  * ASW-01(config-if)# switchport access vlan 30
***
![image](https://user-images.githubusercontent.com/72087963/231421058-d1f92a9f-0dac-405b-bb89-66acbc7f401a.png)

Port nào thì access vào vlan tương ứng. Sau khi access xong toàn bộ ta sẽ vào VPC và gõ lệnh `dhcp` để đổ IP về nhé. Khi có được IP của vlan tương ứng hãy thử ping tới gateway của các vlan trên Firewall.

Sau khi test ping thành công, ta chuyển đến phần ở Layer 3 chính là 2 router:
  Nhiệm vụ của 2 Router Gateway này chính là kết nối mạng local với các ISP bằng giao thức BGP, và tại 2 Router xuống Firewall cần tạo 1 đường Gateway VRRP để mạng local có thể ping tới Layer 3.

  Đầu tiên ta phải chia IP và cấu hình vào port e0/1 tại RGW-01 với IP là 103.95.197.130 và e0/2 tại RGW-02 với IP là 103.95.197.131 nhé
***
  * RGW-01(config)# interface e0/1 
  * RGW-01(config-if)# ip address 103.95.197.130
  * RGW-01(config-if)# no shutdown
***
 Làm tương tự với RGW-02 với dãy IP và port trên nhé.

Để tạo VRRP ta cần chọn 1 địa chỉ IP dành cho VRRP, ở đây mình sẽ chọn địa chỉ IP cuối cùng của dải IP kết nối là 103.95.197.142
***
  * RGW-01(config)# interface e0/1 
  * RGW-01(config-if)# vrrp 1 ip 103.95.197.142 (Đặt IP gateway ảo)
  * RGW-01(config-if)# vrrp 1 priority 110 (Đặt chỉ số priority để quyết định RGW-01 sẽ là Master)
***
  Ở RGW-02 cũng làm như trên nhưng điều chỉnh Priority về 90 để làm backup nhé.
Về cơ bản thì VRRP cũng là một giao thức gộp 1 vài router với nhau tạo thành 1 gateway ảo duy nhất để làm Gateway chính cho mạng local.

  Đến đây thì ta có thể đứng từ user ping ra tới Gateway ảo với IP 103.95.197.142 nhưng từ Router ping tới các user thì chắc chắn sẽ không được =))).
Vì firewall ASA luôn mặc định không cho phép các thiết bị mạng ngoài không được phép vào ping vào mạng Local vậy nên ta cần tạo 1 Accesslist để cho Router có thể ping vào các Vlan.
***
  * FW-ASA-1(config)# access-list outside_access_in extended permit icmp any any 
  * FW-ASA-1(config)# access-group outside_access_in in interface outside
***
2 câu lệnh trên được hiểu như sau, dòng đầu tiên ta khai báo 1 ACL có tên là outside_access_in ở mode extended với rule là permit cho phép SRC và DST có thể sử dụng ICMP(PING). Ở câu lệnh thứ 2 ta đặt ACL(outside_access_in) vừa tạo vào 1 access-group trong (in) interface của vùng outside Firewall(từ ASA tới Router).
