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

- Step 7: Let's check the NameSpaces of the cluster:

.. code-block:: bash

        harry@Azure:~$ kubectl get ns
        NAME                          STATUS   AGE
        arcadia                       Active   30d
        default                       Active   30d
        external-ingress-controller   Active   30d
        internal-ingress-controller   Active   30d
        kube-node-lease               Active   30d
        kube-public                   Active   30d
        kube-system                   Active   30d

Step 8: Look at the pods in each NameSpaces with the command ``kubectl get pods``:

.. code-block:: bash

        harry@Azure:~$ kubectl get pods -n default
        No resources found in default namespace.

.. code-block:: bash

        harry@Azure:~$ kubectl get pods -n arcadia
        NAME                       READY   STATUS    RESTARTS   AGE
        app2-6dcf6d5845-crpv6      1/1     Running   0          30d
        app2-6dcf6d5845-wdxds      1/1     Running   0          30d
        app3-b989dc6dc-6klxk       1/1     Running   0          30d
        app3-b989dc6dc-9vpfm       1/1     Running   0          30d
        backend-56c9b667d5-4x4w2   1/1     Running   0          30d
        backend-56c9b667d5-zfgvc   1/1     Running   0          30d
        main-84cf4949b9-f5x5t      1/1     Running   0          30d
        main-84cf4949b9-pnkwt      1/1     Running   0          30d

.. code-block:: bash

        harry@Azure:~$ kubectl get pods -n external-ingress-controller
        NAME                                               READY   STATUS    RESTARTS   AGE
        nap-external-ingress-controller-54db45d656-fg4tq   1/1     Running   0          30d

.. code-block:: bash

        harry@Azure:~$ kubectl get pods -n internal-ingress-controller
        NAME                                               READY   STATUS    RESTARTS   AGE
        nap-internal-ingress-controller-55fdb8cd95-2dz77   1/1     Running   0          30d


Step 9: Let's check the Ingress Class Name attached to each NIC:

.. code-block:: bash

        harry@Azure:~$ kubectl describe pod nap-external-ingress-controller-54db45d656-fg4tq -n external-ingress-controller
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


.. code-block:: bash

        harry@Azure:~/lab2$ kubectl describe pod nap-internal-ingress-controller-55fdb8cd95-2dz77 -n internal-ingress-controller
        Name:         nap-internal-ingress-controller-55fdb8cd95-2dz77
        Namespace:    internal-ingress-controller
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
              -ingress-class=nginx-internal         ****INGRESS CLASS NAME is nginx-internal****
              .......
              .......


- Step 10: Let's check the Public IP address attached to the external Ingress Controller:

.. code-block:: bash

        harry@Azure:~$ kubectl get services -n external-ingress-controller
        NAME                         TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)                      AGE
        elb-nap-ingress-controller   LoadBalancer   10.200.0.15   52.167.14.0   80:31613/TCP,443:31094/TCP   30d

.. note::
    | Notice the EXTERNAL-IP address and write it somewhere.
    | It will be used later in our labs.


