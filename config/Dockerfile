FROM ubuntu:jammy

#Setup certs in the container.
RUN mkdir -p /root/.ssh
COPY id_rsa /root/.ssh/id_rsa
COPY id_rsa.pub /root/.ssh/id_rsa.pub

#Setup the entrypoint file
COPY entrypoint.sh /root/entrypoint.sh

#Setup permissions.
RUN chmod 600 /root/.ssh/id_rsa \
 && chmod 640 /root/.ssh/id_rsa.pub \
 && chmod 777 /root/entrypoint.sh \
 && echo "Host *" > /root/.ssh/config && echo " StrictHostKeyChecking no" >> /root/.ssh/config

#Install needed packages.
RUN apt-get -y update
RUN DEBIAN_FRONTEND=noninteractive apt-get -y upgrade
RUN DEBIAN_FRONTEND=noninteractive apt-get -y install curl gnupg2 apt-utils openssh-client openssh-server ansible git cron net-tools tcpdump iputils-ping nano sudo

#Upgrade packages, fixing missing.
RUN apt-get upgrade -y --fix-missing

#Add F5 Application Infrastructure Protection Agent
#RUN curl https://app.threatstack.com/APT-GPG-KEY-THREATSTACK | sudo apt-key add -
COPY APT-GPG-KEY-THREATSTACK /root/APT-GPG-KEY-THREATSTACK
RUN cat /root/APT-GPG-KEY-THREATSTACK | sudo apt-key add -
RUN echo "deb https://pkg.threatstack.com/v2/Ubuntu `lsb_release -c | cut -f2` main" | sudo tee /etc/apt/sources.list.d/threatstack.list > /dev/null
RUN sudo DEBIAN_FRONTEND=noninteractive apt-get -y update
RUN sudo DEBIAN_FRONTEND=noninteractive apt-get install threatstack-agent -y
RUN sudo tsagent setup --deploy-key=changethekeyvaluetoyourownthreatstackkey --ruleset="Base Rule Set"
#RUN sudo service threatstack restart
#RUN sudo systemctl start threatstack

EXPOSE 22

WORKDIR /root

ENTRYPOINT ["/root/entrypoint.sh"]