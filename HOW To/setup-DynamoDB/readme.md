RDS Connections
Install MySQL/MariaDB Client
Copy paste the below in the EC2 user-data field

#!/bin/bash
yum -y install mariadb
How to Connect to RDS from EC2
mysql -h <Your-RDS-Endpoint> -P 3306 -u dbmaster -p
# For Example,
mysql -h myinstance.123456789012.us-east-1.rds.amazonaws.com -P 3306 -u dbmaster -p
If the above is giving errors, try disabling selinux by issuing this command setenforce 0 to troubleshoot the issue. Once you are set, reenable SELinux with the appropriate privileges.

Another area to be sure is to get the AWS Security Groups to allow traffic on port 3306 from your Source IP or web-app Security Group

Show Databases
show databases;
use <database-name>
You can retrieve the number of active connections to an Amazon RDS
SHOW STATUS WHERE `variable_name` = 'Threads_connected’;
Retrieve the maximum number of connections allowed for an Amazon RDS
SELECT @@max_connections;
We will look into CRUD basics now,

Create Tables
We will create a Students table with Student ID, Name & City as Columns
CREATE TABLE Students ( StudentID int, LastName varchar(255), FirstName varchar(255), City varchar(255) );
Insert records into tables
INSERT INTO Students ( StudentID, LastName, FirstName, City) VALUES ( "001", "Kumar", "Anil", "Singapore" );

INSERT INTO Students ( StudentID, LastName, FirstName, City) VALUES ( "002", "Reddy", "M", "Hyderabad" );

INSERT INTO Students ( StudentID, LastName, FirstName, City) VALUES ( "003", "Reddy", "N", "Hyderabad" );

INSERT INTO Students ( StudentID, LastName, FirstName, City) VALUES ( "004", "Vel", "D", "Chennai" );

INSERT INTO Students ( StudentID, LastName, FirstName, City) VALUES ( "005", "Student", "Martian", "Mars" );
Retrieve records from table
select * from Students;
Making specific queries
select StudentID from Students WHERE (LastName="Reddy");
select StudentID,City from Students WHERE (LastName="Reddy");
Delete Tables
drop tables Students;
Dynamo DB
Assuming you have a collection named aws-students - If not go ahead and create one, with the following instructions:

Create a new table aws-students
Set the Partition Key to studentId and Choose type as String
For Table Settings, Use Default Settings
Click Create
Lets get into CRUD - Create, Retrieve, Update, Delete.

You will need an EC2/linux client with aws cli configured to run the below commands
Create item
Lets create four student records(as json files) that will be inserted to the DynamoDB Collection called aws-students using aws cli command put-item

cat > "student001.json" << "EOF"
{"studentId":{"S":"1"},"studentDetails":{"S":"[{Name:'John Doe',Age:21,Sex:Male}]"}}
EOF

cat > "student002.json" << "EOF"
{"studentId":{"S":"2"},"studentDetails":{"S":"[{Name:'Jane Doe',Age:18,Sex:Female}]"}}
EOF

cat > "student003.json" << "EOF"
{"studentId":{"S":"3"},"studentDetails":{"S":"[{Name:'Old Mike',Age:881,Sex:''}]"}}
EOF

cat > "student004.json" << "EOF"
{"studentId":{"S":"4"},"studentDetails":{"S":"[{Name:'Young Mike',Age:18,Sex:}]"}}
EOF
Import the json using the below command

aws dynamodb put-item --table-name aws-students --item file://student001.json --return-consumed-capacity TOTAL
aws dynamodb put-item --table-name aws-students --item file://student002.json --return-consumed-capacity TOTAL
aws dynamodb put-item --table-name aws-students --item file://student003.json --return-consumed-capacity TOTAL
aws dynamodb put-item --table-name aws-students --item file://student004.json --return-consumed-capacity TOTAL
Since i am lazy, I do this instead,
for i in {1..4}
do
  aws dynamodb put-item --table-name aws-students --item file://student00$i.json --return-consumed-capacity TOTAL
done
Output
{
    "ConsumedCapacity": {
        "CapacityUnits": 1.0, 
        "TableName": "aws-students"
    }
}
Retrieve item from collection
Lets see if Old Mike's data had been correctly updated.

aws dynamodb get-item --table-name aws-students --key '{"studentId":{"S":"3"}}'
Output
{
    "Item": {
        "studentId": {
            "S": "3"
        },
        "studentDetails": {
            "S": "[{Name:'Old Mike',Age:81,Sex:'Male'}]"
        }
    }
}
Update item in Collection
Since Old Mike age is incorrectly updated as 881, Lets correct it.

cat > "student003.json" << "EOF"
{"studentId":{"S":"3"},"studentDetails":{"S":"[{Name:'Old Mike',Age:81,Sex:'Male'}]"}}
EOF
Run the put-item command
aws dynamodb put-item --table-name aws-students --item file://student003.json --return-consumed-capacity TOTAL
Delete item from Collection
Old Mike is no longer a student with our organization, Lets go ahead and remove him.

aws dynamodb delete-item --table-name aws-students --key '{"studentId":{"S":"3"}}'
Ref
[1] - AWS Docs

[2] - Dynamo DB Get Item Docs
