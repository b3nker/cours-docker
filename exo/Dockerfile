FROM python

ADD . /code/
WORKDIR code/code/
RUN pip install -r requirements.txt
RUN python main.py

EXPOSE 5000