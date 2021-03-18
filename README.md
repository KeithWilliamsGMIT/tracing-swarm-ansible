# Tracing Swarm Ansible

This repository shows how to setup Jaeger and the OpenTelemetry collector running in Docker Swarm to provide tracing for other services running in Docker Swarm. Tracing is one of the pillars of observability, along with monitoring and logging. Ansible is used to automate the various tasks.

## Getting Started

The first step to getting started is to clone the repository:

```bash
git clone https://github.com/KeithWilliamsGMIT/tracing-swarm-ansible.git
```

Next, install Ansible and then we can use two existing Ansible roles to install Docker and initialise a Docker Swarm. These roles are defined in the `requirements.yml` file and will be installed to `~/.ansible/roles/` by default using the below commands:

```bash
cd playbooks/requirements
ansible-galaxy install -r requirements.yml
```

We can install [Docker](https://docs.docker.com/install/) on all the hosts now using the `install-docker.yml` playbook as shown below:

```bash
cd playbooks
ansible-playbook -i ./inventories/localhost install-docker.yml
```

Similarly, to initialise a Docker Swarm between the hosts we can run the `initialize-swarm.yml` playbook. Note that we set extra variables to override some defaults in the playbook. We want to skip everything except the actual initialisation of Docker Swarm.

```bash
ansible-playbook -i ./inventories/localhost initialize-swarm.yml --extra-vars="{'skip_engine': 'True', 'skip_group': 'True', 'skip_docker_py': 'True'}"
```

Finally, we can deploy the stack containing the services to Docker Swarm using the `deploy-tracing.yml` playbook.

```bash
ansible-playbook -i ./inventories/localhost deploy-tracing.yml --ask-become-pass
```

## Does it work?

Once you followed the above steps we can check if the services are running as expected on the docker manager.

```bash
ubuntu:~/tracing-swarm-ansible/playbooks$ sudo docker service ls
ID             NAME                MODE         REPLICAS   IMAGE                                           PORTS
sof2mubtksbz   tracing_collector   replicated   1/1        otel/opentelemetry-collector:latest             *:9464->9464/tcp, *:55680-55681->55680-55681/tcp
i7xmz7ppd2ad   tracing_jaeger      replicated   1/1        jaegertracing/all-in-one:latest                 *:5778->5778/tcp, *:9411->9411/tcp, *:14268->14268/tcp, *:16686->16686/tcp, *:5775->5775/udp, *:6831-6832->6831-6832/udp
```

We can now go to the Jaeger UI by navigating to [http://localhost:16686](http://localhost:16686).

## Further configuration

### Ansible variables

There are also a number of Ansible variables that can be overridden. These can be found in the table below:

| Variable Name | Description | Default Value |
|---------------|-------------|---------------|
| temporary_directory | A path to a directory where temporary files such as the compose file can be written to | "/tmp" |
| volume_directory | A path to a directory where service data can be persisted | "/tmp" |
| docker_network_name | The name of the new Docker overlay network | "tracing_network" |
| jaeger_version | The tag of the Jaeger all-in-one Docker image to use | latest |
| opentelemetry_collector_version | The tag of the OpenTelemetry Collector Docker image to use | latest |

## Using this role

To use this role add the following to your `requirements.yml` file:

```
- src: https://github.com/KeithWilliamsGMIT/tracing-swarm-ansible.git
  version: master
  name: deploy-tracing
```

## Troubleshooting

Here are a few tips you can try to troubleshoot any issues you may encounter:

+ Make sure that all the services are running as expected.

## Contributing

Any contribution to this repository is appreciated, whether it is a pull request, bug report, feature request, feedback or even starring the repository. Some potential areas that need further refinement are:

+ Hardening of role
+ Publishing role to Ansible Galaxy
+ Documentation

## Conclusion

This repository demonstates how to setup Jaeger to trace requests between services. This is key for observability particularily in a microservice architecture. Finally, running all of these commands manually everytime we need to deploy the stack would be time consuming and so Ansible is quite useful in automating this process.
