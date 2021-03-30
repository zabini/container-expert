FROM debian

LABEL app="Linux tips container experts"
ENV AULA="Tosko dockerfile"

RUN apt-get update && apt-get install -y stress && apt-get clean

CMD stress --cpu 1 --vm-bytes 64M --vm 1
