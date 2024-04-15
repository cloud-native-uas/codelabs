summary: Linux Basics on AWS
id: cloud-aws-instance-basics
categories: Linux, AWS
tags: introduction, exoscale, fhstp-nccs
status: Published
authors: Thomas Schuetz

# Linux Basics on AWS

<!-- ------------------------ -->

## What You’ll Learn

In this lab, you will:

- Create a Virtual Machine on AWS

## Set up an AWS Instance
Duration: 5

- Open the AWS Console
- Search for EC2
- Click on Launch a Virtual Machine
-   * Machine Name: MyFirstMachine
    * OS Image: Amazon Linux (2023)
    * Instance Type: t2.micro
    * Key Pair: Create a new key pair
    * Tags:
      * Name: my-first-instance
    * Launch instance

## Linux and Bash

Linux is an open-source operating system modeled on UNIX. Being open-source, the source code of Linux is freely available to its users. Its users can change its code to fix bugs or add improved features.

Key concepts:

- **Kernel**: The core part of the system that handles the CPU, memory, and peripheral devices. The Linux kernel is the interface between your software and your hardware.

- **Shell**: The shell is the utility that processes your requests. When you type in a command at your terminal, the shell interprets the command and calls the program that you want.

- **Terminal**: The terminal is a command line interface where you can type and execute text based commands.

Bash (Bourne Again SHell) is a shell program. It's widely available on various types of Unix-like operating systems, including Linux. Bash is almost synonymous with the shell; in fact, many people use the term "shell" to refer to Bash.

Key concepts:

- **Commands**: These are functions that you can run directly from the terminal. Some basic commands include `ls` (list directory contents), `cd` (change the current directory), `pwd` (print name of current directory), and `exit` (exit the shell).

- **Scripts**: A bash script is a file containing a list of commands to be executed by the bash shell. You can automate repetitive tasks using bash scripts.

- **Variables**: You can store data in variables to be used later in your scripts. You can also use special predefined variables to access specific system properties.

- **Control Structures**: Bash provides a variety of control structures for directing the flow of execution, including `if`/`else` statements, `for` loops, and `while` loops.

Remember, the best way to learn Linux and Bash is by practicing. Try to use the terminal for as many tasks as possible and write scripts to automate tasks. This will help you become familiar with the command line and improve your efficiency.

## Our first steps

* In the instance screen, click on the check mark besides the newly created instance. Afterward, click on the "Connect" button. This will show you how to connect to your instance. Choose "Connect using EC2 Instance Connect".

* A shell window will appear

In Linux "everything is a file". Therefore, it makes sense to make yourself a bit familiar with some file operations.

Let's create our first directory:

```bash
mkdir ~/my-first-directory
```

The tilde means that you will create this directory underneath your home directory. Alternatively you could also type in mkdir `/home/ec2-user/my-first-directory`.

Now let's change into this directory:

```bash
cd ~/my-first-directory
```

You can check your current directory with the `pwd` command. This will print the current directory to the terminal.

Now let's create a file:

```bash
touch my-first-file.txt
```

The `touch` command will simply create an empty file. As we want to put some contents into the file, we can use the `echo` command:

```bash
echo "Hello World" > my-first-file.txt
```

If the file wouldn't already exist, this would create it.

Now let's print the contents of the file to the terminal:

```bash
cat my-first-file.txt
```

The cat command will print the contents of the file to the terminal. If you want to change the contents, you can use an editor, such as nano.

```
nano my-first-file.txt
```

This will open the nano editor and you can change the contents of the file. The detailed commands for nano are shown at the bottom of the terminal.

### Your first steps in Linux
* Create a directory under `/tmp`
* Create a file in this directory
* Write some text into the file

### Summary
In this task, you have learned very basic Linux commands and how to create a file and directory. This is the first step to becoming familiar with the Linux operating system.

## Scripting
One of the most powerful features of Linux (and bash) is the ability to automate tasks using scripts. A script is a file containing a list of commands that can be executed by the shell. This allows you to automate repetitive tasks and save time.

Let's create a simple script that prints "Hello, World!" to the terminal.

Therefore, create a file called `hello.sh`:

```bash
nano hello.sh
```

Now, add the following content to the file:

```bash
#!/bin/bash
echo "Hello, World!"
```

Save the file and exit the editor.

**File permissions in Linux**
In Linux, every file and directory has a set of permissions associated with it. These permissions determine who can read, write, and execute the file or directory.  Permissions are divided into three groups: 

*User:* The owner of the file. The user can have read, write, or execute permissions.  

*Group:* The group that owns the file. Users who are part of this group can have read, write, or execute permissions.  

*Other:* All other users who are not the owner or part of the group. They can also have read, write, or execute permissions.  

