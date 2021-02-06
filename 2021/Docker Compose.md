Docker Compose可以通过一个yaml文件自动对拉取来的docker镜像进行配置

使用Docker Compose来创建容器的流程主要是下面三步：

- 创建编辑Dockerfile
- 使用docker-compose.yml来定义配置服务
- 执行`docker-compose up`启动服务

这篇博客主要讲一下`docker-compose.yml`的编写



## 配置文件选项

### depends_on

服务的依赖，只有其依赖的服务启动完毕后才会启动；

例如配置图床的镜像首先要配置好数据库，这时就会先启动数据库的服务之后才启动图床；



### environment

需要设置的环境变量值

一般docker镜像的服务都可以通过设置环境变量来自动配置；



### volume

文件系统的映射，这里的映射方式和使用`docker`命令行时`-v`参数后面的格式一样；

如果没有设置完整的录几个，默认映射出来的位置  在` /var/lib/docker/volumes/`下



## 示例

例如配置了chevereto图床，只需要拉取mariadb、chevereto的镜像

之后用一个yaml文件写配置就可以

```yaml
version: '3'

services:
  db:
    image: mariadb
    volumes:
      - database:/var/lib/mysql:rw
    restart: always
    networks:
      - private
    environment:
      MYSQL_ROOT_PASSWORD: chevereto_root
      MYSQL_DATABASE: chevereto
      MYSQL_USER: chevereto
      MYSQL_PASSWORD: chevereto

  chevereto:
    depends_on:
      - db
    image: nmtan/chevereto
    restart: always
    networks:
      - private
    environment:
      CHEVERETO_DB_HOST: db
      CHEVERETO_DB_USERNAME: chevereto
      CHEVERETO_DB_PASSWORD: chevereto
      CHEVERETO_DB_NAME: chevereto
      CHEVERETO_DB_PREFIX: chv_
    volumes:
      - chevereto_images:/var/www/html/images:rw
    ports:
      - 8888:80

networks:
  private:
volumes:
  database:
  chevereto_images:
```



## 参考链接

[docker搭建图床 chevereto 非常方便](https://www.cnblogs.com/changeCode/p/11592131.html)