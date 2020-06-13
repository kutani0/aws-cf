# AWS CloudFormation 專題工作日誌

----------
## 主控台操作截圖

主控台介面選擇cloudcformation

![](https://github.com/kutani0/aws-cf/blob/master/pic/01.png)


進入畫面後，點選設計’模板”

![](https://github.com/kutani0/aws-cf/blob/master/pic/02.png)


進入模板設計畫面

![](https://github.com/kutani0/aws-cf/blob/master/pic/03.png)


下方”參數”設定畫面

![](https://github.com/kutani0/aws-cf/blob/master/pic/04.png)


下方”映像”設定畫面

![](https://github.com/kutani0/aws-cf/blob/master/pic/05.png)


下方"條件"設定畫面

![](https://github.com/kutani0/aws-cf/blob/master/pic/06.png)


下方"元數據"設定畫面

![](https://github.com/kutani0/aws-cf/blob/master/pic/07.png)


下方"輸出"設定畫面

![](https://github.com/kutani0/aws-cf/blob/master/pic/08.png)


下方"模板"設定畫面，可設定啟用哪些資源

![](https://github.com/kutani0/aws-cf/blob/master/pic/09.png)

----------

## YAML範例
    UserData:
            Fn::Base64:                                # YAML makes userdata much cleaner
              !Sub |
                  #!/bin/bash -ex
                  yum install -y httpd;
                  echo "<html>I love YAML CloudFormation!!</html>" > /var/www/html/index.html;
                  cd /var/www/html;
                  chmod 755 index.html;
                  service httpd start;
                  chkconfig httpd on;
## JSON範例
    "UserData": {
                        "Fn::Base64": {
                            "Fn::Sub": "#!/bin/bash -ex\nyum install -y httpd;\necho \"<html>I love YAML CloudFormation!!</html>\" > /var/www/html/index.html;\ncd /var/www/html;\nchmod 755 index.html;\nservice httpd start;\nchkconfig httpd on;\n"
                        }
## 
----------

# 架構圖開發過程
## 分工研究架構cloudformation所需yml訊息及格式(2019/01/24~2019/01/25)
----------

AWS官方各項指導說明書：
https://docs.aws.amazon.com/zh_tw/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html

----------
![](https://github.com/kutani0/aws-cf/blob/master/pic/10.png)


**yaml的資源設定語法示意**

    Resources資源:
      ???資源名稱:  #資源名稱可自訂(注意只能大小寫加數字)
        Type資源種類: '???::??:??'  #種類：設定要創建AWS的資源種類
        Properties資源屬性:
          參數設定1:
            - XXXXX
              YYYYY
          參數設定2: !Ref ZZZZZ
----------

**VPC**

    #創建 VPC
    
    AWSTemplateFormatVersion: '2010-09-09'
    Resources:
      VPC: #資源名稱可自訂(注意只能大小寫加數字)
        Type: 'AWS::EC2::VPC'  #種類：設定要創建AWS的資源種類，這裡選vpc資源服務
        Properties:  #屬性
          EnableDnsSupport: true #設置ture提供DNS服務
          EnableDnsHostnames: true #設置ture提供DnsHostname服務
          CidrBlock: 10.10.0.0/16  #使用 10.10.0.0/16
          Tags:  #設置tags
            - Key: Name  #設置tag的key值，可在主控台Name的欄位上看到下行value所設置的內容
              Value: cc104_vcloudlab_pro_vpc  #設置tag的value值，可自訂內容

**subnet**

    #創建 PublicSubnet & PrivateSubnet
    #PublicSubnet 子網段：10.10.1.0/24
    #PrivateSubnet 子網段：10.10.2.0/24
    
    Resources:  
      PublicSubnet: #資源名稱可自訂(注意只能大小寫加數字)
        Type: 'AWS::EC2::Subnet'  #種類：設定要創建AWS的資源種類，這裡選Subnet資源服務
        Properties:  #資源屬性
          VpcId: !Ref VPC  #將PublicSubnet加至剛剛設置的Vpc
          CidrBlock: 10.10.1.0/24  #設置Subnet網域為10.10.2.0/24
          Tags:  #設置tags
            - Key: Name  #設置tag的key值，可在主控台Name的欄位上看到下行value所設置的內容
              Value: cc104_vcloudlab_pro_subnet  #設置tag的value值，可自訂內容
    
      PrivateSubnet:  #資源名稱可自訂(注意只能大小寫加數字)
        Type: 'AWS::EC2::Subnet'  #種類：設定要創建AWS的資源種類，這裡選Subnet資源服務
        Properties:  #資源屬性
          VpcId: !Ref VPC  #將PrivateSubnet加至剛剛設置的Vpc
          CidrBlock: 10.10.2.0/24  #設置Subnet網域為10.10.2.0/24
          Tags:  #設置tags
            - Key: Name  #設置tag的key值，可在主控台Name的欄位上看到下行value所設置的內容
              Value: cc104_vcloudlab_pro_subnet  #設置tag的value值，可自訂內容       

**InternetGateway**

    #創建 InternetGateway
    
    Resources:
      InternetGateway:  #資源名稱可自訂(注意只能大小寫加數字)
        Type: 'AWS::EC2::InternetGateway'  #種類：設定要創建AWS的資源種類，這裡選InternetGateway資源服務
        Properties:  #資源屬性
          Tags:  #設置tags
            - Key: Name  #設置tag的key值，可在主控台Name的欄位上看到下行value所設置的內容
              Value: cc104_vcloudlab_pro_igw  #設置tag的value值，可自訂內容 


----------

---以下這串不用，由連線關聯來產生---

    #將IGW 加到 VPC
    AttachGateway:
      Type: AWS::EC2::VPCGatewayAttachment
      Properties:
        VpcId:
          Ref: myVPC
        InternetGatewayId:
          Ref: myInternetGateway


----------

**R****oute table**
設定Subnet關聯的RouteTable
2019/01/26網路相關資源成功佈署

    #創建 PublicRoutetable
    
    Resources:
      PublicRouteTable:  #資源名稱可自訂(注意只能大小寫加數字)
        Type: 'AWS::EC2::RouteTable' #種類：設定要創建AWS的資源種類，這裡選RouteTable資源服務
        Properties:  #資源屬性
          VpcId: !Ref VPC  #將PublicRoutetable加至剛剛設置的Vpc
          Tags:  #設置tags
            - Key: Name  #設置tag的key值，可在主控台Name的欄位上看到下行value所設置的內容
              Value: cc104_vcloudlab_pro_pubrtable  #設置tag的value值，可自訂內容 
    
    #設定 route 規則並將其加入 routetable
    
    Resources:
      PublicRoute:  #資源名稱可自訂(注意只能大小寫加數字)
        Type: 'AWS::EC2::Route'  #種類：設定要創建AWS的資源種類，這裡選Route資源服務
        Properties:  #資源屬性
          RouteTableId: !Ref PublicRouteTable  #指定使用的RouteTableId
          DestinationCidrBlock: 0.0.0.0/0  #設定目的網段
          GatewayId: !Ref InternetGateway  #指定使用的InternetGateway
    
    #將 routetable 指向 PublicSubnet        
    
    Resources:
      EC2SRTA1Y14R:  #資源名稱可自訂(這裡是系統自帶)
        Type: 'AWS::EC2::SubnetRouteTableAssociation'  #種類：這裡選Subnet所關聯的RouteTable
        Properties:  #資源屬性
          RouteTableId: !Ref PublicRouteTable  #指定使用的RouteTableId
          SubnetId: !Ref PublicSubnet  #指定使用的PublicSubnet
          
----------

**EC2 & SecurityGroup**
2019/01/27(測試單台EC2並運行userdata成功)
2019/01/28(測試兩台EC2並運行userdata成功)

![](https://github.com/kutani0/aws-cf/blob/master/pic/11.png)



    #創建 WebServerInstance(EC2) & WebServerSecurityGroup
    #將 EC2 置於 Subnet 中，模板系統會自動產生連結關係 
    #設置基本應用環境
    #WebServerInstance 指向 PublicSubnet 及 WebServerSecurityGroup
    
    Resources:
      WebServerInstance:  #資源名稱可自訂(注意只能大小寫加數字)
        Type: 'AWS::EC2::Instance'  #種類：設定要創建AWS的資源種類，這裡選EC2資源服務
        Properties:  #資源屬性
          Tags:  #設置tags
            - Key: Name  #設置tag的key值，可在主控台Name的欄位上看到下行value所設置的內容
              Value: cc104_web_server  #設置tag的value值，可自訂內容
          InstanceType: !Ref InstanceType
          ImageId: ami-1a15c77b
          KeyName: !Ref KeyName
          NetworkInterfaces:
            - SubnetId: !Ref PublicSubnet
              DeviceIndex: 0
              AssociatePublicIpAddress: true
              DeleteOnTermination: true
              GroupSet:
                - !Ref WebServerSecurityGroup
              
          UserData: !Base64 
            'Fn::Join':
              - ''
              - - |
                  #!/bin/bash
                - |
                  yum update -y
                - |
                  yum install -y docker git awscli ruby
                - |
                  usermod -aG docker ec2-user
                - |
                  service docker start
                - |
                  chkconfig docker on
                - |
                  $(aws ecr get-login --no-include-email --region ap-northeast-1)
                - >
                  docker pull
                  204065533127.dkr.ecr.ap-northeast-1.apmazonaws.com/cc104devops-repo:0.0.1
                - >
                  docker run -p 80:5000  -d  --name cc104-devops
                  204065533127.dkr.ecr.ap-northeast-1.amazonaws.com/cc104devops-repo:0.0.1
                - >
                  sudo curl -O
                  https://aws-codedeploy-ap-northeast-1.s3.amazonaws.com/latest/install
                - |
                  sudo chmod +x ./install
                - |
                  sudo  ./install auto
                  
      WebServerSecurityGroup:
        Type: 'AWS::EC2::SecurityGroup'
        Properties:
          VpcId: !Ref VPC
          GroupDescription: Allow access from HTTP and SSH traffic
          SecurityGroupIngress:
            - IpProtocol: tcp
              FromPort: 80
              ToPort: 80
              CidrIp: 0.0.0.0/0
            - IpProtocol: tcp
              FromPort: 22
              ToPort: 22
              CidrIp: 0.0.0.0/0
          SecurityGroupEgress:
            - IpProtocol: -1   #-1代表全部通訊埠
              CidrIp: 0.0.0.0/0
    
    
    #創建 WebAppInstance(EC2) & WebAppSecurityGroup
    #將 EC2 置於 Subnet 中，模板系統會自動產生連結關係
    #設置基本應用環境
    #WebAppInstance 指向 PrivateSubnet 及 WebAppSecurityGroup
    
    Resources:
      WebAppInstance:
        Type: 'AWS::EC2::Instance'
        Properties:
          Tags:
            - Key: cc104
              Value: cc104_app_server
          InstanceType: !Ref InstanceType
          ImageId: ami-1a15c77b
          KeyName: !Ref KeyName
          NetworkInterfaces:
            - SubnetId: !Ref PrivateSubnet
              DeviceIndex: 0
              AssociatePublicIpAddress: false
              DeleteOnTermination: true
              GroupSet:
                - !Ref WebAppSecurityGroup
              
          UserData: !Base64 
            'Fn::Join':
              - ''
              - - |
                  #!/bin/bash
                - |
                  yum update -y
                - |
                  yum install -y docker git awscli ruby
                - |
                  usermod -aG docker ec2-user
                - |
                  service docker start
                - |
                  chkconfig docker on
    
      WebAppSecurityGroup:
        Type: 'AWS::EC2::SecurityGroup'
        Properties:
          VpcId: !Ref VPC
          GroupDescription: Only allow access from SSH traffic
          SecurityGroupIngress:
            - IpProtocol: tcp
              FromPort: 22
              ToPort: 22
              CidrIp: 10.10.1.0/24
    

**R****oute53** ****
2019/01/29 Route53-RecordSet成功
2019/01/30 Route53-HostedZone成功

----------
      #設置名為cc104HZone_cf的HostedZone私有域名為vcloudlab2.pro.
      #再將之前設定的VPC加到HostedZone中
      DNSHZone:
        Type: 'AWS::Route53::HostedZone'
        Properties:
          HostedZoneConfig:
            Comment: Lab hosted zone for vcloudlab2.pro
          Name: vcloudlab2.pro.
          VPCs:
            - VPCId: !Ref VPC
              VPCRegion: ap-northeast-1
          HostedZoneTags:
            - Key: Name
              Value: cc104HZone_cf
        Metadata:
          'AWS::CloudFormation::Designer':
            id: 3f4c4fce-c582-4884-8dfb-08a9eb3ca09c
      #用RecordSet將WebServerInstance的PublicIp加上私有域名為app1.vcloudlab2.pro.
      WebRSet1:
        Type: 'AWS::Route53::RecordSet'
        Properties:
          HostedZoneId: !Ref DNSHZone
          Comment: DNS name for my instance.
          Name: app1.vcloudlab2.pro.
          Type: A
          TTL: '900'
          ResourceRecords:
            - !GetAtt 
              - WebServerInstance
              - PublicIp
        Metadata:
          'AWS::CloudFormation::Designer':
            id: d8e17d19-287e-447f-bbda-599ae76f6ab6
      #用RecordSet將WebServerInstance的PrivateDnsName自訂私有域名為web.vcloudlab2.pro.
      WebRSet2:
        Type: 'AWS::Route53::RecordSet'
        Properties:
          HostedZoneId: !Ref DNSHZone
          Comment: DNS name for my instance.
          Name: web.vcloudlab2.pro.
          Type: CNAME
          TTL: '900'
          ResourceRecords:
            - !GetAtt 
              - WebServerInstance
              - PrivateDnsName
        Metadata:
          'AWS::CloudFormation::Designer':
            id: b9d18e01-a7ee-4287-be97-162578b3d944
      #用RecordSet將WebAppInstance的PrivateDnsName自訂私有域名為work.vcloudlab2.pro.
      WorkRset:
        Type: 'AWS::Route53::RecordSet'
        Properties:
          HostedZoneId: !Ref DNSHZone
          Comment: DNS name for my instance.
          Name: work.vcloudlab2.pro.
          Type: CNAME
          TTL: '900'
          ResourceRecords:
            - !GetAtt 
              - WebAppInstance
              - PrivateDnsName
        Metadata:
          'AWS::CloudFormation::Designer':
            id: b975d691-6136-4f41-97f0-8b5836c4bd64

**S3、SQS、DynamoDB (2019/01/30)**
 可成功配置，細部調整待確認

----------
      MySourceQueue:
        Type: 'AWS::SQS::Queue'
        Properties:
          RedrivePolicy:
            deadLetterTargetArn:
              'Fn::GetAtt':
                - MyDeadLetterQueue
                - Arn
            maxReceiveCount: 5
        Metadata:
          'AWS::CloudFormation::Designer':
            id: d382331a-f7ec-4df0-a96e-449212e7573f
      MyDeadLetterQueue:
        Type: 'AWS::SQS::Queue'
        Metadata:
          'AWS::CloudFormation::Designer':
            id: 14b19ab4-06b5-4aed-ba4a-62c81822f83e


      DynamoTable:
        Type: 'AWS::DynamoDB::Table'
        Properties:
          AttributeDefinitions:
            - AttributeName: id
              AttributeType: S
          KeySchema:
            - AttributeName: id
              KeyType: HASH
          ProvisionedThroughput:
            ReadCapacityUnits: '5'
            WriteCapacityUnits: '5'
          TableName: cc104_dynamodb_cf
        Metadata:
          'AWS::CloudFormation::Designer':
            id: 2a8f1d08-01a6-4051-acf9-650dafbbce33


      S3Bucket:
        Type: 'AWS::S3::Bucket'
        Properties:
          AccessControl: PublicRead
          WebsiteConfiguration:
            IndexDocument: index.html
            ErrorDocument: error.html
          BucketName: cc104_S3_cf
        DeletionPolicy: Retain
        Metadata:
          'AWS::CloudFormation::Designer':
            id: 7e5e5fdf-26d2-40d7-b3a7-a8e9b1c39a68
      BucketPolicy:
        Type: 'AWS::S3::BucketPolicy'
        Properties:
          PolicyDocument:
            Id: S3Policy
            Version: 2012-10-17
            Statement:
              - Sid: PublicReadForGetBucketObjects
                Effect: Allow
                Principal: '*'
                Action: 's3:GetObject'
                Resource: !Join 
                  - ''
                  - - 'arn:aws:s3:::'
                    - !Ref S3Bucket
                    - /*
          Bucket: !Ref S3Bucket
        Metadata:
          'AWS::CloudFormation::Designer':
            id: 3786990f-61ff-4269-961f-1f0a86ec0ecf


## 2019/01/31 專題前yaml測試版本

**已攻略項目：**

- VPC
  - PublicSubnet
  - PublicRoute
  - PrivateSubnet
  - PublicRouteTable
- InternetGateway

- VPCGatewayAttachment
- SubnetRouteTableAssociation

- WebServerSecurityGroup
- WebAppSecurityGroup

- EC2Role
- RolePolicies
- InstanceProfile


- WebServerInstance
- WebAppInstance

- DNSHZone
  - WebRSet1
  - WebRSet2
  - WorkRset
- S3Bucket
- BucketPolicy

**可成功佈署，待測試項目：**

- MySourceQueue
- MyDeadLetterQueue
- DynamoTable
----------

**Cloudformation元件組合圖**

![](https://github.com/kutani0/aws-cf/blob/master/pic/12.png)






相關文章說明及連結：
----------
1. AWS CloudFormation 提供一種可讓開發人員和系統管理員建立相關 AWS 資源集合的簡單方式，並透過按順序且可預測的方式進行佈建


![](https://d2mxuefqeaa7sj.cloudfront.net/s_CE04D085437276A46E373411712477B359A19E777460932E2546993EE2ADE46C_1548300680717_howitworks.c316d3856638c6c9786e49011bad660d57687259.png)

- 編碼基礎設定

     從頭開始使用CloudFormation模組語言，可使用YAML或JSON設定檔格式

- 在本地端查看您的模組YAML或JSON設定檔格式，或上傳它進入S3 Bucket
- 使用CloudFormation通過瀏覽器控制台，命令行組工具或API，根據模組YAML或JSON檔創建堆疊
- CloudFormation規劃及配置在指定的堆疊和資源模組
![](https://d2mxuefqeaa7sj.cloudfront.net/s_5CC7DABA23424B7A9D140C91EA23D98C2CD9B5D0337BA6B3CA14B950A8B126A5_1548299879037_image.png)

2. 最簡易版的實作影片：https://www.youtube.com/watch?time_continue=433&v=qKljYgi2lQ0
3. 如何編寫CloudFormation：https://docs.aws.amazon.com/zh_cn/AWSCloudFormation/latest/UserGuide/template-guide.html   (重點在步驟三那塊)
  1. 備註1：模板是一个 JSON 或 YAML 文本文件，其中包含有关您希望在堆栈中创建的 AWS 资源的配置信息
  2. 備註2：個人建議使用yaml，json不是給人類看的。
  3. 使用 *Parameters* 部分可以声明在创建堆栈时可传递给模板的值。参数是指明敏感信息的一种有效手段，这些敏感信息包括用户名和密码一类的和您不想存储在模板内部的信息。.
  4. 參數很多，要用到時再看
  5. 關於參數的說明https://docs.aws.amazon.com/zh_cn/AWSCloudFormation/latest/UserGuide/template-anatomy.html https://docs.aws.amazon.com/zh_cn/AWSCloudFormation/latest/APIReference/CommonParameters.html


4. CloudFormation IAM:

https://docs.aws.amazon.com/zh_cn/AWSCloudFormation/latest/UserGuide/using-iam-servicerole.html

5. CloudFormation模板參考：

 AWS CloudFormation 模板中使用的受支持资源、类型名称、内部函数和伪参数
https://docs.aws.amazon.com/zh_cn/AWSCloudFormation/latest/UserGuide/template-reference.html

6. CloudFormation模版範例：

https://docs.aws.amazon.com/zh_cn/AWSCloudFormation/latest/UserGuide/cfn-sample-templates.html

7. Code Services 

https://docs.aws.amazon.com/zh_tw/AWSCloudFormation/latest/UserGuide/sample-templates-services-us-west-2.html

8. AWS CloudFormation 限制

https://docs.aws.amazon.com/zh_cn/AWSCloudFormation/latest/UserGuide/cloudformation-limits.html


9. Important concepts:
  - TemplatesJson或YAML格式文字檔
  - Stacks: 套件 (堆疊)
  - Change Sets:修改彙整可讓你預先看到更新正在使用的服務會產生哪些影響

10.AWS CloudFormation運作方式
  當你產生一個stack時，AWS CloudFormation會呼叫AWS的相關服務來提供你範本所要的資源，首要條件是要有權限執行那些動作(IAM):

      (1)用AWS CloudFormation Designer或文字編輯器來設計範本，或直接用AWS現成的範本
      (2)範本可存放在本地端或S3上面
      (3)使用你的範本來創建AWS CloudFormation
      (4)使用Change Sets更新stack，當你發佈更新時，AWS CloudFormation會比對原始和新的stack有哪些不      同然而產生一個Change Sets，review後沒問題就可以執行更新
      (5)刪除stack，stack內的所有資源會一併被刪除，如果要保留部份資源就要產生一個delete policy

  

6. AWS CloudFormation 允許您在範本中**定義資源的刪除政策**。您可以指定在刪除 **Amazon EBS 磁碟區**或 **Amazon RDS 資料庫**執行個體前為它們**建立快照**。也可以指定在刪除堆疊時應保留的資源，不要將其刪除。如果要在**刪除堆疊時保留 Amazon S3 儲存貯體**，可以使用此功能
----------
# ★★非常重要--**AWS CloudFormation Designer**
1. 說明文件

簡言之，就是GUI撰寫CloudFormation
https://docs.aws.amazon.com/zh_cn/AWSCloudFormation/latest/UserGuide/working-with-templates-cfn-designer.html

2. 新手練習區

https://docs.aws.amazon.com/zh_cn/AWSCloudFormation/latest/UserGuide/working-with-templates-cfn-designer-additional-info.html

3. 傳送門

https://console.aws.amazon.com/cloudformation/designer


4. 元件關聯

https://docs.aws.amazon.com/zh_cn/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html


建置圖示

https://aws.amazon.com/cn/blogs/china/vpc-cloudformation-template/

----------

範例研究

## **Amazon Route 53**

https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/sample-templates-services-ap-northeast-1.html#w2ab1c23c28c13c29


## **Amazon Simple Storage Service**

https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/sample-templates-services-ap-northeast-1.html#w2ab1c23c28c13c31


## **Auto Scaling**

https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/sample-templates-services-ap-northeast-1.html#w2ab1c23c28c13b7


## **Amazon DynamoDB**

https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/sample-templates-services-ap-northeast-1.html#w2ab1c23c28c13c11


## **Amazon EC2**

https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/sample-templates-services-ap-northeast-1.html#w2ab1c23c28c13c13


## **AWS OpsWorks**

https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/sample-templates-services-ap-northeast-1.html#w2ab1c23c28c13c23


## **Amazon Simple Queue Service**

https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/sample-templates-services-ap-northeast-1.html#w2ab1c23c28c13c35


## **Amazon Virtual Private Cloud**

https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/sample-templates-services-ap-northeast-1.html#w2ab1c23c28c13c37




