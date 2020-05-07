---
title: AWS Account and User
weight: 200
pre: "<b>2-2. </b>"

---

## AWS Account 생성

워크샵을 수행하기 위해서는 AWS 계정이 있어야 합니다. 이미 계정이 있고 관리자 권한이 있는 사용자 credential 로 시스템 설정이 되어 있는 경우 [다음 단계](./300-nodejs.html)로 이동하십시오.

{{% notice warning %}}
이미 존재 하는 개인/법인 계정을 이용하는 경우, 해당 계정에 자원을 생성하게 되므로 유의하십시오.
{{% /notice %}}

AWS 계정이 존재하지 않는 경우 [무료로 계정 생성하기](https://portal.aws.amazon.com/billing/signup) 페이지를 통해 새로운 계정을 생성하십시오.

## 관리자 권한이 있는 User 생성하기

2. AWS 계정으로 [콘솔](https://console.aws.amazon.com)에 로그인
3. AWS IAM 콘솔로 이동하여 [새로운 User 생성](https://console.aws.amazon.com/iam/home?#/users$new) 메뉴 클릭.
4. User 이름을 지정하고 (예: `cdk-workshop`), "Programmatic access"를 선택하십시오.

    ![](/images/10-preq/new-user-1.png)

5. **Next: Permissions** 을 선택하여 다음 단계로 이동하십시오.
6. **Attach existing policies directly** 를 클릭한 뒤 **AdministratorAccess**를 선택하십시오.

    ![](/images/10-preq/new-user-2.png)

7. **Next: Review** 를 클릭하십시오.
8. **Create User** 를 클릭하십시오.
9. 다음 화면에서 **Access key ID** 를 확인할 수 있습니다. **Show** 버튼을 클릭하면 **Secret access key** 역시 확인할 수 있습니다. 이 정보가 AWS CLI 설정에 필요하오니 윈도우를 열어두십시오. 

    ![](/images/10-preq/new-user-3.png)

## Credential 설정

터미널 창을 열고 `aws configure` 명령어를 이용해 환경 설정을 진행합니다.
위에서 확인한 __access key ID__ 와 __secret key__ 를 입력하고, 기본 리전 (서울: `ap-northeast-2`, 도쿄: `ap-northeast-1`, 싱가포르: `ap-southeast-1`...)을 지정하십시오.. 
향후 워크샵 진행을 원활히 하기 위해, 되도록 자원을 많이 생성해두지 않은 리전을 선택하십시오.

```
aws configure
```

콘솔에서 아래 정보 입력:

```
AWS Access Key ID [None]: <key ID 입력>
AWS Secret Access Key [None]: <access key 입력>
Default region name [None]: <region 선택(서울: `ap-northeast-2`, 도쿄: `ap-northeast-1`, 싱가포르: `ap-southeast-1`...)>
Default output format [None]: <입력하지 않아도 됨.>
```
