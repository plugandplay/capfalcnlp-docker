FROM python:3.7-bullseye

RUN apt-get update -qq \
    && apt-get install -yqq --no-install-recommends \
        git \
        openssh-server \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/* \
    && service ssh start \
    && useradd -m codatalab \
    && echo 'codatalab:test' | chpasswd 

COPY resources/ssh_config /etc/ssh/ssh_config
COPY resources/sshd_config /etc/ssh/sshd_config
COPY resources/entrypoint /entrypoint

#USER codatalab
WORKDIR /home/codatalab

RUN git clone --quiet --depth 1 https://github.com/louismartin/capfalcnlp.git \
    && rm -rf capfalcnlp/.git* \
    && cd capfalcnlp \
    && pip install -r requirements.txt \
    && pip install -e . \
    && python -m spacy download fr_core_news_md \
    && chown -R codatalab /home/codatalab

EXPOSE 22

ENTRYPOINT ["/entrypoint"]
 
CMD tail -f /dev/null
