FROM debian:stable-slim AS build
ARG ELECTRS_VERSION
RUN apt-get update \
  && apt-get install build-essential cargo cmake clang git curl -y \
  && apt-get clean \
  && cd /tmp \
  && git clone --branch v${ELECTRS_VERSION} https://github.com/romanz/electrs.git \
  && cd electrs \ 
  && curl https://romanzey.de/pgp.txt | gpg --import \
  && git verify-tag v${ELECTRS_VERSION} \
  && cargo build --no-default-features --locked --release \
  && mv target/release/electrs /usr/local/bin/electrs


FROM debian:stable-slim
ARG GROUP_ID
COPY --from=build /usr/local/bin/electrs /usr/local/bin/electrs
RUN addgroup --gid ${GROUP_ID} electrs \
  && adduser --disabled-password --gecos "" --ingroup electrs electrs \
  && mkdir /home/electrs/.electrs \
  && chown electrs:electrs /home/electrs/.electrs

USER electrs

CMD electrs --conf ~/.electrs/electrs.conf
