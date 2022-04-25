FROM continuumio/miniconda3

ARG user=codatalab
ARG password=test
ARG homedir=/home
ARG branch=unapei-toolbox-server

RUN apt-get update -qq \
    && apt-get install -yqq --no-install-recommends \
        git \
        openssh-server \
        net-tools iputils-ping nano \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/* \
    && ln -s /usr/bin/python3 /usr/bin/python \
    && if [[ $(conda env list | grep 'capfalcnlp' | wc -l) = 0 ]]; then conda create -n capfalcnlp python=3.7 gunicorn -y; fi \
    \
    && if [ $(grep ${user} /etc/passwd | wc -l) = 0 ]; then \
         useradd --create-home --shell /bin/bash ${user} \
         && mkdir -p ${homedir}/${user}/.ssh \
         && chmod 0700 ${homedir}/${user}/.ssh \
         && chown -R ${user}:${user} ${homedir}/${user}; \
       fi \
    && echo "${user}:${password}" | chpasswd \
    \
    && mkdir /run/sshd

RUN rm -rf /run/nologin

USER ${user}
WORKDIR ${homedir}/${user}


# FIXED bashrc skipping if ran non interactively causing conda ativate to fail 
# TMP fix processing.py waiting for the issue to be solved by this MR : 
RUN git clone --quiet --depth 1 https://github.com/cleyrop/capfalcnlp.git -b ${branch} \
    && rm -rf capfalcnlp/.git* \
    && conda init bash \
    && sed -i '/# If not running interactively/,+5d' ~/.bashrc \
    && echo 'conda activate capfalcnlp' >> ~/.bashrc \
    && echo 'cd capfalcnlp' >> ~/.bashrc \
    && echo 'if [ -f ~/.bashrc ]; then' >> ~/.bash_profile \
    && echo '  . ~/.bashrc' >> ~/.bash_profile \
    && echo 'fi' >> ~/.bash_profile \
    && . ~/.bashrc \
    && pip install -r requirements.txt \
    && pip install -e .

RUN . ~/.bashrc \
    && python -c "from capfalcnlp.helpers import load_model; load_model()"

ENV FLASK_APP=toolbox
ENV FLASK_ENV=production


EXPOSE 2222
COPY --chown=${user} resources/gunicorn.conf.py /home/${user}
COPY resources/entrypoint /entrypoint
ENTRYPOINT ["/entrypoint"]
CMD tail -f /dev/null
