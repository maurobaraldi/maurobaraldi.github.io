# Writing logs in /var/log

It is very common for applications, even more common that one then runs on server side, to write logs at any moment. Sometimes for auditing, sometimes for security or even for debuging. Unix/Linux systems have **Syslog** service, that is the general standard service and protocol for logging. The default path for log files in Linux system is `/var/log/`. This is the best, and IMHO, the right place to store logs.

As Syslog is a service, it has a proper user to run this service in Linux systems so you must have permission to read/write this log files. But, usually, only **root** and **syslog** users have access to this files.

**So, why not give access to user to /var/log?**

Because one of the principles of security of systems is the isolation of the data. Each application should have access only to its own resources. Database service should not have access to logs (or any other resources) of web server services and vice and versa.

There are many ways to solve this problem and each one of them has a tradeoff. I will show 3 ways to handle with this issue, and the pros and cons of each one.

## 1 - Give access direct to file

This is the most common approach to solve this problem. Just write the log at the same path of applications, or in another arbitrary path. But this is a very bad practice because the logs should be stored in a different place. Logs are a kind of data of application, pretty much like a database.

Anyway, this the way we can setup a logging application in `/var/log` as user.

- Create a file in `/var/log/my-app.log` 
- Change access to file (change owner).
- Test read/write file

```
$ sudo touch /var/log/my-app.log
$ sudo chown mauro:root /var/log/my-app.log
$ echo "Some log to test." > /var/log/my-app.log
$ cat /var/log/my-app.log
Some log to test.
```

### Pros

This is a fast and effecive solution, ok for simple pruposes like a small script for a punctual task or debug.

### Cons

This solution doesn't scale even for small applications with a few amount of files. You need to create and give access to each file. Now try to imagine it with set modules integrated in an application.

It does not provide logging handler minimal essential features like rotate log file and formating log info as date/time of registries.

This solution is as good (or bad) as save log file in same path of application.

After all, even if you change the owner of file, you may need to change file permissions. And I have to repeat: changing the file permission may not be a good idea. 

## 2 - Changing ACL of /var/log to user's application.

Everytime I walk into a solution that results in handling with ACLs (Access Control Lists) I think that solution comes with another problem: Manage ACLs. As a developer, I don't want to care about system stuffs. That's because good languages and platforms have good logging libraries.

Even so, this is a better approach to handle with logs than give direct access to files at all.

- Create a file in `/var/log/my-app.log` 
- Change ACL to file.
- Test read/write file

```
$ sudo touch /var/log/my-app.log
$ sudo setfacl -m u:mauro:rw /var/log/my-app.log
$ echo "Some log to test." > /var/log/my-app.log
$ cat /var/log/my-app.log
Some log to test.
```

### Pros

This solution is effecive and secure and can be applied also to directories. That means that all new files created by this user on `/var/log` also works.

### Cons

High level of complexity. As I said before, when I'm using the developers hat, I prefer to focus my effort on coding stuffs.

This approach may bring some complexity to application architecture and design, because I need to handle the possible issues trying to write the application logs.

## 3 - Using a logging library

As I said before, some programming languages has rich standard library. It means that one of its advantage is that there is a lib to integrate with most useful OS features. When I've got the inspiration to write this post, I was trying to solve a problem using [Python](https://www.python.org), and luckly it has a very good [logging library](https://docs.python.org/3/library/logging.handlers.html). Before long, I had to use logging with [Go](https://golang.org/) and luckly again, it has a [logging handler](https://golang.org/pkg/log/syslog/) in its standard lib.

If you have Syslog enabled and running, you should consider using it, just because Syslog is a logging service. So it was designed and engineered to solve logging issues. Try to re-invent the wheel should never be one of your first options.

Some other languages that has integration with Syslog are [Ruby](https://docs.ruby-lang.org/en/2.1.0/Syslog/Logger.html) and  [PHP](https://www.php.net/manual/en/function.syslog.php) 

### Pros

Integration with OS logging services, if you use Linux/Unix OS based, library that handles with permission issues as others.

### Cons

These libraries may not work with systems that do not support Syslog like some embbeded systems or non Unix/Linux derived OS.

# Conclusion

If the progamming language has support to Syslog I recommend you to logging this way. If you don't have many flexibility, as in embedded system, other solutions described here could solve your problem.

## References and More Info

[Syslog](https://en.wikipedia.org/wiki/Syslog)

[Why syslog is a user](https://askubuntu.com/questions/1109859/why-is-syslog-a-user)

[setfacl](https://linux.die.net/man/1/setfacl)

[Permissions to write to log](https://unix.stackexchange.com/questions/479890/permission-to-write-to-log)
