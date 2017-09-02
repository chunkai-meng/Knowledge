# SQL SVR 2016  
## Install and Setting for access from another computer.   
> By Chunkai ([CK-SmartScreen@github](https://github.com/CK-SmartScreen/Knowledge))   
> This doc based on [(Connecting from Another Computer)](https://docs.microsoft.com/en-us/sql/relational-databases/lesson-2-connecting-from-another-computer)  and [(Create user and assign privileges)](http://jingyan.baidu.com/article/22fe7ced28dd1f3002617f88.html)   
> regarding to SQL SVR setting and User permission setting.

### Download and install
Download Developer edition from [SQL Server Download](https://www.microsoft.com/en-us/sql-server/sql-server-downloads). After finish installing the SQL SVR install SQL Server Configuration Manager(SSMC) as well.

### Enable TCP/IP connection from another Computer
1. Open SQL Server Configuration Manager.
2. Expand SQL Server Network Configuration, and then click Protocols for
The default instance (an unnamed instance) is listed as MSSQLSERVER.
3. In the list of protocols, right-click the protocol you want to enable (TCP/IP), and then click Enable.

### Opening Ports in the Firewall
**To open a port in the Windows firewall for TCP access (Windows 7/10)**
1. On the Start menu, click Run, type WF.msc, and then click OK.
2. In Windows Firewall with Advanced Security, in the left pane, right-click Inbound Rules, and then click New Rule in the action pane.
3. In the Rule Type dialog box, select Port, and then click Next.
4. In the Protocol and Ports dialog box, select TCP. Select Specific local ports, and then type the port number of the instance of the Database Engine. Type 1433 for the default instance.
5. In the Action dialog box, select Allow the connection, and then click Next.
6. In the Profile dialog box, select any profiles that describe the computer connection environment when you want to connect to the Database Engine, and then click Next.
7. In the Name dialog box, type a name and description for this rule, and then click Finish.

### Creating a new User for remote access
1. Open **Microsoft SQL Server Management Studio 17**, connect Database Engine default connection setting (Windows Authentication and default server name).
2. Expand the Database Server, right-click the Database add a new test db.
3. Then right-click the **Security** then **New** -> **login** Choose **SQL Server Authentication**, (optional: uncheck Enforce password policy), then input your new username and password then click **OK**.
4. Expand **Security** item right-click your new user, click **properties**.
5. In **Server Rules** page grant **serveradmin** privileges.
6. In **User Mapping** page map the test db.

### Connect the sql-server from another computer
1. Open Visual Studio 2015
2. Open **Server Explorer** side panel add a new connection.
3. Choose **SQL Server Authentication**
4. Input username and password.
5. test connection if your are asked to change password, go back to **Microsoft SQL Server Management Studio 17** then change the password, then try to connect again.

### Change **Connection String** in ASP.Net Core MVC project
open **appsettings.json** edit:
```
"ConnectionStrings": {
  //"defaultConnection": "Server=(localdb)\\mssqllocaldb;Database=ContosoUniversity3;Trusted_Connection=True;MultipleActiveResultSets=true"
  "DefaultConnection": "Server=192.168.20.2;Database=ContosoUniversity;user id=CK;password=xxxxxx"
},
```
Then you can have your own SQL SVR for use as a development.
