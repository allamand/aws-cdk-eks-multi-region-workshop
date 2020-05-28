---
title: 샘플 앱 클론하기
weight: 100
pre: "<b>6-1. </b>"
---

아래 명령어를 수행하여 EKS 클러스터에 배포할 어플리케이션 코드를 내려받습니다.


### 샘플 어플리케이션 클론
```
git clone https://github.com/yjw113080/aws-cdk-multi-region-sample-app
```

* 지금까지 워크샵을 진행한 `skeleton` 폴더 외부에 클론되도록 해주십시오. 


### IDE에서 열기
선호하시는 IDE에서 위 프로젝트 폴더를 열어 주십시오.


### 어플리케이션 내용 확인
IDE를 열면 아래와 같이 세 가지 파일이 들어 있음을 확인할 수 있습니다.
1. app.py: 실제 로직 부분. 
2. Dockerfile: 이 파이썬 코드를 컨테이너 이미지로 빌드하기 위한 Dockerfile
3. app-deployment.yaml: 쿠버네티스 클러스터에 배포할 내용을 정의하는 명세.

app.py 파일을 열어 보면, 아래와 같이 컨테이너가 실행되고 있는 환경의 리전 정보를 보여주는 간단한 파이썬 어플리케이션을 볼 수 있습니다.

```python
@app.route('/')
def hello():
  res = requests.get('http://169.254.169.254/latest/dynamic/instance-identity/document')
  data = json.loads(res.text)
  region = data['region']
  return ("Hello World from "+region)
```