# Right-sizing your Kubernetes Infrastructure: Balancing Performance and Cost

You know what is Goldilocks? If you don't know you is in the right place for learn what is it, how you van resizing your infrastructura, otimize your coast and pay less money for the AWS rsrs.

## What is Goldilocks

> Goldilocks is a utility that can help you identify a starting point for resource requests and limits. 

If we visited the Goldilocks website we founded informations about Goldilocks, but there are more benefits about this tools. Imagine that we provisioned bad one application, setting more resources that the application really uses on the daily operations, or imagine that: 100 microservices with misconfigurations about cpu and memory resources this causes many problems

- Over-provisioning
- Under-provisioning
- Incresing more coasts, reminder in Cloud we pay for the use or what we should be use.

Without Goldilocks we deployed our applications and don't have a manner that see for the application and think 'Ok, this application is otimized'

## How Goldilocks and VPA Work Together

## Instalation Guide

Helm Repositories on Artifact Hub

- https://artifacthub.io/packages/helm/fairwinds-stable/vpa

- https://artifacthub.io/packages/helm/fairwinds-stable/goldilocks

### Configuration VPA

For the instalations of the VPA I will only set up the recommendrer [values.yaml](./vpa/values.yaml)

Set

```bash
helm install vpa . -f values.yaml --namespace goldilocks
```

```text
NAME: vpa
LAST DEPLOYED: Sat Apr 25 14:59:25 2026
NAMESPACE: goldilocks
STATUS: deployed
REVISION: 1
DESCRIPTION: Install complete
NOTES:
Congratulations on installing the Vertical Pod Autoscaler!

Components Installed:
  - recommender

To verify functionality, you can try running 'helm -n goldilocks test vpa'
```

### Configurarion Goldilocks

[values.yaml](./goldilocks/values.yaml)

For to install goldilocks we use this command

```bash
helm install goldilocks . -f values.yaml --namespace goldilocks
```

and then

```txt
NAME: goldilocks
LAST DEPLOYED: Sat Apr 25 15:02:58 2026
NAMESPACE: goldilocks
STATUS: deployed
REVISION: 1
DESCRIPTION: Install complete
TEST SUITE: None
NOTES:
1. Get the application URL by running these commands:
  kubectl -n goldilocks port-forward svc/goldilocks-dashboard 8080:80
  echo "Visit http://127.0.0.1:8080 to use your application"
```

Now we can visit the dashboard using the port-forward command, like this

```bash
kubectl -n goldilocks port-forward svc/goldilocks-dashboard 8080:80
```

![Goldilocks](.github/images/dashboard-goldilocks.png)

Ok, now we need to set the label on the select namespaces that we can monitoring the resources uses. For this we use the command

```bash
for ns in foo goldilocks ; do kubectl label ns $ns goldilocks.fairwinds.com/enabled=true; done
```

Now we can see that two namespaces


![Goldilocks Namespaces](.github/images/goldilocks-namespaces.png)

![Goldilocks Namespace Details](.github/images/goldilocks-namespace-details.png)

Now, we analise the Burtable QoS values 

![Goldilocks Bustable Values](.github/images/goldilocks-burtable-values.png)

Goldilocks now surger that we can resize own application for 
```yaml
resources:
  requests:
    cpu: 15m
    memory: 100Mi
  limits:
    cpu: 10649m
    memory: 5325M
```

Note, quanto more time do you deixar goldilocks and vpa analize your applciation, more can better he can set the better values for resources

## Logs

Checking the logs of kubectl logs of goldilocks controller pod we can saw that thera are two vpa now, one for each namespace

```bash
kubectl logs goldilocks-controller-859978bf88-gkrdq -ngoldilocks -f

I0425 18:15:09.288442       1 vpa.go:278] There are 0 vpas in Namespace/foo
I0425 18:15:09.288503       1 vpa.go:103] Namespace/foo is not managed, cleaning up VPAs if they exist...
I0425 18:15:09.300126       1 vpa.go:278] There are 0 vpas in Namespace/foo
I0425 18:15:09.300222       1 vpa.go:103] Namespace/foo is not managed, cleaning up VPAs if they exist...
I0425 18:15:27.011214       1 namespace.go:39] Namespace goldilocks updated. Check the labels.
I0425 18:15:27.019423       1 vpa.go:278] There are 0 vpas in Namespace/goldilocks
I0425 18:15:27.193996       1 vpa.go:191] Reconciling Namespace/goldilocks for Deployment/goldilocks-dashboard with VPA/none
I0425 18:15:27.212971       1 vpa.go:311] Created VPA/goldilocks-goldilocks-dashboard in Namespace/goldilocks
I0425 18:15:27.213019       1 vpa.go:191] Reconciling Namespace/goldilocks for Deployment/vpa-recommender with VPA/none
I0425 18:15:27.223302       1 vpa.go:311] Created VPA/goldilocks-vpa-recommender in Namespace/goldilocks
I0425 18:15:27.223367       1 vpa.go:191] Reconciling Namespace/goldilocks for Deployment/goldilocks-controller with VPA/none
I0425 18:15:27.229429       1 vpa.go:311] Created VPA/goldilocks-goldilocks-controller in Namespace/goldilocks
I0425 18:15:50.493599       1 namespace.go:39] Namespace foo updated. Check the labels.
I0425 18:15:50.507160       1 vpa.go:278] There are 0 vpas in Namespace/foo
I0425 18:15:50.644718       1 vpa.go:191] Reconciling Namespace/foo for Deployment/foo-nginx with VPA/none
I0425 18:15:50.660182       1 vpa.go:311] Created VPA/goldilocks-foo-nginx in Namespace/foo
```

