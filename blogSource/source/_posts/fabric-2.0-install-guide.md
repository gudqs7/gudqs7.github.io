---
title: Fabric-2.0-尝鲜体验（安装，HelloWorld 链码）
date: 2020-05-20 08:48:14
tags: 
  - fabric
categories:
  - blockchain
  - fabric
---

### 基本环境准备

- Docker
- docker-compose
- docker 阿里镜像加速
- Git

#### 对于 Mac

> 上诉环境只需要下载 docker的 dmg 安装后即可拥有 docker 和 docker-compose

#### 对于 CentOs

##### docker

```bash
sudo yum update
# 移除旧版本
yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-selinux docker-engine-selinux docker-engine
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

#安装 docker
sudo yum install docker-ce
sudo systemctl start docker
# 开机启动
sudo systemctl enable docker
# 验证
docker --version
```

##### docker-compose

```bash
curl -L "https://github.com/docker/compose/releases/download/1.22.0/docker-compose-(uname−s)−(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

##### git

```bash
sudo yum install -y git
```

##### go

```bash
wget -c https://studygolang.com/dl/golang/go1.14.linux-amd64.tar.gz
tar -C /usr/local/ -zxvf go1.14.linux-amd64.tar.gz
vi /etc/profile
# 新增
export PATH=$PATH:/usr/local/go/bin
export GOROOT=/usr/local/go
export GOPATH=/root/go/

source /etc/profile
#验证 
go version
```



### 下载文件及 docker 镜像

#### 一键脚本

```bash
curl -sSL https://bit.ly/2ysbOFE | bash -s -- 2.0.0 1.4.4 0.4.18
```

- 2.0.0：表示Hyperledger Fabric的版本号
- 1.4.4：表示Fabric CA的版本号
- 0.4.18：表示第三方引用的版本号

#### 加速技巧

> 首先自己整个科学上网, 设置好 http 代理端口(如 1087)

然后在docker 偏好设置中设置代理, 如下图

![image-20200310095814860](https://tva1.sinaimg.cn/large/00831rSTgy1gcolmgcgpfj30rs0s60wn.jpg)

设置完成后点击 Apply & Restart 重启生效.

> 若没有科学上网, 则可以手动执行 docker pull 命令, 单个执行不容易卡住

```bash
docker pull hyperledger/fabric-peer:2.0.0
docker pull hyperledger/fabric-orderer:2.0.0
docker pull hyperledger/fabric-ccenv:2.0.0
docker pull hyperledger/fabric-tools:2.0.0
docker pull hyperledger/fabric-nodeenv:2.0.0
docker pull hyperledger/fabric-baseos:2.0.0
docker pull hyperledger/fabric-javaenv:2.0.0
docker pull hyperledger/fabric-ca:1.4.4
docker pull hyperledger/fabric-zookeeper:0.4.18
docker pull hyperledger/fabric-kafka:0.4.18
docker pull hyperledger/fabric-couchdb:0.4.18
```

#### 最后检查

```bash
cd fabric-samples/bin
ls -l
export PATH=$PATH:$(pwd)
orderer version
```

> 1.存在 bin 目录
>
> 2.bin 下存在 orderer 等可执行文件
>
> 3.将 bin 添加到 PATH 下
>
> 4.检查 orderer 版本是否正确

#### 测试网络

```bash
cd fabric-samples/test-network
#注意会删除全部容器
docker stop $(docker ps -q) 
docker rm $(docker ps -aq)
./network.sh up
```

> 1.进入测试网络目录
>
> 2.停止并删除之前的容器防止命名冲突
>
> 3.运行测试网络(通过 docker-compose)



### 链码入门示例(智能合约)

#### 启动环境及安装 npm

```bash
cd fabric-samples/fabcar
#将启动 first-network 环境, 然后将早已存在镜像中的链码代码打包, 安装, 验证, 提交, 使其可执行
./startFabric.sh javascript
cd javascript
# 自行安装cnpm
cnpm i
```

#### 测试链码

官方提示

```
JavaScript:

  Start by changing into the "javascript" directory:
    cd javascript

  Next, install all required packages:
    npm install

  Then run the following applications to enroll the admin user, and register a new user
  called user1 which will be used by the other applications to interact with the deployed
  FabCar contract:
    node enrollAdmin
    node registerUser

  You can run the invoke application as follows. By default, the invoke application will
  create a new car, but you can update the application to submit other transactions:
    node invoke

  You can run the query application as follows. By default, the query application will
  return all cars, but you can update the application to evaluate other transactions:
    node query
