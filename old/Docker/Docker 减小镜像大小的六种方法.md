# Docker 减小镜像大小的方法

##### 1.使用Alpine Linux

```java
1.Alpine Linux是一个基于BusyBox和Musl Libc的Linux发行版，其最大的优势就是小。一个纯的基础Alpine Docker镜像在压缩后仅有2.67MB。
2.但是在Docker Hub中，大部分镜像是没有Alpine版本的，比如Mysql和PHP-Apache，如果我们需要基于这些环境开发，就不得不自己编写Alpine版本，或者找一些第三方镜像。
3.Alpine的另一个缺点是，其使用了Musl Libc作为传统的glibc的替代，编译软件的时候可能会遇到一些不可预知的问题，这一点会导致我们耗费不少不必要的时间。
```

##### 2.通过Docker多阶段构建将多个层压缩为一个

```java
1.合并RUN命令。
2.分阶段提交。
```