Analizing the logs of the vpa

I0425 18:17:27.619970       1 recommender.go:176] "Recommender Run"
I0425 18:17:27.620061       1 cluster_feeder.go:388] "Start selecting the vpaCRDs."
I0425 18:17:27.620090       1 cluster_feeder.go:429] "Fetching VPAs" count=4
I0425 18:17:27.620269       1 cluster_feeder.go:439] "Using selector" selector="app.kubernetes.io/component=dashboard,app.kubernetes.io/instance=goldilocks,app.kubernetes.io/name=goldilocks" vpa="goldilocks/goldilocks-goldilocks-dashboard"
I0425 18:17:27.620393       1 cluster_feeder.go:439] "Using selector" selector="app.kubernetes.io/component=recommender,app.kubernetes.io/instance=vpa,app.kubernetes.io/name=vpa" vpa="goldilocks/goldilocks-vpa-recommender"
I0425 18:17:27.620446       1 cluster_feeder.go:439] "Using selector" selector="app.kubernetes.io/component=controller,app.kubernetes.io/instance=goldilocks,app.kubernetes.io/name=goldilocks" vpa="goldilocks/goldilocks-goldilocks-controller"
I0425 18:17:27.620474       1 cluster_feeder.go:439] "Using selector" selector="run=nginx" vpa="foo/goldilocks-foo-nginx"

note, thera are four VPAs, one for each deployment. 

- 3 deployments on goldilocks
- 1 deployment on foo

paulo-fabiano@spark:~/goldilocks$ kubectl get deploy -nfoo
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
foo-nginx   10/10   10           10          16m
paulo-fabiano@spark:~/goldilocks$ kubectl get deploy -ngoldilocks
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
goldilocks-controller   1/1     1            1           27m
goldilocks-dashboard    2/2     2            2           27m
vpa-recommender         1/1     1            1           31m


## A few moments later

If we see the recommendations now, I can to see

![Goldilocks Few Moments Later](.github/images/goldilocks-few-moments-later.png)

How many time do you stay the godoicloksd monitoring your applicatiom, better is the recommendation

## Integration

Ok, all of this is good, but if we need to acess every the dashboard this isn't good, how we can apply DevOps practises such as automation

If we get all VPAs

kubectl get vpa -A
NAMESPACE    NAME                               MODE   CPU   MEM     PROVIDED   AGE
foo          goldilocks-foo-nginx               Off    15m   100Mi   True       40m
goldilocks   goldilocks-goldilocks-controller   Off    15m   100Mi   True       40m
goldilocks   goldilocks-goldilocks-dashboard    Off    15m   100Mi   True       40m
goldilocks   goldilocks-vpa-recommender         Off    15m   100Mi   True       40m

We can get the values of containerRecommendations in each vpa and send this values for one pipeline on Jenkins, GitHub actions

kubectl get vpa goldilocks-foo-nginx -nfoo -ojson | jq '.status.recommendation'
{
  "containerRecommendations": [
    {
      "containerName": "nginx",
      "lowerBound": {
        "cpu": "15m",
        "memory": "100Mi"
      },
      "target": {
        "cpu": "15m",
        "memory": "100Mi"
      },
      "uncappedTarget": {
        "cpu": "15m",
        "memory": "100Mi"
      },
      "upperBound": {
        "cpu": "405m",
        "memory": "423953300"
      }
    }
  ]
}

2. Integração com CI/CD (Pipeline de Auditoria)
Você pode criar um step no seu pipeline (GitHub Actions, Jenkins, ou GitLab CI) que:

Executa um script Python ou Bash.

Compara os resources atuais no seu Helm Chart/Manifesto com as recomendações do VPA/Goldilocks.

Se a diferença for maior que 30%, o pipeline emite um Warning ou até falha, forçando o desenvolvedor a revisar os custos.