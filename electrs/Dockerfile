FROM debian:stable-slim

ARG ELECTRS_VERSION
ARG USER_ID
ARG GROUP_ID

ENV PATH=/opt/electrs:$PATH

RUN apt-get update -y \
  && apt-get install wget cargo clang cmake build-essential -y \
  && apt-get clean

RUN mkdir /tmp/electrs \
  && cd /tmp/electrs \
  && wget https://github.com/romanz/electrs/archive/refs/tags/v${ELECTRS_VERSION}.tar.gz \
  && tar -xzf /tmp/electrs/*.tar.gz -C /tmp/electrs \
  && cd /tmp/electrs/electrs-${ELECTRS_VERSION} \
  && cargo build --no-default-features --locked --release
  
RUN mkdir /opt/electrs \
  && mv /tmp/electrs/electrs-${ELECTRS_VERSION}/target/release/electrs /opt/electrs/electrs \
  && groupadd electrs -g ${GROUP_ID} \
  && useradd -u ${USER_ID} -g ${GROUP_ID} -m electrs \
  && mkdir /home/electrs/.electrs \
  && chown electrs:electrs /home/electrs/.electrs

COPY --chown=electrs:electrs electrs.conf /home/electrs/.electrs/electrs.conf
  
RUN apt-get remove wget cargo clang cmake build-essential -y \
  && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

USER electrs

CMD electrs --conf ~/.electrs/electrs.conf
