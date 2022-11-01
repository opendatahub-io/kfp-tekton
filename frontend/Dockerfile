FROM registry.redhat.io/rhel8/nodejs-14:1-83 as build

USER root

COPY . .

WORKDIR ./frontend

RUN npm ci && npm run postinstall 
RUN npm run build

# Generate the dependency licenses files (one for the UI and one for the webserver),
# concatenate them to one file under ./src/server
RUN npm i -D license-checker
RUN node gen_licenses . && node gen_licenses server && \
    cat dependency-licenses.txt >> server/dependency-licenses.txt

FROM registry.redhat.io/ubi8/nodejs-14-minimal:1-66

RUN mkdir ./{server,client} && \
    chown 1001:root ./{server,client}

COPY --from=build /opt/app-root/src/frontend/server ./server
COPY --from=build /opt/app-root/src/frontend/build ./client

WORKDIR ./server

USER 1001

EXPOSE 3000
RUN npm run build
ENV API_SERVER_ADDRESS http://localhost:3001
CMD node ./dist/server.js ../client/ 3000
