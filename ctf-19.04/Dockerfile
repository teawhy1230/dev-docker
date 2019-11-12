FROM yasmu/dev-docker:ubuntu-19.04


RUN mkdir /home/admin/tools

USER root

# ROPGadget
RUN git clone https://github.com/JonathanSalwan/ROPgadget /home/admin/tools/ROPgadget \
    && cd /home/admin/tools/ROPgadget \
    && python3 setup.py install

# Install one_gadget
RUN apt-get install -yq ruby \
    && gem install one_gadget

# rp++
RUN wget -q https://github.com/downloads/0vercl0k/rp/rp-lin-x64 -O /usr/local/bin/rp++ \
    && chmod 755 /usr/local/bin/rp++


USER admin

# pwntools
RUN pip3 install pwntools

# gef
# Need LC_TYPE for gef (https://github.com/hugsy/gef/issues/195)
RUN curl -L -s -o /home/admin/tools/.gdbinit-gef.py https://github.com/hugsy/gef/raw/master/gef.py \
    && echo "\ndefine gef\n source /home/admin/tools/.gdbinit-gef.py\nend\n" > /home/admin/.gdbinit \
    && echo "export LC_CTYPE=C.UTF-8" >> /home/admin/dotfiles/.bashrc
