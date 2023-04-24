# ML_ClassificationOfFirewall

data ที่นำมาใช้เป็น data firewall log โดยจะทำการ predict action ของ firewall ซึ่งเป็น multiple classes โดย model ที่เลือกใช้ classificaiton คือ Decision Tree และ KNN

ข้อมูลของ Dataset มีดังนี้

Columns Description
- Source Port: Client Source Port 
- Destination Port: Client Destination Port 
- NAT Source Port: Network Address Translation Source Port 
- NAT Destination Port: Network Address Translation Destination Port 
- Elapsed Time (sec): Elapsed Time for flow 
- Bytes: Total Bytes 
- Bytes Sent: Bytes Sent 
- Bytes Received: Bytes Received 
- Packets: Total Packets 
- pkts_sent: Packets Sent 
- pkts_received: Packets Received
- Action: Class (allow, deny, drop, reset-both)

Labels Description
- Allow: Allows the internet traffic. 
- Deny: Blocks traffic and enforces the default Deny Action defined for the application that is being denied. 
- Drop: Silently drops the traffic; for an application, it overrides the default deny action. A TCP reset is not sent to the host/application. 
- Reset-Both: Sends a TCP reset to both the client-side and server-side devices
