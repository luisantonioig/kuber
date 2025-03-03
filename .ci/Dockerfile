FROM ubuntu:20.04 as layer1
ARG OVERRIDE_CHROOT_DIR=./chroot-env
COPY --chown=root:root ${OVERRIDE_CHROOT_DIR} /layer1
COPY  ./.ci/merge-root.sh /merge-root
RUN bash -e /merge-root /layer1 \
 && rm -rf /layer1 \
 && rm -rf merge-root

# RUN mkdir /layer2 && mv /layer1/bin /layer2

# ## psql is required for migration
# FROM ubuntu:20.04 as postgresql-cli
# RUN apt-get update && apt-get  install -y postgresql-client   && rm -rf /var/lib/apt/lists/* && apt-get clean autoclean && apt-get autoremove -y


# ## layer with library and executables
# FROM  ubuntu:20.04 as layer2
# COPY --from=layer1 /layer1  /layer1

FROM node:16 as ui-layer
WORKDIR /app
ENV NODE_OPTIONS=--max_old_space_size=6144 
COPY ./playground/package.json ./playground/package-lock.json ./
RUN npm ci
COPY ./playground .
RUN VITE_API_URL= npm run build

FROM ubuntu:20.04
LABEL "maintainer"="Sudip Bhattarai"
COPY --from=layer1  / /
WORKDIR /app
COPY  --from=ui-layer /app/dist/ .
EXPOSE 8081
HEALTHCHECK --interval=40s --timeout=10s --start-period=30s --retries=2 CMD [ "/bin/kuber" , "--healthcheck" ]
ENTRYPOINT  /bin/kuber
