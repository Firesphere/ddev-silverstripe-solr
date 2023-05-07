[![tests](https://github.com/ddev/ddev-silverstripe-solr/actions/workflows/tests.yml/badge.svg)](https://github.com/ddev/ddev-drupal9-solr/actions/workflows/tests.yml)

## What is this?

This repository allows you to quickly install Apache Solr for Silverstripe 4+ into a [Ddev](https://ddev.readthedocs.io) project using just `ddev get firesphere/ddev-silverstripe-solr`. It uses the [documentation for `firesphere/solr-search`](https://firesphere.github.io/solr-docs).

## Installation on Silverstripe


1. `ddev get ddev/ddev-silverstripe-solr && ddev restart`
2. Install the required/relevant Silverstripe requirements: `ddev composer require firesphere/solr-search`
    * Note that there are currently some known issues around Guzzle dependencies in the module, that are currently in the process of being resolved.
3. Solr is set to use the FileConfigStore. Ensure its data location is at `.ddev/solr` and the host is set to `solr`:
    ```yml
    ---
    Name: LocalSolrSearch
    After:
      - 'SolrSearch'
    Only:
      environment: 'dev'
    ---
    Firesphere\SolrSearch\Services\SolrCoreService:
      config:
        endpoint:
          localhost:
            host: solr
      store:
        path: './.ddev/solr'
    Firesphere\SolrSearch\Indexes\BaseIndex:
      dev:
        Classes:
          - Page
        FulltextFields:
          - Title
          - Content
    ```
4. Run the Silverstripe Solr configuration command `ddev exec sake dev/tasks/SolrConfigureTask` or with the ddev-contrib add-on installed `ddev sake dev/tasks/SolrConfigureTask`
5. Restart ddev with `ddev restart`
6. Now your core configuration should be visible in Solr by visiting {yourproject}.ddev.site:8983, and check the schema for `dev`*
7. Now you can add documents, update your index, etc. [as per the documentation](https://firesphere.github.io/solr-docs/).
* **NOTE** You will need to restart your solr engine, every time you change your core configuration with `ddev restart`.
* **NOTE** Be aware you'll need to either rename or copy your Solr configuration, for production environments and update it as such.

## Explanation

* It installs a [`.ddev/docker-compose.solr.yaml`](docker-compose.solr.yaml) using the solr:8 docker image.
* A [.ddev/docker-entrypoint-initdb.d/solr-configupdate.sh](solr/docker-entrypoint-initdb.d/solr-configupdate.sh) is included and mounted into the Solr container so that you can change Solr config in `.ddev/solr/conf` with just a `ddev restart`.

## Interacting with Apache Solr

* The Solr admin interface will be accessible at: `http://<projectname>.ddev.site:8983/solr/` For example, if the project is named `myproject` the hostname will be: `http://myproject.ddev.site:8983/solr/`.
* To access the Solr container from inside the web container use: `http://solr:8983/solr/`
* A Solr core is automatically created by default with the name "dev"; it can be accessed (from inside the web container) at the URL: `http://solr:8983/solr/dev` or from the host at `http://<projectname>.ddev.site:8983/solr/#/~cores/dev`. You can obviously create other cores to meet your needs.
