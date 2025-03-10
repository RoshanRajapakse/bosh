#bosh/integration:main
#
FROM ubuntu:latest

ENV DEBIAN_FRONTEND "noninteractive"
ENV RUBY_VERSION "3.1"
ENV RUBY_INSTALL_VERSION "0.8.5"
ENV GEM_HOME "/usr/local/bundle"
ENV POSTGRES_MAJOR_VERSION "10"

RUN apt-get update -y \
    && apt-get -y install --no-install-recommends \
        software-properties-common \
    && apt-get install -y --no-install-recommends \
        locales \
        awscli \
        build-essential \
        curl \
        coreutils \
        dnsutils \
        git \
        gpg-agent \
        jq \
        libcurl4-openssl-dev \
        libpq-dev \
        libreadline-dev \
        libsqlite3-dev \
        libssl-dev \
        netcat-openbsd \
        rsyslog \
        sqlite3 \
        sudo \
        tar \
        wget \
        xvfb \
        vim \
        openssh-client \
        clang \
        clang-14 \
        lib32gcc-s1 \
        lib32stdc++6 \
        libc6-i386 \
        libclang-common-14-dev \
        libclang-cpp14 \
        libclang1-14 \
        libgc1 \
        libllvm14 \
        libobjc-11-dev \
        libobjc4 \
        llvm-14-linker-tools

ENV CXX=/usr/bin/clang++ CC=/usr/bin/clang
RUN echo 'gem: --no-document' > /etc/gemrc \
    && curl -sL -O https://github.com/postmodern/ruby-install/archive/refs/tags/v${RUBY_INSTALL_VERSION}.tar.gz \
    && tar -xzf v${RUBY_INSTALL_VERSION}.tar.gz \
    && cd ruby-install-${RUBY_INSTALL_VERSION}/ \
    && make -s install \
    && ruby-install --jobs=2 --cleanup --system ruby ${RUBY_VERSION} -- --disable-install-rdoc -- CC=clang

RUN gem update --system \
    && bundle config --global path "${GEM_HOME}" \
    && bundle config --global bin "${GEM_HOME}/bin"

# Database clients
RUN echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/trusted.gpg.d/psql_org.gpg] https://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" \
    | tee -a /etc/apt/sources.list.d/pgdg.list \
    && curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc \
    | gpg --dearmor -o /etc/apt/trusted.gpg.d/psql_org.gpg \
    && apt-get update -y \
    && apt-get install -y --no-install-recommends \
    mysql-client \
    libmysqlclient-dev \
    postgresql-client-${POSTGRES_MAJOR_VERSION}

COPY --from=golang:1 /usr/local/go /usr/local/go
ENV GOROOT=/usr/local/go PATH=/usr/local/go/bin:$PATH

# Java to start UAA
ENV LANG en_US.UTF-8
ENV LC_ALL en_US.UTF-8
RUN locale-gen en_US.UTF-8

ADD install-java.sh /tmp/install-java.sh
RUN chmod a+x /tmp/install-java.sh
RUN cd /tmp && ./install-java.sh && rm install-java.sh
ENV JAVA_HOME /usr/lib/jvm/zulu8.23.0.3-jdk8.0.144-linux_x64
ENV PATH $JAVA_HOME/bin:$PATH
