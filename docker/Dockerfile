FROM continuumio/miniconda3

RUN apt-get update -qq \
    && apt-get install -yqq --no-install-recommends \
        git \
        openssh-server \
        net-tools iputils-ping nano acl \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/* \
    && ln -s /usr/bin/python3 /usr/bin/python \
    && conda create -n capfalcnlp python=3.7 -y \
    \
    && mkdir -p /usr/local/home/codatalab/.ssh \
    && useradd --home /usr/local/home/codatalab --no-create-home --shell /bin/bash codatalab --no-log-init \
    && chmod 0700 /usr/local/home/codatalab/.ssh \
    && chown -R codatalab:codatalab /usr/local/home/codatalab \
    && echo 'codatalab:test' | chpasswd \
    \
    && mkdir /opt/ssh \
    && /usr/bin/ssh-keygen -A \
    && cp /etc/ssh/ssh_host_* /opt/ssh/ \
    && cp /etc/ssh/sshd_config /opt/ssh/ \
    && sed -i -E 's:#?HostKey /etc:HostKey /opt:' /opt/ssh/sshd_config \
    && sed -i -E 's/#?PasswordAuthentication yes/PasswordAuthentication yes/' /opt/ssh/sshd_config \
    && echo 'Port 2222' >> /opt/ssh/sshd_config \
    && echo 'LogLevel DEBUG3' >> /opt/ssh/sshd_config \
    && chmod 700 /opt/ssh \
    && chmod 600 /opt/ssh/* \
    && chmod 644 /opt/ssh/sshd_config \
    && chown -R codatalab:codatalab /opt/ssh \
    && mkdir /run/sshd

# COPY --chown=codatalab resources/ssh_config /opt/ssh/ssh_config
# COPY --chown=codatalab resources/sshd_config /opt/ssh/sshd_config

RUN rm -rf /run/nologin

USER codatalab
WORKDIR /usr/local/home/codatalab


# FIXED bashrc skipping if ran non interactively causing conda ativate to fail 
# TMP fix processing.py waiting for the issue to be solved by this MR : 
RUN git clone --quiet --depth 1 https://github.com/cleyrop/capfalcnlp.git -b unapei-toolbox \
    && rm -rf capfalcnlp/.git* \
    && conda init bash \
    && sed -i '/# If not running interactively/,+5d' ~/.bashrc \
    && echo 'conda activate capfalcnlp' >> ~/.bashrc \
    && echo 'cd capfalcnlp' >> ~/.bashrc \
    && . ~/.bashrc \
    && pip install -r requirements.txt \
    && pip install -e . \
    && python -m spacy download fr_core_news_md \
    && chown -R codatalab /usr/local/home/codatalab

RUN . ~/.bashrc \ 
    && python cli.py --input-file example_text.txt

EXPOSE 2222
COPY resources/entrypoint /entrypoint
ENTRYPOINT ["/entrypoint"]
CMD tail -f /dev/null
