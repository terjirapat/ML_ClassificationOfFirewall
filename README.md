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

```bash
RangeIndex: 65532 entries, 0 to 65531
Data columns (total 12 columns):
 #   Column                Non-Null Count  Dtype 
---  ------                --------------  ----- 
 0   Source Port           65532 non-null  int64 
 1   Destination Port      65532 non-null  int64 
 2   NAT Source Port       65532 non-null  int64 
 3   NAT Destination Port  65532 non-null  int64 
 4   Action                65532 non-null  object
 5   Bytes                 65532 non-null  int64 
 6   Bytes Sent            65532 non-null  int64 
 7   Bytes Received        65532 non-null  int64 
 8   Packets               65532 non-null  int64 
 9   Elapsed Time (sec)    65532 non-null  int64 
 10  pkts_sent             65532 non-null  int64 
 11  pkts_received         65532 non-null  int64 
dtypes: int64(11), object(1)
```

จะแบ่งขั้นตอนดังนี้ Data Cleansing and EDA, Training Testing and Evaluation

# Data Cleansing and EDA
- เปลี่ยน data type ของ feature 4 ตัว ['Source Port', 'Destination Port', 'NAT Source Port', 'NAT Destination Port'] จาก int เป็น str
- upsampling label 'reset-both' ด้วย SMOTE โดยเลือกใช้ function SMOTENC ที่เหมาะกับ dataset ที่มีทั้ง categorical และ numerical data

![image](https://user-images.githubusercontent.com/77285026/234031546-d0e64b15-0b5d-4998-9aa0-974e798ea11c.png)

- feature scaling ด้วย minmax เนื่องจาก feature ทั้งหมดกระจายตัวแบบ right-skewed

![image](https://user-images.githubusercontent.com/77285026/234031477-012fc7e9-4118-4990-925a-a0152836b3df.png)

- เนื่องจาก feature กระจายตัวแบบ right-skewed จึงนำ feature มา plot ด้วย median เพื่อหา pattern ของ data พบว่า feature ทุกตัวใน label 'allow' มีค่าสูงกว่า label อื่นที่ไม่มีนัยสำคัญ จึงสรุปว่าควรนำ feature ทุกตัวเข้า model

![image](https://user-images.githubusercontent.com/77285026/234032593-012bf84d-b22a-41ae-824f-257f1c6929cc.png)

- correlation ของ bytes และ packets มีค่าสูงเข้าใกล้ 1 สรุปว่า feature สองตัวนี้ไปในทางเดียวกันและ สามารถเลือกตัดตัวนึงออกได้ ในที่นี้จะตัด packets ออกเนื่องจาก bytes มีตัวเลขที่ละเอียดกว่า 
