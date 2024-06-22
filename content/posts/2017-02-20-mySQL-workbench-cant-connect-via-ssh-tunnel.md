---
title: "mySQL workbench can't connect via ssh tunnel"
description: "how to solve ValueError: CTR mode needs counter parameter, not IV if your mySQL workbench can't connect to a database via ssh sshtunnel"
date: 2017-02-20
---

If you can't connect with your mysql-workbench to a database via ssh have a look at your logfile:
```bash
tail -f -n 100 ~/.mysql/workbench/log/wb.log
```
and you will see this error:
```bash
14:12:58 [ERR][sshtunnel.py:notify_exception_error:233]: Traceback (most recent call last):
File "/usr/share/mysql-workbench/sshtunnel.py", line 265, in _connect_ssh
look_for_keys=has_key, allow_agent=has_key)
File "/usr/lib/python2.7/dist-packages/paramiko/client.py", line 306, in connect
t.start_client()
File "/usr/lib/python2.7/dist-packages/paramiko/transport.py", line 465, in start_client
raise e
ValueError: CTR mode needs counter parameter, not IV
```
change your transport.py:
```
sudo nano /usr/lib/python2.7/dist-packages/paramiko/transport.py
```
press CTRL+W for searching for the term
```Python
return self._cipher_info[name]['class'].new(key, self._cipher_info[name]['mode']
```
and replace this
```Python
return self._cipher_info[name]['class'].new(key, self._cipher_info[name]['mode'], iv, counter)
```
with
```Python
return self._cipher_info[name]['class'].new(key, self._cipher_info[name]['mode'], '', counter)
```
as mentioned in the [Github pull request](https://github.com/paramiko/paramiko/pull/714/commits/4752287a7379da61245087ee7e35635a4e42bb3f)


All credits are going to user [hansaplast](http://stackoverflow.com/users/119861/hansaplast) for [his answer](http://stackoverflow.com/a/42029615/863403) on stackoverflow!
