FROM debian:stretch-slim

ARG GRAFANA_TGZ="grafana-latest.linux-x64.tar.gz"

RUN apt-get update && apt-get install -qq -y tar vim wget && \
    apt-get autoremove -y && \
    rm -rf /var/lib/apt/lists/*

RUN wget https://dl.grafana.com/oss/release/grafana-5.4.2.linux-amd64.tar.gz -O /tmp/grafana.tar.gz

RUN mkdir /tmp/grafana && tar xfvz /tmp/grafana.tar.gz --strip-components=1 -C /tmp/grafana

FROM debian:stretch-slim

ARG GF_UID="472"
ARG GF_GID="472"

ENV PATH=/usr/share/grafana/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin \
    GF_PATHS_CONFIG="/etc/grafana/grafana.ini" \
    GF_PATHS_DATA="/var/lib/grafana" \
    GF_PATHS_HOME="/usr/share/grafana" \
    GF_PATHS_LOGS="/var/log/grafana" \
    GF_PATHS_PLUGINS="/var/lib/grafana/plugins" \
    GF_PATHS_PROVISIONING="/etc/grafana/provisioning"

WORKDIR $GF_PATHS_HOME

RUN apt-get update && apt-get -y upgrade && \
    apt-get install -qq -y libfontconfig ca-certificates curl && \
    apt-get autoremove -y && \
    rm -rf /var/lib/apt/lists/*

COPY --from=0 /tmp/grafana "$GF_PATHS_HOME"

RUN mkdir -p "$GF_PATHS_HOME/.aws" && \
    groupadd -r -g $GF_GID grafana && \
    useradd -r -u $GF_UID -g grafana grafana && \
    mkdir -p "$GF_PATHS_PROVISIONING/datasources" \
             "$GF_PATHS_PROVISIONING/dashboards" \
             "$GF_PATHS_LOGS" \
             "$GF_PATHS_PLUGINS" \
             "$GF_PATHS_DATA" && \
    cp "$GF_PATHS_HOME/conf/sample.ini" "$GF_PATHS_CONFIG" && \
    cp "$GF_PATHS_HOME/conf/ldap.toml" /etc/grafana/ldap.toml && \
    chown -R grafana:grafana "$GF_PATHS_DATA" "$GF_PATHS_HOME/.aws" "$GF_PATHS_LOGS" "$GF_PATHS_PLUGINS" && \
    chmod 777 "$GF_PATHS_DATA" "$GF_PATHS_HOME/.aws" "$GF_PATHS_LOGS" "$GF_PATHS_PLUGINS"

EXPOSE 3000

COPY ./run.sh /run.sh

RUN chmod 755 /run.sh
RUN chmod +x /run.sh

# chmod some directories to avoid openshift permissions issues on RBAC
RUN mkdir -p /var/lib/grafana
RUN chmod -R 777 /var/lib/grafana

ENTRYPOINT [ "/run.sh" ]
