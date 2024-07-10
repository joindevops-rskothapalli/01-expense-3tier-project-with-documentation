Backend service is responsible for adding the given values to database. Backend service is written in NodeJS, Hence we need to install NodeJS.
In a different project, backend may be written in java, python etc, if so, then we need to install respective softwares. 

**Developer has chosen NodeJs, Check with developer which version of NodeJS is needed. Developer has set a context that it can work with NodeJS >20**

Install NodeJS. By default NodeJS 16 is available, We would like to enable 20 version and install this.

**You can list modules by using dnf module list**

```
dnf module disable nodejs -y(This will disable whatever version of nodeJS is there by default)
```
```
dnf module enable nodejs:20 -y(Now we enabled version 20)
```

```
dnf install nodejs -y
```

Configure the application.

Add application User
```
useradd expense
```

User expense is a function / daemon user to run the application. Apart from that we don't use this user to login to server.

Also, username expense has been picked because it more suits to our project name.

We keep application in one standard location. This is a usual practice that runs in the organization.

Lets setup an app directory.

```
mkdir /app
```

Download the application code to created app directory.

```
curl -o /tmp/backend.zip https://expense-builds.s3.us-east-1.amazonaws.com/expense-backend-v2.zip
```
```
cd /app
```
```
unzip /tmp/backend.zip
```

Every application is developed by development team will have some common softwares that they use as libraries. This application also have the same way of defined dependencies in the application configuration.

Lets download the dependencies.

```
cd /app
```
```
npm install
```

We need to setup a new service in systemd so systemctl can manage this service

Setup SystemD Expense Backend Service
```
vim /etc/systemd/system/backend.service
```

```
[Unit]
Description = Backend Service

[Service]
User=expense
Environment=DB_HOST="<MYSQL-SERVER-PRIVATE-IPADDRESS>"(We use private IP vs public because, backend will only talk wth DB internally, so it's good to use Pri.IP so it can talk to DB quicly)
ExecStart=/bin/node /app/index.js
SyslogIdentifier=backend

[Install]
WantedBy=multi-user.target
```

**NOTE: Ensure you replace <MYSQL-SERVER-PUBLIC-IPADDRESS> with db private IP address**

Load the service.

```
systemctl daemon-reload
```

Start the service.
```
systemctl start backend
```
```
systemctl enable backend
```

For this application to work fully functional we need to load schema to the Database.

We need to load the schema. To load schema we need to install mysql client.

To have it installed we can use

```
dnf install mysql -y
```

Load Schema

```
mysql -h <MYSQL(db)-SERVER-PRIVATE-IPADDRESS> -uroot -pExpenseApp@1 < /app/schema/backend.sql
```

Restart the service.
```
systemctl restart backend
```
