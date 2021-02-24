# redis

## redis安装

1. yum install wget

2. cd ~

3. mkdir soft

4. cd soft

5. wget http://download.redis.io/release/redis-5.0.5.tar.gz

6. tar xf redis....tar.gz

7. cd redis-src

8. 看README.md

9. make

   ...yum install gcc

   ...make distclean

10. make

11. cd src  ....生成可执行程序

12. cd ..

13. make install PREFIX=/opt/mashibing/redis

14. vi /etc/profile

    .... export REDIS_HOME=/opt/mashibing/redis

    .... export  PATH=$PATH.$REDIS_HOME/bin

15. cd utils

16. ./install sever.sh

    一个物理机中可以有多个redis实例（进程），通过port区分

    可执行程序就一份在目录，但在内存中未来的多个实例需要各自目的配置文件，持久化目录等资源

    service redis_6379 start/stop/status  >linux  /etc/init.d/...

    脚本还会帮你启动

    ps -fe | grep redis


## 布隆过滤器加载

解决缓存穿透

- 在缓存中设置空对象，在查缓存后，查看是否是空对象，如果是空对象就不要往数据库查了

- 布隆过滤器；底层通过哈希算法实现的（具体不知道），能通过它得到缓存中是否没有该值，如果没，则一定没，如果有则也不一定有

  - 访问redis.io

  - modules 

  - 访问RedisBloom的github   https://github.com/RedisBloom

  - linux中wget *.zip

  - unzip *.zip

  - make

  - cp bloom.so /opt/*

  - redis-server --loadmodule /opt/*/redisbloom.so

  - redis-cli

  - bf.add ooxx abc

    bf.exists abc

    bf.exists sdfsdf

  - cf add # 布谷鸟过滤器

Redis过期判定原理：

- 被动访问判定

  当数据过期了，用户访问这条数据，就会将其清除

- 周期轮询判定：在稍微牺牲下内存，但保存了redis性能为王

  每隔一段时间进行轮询查看数据是否过期进行清除

 