# FROM    centos:6.7
# MAINTAINER      Fisher "fisher@sudops.com"

# RUN     /bin/echo 'root:123456' |chpasswd
# RUN     useradd runoob
# RUN     /bin/echo 'runoob:123456' |chpasswd
# RUN     /bin/echo -e "LANG=\"en_US.UTF-8\"" >/etc/default/local
# EXPOSE  22
# EXPOSE  80
# CMD     /usr/sbin/sshd -D

FROM nginx
RUN echo '这是一个本地构建的nginx镜像' > /usr/share/nginx/html/index.html