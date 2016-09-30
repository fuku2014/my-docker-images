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

# yum update
RUN sudo yum -y update

RUN sudo yum -y install http://ftp.riken.jp/Linux/repoforge/redhat/el6/en/x86_64/rpmforge/RPMS/rpmforge-release-0.5.3-1.el6.rf.x86_64.rpm
RUN sudo yum -y install http://mirror.fairway.ne.jp/dag/redhat/el6/en/x86_64/rpmforge/RPMS/rpmforge-release-0.5.3-1.el6.rf.x86_64.rpm
# install package
RUN sudo yum -y install \
        tar \
	tig \
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
        perl-ExtUtils-MakeMaker \
	openssh-server

# install git 2.9.0
RUN wget https://www.kernel.org/pub/software/scm/git/git-2.9.0.tar.gz
RUN tar xzvf git-2.9.0.tar.gz && cd git-2.9.0 && sudo make prefix=/usr/local all && sudo make prefix=/usr/local install

# setup dotfiles
RUN git clone https://github.com/fuku2014/dotfiles.git
RUN cd dotfiles && git submodule init && git submodule update && sh dotfilesLink.sh


ENV GOLANG_VERSION 1.7.1
ENV GOLANG_DOWNLOAD_URL https://golang.org/dl/go$GOLANG_VERSION.linux-amd64.tar.gz
ENV GOLANG_DOWNLOAD_SHA256 e40c36ae71756198478624ed1bb4ce17597b3c19d243f3f0899bb5740d56212a

RUN curl -fsSL "$GOLANG_DOWNLOAD_URL" -o golang.tar.gz \
    && echo "$GOLANG_DOWNLOAD_SHA256  golang.tar.gz" | sha256sum -c - \
    && sudo tar -C /usr/local -xzf golang.tar.gz \
    && rm golang.tar.gz

ENV GOPATH /go
ENV PATH $GOPATH/bin:/usr/local/go/bin:$PATH

RUN sudo mkdir -p "$GOPATH/src" "$GOPATH/bin" && sudo chmod -R 777 "$GOPATH"

# glide install
ENV GLIDE_VERSION 0.10.2
ENV GLIDE_DOWNLOAD_URL https://github.com/Masterminds/glide/releases/download/$GLIDE_VERSION/glide-$GLIDE_VERSION-linux-amd64.tar.gz
RUN curl -fsSL "$GLIDE_DOWNLOAD_URL" -o glide.tar.gz \
    && sudo mkdir /usr/local/glide \
    && sudo tar -C /usr/local/glide -xzf glide.tar.gz \
    && sudo ln -s /usr/local/glide/linux-amd64/glide /usr/local/bin/ \
&& rm glide.tar.gz

# ssh
RUN sudo sed -ri 's/^#PermitRootLogin yes/PermitRootLogin yes/' /etc/ssh/sshd_config
RUN sudo /etc/init.d/sshd start

RUN sudo echo 'root:root' | sudo chpasswd

EXPOSE 8080

CMD ["sudo", "/usr/sbin/sshd", "-D"]
