Container's eleven
====================

A checklist for containers - 11 things that will make your app production-ready.

1. Explicitly declared and isolated dependencies
---------------------------------------------------

- Use container descriptors (e.g. ``Dockerfile``s)
- Store descriptors with the codebase
- Select a proper base image
- Minimize usage of system-wide packages
- Make use of ``rubygems/pip/nuget/npm/yarn/apt-get/yum/apk``
- Use ONLY those packages that are actually needed 

2. One stateless and share-nothing process 
---------------------------------------------------

- Make no disctinction between local and third-party services.
- Treat backing services as attached resources
- Don’t run your app in a container along with datastores (such as MySQL or CouchDB), 
  messaging/queueing systems (such as RabbitMQ or Beanstalkd), 
  email services (such as Postfix) or caching systems (such as Memcached).
- Don’t assume the local file system is permanent.
- Don't keep session state in your application.

## 3. Configuration stored in the environment
---------------------------------------------------

- Make your app to use environment variables out of the box.
- Define default values for variables, fail when it’s required and there is no value.
- Provide a documentation file – simple ``readme.md`` will do
- Allow to mount configuration files as a volume – if modification of the app is not possible

## 4. Services exported via port bindings
---------------------------------------------------

- Don’t rely on runtime injection of a webserver into the execution environment to create a web-facing service
- Embed a webserver in your app such as Tornado for Python, Thin for Ruby, Jetty/Tomcat/Undertow for Java
- Reuse webservers such as nginx or apache as part of your app
- Explicitly expose ports in the image manifest file
- Remember that an external proxy is a thing – your app should be prepared to have it’s base address rewritten (e.g. passed as env vars)
- Make apps, not libraries

## 5. Fast startup and graceful shutdown
---------------------------------------------------

- Startup should take a few seconds – counting from the time the launch command is executed 
  until the process is up and ready to receive requests or jobs
- React on a ``SIGTERM`` signal – perform a graceful shutdown and free all resources (release locks, close ports etc.)
- Be robust against sudden death – handle unexpected, non-graceful terminations (such as clients disconnect or timeouts)

## 6. Logs treated as event streams
---------------------------------------------------

- Treat logs as streams and write them unbuffered, to stdout or stderr
- Never concern your app with routing or storage of its output stream
- Don’t write logs to the file system 
- Don’t attempt to manage logfiles

## 7. Reasonably small size
---------------------------------------------------

- Use a smaller base image. Alpine is the way to start. Most likely, there are alpine tags for the programming language you are using
- Minimize layers by combining your `RUN` statements
- Fine-tune your package manager to install only what’s needed with no overhead
  use commands like ``apt-get --no-install-recommends`` , ``apk add --no-cache``
- Clean up after installing packages in the same layer
  use commands like ``rm -rf /var/lib/apt/lists/*`` , yum clean all
- Don’t install debug tools (like ``vim/curl``) in your production images
- Verify the quality of your images – lint your ``Dockerfile``s in external tools such as http://www.fromlatest.io


## 8. Verifiable readiness and health status
---------------------------------------------------

- Expose health-check endpoints
- Use HTTP endpoints where possible
- Provide an additional shell scripts where HTTP is not an option

## 9. Discoverability
---------------------------------------------------

- Give meaningful content where it is expected – when the container exposes port `80` or `8080`, it is expected that hitting the base URL 
  will deliver features, or at least a guide/hyperlink telling how to get to the features.
- Use standard port bindings for standard protocols
- Deliver mature REST APIs – define proper resources, HTTP verbs and hypermedia controls. 
  Refer to Richardson’s REST maturity model at https://martinfowler.com/articles/richardsonMaturityModel.html
- Use Swagger for API description and expose the definition in your app
- Embed Swagger Sandbox right in your application

## 10. Unidirectional, declarative deployment
---------------------------------------------------

- Be as declarative as possible when describing your deployment – prefer deployment descriptors
  (such as `docker-compose` yaml files, kubernetes yaml descriptors etc.) over shell scripts.
- Keep your deployment versioned – e.g. in a git repository
- Don’t mutate releases/deployed apps – make sure there are no additional manual steps and 
  no tampering with containers once they are deployed.
- Use rollbacks if you want to return to previous versions.
- Enforce a strict, one-way separation between the build, release, and run stages - 
  it is impossible to make changes to the code at runtime and to propagate those changes back to the build stage
- Ship admin code with application code to avoid synchronization issues – things such as database migrations, one-time scripts
- Run admin/management tasks as one-off processes

## 11. Dev/Prod parity
---------------------------------------------------

- Make the time gap small - write code and have it deployed hours or even just minutes later (or fight for it if it’s still impossible!)
- Make the personnel gap small – if you wrote the code, get involved in deploying it and watching its behavior in production!
- Make the tools gap small - keep development and production as similar as possible – preferably the same declarative descriptors. 
- Don’t use different backing services between dev and prod – if it’s Postgres on prod, why it’s SQLite on dev? 
- Use declarative provisioning tools such as **Puppet**, **Ansible**, **Chef** along with lightweight virtual environments like **Vagrant** 
  to run local environments which closely approximate production
- Give a great zero-to-dev experience – preferably a one-liner 
 (like ``vagrant up`` or ``docker-compose up``) to spin the whole thing up, 
  a ``readme.md`` comment wouldn’t kill you either.
