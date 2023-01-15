FROM debian:stable-slim AS build
ARG ELECTRS_VERSION
RUN apt-get update \
  && apt-get install build-essential cargo cmake clang wget -y \
  && apt-get clean \
  && mkdir /tmp/electrs \
  && cd /tmp/electrs \
  && wget https://github.com/romanz/electrs/archive/refs/tags/v${ELECTRS_VERSION}.tar.gz \
  && tar -xzf /tmp/electrs/*.tar.gz -C /tmp/electrs \
  && cd /tmp/electrs/electrs-${ELECTRS_VERSION} \
  && cargo build --no-default-features --locked --release \
  && mv /tmp/electrs/electrs-${ELECTRS_VERSION}/target/release/electrs /usr/local/bin/electrs

FROM debian:stable-slim
ARG GROUP_ID
COPY --from=build /usr/local/bin/electrs /usr/local/bin/electrs
RUN addgroup --gid ${GROUP_ID} electrs \
  && adduser --disabled-password --gecos "" --ingroup electrs electrs \
  && mkdir /home/electrs/.electrs \
  && chown electrs:electrs /home/electrs/.electrs

USER electrs

CMD electrs --conf ~/.electrs/electrs.conf
