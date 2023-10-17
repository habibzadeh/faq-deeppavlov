FROM python:3.8-slim-bullseye

ARG COMMIT=master

EXPOSE 5000
RUN mkdir /base
RUN cd /base
WORKDIR /base
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        build-essential \
        git \
        locales && \
    printf '%s\n%s\n%s\n' 'en_US.UTF-8 UTF-8' 'ru_RU.UTF-8 UTF-8' 'fa_IR.UTF-8 UTF-8'>> /etc/locale.gen && \
    locale-gen && \
    apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false && \
	rm -rf /var/lib/apt/lists/* && \
	pip install virtualenv==16.7.5 && \
	virtualenv venv && \
	mkdir -p /venv/dp_internal_token_dir && \
    rm -rf /root/.cache

ENV PATH=/base/venv/bin:$PATH
ENV LANG='en_US.UTF-8' LANGUAGE='en_US.UTF-8' LC_ALL='en_US.UTF-8'
RUN git clone https://github.com/habibzadeh/DeepPavlov /base/DeepPavlov
RUN cd /base/DeepPavlov
WORKDIR /base/DeepPavlov
RUN git pull
RUN /base/venv/bin/pip install -U numpy==1.23.5 Cython
RUN /base/venv/bin/python /base/DeepPavlov/setup.py develop
RUN /base/venv/bin/python -c "import deeppavlov.models"
RUN rm -rf /root/.cache

RUN /base/venv/bin/pip install email-validator

RUN mkdir /base/config
RUN mkdir /base/train

COPY config/tfidf_logreg_en_faq.json /base/config/tfidf_logreg_en_faq.json
COPY train/file.csv /base/train/file.csv

RUN mkdir -p /root/.deeppavlov

RUN /base/venv/bin/python -m deeppavlov install /base/config/tfidf_logreg_en_faq.json
RUN /base/venv/bin/python -m deeppavlov download /base/config/tfidf_logreg_en_faq.json
RUN /base/venv/bin/python -m deeppavlov train /base/config/tfidf_logreg_en_faq.json
CMD /base/venv/bin/python -m deeppavlov riseapi /base/config/tfidf_logreg_en_faq.json