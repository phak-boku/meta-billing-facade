Service Parameters
==================
<dl>
  <dt>Service Port</dt> <dd>21100</dd>
  <dt>JMX Port</dt><dd>21103</dd>
  <dt>Application Name</dt><dd>meta-billing-facade</dd>
</dl>

Maven Profiles and Plugins
==========================

Must be invoked explicitly.  (Re)Creates the database schema and ensures the correct users are created and have
applicable permissions.  Invoke maven with this profile any time you'd like to set the database back to default
settings.  You should invoke maven with this profile the first time you check this project out to ensure there is a
schema available for the liquibase plugin.

Invoking:

    $> mvn clean package -Pinitdb


Runs by default in dev environment. Applies the projects' database changesets to an existing schema, ensuring your
database always has the latest changes.

Invoking:

    $> mvn clean package -Pliquibase

To prevent this profile from running, even in dev mode (useful in conjunction with liquibase-gen-sql), prefix the
profile with "-":

    $> mvn clean package -P-liquibase,liquibase-gen-sql


Must be invoked explicitly.  Generates a migration.sql file containing the changes that would have been applied
with the liquibase profile. This is mostly used for applying changesets in production, allowing ops to review and
tweak before executing against the database server.  To run this in dev mode, you'll need to prevent the liquibase
plugin from running automatically.  See the "liquibase" profile description. The resulting script can be found at
argo-billing-facade-persistence/target/liquibase/migrate.sql.

Invoking:

    $> mvn clean package -Pliquibase-gen-sql

Must be invoked explicitly. Assembles deployment bundles for any projects that require one.  Typically a bundle is a
tar.gz file containing appropriate libraries, resources, and scripts for starting and stopping the application. The
bundles can be found at
<bundled_project>/target/*.tar.gz.

The archetype that created this project provides configs for two bundles, by default. One is the server bundle that
gets deployed to production.  The second is a bundle containing the minimum liquibase configs required to run
liquibase in production, alleviating ops from downloading the whole source tree.

Invoking:

    mvn clean package -Pbundle

The maven exec plugin has been configured to allow running the server component of this project from the command
line.

Invoking:

    $> cd argo-billing-facade-server
    $> mvn exec:java

    (Control-c to quit.)

This plugin runs any tests ending in "IT".  This plugin has not been explicitly tied in to the build process; this
is done to allow for manual invocation, since a running server is generally required in order to run integration
tests.  This plugin can be called from within any module, including the parent.  Executing within a specific module
only runs the test from that module.

Invoking (ensure the server is running - see exec:java):

    $> mvn failsafe:integration-test


Runtime Switches and Parameters
===========================
The runtime behavior of the server can be affected by supplying system properties at startup. Any property managed by
Spring can be overridden by passing a system property to the java command during start up. A system property can be
supplied in an IDE's VM options box in a run/debug profile.  It can be supplied on the bundle's service script, or using
mvn exec:java, as well.

Examples:

    $> ./bin/service start -Dmy.property=foo
    $> mvn exec:java -Dmy.property=foo


Valid values - true | false

Allows for an explicit data store to be specified, overriding the environment's default store.  An in-memory h2
database is the default store in "test" environment.  mysql is used in all other environments.  It can be useful to
explicitly run the server using the h2 datastore when running integration tests.

Invoking:

    $> ./bin/service start -DinMemoryDb=true
    $> ./bin/service start -DinMemoryDb=false
    $> mvn exec:java -DinMemoryDb=true
    $> mvn exec:java -DinMemoryDb=false


Server Bundle
=============
The server bundle has the following structure:

    /bin
       /service
       /service.settings
    /lib
       /*.jar
    /resources (optional)
       /(any external resources, such as overriding properties files)
    /state
        service.pid
        /logs
            /app.log
            /err.log
            /archive
                /app.*.log
                /err.*.log

The ./bin/service script starts and stops the server.  It uses applications-specific parameters defined in
./bin/service.settings.  This script can be executed from any directory, even if it has been symlinked. Any options
provided to the script are passed to the JVM, not the application's main.  This allows system properties to be specified
when starting the service.

./bin/service has the following interface/commands:

Starts the server, if it is not already running.

Stops the server.

Stops the server if it is running, and then starts the server.

Displays the status of the server, and if it is running, what PID it is running under.

Displays the command that would be issued, the jars and directories that will be on the classpath, and the
environment that the process will run under.

Examples

    $> ./bin/service start
    $> ./bin/service start -DinMemoryDb=true
    $> ./bin/service stop

The ./bin/service.settings file contains the Main java class to be executed, the service name, and the JVM arguments
such as memory settings.
