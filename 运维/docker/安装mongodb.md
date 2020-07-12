# 安装MongoDB

## 下载镜像

```shell
docker search mongo
docker pull mongo:latest
docker images
```

## 启动

```shell
# -i, --interactive=false 打开STDIN，用于控制台交互
# -t, --tty=false 分配tty设备，该可以支持终端登录，默认为false
# -d, --detach=false 指定容器运行于前台还是后台，默认为false
# -v /my/own/datadir:/data/db
# -p, --publish=[] 指定容器暴露的端口
docker run -itd --name mongo -p 27017:27017 mongo --auth
```

## 进入容器

```shell
docker exec -it mongo bash
docker exec -it 305ebd823667 bash
db.createUser({ user:'admin',pwd:'wq182654',roles:[ { role:'userAdminAnyDatabase', db: 'admin'}]});
db.createUser({ user: 'goahead', pwd: 'wq182654', roles: [ { role: "readWrite", db: "wanqian" } ] });
db.createUser({user:"goahead",pwd:"wq182654",roles:[{role:"dbOwner",db:"wangqian"}]})
```



