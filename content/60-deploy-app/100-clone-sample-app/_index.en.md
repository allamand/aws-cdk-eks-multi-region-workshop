---
title: Clone the sample app
weight: 100
pre: "<b>6-1. </b>"
---


Clone the application code which will be deployed in EKS clusters.


### Clone the sample application
```
cd ..
git clone https://github.com/yjw113080/aws-cdk-multi-region-sample-app
```

* Please make sure that it is cloned outside the 'skeleton' folder where you have conducted the workshop so far.


### Open in IDE
Open the above project folder in your favorite IDE.


### Check application contents


When you open the IDE, you can see that it contains three files except README.
1. app.py: the actual logic part.
2. Dockerfile: Dockerfile to build this Python code into a container image
3. app-deployment.yaml: Specification that defines what to deploy to the Kubernetes cluster.

If you open the app.py file, you can see a simple Python         3e4w application showing the region information of the environment where the container is running.

```python
@app.route('/')
def hello():
  res = requests.get('http://169.254.169.254/latest/dynamic/instance-identity/document')
  data = json.loads(res.text)
  region = data['region']
  return ("Hello World from "+region)
```