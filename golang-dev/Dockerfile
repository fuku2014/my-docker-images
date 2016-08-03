FROM centos:centos6

ARG proxy
ARG user
ARG pw

# set yum proxy
RUN echo "proxy=${proxy}" >> /etc/yum.conf

# install sudo and edit sudoers
RUN yum -y install sudo wget
RUN sed -i -e "s/Defaults    requiretty.*/ #Defaults    requiretty/g" /etc/sudoers

# set proxy
ENV http_proxy=${proxy}
ENV https_proxy=${proxy}
ENV HTTP_PROXY=${proxy}
ENV HTTPS_PROXY=${proxy}

# add user
RUN useradd ${user}
RUN echo ${user}:${pw} | chpasswd
RUN echo "${user} ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
USER ${user}

# create work dir
RUN mkdir /home/${user}/work
WORKDIR /home/${user}/work

# rpm repository
RUN wget http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.3-1.el6.rf.x86_64.rpm
RUN wget http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm

RUN sudo rpm -ivh rpmforge-release-0.5.3-1.el6.rf.x86_64.rpm
RUN sudo rpm -Uvh epel-release-6-8.noarch.rpm

# yum update
RUN sudo yum -y update

# install package
RUN sudo yum -y install \
        tar \
        gcc-c++ \
        g++ \
        gcc \
        libc6-dev \
        make \
        curl \
        bzip2 \
        vim \
        tig \
        tmux \
        curl-devel \
        expat-devel \
        gettext-devel \
        openssl-devel \
        zlib-devel \
        perl-ExtUtils-MakeMaker

# install git 2.9.0
RUN wget https://www.kernel.org/pub/software/scm/git/git-2.9.0.tar.gz
RUN tar xzvf git-2.9.0.tar.gz && cd git-2.9.0 && sudo make prefix=/usr/local all && sudo make prefix=/usr/local install

# setup dotfiles
RUN git clone https://github.com/fuku2014/dotfiles.git
RUN cd dotfiles & git submodule init & git submodule update & sh dotfilesLink.sh


ENV GOLANG_VERSION 1.6.2
ENV GOLANG_DOWNLOAD_URL https://golang.org/dl/go$GOLANG_VERSION.linux-amd64.tar.gz
ENV GOLANG_DOWNLOAD_SHA256 e40c36ae71756198478624ed1bb4ce17597b3c19d243f3f0899bb5740d56212a

RUN curl -fsSL "$GOLANG_DOWNLOAD_URL" -o golang.tar.gz \
    && echo "$GOLANG_DOWNLOAD_SHA256  golang.tar.gz" | sha256sum -c - \
    && tar -C /usr/local -xzf golang.tar.gz \
    && rm golang.tar.gz

ENV GOPATH /go
ENV PATH $GOPATH/bin:/usr/local/go/bin:$PATH

RUN mkdir -p "$GOPATH/src" "$GOPATH/bin" && chmod -R 777 "$GOPATH"

# glide install
ENV GLIDE_VERSION 0.10.2
ENV GLIDE_DOWNLOAD_URL https://github.com/Masterminds/glide/releases/download/$GLIDE_VERSION/glide-$GLIDE_VERSION-linux-amd64.tar.gz
RUN curl -fsSL "$GLIDE_DOWNLOAD_URL" -o glide.tar.gz \
    && mkdir /usr/local/glide \
    && tar -C /usr/local/glide -xzf glide.tar.gz \
    && ln -s /usr/local/glide/linux-amd64/glide /usr/local/bin/ \
&& rm glide.tar.gz

EXPOSE 8080

CMD /bin/bash
