FROM quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.22.0

USER root
RUN curl -O https://dl.signalsciences.net/sigsci-module-nginx-native/1.0.10/debian/stretch/nginx-module-sigsci-nxo_1.15.8-47-stretch_amd64.deb
RUN dpkg --force-all -i ./nginx-module-sigsci-nxo_1.15.8-47-stretch_amd64.deb
USER www-data