```



> 通过SDK(js) 调用合约

```shell
#添加 admin 用户和普通用户 user1
node enrollAdmin.js
node registerUser.js
# 执行合约的 queryAll 方法
node query.js
# 执行合约的 createCar 方法
node invoke.js
# 重新查询
node query.js
```



#### 实际合约(js 代码)

`fabric-samples/chaincode/fabcar/javascript/lib/fabcar.js`

```js
/*
 * SPDX-License-Identifier: Apache-2.0
 */

'use strict';

const { Contract } = require('fabric-contract-api');

class FabCar extends Contract {

    async initLedger(ctx) {
        console.info('============= START : Initialize Ledger ===========');
        const cars = [
            {
                color: 'blue',
                make: 'Toyota',
                model: 'Prius',
                owner: 'Tomoko',
            },
            {
                color: 'red',
                make: 'Ford',
                model: 'Mustang',
                owner: 'Brad',
            },
            {
                color: 'green',
                make: 'Hyundai',
                model: 'Tucson',
                owner: 'Jin Soo',
            },
            {
                color: 'yellow',
                make: 'Volkswagen',
                model: 'Passat',
                owner: 'Max',
            },
            {
                color: 'black',
                make: 'Tesla',
                model: 'S',
                owner: 'Adriana',
            },
            {
                color: 'purple',
                make: 'Peugeot',
                model: '205',
                owner: 'Michel',
            },
            {
                color: 'white',
                make: 'Chery',
                model: 'S22L',
                owner: 'Aarav',
            },
            {
                color: 'violet',
                make: 'Fiat',
                model: 'Punto',
                owner: 'Pari',
            },
            {
                color: 'indigo',
                make: 'Tata',
                model: 'Nano',
                owner: 'Valeria',
            },
            {
                color: 'brown',
                make: 'Holden',
                model: 'Barina',
                owner: 'Shotaro',
            },
        ];

        for (let i = 0; i < cars.length; i++) {
            cars[i].docType = 'car';
            await ctx.stub.putState('CAR' + i, Buffer.from(JSON.stringify(cars[i])));
            console.info('Added <--> ', cars[i]);
        }
        console.info('============= END : Initialize Ledger ===========');
    }

    async queryCar(ctx, carNumber) {
        const carAsBytes = await ctx.stub.getState(carNumber); // get the car from chaincode state
        if (!carAsBytes || carAsBytes.length === 0) {
            throw new Error(`${carNumber} does not exist`);
        }
        console.log(carAsBytes.toString());
        return carAsBytes.toString();
    }

    async createCar(ctx, carNumber, make, model, color, owner) {
        console.info('============= START : Create Car ===========');

        const car = {
            color,
            docType: 'car',
            make,
            model,
            owner,
        };

        await ctx.stub.putState(carNumber, Buffer.from(JSON.stringify(car)));
        console.info('============= END : Create Car ===========');
    }

    async queryAllCars(ctx) {
        const startKey = 'CAR0';
        const endKey = 'CAR999';
        const allResults = [];
        for await (const {key, value} of ctx.stub.getStateByRange(startKey, endKey)) {
            const strValue = Buffer.from(value).toString('utf8');
            let record;
            try {
                record = JSON.parse(strValue);
            } catch (err) {
                console.log(err);
                record = strValue;
            }
            allResults.push({ Key: key, Record: record });
        }
        console.info(allResults);
        return JSON.stringify(allResults);
    }

    async changeCarOwner(ctx, carNumber, newOwner) {
        console.info('============= START : changeCarOwner ===========');

        const carAsBytes = await ctx.stub.getState(carNumber); // get the car from chaincode state
        if (!carAsBytes || carAsBytes.length === 0) {
            throw new Error(`${carNumber} does not exist`);
        }
        const car = JSON.parse(carAsBytes.toString());
        car.owner = newOwner;

        await ctx.stub.putState(carNumber, Buffer.from(JSON.stringify(car)));
        console.info('============= END : changeCarOwner ===========');
    }

}

module.exports = FabCar;
```











