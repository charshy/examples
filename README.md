# Amalgam8 examples

Sample microservice-based applications and local sandbox environment for Amalgam8. www.amalgam8.io.

## Overview <a id="overview"></a>

This project includes 2 Amalgam8 sample apps, scripts, and a preconfigured environment to allow you to easily run, build, and experiment with the provided samples. The scripts are designed to be generic, so that you can easily deploy the samples to other environments.

The following samples are available for Amalgam8:

* [Helloworld](https://github.com/amalgam8/examples/blob/master/apps/helloworld/README.md) is a single microservice app that demonstrates how to route traffic to different versions of the same microservice.
* [Bookinfo](https://github.com/amalgam8/examples/blob/master/apps/bookinfo/README.md) is a multiple microservice app used to demonstrate and experiment with several Amalgam8 features, including routing and resilience testing.

The repository's root directory includes a Vagrant file that provides an environment with everything needed to run and build the samples:
([Go](http://golang.org/), [Docker](http://www.docker.com/), [Kubernetes](http://kubernetes.io/),
[Amalgam8 CLI](https://github.com/amalgam8/controller/tree/master/cli), [Gremlin SDK](https://github.com/ResilienceTesting/gremlinsdk-python))
Depending on the runtime environment you want to try, using the Vagrant file might be the easiest way to get Amalgam8 up and running.

## Running the samples <a id="running"</a>

To run the sample apps, follow the steps that correspond to the environment that you want to use:

* Localhost Deployment
    * [Docker](#local-docker)
    * [Kubernetes](#local-k8s)
    * [Marathon/Mesos](#local-marathon)
* Cloud Deployment
    * [IBM Bluemix](#bluemix)
    * [Google Compute Cloud](#gcp)

If you'd like to be able to change and compile the code, or build the images, refer to the [Developer Instructions](https://github.com/amalgam8/examples/blob/master/development.md).

## Amalgam8 with Docker - local environment <a id="local-docker"></a>

To run in a local Docker environment, you can either use the Vagrant sandbox, or install the
[Amalgam8 python CLI](https://pypi.python.org/pypi/a8ctl),
[Docker 1.10 or later](https://docs.docker.com/engine/installation/) and
[Docker Compose 1.5.1 or later](https://docs.docker.com/compose/install/).

The installation steps below were tested with the Vagrant sandbox
environment (based on Ubuntu 14.04) as well as with Docker for Mac Beta
(v1.11.2-beta15 or later versions). These steps are not tested on Docker for
Windows Beta.

The following instructions assume that you are using the Vagrant
environment. Where appropriate, environment-specific instructions are
provided.

1. Clone the Amalgam8 examples repo and then start the Vagrant environment:

    ```bash
    git clone git@github.com:amalgam8/examples.git

    cd examples
    vagrant up
    vagrant ssh

    cd $GOPATH/src/github.com/amalgam8/examples
    ```

   If you are not using the Vagrant file, then you must download and install the following technologies for your environment:

  * [Go](http://golang.org/),
  * [Docker](http://www.docker.com/)
  * [Kubernetes](http://kubernetes.io/)
  * [Amalgam8 CLI](https://github.com/amalgam8/controller/tree/master/cli)
  * [Gremlin SDK](https://github.com/ResilienceTesting/gremlinsdk-python)

1. Start the control plane services (registry and controller) by running the
    following command:

    ```bash
    docker/run-controlplane-docker.sh start
    ```

    The above command also creates a tenant named "local" in the
    control plane.

1. Set the `A8_CONTROLLER_URL` and `A8_REGISTRY_URL` environment variables to point to the addresses of the controller and registry.

        * If you are running Docker by using the Vagrant file in the
        `examples` folder, or if you are running Docker locally (on Linux or
        Docker for Mac Beta), run the following commands:

        ```bash
        export A8_CONTROLLER_URL=http://localhost:31200
        export A8_REGISTRY_URL=http://localhost:31300
        ```

        * If you are running Docker by using the Docker Toolbox with Docker Machine,
        then set the environment variables to the IP address of the VM that is created
        by Docker Machine. For example, with only one Docker
        Machine running on your system, run the following commands, replacing `docker-machine ip` with the IP address of the VM:

    ```bash
    export A8_CONTROLLER_URL=`docker-machine ip`
    export A8_REGISTRY_URL=`docker-machine ip`
    ```

1.  Confirm that the control plane services are working correctly by running the following command:

    ```bash
    a8ctl service-list
    ```

    If the control plane services and Amalgam8 CLI are working as expected, the following empty table is returned as output.

    ```
    +---------+-----------+
    | Service | Instances |
    +---------+-----------+
    +---------+-----------+
    ```

    Note that no services are listed, because we have not yet started any.

1. Start the API gateway by running the following command:

    ```bash
    docker-compose -f docker/gateway.yaml up -d
    ```

    Usually, the API gateway is mapped to a DNS route. However, in this local
    standalone environment, you can access it at http://localhost:32000.

1. Confirm that the API gateway is running by accessing
    ```http://localhost:32000``` from your browser.

    If the API gateway is running correctly, the **Welcome to nginx!**
    page opens in your browser.

1. Follow the instructions for the sample that you want to run.

    (a) **helloworld** sample

    * Start the helloworld application:

    ```bash
    docker-compose -f docker/helloworld.yaml up -d
    docker-compose -f docker/helloworld.yaml scale helloworld-v1=2
    docker-compose -f docker/helloworld.yaml scale helloworld-v2=2
    ```

    * Follow the instructions at https://github.com/amalgam8/examples/blob/master/apps/helloworld/README.md

    * To shutdown the helloworld instances, run the following commands:

    ```bash
    docker-compose -f docker/helloworld.yaml kill
    docker-compose -f docker/helloworld.yaml rm -f
    ```

    (b) **bookinfo** sample

    * Start the bookinfo application:

    ```bash
    docker-compose -f docker/bookinfo.yaml up -d
    ```

    * Follow the instructions at https://github.com/amalgam8/examples/blob/master/apps/bookinfo/README.md

    * To shutdown the bookinfo instances, run the following commands:

    ```
    docker-compose -f docker/bookinfo.yaml kill
    docker-compose -f docker/bookinfo.yaml rm -f
    ```

    When you are finished, shut down the API gateway and control plane servers by running the following command:

    ```
    docker/cleanup.sh
    ```

## Amalgam8 with Kubernetes - local environment <a id="local-k8s"></a>

The following installation steps were tested with Kubernetes v1.2.3.

1. Clone the Amalgam8 examples repo and then start the Vagrant environment:

    ```bash
    git clone git@github.com:amalgam8/examples.git

    cd examples
    vagrant up
    vagrant ssh

    cd $GOPATH/src/github.com/amalgam8/examples
    export A8_CONTROLLER_URL=http://localhost:31200
    export A8_REGISTRY_URL=http://localhost:31300
    ```

1. Install Kubernetes by running the following command:

    ```bash
    sudo kubernetes/install-kubernetes.sh
    ```

    If you stopped a previous Vagrant VM and restarted it, Kubernetes might be started already, but in a bad state.
    If you have problems, uninstall Kubernetes and repeat the install. You can uninstall Kubernetes by using the following command:

    ```bash
    sudo kubernetes/uninstall-kubernetes.sh
    ```

1. Start the local control plane services (registry and controller) by running the following commands:

    ```bash
    kubernetes/run-controlplane-local-k8s.sh start
    ```

1.  Confirm the control plane services are working correctly by running the following command:

    ```bash
    a8ctl service-list
    ```

    If the control plane services and Amalgam8 CLI are working as expected, the following empty table is returned as output.

    ```
    +---------+-----------+
    | Service | Instances |
    +---------+-----------+
    +---------+-----------+
    ```

    Note that no services are listed, because we have not yet started any.

    If the command doesn't work, it's usually because the image download and/or service initialization took too long.
    You can fix this by waiting a minute or two, then run ```kubernetes/run-controlplane-local-k8s.sh stop```, and then
    repeat the previous step.

    You can also access the registry at http://localhost:31300 from the host machine
    (outside the Vagrant box), and the controller at http://localhost:31200.
    To access the control plane details of tenant *local*, access
    http://localhost:31200/v1/tenants/local from your browser.

1. Run the API Gateway with the following commands:

    ```bash
    kubectl create -f kubernetes/gateway.yaml
    ```

    Usually, the API gateway is mapped to a DNS route. However, in our local
    standalone environment, you can access it at port 32000 on localhost.

1. Confirm that the API gateway is running by accessing
    http://localhost:32000 from your browser.

    If the API gateway is running correctly, the **Welcome to nginx!**
    page opens in your browser.

1. View a graphical representation of your deployment in [Weave Scope](https://github.com/weaveworks/scope) by accessing
   http://localhost:30040, and click on `Pods` tab. You should see a graph of
   pods depicting the connectivity between them. As you create more apps
   and manipulate routing across microservices, the graph changes in
   real-time.

1. Follow the instructions for the sample that you want to run.

    (a) **helloworld** sample

    * Start the helloworld application:

        ```bash
        kubectl create -f kubernetes/helloworld.yaml
        ```

    * Follow the instructions at https://github.com/amalgam8/examples/blob/master/apps/helloworld/README.md

    * To shutdown the helloworld instances, run the following command:

        ```bash
        kubectl delete -f kubernetes/helloworld.yaml
        ```

    (b) **bookinfo** sample

    * Start the bookinfo application:

        ```bash
        kubectl create -f kubernetes/bookinfo.yaml
        ```

    * Follow the instructions at https://github.com/amalgam8/examples/blob/master/apps/bookinfo/README.md

    * To shutdown the bookinfo instances, run the following command:

        ```bash
        kubectl delete -f kubernetes/bookinfo.yaml
        ```

1. When you are finished, shut down the gateway and control plane servers by running the following commands:

    ```bash
    kubernetes/cleanup.sh
    ```

## Amalgam8 with Marathon/Mesos - local environment <a id="local-marathon"></a>

The following setup was tested with Marathon 0.15.2 and Mesos 0.26.0.

1. Clone the Amalgam8 examples repo:
    ```bash
    git clone git@github.com:amalgam8/examples.git

    cd examples
    ```

1. Edit the Vagrant file in the *examples* folder. Uncomment the line
that begins with `config.vm.network "private_network", ip: "192.168.33.33/24"`.

1. Start the Vagrant environment:

    ```bash
    vagrant up
    vagrant ssh

    cd $GOPATH/src/github.com/amalgam8/examples
    export A8_CONTROLLER_URL=http://192.168.33.33:31200
    export A8_REGISTRY_URL=http://192.168.33.33:31300
    ```

1. The `run-controlplane-marathon.sh` script in the *marathon* folder sets up a
   single host (local) marathon/mesos cluster (based on Holiday Check's
   [mesos-in-the-box](https://github.com/holidaycheck/mesos-in-the-box)), and launches the Amalgam8 control plane (controller and the
   registry) as apps in the marathon framework.

    ```bash
    marathon/run-controlplane-marathon.sh start
    ```

2. From your browser, confirm that the Marathon dashboard is accessible at http://192.168.33.33:8080 and the Mesos dashboard at          http://192.168.33.33:5050

    Verify that the controller and registry are running by using the Marathon dashboard.

1. Launch the API Gateway

    ```bash
    marathon/run-component.sh gateway start
    ```
    Usually, the API gateway is mapped to a DNS route. However, in our local
    standalone environment, you can access it at port 32000 on localhost.

1. Confirm that the API gateway is running by accessing
    http://localhost:32000 from your browser.

    If the API gateway is running correctly, the **Welcome to nginx!**
    page opens in your browser.

1. Follow the instructions for the sample that you want to run.

    (a) **helloworld** sample

    * Start the helloworld application:

    ```bash
    marathon/run-component.sh helloworld start
    ```

    * Follow the instructions at https://github.com/amalgam8/examples/blob/master/apps/helloworld/README.md

    * To shutdown the helloworld instances, run the following commands:

    ```bash
    marathon/run-component.sh helloworld stop
    ```

    (b) **bookinfo** sample

    * Start the bookinfo application:

    ```bash
    marathon/run-component.sh bookinfo start
    ```

    * Follow the instructions at https://github.com/amalgam8/examples/blob/master/apps/bookinfo/README.md

    * To shutdown the bookinfo instances, run the following commands:

    ```bash
    marathon/run-component.sh bookinfo stop
    ```

    When you are finished, shut down the gateway and control plane servers by running the following commands:

    ```bash
    marathon/cleanup.sh
    ```

## Amalgam8 on IBM Bluemix <a id="bluemix"></a>

To run the [Bookinfo sample app](https://github.com/amalgam8/examples/blob/master/apps/bookinfo/README.md)
on Bluemix, follow the steps below.

If you are not a bluemix user, register a user account at [bluemix.net](http://bluemix.net/).

1. Download [Docker 1.10 or later](https://docs.docker.com/engine/installation/),
    [CF CLI 6.12.0 or later](https://github.com/cloudfoundry/cli/releases),
    [CF CLI IBM Containers plugin](https://console.ng.bluemix.net/docs/containers/container_cli_ov.html),
    [jq 1.5 or later](https://stedolan.github.io/jq/),
    and the [Amalgam8 CLI](https://pypi.python.org/pypi/a8ctl)

1. Log in to Bluemix and initialize the containers environment by using the CF CLI commands ```cf login``` and ```cf ic init```.

1. Create Bluemix routes to be mapped to the controller/bookinfo gateway. For example:  
    ```cf create-route myspace mybluemix.net -n myamalgam8-controller```  
    ```cf create-route myspace mybluemix.net -n myamalgam8-bookinfo```

1. Configure the [.bluemixrc file](bluemix/.bluemixrc) to your environment variable values:
    * BLUEMIX_REGISTRY_NAMESPACE to your Bluemix registry namespace. Run ```cf ic namespace get``` for the namespace value.
    * BLUEMIX_REGISTRY_HOST to the Bluemix registry hostname. Set this if you're targeting a Bluemix region that is not US-South.
    * CONTROLLER_HOSTNAME to the (globally unique) Bluemix route to be mapped to the controller.
    * BOOKINFO_ROUTE to the (globally unique) Bluemix route to be mapped to the bookinfo gateway.
    * ROUTES_DOMAIN to the domain that is used for the Bluemix routes (for example, mybluemix.net).
    * ENABLE_SERVICEDISCOVERY specifies whether to use the Bluemix-provided [IBM Service Discovery](https://console.ng.bluemix.net/docs/services/ServiceDiscovery/index.html)
      registry instead of the Amalgam8 registry. When set to *false*, you can deploy your own customized Amalgam8 registry (not yet implemented).
    * ENABLE_MESSAGEHUB determines whether to use the Bluemix-provided [IBM Message Hub](https://console.ng.bluemix.net/docs/services/MessageHub/index.html#messagehub).
      When set to false, the Amalgam8 proxies use a slower polling algorithm to get changes from the Amalgam8 Controller.  
      Note that the Message Hub service is not a free service, and using it might incur costs.

1. Deploy the Amalgam8 control plane by running [bluemix/deploy-controlplane.sh](bluemix/deploy-controlplane.sh).

1. Verify that the controller is running by running the CF  ```cf ic group list```, and checking if the ```amalgam8_controller``` group is running.

1. Configure the Amalgam8 CLI according to the routes that are defined in
   [.bluemixrc file](bluemix/.bluemixrc). For example:

    ```
    export A8_CONTROLLER_URL=https://mya8controller.mybluemix.net
    ```


 1.  Confirm the control plane services are working correctly by running the following command:

    ```bash
    a8ctl service-list
    ```

    If the control plane services and Amalgam8 CLI are working as expected, the following empty table is returned as output.

    ```
    +---------+-----------+
    | Service | Instances |
    +---------+-----------+
    +---------+-----------+
    ```

   Note that no services are listed, because we have not yet started any.

1. Deploy the API Gateway and the Bookinfo app by running [bluemix/deploy-bookinfo.sh](bluemix/deploy-bookinfo.sh).


1. Confirm the microservices are running:

    ```
    a8ctl service-list
    ```

    If the microservices are running, the output displays as the following table:

    ```
    +-------------+---------------------+
    | Service     | Instances           |
    +-------------+---------------------+
    | productpage | v1(1)               |
    | ratings     | v1(1)               |
    | details     | v1(1)               |
    | reviews     | v1(1), v2(1), v3(1) |
    +-------------+---------------------+
     ```

1. Follow the [Bookinfo sample app](https://github.com/amalgam8/examples/blob/master/apps/bookinfo/README.md) instructions for the rest of the demo.

    When you reach the part where it instructs you to open, in your browser, the bookinfo application at
    http://localhost:32000/productpage/productpage, make sure to change "http://localhost:32000" to the value of the `BOOKINFO_URL`
    environment variable that you defined in your `.bluemixrc` file.

    **Note:** The Bluemix version of the bookinfo sample app does not yet support running the Gremlin recipe.
    We are working on integrating the app with the Bluemix Logmet services, to enable support for running Gremlin recipes.

1. When you are finished, shut down the gateway and control plane servers by running the following commands:
    ```bash
    ./kill-bookinfo.sh
    ./kill-controlplane.sh
    ```

## Amalgam8 on Google Cloud Platform <a id="gcp"></a>

1. Set up [Google Cloud SDK](https://cloud.google.com/sdk/) on your machine.

1. Set up a cluster of 3 nodes.

1. Launch the control plane services:

    ```bash
    kubernetes/run-controlplane-gcp.sh start
    ```

1. Locate the node where the controller is running, and assign an
   external IP to the node if required.

1. Initialize the first tenant. The `run-controlplane-gcp.sh` script stores
   the JSON payload to initialize the tenant in `/tmp/tenant_details.json`.

    ```bash
    cat /tmp/tenant_details.json|curl -H "Content-Type: application/json" -d @- http://ControllerExternalIP:31200/v1/tenants
    ```

1. Deploy the API gateway:

    ```bash
    kubectl create -f kubernetes/gateway.yaml
    ```

    Obtain the public IP of the node where the gateway is running. This will be
    the be IP at which the sample app will be accessible.

1. You can now deploy the sample apps as described in "Running the sample
    apps" section above. Remember to replace the IP address `localhost`
    with the public IP address of the node where the gateway service is
    running on the Google Cloud Platform.

1. View a graphical representation of your deployment in [Weave Scope](https://github.com/weaveworks/scope):

    ```bash
    kubectl create -f 'https://scope.weave.works/launch/k8s/weavescope.yaml' --validate=false
    ```

    When weavescope is running, you can view the weavescope dashboard
    on your local host by using the following commands:

    ```bash
    kubectl port-forward $(kubectl get pod --selector=weavescope-component=weavescope-app -o jsonpath={.items..metadata.name}) 4040
    ```

    You can open http://localhost:4040 on your browser to access the Scope UI.
