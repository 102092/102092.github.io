---
author: Han
title: AWS EC2 instance issue
date: 2021-09-03
tags: ['bug', 'aws']
ShowToc: false
---

# 1. 버그 내용
- 특정 시간만 되면 혹은 간헐적이나 지속적으로, Redis 운영에 장애가 생김. (약 10분 ~ 20분)
- Error Log -> `... Redis command timed out;`
  - 로그 내용으로 보아, 어떠한 이유에서 인 지 레디스 멈췄고, 
  - 다른 작업들을 처리하지 못해서 생기는 이슈 같음.


# 2. 생각해볼 수 있는 원인
1. 특정 작업을 처리하는 데, 시간이 오래 걸려서, 레디스에서 병목이 생기는거 아닐까?
    - `slow log` 찾아볼 것
    - [SLOWLOG subcommand](https://redis.io/commands/slowlog)

2. 특정 시간에 너무 많은 키가 `expire` 되어서 발생한 문제가 아닐까?
    - 에러 발생한 시간 대, 키들의 `expire` 되고 있는 시간을 조절.

3. 해당 시간 진행되는 작업 중, 발생하는 트랜잭션에서 뭔가 문제가 발생한 건 아닐까?
    - 코드 점검
    - 트랜잭션 진행되고 있는 코드 확인.

4. AWS EC2 instance issue 일수도..?
    - [문서](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-lifecycle.html#lifecycle-differences) 를 참고하여, `reboot` -> `stop and start` 순으로 시도해보기.


# 3. 해결 시도
1. 우선 팀에서 필요하다고 생각되는 부분을 개선.
    - redis connection pool 조정.
    - redis tcp-keepalive 조정을 통한 connection 확인.
    - 추후 에러 빈도가 낮아지면, 사용 중인 라이브러리를 `JEDIS` -> `Lettuce` 로 변경.
        - `Lettuce` 의 경우, 커넥션 풀 없이 사용할 수 있도록 디자인 되어 있어서 선택.

2. 1번 해결방안 시도 후, 어느정도 에러 발생 빈도가 줄어드는 것 같이 보였지만, 특정 시간에 발생하는 레디스 에러는 여전히 있었음.
    - 즉 connection pool로 인해 발생된 에러가 아닌듯.
    - 작성된 코드 이슈?
        - 에러가 발생하는 코드 중에, 레디스 키를 expire 하는 부분에 delay를 줌.
        - 또한, [SMEMEBER](https://redis.io/commands/smembers) 사용하는 코드를 [SSCAN](https://redis.io/commands/sscan) 으로 변경하는 등의 개선 작업 진행.


3. 여전히 발생. 또한 레디스에서 파생된 에러로 인해 `Database` 까지 전파되는듯 함.
    - 해당 에러가 발생하는 시간 대에, 인스턴스 내부에 cronjob 을 수행되나, `ssh` 를 통한 접근이 불가함을 파악
    - 또한, 해당 시간 전에 AWS EC2 검사 실패가 발생함을 확인.
    - AWS EC2 Reboot 하기로 결정.
        - 서버 off -> redis-save -> ec2 reboot -> redis on 확인 -> 서버 on 과정을 진행 
        - (다행히 5분내에 끝났다.)

4. Redis 랑 지속적으로 커넥션을 맺고 있는 서버 내림(에러 발생 원인을 명확하게 하기 위해). 
    - AWS EC2 Detailed monitoring 활성화.
    - 그래도 에러 발생 (AWS EC2 instance 검사 실패, 인스턴스 접속 불가, Redis monioring 접근 불가)
    - 네크워크 이슈?
    - AWS EC2 stop -> start를 통해 호스트 변경.
    - 다행스럽게도 해결

# 5. 정리
- 발생한 이슈는 **AWS EC2 Instacne 이슈**였던듯 싶음.
- 지금 되돌아보면, 몇가지 시그널을 있었던 듯. (네트워크 관련..)
- 이중화.. 구조의 필요성을 느꼈음.
    - 하나가 장애가 나더라도, 바로 대체할 수 있도록.

# 6. 참고
- https://jihooyim1.gitbooks.io/linuxbasic/content/contents/08.html
- https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-lifecycle.html#lifecycle-differences
- https://redis.io/commands/
