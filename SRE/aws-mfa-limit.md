# AWS MFA 적용 시 알아 두어야 할 점

## AWS CLI 에서는 MFA 검증을 하지 않게끔 설정 가능

* IAM Policy

```bash
  {
        "Sid": "DoNotAllowAnythingOtherThanAboveUnlessMFAd",
        "Effect": "Deny",
        "NotAction": [
            "iam:*",
            "sts:AssumeRole",
            "sts:GetSessionToken"
        ],
        "Resource": "*",
        "Condition": {
            "Bool": {
                "aws:MultiFactorAuthPresent": "false"
            }
        }
    }
```

* 위 정책에 의해 aws resource 접근 요청 시,
  * aws:MultiFactorAuthPresent 가 제공 되지 않을 경우,
  * aws:MultiFactorAuthPresent 가 false 인 경우,
  * 요청이 거부 됨

* aws cli 로 aws resource 접근 요청 시, `aws:MultiFactorAuthPresent` 는 존재 하지 않음
  * 그러므로 Bool Condition 에서는 `aws:MultiFactorAuthPresent` 는 존재 하지 않음 은 무시 되므로 접근이 허용 됨

* `aws:MultiFactorAuthPresent` 는 임시 보안 자격 증명 을 사용 하는 경우에만 존재 함
  * 임시 보안 자격 증명을 사용 하는 경우는 aws web console 등에서 사용 됨
    > 임시 자격 증명은 IAM 역할, 연합된 사용자, sts:GetSessionToken의 임시 토큰을 가진 IAM 사용자 및 AWS Management Console의 사용자를 인증하는 데 사용됩니다.

* aws cli, api 으로 aws resource 에 접근 하는 경우에는 임시 자격 증명이 아닌 장기 자격 증명을 사용함
  > 사용자 액세스 키 페어와 같은 장기 자격 증명으로 API 또는 CLI 명령을 호출

* 그러므로, aws cli 로 aws resource 를 접근 하는 경우 `aws:MultiFactorAuthPresent` 가 존재 하지 않음

* 참고 문서
  * <https://docs.aws.amazon.com/ko_kr/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-multifactorauthpresent>
  * <https://aws.amazon.com/ko/premiumsupport/knowledge-center/mfa-iam-user-aws-cli/?nc1=h_ls>

## 하지만 모든 AWS CLI 가 MFA 검증을 회피할 수 있는 것은 아님

* 다음 aws command 실행 시 AWS Resource 접근 불가 발생

```bash
  aws ec2-instance-connect send-ssh-public-key \
    --instance-id $INSTANCE_ID \
    --availability-zone $AVAILABILITY_ZONE \
    --instance-os-user ec2-user \
    --ssh-public-key file://$PUB_KEY_PATH > /dev/null \
    --profile=$AWS_PROFILE
```

* 기본적인 aws cli command 는 정상적으로 실행 됨. 다만, 위 command 만 실행 시 접근 불가 현상 발생

## AWS CLI 에서만 MFA 검증을 회피 하는 것은 올바르지 않는 방법

* 접근 편의성을 위해 AWS CLI 만 MFA 검증 회피를 위해 지속적으로 예외적인 케이스를 허용 하기 위해 IAM MFA 정책을 복잡 하게 하기 보다는,

* mfa 를 적용 할지 말지 에 대해 결정 하는 수준으로 간단하게 해소 하는 것이 더 나은 방법

* mfa 를 적용 할 거면 aws cli 만 허용 하는 등의 예외를 두지 말고, 모든 aws resource 접근 요청에 대해 mfa 에 대한 검증 수행

## AWS CLI 에서 MFA 인증을 하는 방법

* 참고 링크 <https://aws.amazon.com/ko/premiumsupport/knowledge-center/authenticate-mfa-cli/>
