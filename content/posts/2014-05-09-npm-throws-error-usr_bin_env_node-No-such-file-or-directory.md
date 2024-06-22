---
title: "npm throws error /usr/bin/env: node: No such file or directory"
description: "solving the problems for npm throws error /usr/bin/env: node: No such file or directory"
date: 2014-05-09
---

If the nodejs server was installed via the package manager, it is not in:    
`/usr/bin/env/node`,     
it is in:      
`/usr/bin/env/nodejs`    

To solve this problem, you should set a symlink
```bash
ln -s /usr/bin/nodejs /usr/bin/node
```

credits for this hint: [digitalmediums at github](https://github.com/joyent/node/issues/3911#issuecomment-8956154)
