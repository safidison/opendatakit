# Introduction #

Following are basic details on setting up ODK Aggregate to run on a Linux micro-instance on the Amazon Web Services EC2 infrastructure. For screenshots and more on the general set-up of Tomcat on AWS, see the excellent three-part “Cat in the Cloud: Apache Tomcat in Amazon EC2” series at http://www.excelsior-usa.com/articles/tomcat-amazon-ec2-basic.html. Amazon’s getting-started guides are also quite helpful: http://aws.amazon.com/documentation/gettingstarted/.


# Details #

  1. First, sign up for Amazon Web Services EC2 at http://aws.amazon.com/ec2/.
    * Note that you will need to confirm a working phone number. If you want to use a U.S. number but are presently overseas, note that you can use a Google Voice number routed to Google Chat.
  1. Go to the AWS/EC2 management console and note your region (shown in the upper-left).
  1. Launch a new instance with the Launch Instance button prominently displayed on the EC2 console home screen. Accept the default behavior and use the quick-launch wizard.
    * Create a new key pair, download the private key, and keep the private key safe. This will be your only method of communicating with your new instance and you will not be allowed to download it again.
    * For the launch configuration, choose the “Amazon Linux AMI: EBS-Backed (64-bit)” option. (The exact AMI name and ID will depend on your region.) This is one of the instance types that you can run on a micro-instance as part of their “free tier” (http://aws.amazon.com/free/).
    * Leave everything else at the defaults, including the instance type. The instance type will default to “t1.micro” which is a small, limited instance that can be run for free. For a price, you can upgrade the instance type later if you need better performance.
  1. After creating the instance, add security rules for allowing both HTTP and HTTPS.
    * Choose the Security Groups tab and click on the auto-created security group associated with your new instance. (This might have been called “quicklaunch-1”. If you’re not sure, you can go to the Instances tab to see which Security Group is listed for the new instance.)
    * In the properties pane at the bottom, click to the Inbound tab, select HTTP from the “create a new rule” drop-down, then click Add Rule. Do the same for HTTPS. Then click Apply Rule Changes.
    * To avoid potential problems with MTU settings and packet loss, also add a rule to allow "All ICMP". After you create the new rule, click Apply Rule Changes.
    * (Tomcat defaults to listening on nonstandard ports 8080 and 8443. Below will be instructions to use the standard HTTP and HTTPS ports instead. However, if you want to leave Aggregate on the non-standard ports, you can certainly do so; in that case, simply add two additional security rules to allow access via 8080 and 8443.)
  1. Switch to the Instances tab, click on your instance, and note its Public DNS Address in the properties pane below. This is the default address that you will use to access your instance.
  1. Presuming that you want a friendlier way to access your instance, allocate it an “elastic IP” and domain name.
    * Navigate to Elastic IPs and click Allocate New Address. Associate it with your new instance.
    * Note that the IP is free so long as you keep it associated with a running instance. If you stop your instance and do not release the IP address for others to use (in essence, wasting it), then Amazon will begin charging you for holding the unused address.
    * Note your new IP. Also, if possible, configure DNS to route one or more names to this address. You can then use this IP and/or name to access your instance (and can forget the “Public DNS Address” assigned by AWS).
      * Note that when you set up Aggregate below, you will need to configure it with the domain name you will use to access it. Thus, it is best if you configure the domain name first.
  1. Connect to your instance.
    * Go to the Instances tab and select Connect from the Instance Actions drop-down (alternatively, you can right-click on the instance and choose Connect).
    * From here AWS presents you with several options.
      * The easiest is to connect using their Java SSH client. If you choose that option, you just have to specify the location of your private key file (created above) and AWS launches an in-browser SSH client to connect to your instance.
      * You can also select to connect with a stand-alone SSH client. If you choose this option, AWS will provide extremely helpful instructions, including an SSH command that you can cut and paste into your local command window. It will also inform you that you may need to update the permissions on your local private key file in order for the ssh client to run properly, and it will even give you the command to run (e.g., “chmod 400 xxx.pem”).
    * Once you connect, you will probably be told that there are new security updates to install. You can run “sudo yum update” to install these updates, as it advises.
  1. Transferring files to/from your instance.
    * When you login via ssh, you will default to being in the (empty) ec2-user home directory. You will want to be able to transfer files between here and your local directory. You have several options.
      * If you’re using the command-line ssh, you can also use the command-line scp to copy files. The syntax is similar to ssh, but of course you also need to specify the source and destination file paths.
    * An easier option is to use an FTP program like FileZilla (as long as it supports SFTP).
      * To configure FileZilla to connect to your instance, go into Preferences…Connection…SFTP and add your private key to FileZilla’s keystore (it will offer to convert the key format, which you should accept). Then, go into Site Manager and create a new site. The host should be the IP, name, or Public DNS for your instance, the port can be blank, the protocol should be “SFTP – SSH File Transfer Protocol,” the login type should be Normal, and the user should be “ec2-user”. Everything else should be left at the defaults, including the password (which will be blank).
      * When you connect, the default directory will be the ec2-user’s home directory, but you can also navigate to other directories.
  1. Install Tomcat 6.
    * This can be done by simply running “sudo yum install tomcat6”. This installs configuration files into /etc/tomcat6 and other files into /usr/share/tomcat6. Log files go into /var/log/tomcat6.
  1. Install and run MySQL.
    * This can be done by simply running “sudo yum install mysql mysql-server” followed by “sudo service mysqld start”.
  1. Install and transfer ODK Aggregate files.
    * First, install ODK Aggregate on your local computer (not on your AWS instance).
      * During set-up, it’s important to specify that this will be a MySQL installation, and it is also very important that you specify the correct domain name or IP address that will be used to access your Aggregate server. Ideally, this will be a specific domain name that you have already mapped to an elastic IP (and can re-map later if you change the IP).
      * You will also specify a Google account address with which you can later login to your Aggregate instance.
    * The installation will create a create\_db\_and\_user.sql file. Upload this to your ec2-user home directory. (See above for how to transfer files.)
    * The installation will also create an ODKAggregate.war file. Rename this to ROOT.war and upload it to the /usr/share/tomcat6/webapps folder.
  1. Configure MySQL.
    * On your AWS instance, run “/usr/bin/mysql\_secure\_installation” to set a root password and generally secure your MySQL installation.
    * Then, run “mysql –u root -p” to log in to MySQL (specifying the password you just set), and type “source ~/create\_db\_and\_user.sql”. This will create the ODK user and database.
    * Finally, run “sudo /sbin/chkconfig --levels 235 mysqld on” to auto-start MySQL whenever your instance boots up.
  1. Configure Tomcat.
    * Download the MySQL Connector/J from the MySQL download site (http://dev.mysql.com/downloads/connector/j/), unzip it, and transfer the “mysql‐connector‐java‐x.x.x‐bin.jar” file up to your instance’s /usr/share/tomcat6/lib directory.
    * Edit /etc/tomcat6/server.xml in order to customize settings. (If you’re not used to Linux text editors, you can always download the file, edit it, and upload it back.)
      * Assuming that you want to run Aggregate on the standard HTTP port (80) and HTTPS port (443):
        * Change “<Connector port="8080" protocol="HTTP/1.1"” to “<Connector port="8080" proxyPort="80" protocol="HTTP/1.1"” (i.e., add the proxyPort attribute).
        * Also change “<Connector port="8443" protocol="HTTP/1.1" SSLEnabled="true"” to “<Connector port="8443" proxyPort="443" protocol="HTTP/1.1" SSLEnabled="true"”.
        * Execute the following commands to have Linux forward to the ports on which Tomcat listens:
          * sudo /sbin/iptables -t nat -I PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080
          * sudo /sbin/iptables -t nat -I PREROUTING -p tcp --dport 443 -j REDIRECT --to-port 8443
          * sudo /sbin/service iptables save
      * If you have an SSL certificate for HTTPS support:
        * Make sure that the “<Connector port="8443"” part of the configuration file is not commented out. If it is, un-comment it.
        * Upload your SSL keystore file and the certificate(s) to the server.
        * Install it as instructed. (If you buy from RapidSSL, for example, they provide you with Tomcat installation instructions. E.g., you may need to download a special P7S certificate file, then install it on the server with “keytool -import -alias YOURALIAS -trustcacerts -file xxxxx.p7s  -keystore xxxxx.keystore”.)
        * In the “<Connector port="8443"” part of the configuration file, specify the location of your keystore file and password (e.g., “keystoreFile="/…/xxxxx.keystore" keystorePass="changeit"”).
    * Start Tomcat with “sudo service tomcat6 start”.
    * Configure Tomcat to auto-start when the instance boots with “sudo chkconfig --level 345 tomcat6 on”.
  1. Login and test.
    * At this point, you should be able to login to your AWS-hosted Aggregate instance by going to its name or IP in your web browser (with or without HTTPS, depending on your set-up).
    * For your first login, you will need to login with the Google account you specified during the Aggregate installation process. Then you can add additional users from the Site Admin tab.
  1. Once you have confirmed that your Aggregate instance is working, you can back it up by creating an image of the instance (an AMI).
    * You can do this by going to the Instances tab in the AWS-EC2 console, then selecting the “Create Image (EBS AMI)” Instance Action for your instance.
  1. Further set-up for production servers.
    * You will want to create a system to monitor and manage the log files in /var/log/tomcat6.
    * You will also want to create a system for regular back-ups and a plan for how to restore them when needed. This will need to take care to safely back up the MySQL database, which may be in-use at any given time.
    * Note that the micro instance is only free for 12 months from AWS sign-up, and that you may exceed the free quotas on disk space or network bandwidth before that point (see http://aws.amazon.com/free/).
    * Note also that you may at some point need to upgrade your instance to a “standard instance” if the micro instance is not providing enough performance.