Each permission is represented by a letter:  

*Read (r):* The file can be opened and its content viewed.  
*Write (w):* The file can be modified or deleted.  
*Execute (x):* The file can be run as a program.  
You can view the permissions of a file or directory using the ls -l command in the terminal. The permissions will be displayed in the form of a string, such as -rwxr-xr--. This string can be broken down as follows:  
The first character indicates the type of file. A - indicates a regular file, while a d indicates a directory.
The next three characters represent the user permissions (in this case, rwx means the user can read, write, and execute).
The following three characters represent the group permissions (here, r-x means the group can read and execute, but not write).
The final three characters represent the permissions for others (in this case, r-- means others can only read).
You can change the permissions of a file or directory using the chmod command followed by the permission set you want to apply, and the name of the file or directory. For example, chmod 755 filename would give the user full permissions, and read and execute permissions to the group and others.

In our case, we want to make the script executable. Therefore, we have to change the permissions of the file:

```bash
chmod +x hello.sh
```

Now you can run the script:

```bash
./hello.sh
```

This will print "Hello, World!" to the terminal.

This was the first step in scripting. You can now create more complex scripts to automate tasks and improve your efficiency.

Now, let's create a script that prints "Hello, {{country-name}}" to the terminal. Therefore, create a file called `hello-country.sh`:

```bash
#!/bin/bash

countries=("Austria" "Germany" "Switzerland" "Italy" "France")

for country in "${countries[@]}"; do
  echo "Hello, $country!"
done
```

Save the file and exit the editor.

Now, make the script executable:

```bash
chmod +x hello-country.sh
```

After running this script, you should get greetings to different countries in your shell.

### A new task
Create a script that prints "Hello, {{different names}}" to the terminal. Your name should be part of it and when your name is printed, add "This is me!" to the output. Therefore you will have to use an if statement which might look like this:

```bash
if [ conditional == "Thomas" ]; then
  echo "output, This is me!
else
  echo "output"
fi
```

### Summary
In this task, you have learned how to create a simple script and how to make it executable. You have also learned how to use a for loop to iterate over an array and print the elements to the terminal. This is the first step in scripting and automation.

## Running a simple webserver
In this task, we will install a simple webserver on our virtual machine. We will use the `httpd` package, which is the Apache webserver.

**Packages in Linux**
In Linux, a package is a compressed file archive that contains all of the files that come with a particular application. The files are usually stored in a specific directory hierarchy that the application expects. A package also contains metadata, such as the software's name, description of its purpose, version number, vendor, checksum, and a list of dependencies necessary for the software to run properly.  Packages in Linux come in different formats depending on the distribution. For example, Debian-based distributions (like Ubuntu) use .deb files, while Red Hat-based distributions (like Fedora) use .rpm files.  Package management systems, such as APT (for Debian-based systems) or YUM (for Red Hat-based systems), are used to install, upgrade, or remove software packages in Linux. These tools handle dependencies and will automatically download and install any required packages for a software package.  For example, to install a package on a Debian-based system, you would use the apt-get install command followed by the package name. On a Red Hat-based system, you would use the yum install command.  In addition to these standard package management systems, many languages have their own package managers, such as npm for Node.js, pip for Python, and gem for Ruby.  Remember, it's important to keep your system and packages up-to-date to ensure you have the latest features and security updates.

Let's try to find out, if the `httpd` package exists:

```bash
yum search httpd
```

As we see, the package is available. Let's install it:

```bash
sudo yum install httpd -y
```

We need sudo in this case, as we are installing software on the system and this can only be achieved using superuser permissions (sudo). The `-y` flag will automatically answer "yes" to all questions.

Now, let's start the webserver:

```bash
sudo service httpd start
```

You can check if the webserver is running by opening a browser and typing in the public IP address of your instance (. You should see the default Apache page.

Using `curl localhost` you can also check if the webserver is running.

### Add a new website to the webserver
Simply create a new file in the directory `/var/www/html/index.html and add your content. Afterward, try to curl the content from the webserver.

### Summary
You created a simple webserver using the Apache webserver package. You learned how to install a package using the package manager and how to start a service. You also learned how to check if the service is running and how to access it using a browser.

## Making your server accessible
In this task, we will make our server accessible from the public internet. Therefore, we have to open the port 80 in the security group of our instance.

* Open the AWS Console
* Search for EC2
* Find the instance you created
* Click on the Security Group Tab
* Click on the Security Group
* Click Inbound Rules
* Add a new rule
  * Type: HTTP
  * Source: 0.0.0.0/0
  * Port: 80
* Save the rule

Now you should be able to access the Webserver

### Putting it all together
* Delete the instance
* Create a new instance
* Install a webserver (maybe nginx)
* Add your own content to the webserver




