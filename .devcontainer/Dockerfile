# FROM python:3.10
# FROM ghcr.io/quarto-dev/quarto-full:1.5.57
FROM ghcr.io/quarto-dev/quarto:1.6.10

# ARG OS_USERNAME=quarto
# ARG UID=1000
# ARG GID=1000

# Arguments
# ARG PYTHON_VER
# ARG ENV_NAME

# Environment variables
# ENV ENV_NAME=$ENV_NAME
# ENV PYTHON_VER=$PYTHON_VER

# Copy files
# RUN mkdir requirements
# COPY requirements.txt requirements/
# COPY install_dependencies.sh requirements/

RUN apt -y update && apt -y upgrade

RUN apt -y install git python3 pip plantuml
RUN pip install jupyter 

RUN quarto check

RUN quarto install tool tinytex --no-prompt --update-path
RUN quarto install chromium  --no-prompt --update-path
RUN quarto install extension pandoc-ext/diagram --no-prompt --update-path

# fra https://groups.google.com/a/apereo.org/g/sakai-dev/c/-6eI2Y5fB88
RUN apt-get install -y libgbm-dev libxkbcommon-x11-0 libgtk-3-0

# Install dependencies
# RUN bash requirements/install_dependencies.sh $ENV_NAME $PYTHON_VER