title: "MongoDB4 事务 简单易懂的\U0001F330"
date: 2018-09-30 10:30:25
categories: mongodb
tags: mongodb
---
### 简介

mongodb 的事务是依靠 mongodb 连接的客户端 session 实现，事务执行的流程大致是
建立 session，通过 session startTransaction 启动事务，如果一系列事务都完成，那么 commitTransaction 完成事务操作，并结束当前事务 session；如果一系列事务中有任意事件失败， 那么 abortTransaction 中止事务，内部将已完成的任务回退到修改之前，并结束当前事务 session。
<!--more-->
```javascript
session = client.startSession();
session.startTransaction();
session.commitTransaction();
session.abortTransaction();
session.endSession();
```

### 场景描述

当前有两个用户，A 用户有余额 50 软妹币，B 用户有余额 10 软妹币，A 用户给 B 转账 10 ，场景设定 转账很安全，网络也很畅通，没有黑客拦截，没有发生意外，这次转账成功了，这时 A 用户余额剩下40 ，B 用户余额有20。A 感觉很安全，这时又给 B 转账，A 忘记自己余额有多少，给 B 转了 50 ，结果出错了。

在没有事务的情况下，操作数据库是这样的，1.A 账户的余额 -50，2. B 账户增加50. 当 A 余额不足时或在操作 A 账户成功后网络发生错误，B 账户的金额没能正确修改。

在有事务的情况下，即使在操作 A 账户金额后出现错误，则事务会将整个转账过程回退到修改之前。

### 下载 mongodb4 并解压

[https://www.mongodb.com/download-center#community](https://www.mongodb.com/download-center#community
)

```
wget https://fastdl.mongodb.org/osx/mongodb-osx-ssl-x86_64-4.0.0.tgz

tar -xzvf mongodb-osx-ssl-x86_64-4.0.0.tgz

cd mongodb-osx-ssl-x86_64-4.0.0
```

> !!! Transaction 只适用复制集 Replica Set， 所以要先搭建mongodb 复制集

### 启动多个 mongodb 实例

```

// 创建data目录
mkdir -p data/1301
mkdir -p data/1302
mkdir -p data/1303

// 起三个 mongodb 实例
./bin/mongod --replSet shard1  --dbpath=./data/1301 --port=1301
./bin/mongod --replSet shard1  --dbpath=./data/1302 --port=1302
./bin/mongod --replSet shard1  --dbpath=./data/1303 --port=1303
```

![](https://user-gold-cdn.xitu.io/2018/7/9/1647f1a58ed8c492?w=2946&h=2138&f=png&s=3000404)

```
// 配置复制集
./bin/mongo --port 1301
rsconf = {_id: "shard1", members: [ { _id: 0, host: "127.0.0.1:1301" } ] }
rs.initiate( rsconf )
rs.add("127.0.0.1:1302")
rs.add("127.0.0.1:1303")
```

![](https://user-gold-cdn.xitu.io/2018/7/9/1647f1b57edd5e34?w=3814&h=1894&f=png&s=1012311)

![](https://user-gold-cdn.xitu.io/2018/7/9/1647f1c9b793bc2a?w=2706&h=1472&f=png&s=460551)

```
//查看是否是主节点
rs.isMaster()

// 查看复制集状态
rs.status()
```

![](https://user-gold-cdn.xitu.io/2018/7/9/1647f1ce6c4826ae?w=2706&h=1472&f=png&s=460551)

### 编码

```
mkdir mongodb4
cd mongodb4
npm init
npm i mongodb -S
vi app.js
```

```javascript
//app.js

(async function()  {

  // 连接DB
  const { MongoClient } = require('mongodb');
  const uri = 'mongodb://localhost:1301/dbfour';
  const client = await MongoClient.connect(uri, { useNewUrlParser: true });

  const db = client.db();
  await db.dropDatabase();
  console.log('(1) 首先 删库 dbfour, then 跑路\n')

  // 插入两个账户并充值一些金额
  await db.collection('Account').insertMany([
    { name: 'A', balance: 50 },
    { name: 'B', balance: 10 }
  ]);
  console.log('(2) 执行 insertMany,  A 充值 50, B 充值 10\n')


  await transfer('A', 'B', 10); // 成功
  console.log('(3) 然后 A 给 B 转账 10\n')

  try {
    // 余额不足 转账失败
    console.log('(4) A 再次转账给 B 50\n')
    await transfer('A', 'B', 50);

  } catch (error) {
    //error.message; // "Insufficient funds: 40"
    console.log(error.message)
    console.log('\n(5) A 余额不够啊，所以这次转账操作不成功')
  }

  // 转账逻辑
  async function transfer(from, to, amount) {
    const session = client.startSession();
    session.startTransaction();
    try {
      const opts = { session, returnOriginal: false };
      const A = await db.collection('Account').
        findOneAndUpdate({ name: from }, { $inc: { balance: -amount } }, opts).
        then(res => res.value);
      if (A.balance < 0) {
        // 如果 A 的余额不足，转账失败 中止事务
        // `session.abortTransaction()` 会撤销上面的 `findOneAndUpdate()` 操作
        throw new Error('Insufficient funds: ' + (A.balance + amount));
      }

      const B = await db.collection('Account').
        findOneAndUpdate({ name: to }, { $inc: { balance: amount } }, opts).
        then(res => res.value);

      await session.commitTransaction();
      session.endSession();
      return { from: A, to: B };
    } catch (error) {
      // 如果错误发生，中止全部事务并回退到修改之前
      await session.abortTransaction();
      session.endSession();
      throw error; //使其调用者 catch error
    }
  }

})()

```

###  查看结果

```
node app.js
```

![](https://user-gold-cdn.xitu.io/2018/7/9/1647f1cc10aa9b99?w=1166&h=482&f=png&s=79963)