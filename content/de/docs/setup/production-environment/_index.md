---
title: "Produktionsumgebung"
description: Erstellen eines produktionsreifen Kubernetes Clusters 
weight: 30
no_list: true
---

<!-- overview -->

Ein produktionsreifer Kubernetes Cluster benötigt Planung und
Vorbereitung.
Sofern Ihr Kubernetes Cluster kritische Workloads verrichtet, muss er ausfallsicher konfiguriert sein.
Diese Seite zeigt Ihnen Schritte, um einen produnktionsreifen Cluster aufzusetzen 
oder um einen bestehenen Cluster produktionsreif zu machen.
Wenn Sie bereits mit dem Setup für eine Produktionsumgebung vertraut sind, können
Sie direkt zu [Nächste Schritte](#nächste-schritte) springen.

<!-- body -->

## Production considerations

Ein Kubernetes Cluster in einer Produktionsumgebung hat üblicherweise höhere
Anforderungen als ein eigener Cluster zum Lernen, oder Cluster in einer
Entwicklungsumbebung oder einer Testumgebung. Eine Produktionsumgebung benötigt 
möglicherweise sicheren Zugriff bei vielen Nutzern, konstante Verfügbarkeit, 
und die Ressourcen um sich ändernden Anforderungen gerecht zu werden. 

Da Sie entscheiden wie Sie Ihre Kubernetes Umgebung hosten (On-Premise oder in
der Cloud), und wie viel Aufwand Sie durch die eigene Verwaltung aufbringen
möchten, oder diese Dritten überlassen, beachten Sie wie Ihre Anforderungen an
den Kubernetes Cluster durch folgende Punkte beeinflusst werden:

- *Verfügbarkeit*: Kubernetes auf einem einzigen Host für eine
[Lernumgebung](/docs/setup/#learning-environment) hat einen einzelnen
Ausfallpunkt. Für einen Cluster mit hoher Verfügbarkeit muss in Betracht gezogen
werden:
  - Die Control-Plane wird getrennt von den Worker-Nodes gehosted.
  - Replizierung der Control-Plane hat auf mehreren Nodes zu erfolgen.
  - Lastverteilung des Traffics zum {{< glossary_tooltip term_id="kube-apiserver" text="API server" >}} des Clusters.
  - Genügend Worker-Nodes sind verfügbar, oder können schnell bereitgestellt werden, wenn der Bedarf sich ändert.

- *Skalierbarkeit*: Wenn Sie eine gleichbleibende Last auf Ihrem Cluster
erwarten, können Sie die Ressourcen für ausreichende Kapazität aufsetzen und
sind fertig. Falls Sie jedoch erwarten, dass der Bedarf an Ressourcen über die
Zeit zunimmt, oder sich je nach Saison, oder durch bestimmte Ereignisse
verändert, benötigen Sie das Vorausplanen, wie Sie skalieren. Das ist notwendig, um die Control-Plane 
und Worker-Nodes zu entlasten, oder zum Runterskalieren von nicht benötigten Ressourcen.

- *Sicherheit und Zugriffskontrolle*: Sie haben Vollzugriff auf Ihren eigenen
  Cluster in einer Lernumgebung. Kubernetes Cluster mit mehr als ein, zwei Nutzern
  und kritischen Workloads benötigen jedoch eine granulare Verwaltung "Wer" und "Was" Zugriff
  auf Ressourcen in Ihrem Cluster hat. Dafür stehen die role-based access control
  ([RBAC](/docs/reference/access-authn-authz/rbac/)) und weitere
  Sicherheitsmechanismen zur Verfügung. Diese können Sie benutzen, um
  sicher zu stellen, dass Nutzer und Workloads Zugriff auf die Ressourcen haben,
  die Sie benötigen. Gleichzeitig halten Sie damit ihre Workloads und Ihren
  Cluster selbst sicher. Sie können mit [policies](/docs/concepts/policy/) und 
  [container resources](/docs/concepts/configuration/manage-resources-containers/)
  die Einschränkungen für Ressourcen, auf die Nutzer zugreifen können, festlegen.

Bevor Sie Ihren eigenen Cluster in einer Produktionsumgebung aufsetzen, überlegen
Sie einen Teil, oder alle damit verbundenen Aufgaben, von 
[Turnkey Cloud Solutions](/docs/setup/production-environment/turnkey-solutions/) 
oder anderen [Kubernetes Partnern](/partners/) übernehmen zu lassen.
Mögliche Optionen sind unter Anderem:

- *Serverless*: Lassen Sie Ihre Worklodas auf vollständig verwalteter Hardware
  laufen. Ressourcen wie CPU Nutzung, Speichernutzung und Festplattenzugriffe
  werden berechnet. Dabei müssen Sie keinerlei Clusterverwaltung betreiben.
- *Vollständig verwaltete Control-Plane*: Der Provider übernimmt die Skalierung
  und Verfügbarkeit der Control-Plane, sowie deren Updates und Upgrades.
- *Vollständig verwaltete Worker-Nodes*: Sie Konfigurieren Ihren Bedarf an
  Worker-Nodes. Der Provider übernimmt die Bereitstellung und Verfügbarkeit dieser.
// TODO - and implement upgrades when needed - Are we talking about K8s Upgrades
// or Node Pool Upgrades, e.g. provisioning more nodes
- *Integriert*: Manche Anbieter integrieren Kubernetes mit anderen Diensten, die
  Sie möglicherweise benötigen. Darunter fallen zum Beispiel Speicher, Container 
  Registries, Authentifizierungsmethoden oder Entwicklungswerkzeuge.

Ungeachtet dessen, ob Sie einen produnktionsreifen Kubernetes Cluster selbst aufsetzen 
oder mithilfe eines Partneranbieters, lesen Sie die nächsten Abschnitte
um die Anforderungen an die Control-Plane, Worker-Nodes und die Zugriffsverwaltung
in Ihrem Cluster zu bestimmen. 

## Cluster Setup in einer Produktionsumgebung

In a production-quality Kubernetes cluster, the control plane manages the
cluster from services that can be spread across multiple computers
in different ways. Each worker node, however, represents a single entity that
is configured to run Kubernetes pods.

In einem produktionsreifen Kubernetes Cluster übernimmt die Control-Plane die
Verwaltung des Clusters mithilfe von Diensten, welche auf verschiedene Weise auf
unterschiedlichen Computern verteilt sein könne. Dabei stellen Worker-Nodes
jedoch Einheiten dar, welche konfiguriert sind Kubernetes Pods laufen zu lassen.

### Control-Plane in einer Produktionsumgebung

Der minimalste Kubernetes Cluster hat die gesamten Control-Plane Dienste und
Worker-Node Dienste auf derselben Maschine laufen. Dieses Setup können Sie
erweitern, indem Sie Worker-Nodes hinzufügen, wie in dem Diagramm
[Kubernetes Components](/docs/concepts/overview/components/) dargestellt. Falls
der Cluster nur für einen kurzen Zeitraum benötigt wird, oder verworfen werden
kann, falls etwas schief geht, kann dies das richtige Setup für Sie sein.

If you need a more permanent, highly available cluster, however, you should
consider ways of extending the control plane. By design, one-machine control
plane services running on a single machine are not highly available.
If keeping the cluster up and running
and ensuring that it can be repaired if something goes wrong is important,
consider these steps:

Benötigen Sie jedoch einen dauerhaften, hoch verfügbaren Cluster, sollten Sie
die Möglichkeiten die Control-Plane zu erweitern in Betracht ziehen. Die
Control-Plane Dienste auf einer einzigen Maschine zu hosten ist systembedingt
nicht hochverfügbar. Falls den Cluster verfügbar zu halten und sicherzustellen,
dass Fehler automatisch repariert werden wichtig ist, beachten Sie die
nachfolgenden Schritte:

- *Wählen Sie Deployment Programme*: Sie können die Control-Plane mit Werkzeugen
  wie kubeadm, kops und kubespray aufsetzen. Sehen Sie in [Kubernetes mit
  Deployment Programmen installieren](/docs/setup/production-environment/tools/)
  nach, um Tipps für produktionsreife Deployments unter Benutzung dieser Programme
  zu erhalten. Verschiedene [Container Runtimes](/docs/setup/production-environment/container-runtimes/) 
  stehen Ihnen dabei zur Auswahl.
- *Zertifikatverwaltung*: Die sichere Kommunikation ziwschen Diensten der
  Control-Plane wird durch Zertifikate garantiert. Diese werden automatisch
  während des Deployments generiert, oder durch Ihre eigene Zertifizierungsstelle.
  Siehe [PKI certificates and requirements](/docs/setup/best-practices/certificates/) für weitere Informationen.
- *Loadbalancer für den API-Server*: Konfigureiren Sie einen Loadbalancer, um
  externe API requests zu den apiserver Diensten auf unterschiedliche Nodes zu
  verteilen. Siehe [Create an External Load Balancer](/docs/tasks/access-application-cluster/create-external-load-balancer/).
- *Separierung von Backup- und etcd Dienst*: Der etcd Dienst kann entweder auf
  den gleichen Maschinen wie die anderen Control-Plane Dienste laufen, oder auf
  davon verschiedenen Maschinen, um höhere Sicherheit und Verfügbarkeit zu
  erzielen. Da der etcd Dienst Cluster Konfigurationsdaten speichert, sollten Sie
  die ectd Datenbank regelmäßig sichern, um diese bei Bedarf reparieren zu
  könnnen. Schauen Sie im [etcd FAQ](https://etcd.io/docs/v3.5/faq/) für Details
  zur Konfigurierung von etcd nach. In 
  [Betreiben von etcd Clustern für Kubernetes](/docs/tasks/administer-cluster/configure-upgrade-etcd/)
  und [Aufsetzen eines hochverfügbaren ectd Clusters mit kubeadm](/docs/setup/production-environment/tools/kubeadm/setup-ha-etcd-with-kubeadm/) finden Sie ebenfalls weitere Informationen.
- *Erstellen mehrere Control-Plane Systeme*: Um hohe Verfügbarkeit zu erzielen,
  sollte die Control-Plane auf mehr als einer einzelnen Maschine laufen. Falls
  die Control-Plane Dienste durch einen Initialisierungsdienst (wie systemd)
  laufen, sollte jeder Dienst auf mindestens drei Maschinen laufen. Alternativ kann
  das Laufenlassen der Control-Plane Dienste als Kubernetes Pods sicherstellen, dass
  Ihre angefragte Anzahl der Replikas stets verfügbar ist.
  Der Scheduler sollte fehlertolerant sein, muss jedoch nicht hochverfügbar
  sein. Manche Deployment Tools setzen den [Raft](https://raft.github.io/)
  Konsensalgorithmus auf, für die Wahl des Leaders für Kubernetes Dienste. Falls der
  gewählte Leader verschwindet, wird ein anderer Dienst gewählt und übernimmt.


- *Span multiple zones*: If keeping your cluster available at all times is
  critical, consider creating a cluster that runs across multiple data centers,
  referred to as zones in cloud environments. Groups of zones are referred to as regions.
  By spreading a cluster across
  multiple zones in the same region, it can improve the chances that your
  cluster will continue to function even if one zone becomes unavailable.
  See [Running in multiple zones](/docs/setup/best-practices/multiple-zones/) for details.
- *Manage on-going features*: If you plan to keep your cluster over time,
  there are tasks you need to do to maintain its health and security. For example,
  if you installed with kubeadm, there are instructions to help you with
  [Certificate Management](/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/)
  and [Upgrading kubeadm clusters](/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/).
  See [Administer a Cluster](/docs/tasks/administer-cluster/)
  for a longer list of Kubernetes administrative tasks.

To learn about available options when you run control plane services, see
[kube-apiserver](/docs/reference/command-line-tools-reference/kube-apiserver/),
[kube-controller-manager](/docs/reference/command-line-tools-reference/kube-controller-manager/),
and [kube-scheduler](/docs/reference/command-line-tools-reference/kube-scheduler/)
component pages. For highly available control plane examples, see
[Options for Highly Available topology](/docs/setup/production-environment/tools/kubeadm/ha-topology/),
[Creating Highly Available clusters with kubeadm](/docs/setup/production-environment/tools/kubeadm/high-availability/),
and [Operating etcd clusters for Kubernetes](/docs/tasks/administer-cluster/configure-upgrade-etcd/).
See [Backing up an etcd cluster](/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster)
for information on making an etcd backup plan.

### Worker-Nodes in einer Produktionsumgebung
### Production worker nodes

Production-quality workloads need to be resilient and anything they rely
on needs to be resilient (such as CoreDNS). Whether you manage your own
control plane or have a cloud provider do it for you, you still need to
consider how you want to manage your worker nodes (also referred to
simply as *nodes*).  

- *Configure nodes*: Nodes can be physical or virtual machines. If you want to
  create and manage your own nodes, you can install a supported operating system,
  then add and run the appropriate
  [Node services](/docs/concepts/architecture/#node-components). Consider:
  - The demands of your workloads when you set up nodes by having appropriate memory, CPU, and disk speed and storage capacity available.
  - Whether generic computer systems will do or you have workloads that need GPU processors, Windows nodes, or VM isolation.
- *Validate nodes*: See [Valid node setup](/docs/setup/best-practices/node-conformance/)
  for information on how to ensure that a node meets the requirements to join
  a Kubernetes cluster.
- *Add nodes to the cluster*: If you are managing your own cluster you can
  add nodes by setting up your own machines and either adding them manually or
  having them register themselves to the cluster’s apiserver. See the
  [Nodes](/docs/concepts/architecture/nodes/) section for information on how to set up Kubernetes to add nodes in these ways.
- *Scale nodes*: Have a plan for expanding the capacity your cluster will
  eventually need. See [Considerations for large clusters](/docs/setup/best-practices/cluster-large/)
  to help determine how many nodes you need, based on the number of pods and
  containers you need to run. If you are managing nodes yourself, this can mean
  purchasing and installing your own physical equipment.
- *Autoscale nodes*: Read [Node Autoscaling](/docs/concepts/cluster-administration/node-autoscaling) to learn about the
  tools available to automatically manage your nodes and the capacity they
  provide.
- *Set up node health checks*: For important workloads, you want to make sure
  that the nodes and pods running on those nodes are healthy. Using the
  [Node Problem Detector](/docs/tasks/debug/debug-cluster/monitor-node-health/)
  daemon, you can ensure your nodes are healthy.

## Production user management

In production, you may be moving from a model where you or a small group of
people are accessing the cluster to where there may potentially be dozens or
hundreds of people. In a learning environment or platform prototype, you might have a single
administrative account for everything you do. In production, you will want
more accounts with different levels of access to different namespaces.

Taking on a production-quality cluster means deciding how you
want to selectively allow access by other users. In particular, you need to
select strategies for validating the identities of those who try to access your
cluster (authentication) and deciding if they have permissions to do what they
are asking (authorization):

- *Authentication*: The apiserver can authenticate users using client
  certificates, bearer tokens, an authenticating proxy, or HTTP basic auth.
  You can choose which authentication methods you want to use.
  Using plugins, the apiserver can leverage your organization’s existing
  authentication methods, such as LDAP or Kerberos. See
  [Authentication](/docs/reference/access-authn-authz/authentication/)
  for a description of these different methods of authenticating Kubernetes users.
- *Authorization*: When you set out to authorize your regular users, you will probably choose
  between RBAC and ABAC authorization. See [Authorization Overview](/docs/reference/access-authn-authz/authorization/)
  to review different modes for authorizing user accounts (as well as service account access to
  your cluster):
  - *Role-based access control* ([RBAC](/docs/reference/access-authn-authz/rbac/)): Lets you
    assign access to your cluster by allowing specific sets of permissions to authenticated users.
    Permissions can be assigned for a specific namespace (Role) or across the entire cluster
    (ClusterRole). Then using RoleBindings and ClusterRoleBindings, those permissions can be attached
    to particular users.
  - *Attribute-based access control* ([ABAC](/docs/reference/access-authn-authz/abac/)): Lets you
    create policies based on resource attributes in the cluster and will allow or deny access
    based on those attributes. Each line of a policy file identifies versioning properties (apiVersion
    and kind) and a map of spec properties to match the subject (user or group), resource property,
    non-resource property (/version or /apis), and readonly. See
    [Examples](/docs/reference/access-authn-authz/abac/#examples) for details.

As someone setting up authentication and authorization on your production Kubernetes cluster, here are some things to consider:

- *Set the authorization mode*: When the Kubernetes API server
  ([kube-apiserver](/docs/reference/command-line-tools-reference/kube-apiserver/))
  starts, supported authorization modes must be set using an *--authorization-config* file or the *--authorization-mode*
  flag. For example, that flag in the *kube-adminserver.yaml* file (in */etc/kubernetes/manifests*)
  could be set to Node,RBAC. This would allow Node and RBAC authorization for authenticated requests.
- *Create user certificates and role bindings (RBAC)*: If you are using RBAC
  authorization, users can create a CertificateSigningRequest (CSR) that can be
  signed by the cluster CA. Then you can bind Roles and ClusterRoles to each user.
  See [Certificate Signing Requests](/docs/reference/access-authn-authz/certificate-signing-requests/)
  for details.
- *Create policies that combine attributes (ABAC)*: If you are using ABAC
  authorization, you can assign combinations of attributes to form policies to
  authorize selected users or groups to access particular resources (such as a
  pod), namespace, or apiGroup. For more information, see
  [Examples](/docs/reference/access-authn-authz/abac/#examples).
- *Consider Admission Controllers*: Additional forms of authorization for
  requests that can come in through the API server include
  [Webhook Token Authentication](/docs/reference/access-authn-authz/authentication/#webhook-token-authentication).
  Webhooks and other special authorization types need to be enabled by adding
  [Admission Controllers](/docs/reference/access-authn-authz/admission-controllers/)
  to the API server.

## Set limits on workload resources

Demands from production workloads can cause pressure both inside and outside
of the Kubernetes control plane. Consider these items when setting up for the
needs of your cluster's workloads:

- *Set namespace limits*: Set per-namespace quotas on things like memory and CPU. See
  [Manage Memory, CPU, and API Resources](/docs/tasks/administer-cluster/manage-resources/)
  for details.
- *Prepare for DNS demand*: If you expect workloads to massively scale up,
  your DNS service must be ready to scale up as well. See
  [Autoscale the DNS service in a Cluster](/docs/tasks/administer-cluster/dns-horizontal-autoscaling/).
- *Create additional service accounts*: User accounts determine what users can
  do on a cluster, while a service account defines pod access within a particular
  namespace. By default, a pod takes on the default service account from its namespace.
  See [Managing Service Accounts](/docs/reference/access-authn-authz/service-accounts-admin/)
  for information on creating a new service account. For example, you might want to:
  - Add secrets that a pod could use to pull images from a particular container registry. See
    [Configure Service Accounts for Pods](/docs/tasks/configure-pod-container/configure-service-account/)
    for an example.
  - Assign RBAC permissions to a service account. See
    [ServiceAccount permissions](/docs/reference/access-authn-authz/rbac/#service-account-permissions)
    for details.

## {{% heading "whatsnext" %}}

- Decide if you want to build your own production Kubernetes or obtain one from
  available [Turnkey Cloud Solutions](/docs/setup/production-environment/turnkey-solutions/)
  or [Kubernetes Partners](/partners/).
- If you choose to build your own cluster, plan how you want to
  handle [certificates](/docs/setup/best-practices/certificates/)
  and set up high availability for features such as
  [etcd](/docs/setup/production-environment/tools/kubeadm/setup-ha-etcd-with-kubeadm/)
  and the
  [API server](/docs/setup/production-environment/tools/kubeadm/ha-topology/).
- Choose from [kubeadm](/docs/setup/production-environment/tools/kubeadm/),
  [kops](https://kops.sigs.k8s.io/) or
  [Kubespray](https://kubespray.io/) deployment methods.
- Configure user management by determining your
  [Authentication](/docs/reference/access-authn-authz/authentication/) and
  [Authorization](/docs/reference/access-authn-authz/authorization/) methods.
- Prepare for application workloads by setting up
  [resource limits](/docs/tasks/administer-cluster/manage-resources/),
  [DNS autoscaling](/docs/tasks/administer-cluster/dns-horizontal-autoscaling/)
  and [service accounts](/docs/reference/access-authn-authz/service-accounts-admin/).
