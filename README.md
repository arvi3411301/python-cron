# python-cron

This project consists of a basic Hasura project with a simple cron job app running on it. Once this app is deployed on a Hasura cluster, you will have the app running at `https://www.<cluster-name>.hasura-app.io`

This is the right place to start if you are planning to build or want to learn to build an app with Hasura.

## Sections

* [Introduction](#introduction)
* [Quickstart](#quickstart)
* [Adding your own code](#adding-your-existing-code)
* [Local development](#local-development)
* [FAQ](#faq)

## Introduction

This quickstart project comes with the following by default:

1. A basic Hasura project

## Quickstart

Follow this section to get this project working. Before you begin, ensure you have the latest version of [hasura cli tool](https://docs.hasura.io/0.15/manual/install-hasura-cli.html) installed.

### Step 1: Getting the project

```sh
$ hasura quickstart python-cron
$ cd python-cron
```

The above command does the following:
1. Creates a new folder in the current working directory called `python-cron`
2. Creates a new free Hasura cluster for you and sets that cluster as the default cluster for this project
3. Initializes `python-cron` as a git repository and adds the necessary git remotes.

### Step 2: Deploying this project

To deploy the project:

```sh
$ git add .
$ git commit -m "Initial Commit"
$ git push hasura master
```
When you push for the first time, it might take sometime. Next time onwards, it is really fast.

Once the above commands are executed successfully, head over to `https://www.<cluster-name>.hasura-app.io` (in this case `https://www.h34-excise98-stg.hasura-app.io`) to view your app.

## Adding your existing code
The microservice[1] sample code is inside the `microservices/www/cron` directory. You can copy all your existing code directly inside this directory, and start deploying your own code to Hasura cluster.

### Step 1: Add your code in the microservices directory
Copy all your exising source code in `microservices/www/cron` directory or replace the `microservices/www/cron` directory with your cron directory. Ensure that the structure of the directory is coherent with the current structure.

### Step 2: How to use CronTab Module
Getting access to a crontab can happen via tabfile method:

```python
file_cron = CronTab(tabfile='filename.tab')
```

Tabfile name can be changed by the following steps

1. Edit the filename in the Dockerfile present in `microservices/www/cron`

    ```
    RUN touch filename.tab
    ```

2. Edit the filename in `main.py` present in `microservices/www/cron`

    ```python
    my_cron = CronTab(tabfile='filename.tab')
    ```

Creating a new job is as simple as:

```python
job  = cron.new(command='/usr/bin/echo')
```

And setting the job’s time restrictions:

```python
job.minute.during(5,50).every(5)
job.hour.every(4)
job.day.on(4, 5, 6)

job.dow.on('SUN')
job.dow.on('SUN', 'FRI')
job.month.during('APR', 'NOV')
```

Each time restriction will clear the previous restriction:

```python
job.hour.every(10) # Set to * */10 * * *
job.hour.on(2)     # Set to * 2 * * *
```

Appending restrictions is explicit:

```python
job.hour.every(10)  # Set to * */10 * * *
job.hour.also.on(2) # Set to * 2,*/10 * * *
```

Setting all time slices at once:

```python
job.setall(2, 10, '2-4', '*/2', None)
job.setall('2 10 * * *')
```

Setting the slice to a python date object:

```python
job.setall(time(10, 2))
job.setall(date(2000, 4, 2))
job.setall(datetime(2000, 4, 2, 10, 2))
```

Disabled or Enable Job:

```python
job.enable()
job.enable(False)
```

Clean a job of all rules:

```python
job.clear()
```

Write CronTab back to system or filename:

```python
cron.write()
```

Running the scheduler

```python
my_cron = CronTab(tabfile='my_cron.tab')
for result in my_cron.run_scheduler():
    print "This was printed to stdout by the process."
```

### Step 3: Git add and commit
```
$ git add .
$ git commit -m "Added my Cron code"
```

### Step 4: Deploy
```
$ git push hasura master
```
Now your application should be running at: `https://www.<cluster-name>.hasura-app.io`

[1] a microservice is a running application on the Hasura cluster. This could be an www, a web app, a Javascript app etc.

## Hasura API console

Every Hasura cluster comes with an api console that gives you a GUI to test out the BaaS features of Hasura. To open the api console

```sh
$ hasura api-console
```

## Custom Microservice

There might be cases where you might want to perform some custom business logic on your apis. For example, sending an email/sms to a user on sign up or sending a push notification to the mobile device when some event happens. For this, you would want to create your own custom microservice which does these for you on the endpoints that you define.

This quickstart comes with one such custom microservice written in Python using the CronTab module. Check it out in action at `https://www.cluster-name.hasura-app.io` . Currently, it just returns a JSON response of "Hello World" at that endpoint.

In case you want to use another language/framework for your custom microservice. Take a look at our docs to see how you can add a new custom microservice.

## Local development

Everytime you push, your code will get deployed on a public URL. However, for faster iteration you should locally test your changes.

### Testing your app locally

Follow these steps to test out your app locally

```sh
$ cd microservices/cron/
$ docker build -t python-cron:<tag> .
$ docker run -d python-cron:<tag>
```

## Files and Directories

The project (a.k.a. project directory) has a particular directory structure and it has to be maintained strictly, else `hasura` cli would not work as expected. A representative project is shown below:

```
.
├── hasura.yaml
├── clusters.yaml
├── conf
│   ├── authorized-keys.yaml
│   ├── auth.yaml
│   ├── ci.yaml
│   ├── domains.yaml
│   ├── filestore.yaml
│   ├── gateway.yaml
│   ├── http-directives.conf
│   ├── notify.yaml
│   ├── postgres.yaml
│   ├── routes.yaml
│   └── session-store.yaml
├── migrations
│   ├── 1504788327_create_table_user.down.yaml
│   ├── 1504788327_create_table_user.down.sql
│   ├── 1504788327_create_table_user.up.yaml
│   └── 1504788327_create_table_user.up.sql
└── microservices
    └── www
        ├── app/
        ├── k8s.yaml
        └── Dockerfile
```

### `hasura.yaml`

This file contains some metadata about the project, namely a name, description and some keywords. Also contains `platformVersion` which says which Hasura platform version is compatible with this project.

### `clusters.yaml`

Info about the clusters added to this project can be found in this file. Each cluster is defined by it's name allotted by Hasura. While adding the cluster to the project you are prompted to give an alias, which is just hasura by default. The `kubeContext` mentions the name of kubernetes context used to access the cluster, which is also managed by hasura. The `config` key denotes the location of cluster's metadata on the cluster itself. This information is parsed and cluster's metadata is appended while conf is rendered. `data` key is for holding custom variables that you can define.

```yaml
- name: h34-ambitious93-stg
  alias: hasura
  kubeContext: h34-ambitious93-stg
  config:
    configmap: controller-conf
    namespace: hasura
  data: null
```
