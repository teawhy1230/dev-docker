FROM ubuntu:20.10

ENV DEBIAN_FRONTEND noninteractive

## setup a user
RUN useradd -m -s /bin/bash admin \
    && usermod -aG sudo admin \
    && /bin/echo -e "admin\nadmin"|passwd admin \
    && chmod 4750 /home/admin \
    && chown -R admin: /home/admin \
    && mkdir -p /etc/sudoers.d \
    && echo "admin ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/admin \
    # && echo "kernel.yama.ptrace_scope = 0" > /etc/sysctl.d/10-ptrace.conf, \
    && sysctl -p

RUN apt-get update && apt-get upgrade -y
RUN apt-get install -yq \
    curl \
    wget \
    git \
    sudo \
    python3 \
    python3-pip \
    gdb \
    gdb-multiarch \
    gdbserver \
    apt-utils \
    tmux \
    ninja-build \
    gettext \
    libtool \
    libtool-bin \
    autoconf \
    automake \
    cmake \
    g++ \
    pkg-config \
    unzip \
    software-properties-common \
    zlib1g-dev \
    libz3-4


# Install clang
RUN cd /opt \
    && wget https://releases.llvm.org/9.0.0/clang+llvm-9.0.0-x86_64-pc-linux-gnu.tar.xz \
    && tar xf clang+llvm-9.0.0-x86_64-pc-linux-gnu.tar.xz

ENV PATH $PATH:/opt/clang+llvm-9.0.0-x86_64-pc-linux-gnu/bin

# Install neovim
RUN wget -q https://github.com/neovim/neovim/archive/v0.4.4.zip \
    && unzip v0.4.4.zip \
    && cd neovim-0.4.4 \
    && make CMAKE_BUILD_TYPE=Release \
    && make install \
    && cd ../ \
    && rm v0.4.4.zip \
    && rm -rf neovim-0.4.4

RUN pip3 install --upgrade pynvim

# Install node
# https://github.com/nodesource/distributions#installation-instructions
RUN curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash - \
        && apt-get install -y nodejs

# Install yarn
RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add - \
    && echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list \
    && apt-get update && apt-get install -y yarn

RUN npm install -g neovim

# Install dotnet
# https://dotnet.microsoft.com/learn/dotnet/hello-world-tutorial/install
RUN wget https://packages.microsoft.com/config/ubuntu/20.10/packages-microsoft-prod.deb -O packages-microsoft-prod.deb \
    && dpkg -i packages-microsoft-prod.deb \
    && add-apt-repository universe \
    && apt-get update \
    && apt-get install -y apt-transport-https \
    && apt-get update \
    && apt-get install -y dotnet-sdk-5.0 \
    && add-apt-repository universe \
    && apt-get update \
    && apt-get install apt-transport-https \
    && apt-get update \
    && apt-get install -y dotnet-runtime-5.0

# Build mpls
RUN cd /opt \
    && git clone --depth=1 https://github.com/Microsoft/python-language-server.git \
    && cd python-language-server/src/LanguageServer/Impl \
    && dotnet build

# Build and install ccls
RUN cd /opt \
    && ln -s /usr/lib/x86_64-linux-gnu/libz3.so.4 /usr/lib/x86_64-linux-gnu/libz3.so.4.8 \
    && ln -s /usr/lib/x86_64-linux-gnu/libz3.so.4 /usr/local/lib/libz3.so \
    && apt-get install -y libtinfo-dev \
    && git clone --depth=1 --recursive https://github.com/MaskRay/ccls \
    && cd ccls \
    && cmake -H. -BRelease -DCMAKE_BUILD_TYPE=Release -DCMAKE_PREFIX_PATH=/opt/clang+llvm-9.0.0-x86_64-pc-linux-gnu.tar.xz \
    && cmake --build Release \
    && ln -s /opt/ccls/Release/ccls /usr/local/bin/ccls

# Install user level stuff
USER admin
## dotfiles
## Need LC_TYPE for gef (https://github.com/hugsy/gef/issues/195)
RUN git clone https://github.com/teawhy1230/dotfiles.git /home/admin/dotfiles \
    && mkdir -p /home/admin/.config/ \
    && ln -s /home/admin/dotfiles/nvim /home/admin/.config/nvim \
    && ln -s /home/admin/dotfiles/.tmux.conf /home/admin/.tmux.conf \
    && rm /home/admin/.bashrc \
    && echo "alias tmux='tmux -u -2'" >> /home/admin/dotfiles/.bashrc \
    # && echo "export LC_CTYPE=C.UTF-8" >> /home/admin/dotfiles/.bashrc \
    && ln -s /home/admin/dotfiles/.bashrc /home/admin/.bashrc \
    && cd /home/admin/dotfiles \
    && echo "source /etc/bash_completion.d/git-prompt" >> /home/admin/.bashrc \
    && chown -R admin:admin /home/admin/dotfiles

# Install dein
RUN curl https://raw.githubusercontent.com/Shougo/dein.vim/master/bin/installer.sh > /home/admin/.config/nvim/installer.sh \
    && sh /home/admin/.config/nvim/installer.sh /home/admin/.config/nvim/dein \
    && rm /home/admin/.config/nvim/installer.sh

# update dein from shell
# https://github.com/Shougo/dein.vim/blob/540345405ab9184459bf7a20519e0a5af2da5330/doc/dein.txt#L1182
RUN nvim -c "try | call dein#update() | finally | qall! | endtry"
# install extensios from shell
# https://github.com/neoclide/coc.nvim/wiki/Using-coc-extensions
RUN nvim -c "CocInstall -sync coc-tsserver | qall!"


WORKDIR /home/admin

CMD ["/bin/bash", "-i"]
