---
title: "install node.js from binaries"
date: 2017-01-31
---

Source: [http://www.thegeekstuff.com/2015/10/install-nodejs-npm-linux/](http://www.thegeekstuff.com/2015/10/install-nodejs-npm-linux/)    

```bash
wget https://nodejs.org/dist/v7.4.0/node-v7.4.0-linux-x64.tar.gz
cd /usr/local
tar --strip-components 1 -xzf /path/to/node-v7.4.0-linux-x64.tar.gz
```
node should be in the newest version:
```bash
node -v
```
