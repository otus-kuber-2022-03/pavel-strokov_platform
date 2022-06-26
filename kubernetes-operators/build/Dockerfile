FROM python:3.7
COPY templates ./templates
COPY mysql-operator.py ./mysql-operator.py
RUN /usr/local/bin/python -m pip install --upgrade pip && pip install kopf kubernetes pyyaml jinja2
CMD kopf run /mysql-operator.py