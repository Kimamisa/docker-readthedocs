FROM debian:stretch-slim
LABEL maintainer='\
Florian "floross" Rossiaud <o>, \
Simon "svvac" Wachter <himself@swordofpain.net>, \
Yevgeny Popovych <jmennius@gmail.com>'

ENV DEBIAN_FRONTEND=noninteractive \
    APPDIR=/app \
    DJANGO_SETTINGS_MODULE=readthedocs.settings.container \
    LANG=en_US.UTF-8 \
    LC_ALL=en_US.UTF-8 \
    VIRTUAL_ENV=/venv \
    PATH=/venv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin \
    PYTHON=/venv/bin/python \
    PIP="/venv/bin/pip --disable-pip-version-check" \
    RTD_REF=3.5.2


# Set locale to UTF-8
RUN apt-get -qq update && \
    echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && \
    apt-get install -y locales && \
    rm -rf /var/lib/apt/lists/*

# Install basic dependencies
RUN apt-get -qq update && \
    mkdir -p /usr/share/man/man1 && \
    apt-get install -qq --no-install-recommends \
        python-setuptools build-essential python-dev python-pip git libevent-dev \
        libxml2-dev libssl-dev libxslt1-dev gettext texlive texlive-latex-extra zlib1g-dev libffi-dev \
        wget nginx libpq-dev libbz2-dev python3-bz2file \
        openjdk-8-jre unzip libcurl4-openssl-dev && \
    rm -rf /var/lib/apt/lists/*

RUN cd /usr/src && \
    wget https://www.python.org/ftp/python/3.6.3/Python-3.6.3.tar.xz && \
    tar xJf Python-3.6.3.tar.xz && \
    cd Python-3.6.3 && \
    ./configure --enable-optimizations && \
    make install

RUN cd /usr/src && \
    wget https://www.python.org/ftp/python/3.7.3/Python-3.7.3.tar.xz && \
    tar xJf Python-3.7.3.tar.xz && \
    cd Python-3.7.3 && \
    ./configure --enable-optimizations && \
    make altinstall

# Install test dependencies
RUN pip install wheel virtualenv && \
    pip3.7 install wheel virtualenv

# Install test dependencies
RUN wget "https://bootstrap.pypa.io/get-pip.py" -O "get-pip.py" && \
    python3 get-pip.py && \
    pip3 install wheel virtualenv supervisor

# Setting up virtualenv
RUN virtualenv --python=python3 /venv && \
    cp /usr/lib/python3.5/lib-dynload/_bz2.cpython-35m-x86_64-linux-gnu.so /venv/lib/python3.6/lib-dynload/_bz2.cpython-36m-x86_64-linux-gnu.so && \
    cp /usr/lib/python3.5/lib-dynload/_sqlite3.cpython-35m-x86_64-linux-gnu.so /venv/lib/python3.6/lib-dynload/_sqlite3.cpython-36m-x86_64-linux-gnu.so

# Add user py
RUN adduser --gecos 'py' --disabled-password py

RUN mkdir -p $APPDIR
WORKDIR $APPDIR

## Extract readthedocs
RUN wget -q --no-check-certificate -O - \
        https://github.com/rtfd/readthedocs.org/archive/${RTD_REF}.tar.gz | \
        tar -xzf - --strip-components 1 && \
    $PIP install -U -r $APPDIR/requirements/deploy.txt && \
    $PIP install git+https://github.com/rtfd/readthedocs-sphinx-ext.git

## Copy configuration, scripts and patch RTD
COPY . /
#RUN find /patches -type f -exec patch -p1 -i {} \; && rm -rf /patches

## Copy special configuration for read the docs
RUN ln -s $APPDIR/manage.py $APPDIR/readthedocs/manage.py && \
    ln -s /etc/nginx/sites-available/readthedocs /etc/nginx/sites-enabled/readthedocs && \
    rm /etc/nginx/sites-enabled/default && \
    chmod +x $APPDIR/bin/* && \
    chown -R py .

# Download plantUML jar
RUN wget -nc https://netix.dl.sourceforge.net/project/plantuml/plantuml.jar -O /plantuml.jar

# Install latest git
RUN wget https://github.com/git/git/archive/v2.21.0.zip -O git.zip && \
    unzip git.zip && \
    cd git-2.21.0 && \
    make prefix=/usr/local all && \
    make prefix=/usr/local install

# Docker config
EXPOSE 80
VOLUME [ "/app/user_builds" ]

CMD [ "supervisord", "-c", "/etc/supervisord.conf", "-n" ]
