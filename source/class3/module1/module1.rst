LAB 2A: Exploring and understanding the K8S cluster
####################################################

.. contents:: Contents
    :local:
    :depth: 1

LAB 2A: Exploring and understanding the K8S cluster
****************************************************

    .. note::
        | In order to be independant of a specific K8S distribution, standard tools will be used for managing the cluster.
        | The tool ``kubectl`` will be used during that workshop.


- Step 1: logging with SSH to your attributed K8S Cluster

.. code-block:: bash

    $ ssh -i jumphost.key cyber@jumphost-aksdistrict{{site_ID}}.{{region}}.cloudapp.azure.com


- Step 2: Let's verify the CRDs installed:

.. code-block:: bash

    $ kubectl get crds


output:

.. code-block:: bash

    NAME                                 CREATED AT
    aplogconfs.appprotect.f5.com         2021-03-08T10:00:03Z
    appolicies.appprotect.f5.com         2021-03-08T10:00:03Z
    apusersigs.appprotect.f5.com         2021-03-08T10:00:03Z
    globalconfigurations.k8s.nginx.org   2021-03-08T10:00:03Z
    policies.k8s.nginx.org               2021-03-08T10:00:03Z
    transportservers.k8s.nginx.org       2021-03-08T10:00:03Z
    virtualserverroutes.k8s.nginx.org    2021-03-08T10:00:03Z
    virtualservers.k8s.nginx.org         2021-03-08T10:00:04Z


- Step 3: Let's check the NameSpaces of the cluster:

.. code-block:: bash

        $ kubectl get ns


output:

.. code-block:: bash

        NAME                          STATUS   AGE
        arcadia                       Active   30d
        default                       Active   30d
        external-ingress-controller   Active   30d
        internal-ingress-controller   Active   30d
        kube-node-lease               Active   30d
        kube-public                   Active   30d
        kube-system                   Active   30d


- Step 4: Look at the pods in some NameSpaces with the command ``kubectl get pods``:

Namespace Default:

.. code-block:: bash

        $ kubectl get pods -n default


output:

.. code-block:: bash

        No resources found in default namespace.


Namespace external-ingress-controller:

.. code-block:: bash

        $ kubectl get pods -n external-ingress-controller


output:

.. code-block:: bash

        NAME                                               READY   STATUS    RESTARTS   AGE
        nap-external-ingress-controller-54db45d656-fg4tq   1/1     Running   0          30d



- Step 5: Let's check the Ingress Class Name attached to each Externale Ingress Controller:

.. code-block:: bash

        $ kubectl describe pod nap-external-ingress-controller-54db45d656-fg4tq -n external-ingress-controller


output:

.. code-block:: bash

        Name:         nap-external-ingress-controller-54db45d656-fg4tq
        Namespace:    external-ingress-controller
        .......
        .......
        Containers:
          external-nginx-plus-ingress-nginx-ingress:
            .......
            .......
            Ports:         80/TCP, 443/TCP, 9113/TCP, 8081/TCP
            Host Ports:    0/TCP, 0/TCP, 0/TCP, 0/TCP
            Args:
              -nginx-plus=true
              -nginx-reload-timeout=0
              -enable-app-protect=true
              .......
              .......
              -ingress-class=nginx-external        ****INGRESS CLASS NAME is nginx-external****
              .......
              .......


- Step 6: Let's check the Public IP address attached to the external Ingress Controller:

.. code-block:: bash

        $ kubectl get services -n external-ingress-controller


output:

.. code-block:: bash

        NAME                         TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)                      AGE
        elb-nap-ingress-controller   LoadBalancer   10.200.0.15   52.167.14.0   80:31613/TCP,443:31094/TCP   30d


.. note::
    | Notice the EXTERNAL-IP address and write it somewhere.
    | It will be used later in our labs.


