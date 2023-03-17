FROM eclipse-temurin:11.0.18_10-jdk-jammy

ENV TINI_VERSION v0.19.0
ARG TARGETPLATFORM
RUN case ${TARGETPLATFORM} in \
         "linux/amd64")  TINI_ARCH=amd64 ;; \
         "linux/arm64")  TINI_ARCH=arm64 ;; \
    esac \
 && wget -q https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini-static-${TINI_ARCH} -O /tini

# Install common Dojo scripts
ENV DOJO_VERSION=0.10.3
RUN apt-get update && \
  DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
  sudo git ca-certificates wget make && \
  git clone --depth 1 -b ${DOJO_VERSION} https://github.com/kudulab/dojo.git /tmp/dojo_git &&\
  /tmp/dojo_git/image_scripts/src/install.sh && \
  rm -r /tmp/dojo_git

RUN DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends nano less gpg gpg-agent zip unzip

ENV GRADLE_VERSION 8.0.2
ENV GRADLE_HOME /usr/lib/gradle/gradle-${GRADLE_VERSION}
RUN cd /tmp &&\
  wget https://services.gradle.org/distributions/gradle-${GRADLE_VERSION}-bin.zip &&\
  unzip gradle-${GRADLE_VERSION}-bin.zip && mv gradle-${GRADLE_VERSION}/ /usr/lib/ &&\
  rm gradle-${GRADLE_VERSION}-bin.zip &&\
  ln -s /usr/lib/gradle-${GRADLE_VERSION}/bin/gradle /usr/bin/gradle

ENV AWS_CLI_VERSION=2.9.23
COPY aws-cli.rsa /tmp/aws-cli.rsa
RUN gpg --import /tmp/aws-cli.rsa && \
    case ${TARGETPLATFORM} in \
         "linux/amd64")  AWS_ARCH=x86_64 ;; \
         "linux/arm64")  AWS_ARCH=aarch64 ;; \
    esac && \
    curl "https://awscli.amazonaws.com/awscli-exe-linux-${AWS_ARCH}-${AWS_CLI_VERSION}.zip" -o "awscliv2.zip" && \
    curl -o awscliv2.sig https://awscli.amazonaws.com/awscli-exe-linux-${AWS_ARCH}-${AWS_CLI_VERSION}.zip.sig && \
    gpg --verify awscliv2.sig awscliv2.zip && \
    unzip awscliv2.zip && \
    sudo ./aws/install && \
    rm awscliv2.sig && \
    rm awscliv2.zip

ENV TERRAFORM_VERSION=1.3.8
RUN case ${TARGETPLATFORM} in \
         "linux/amd64")  TERRAFORM_ARCH=amd64 ;; \
         "linux/arm64")  TERRAFORM_ARCH=arm64 ;; \
    esac && \
  wget https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_${TERRAFORM_ARCH}.zip &&\
  wget -O terraform_${TERRAFORM_VERSION}_SHA256SUMS https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_SHA256SUMS &&\
  cat terraform_${TERRAFORM_VERSION}_SHA256SUMS | grep linux_${TERRAFORM_ARCH} > mySHA256SUM.txt &&\
  sha256sum -c --strict mySHA256SUM.txt &&\
  unzip terraform_${TERRAFORM_VERSION}_linux_${TERRAFORM_ARCH}.zip -d /bin &&\
  rm -f terraform_${TERRAFORM_VERSION}_linux_${TERRAFORM_ARCH}.zip

RUN mkdir -p /home/dojo/.terraform.d/plugin-cache &&\
  chown dojo:dojo -R /home/dojo/.terraform.d

COPY terraformrc /home/dojo/.terraformrc
RUN chown dojo:dojo /home/dojo/.terraformrc

# Install jq to parse json in bash
RUN apt-get install -y jq

COPY etc_dojo.d/scripts/* /etc/dojo.d/scripts/
COPY etc_dojo.d/variables/* /etc/dojo.d/variables/

COPY profile /home/dojo/.profile
COPY bashrc /home/dojo/.bashrc
RUN chown dojo:dojo /home/dojo/.profile /home/dojo/.bashrc

RUN chmod +x /tini
ENTRYPOINT ["/tini", "-g", "--", "/usr/bin/entrypoint.sh"]
CMD ["/bin/bash"]