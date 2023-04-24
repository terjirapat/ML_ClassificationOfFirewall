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
- เปลี่ยน data type ของ feature 4 ตัว ['Source Port', 'Destination Port', 'NAT Source Port', 'NAT Destination Port'] จาก int เป็น str และทำการ onehot encoding

- upsampling label 'reset-both' ด้วย SMOTE โดยเลือกใช้ function SMOTENC ที่เหมาะกับ dataset ที่มีทั้ง categorical และ numerical data

![image](https://user-images.githubusercontent.com/77285026/234031546-d0e64b15-0b5d-4998-9aa0-974e798ea11c.png)

- feature scaling ด้วย minmax เนื่องจาก feature ทั้งหมดกระจายตัวแบบ right-skewed

![image](https://user-images.githubusercontent.com/77285026/234031477-012fc7e9-4118-4990-925a-a0152836b3df.png)

- เนื่องจาก feature กระจายตัวแบบ right-skewed จึงนำ feature มา plot ด้วย median เพื่อหา pattern ของ data พบว่า feature ทุกตัวใน label 'allow' มีค่าสูงกว่า label อื่นที่ไม่มีนัยสำคัญ จึงสรุปว่าควรนำ feature ทุกตัวเข้า model

![image](https://user-images.githubusercontent.com/77285026/234032593-012bf84d-b22a-41ae-824f-257f1c6929cc.png)

- correlation ของ bytes และ packets มีค่าสูงเข้าใกล้ 1 สรุปว่า feature สองตัวนี้ไปในทางเดียวกันและ สามารถเลือกตัดตัวนึงออกได้ ในที่นี้จะตัด packets ออกเนื่องจาก bytes มีตัวเลขที่ละเอียดกว่า 

# Training Testing and Evaluation
- feature ที่เลิอกใช้ในการเข้า model ได้แก่ ['Source Port', 'Destination Port', 'NAT Source Port', 'NAT Destination Port', 'Bytes', 'Bytes Sent', 'Bytes Received', 'Elapsed Time (sec)'] 
โดยเลือก port เนื่องจากเป็นแหล่งที่ใช้ในการระบุที่มาและปลายทางของ traffic ซึ่งเป็นสิ่งสำคัญในการ action ของ firewall  

![image](https://user-images.githubusercontent.com/77285026/234037414-f9d0b48f-0805-4058-82a2-55281c1e611b.png)

- ในขั้นตอนการนำ data เข้าโมเดลได้ทำเป็น pipeline โดยเริ่มที่การ preprocessing ซึ่งทำ feature scaling ใน feature ที่เป็น numerical และทำการ onehot encoding ใน feature ที่เป็น categorical จากนั้น flow ไปเข้า model

![image](https://user-images.githubusercontent.com/77285026/234035447-415d976e-952b-42ea-a398-dd99c34d4563.png)

- การเลือก model จะทำ cross validation ระหว่าง decision tree และ KNN โดยกำหนด flod = 5 และ วัดผลด้วยค่า f1 macro เนื่องจากต้องการให้ความสำคัญกับทุก label เท่ากัน ผลคือ decision tree มี f1 macro มากกว่าที่ 0.93 เทียบกับ KNN ที่มีค่า 0.85
- นำ decision tree มาทำ cross validation เพื่อหา max dept ที่เหมาะสมโดยกำหนดตั้งแต่ 1-10 และ flod = 5 ผลที่ได้ max dept ที่เหมาะสมคือ 10 และวัดโดย entropy

```python
numericTransformer = Pipeline(steps=[('scaler', MinMaxScaler())])
catsTransformer = Pipeline(steps=[('OneHotEncoder', OneHotEncoder(handle_unknown='ignore'))])
preprocessor = ColumnTransformer(transformers=[('nums', numericTransformer, make_column_selector(dtype_include=int)),
                                               ('cats', catsTransformer, make_column_selector(dtype_include=object))])
tree = DecisionTreeClassifier(criterion='entropy', max_depth=10)
tree_pipe = Pipeline(steps=[('preprocessor', preprocessor),
                           ('tree', tree)])

tree_pipe.fit(X_train, y_train)
pred = tree_pipe.predict(X_test)                    
```
```bash
confusion_matrix
[[7524    9    0    0]
 [   0 2938   13    0]
 [   0    2 2608    0]
 [   0    3    0   10]]

classification_report
              precision    recall  f1-score   support

       allow       1.00      1.00      1.00      7533
        deny       1.00      1.00      1.00      2951
        drop       1.00      1.00      1.00      2610
  reset-both       1.00      0.77      0.87        13

    accuracy                           1.00     13107
   macro avg       1.00      0.94      0.97     13107
weighted avg       1.00      1.00      1.00     13107

f1 score macro : 0.9653815056895183
```

### สรุป
- จากการทำ cross validation ด้วย model decision tree และ knn ผลที่ได้คือ decision tree มี performance ที่ดีกว่าจาก f1 macro ที่ใช้เป็นตัววัด
- decision tree ที่ perform ที่สุดคือ model ที่ max dept = 10 และวัดโดย entropy จากการทำ cross validation ที่กำหนด max dept 1-10
- การ upsampling และ feature scaling มีส่วนช่วยใน performance ของ model ดีขึ้น
