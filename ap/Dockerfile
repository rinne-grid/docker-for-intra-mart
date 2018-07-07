FROM centos:7.5.1804

RUN chsh -s /usr/bin/bash root


# --build-arg docker-composeのargsを受け取る
ARG arg_resin_guest_port
ARG arg_resin_server_name
ARG arg_jdbc_file_name
ARG arg_jdbc_download_url
ARG arg_im_java_home
ARG arg_resin_home

# 環境変数にHTTPプロキシ情報を設定する
# 定義: ./ap/.env HTTP_PROXY
ENV http_proxy=$arg_http_proxy
ENV HTTP_PROXY=$arg_http_proxy
ENV https_proxy=$arg_https_proxy
ENV HTTPS_PROXY=$arg_https_proxy

# resinサーバー名称を設定
ENV RESIN_SERVER_NAME="${arg_resin_server_name}"
# RESIN_HOMEを設定
ENV RESIN_HOME="${arg_resin_home}"
RUN export RESIN_HOME=${RESIN_HOME}


# 設定ファイル: ./ap/.env IM_JAVA_HOME
ENV JAVA_HOME=${arg_im_java_home}
# ENV JAVA_HOME="/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.171-8.b10.el7_5.x86_64"
ENV CLASS_PATH="$JAVA_HOME/jre/lib:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar"

#---------------------------------------------------------------------
# パッケージインストール
#---------------------------------------------------------------------
RUN rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
RUN yum update -y && \
    yum install -y epel-release && \
    yum install -y java-1.8.0-openjdk java-1.8.0-openjdk-devel gcc glibc-devel make openssl openssl-devel which tzdata wget && \
    ln -sf /usr/share/zoneinfo/Asia/Tokyo /etc/localtime

#---------------------------------------------------------------------
# RESINの設定
# * ホストの$RESIN_SERVER_NAMEをコンテナの$RESIN_HOMEに追加
# * $RESIN_HOME/libにjdbcドライバを追加 
#---------------------------------------------------------------------
ADD "./${RESIN_SERVER_NAME}/" "$RESIN_HOME"

WORKDIR $RESIN_HOME/lib
RUN wget "${arg_jdbc_download_url}/${arg_jdbc_file_name}"

WORKDIR $RESIN_HOME

#---------------------------------------------------------------------
# RESINのインストール
#---------------------------------------------------------------------
RUN chmod 755 "$RESIN_HOME/configure"
RUN $RESIN_HOME/configure --prefix=$RESIN_HOME --enable-64bit && \
     make && \
     make install

# コンテナのresinポートを開放
EXPOSE ${arg_resin_guest_port}
