
Working with Solr Plugins System
================================

### Table of Contents

*   [Solr Plugin Management](#toc-solr-plugin-management-0)
*   [Package Management Basics](#toc-package-management-basics-1 "Package Management Basics")
*   [Preparing the Package Repository](#toc-preparing-the-package-repository-2 "Preparing the Package Repository")
*   [Creating a Private and a Public Key](#toc-creating-a-private-and-a-public-key-3 "Creating a Private and a Public Key")
*   [Generating Package Signature](#toc-generating-package-signature-4 "Generating Package Signature")
*   [Repository Description](#toc-repository-description-5 "Repository Description")
*   [Adding a New Package Repository](#toc-adding-a-new-package-repository-6 "Adding a New Package Repository")
*   [Installing and Removing Solr Packages](#toc-installing-and-removing-solr-packages-7 "Installing and Removing Solr Packages")
*   [How Solr Packages Work Under the Hood](#toc-how-solr-packages-work-under-the-hood-8 "How Solr Packages Work Under the Hood")
*   [Solr Package API](#toc-solr-package-api-9 "Solr Package API")
*   [Security](#toc-security-10 "Security")
*   [Summary](#toc-summary-11 "Summary")

[Apache Solr](https://sematext.com/guides/solr/) was always ready to be extended. What was only needed is a binary with the code and the modification of the Solr configuration file, the **solrconfig.xml** and we were ready. It was even simpler with the Solr APIs that allowed us to create various configuration elements – for example, request handlers. What’s more, the default Solr distribution came with a few plugins already – for example, the Data Import Handler or Learning to Rank.

As consultants working with clients across different industries, dealing with a wide variety of use cases with Solr clusters monitored by [Sematext Cloud](https://sematext.com/cloud/) the next thing that we saw the need for were plugins. Installing those plugins was not hard – put a jar file in a defined place, modify the configuration, reload the core/collection or restart Solr and we are ready. Well not so fast. What if you had hundreds of Solr nodes and you needed to upgrade the plugin or even install it. Yes, that’s where things can get nasty and require automation. Solr was not very supportive in this until one of the recent releases. All of the users that wanted to extend Solr were doing the same thing – manual jar loading. We did the same thing with our plugins – like the [Researcher](https://github.com/sematext/solr-researcher) or [Query Segmenter](https://github.com/sematext/query-segmenter).

With the release of Solr 8.4.0, we’ve got a new functionality that helps us with extending Solr – the plugin management. It allows installing plugins from remote locations and it makes it very easy to do so for us as users. Today I wanted to show you not only how to install Solr plugins using this new feature, but also how to prepare your own plugin repository. Let’s get started.

![](https://sematext.com/wp-content/uploads/2020/05/solr-plugins-post-image1.png)

[Solr Plugin Management](#toc-solr-plugin-management-0)
-------------------------------------------------------

With Solr 8.4.0, we didn’t only get the script itself but also the whole set of changes under the hood. Those changes include things like package management APIs and scripts, class loader isolation, artifact read and write API and more.

Let’s start from the beginning though. By default Solr comes with the package loading turned off. One of the reasons for such a decision is security. Users could potentially force Solr to download malicious content, so you need to be sure that your environment is secure and you need to know potential downsides and risks of using that feature. But if we are sure that we want to run Solr with plugin management mechanism turned on we need to add the **enable.packages** property to Solr startup parameters and set it to true:

    $ bin/solr start -c -f -Denable.packages=true

Now we can start playing around with the packages.

Package Management Basics
-------------------------

Let’s try using the bin/solr script and see what it allows us to do when it comes to package management. The simplest way to check that is just by running the following command:

    $ cd bin/
    $ ./solr package

In the result we will get the following response:

    Found 1 Solr nodes:
    
    Solr process 20949 running on port 8983
    Package Manager
    
    ./solr package add-repo
    Add a repository to Solr.
    
    ./solr package install [:]
    Install a package into Solr. This copies over the artifacts from the repository into Solr's internal package store and sets up classloader for this package to be used.
    
    ./solr package deploy [:] [-y] [--update] -collections <package-name>[:] [-y] [--update] -collections  [-p param1=value1 -p param2=value2 …
    Bootstraps a previously installed package into the specified collections. It the package accepts parameters for its setup commands, they can be specified (as per package documentation).
    
    ./solr package list-installed
    Print a list of packages installed in Solr.
    
    ./solr package list-available
    Print a list of packages available in the repositories.
    
    ./solr package list-deployed -c
    Print a list of packages deployed on a given collection.
    
    ./solr package list-deployed
    Print a list of collections on which a given package has been deployed.
    
    ./solr package undeploy  -collections
    Undeploys a package from specified collection(s)
    
    Note: (a) Please add '-solrUrl http://host:port' parameter if needed (usually on Windows).
          (b) Please make sure that all Solr nodes are started with '-Denable.packages=true' parameter.

It seems we get everything that is needed. We can add repositories, we can list installed packages, we can install packages, we can deploy packages, list deployed ones and of course undeploy the ones that we no longer need.

At the time of writing this blog post, there were no Solr plugin repositories publicly available. But for us this is not bad – we can use that to learn even more. We just need to start by preparing our own plugin repository.

Preparing the Package Repository
--------------------------------

If you are using a repository that was already created, where the plugins are available you can skip this part of the blog post. But if you would like to learn how to set up a Solr plugin repository on your own, I’ll try to guide you through that process.

So there are a few steps that need to be taken:

*   You need to create a private key that will be used to sign your binaries
*   You need to create a public key that Solr will use to verify the signed packages
*   You need to create a repository description file that Solr will read when requesting packages from the repository
*   And of course, you need to have the binaries that you would like to expose as plugins. We will not be discussing this step though and I will assume you already have that. We created a very naive and simple code at [https://github.com/sematext/example-solr-module](https://github.com/sematext/example-solr-module). Have a look if you want.

![](data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7)

### Creating a Private and a Public Key

We will start by creating a **private key**. This key will be used to generate a signature of the binaries that we will be exposing as plugins. For that we will use **openssl**:

    $ openssl genrsa -out sematext_example.pem 512

The above command creates a 512 bits RSA key called **sematext\_example.pem**. With that generated, we can now create a public key that will be based on the above key.

The **public key** will be created from the private one and Solr will use it to verify the signatures of the files. The idea is as follows:

*   The package maintainer creates a signature of the package file using the **private key** and writes the signature in the repository description file,
*   During package deployment, the package signature is verified by Solr using the **public key**. If the signature doesn’t match – the package will not be deployed.

To create a public key we will again use the **openssl** command:

    $ openssl rsa -in sematext_example.pem -pubout -outform DER -out publickey.der

The output of the above command is a **publickey.der** file that we will upload to our repository location along with the binary file and the repository description file.

### Generating Package Signature

The last step is generating the signature of the file. We will once again use the **openssl** command:

    $ openssl dgst -sha1 -sign sematext.pem solr-example-module-1.0.jar | openssl enc -base64 | tr -d \\n | sed

As a result we will have the signature, which in our case looks as follows:

    iXyDDhYkYZgBrYCTxawAdeIJFYR+KHglK4m6uLSR1lo9pFm67dKfIzTmXPHasFVgLwVRbYvGMJG5p69TowMPAg==

Note it somewhere as we will need it soon.

Repository Description
----------------------

Now that we already have our binary file, the private and the public keys we can create the repository description file that Solr will be looking for inside the repository. This repository has to be called repository.json and needs to include a list of plugins that are available in our repository. Each plugin is defined by:

*   A name,
*   A description,
*   An array of versions, that include:

*   Version itself,
*   Release date of the given plugin version,
*   An array of artifacts for the version – the URL of the file and the signature that we generated earlier,
*   The manifest which includes supported Solr versions, default parameters, setup, uninstall and verification commands.

The **repository.json** file that we are using for the purpose of this blog post looks as follows:

    [
      {
    "name": "sematext-example", "description": "Example plugin created for blog post", "versions": [{
        "date": "2020-04-16", "artifacts": [{
                "url": "solr-example-module-1.0.jar",
                "sig": "iXyDDhYkYZgBrYCTxawAdeIJFYR+KHglK4m6uLSR1lo9pFm67dKfIzTmXPHasFVgLwVRbYvGMJG5p69TowMPAg=="
              }
            ],
            "manifest": {
              "version-constraint": "8 - 9",
              "plugins": [
                {
                  "name": "request-handler",
                  "setup-command": {
                    "path": "/api/collections/${collection}/config",
                    "payload": {"add-requesthandler": {"name": "${RH-HANDLER-PATH}", "class":
            "sematext-example:com.sematext.blog.solr.ExampleRequestHandler"}},
                    "method": "POST"
                  },
                  "uninstall-command": {
                    "path": "/api/collections/${collection}/config",
                    "payload": {"delete-requesthandler": "${RH-HANDLER-PATH}"},
                    "method": "POST"
                  },
                  "verify-command": {
                    "path": "/api/collections/${collection}/config/requestHandler?componentName=${RH-HANDLER-PATH}&meta=true",
                    "method": "GET",
                    "condition":
            "$['config'].['requestHandler'].['${RH-HANDLER-PATH}'].['_packageinfo_'].['version']",
                    "expected": "${package-version}"
                  }
                }
              ],
              "parameter-defaults": {
                "RH-HANDLER-PATH": "/sematextexample"
              }
            }
          }
        ]
      }
    ]

While most of the properties are self-descriptive you should put attention to one thing – the class of the request handler in the setup-command definition. Because of the out-of-the-box class loaders isolation, we need to provide a prefix with the name of the plugin to be able to create the request handler. If we won’t do that Solr will fail to create the request handler, because our class that implements the request handler will not be visible. Keep that in mind when creating the repository description file for your own plugins.

With all that we can upload it to some remote location like we did with [http://pub-repo.sematext.com/training/solr/blog/repo/](https://pub-repo.sematext.com/training/solr/blog/repo/) and start using it.

Adding a New Package Repository
-------------------------------

Once we are ready with setting up our own repository or we already have a repository that we would like to install plugins from we can add that repository to Solr. Just remember, to successfully add the repository it needs to provide the **repository.json** file. The second thing is security – you should avoid adding repositories that don’t use SSL. Adding a repository that doesn’t use a secure connection exposes you and your Solr for **man in the middle** attacks. The potential thing that can happen is that during the download of the package it can be replaced with a malicious version. Keeping your Solr secure is as important as keeping an eye on the Solr metrics by using one of the Solr monitoring tools like [Sematext Cloud](https://sematext.com/cloud/).

Now that we know about the potential security issue let’s use a secure location of the example Solr repository. We do that by using the following command:

    $ ./solr package add-repo sematext https://pub-repo.sematext.com/training/solr/blog/repo/

We are using new functionalities of the **bin/solr** script – the **package** one. We use one of the possible options, the **add-repo** which requires us to provide a name and the location. The name in our case is **sematext** and the location is the last provided parameter.

If the operation was successful Solr will give us information about the number of nodes found in the cluster, the process identifier and the port on which the instance is running. And finally, the last information that tells that the repository was added:

    Found 1 Solr nodes:
    
    Solr process 65854 running on port 8983
    Added repository: sematext

As a side note – I’ll omit the information about the number of nodes, process identifier and the Solr port from the other examples. It should be easier to see the crucial information returned by Solr.

Installing and Removing Solr Packages
-------------------------------------

Once the repository is added we can start using it. The first thing that you would usually do is listing the available packages and look for something that we can install to extend our Solr. To list all the available packages we should run the following command:

    $ bin/solr package list-available

The response to the above command should be similar to the following one:

    Available packages:
    -----
    sematext-example    Example plugin created for blog post
      Version: 1.0.0

In the response, we have a list of packages – each described with a name, description and version. Just as they were defined in the **repository.json** file. We are very close to being ready for installation. But there is one more thing – the public key that Solr will use to verify the package signature. Where to look for such a key? It will be provided to you or you can download it from the repository itself under the **publickey.der** name. I’ll do the latter and will download the key by using the following command:

    $ curl -s -o publickey.der -LO http://pub-repo.sematext.com/training/solr/blog/repo/publickey.der

Once we will have the key we can add it to Solr by using the **bin/solr** script its **package** part of the functionality and the **add-key** action:

    $ ./solr package add-key publickey.der

After all those steps we can finally start installing the packages. For example, let’s install the one package that we have available in our sample repository. We do that by running the following command:

    $ ./solr package install sematext-example:1.0.0

The response that I got from Solr was as follows:

    Posting manifest...
    Posting artifacts...
    Executing Package API to register this package...
    Response: {"responseHeader":{
        "status":0,
        "QTime":68}}
    sematext-example installed.

This means that our package is now ready to be used. Let’s create a collection where we can use the package by running the following command:

    $ ./solr create_collection -c test

By now we should have the package installed and a sample collection created. This means that we are finally ready to use that plugin. To do that we need to deploy it. We can do that to a single collection or multiple ones at the same time. For the purpose of this blog post I will use our **test** collection and will deploy our plugin by using the following command:

    $ ./solr package deploy sematext-example:1.0.0 -collections test

In addition to the name of the collection or collections that our plugin should be installed to, we need to provide the name of the plugin and its version. The response was as follows:

    Executing {"add-requesthandler":{"name":"/sematextexample","class":"sematext-example:com.sematext.blog.solr.ExampleRequestHandler"}} for path:/api/collections/test/config
    Execute this command (y/n):
    y
    Executing http://localhost:8983/api/collections/test/config/requestHandler?componentName=/sematextexample&meta=true for collection:test
    {
      "responseHeader":{
        "status":0,
        "QTime":1},
      "config":{"requestHandler":{"/sematextexample":{
            "name":"/sematextexample",
            "class":"sematext-example:com.sematext.blog.solr.ExampleRequestHandler",
            "_packageinfo_":{
              "package":"sematext-example",
              "version":"1.0.0",
              "files":["/package/sematext-example/1.0.0/solr-example-module-1.0.jar"],
              "manifest":"/package/sematext-example/1.0.0/manifest.json",
              "manifestSHA512":"da463cdad3efbe4c9159b29156bbaf26f4aa35a083a8b74fd57e1dfa1f79ee7eaadfd3863f5d88fa2550281c027e82b516ebc64a7fa4159089f32c565813c574"}}}}}
    
    Actual: 1.0.0, expected: 1.0.0
    Deployed on [test] and verified package: sematext-example, version: 1.0.0
    Deployment successful

During the execution of the above command, the **bin/solr** script will ask you if you are certain that you would like to deploy the chosen package. If you agree to that Solr will deploy and try to verify the package by using the verification command provided in the **repository.json** description file. If that went well – the plugin is ready and we can use it.

When we no longer need a package we can remove it by running the **undeploy** command. For example, if we would like to remove the previously deployed package we just need to run the following command:

    $ bin/solr package undeploy sematext-example -collections test

In this case, the response says that everything went well and we will no longer be using the package:

    Executing {"delete-requesthandler":"/sematextexample"} for path:/api/collections/test/config

How Solr Packages Work Under the Hood
-------------------------------------

The heart of the implementation of the plugin mechanism is related to the isolation of classloaders of the plugins and the core Solr classes. The plugin mechanism assumes that any change in the files that are in the Solr **classpath** requires a restart. The rest of the files can be loaded dynamically and are bound to the configuration stored in Zookeeper.

The basis of the mechanism is a so-called **Package Store**. It is a distributed file system that keeps its data on each Solr node in the **$SOLR\_HOME/filestore** directory and each of the files is described by metadata written in a JSON file. Of course, each file stores the checksum in its metadata for verification purposes. That way replacing the binary itself is not enough to load malicious versions of the plugin – the signature is still there and needs to be adjusted as well. That gives us a certain degree of security.

On top of all of that, we have an API allowing us not only to manage the whole package repository but also single files.

Solr Package API
----------------

Of course, our **bin/solr** tool and installing the packages using it is not everything that Solr gives us. In addition to that we got the API that allows us to:

*   add files using the PUT HTTP method and the **/api/cluster/files/{file\_path}** endpoint
*   retrieve files using the GET HTTP method and the **/api/cluster/files/{file\_path}** endpoint
*   retrieve file metadata using the GET HTTP method and the **/api/cluster/files/{file\_path}?meta=true**
*   retrieve files available at a given path using the GET HTTP method using the **/api/cluster/files/{directory\_path}** endpoint

You should remember that adding a file to Solr is not only about sending it to Solr. You need to sign it using a key that will be available to Solr – we saw that already.

Similar to manipulating the files in the package repository we also have the option to manage packages. We have the option to add, remove and download the packages and their versions:

*   **GET** on **/api/cluster/package** to download the list of packages
*   **PUT** on **/api/cluster/package** to add a package
*   **DELETE** on **/api/cluster/package** to remove a package

For example to add a package to Solr we could use a command like this:

    $ curl -XPUT 'http://localhost:8983/api/cluster/package' -H 'Content-type:application/json' -d  '{
     "add": {
      "package" : "sematext-example",
      "version" : "1.0.0",
      "files" : [
       "/test/sematext/1.0.0/sematext-example.jar"
      ]
     }
    }'

Security
--------

The package management functionality brings a new way of extending Solr functionality. However, you should remember that flexibility doesn’t come for free. Having the option to use **hot-deploy** and being able to install Solr extensions on the fly, without the need of bringing the whole cluster down carries limitations and security threats. Because of that remember not to add package repositories that you don’t know. Such repositories can be dangerous and can result in downloading and installing malicious code. The second thing to remember is that you shouldn’t add repositories that are not using SSL. Adding a repository that is not using SSL exposes you to the **man in the middle** attack during which the files can be replaced on the fly leading to installing the malicious code. That can result in compromising your cluster, which may lead to data leaks or the whole environment being compromised. That is something that you should remember and keep your Solr secure no matter if you use package management or not.

Summary
-------

The functionality of installing Solr extensions without the need to manually download them to each node, restarting the nodes and so on is very nice and tempting. Especially to those of us who use such extensions. However, please remember about security and the limitations of the mechanism. If we will be cautious we will have a way of extending Solr in a flexible way.


