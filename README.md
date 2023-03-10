# ISPRA Demo Server
This demonstration server shows how to set up a Download service based on the [OGC API-Features](https://ogcapi.ogc.org/features/) standards, following the INSPIRE [guidelines](https://github.com/INSPIRE-MIF/gp-ogc-api-features/blob/master/spec/oapif-inspire-download.md).

Technologies used:
- [pygeoapi](https://pygeoapi.io/) to setup multiple OGC API Complilant server instances
- [Traefik](https://traefik.io/) as HTTPS reverse proxy

Docker and Docker Compose are used for orchestration.

# Getting started
1. Run the initialization script

    ```sh
    $ chmod +x init.sh
    $ ./init.sh
    ```

    This will generate the `logs` folder and the `acme.json` file that Traefik will use to store generated LetsEncrypt certificates.

2. Set the LetsEncrypt registration email into the `traefik.yml` file

    ```yml
    # file: traefik.yml
    certificatesResolvers:
        letsEncryptResolver:
            # Enable ACME (Let's Encrypt): automatic SSL.
            acme:
              # Email address used for registration.
              #
              # Required
              #
              email: "test@example.com"
    ```

3. For each container into the `docker-compose.yml` file, check the Traefik `Host` rule points to the correct domain. Change it if necessary.

    Example:
    
    ```yml
    # file: docker-compose.yml
    # pygeoapi instance configuration
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.hydrography.rule=Host(`labgis.di.unisa.it`) && PathPrefix(`api/hydrography`)"
    ```
    
4. For each pygeoapi instance, check the `server.url` parmeter into the `config.yml` file points to the correct domain. Change it if necessary:

    Example:
    
    ```yml
    # file: hydrography/config.yml
    server:
        url: https://labgis.di.unisa.it/hydrography
    ```
5. Check the `traefik_dynamic.yml` dynamic configuration file:

    Some traefik options (like the Dashboard [api router](https://doc.traefik.io/traefik/operations/api/#configuration)
    and the [BasicAuth](https://doc.traefik.io/traefik/middlewares/http/basicauth/) middleware) must be configured with traefik dynamic configuration.
    
    These options are provided by the `traefik_dynamic.yml` file.
    
    - Check the `http.routers.api.rule` `Host` rule points to the correct domain. Change it if necessary.
    
        Example:
        ```yml
        # file: traefik_dynamic.yml
        http:
          routers:
            # Overrides the default 'api' router 
            api:
              # The rule matches http://example.com/api or http://example.com/dashboard
              # but does not match http://example.com/hello
              rule: Host(`monitor.labgis.di.unisa.it`) && (PathPrefix(`/api`) || PathPrefix(`/dashboard`))
        ```

    - Check the users list at `http.middlewares.auth.basicAuth.users`
    
        This users will be allowed to login into the Traefik Dashboard page. The default user is `admin:labgis`. 

        Use [htpasswd](https://httpd.apache.org/docs/2.4/programs/htpasswd.html) to generate new users:
        
        ```sh
        $ htpasswd -nb user your_secure_password
        user:$apr1$ClkgSq7N$MxFbvzUIKChlTiTEYpNRj1
        ```
        
        Add the output to the users list.

6. Run Docker Compose

    ```sh
    $ docker-compose up
    ```

Using the default configuration, the following urls should be available:

- [https://monitor.labgis.di.unisa.it/dashboard/](https://monitor.labgis.di.unisa.it/dashboard/): Traefik Dashboard monitor
- [https://labgis.di.unisa.it/oapi/aqd](https://labgis.di.unisa.it/oapi/aqd/): ISPRA Air Quality pygeoapi instance
- [https://labgis.di.unisa.it/oapi/hydrography](https://labgis.di.unisa.it/oapi/hydrography/): ISPRA Hydrography pygeoapi instance
