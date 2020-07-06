FROM ty60/dev-docker:ubuntu-16.04


RUN mkdir /home/admin/tools

USER root

RUN apt-get update -y \
    && apt-get install -y python3 python3-pip python3-dev git libssl-dev libffi-dev build-essential

# gdb attach
RUN echo "kernel.yama.ptrace_scope = 0" > /etc/sysctl.d/10-ptrace.conf

# ROPGadget
RUN git clone https://github.com/JonathanSalwan/ROPgadget /home/admin/tools/ROPgadget \
    && cd /home/admin/tools/ROPgadget \
    && python3 setup.py install

# Install one_gadget
RUN apt-get install -y ruby \
    && gem install one_gadget

# rp++
RUN wget -q https://github.com/downloads/0vercl0k/rp/rp-lin-x64 -O /usr/local/bin/rp++ \
    && chmod 755 /usr/local/bin/rp++

# Downgrade gdb for gef
RUN apt-get install -y --allow-downgrades gdb=7.11-0ubuntu1

USER admin

# pwntools
RUN python3 -m pip install --upgrade pwntools


# gef
# Need LC_TYPE for gef (https://github.com/hugsy/gef/issues/195)
RUN curl -L -s -o /home/admin/tools/.gdbinit-gef.py https://github.com/hugsy/gef/raw/master/gef.py \
    && echo "\ndefine gef\n source /home/admin/tools/.gdbinit-gef.py\nend\n" > /home/admin/.gdbinit \
    && echo "export LC_CTYPE=C.UTF-8" >> /home/admin/dotfiles/.bashrc
