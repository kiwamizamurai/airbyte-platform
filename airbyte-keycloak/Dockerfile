# the keycloak version here should be kept in sync with the keycloak version in deps.toml
# Keycloak is hosted on quay, which has historically been unreliable. Increase reliability
# by mirroring the image on dockerhub, which experience has shown to be more reliable.
# This also lets us take advantage of our pull-through cache.
# Thus, when upgrading this image, be sure to also upgrade the mirrored image on dockerhub.
# See README.md for instructions.
FROM airbyte/mirrored-keycloak:23.0.3

WORKDIR /opt/keycloak

COPY bin/scripts/entrypoint.sh entrypoint.sh
COPY bin/themes themes

# Doing this instead of creating a separate file and copying it to ensure that we get any keycloak updates to this conf file.
RUN cp conf/cache-ispn.xml conf/cache-ispn-override.xml
RUN sed -i conf/cache-ispn-override.xml -e 's/<distributed-cache name=\"sessions\" owners=\"2\">/<distributed-cache name=\"sessions\" owners=\"3\">/g'
RUN sed -i conf/cache-ispn-override.xml -e 's/<distributed-cache name=\"authenticationSessions\" owners=\"2\">/<distributed-cache name=\"authenticationSessions\" owners=\"3\">/g'

# Make sure that the two lines we wanted to be there are actually there
# i.e. keycloak didn't change its config file
RUN grep '<distributed-cache name="sessions" owners="3">' conf/cache-ispn-override.xml -q
RUN grep '<distributed-cache name="authenticationSessions" owners="3">' conf/cache-ispn-override.xml -q

# Create the directory for the infinispan global-state persistence
RUN mkdir -p /opt/keycloak/data/infinispan

# Inserting the <global-state> block after the <cache-container> start tag
RUN sed -i '/<cache-container /a \    <global-state><persistent-location path="\/opt\/keycloak\/data\/infinispan"\/><\/global-state>' conf/cache-ispn-override.xml
# Make sure that the <global-state> block is actually there
# i.e. keycloak didn't change its config file
RUN grep '<global-state><persistent-location path="/opt/keycloak/data/infinispan"/></global-state>' conf/cache-ispn-override.xml -q

ENTRYPOINT ["./entrypoint.sh"]
